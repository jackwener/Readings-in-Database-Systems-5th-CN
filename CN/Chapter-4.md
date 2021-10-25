# Chapter 4: New DBMS Architectures

## Introduced by Michael Stonebraker

> Readings:
>
> [Mike Stonebraker, Daniel J. Abadi, Adam Batkin, Xuedong Chen, Mitch Cherniack, Miguel Ferreira, Edmond Lau, Amerson Lin, Sam Madden, Elizabeth O'Neil, Pat O'Neil, Alex Rasin, Nga Tran, Stan Zdonik. C-store: A Column-oriented DBMS. *SIGMOD*, 2005.](https://scholar.google.com/scholar?cluster=12924804892742402591)
>
> [Cristian Diaconu, Craig Freedman, Erik Ismert, Per-Ake Larson, Pravin Mittal, Ryan Stonecipher, Nitin Verma, Mike Zwilling. Hekaton: SQL Server's Memory-optimized OLTP Engine. *SIGMOD*, 2013.](https://scholar.google.com/scholar?cluster=14161764654889427045)
>
> [Stavros Harizopoulos, Daniel J. Abadi, Samuel Madden, Michael Stonebraker. OLTP Through the Looking Glass, and What We Found There. *SIGMOD*, 2008.](https://scholar.google.com/scholar?cluster=12931776946707721868)https://scholar.google.com/scholar?cluster=17642052422667212790)

在 DBMS 领域发生的最重要的事情可能是 “one size fits all” 的消亡。直到21世纪初，传统基于磁盘的行存储架构一直一统天下。实际上，厂商们有一把锤子，所有东西都是钉子。

在过去的15年里，发生了几次重大的变更，让我们依次讨论。

首先，社区意识到，在数据仓库领域上列存比行存有很大优势。数据仓库在面向客户的零售环境中发现了早期的接受度，并迅速蔓延到面向客户的一般数据。仓库记录了客户交易的历史信息。实际上，这是每个客户互动的 "who-what-why-when-where"。

传统的智慧是围绕一个中央事实表来构建数据仓库，在这个表中记录交易信息。围绕着这个表的是维度表，它记录了可以从事实表中分解出来的信息。在零售业的情况下，我们有商店、客户、产品和时间的维度表。其结果是一个所谓的星形模式[3]。如果商店被分组为区域，那么可能会有多层次的维度表和雪花模式的结果。

关键的观察是，事实表通常是"臃肿"的，经常包含一百个或更多的属性。很明显，它们也很 "长"，因为有很多很多的事实需要记录。一般来说，对数据仓库的查询是重复请求（按商店生成每月的销售报告）和 "临时 "请求的混合。例如，在一个零售仓库中，人们可能想知道当暴风雪发生时，东北地区的销售情况如何，以及在飓风期间，大西洋沿岸的销售情况如何。

此外，没有人会运行`select *`来获取表中的所有行。相反，他们总是指定一个集合，从表中的100条属性中检索出半打属性。下一个查询会检索到不同的属性集，而且在过滤条件中几乎没有定位。

在这个用例中，很明显，列存储从磁盘到主内存的数据量比行存储要少16倍（6列对100）。因此，它有一个不公平的优势。此外，考虑一个存储块。在列存储中，该块上只有一个属性，而行存储将有100个。压缩在一个属性上显然比在100个属性上效果更好。此外，行存储在每条记录的前面都有一个头（在 SQLServer 中，它显然是16字节）。相比之下，列式存储则注意没有这样的头。

最后，基于行的执行器有一个内循环，记录在输出中被检查是否有效。因此，内循环的开销是相当大的，是为每条检查的记录支付的。相比之下，列式存储的基本操作是检索一个列并挑选出合格的项目。因此，内循环的开销是每检查一列支付一次，而不是每检查一行支付一次。因此，列执行器在CPU时间上更有效率，从磁盘上检索的数据也更少。在大多数实际环境中，列存比行存快 50-100 倍。

早期的列存包括20世纪90年代出现的 Sybase IQ[5]，以及 MonetDB[2]。然而，该技术可以追溯到20世纪70年代[1，4]。在2000年，C-Store/Vertica 作为资金雄厚的初创公司出现，并有高性能的实现。在接下来的十年里，整个数据仓库领域的世界从行存演变成了列存。可以说，如果 Sybase 在技术上有更积极的投资并进行多节点的实施，那么 Sybase IQ 也可以更早地完成同样的事情。列执行器的优点在[2]中得到了令人信服的讨论，尽管它是 "在杂草中"，难以阅读。

第二个重大变化是主存储器价格的急剧下降。目前，人们可以用25,000美元买到1TB的内存，而用几TB的高性能计算集群也许可以用10万美元买到。关键的见解是，OLTP数据库并没有那么大。一兆字节是一个非常大的OLTP数据库，并且是主内存部署的候选者。正如本节中的望远镜论文所指出的，当数据适合于主内存时，人们并不想运行基于磁盘的行存储--开销太高了。

实际上，OLTP市场现在正成为一个主内存 DBMS 市场。同样，传统的基于磁盘的行存储是没有竞争力的。为了工作得好，需要为并发控制、崩溃恢复和多线程提供新的解决方案，我预计OLTP架构将在未来几年内不断发展。

我目前的最佳猜测是，没有人会使用传统的两阶段锁。基于时间戳排序或多个版本的技术可能会占上风。本节的第三篇论文讨论了 Hekaton，它实现了最先进的 MVCC 方案。

在任何情况下，OLTP 都将转移到主内存部署，而一类新的主内存 DBMS 正在展开，以支持这种使用情况。

已经展开的第三个现象是 "no SQL "运动。实质上，有 100 个左右的 DBMS，它们支持各种数据模型，具有以下两个特点:

1. "开箱即用 "的经验。它们对程序员来说很容易上手，做一些有意义的事情。与此相反，RDBMSs是非常沉重的，需要一个模式在前面。
2. 对半结构化数据的支持。如果每条记录都可以有不同的属性值，那么传统的行存储将有非常非常宽的图元，而且非常稀疏，因此效率很低。

这给商业厂商敲响了警钟，让他们制造出更容易使用和支持半结构化数据类型的系统，如JSON。总的来说，我预计随着RDBMS对上述两点的反应，No SQL市场将逐渐与SQL市场合并。

第四个变化是 Hadoop/HDFS/Spark 环境的出现，这将在第六章中讨论。

## References:

[1] Batory, D.S. On searching transposed files. *ACM Transactions on Database Systems (TODS)*. 4, 4 (Dec. 1979).

[2] Boncz, P.A., Zukowski, M. and Nes, N. MonetDB/X100: Hyper-pipelining query execution. *CIDR*, 2005.

[3] Kimball, R. and Ross, M. The data warehouse toolkit: The complete guide to dimensional modeling. John Wiley & Sons. 2011.

[4] Lorie, R. and Symonds, A. A relational access method for interactive applications. *Courant Computer Science Symposia, Vol. 6: Data Base Systems*. (1971).

[5] MacNicol, R. and French, B. Sybase iQ multiplex-designed for analytics. *VLDB*, 2004.

[6] Malviya, N., Weisberg, A., Madden, S. and Stonebraker, M. Rethinking main memory OLTP recovery. *ICDE*, 2014.