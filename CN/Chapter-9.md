# Chapter 9: Languages

## Introduced by Joe Hellerstein

>Readings:
>
>[Joachim W. Schmidt. Some High Level Language Constructs for Data of Type Relation. *ACM Transactions on Database Systems*, 2(3), 1977, 247-261.](https://scholar.google.com/scholar?cluster=5934767222958591409)
>
>[Arvind Arasu, Shivnath Babu, Jennifer Widom. The CQL Continuous Query Language: Semantic Foundations and Query Execution. *The VLDB Journal*, 15(2), 2006, 121-142.](https://scholar.google.com/scholar?cluster=17215743948117955326)
>
>[Peter Alvaro, Neil Conway, Joseph M. Hellerstein, William R. Marczak. Consistency Analysis in Bloom: A CALM and Collected Approach. *CIDR*, 2011.](https://scholar.google.com/scholar?cluster=9165311711752272482)https://scholar.google.com/scholar?cluster=4916926405792203059)

通过阅读数据库论文，您可能会认为典型的数据库用户是数据分析师、业务决策者或IT人员。实际上，大多数数据库用户是软件工程师，他们构建数据库支持的应用程序，这些应用程序在堆栈的更上层使用。虽然SQL最初是为非技术用户设计的，但是很少有人通过SQL这样的语言直接与数据库交互，除非他们正在编写一个数据库支持的应用程序。

那么，如果数据库系统主要是用于软件开发的api，它们能给程序员提供什么呢?像大多数好的软件一样，数据库系统提供了有力的抽象。

两个突出的优点：

1. 事务模型为程序员提供了一个单进程、顺序机器的抽象，该机器在任务中途不会出现故障。这就保护了程序员不受巨大的复杂性的影响，也就是现代计算的固有的并行性。今天，一个单一的计算机架有成千上万的内核在几十台机器上并行运行，这些机器可以独立地发生故障。然而，应用程序的程序员仍然可以轻率地编写顺序代码，就像1965年一样，他们把一副打孔卡片装进大型机，从头到尾单独执行。
2. SQL 等声明式查询语言为程序员提供了操作数据集的抽象。众所周知，声明式语言使程序员无需考虑如何访问数据项，而是让他们专注于返回哪些数据项。这种数据独立性还使应用程序程序员免受底层数据库组织变化的影响，并使数据库管理员免于参与应用程序的设计和维护。

只是，随着时间的推移，这些抽象概念有多大用处？今天它们有多大作用？

1. 作为一种编程结构，可序列化的事务一直很有影响力。对于程序员来说，用BEGIN和COMMIT/ROLLBACK括住他们的代码是很容易的。不幸的是，正如我们在第6章中所讨论的那样，事务是昂贵的，而且经常受到损害。"宽松的 "事务性语义打破了用户的串行抽象，并将应用逻辑暴露在潜在的竞赛和/或不可预测的异常中。如果应用程序开发人员想要说明这一点，他们就必须管理并发性、失败和分布式状态的复杂性。对缺乏事务的一个常见的反应是渴望 "最终一致性"[21]，正如我们在弱隔离部分讨论的那样。但正如我们在第6章中所讨论的，这仍然将所有的正确性负担转移给了应用开发者。在我看来，这种情况代表了现代软件开发的一个重大危机。
2. 声明式查询语言也取得了成功--肯定比之前的导航式语言有进步，导航式语言导致了每次重组数据库时都需要重写的面条式代码。不幸的是，查询语言与程序员通常使用的命令式语言有很大不同。查询语言消费和产生简单的无序的 "集合类型"（集合、关系、流）；编程语言通常进行指令的有序执行，通常是在复杂的结构化数据类型（树、哈希表等）上。数据库应用程序的程序员被迫在程序和数据库查询之间弥补这种所谓的 "阻抗不匹配"。从最早的关系型数据库开始，这一直是数据库程序员的一个麻烦。

## Database Language Embeddings: Pascal/R

本节的第一篇文章给出了解决第二个问题的经典示例:帮助命令式程序员解决阻抗不匹配问题。论文首先定义了我们现在可能认识到的(40多年后!)熟悉的集合类型的操作:Python中的“dictionary”类型、Java或Ruby中的“map”类型，等等。

这篇论文耐心地向我们介绍了各种语言结构的可能性和缺陷，这些结构似乎在几十年的应用中反复出现。一个关键的主题是区分枚举(用于生成输出)和量化(用于检查属性)的愿望——如果您是显式的，则后者通常可以得到优化。最后，本文提出了一种声明式的、类似sql的关系类型子语言，并将其嵌入到Pascal中。其结果是相对自然的，与当今一些更好的接口没有什么不同。

虽然这种方法现在看起来很自然，但这个话题花了几十年的时间才得到大众的关注。在这一过程中，数据库的 "连接性 "API（如ODBC和JDBC）作为C/C++和Java的拐杖出现了--它们允许用户向数据库管理系统推送查询，并通过结果进行迭代，但类型系统仍然是独立的，从SQL类型到主机语言类型的衔接是不愉快的。也许微软的LINQ库是Pascal/R等思想在现代的最佳演化，它提供了语言嵌入的集合类型和函数，因此应用程序开发人员可以在各种后端数据库和其他集合（XML文档、电子表格等）上编写类似查询的代码。

在2000年代，像社交媒体、在线论坛、互动聊天、照片分享和产品目录这样的网络应用在关系型数据库的后端被实现和重新实现。用于网络编程的现代脚本语言比Pascal更方便一些，而且通常包括体面的集合类型。在这种环境下，应用程序开发人员最终在他们的代码中看到了公认的模式，并将它们编入现在所谓的对象-关系映射（ORM）。Ruby on Rails是最有影响力的ORM之一，尽管现在还有很多其他的ORM。每一种流行的应用编程语言都至少有一个，而且在功能和理念上也有差异。有兴趣的读者可以参考维基百科的 "对象关系映射软件列表 "维基。

ORM为web程序员做了一些简单的事情。首先，它们为使用集合提供了原生语言的支持，非常类似于Pascal/R。其次，它们可以使对内存语言对象的更新透明地反映在数据库支持的状态中。它们通常为常见的数据库设计概念(如实体、关系、键和外键)提供一些原生语言语法。最后，包括Rails在内的一些orm提供了很好的工具，用于跟踪数据库模式随时间变化的方式，以反映应用程序代码中的变化(Rails 技术中的“迁移”)。

这是数据库研究界和行业应该更加关注的领域：这些是我们的用户！ ORM 和数据库 [6] 之间存在一些令人惊讶且偶尔令人不安的脱节。例如，Rails 的作者是一个名叫 David Heinemeier Hansson（“DHH”）的多彩人物，他相信“有意见的软件”（当然这反映了他的观点）。他被引述如下：

> I don’t want my database to be clever! ...I consider stored procedures and constraints vile and reckless destroyers of coherence. No, Mr. Database, you can not have my business logic. Your procedural ambitions will bear no fruit and you’ll have to pry that logic from my dead, cold object-oriented hands . . . I want a single layer of cleverness: My domain model.

这种不愿意相信DBMS的态度导致了Rails应用中的许多问题。针对ORM编写的应用程序通常都很慢--ORM本身并没有对生成查询的方式做什么优化。相反，Rails程序员往往需要学习 "不同 "的编程方式，以鼓励Rails生成高效的SQL--类似于Pascal/R论文中的讨论，他们需要学习避免循环和每次表的迭代。一个典型的Rails应用程序的演变是：天真地写，观察缓慢的性能，研究它产生的SQL日志，并重写应用程序以说服ORM产生 "更好的 "SQL。Cheung及其同事最近的工作探讨了程序合成技术可以自动生成这些优化的想法[9]；这是一个有趣的方向，时间会证明它可以自动消除多少复杂性。数据库和应用程序之间的分离也会对正确性产生负面影响。例如，Bailis最近展示了[6]许多现有的开源Rails应用程序由于在应用程序中（而不是在数据库中）执行不当而容易受到完整性侵犯。

尽管有一些盲点，ORM在数据库支持的应用程序的可编程性方面通常是一个重要的实际飞跃，并且验证了最早可以追溯到Pascal/R的想法。一些好的想法需要时间来实现。

## Stream Queries: CQL

我们的第二篇关于CQL的论文是一篇不同风格的语言工作——它是一篇查询语言设计论文。它为流的数据模型提供了一种新的声明性查询语言的设计。这篇论文被采纳有几个原因。 首先，它是一个简洁、易读、相对现代的查询语言设计示例。每隔几年，就会出现一群人，他们使用另一种数据模型和查询语言:例如对象和OQL、XML和XQuery，或者RDF和SPARQL。这些语言大多以断言“X改变了某些数据模型X的一切”开始，这导致了一种新的查询语言的出现，这种语言通常看起来很熟悉，但与SQL却有奇怪的不同。CQL是一个令人耳目一新的语言设计示例，因为它的作用正好相反:它强调了这样一个事实，即通过正确的视角查看的流数据实际上变化很小。CQL对SQL的发展足以隔离查询“静态”表和“移动”流之间的关键区别。这给了我们一个清晰的理解什么是真正不同的，在语义上，当你不得不谈论流数据的时候;许多其他当前的流语言比CQL更加特别和混乱。除了这篇论文是深思熟虑的查询语言设计的一个很好的范例之外，它还代表了一个在数据库文献中受到很多关注的研究领域，并且在实践中仍然很有趣。从2000年早期开始的第一代流数据研究系统[3,120,118,36]既没有作为开放源码，也没有作为从这些系统中产生的各种初创公司的主流。然而，流查询的话题近年来再次引起了业界的兴趣，像Spark- Streaming、Storm和Heron这样的开源系统看到了它的发展，像谷歌这样的公司也意识到连续数据流作为现代服务[8]的一种新现实的重要性。我们可能还会看到，流查询系统在金融服务领域所占的份额，已经超出了它们目前的小众市场。

CQL有趣的另一个原因是流是数据库和“事件”之间的中间地带。数据库存储和检索集合类型;事件系统传输和处理离散事件。一旦您将事件视为数据，那么事件编程和流编程看起来就非常相似。考虑到事件编程在某些领域(例如面向用户的Javascript、面向分布式系统的Erlang)是一种广泛使用的编程模型，事件编程语言(如Javascript)与数据流系统之间应该存在相对较小的阻抗失配。这种方法的一个有趣的例子是Rx(反应性扩展)语言，它是LINQ的一个流添加，使编程事件流感觉像编写函数查询计划;或者正如它的作者Erik Meijer所说，“你的鼠标就是一个数据库”[114]。

## Programming Correct Applications without Transactions: Bloom

第三篇关于Bloom的论文连接了以上几点;它有一个应用程序级别的状态关系模型，以及一个与CQL流相关的网络通道概念。但是主要的目标是帮助程序员管理本章开头介绍的第一个抽象的损失;我称之为重大危机。现代开发人员面临的一个大问题是:您能否为您的程序找到一个正确的分布式应用程序，而不使用事务或其他昂贵的方案来控制操作顺序?

Bloom对这个问题的回答是，为程序员提供一种“无序”的编程语言:一种不鼓励他们偶然使用排序的语言。Bloom的默认数据结构是关系;它的基本编程结构是可以以任何顺序运行的逻辑规则。简而言之，它是一种与关系查询语言类似的通用语言。SQL查询可以在不改变输出的情况下进行优化和并行化，出于同样的原因，简单的Bloom程序具有定义良好的(一致的)结果，而与执行顺序无关。这种直觉的一个例外是Bloom代码的行是“非单调的”，测试随着时间的推移在真与假之间摇摆的属性(例如“NOT EXISTS x”或“HAVING COUNT() = x”)。这些规则对执行和消息排序很敏感，需要协调机制的“保护”。

CALM定理形式化了这个概念，明确地回答了上面的问题:当且仅当它的规范是单调的时，你可以为你的程序找到一个一致的、分布式的、无协调的实现[84,14]。Bloom论文还说明了编译器如何在实践中使用CALM来确定Bloom程序中对协调的需求。在程序员注释[12]的帮助下，CALM分析也可以应用于Storm等系统中的数据流语言。在[13]中对这一领域的理论结果进行了综述。已经有很多关于避免协调的相关语言工作:一些论文建议使用联想、交换、幂等运算[83,142,42];它们本质上是单调的。另一组工作检查可选的正确性标准，例如，只确保数据库状态[20]上的特定不变量，或者使用可选的程序分析来获得可反序列化的结果，而不影响传统的读写并发性[137]。这个领域仍然很新;论文有不同的模型(例如，有些有事务边界，有些没有)，并且常常对“一致性”或“协调”的定义不一致。(CALM将一致性定义为全局确定的结果，协调定义为无论数据分区还是复制[14]都需要的消息传递。)在这里获得更清晰的思路是很重要的——如果程序员不能进行事务处理，那么他们就需要在应用程序开发层获得帮助。

Bloom也是数据库研究中反复出现的主题的一个例子:通用声明性语言(也称为“逻辑编程”)。Datalog是一个标准的例子，在数据库研究中有着悠久而有争议的历史。Datalog是20世纪80年代数据库理论家最喜欢讨论的一个话题，但由于在实践中不相关，它遭到了当时系统研究人员的强烈反对[152]。最近，它得到了数据库和其他应用领域(年轻的)研究人员的一些关注[74]——例如，Nicira的软件定义网络栈(被VMWare以10亿美元的价格收购)使用Datalog语言进行网络转发状态[97]。使用声明性子语言来访问数据库状态和非常积极地使用声明性编程(如Bloom)来指定应用程序逻辑之间存在一定的差异。时间会告诉我们，在各种不同的环境下，包括基础设施、应用程序、web客户机和移动设备，程序员的这种命令式边界是如何变化的。

## References:

[1] Abadi, D.J., Carney, D., Çetintemel, U., Cherniack, M., Convey, C., Lee, S., Stonebraker, M., Tatbul, N. and Zdonik, S. Aurora: A new model and architecture for data stream management. *The VLDB Journal—The International Journal on Very Large Data Bases*. 12, 2 (2003), 120-139.

[2] Akidau, T. and others The dataflow model: A practical approach to balancing correctness, latency, and cost in massive-scale, unbounded, out-of-order data processing. *VLDB*, 2015.

[3] Alvaro, P., Conway, N., Hellerstein, J.M. and Maier, D. Blazes: Coordination analysis for distributed programs. *Data engineering (iCDE), 2014 iEEE 30th international conference on*, 2014, 52-63.

[4] Ameloot, T.J. Declarative networking: Recent theoretical work on coordination, correctness, and declarative semantics. *ACM SIGMOD Record*. 43, 2 (2014), 5-16.

[5] Ameloot, T.J., Neven, F. and Van den Bussche, J. Relational transducers for declarative networking. *Journal of the ACM (JACM)*. 60, 2 (2013), 15.

[6] Bailis, P., Fekete, A., Franklin, M.J., Ghodsi, A., Hellerstein, J.M. and Stoica, I. Feral Concurrency Control: An empirical investigation of modern application integrity. *SIGMOD*, 2015.

[7] Bailis, P., Fekete, A., Franklin, M.J., Hellerstein, J.M., Ghodsi, A. and Stoica, I. Coordination avoidance in database systems. *VLDB*, 2015.

[8] Chandrasekaran, S., Cooper, O., Deshpande, A., Franklin, M.J., Hellerstein, J.M., Hong, W., Krishnamurthy, S., Madden, S., Raman, V., Reiss, F. and others TelegraphCQ: Continuous dataflow processing for an uncertain world. *CIDR*, 2003.

[9] Cheung, A., Arden, O., Madden, S., Solar-Lezama, A. and Myers, A.C. StatusQuo: Making familiar abstractions perform using program analysis. *CIDR*, 2013.

[10] Clements, A.T., Kaashoek, M.F., Zeldovich, N., Morris, R.T. and Kohler, E. The scalable commutativity rule: Designing scalable software for multicore processors. *ACM Transactions on Computer Systems (TOCS)*. 32, 4 (2015), 10.

[11] Green, T.J., Huang, S.S., Loo, B.T. and Zhou, W. Datalog and recursive query processing. *Foundations and Trends in Databases*. 5, 2 (2013), 105-195.

[12] Helland, P. and Campbell, D. Building on quicksand. *CIDR*, 2009.

[13] Hellerstein, J.M. The declarative imperative: Experiences and conjectures in distributed logic. *ACM SIGMOD Record*. 39, 1 (2010), 5-19.

[14] Koponen, T., Amidon, K., Balland, P., Casado, M., Chanda, A., Fulton, B., Ganichev, I., Gross, J., Gude, N., Ingram, P. and others Network virtualization in multi-tenant datacenters. *USENIX nSDI*, 2014.

[15] Meijer, E. Your mouse is a database. *Queue*. 10, 3 (2012), 20.

[16] Motwani, R., Widom, J., Arasu, A., Babcock, B., Babu, S., Datar, M., Manku, G., Olston, C., Rosenstein, J. and Varma, R. Query processing, resource management, and approximation in a data stream management system. *CIDR*, 2003.

[17] Naughton, J.F., DeWitt, D.J., Maier, D., Aboulnaga, A., Chen, J., Galanis, L., Kang, J., Krishnamurthy, R., Luo, Q., Prakash, N. and others The niagara internet query system. *IEEE Data Eng. Bull.* 24, 2 (2001), 27-33.

[18] Roy, S., Kot, L., Bender, G., Ding, B., Hojjat, H., Koch, C., Foster, N. and Gehrke, J. The homeostasis protocol: Avoiding transaction coordination through program analysis. *SIGMOD*, 2015.

[19] Shapiro, M., Preguica, N., Baquero, C. and Zawirski, M. Shapiro Marc. Preguica Nuno. Baquero Carlos. Zawirski Marek. A comprehensive study of convergent and commutative replicated data types. INRIA TR 7506. 2011.

[20] Stonebraker, M. and Neuhold, E. The laguna beach report. International Institute of Computer Science. 1989.

[21] Terry, D.B., Demers, A.J., Petersen, K., Spreitzer, M.J., Theimer, M.M. and others Session guarantees for weakly consistent replicated data. *PDIS*, 1994.
