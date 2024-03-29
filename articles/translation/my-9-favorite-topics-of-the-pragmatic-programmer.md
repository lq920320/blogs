## 渣翻：《The Pragmatic Programmer》中我最喜欢的9个主题



> 原文地址： https://felixgerschau.com/pragmatic-programmer-20th-anniversary-favorite-topic-summary



*译者注：《The Pragmatic Programmer》即为《程序员修炼之道：从小工到专家》，直译为《务实的程序员》。*



《程序员修炼之道》第一版于 1999 年出版，已成为软件开发人员最重要的读物之一。

去年，在本书首次发布 20 年后，作者发布了本书的新版本，不仅包括代码示例的更新，还整合了他们此时收集的反馈。

虽然本书包含代码示例，但它并不是教你如何编码：作者解释了如何以实用的方式改进软件开发过程。 它包含大量提示，可在您的整个编程生涯中为您提供帮助。

务实程序员的一些特征是他们处理问题的态度和哲学，将问题置于更大的背景中，并对他们所做的一切负责。

我会向所有认真提高技能的软件开发人员推荐它。 然而，虽然它包含了很多有用的建议，但我觉得如果你刚刚开始编程，你可能不会欣赏这本书提供的所有智慧。

### 我最喜欢的主题

*Pragmatic Programmer* 共有 53 个主题。 与任何非小说类书籍一样，有些想法会引起读者的共鸣，而另一些则不会。

我想在这里分享我最喜欢的每个主题的简短摘要，以及我如何将它们整合到我的工作中。 这些是我第一次读这本书后最喜欢的主题，但我敢肯定，如果我再过几年再读这本书，我的收获会有所不同。

- 这是**你**的人生

- 知识组合

- 正交性

- 曳光弹

- 基础工具

- 死掉的程序不会说谎

- 巧合式编程

- 测试你的代码

- 务实的入门套件

### 这是你的人生

这个话题出现在本书的开头，并具有激励人心的氛围。 与 Dave 和 Andi 交谈过的许多开发人员都对他们的工作或正在使用的技术感到沮丧。

作者强调了这样一个事实，即软件开发是当今最好的职业之一，因为开发人员通常收入很高，可以在世界任何地方工作，并且在工作中拥有很大的自由度。

如果你对现在的工作不满意，可以尝试通过与老板交谈来解决它，或者轻松地为自己找到另一份工作。

考虑到软件开发人员的特权，没有理由不过上自己想要的生活。 如果你积极主动并保持领先地位，那么将会获得该行业的大量机会。

这是你的人生，你是可以做主的。

### 知识组合

每个开发人员的知识组合就像一个常规的投资组合：如果你定期投资，那么将在以后获得收益。

我发现这个类比是本书中最有力的类比，因为我是在我没有进一步提高我的编程技能并且感觉自己正在触及玻璃天花板的时候阅读它的。 我没有投资于我的知识组合，并且我想到了它（没投资）的结果。

通过阅读非小说类书籍定期投资于自己将帮助你保持对自身职业的好奇心和动力（它们不必全部与编程相关）。

它还有助于发现更多你不知道的东西，了解未知的未知，已知的未知。 这个想法不是来自书本，但我认为它是相关的。

对某个主题了解得越多，就越能看到其中的复杂性，而对它的了解就越少。

作者建议你可以通过以下方式投资您的投资组合：

- 每年至少学习一门新语言

- 每个月读一本技术书籍

- 也要阅读非技术书籍

- 上课

- 参加本地组活动和聚会

- 保持最新（通过在线阅读新闻和帖子）

你不必全部都做，但根据这些建议不断发展自己肯定会帮助扩展你的投资组合。

### 正交性

这是从几何学中借来的术语。 我们可能以**模块化**或**分层系统**等名称了解这个概念。 最后，它们都是指组件松散耦合的系统，这意味着组件没有很多依赖关系。

在正交系统中，很容易更改一个组件，而不必担心会破坏另一端的某些东西。 这些系统也更容易测试。

在系统的开发或设计阶段考虑正交性有助于使系统在未来更易于使用并防止错误。

*即松耦合性。*

### 曳光弹

在动作片的拍摄场景中，都可以经常看到子弹的轨迹。 标记它们所走路径的子弹称为曳光弹。

曳光弹也用于现实世界的战斗，因为它们通过**在真实情况下提供即时反馈**来帮助在困难环境下瞄准。 他们不会命中目标，但快速反馈可以让炮手调整以指向目标。

