# 不受欢迎的意见--数据科学家应该更注重端到端
_作者：**Eugene Yan**_  
_原文链接：<https://eugeneyan.com/writing/end-to-end-data-science/>_  
`data science` `machine learning` `productivity` `career`

---
最近，我看到一个Reddit帖子，是关于数据科学和机器学习的不同角色：
数据科学家，决策科学家，产品数据科学家，数据工程师，机器学习工程师，机器学习工具工程师，AI架构师等等。

我觉得这很令人担忧。
因为当一个完整的数据科学过程（问题框架、数据工程、机器学习、部署/维护）被分割在不同的角色身上时，它很难达到应有的效果。
会导致协调开销，责任分散，大局观缺乏。

个人浅见，我相信数据科学家可以通过端到端的方式来提高工作效率。
在这里，我将讨论这样做的好处和有争议的论点，以及如何做到端到端，同时分享 Stitch Fix 和 Netflix 的经验。

## 从开始（发现问题）到结束（解决问题）

你可能遇到过类似的标签和定义，比如:  
- **通才**：注重角色（产品经理、业务分析师、数据工程师、数据科学家、机器学习工程师）；还有一些负面含义的词。
- **全栈**：注重技术（Spark、Torch、Docker）；由全栈开发人员推广。
- **独角兽**：专注于神话故事；被认为不存在。

我不喜欢这些定义，它们太规范了。
相反，我有一个简单（和实用）的定义：
一个端到端的数据科学家可以**用数据来识别和解决问题，以发挥数据的价值**。
为了实现这个目标，他们会根据需要戴上尽可能多（或少）的帽子。
他们也会学习和应用任何有效的技术、方法和流程。
在整个过程中，他们会问一些问题，比如说：  
- 问题是什么？为什么它很重要？
- 我们能解决它吗？我们应该如何解决它？
- 估计的价值是多少？实际值是多少？

> ### 数据科学流程
>> 定义端到端数据科学的另一种方式是通过流程。
这些流程通常很复杂，我在主要的讨论中就不说了。
尽管如此，这里有几个例子，以防你好奇。
>> - CRISP-DM： 跨行业数据挖掘标准流程（1997）。
>> - KDD：数据库中的知识发现。
>> - TDSP：团队数据科学流程，由微软在2018年提出。
>> - DSLP：数据科学生命周期流程。  
>>
>> 如果这些流程看起来很沉重，让人不知所措，不要担心。你不必一下子全盘吸收它们--从一点一滴开始，保留有效的东西，然后再调整其余的。

# 更多的背景内容，更快的迭代速度，更高的满意度
对于大多数数据科学角色来说，端到端的训练会提高你做出有意义的影响的能力。（不过，也有些角色需要专注于机器学习。）

**端到端工作提供了更多的背景内容。**
尽管为整个流程中每个节点设置专有的角色可以提高效率，但对于数据科学家来说，这会减少背景的了解，可能导致最终的解决方案并不是最优的。

> _忘掉全局的诀窍是近距离观察一切。-- Chuck Palahniuk_

在没有充分了解上游问题的背景下，很难设计出一个整体的解决方案。
比方说，转化率下降了，一个PM提出了改进我们搜索算法的需求。
然而，首先是什么原因导致了转化率的下降？可能有多种原因：  
- 产品：假冒/劣质产品是否降低了客户的信任度？
- 数据管道：数据质量是否受到影响，或者是否有延迟/中断？
- 模型更新：模型是否没有定期/正确地更新？

更多的时候，问题--以及解决方案--都与机器学习本身无关。
改进算法的解决方案会错过问题的根本原因。

同样，在没有意识到下游工程和产品约束的情况下开发解决方案也是很冒险的。
这些都是没有意义的：  
- 构建一个近乎实时的推荐器，然而基础设施和工程师不能支持；
- 构建一个无限滚动的推荐器，然而它并不适合用于我们的产品和应用中。

