## Kube-Scheduler
kube-scheduler 是 kubernetes 的核心模块，属于控制面进程，负责将 pods 指派到节点上。

### SIG 介绍
1. 核心成员：[sig-scheduling](https://github.com/kubernetes/community/tree/master/sig-scheduling) 小组负责维护整个项目，目前该 sig co-chair 是 [@Huang-Wei](https://github.com/Huang-Wei) 和 [@ahg-g](https://github.com/ahg-g)，除此之外，几个核心的成员包括 [@alculquicondor](https://github.com/alculquicondor)，[@damemi](https://github.com/damemi)

2. 子项目：sig-scheduling 负责的项目除了k/k scheduler模块，还包括 [cluster-capacity](https://github.com/kubernetes-sigs/cluster-capacity), [descheduler](https://github.com/kubernetes-sigs/descheduler), [kube-batch](https://github.com/kubernetes-sigs/kube-batch), [kube-scheduler-simulator](https://github.com/kubernetes-sigs/kube-scheduler-simulator), [poseidon](https://github.com/kubernetes-sigs/poseidon), [scheduler-plugins](https://github.com/kubernetes-sigs/scheduler-plugins)

3. 例会：[Agenda](https://docs.google.com/document/d/13mwye7nvrmV11q9_Eg77z-1w3X7Q1GTbslpml4J7F3A/edit)，往期录制可点击[往期](https://www.youtube.com/watch?v=PweKj6SU7UA&list=PL69nYSiGNLP2vwzcCOhxrL3JVBc-eaJWI)

### 社区介绍
Slack：[#sig-scheduling](https://app.slack.com/client/T09NY5SBT/C09TP78DV)

Group: [kubernetes-sig-scheduling](https://groups.google.com/g/kubernetes-sig-scheduling)

KEP: [sig-scheduling](https://github.com/kubernetes/enhancements/tree/master/keps/sig-scheduling)

Website: [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)
