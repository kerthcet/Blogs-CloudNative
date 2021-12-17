## 一、介绍
Kubernetes 调度器 kube-scheduler 通过预选和优选两个阶段来决策当前 pod 调度到哪个节点，整个决策过程是基于当前集群的快照，通过具体配置的调度策略最终决定，但是整个集群是动态的，不管是 node 还是 pod 可能会不断迁移和重建，比如节点下线，新增节点，节点标签和污点发生变化等等，此时，不管是出于集群资源优化的需求还是纠正一些不符合预期的调度结果，需要对整个集群进行重新调度，`descheduler` 应运而生。


## 二、整体配置

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    nodeSelector: prod=dev
    evictLocalStoragePods: true
    evictSystemCriticalPods: true
    maxNoOfPodsToEvictPerNode: 40
    ignorePvcPods: false
    strategies:
        ...


## 三、驱逐策略
### 1. RemoveDuplicates

场景：应用高可用部署会尽可能分散部署服务副本，但是可能遇到某些节点异常下线，导致一个节点存在多副本情况，当下线的节点重新上线后，该策略可以对这些副本进行重调度，重新打散。

例子：

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "RemoveDuplicates":
            enabled: true
            params:
                removeDuplicates:
                    excludeOwnerKinds:
                    - "ReplicaSet"


### 2. LowNodeUtilization

场景：集群节点资源占用不均衡，某些节点资源使用过载，某些节点资源未充分利用，此时需要对集群资源进行再分配。

例子，targetThresholds 表示上水位，超过该水位表示该节点资源使用过载。thresholds 表示下水位。低于该水位则表示资源未充分利用。该例子表示过载节点（cpu 超过50%， mem 超过50%， pods 数超过50）将驱逐部分pod到未充分利用的节点（cpu低于50%，memory低于50%，pod数低于50）。

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "LowNodeUtilization":
            enabled: true
            params:
                nodeResourceUtilizationThresholds:
                    thresholds:
                        "cpu" : 20
                        "memory": 20
                        "pods": 20
                    targetThresholds:
                        "cpu" : 50
                        "memory": 50
                        "pods": 50

### 3. HighNodeUtilization

场景：集群运行尽可能少的节点，配合CA可以降低集群成本，需要和 `MostRequestedPriority` 配合使用，

例子，表示资源未充分利用节点（cpu低于20%，memory低于20%，pod数低于20）将会驱逐自身的 pod。

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "HighNodeUtilization":
            enabled: true
            params:
                nodeResourceUtilizationThresholds:
                    thresholds:
                    "cpu" : 20
                    "memory": 20
                    "pods": 20

### 4. RemovePodsViolatingInterPodAntiAffinity

场景：集群节点状态在不断变化，可能pod刚开始调度的时候符合 `nodeAffinity` ，后面已经不再满足，但是现在没有类似 `requiredDuringSchedulingRequiredDuringExecution` 这种强制驱逐的功能。

例子：

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "RemovePodsViolatingNodeAffinity":
            enabled: true
            params:
            nodeAffinityType:
            - "requiredDuringSchedulingIgnoredDuringExecution"

### 5. RemovePodsViolatingNodeTaints

场景：类似 `RemovePodsViolatingInterPodAntiAffinity`，驱逐不符合污点的 pod

例子：

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "RemovePodsViolatingNodeTaints":
            enabled: true

### 6. RemovePodsViolatingTopologySpreadConstraint

场景：驱逐不符合拓扑分布约束的 pod

例子：

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "RemovePodsViolatingTopologySpreadConstraint":
            enabled: true
            params:
                includeSoftConstraints: false

### 7. RemovePodsHavingTooManyRestarts

场景：pod 调度到某个节点后无法成功挂载，一直在重启，此时需要调度到其他节点。

例子，includingInitContainers 表示是否包含init容器

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "RemovePodsHavingTooManyRestarts":
            enabled: true
            params:
                podsHavingTooManyRestarts:
                    podRestartThreshold: 100
                    includingInitContainers: true

### 8. PodLifeTime

场景：驱逐处于长时间 Running 或 Pending 状态的 pod