通过端到端工作，数据科学家将拥有完整的背景，以确定正确的问题并开发可用的解决方案。
它还可以带来创新的想法，而专家们由于其狭窄的背景可能会错过这些 idea。
总的来说，端到端提高了提供价值的能力。

**减少沟通和协调的开销**。
有了多个角色就会有额外的开销。
我们来看一个例子，一个数据工程师（DE）清理数据并创建特征，一个数据科学家（DS）分析数据并训练模型，一个机器学习工程师（MLE）进行部署和维护。

>_一个程序员一个月能做的事，两个程序员两个月就能完成。-- Frederick P. Brooks_

DE 和 DS 需要讨论，来决定哪些数据是（或不是）可用的，如何清理数据（例如，离群值、正常化），以及应该创建哪些特征。
同样，DS 和 MLE 也要讨论如何部署、监控和维护模型，以及应该多久刷新一次。
当问题发生时，我们需要三个人在房间里（可能还有一个 PM）对问题发生的原因进行分类，看看是那个流程出现了错误，然后再安排下一步的修复措施。

这也会导致额外的协调，在执行任务时，各个成员需要按照工作流程顺序来调整时间表。
如果 DS 想要试验更多的数据和功能，我们需要等待 DE 摄取数据并创建特征。
如果一个新的模型已经准备好进行 A/B 测试，我们需要等待 MLE 来（将其转换为生产代码）才能部署它。

虽然实际的开发工作可能需要几天的时间，但来回的沟通和协调可能需要几周甚至更长的时间。
有了端到端的数据科学家，我们可以将这种开销降到最低，也可以防止技术细节在传递过程中丢失。

（但是，一个端到端 DS 真的能做到这些吗？我想是的。
虽然 DS 可能在某些任务上不如 DE 或 MLE 那么熟练，但他们将能够有效地执行大多数任务。
如果他们在扩展或持久化方面需要帮助，他们可以随时从专业的 DE 和 MLE 那里获得帮助。）

