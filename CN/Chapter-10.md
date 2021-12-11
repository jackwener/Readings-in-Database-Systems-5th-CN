# Chapter 10: Web Data

## Introduced by Peter Bailis

>Readings:
>
>[Sergey Brin and Larry Page. The Anatomy of a Large-scale Hypertextual Web Search Engine. *WWW*, 1998.](https://scholar.google.com/scholar?cluster=9820961755208603037)
>
>[Eric A. Brewer. Combining Systems and Databases: A Search Engine Retrospective. *Readings in Database Systems, Fourth Edition*, 2005.](https://scholar.google.com/scholar?cluster=15869287167041695406)
>
>[Michael J. Cafarella, Alon Halevy, Daisy Zhe Wang, Eugene Wu, Yang Zhang. WebTables: Exploring the Power of Tables on the Web. *VLDB*, 2008.](https://scholar.google.com/scholar?cluster=11659194181134800008)

自本文集的上一版以来，万维网已经明确地将任何有关其寿命和全球影响的遗留问题抛之脑后。包括谷歌和脸书在内的几个数十亿用户的服务已经成为第一世界现代生活的核心，而互联网和网络相关技术已经渗透到商业和个人互动中。毫无疑问，网络将继续存在，至少在可预见的未来。

网络数据系统带来了一系列新的挑战，包括高规模、数据异质性，以及复杂的、不断发展的用户交互模式。经典的关系型数据库系统的设计并没有考虑到网络的工作量，在这种情况下也不是首选的技术。相反，网络数据管理需要一系列的技术，包括信息检索、数据库内部、数据集成和分布式系统。在本节中，我们包括三篇论文，它们强调了对网络数据管理中固有问题的技术解决方案。

我们的前两篇论文描述了搜索引擎和索引技术的内部结构。我们的第一篇论文来自谷歌联合创始人拉里佩奇和谢尔盖布林，描述了谷歌早期原型的内部结构。从历史和技术角度来看，这篇论文都很有趣。第一个 Web 索引，例如 Yahoo!，由人工策划的“目录”组成。虽然目录管理被证明是有用的，但目录难以扩展并且需要相当多的人力来维护。因此，许多搜索引擎，包括谷歌和第二篇论文的作者 Eric Brewer 共同创建的 Inktomi，都在寻求自动化方法。这些引擎的设计在概念上很简单：一组爬虫下载网络数据的副本并构建（和维护）用于计算相关性评分函数的只读索引。反过来，查询由前端 Web 服务提供服务，该服务从索引中读取并呈现一组有序的结果，按评分函数排序。

这些引擎的实现和实现是复杂的。例如，评分算法经过高度调整，即使在今天的搜索引擎中，它们的实施也被认为是商业秘密：Web 作者有很大的动机来操纵评分函数来为自己谋利。 Google 论文中描述的 PageRank 算法（并在 [5] 中有详细说明）是评分函数的一个著名示例，它根据超链接图测量每个页面的“影响力”。两篇论文都描述了如何在实践中使用大部分未指定属性的组合进行评分，包括“锚文本”（提供链接源的上下文）和其他形式的元数据。这些技术的算法基础，例如关键字索引日期，可以追溯到 1950 年代 [4]，而其他技术，例如 TFxIDF 排名和倒排索引，可以追溯到 1960 年代 [6]。构建互联网搜索引擎的许多关键系统创新来自于扩展它们和处理脏的、异构的数据源。

虽然这些论文的高级细节有助于理解现代搜索引擎的运作方式，但这些论文对构建生产 Web 搜索引擎的过程的评论也很有趣。每个中的一个中心信息是 Web 服务必须考虑多样性；谷歌的作者描述了在典型的信息检索技术中做出的假设如何不再适用于 Web 上下文（例如，“比尔·克林顿很烂”的网页）。网络资源以不同的速度变化，需要优先爬行以维护最新的索引。 Brewer 还强调了容错和操作可用性的重要性，呼应了他在现场构建 Inktomi 的经验（这也导致了包括收获和产量 [2] 和 CAP 定理在内的概念的发展；参见第 7 章）。 Brewer 概述了使用商品数据库引擎构建搜索引擎的困难（例如，Informix 比 Inktomi 的自定义解决方案慢 10 倍）。然而，他指出数据库系统设计的原则，包括“自顶向下”设计、数据独立性和声明式查询引擎，在这种情况下是有价值的——如果适当调整的话。

今天，网络搜索引擎被认为是成熟的技术。然而，相互竞争的服务通过增加额外的功能不断改善搜索体验。今天的搜索引擎远不止是文本数据网页的信息检索引擎；前两篇论文的内容只是谷歌或百度等服务内部的一小部分。这些服务提供了一系列的功能，包括定向广告、图片搜索、导航、购物和移动搜索。毫无疑问，这些领域之间在检索、实体解析和索引技术方面存在渗入，但每个领域都需要特定的适应性。
作为一个由海量网络数据促成的新型搜索的例子，我们包括一篇来自Google的Alon Halevy领导的WebTables项目的论文。WebTables允许用户查询和理解存储在HTML表中的数据之间的关系。由于缺乏固定的模式，HTML表在结构上本来就是多样的。然而，在网络规模上聚集足够多的表格，并进行一些轻量级的自动数据整合，可以实现一些有趣的查询（例如，一个流感爆发地点的表格可以与一个包含城市人口数据的表格相结合）。挖掘这些表的模式，确定它们的结构和真实性（例如，在论文语料库中只有1%的表实际上是关系），并有效地推断它们的关系是很困难的。我们收录的论文描述了建立属性相关统计数据库（AcsDB）的技术，以回答关于表元数据的查询，实现了包括模式自动完成的新功能。WebTables项目今天仍以各种形式继续进行，包括Google表搜索和与Google核心搜索技术的整合；关于该项目的更新可以在[1]中找到。产生结构化搜索结果的能力在一些非传统的领域是可取的，包括移动、上下文和基于音频的搜索。
WebTables的论文特别强调了大规模处理网络数据的力量。在2009年的一篇文章中，Halevy及其同事描述了 "数据的不合理有效性"，有效地论证了在有足够的数据量的情况下，有足够的潜在结构被捕获以使建模更简单：相对简单的数据挖掘技术往往能击败数学上更复杂的统计模型[3]。这个论点强调了通过纯粹的数据量和计算来解开隐藏结构的潜力，无论是挖掘模式的相关性还是在语言之间进行机器翻译。有了一个足够大的草堆，针就会变得很大。即使检查网络语料库中1%的表，VLDB 2009的论文也研究了1.54亿个不同的关系，这个语料库 "比[以前]考虑的最大的语料库大五个数量级。"
由于廉价的商品存储和云计算资源，在这些公司之外进行大规模数据集和系统架构分析的障碍正在减少。然而，很难复制用户（如垃圾邮件发送者）和算法（如搜索排名算法）之间的反馈循环。互联网公司在开拓考虑这种反馈循环的系统设计方面具有独特的地位。随着数据库技术为更多的互动领域提供动力，我们相信这种模式将变得更加重要。也就是说，数据库市场和有趣的数据库工作负载可能从类似的分析中受益。例如，对亚马逊Redshift和微软SQL Azure等托管数据库平台进行类似的分析将是有趣的，这些平台可以实现各种功能，包括索引自动调整、自适应查询优化、从非结构化数据中发现模式、查询自动完成和可视化建议。

## References:

[1] Balakrishnan, S., Halevy, A., Harb, B., Lee, H., Madhavan, J., Rostamizadeh, A., Shen, W., Wilder, K., Wu, F. and Yu, C. Applying webTables in practice. *CIDR*, 2015.

[2] Brewer, E. and others Lessons from giant-scale services. *Internet Computing, IEEE*. 5, 4 (2001), 46-55.

[3] Halevy, A., Norvig, P. and Pereira, F. The unreasonable effectiveness of data. *IEEE Intelligent Systems*. 24, 2 (Mar. 2009), 8-12.

[4] Luhn, H.P. Auto-encoding of documents for information retrieval systems. *Modern Trends in Documentation*. (1959), 45-58.

[5] Page, L., Brin, S., Motwani, R. and Winograd, T. The PageRank citation ranking: Bringing order to the web. Stanford InfoLab. 1999. SIDL-WP-1999-0120.

[6] Salton, G. and Lesk, M.E. Computer evaluation of indexing and text processing. *Journal of the ACM (JACM)*. 15, 1 (1968), 8-36.
