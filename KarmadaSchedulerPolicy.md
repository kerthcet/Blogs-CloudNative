## 项目介绍
[Karmada](https://github.com/karmada-io/karmada)是一个多云多集群 Kubernetes 编排系统，填补了 Kubernetes 在多集群管理方面的空白，更多详细介绍可以参考 Github 主页，本文主要是对 Karmada 调度策略做一些介绍。

    版本号: karmada-io/karmada@be2f596ecf

## 整体架构
![image](https://github.com/kerthcet/Blogs-CloudNative/blob/main/snapshots/karmada-arch.png)

这是 karmada 整体架构图，他将整个环境分成了两类集群：
* Host集群

    karmada 控制面集群，主要有两个控制面，一个用于部署 karmada 集群，而另一个则是真正的 karmada 控制面集群。

    控制面集群组件：
    * API-Server: 功能类似 kubernetes apiserver，可以直接使用 kubernetes 原生的 API 资源
    * Karmada Scheduler: 区别于 `kube-system` 中的 scheduler，kube-system 中的 scheduler只是用于负责部署 karmada 组件，这里的 scheduler 指的是 `karmada-system` 中的scheduler，工作在集群层面，将应用调度到各个 member 集群中
    * ETCD：存储所有的资源对象
    * Karmada Controllers: 主要包含了4个 controller，配置 scheduler 进行多云多集群的资源编排和调度

        * ClusterController: 负责 member 集群的生命周期管理
        * PolicyController: 负责监听 `PropagationPolicy` 资源对象和 kubernetes 原生资源对象（也包括 CRD 资源对象），实现两者的绑定并创建对应的 `ResourceBinding` 资源
        * BindingController: 将 `ResourceBinding` 转换为 `work` 对象，如果有对应的 `override policy` 则会运用该 `policy`
        * ExecutionController: 将 `work` 中包含的 kubernetes 资源对象同步到 member 集群中
* Member集群

    由 N 个 kubernetes 集群组成，运行实际的 workloads。

## 调度逻辑
整体调度逻辑如下图所示，图片逻辑已经比较清楚了，这里就不展开。

![image](https://github.com/kerthcet/Blogs-CloudNative/blob/main/snapshots/karmada-schedule-process.png)


## 调度策略
我们先简单部署一个 karmada 集群，直接在该项目根目录运行 `"./hack/local-up-karmada.sh"` 就可以通过 `kind` 部署一个4节点集群，一个 `master` 节点和 3个 `member` 节点。假设我们在控制集群创建了一个副本数为5的 `nginx deployment`，现在要创建 `PropagationPolicy` 进行调度。

### 副本机制
1. Duplicated 策略
    ```yaml
    kind: PropagationPolicy
    metadata:
      name: nginx-propagation
    spec:
      resourceSelectors:
      - apiVersion: apps/v1
        kind: Deployment
        name: nginx
      placement:
        clusterAffinity:
          clusterNames:
          - member1
          - member2
        replicaScheduling:
          replicaSchedulingType: Duplicated
    ```

    简单的说就是复制，所有集群副本数都相同，此时 `member1` 和 `member2` 集群都拥有5个 `nginx` 副本，一共有10个副本。此类型适合配置同构的集群。

2. Divided 策略
    ```yaml
    kind: PropagationPolicy
    metadata:
      name: nginx-propagation
    spec:
      resourceSelectors:
      - apiVersion: apps/v1
        kind: Deployment
        name: nginx
      placement:
        replicaScheduling:
          replicaDivisionPreference: Weighted
          replicaSchedulingType: Divided
          weightPreference:
            staticWeightList:
            - targetCluster:
                clusterNames:
                - member1
              weight: 2
            - targetCluster:
                clusterNames:
                - member2
              weight: 3
    ```

    `Divided` 策略下`replicaDivisionPreference` 选择 `Weighted` 表示按照权重来分配副本数量, 此时，`member1` 拥有2个副本，`member2` 拥有3个副本。

    `replicaDivisionPreference` 值还可以是 `Aggregated`，表示根据资源空闲情况调度到尽可能少的集群。此时可能的情况是所有的5个副本都调度到了 `member1`。

### 集群调度机制
1. clusterAffinity
    ```yaml
    kind: PropagationPolicy
    metadata:
      name: nginx-propagation
    spec:
      resourceSelectors:
      - apiVersion: apps/v1
        kind: Deployment
        name: nginx
      placement:
        clusterAffinity:
          clusterNames:
          - member1
          - member2
    ```
    `clusterAffinity` 功能类似于 `nodeAffinity` 和 `podAffinity`，属于亲和性调度。

2. clusterTolerations
    ```yaml
    apiVersion: policy.karmada.io/v1alpha1
    kind: PropagationPolicy
    metadata:
      name: nginx-propagation
    spec:
      resourceSelectors:
        - apiVersion: apps/v1
          kind: Deployment
          name: nginx
      placement:
        clusterAffinity:
          clusterNames:
            - member1
            - member2
        clusterTolerations:
          - effect: "NoSchedule"
    ```
    污点容忍性调度策略，用法同 `pod tolerations`。


3. spreadConstraints
    ```yaml
    apiVersion: policy.karmada.io/v1alpha1
    kind: PropagationPolicy
    metadata:
      name: example-policy
      namespace: default
    spec:
      resourceSelectors:
        - apiVersion: apps/v1
          kind: Deployment
          name: nginx
      association: false
      placement:
        spreadConstraints:
          - spreadByLabel: failuredomain.kubernetes.io/zone
            maxGroups: 3
            minGroups: 3
    ```
   分布约束，`maxGroups` 表示支持最多的集群数，`minGroups` 则表示最少支持的集群数，默认是1。除了 `spreadByLabel`，还支持 `spreadByField`, `field` 目前有4类： `cluster`， `region`，`zone`， `provider`。`spreadByLabel` 和 `spreadByField` 不能共存。

### Cluster Accurate Scheduler Estimator
这是最近加入到 `karmada` 的组件，主要是用于检测调度时集群是否有足够的资源，如果没有可用资源，就放弃调度到该集群。目前作为 `scheduler` 的组件默认启动。只能工作在 `replicaSchedulingType=Divided`，`replicaDivisionPreference=Aggregated` 的模式下。

### Descheduler
`descheduler` 目前还处于开发阶段，工作原理同 [`kubernetes-sigs/descheduler`](https://github.com/kubernetes-sigs/descheduler) 相同，主要解决资源调度成功后，后续集群资源调整，进行动态重调度，可参考 `kubernetes descheduler`。

### 总结
`karmada` 目前已经作为 `sandbox` 项目由华为捐赠给了 `CNCF`，但是整个 `Roadmap` 还是有很多功能没有完成，我自己最近也在为该项目贡献代码，希望后续可以持续发力。

![image](https://github.com/kerthcet/Blogs-CloudNative/blob/main/snapshots/karmada-roadmap.png)