> ### 沟通和协调的成本
>> 哈佛大学心理学家 Richard Hackman 表明，如果一个团队中的关系数量为 N(N-1)/2，其中 N 为人数。这就导致了链接的指数级增长，其中:  
>> - 一个 7 人的创业团队有 21 个环节需要维持；
>> - 一个 21 人的小组（即 3 个创业团队）有 210 个环节；
>> - 一个 63 人的小组有近 2000 个链接；
>> 在我们的简单例子中，我们只有三个角色（即六个环节）。但是，随着 PM、BA 和其他成员的加入，会导致沟通和协调成本的增长大于线性增长。
因此，虽然每增加一个成员都会提高团队的总生产力，但开销的增加意味着生产力的增长速度在下降。
亚马逊的‘[two-pizza teams](https://buffer.com/resources/small-teams-why-startups-often-win-against-google-and-facebook-the-science-behind-why-smaller-teams-get-more-done/)’（团队成员刚好能吃完俩个比萨）是一个可能的解决方案。

**迭代和学习率提高**。
有了更多的背景内容和更少的团队开销，我们现在可以更快地迭代项目、修正错误、并完成交付。

这对于开发数据和算法产品尤为重要。
与软件工程（相对成熟得多的工艺）不同，我们无法在开始构建之前完成所有的学习和设计--我们的蓝图、架构和设计模式都没有那么成熟。
因此，快速迭代对于设计-建设-学习的周期是必不可少的。

**有更大的所有权和问责制**。
让数据科学过程分裂在多人身上，可能会导致责任的分散，更糟糕的是，会使整个团队变得闲散懒惰。

在实际中经常可以观察到的一种错误工作模式， "扔过墙"。
例如，DE 创建特征并将数据库表扔给 DS，DS 训练模型并将 R 代码扔给 MLE，MLE 将其翻译成 Java 语言到生产环境。

如果在信息在传递过程中发生损失，或者结果出乎意料，谁来负责？
有了强大的主人翁文化，每个人都会站出来在各自的角色上做出贡献。
但是，如果没有这种文化，工作就会沦落为掩饰和指责，而问题却一直存在，客户和业务也会受到影响。

让端到端的数据科学家对整个流程承担所有权和责任，可以缓解这种情况。
他们应该被授权从头到尾采取行动，贯穿整个客户问题的输入端（即原始数据）到可量化结果的输出端（即部署的模型）。

> ### 责任的分散与团队闲散
>> 责任的分散：当有其他人在场时，我们不太可能承担责任和采取行动。
如果我们知道有其他人也在关注这一情况，那么个人的责任感就会降低。
>>
>> 旁观者效应就是其中的一种形式，Kitty Genovese 在她居住的街对面的公寓楼外被刺伤。
虽然当时有 38 名目击者看到或听到了袭击，但没有一个人报警或帮助她。
>>
>> 团队闲散：我们在团队中工作时比单独工作时付出的努力要少。
在 19 世纪 90 年代，Ringelmann 让人们单独或分组拉绳索，
他测量了他们拉绳的力度，发现小组成员在拉绳时往往比个人单独拉绳耗费的精力要少。

**对于（一些）数据科学家来说，这可以提高他们的积极性和工作满意度**，这与自主性、掌控性和目的性密切相关。
- **自主性**：能够独立解决问题。
端到端的数据科学家不需要等待或依赖他人，而是能够发现和定义问题，建立自己的数据管道，并部署和验证解决方案。
- **掌控性**：在从端到端过程中遇到的问题、解决方案、结果时，
他们可以根据需求选择合适的方案和技术。
- **目的性**：通过深度参与整个过程，他们与工作和成果有更直接的联系，从而增强目标感。

## 但是，我们也需要专家
端到端并不适合每个人（或每个团队），原因有：

**想要专攻机器学习**，或者想专门研究机器学习的特定领域，例如神经文本生成（[GPT-3入门](https://mc.ai/the-subtle-art-of-priming-gpt-3/)）。
虽然端到端很有价值，但我们也需要世界一流的研究人员和行业专家，来推动这一领域的发展。
我们在 ML 领域的很多东西都来自学术界和纯粹的科研工作。

> _没有人可以通过成为通才来获得伟大的成就。
你不关注技能的发展就没法磨练提升你的技能。
唯一能让你达到下一个层次的方法就是专注。-- John C. Maxwell_

**缺乏兴趣**。
不是每个人都热衷于与客户和业务接触，以及定义问题、收集需求和编写设计文档。
同样，不是每个人都对软件工程、生产代码、单元测试和 CI/CD 管道感兴趣。

**在大型高杠杆系统上工作，0.01% 的改进会产生巨大影响**。
例如，算法交易和广告。
在这样的情况下，需要超专业化的工作来勉强实现这些改进。

其他人也提出了为什么数据科学家应该专业化（而不是端到端）的论点。这里有几篇文章可以参考：
- [为什么你不应该成为一个数据科学通才？](https://towardsdatascience.com/why-you-shouldnt-be-a-data-science-generalist-f69ea37cdd2c)
- [为什么每个数据科学家都需要专业化？](https://www.simplilearn.com/why-every-data-scientist-needs-to-specialize-article)
- [想找一份数据科学方面的工作？你为什么要专攻数据科学？](https://www.dataquest.io/blog/job-data-science-specialize/)

## 最好的学习方法是工作来学习
如果你还热衷于成为更多的端到端，我们现在就来讨论如何做到这一点。
在此之前，不谈具体的技术，以下是端到端数据科学家常用的技能：
- 产品：了解客户的问题，定义和优先考虑需求
- 沟通：促进各团队之间的交流，获得认同，编写文件，分享成果
- 数据工程：将数据从 A 形态移动/转换到 B 形态
- 数据分析：理解和可视化数据，A/B 测试和推理
- 机器学习：通常加上实验、实施和验证标准
- 软件工程：生产代码实践，包括单元测试、文档、日志
- 开发运营：基本的容器化和云计算能力，构建和自动化相关工具

（这个清单既不是强制性的，也不是详尽无遗的。大多数项目并不需要用到所有的技能）

这里有四种方法可以让你更容易成为一个端到端的数据科学家。

**学习正确的书籍和课程**。（好吧，这不是边做边学，但我们都需要从某个地方开始）。
我会关注那些涵盖隐性知识而非具体工具的课程。
虽然我没有遇到过这样的教材，但我听说《[全栈深度学习](https://course.fullstackdeeplearning.com/)》这个课程不错。

**动手做自己的端到端项目**，获得整个过程的第一手经验。
你可能面临过度简单化的风险，以下是我会采取的一些步骤及其相关技能。

>_不闻不若闻之，闻之不若见之，见之不若知之，知之不若行之。-- ~~孔子~~ 荀子_
>> 原文写的是这句话来自孔子，网上查原句的时候发现这句话并不是孔子说的，是孔子的弟子荀子在《儒孝篇》中的名句，翻译后海外误传至今。
有人说可能老外只知道孔子这一个人，所以孟子老子荀子什么的都推到他身上了。

首先确定一个要解决的问题，确定成功的指标（产品）。
然后，找到一些原始数据（不要用 Kaggle 竞赛的数据）；
这可以让你清理和准备数据并创建特征（数据工程）。
接下来，尝试各种 ML 模型，检查学习曲线、错误分布和评估指标（数据科学）。

在选择一个模型并围绕它编写一个基本的推理类用于生产之前，评估每个模型的性能（例如，查询延迟、内存占用）（软件工程）。
你可能还想建立一个简单的用户界面。然后，将其容器化，并通过你喜欢的云提供商在线部署给其他人使用（开发运营）。

一旦完成了这些工作，就可以额外地去分享你的工作。
你可以为你的网站写一篇文章，或者在见面会上讲一讲（交流）。
通过有意义的可视化和表格来展示你在数据中发现的东西（数据分析）。
在GitHub上分享你的工作。
在公共场合学习和工作是获得反馈和寻找潜在合作者的好方法。

**像 [DataKind](https://www.datakind.org/) 这样的团体一样做志愿者**。
DataKind与社会组织（如非政府组织）和数据专业人士合作，解决人道主义问题。
通过与这些非政府组织合作，你有机会作为团队的一部分，用真实（混乱）的数据解决实际问题。

虽然志愿者们在工作时可能会被分配到特定的角色上（例如，PM、DS），但其他人随时欢迎你加入并观察他们的工作。
你将看到（并学习）PM 如何与 NGO 合作，以确定问题的框架，定义解决方案，并围绕它组织团队。
你将从其他志愿者那里学习到如何利用数据来制定可行的解决方案。
在类似于黑客行动的 DataDives 和长期的 DataCorps 中做志愿者是对数据科学过程进行端到端学习的好方法。

**加入一个类似初创企业的团队**。
注意：类似创业公司的团队并不是创业公司的代名词。
有一些大的组织以类似初创公司的方式管理团队（例如，两个披萨团队），也有由专家组成的初创公司。
找一个精益的团队，在那里你会受到鼓励，并且有机会进行端到端工作。

## Stitch Fix 和 Netflix 中端到端的案例
Stitch Fix 公司的 Eric Colson 最初的想法是“通过功能的分工，提高流程效率”（即数据科学流程工厂）。
但经过试错，他发现端到端的数据科学过程更加有效。
现在，Stitch Fix 组织数据团队不是为了专业化和生产力，而是为了**学习和开发新的数据和算法产品**。

> _数据科学的目标不是执行。
相反，它目标是学习和开发新的业务能力。
...没有蓝图，这些都是具有内在不确定性的新能力。
...你所需要的所有元素都必须通过实验、试错和迭代来学习。-- Eric Colson_

他建议，数据科学的角色应该变得更加通用，承担与技术功能无关的广泛职责，并针对学习过程进行优化。
因此，他的团队专门雇佣和培养能够概念化、建模、实施和测量的通才。
当然，这有赖于一个坚实的数据平台，这个平台可以抽象出基础设施设置、分布式处理、监控、自动故障转移等复杂问题。

拥有端到端的数据科学家提高了 Stitch Fix 的学习和创新能力，使他们能够发现和建立更多的业务能力（相对于专家团队）。

Netflix Edge Engineering 项目最初为各个流程配有专门的角色。
然而，这造成了整个产品生命周期的低效率。
代码发布需要更多的时间（几周而不是几天），部署问题需要更长的时间来检测和解决，生产问题需要多次来回沟通。

<div align=center><img src="https://s1.ax1x.com/2020/09/16/wg2MwT.jpg"></div>
<div align=center><h6>在极端情况下，每个功能区/产品由7个人组成</h6></div>

为了解决这个问题，Netflix 尝试了全周期开发人员，他们被授权在整个软件生命周期内工作。
这就需要转变思维方式--开发人员不仅要考虑设计和开发，还要考虑部署和可靠性。

<div align=center><img src="https://s1.ax1x.com/2020/09/16/wg2QTU.jpg" width='600'></div>
<div align=center><h6>不是多个角色和人员，我们现在有了全周期的开发人员</h6></div>

为了支持全周期开发，中心团队建立了各种工具，以自动化和简化常见的开发流程（例如，构建和部署管道、监控、管理回滚）。
这样的工具可以在多个团队之间重用，起到了力量倍增的作用，并帮助开发人员在整个周期内有效地进行开发。

通过全周期的开发人员方法，Edge Engineering 能够更快地进行迭代，并进行更快、更常规的部署。

## 它对我有用吗？这里有几个例子
在 IBM，我是所在的团队主要任务是为在职者提供工作建议。
运行整个流水线需要很长时间。
我想，我们可以把数据准备和功能工程管道移到数据库中，从而把时间缩短一半。
但是，数据库的家伙没有时间来测试这个。
由于没有耐心，我跑了一些基准，将整体运行时间减少了90%。
这让我们在生产中的实验速度提高了10倍，并节省了计算成本。

在构建 Lazada 的排名系统时，我发现 Spark 对于数据管道是必要的（由于数据量大）。
然而，我们的集群只支持我不熟悉的 Scala API。
由于不想等待（数据工程师的支持），我选择了更快，但也很痛苦的途径，即自己摸索 Scala Spark 并编写管道。
这使我之后的开发时间减少了一半，并让我对数据有了更好的理解，从而建立更好的模型。

在 A/B 测试成功之后，我们发现业务利益相关者并不信任这个模型。
结果，他们手动挑选了产品进行首页展示，这导致当期的在线指标（如 CTR、转化率）降低了。
为了了解更多客户的想法，我到我们的市场平台（如印尼、越南）进行了考察。
通过相互教育，我们解决了他们的顾虑，减少人工操作的覆盖量，并最终获得收益。

在上面的例子中，**脱离常规的 DS 和 ML 工作范围，有助于我们更快地交付更多价值**。
在最后一个例子中，某些情况下，我们有必要解除数据科学家的身份去工作。

## 现在就试试吧
你现在可能无法做到端到端。这没关系--很少有人能做到。
尽管如此，想想这么做的好处，并慢慢向这方面靠近。

哪些方面会不成比例地提高你作为数据科学家的交付能力？
增加与客户和利益相关者的接触，以设计更全面的创新解决方案？
建立和协调自己的数据管道？
提高对工程和产品限制的认识，以加快集成和部署？

选择一个并尝试一下。
在你更擅长它之后，再尝试其他的东西。

---
[返回目录](https://github.com/datugou/Article_Translation/tree/master/LEARNING_data_science)
