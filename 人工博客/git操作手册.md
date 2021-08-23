### git操作手册

### 1、命令大全

![git命令大全](https://s1.ax1x.com/2020/10/12/027NMd.jpg)

### 2、附加的命令说明

| 命令                                  | 说明                   |
| ------------------------------------- | ---------------------- |
| git checkout . && git clean -xdf      | 回归合并代码的请求     |
| git branch -D branchName              | 强制删除本地的代码分支 |
| git checkout -b 本地分支名 远程分支名 | 检出代码到本地         |
| git remote update origin -p           | 强制更新远程仓库       |

### 3、主流分支管控模式

- TrunkBased
- GitFlow
- AoneFlow

#### 3.1、TrunkBased

TrunkBased 模式 是持续集成思想所崇尚的工作方式，它由单个主干分支和许多发布分支组成，每个发布分支在特定版本的提交点上从主干创建出来，用来进行上线部署和 Hotfix。

在 TrunkBased 模式中，没有显性的特性分支。当然实际上 Git 的分布式特征天生允许每个人有本地分支，TrunkBased 也并非排斥短期的特性分支存在，只不过在说这种模式的时候，大家通常都不会明确强调罢了。

**优点**：使团队更加灵活。可以随时发布主分支，合并冲突少。

**缺点**：太多的团队同时工作在主干上，到发布的时候就可能出现灾难（尤其是多版本并行开发的情况）。

**弥补措施**： FeatureToggle 以及频繁的集成和足够的测试覆盖，这对开发团队的能力提出了比较高的要求。

**适用项目**：TrunkBased 模式适用在不需要同时维护多个历史版本的 SAAS 型项目，经过微服务改造的各种小型服务上。

![TrunkBased](https://s1.ax1x.com/2020/10/12/02x2y8.png)


#### 3.2、GitFlow

GitFlow 模式 是若干模式的集大成者，包含一个主干分支、一个开发分支、许多的特性分支、许多的发布分支和 Hotfix 分支，以及许多繁琐的合并规则。

**优点**：每个阶段的每项操作定义十分明确，重视流程。

**缺点**：大量的合并冲突；功能分离对集成测试不友好，不可预测的发布（如果功能分支尚未合并，则不可能知道需要多少时间才能发布。你不知道它是因为你不知道哪个合并会发生冲突，你也不知道新功能将如何相互影响）。

![GitFlow](https://s1.ax1x.com/2020/10/12/02xoYn.png)




#### 3.3、 AoneFlow

AoneFlow 模式 是阿里巴巴的分支管理模式。

在AoneFlow 上能看到许多其他分支模式的影子。它基本上兼顾了 TrunkBased 的“易于持续集成”和 GitFlow 的“易于管理需求”特点，同时规避掉 GitFlow 的那些繁文缛节。

AoneFlow 的三种分支类型：主干分支、特性分支、发布分支，以及三条基本规则。

**规则一**，开始工作前，从主干创建特性分支。
![创建特性分支](https://s1.ax1x.com/2020/10/12/02xcSP.png)

**规则二**，通过合并特性分支，形成发布分支。
![合并分支](https://s1.ax1x.com/2020/10/12/02xgQf.png)

**规则三**，发布到线上正式环境后，合并相应的发布分支到主干，在主干添加标签，同时删除该发布分支关联的特性分支。
![合并到主干](https://s1.ax1x.com/2020/10/12/02xROS.png)

#### 3.4、 写在最后



**代码分支模式的选择并没有绝对的正确和错误之分，关键是与项目的规模和发布节奏相匹配。最合适的，才是最好的！**

**参考文章**：

GitFlow有害论：http://insights.thoughtworkers.org/gitflow-consider-harmful

在阿里，我们怎么管控分支？：https://yq.aliyun.com/articles/573549

我们为什么从GitFlow切换到Trunkbased：https://blog.csdn.net/wangqingjiewa/article/details/78519686

Feature Flag功能发布控制：http://fex.baidu.com/blog/2014/07/feature-flag


