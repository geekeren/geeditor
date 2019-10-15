**极客MD**是一款专为**邮件**打造的**Markdown编辑器**，UI简约，功能强大，是居家、工作必备的邮件排版神器！
目前支持用Markdown语法编辑**标题**、**段落**、**列表**、**图片**、**待办事项**、**代码块（高亮）** 等所有主流Markdown编辑器的语法。

编辑完成后，点击右上角的**复制**按钮即可将**右侧预览区**的内容连带样式**无损**地粘贴到邮件。

>妈妈再也不用担心我的邮件排版了!

- 开源地址：https://github.com/geekeren/geekmd


![加我微信，关注"极客MD"](https://wangbaiyuan.cn/wp-content/uploads/2018/08/qrcode_for_gh_5f4c204be810_344.jpg) | ![加我微信，关注"极客MD"](https://wangbaiyuan.cn/wp-content/uploads/image/wechat_zanshang.jpeg)
:--: | :--:
加我微信，关注"极客MD" | 微信打赏

-----------

# 下面是示例文档：



>本文翻译自：[Freecodecamp](https://medium.freecodecamp.org/amazon-fargate-goodbye-infrastructure-3b66c7e3e413), 原文地址：[An intro to Amazon Fargate: what it is, why it’s awesome (and not), and when to use it](https://medium.freecodecamp.org/amazon-fargate-goodbye-infrastructure-3b66c7e3e413), 英文原作者为 [Emmanuel Marboeuf](https://medium.freecodecamp.org/@emmanuelboumeraf)

当亚马逊在2017年底的AWS re:Invent大会上和EKS一起[宣布](https://www.youtube.com/watch?v=8i82i9QYUGs) Fargate的时候，它还备受冷落，当时我所关注的博客和大佬只是轻描淡写地说：

> 哦，有这么个新玩意，它将允许ECS用户直接在云中运行容器。

作为开发人员，这真的让我大吃一惊。**让我们看看为什么。**


## 解放生产力

我觉得软件开发领域已有**五次**重大革命，大大提高了开发者的工作效率，并以最高效率编写与部署应用。他们都解决了一系列的重大问题：

*   **云服务（IaaS）的出现**：解决了基础架构的成本和可扩展性问题
*   **开源社区，会议，工作坊，技术博客，StackOverflow**等：让知识惠及到更多人
*   **版本控制系统，协作工具，持续集成工具** 解决了项目的并行开发和集成问题
*   **容器化架构**
*   **无服务器计算服务（PaaS**） 降低服务器和系统管理成本

这些革命中的都有一个共同特点：它们都**为软件工程师赋予了更多的控制权**，降低了对服务器的系统管理员，DevOps，IT支持人员等的依赖。

所以，Fargate算其中的哪一种呢？

## 你的“船”才是问题所在
![Life jacket is advised](https://wangbaiyuan.cn/wp-content/uploads/2018/08/20180823175209119.jpg)

Docker使容器技术普及开来，便很快成为开发中被广泛采用的新标准。

不久之后，随着**Kubernetes**的成功，AWS推出了自己的（更基本的）容器管理服务：Amazon Elastic Container Service（ECS），它引入了任务（Task）的概念。

一个任务可以是一起工作的容器的任何实例。它可以是从运行一个服务的Web应用，多个微服务，数据库和反向代理，还可以是定期运行的批处理shell脚本。

作为使用ECS的“老司机”，我很喜欢它，曾经在一段时间内用得很爽，但到最后，我发现在管理EC2 Instance的同时，还必须管理一些额外的层（任务和容器），这让ECS变得越来越复杂。（译者注，请参考[AWS“层”的概念](https://wangbaiyuan.cn/wp-content/uploads/2018/08/20180823175211214.jpg)）

同时，我对ECS的安全性也不甚满意，层越多，就越要保持警惕，而每一层都带来了更多的复杂性，以及误配置、漏洞的可能性。

使用ECS在集群中运行中的EC2 实例（Instance）配置弹性伸缩时，每个实例上可以承载多个不同的Task，每个Task都可以运行多个容器。
由于任务（Task）会在具有可用资源的相同类型的EC2实例上随机部署（默认情况下），将面临以下问题：

*   一个集群必须遵循相同的弹性伸缩规则，伸缩的EC2实例类型必须相同。
*   有些容器需要完全不同的资源（译者注，AWS将网络、虚拟机、负载均衡都称为资源），但仍需要协同工作。
*   某些容器不一定遵循相同的弹性伸缩规则。
*   有时同一任务中的多个容器都需要有自己的负载均衡，无法为同一任务设置多个负载均衡器。

面对这些问题时，首选的解决方法是：

*   根据需求手动部署一些具有不同资源的EC2实例
*   将这些EC2实例添加到您的集群
*   为任务运行一个容器
*   手动链接到您的EC2实例
*   在ECS上配置复杂的任务策略，把不用任务根据他它们的需求配置在适当的EC2上

这需要大量、繁琐的工作，还不易维护，这违背了使用容器的初衷。

有人不得不提出一个更好的主意。

## 让它们“漂”起来

事实证明，AWS团队在过去一年中也遇到了同样的问题，他们考虑过并致力于解决这些问题：

> 我们怎样才能运行容器而不必担心服务器和集群？

**这就是AWS Fargate的意义所在**。它完全抽象了底层基础架构，你可以把每个容器视为一台机器。

你只需定义每个容器所需的资源，它将为你完成复杂的任务而不必再管理多个“层”之间的访问规则，您可以像在单个EC2实例之间那样调度容器那样。

![Fargate上的容器](https://wangbaiyuan.cn/wp-content/uploads/2018/08/20180823175211214.jpg)

这就让你的容器像水面上的船只拥有自己的帆、船舵、船员一样，自己可以漂流到想要去的地方。

## 容器即服务（CaaS）

我一直坚信**容器即服务**（CaaS）是开发人员梦寐以求的、真正的平台及服务（PaaS）。它允许开发人员直接在云中部署他们的容器，而不必担心底层细节。

当然，已经有很多技术允许您在云上无缝运行代码，而不必担心扩展或服务器管理（比如**Heroku**，**Lambda，**甚至以自己的方式使用**Google应用引擎）**，但它们都有局限性。

*   必须选择丧失一点灵活性
*   必须使用它们所支持的语言
*   无法使用它们所支持的语言完成拟定项目，因为可能您的项目需要在指定操作系统系统上可用的低版本的原生库
*   您的项目采用了尖端技术，未来几年将无法为大众所用
*   其中一些平台非常非常昂贵，特别是在扩展时

**Fargate(或CaaS)** 为您带来两全其美的方案。

**容器化架构**为您提供所需的灵活性和控制权，它允许您在**任何**类型的系统中使用**任何**技术。而容器技术能确保你的应用在每个主机上具有相同的行为，无论是开发（Dev），测试(Testing/QA)，预发布（Staging）还是生产(Production)环境。

我觉得这一点对众多的科技初创公司至关重要。事实上，有时您的竞争优势之一就是，您参与开发出最先进的技术，或者是把机智地原有的技术用在新的应用场景中。

**无服务器部署**使您可以专注于编写优秀的代码，没有配置，易于扩展。

## 限制

### CaaS vs PaaS

也许确实你放弃了真正的“平台即服务”的一些优势，比如您仍然需要**手动更新**容器的镜像，有时您必须编写自己的Docker镜像。如果您不了解**系统管理**的基础知识，这的确很麻烦。

但是，这也意味着您可以放手地做任何您能想做的事情，并且在您想要使用的系统上，保证语言，工具，库和版本中的**灵活性和自由度**。

## 成本

让我们直面这样的现实：云服务（IaaS）比拥有自己的基础架构**更昂贵**（如果你想要按需扩展你的基础架构）。同样的道理，不必关心配置、管理和扩展服务器是要需要付出代价。如果你的应用十分简单，云服务并不是最优解。

Fargate实好，但是[其价格](https://aws.amazon.com/fargate/pricing/)几乎是等配置EC2实例的价格的 4 倍（例如t2.medium）这让我们很难抉择，让我们期待它能够降价。

![Fargate 和 EC2 价格对比（USD）](https://wangbaiyuan.cn/wp-content/uploads/2018/08/20180823175211315.jpg)



## 我应该将所有ECS任务都切换到Fargate吗？

当然不是。如上所述，在某些情况下，您的成本将增加三倍以上，所以在它们降低成本之前，您可能最好使用标准EC2实例。

但是，在以下情况下，Fargate可能对您更有益：

*   如果您无法很好地伸缩你的ECS任务，会导致大量空闲的**CPU或内存**，而使用Fargate，您**只需为在任务中定义的资源付费**。
*   对于**按需或按定时计划**运行的任务，并不需要一直运行着的EC2实例。使用Fargate，您**只需在任务运行时付款。**
*   适用于有**内存和/或CPU使用率峰值**的任务。这仅仅是因为使用Fargate可以在这类情况下节省您配置和管理的时间。

## 彩蛋

对于那些喜欢**Kubernetes**而非**ECS的人来说**，Fargate很快就会支持[EKS](https://aws.amazon.com/eks/)了。

----


# 效果预览

![编辑邮件，粘贴到邮箱效果预览](https://raw.githubusercontent.com/geekeren/geekmd/master/docs/assets/preview.png)

![QQ邮箱收到邮件的效果预览](https://raw.githubusercontent.com/geekeren/geekmd/master/docs/assets/preview2.jpeg)

![Mac Mail收到邮件的效果预览](https://raw.githubusercontent.com/geekeren/geekmd/master/docs/assets/preview3.jpeg)