软件开发中的等价物则是**让系统的一小部分尽快工作**并征求用户和利益相关者的反馈。

曳光弹方法的优势在于：

- 为用户和赞助商提供早期和可见的结果。

- 开发人员有一个可以构建的架构基础。

- 该项目成为一个**集成平台**：新的变化会被集成到一个已经工作的系统中。

“曳光弹方法”的替代方案是分别编写系统的每个模块，最终将它们连接在一起，并在项目的最后阶段测试整个系统。

曳光弹并不意味着是原型。 它们旨在获得最终系统功能或方面的稳定版本，这将成为未来发展的支柱。

就像子弹一样，你不会总是在第一枪就击中目标。 用户或利益相关者可能不喜欢这个结果，但整个想法是在项目开始时而不是在结束时获得反馈。

### 基础工具

本书的这一章概述了每个程序员都应该使用的工具。 作者将编程工具与木工的工具进行了比较，这也启发了本书的封面。

木工使用各种工具，每种工具都有其特定用途。 他们使用尺子、钻头、夹子等工具。 木工知道何时使用哪些工具，并且已经掌握了如何使用它。

在编程中，我们也有自己的工具来完成工作。 这些工具通过提高生产力来扩大我们的才能。

在书中，他们建议不要使用单一的效率工具、 IDE，并鼓励使用 IDE 背后使用的工具。

本书涵盖的工具分为以下几类：

- 纯文本的力量

- Shell 游戏（Unix 命令）

- 版本空值

- 强力编辑（你的文本编辑器）

- 调试

- 工程手册

**不要依赖 GUI 环境**

Unix 命令之所以如此强大，是因为它们每个都有一个单一的目的，并且它们可以通过管道组合在一起，这为你可以用它们做什么提供了无限的可能性。命令终端对程序员就像工作台对木工来说一样。

使用 shell 可以让你以 GUI 无法跟上的灵活方式自动化操作并组合命令。

### 死掉的程序不会说谎

*一个死程序通常比一个瘫痪的程序造成的损害要小得多。*

该主题的作者的要点是尽早崩溃。

为什么？ 因为如果不这样做，程序可能看起来工作正常，但由于系统处于错误状态而返回包含小错误的结果。

你可以通过尽早和经常崩溃来避免这种情况。 这将帮助你更快地找到错误的根源，并且在程序出现问题时会更加明显。

将代码包装在 `try catch` 语句中可能会保持程序运行，但由此产生的细微错误更难捕获。

### 巧合式编程

*不要假设它，证明它。*

巧合的编程意味着你在猜测你正在使用的函数和接口。 它可能会工作一段时间，但是由于你**不知道**自己在使用什么，因此可能会出现错误，以后很难追踪。

结果是，无论你的代码是否正常工作，你都不知道为什么。

作为巧合编程的补救措施，不应该猜测给定接口的目的是什么，而是阅读文档。 如果你不能依赖文档，或者没有文档可以开始，请记录（并测试）那些假设。

我发现这个主题的建议在处理带有杂乱代码库的项目时很有帮助，因为函数会产生意想不到的副作用。

### 测试你的代码

*测试不是为了发现BUG。*

这个主题都是关于测试以及为什么我们首先测试我们的代码。 尤其是单元测试可以帮助我们以更加面向目标和解耦的方式思考我们的代码。

其中一位作者（Dave）暂时停止编写测试，看看它会对他的代码产生什么影响。 令他惊讶的是，他的代码质量丝毫没有受到影响。

这并不意味着你也应该停止编写测试。 他已经编程超过 45 年了，他会自动以一种解耦、可测试的方式编写代码。

### 务实的入门套件

这个主题更多的是对本书所涵盖主题的回顾。它涵盖了任何软件项目所需的最重要的东西，它们是：

- 版本控制

- 回归测试
  
  - 单元测试
  
  - 集成测试
  
  - 性能测试
  
  - 基于属性测试

- 全自动化



### 总结

书籍总是常读常新，我们在不同阶段去读同一本书，体会自然也是不同的。所关注的主题有时候也会有所侧重。



### 相关阅读

- Unit testing React - What you need to know： https://felixgerschau.com/unit-testing-react-introduction/

- SQL 背后的故事（每个版本 10 万个测试 case，会跑 10 亿个测试）: https://liyafu.com/2022-07-31-sqlite-untold-story/
