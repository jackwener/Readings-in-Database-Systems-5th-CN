# Chapter 2: Traditional RDBMS Systems

## Introduced by Michael Stonebraker

> Readings:
>
> [Morton M. Astrahan, Mike W. Blasgen, Donald D. Chamberlin, Kapali P. Eswaran, Jim Gray, Patricia P. Griffiths, W. Frank King III, Raymond A. Lorie, Paul R. McJones, James W. Mehl, Gianfranco R. Putzolu, Irving L. Traiger, Bradford W. Wade, Vera Watson. System R: Relational Approach to Database Management. *ACM Transactions on Database Systems*, 1(2), 1976, 97-137.](https://scholar.google.com/scholar?cluster=15466550502837111601)
>
> [Michael Stonebraker and Lawrence A. Rowe. The design of POSTGRES. *SIGMOD*, 1986.](https://scholar.google.com/scholar?cluster=7945977557090027847)
>
> [David J. DeWitt, Shahram Ghandeharizadeh, Donovan Schneider, Allan Bricker, Hui-I Hsiao, Rick Rasmussen. The Gamma Database Machine Project. *IEEE Transactions on Knowledge and Data Engineering*, 2(1), 1990, 44-62.](https://scholar.google.com/scholar?cluster=8912521541627865753)

本章讨论的论文（可以说是）是关于 DBMS 系统最重要的三篇论文。我们将在这章中按时间顺序讨论它们。

System R 项目是在 IBM 研究院的 Frank King 的指导下开始的--大概在 1972 年。那时 Ted Codd 的开创性论文已经发表了 18 个月，对很多人来说，显然一般都会建立一个原型系统来测试他的 ideas。不幸的是，Ted 没有被允许领导这项工作，于是他转而去思考 DBMS 的 natural language interfaces 了。System R 很快决定实现 SQL，SQL 是从1972年的干净的块结构语言（block structured language）[[2](http://www.redbook.io/ch2-importantdbms.html#ref-chamberlin-sequel)] 演变成为这篇论文[[1](http://www.redbook.io/ch2-importantdbms.html#ref-chamberlin-history)]中描述的更复杂的结构。关于SQL语言设计的评述见 [[3](http://www.redbook.io/ch2-importantdbms.html#ref-date)] ，它是在十年后写的。

System R 的结构分为两组，"下半部"和"上半部"。它们并不是完全同步的，因为下半部分实现了链接，而上半部分并不支持这些链接。为下半部团队的决定辩护，很明显他们是在与IMS竞争，后者有这样的结构，所以包括它是很自然的。上半部分只是没有让优化器为这种结构工作。

transaction manager 可能是这个项目最大的遗产，它然是已故的 Jim Gray 的作品。他的许多设计至今仍在商业系统中延续。第二名是 System R 的优化器。基于成本（cost-based）的动态规划（dynamic programming）方法仍然是优化器技术的黄金标准。

我对 System R 最大的抱怨是，这个团队从来没有停下来去清理 SQL 。因此，"上半部"整体被简单地粘在 VSAM （注：[Virtual Storage Access Method - Wikipedia](https://en.wikipedia.org/wiki/Virtual_Storage_Access_Method)）上形成DB2时，语言层被完整地保留下来。该语言的所有恼人的特征一直持续到今天。SQL将是2020年的 COBOL，一种我们被困于其中的语言，所有人都会抱怨。

我第二大的抱怨是，System R 使用子程序调用接口(subroutine call interface)（现在的 ODBC ）将客户端应用程序与 DBMS 耦合。我认为 ODBC 是这个星球上最糟糕的接口之一。要发布一个单一的查询，必须打开一个数据库，打开一个游标，将其与一个查询绑定，然后为数据记录发布单独的检索。仅仅为了运行一个查询，就需要一页相当难以理解的代码。Ingres[11]和Chris Date[4]都有更简洁的语言嵌入。此外，Pascal-R[9] 和 Rigel[7] 也是在编程语言中包含 DBMS 功能的优雅方式。直到最近，随着 Linq[6] 和 Ruby on Rails[8] 的出现，我们才看到干净的特定语言嵌入的重新出现。

在 System R 之后，Jim Gray 去了 Tandem 从事 Non-stop SQ L的工作，Kapali Eswaren 做了一个关系型的启动。团队中的大多数人都留在了 IBM ，并继续从事其他各种项目的工作，包括 R*。

第二篇论文涉及 Postgres。这个项目开始于1984年，当时很明显，继续使用学术上的 Ingres 代码库做原型是没有意义的。对 Postgres 历史的叙述出现在[10]中，读者可以在那里看到对开发过程中的起伏的全面回顾。

然而，在我看来，Postgres 的重要遗产是其抽象数据类型（ADT）系统。用户定义的类型和函数已经被添加到大多数主流的关系型 DBMS 中，使用 Postgres 的模型。因此，这个设计特点一直延续到今天。该项目还试验了时间旅行，但效果并不理想。我认为，随着更快的存储技术改变了数据管理的经济性，无覆盖存储会有它的一天。

还应该指出的是，Postgres 的大部分重要性来源于于一个强大的、性能良好且高可用的开源代码工作线。这是开源社区开发和维护模式的一个最佳例子。在20世纪90年代中期，一个志愿者组建的自上而下的团队接管了 Berkeley 代码线，并从那时起一直在指导其发展。Postgres 和 4BSD Unix[[5](http://www.redbook.io/ch2-importantdbms.html#ref-bsdunix)]  都推动了使开源成为代码开发的首选机制。

Postgres 项目一直在伯克利直到到1992年，当时成立了商业公司 Illustra 来支持商业代码工作线（commercial code line）。关于 Illustra 在市场上经历的起伏，见 [[10](http://www.redbook.io/ch2-importantdbms.html#ref-stonebraker-turing)] 。

除了 ADT 系统和开源分发模型（open source distribution model），Postgres 项目的一个重要遗产是培养一代训练有素的 DBMS 实现者，他们在建立其他几个商业系统时发挥了重要作用。

本节中的第三个系统是 Gamma，1984 年至 1990 年期间在威斯康星州构建的。在我看来，Gamma 普及了多节点数据管理的无共享分区表方法。尽管Teradata也有同样的想法，但正是 Gamma 推广了这些概念。此外，在 Gamm a之前，没有人谈论 Hash Join，所以 Gamma 应该被认为是（与 Kitsuregawa Masaru 一起）提出了这一类算法。

基本上所有的数据仓库系统都使用 Gamma 风格的架构。使用 shared disk 或 shared memory 系统的 idea 都几乎消失了。除非网络延迟和带宽能够与磁盘带宽接近，否则我预计目前的 shared-nothing 架构将继续存在。

## References:

[1] Chamberlin, D.D. Early history of sQL. *Annals of the History of Computing, IEEE*. 34, 4 (2012), 78-82.

[2] Chamberlin, D.D. and Boyce, R.F. SEQUEL: A structured english query language. *Proceedings of the 1974 aCM sIGFIDET (now sIGMOD) workshop on data description, access and control*, 1974, 249-264.

[3] Date, C.J. A critique of the SQL database language. *ACM SIGMOD Record*. 14, 3 (Nov. 1984).

[4] Date, C.J. An architecture for high-level language database extensions. *SIGMOD*, 1976.

[5] McKusick, M.K., Bostic, K., Karels, M.J. and Quarterman, J.S. The design and implementation of the 4.4 BSD operating system. Pearson Education. 1996.

[6] Meijer, E., Beckman, B. and Bierman, G. Linq: Reconciling object, relations and XML in the .NET framework. *SIGMOD*, 2006.

[7] Rowe, L.A. and Shoens, K.A. Data abstraction, views and updates in RIGEL. *SIGMOD*, 1979.

[8] Ruby on rails: Ruby on rails. [http://www.rubyonrails.org](http://www.rubyonrails.org/).

[9] Schmidt, J.W. Some high level language constructs for data of type relation. *ACM Trans. Database Syst.* 2, 3 (Sep. 1977).

[10] Stonebraker, M. The land sharks are on the squawk box. *Communications of the ACM*.

[11] Stonebraker, M., Held, G., Wong, E. and Kreps, P. The design and implementation of iNGRES. *ACM Transactions on Database Systems (TODS)*. 1, 3 (1976), 189-222.