例子，podStatusPhases 目前支持 Running 和 Pending 两种 Phase

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "PodLifeTime":
            enabled: true
            params:
                podLifeTime:
                    maxPodLifeTimeSeconds: 86400
                    podStatusPhases:
                    - "Pending"

### 9. RemoveFailedPods

场景：驱逐失败的pod

例子：

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "RemoveFailedPods":
            enabled: true
            params:
                failedPods:
                    reasons:
                    - "NodeAffinity"
                    includingInitContainers: true
                    excludeOwnerKinds:
                    - "Job"
                    minPodLifetimeSeconds: 3600


## 四、过滤策略
### 1. Namespace

使用 `include` 和 `exclude` 进行策略配置：

例子：

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "PodLifeTime":
            enabled: true
            params:
                podLifeTime:
                    maxPodLifeTimeSeconds: 86400
                namespaces:
                    include:
                    - "namespace1"
                    - "namespace2"

### 2. Priority

低于指定 Priority 值的 pod 才能被驱逐，可以直接配置 Priority 或者指定 PriorityClass, 默认取值为 `system-cluster-critical`

例子：

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "PodLifeTime":
            enabled: true
            params:
                podLifeTime:
                    maxPodLifeTimeSeconds: 86400
                thresholdPriority: 10000
    ---
    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "PodLifeTime":
            enabled: true
            params:
                podLifeTime:
                    maxPodLifeTimeSeconds: 86400
                thresholdPriorityClassName: "priorityclass1"

### 3. Label
例子：

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "PodLifeTime":
            enabled: true
            params:
                podLifeTime:
                    maxPodLifeTimeSeconds: 86400
                labelSelector:
                    matchLabels:
                        component: redis
                    matchExpressions:
                    - {key: tier, operator: In, values: [cache]}
                    - {key: environment, operator: NotIn, values: [dev]}

### 4. Node Fit
当 nodeFit 设置为 true，表示驱逐前需要确认驱逐后 pod 是否可以被重新调度到其他节点，如果不行，则拒接驱逐操作

例子：

    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
        "LowNodeUtilization":
            enabled: true
            params:
                nodeResourceUtilizationThresholds:
                    thresholds:
                        "cpu" : 20
                        "memory": 20
                        "pods": 20
                    targetThresholds:
                        "cpu" : 50
                        "memory": 50
                        "pods": 50
                    nodeFit: true


## 五、驱逐约束
1. 关键pod（priorityClassName 是 system-cluster-critical/system-node-critical）不会被驱逐，除非设置`evictSystemCriticalPods: true`
2. 孤立的 pod 不会被驱逐，因为他们删除后不会被重建，比如 static pod。
3. DaemonSet不会被驱逐
4. 使用本地存储的 pod 不会被驱逐，除非设置 `evictLocalStoragePods: true`
5. 使用 `LowNodeUtilization` 和 `RemovePodsViolatingInterPodAntiAffinity` 时，pod 会按照优先级从小到大的顺序被驱逐，如果优先级相同，则根据 qos 顺序驱逐
6. 正在被删除的 pod 默认不驱逐
7. pod 可以通过添加注释 `descheduler.alpha.kubernetes.io/evict` 来表示自己可以被驱逐，所有类型的 pod 都生效，比如之前无法驱逐的 static pod。


## 六、运行
descheduler 支持以 [job](https://github.com/kubernetes-sigs/descheduler/blob/master/kubernetes/job/job.yaml)、[cronjob](https://github.com/kubernetes-sigs/descheduler/blob/master/kubernetes/cronjob/cronjob.yaml)、[deployment](https://github.com/kubernetes-sigs/descheduler/blob/master/kubernetes/deployment/deployment.yaml) 3种方式运行，deployment 通过设置启动参数 `--descheduling-interval` 配置执行间隔。


## 七、其他
1. `Descheduler` 尊重 `PDB`，所以可以和 PDB 结合使用保证重调度过程中服务的稳定性
2. 配合 `NPD` 和 `CA` 更好的进行集群自动化运维


## 八、 Roadmap
1. 支持 `pod affinity` 策略
2. 支持 `pending pods` 数量相关策略
3. 继续完善和 CA 的功能集成
4. 完善和 metrics 的集成
5. 考虑引入 kube-scheduler 预选流程


## 九、参考
1. https://github.com/kubernetes-sigs/descheduler