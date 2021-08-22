## <a id="chapter1">一. 序言</a>

本手册旨在帮助对开源贡献感兴趣的同学快速入门，并为之后的进阶之路提供一些参考和指导意义。如果你已经是 `kubernetes approver` 及以上<a href="#chapter2">角色</a>，本手册已超出该知识范畴，你可以推荐给其他感兴趣的同学阅读。

由于 Kubernetes 本身的复杂性及手册的篇幅和重点受限，本手册受众需满足以下要求：
1. 熟悉 Golang 语言
2. 了解 Kubernetes 基本组件及运行原理
3. 读过 kubernetes 源码是加分项

如果你未达到该要求，不建议直接进行 Kubernetes 开发，深入代码细节会让你迷失方向，就好比不携带地图就深入亚马逊丛林，很容易迷路。

如果你希望尽快掌握 Kubernetes 相关知识，可参考[官方文档](https://kubernetes.io/docs/concepts/)。

### Note: 本手册持续维护中！
</br>

## <a id="chapter2">二. 了解 Kubernetes 开源社区</a>
这一章节，你需要了解整个[ Kubernetes 社区](https://github.com/kubernetes/community)是如何治理的。

1. [分布式协作]()

    与公司内部集中式的项目开发模式不同，几乎所有的开源社区都是一个分布式的、松散的的组织，为此 Kubernetes 建立了一套完备的社区治理制度。

    协作上，社区大多数的讨论和交流主要围绕 `issue` 和 `PR` 展开。由于 `Kubernetes` 生态十分繁荣，因此所有对 Kubernetes 的修改都十分谨慎，每个提交的 `PR` 都需要通过两个以上成员的 `Review` 以及经过几千个单元测试、集成测试、端到端测试以及扩展性测试，所有这些举措共同保证了项目的稳定。

2. [SIG](https://github.com/kubernetes-sigs)

    SIG 的全称是 `Special Interest Group`，即特别兴趣小组，它们是 Kubernetes 社区中关注特定模块的永久组织，Kubernetes 作为一个拥有几十万行源代码的项目，单一的小组是无法了解其实现的全貌的.

    Kubernetes 目前包含 20 多个 SIG，它们分别负责了 Kubernetes 项目中的不同模块，这是我们参与 Kubernetes 社区时关注最多的小组；作为刚刚参与社区的开发者，我们可以从 sig/apps、sig/node 和 sig/scheduling 这几个 SIG 开始，它们的职责相对比较明确，从这几个 SIG 的工作入手可以较快的了解社区的工作流程。
3. [KEP](https://github.com/kubernetes/enhancements/tree/master/keps/)

    KEP 的全称是 `Kubernetes Enhancement Proposa`，因为 Kubernetes 目前已经是比较成熟的项目了，所有的变更都会影响下游的使用者，对于`功能`和 `API` 的`修改`都需要先在 kubernetes/enhancements 仓库对应 SIG 的目录下提交提案才能实施，所有的提案都必须经过讨论、通过社区 SIG Leader 的批准。

4. [WorkingGroup](https://docs.google.com/document/d/13mwye7nvrmV11q9_Eg77z-1w3X7Q1GTbslpml4J7F3A/edit)

    这是由社区贡献者自由组织的兴趣小组，对现阶段的一些方案和社区未来发展方向进行讨论，并且会周期性的举行会议。会议大家都可以参加，只是时间不是很友好，大多是在午夜时分。以`scheduling`为例，你可以查看谷歌文档[ Kubernetes Scheduling Interest Group ](https://docs.google.com/document/d/13mwye7nvrmV11q9_Eg77z-1w3X7Q1GTbslpml4J7F3A/edit#heading=h.ukbaidczvy3r)了解例次会议纪要。会议使用 `Zoom` 进行录制并且会上传到 `Youtube`, 过程中会有主持人主持，如果你是`新人`，可能会让你自我介绍😀.

4. [MemberShip](https://github.com/kubernetes/community/blob/master/community-membership.md)

    Kubernetes社区成员有如下四种角色：

        Member
        Reviewer
        Approver
        Subproject owner

    每种角色承担不同的职责，同时也拥有不同的权限。角色晋升主要参考你对社区的贡献，具体内容可点击段首`MemberShip`超链。

5. [Issue 分类](https://hackmd.io/O_gw_sXGRLC_F0cNr3Ev1Q)

    `Kubernetes Issues` 有很多 `label`。如 `bug`, `feature`, `sig` 等，有时候你需要对这些 `issue` 进行手动分类，关于如何分类可以[参考此文](https://hackmd.io/O_gw_sXGRLC_F0cNr3Ev1Q)


5. [其他关注项]()

    [Slack](https://slack.k8s.io/)

    [StackOverFlow](https://stackoverflow.com/questions/tagged/kubernetes)

    [Discussion](https://groups.google.com/g/kubernetes-dev)
</br>
</br>

## <a id="chapter3">三. 开始你的first-good-issue</a>

* 建议阅读[官方文档](https://www.kubernetes.dev/docs/guide/contributing/)。文档详细描述了该如何提交PR，以及应该遵循什么样的原则，并给到了一些最佳实践。

* 开始我们的PR之旅
    1. 申请 `CLA`

        当你提交 `PR` 时，Kubernetes 代码仓库CI流程会检查是否有CLA证书。

        如何申请证书可以参考[官方文档](https://github.com/kubernetes/community/blob/master/CLA.md)。

    2. 搜索 `first-good-issue`（你可以选择你感兴趣的或者你比较熟悉的 `SIG` ）

            first-good-issue 是 Kubernetes 社区为培养新参与社区贡献的开发人员而准备的 issue，比较容易上手。

        以 `sig/scheduling` 为例，在 `Issues` 中输入：

            is:issue is:open label:sig/scheduling label:"good first issue" no:assignee

        该 `Filters` 表示筛选出没有被关闭的，属于 `sig/scheduling`，没有 `assign` 给别人的 `good first issue`。

        如果没有相关的 `good-first-issue`，你也可以选择 `kind/documentation` 或者 `kind/cleanup` 类型 issue。

    3. 了解 `issue` 上下文，确定自己能够完成就通过 `/assign` 命令标记给自己。

        这里涉及到一些常见命令，如下。更多命令，见[Command-Help](https://prow.k8s.io/command-help?repo=kubernetes%2Fkubernetes)。

            /retitle                    重命名标题
            /close                      关闭issue
            /assign                     将issue assign给自己
            /sig scheduling             分类sig/scheduling
            /remove-sig scheduling      取消分类
            /help                       需要帮助，会打上标签help wanted
            /good-first-issue           打标签good first issue
            /retest                     重测出错的测试用例

    4. 编码
        1. Fork代码仓库。 将 [kubernetes/Kubernetes](https://github.com/kubernetes/kubernetes) `fork` 到自己的 `GitHub` 账号名下。

        2. Clone 自己的代码仓库

                git clone git@github.com:<your github id>/kubernetes.git

        3. 追踪源代码仓库代码变动
            1. 添加upstream

                    git remote add upstream https://github.com/kubernetes/kubernetes.git
            2. `git remote -v` 检查是否添加成功，成功则显示：

                    origin  git@github.com:<your github id>/kubernetes.git (fetch)
                    origin  git@github.com:<your github id>/kubernetes.git (push)
                    upstream        https://github.com/kubernetes/kubernetes.git (fetch)
                    upstream        https://github.com/kubernetes/kubernetes.git (push)
            3. 同步 `upstream kubernetes` 最新代码

                    git checkout master
                    git pull upstream master
                    git push


        4. 切分支，编码

                git checkout -b <branch name>

        5. Commit，并提交PR
            * 命令行

                    git commit -s -m '<change me>'
            * 注意：
                1. `commit push` 前先执行 `make update`
                2. 如果本次修改还没有完成，可以使用 `GitHub Draft` 模式，并添加 `[WIP]` 在标题中
                3. `Commit` 信息过多，且没有特别大的价值，建议合成一条 `commit` 信息

                        git rebase -i HEAD~2

        6. 提交PR
            1. 在 `GitHub` 页面按照模版提交 `PR`

    5. CI
        1. PR提交后需要执行 `Kubernetes CI` 流程，此时需要 `Kubernetes Member` 打 `/ok-to-test` 标签，然后会自动执行 `CI`，包括验证和各种测试。可以 `@` 社区大佬或者团队成员打标签。
        2. 一旦测试失败，修复后可以执行 `/retest` 重新执行失败的测试，此时，你已经可以自己打 `tag`

    6. Code Review
        1. 每次提交需要有2个 `Reviewer` 进行 `Code Review`， 如果通过，他们会打上 `/lgtm` 标签，表示 `looks good to me`, 代码审核完成。
        2. 另外需要一个 `Approver` 打上 `/approve` 标签，表示代码可以合入主干分支，`GitHub` 机器人会自动执行 `merge` 操作。
        3. `PR` 跟进没有想像中那么快，尤其是国外休假时间都比较长，`1-2周`也正常

    7. 恭喜，你完成了第一个 `PR` 的提交

## <a id="chapter4">四. 如何成为Kubernetes Member</a>

1. 官方文档要求对项目有多个贡献，并得到两个 `reviewer` 的同意。

        Sponsored by 2 reviewers and multiple contributions to the project

2. 贡献包括：

        1. PR
        2. Issue
        3. Kep
        4. Comment
        5. Review

3. 建议：
    1. 多参与社区的讨论，表达自己的观点
    2. 多参与 `issue` 的解答，帮助提问者解决问题，社区的意义也在于此
    3. 对源码了解不多，可以从一些简单的工作入手。如在看源代码的时候多留意一些语法、命名和重复代码问题，做一些重构相关的工作
    4. 从测试入手也是一个好办法，如补全测试，或者修复测试
    5. 参与一些代码的 `review`，同时可以学到一些知识。
    6. 最有价值的肯定是 `feature` 的实现，或者提出一些 `kep`

4. 关于如何获得 `reviewer` 同意，可以事先通过邮件打好招呼，把自己做的一些事情同步给他们，以及自己未来在开源社区的规划。

5. 成为 `Kubernetes Member` 不是目的，而是水到渠成的结果。`期待你的加入!`


## <a id="chapter5">五. 常用命令</a>
`参考 Makefile 文件`

* 单元测试（单方法）

        go test -v --timeout 30s k8s.io/kubectl/pkg/cmd/get -run ^TestGetSortedObjects$

* 集成测试（单方法）

        make test-integration WHAT=./vendor/k8s.io/kubectl/pkg/cmd/get GOFLAGS="-v" KUBE_TEST_ARGS="-run ^TestRuntimeSortLess$"

* E2E测试

    可以使用 `GitHub` 集成的 `E2E` 测试：

        /test pull-kubernetes-node-kubelet-serial
