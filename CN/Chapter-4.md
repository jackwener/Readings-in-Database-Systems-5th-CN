# Chapter 4: New DBMS Architectures

## Introduced by Michael Stonebraker

> Readings:
>
> [Mike Stonebraker, Daniel J. Abadi, Adam Batkin, Xuedong Chen, Mitch Cherniack, Miguel Ferreira, Edmond Lau, Amerson Lin, Sam Madden, Elizabeth O'Neil, Pat O'Neil, Alex Rasin, Nga Tran, Stan Zdonik. C-store: A Column-oriented DBMS. *SIGMOD*, 2005.](https://scholar.google.com/scholar?cluster=12924804892742402591)
>
> [Cristian Diaconu, Craig Freedman, Erik Ismert, Per-Ake Larson, Pravin Mittal, Ryan Stonecipher, Nitin Verma, Mike Zwilling. Hekaton: SQL Server's Memory-optimized OLTP Engine. *SIGMOD*, 2013.](https://scholar.google.com/scholar?cluster=14161764654889427045)
>
> [Stavros Harizopoulos, Daniel J. Abadi, Samuel Madden, Michael Stonebraker. OLTP Through the Looking Glass, and What We Found There. *SIGMOD*, 2008.](https://scholar.google.com/scholar?cluster=12931776946707721868)https://scholar.google.com/scholar?cluster=17642052422667212790)

在DBMS领域发生的最重要的事情可能是 "一刀切 "的死亡。直到21世纪初，传统的基于磁盘的行存储架构是全面存在的。实际上，商业供应商有一把锤子，所有东西都是钉子。

在过去的15年里，发生了几次重大的动荡，我们依次讨论这些动荡。

首先，社区意识到，在数据仓库市场上，列存储比行存储有很大的优势。数据仓库在面向客户的零售环境中发现了早期的接受度，并迅速蔓延到面向客户的一般数据。仓库记录了客户交易的历史信息。实际上，这是每个客户互动的 "谁-什么-为什么-什么时候-什么地方"。

传统的智慧是围绕一个中央事实表来构建数据仓库，在这个表中记录交易信息。围绕着这个表的是维度表，它记录了可以从事实表中分解出来的信息。在零售业的情况下，我们有商店、客户、产品和时间的维度表。其结果是一个所谓的星形模式[3]。如果商店被分组为区域，那么可能会有多层次的维度表和雪花模式的结果。

关键的观察是，事实表通常是 "胖 "的，经常包含一百个或更多的属性。很明显，它们也很 "长"，因为有很多很多的事实需要记录。一般来说，对数据仓库的查询是重复请求（按商店生成每月的销售报告）和 "临时 "请求的混合。例如，在一个零售仓库中，人们可能想知道当暴风雪发生时，东北地区的销售情况如何，以及在飓风期间，大西洋沿岸的销售情况如何。

此外，没有人运行选择*查询来获取事实表中的所有行。相反，他们总是指定一个集合，从表中的100条属性中检索出半打属性。下一个查询会检索到不同的属性集，而且在过滤条件中几乎没有定位。

在这个用例中，很明显，列存储从磁盘到主内存的数据量比行存储要少16倍（6列对100）。因此，它有一个不公平的优势。此外，考虑一个存储块。在列存储中，该块上只有一个属性，而行存储将有100个。压缩在一个属性上显然比在100个属性上效果更好。此外，行存储在每条记录的前面都有一个头（在SQLServer中，它显然是16字节）。相比之下，列式存储则注意没有这样的头。

最后，基于行的执行器有一个内循环，记录在输出中被检查是否有效。因此，内循环的开销是相当大的，是为每条检查的记录支付的。相比之下，列式存储的基本操作是检索一个列并挑选出合格的项目。因此，内循环的开销是每检查一列支付一次，而不是每检查一行支付一次。因此，列执行器在CPU时间上更有效率，从磁盘上检索的数据也更少。在大多数实际环境中，列存储比行存储快50-100倍。

早期的列存储包括20世纪90年代出现的Sybase IQ [5]，以及MonetDB [2]。然而，该技术可以追溯到20世纪70年代[1，4]。在2000年，C-Store/Vertica作为资金雄厚的初创公司出现，并有高性能的实现。在接下来的十年里，整个数据仓库市场从行存储的世界演变成了列存储的世界。可以说，如果Sybase在技术上有更积极的投资并进行多节点的实施，那么Sybase IQ也可以更早地完成同样的事情。列执行器的优点在[2]中得到了令人信服的讨论，尽管它是 "在杂草中"，难以阅读。

第二个重大变化是主存储器价格的急剧下降。目前，人们可以用25,000美元买到1TB的内存，而用几TB的高性能计算集群也许可以用10万美元买到。关键的见解是，OLTP数据库并没有那么大。一兆字节是一个非常大的OLTP数据库，并且是主内存部署的候选者。正如本节中的望远镜论文所指出的，当数据适合于主内存时，人们并不想运行基于磁盘的行存储--开销太高了。

实际上，OLTP市场现在正成为一个主内存DBMS市场。同样，传统的基于磁盘的行存储是没有竞争力的。为了工作得好，需要为并发控制、崩溃恢复和多线程提供新的解决方案，我预计OLTP架构将在未来几年内不断发展。

我目前的最佳猜测是，没有人会使用传统的两相锁。基于时间戳排序或多版本的技术可能会占上风。本节中的第三篇论文讨论了Hekaton，它实现了一种最先进的锁。

## References:

[1] Batory, D.S. On searching transposed files. *ACM Transactions on Database Systems (TODS)*. 4, 4 (Dec. 1979).

[2] Boncz, P.A., Zukowski, M. and Nes, N. MonetDB/X100: Hyper-pipelining query execution. *CIDR*, 2005.

[3] Kimball, R. and Ross, M. The data warehouse toolkit: The complete guide to dimensional modeling. John Wiley & Sons. 2011.

[4] Lorie, R. and Symonds, A. A relational access method for interactive applications. *Courant Computer Science Symposia, Vol. 6: Data Base Systems*. (1971).

[5] MacNicol, R. and French, B. Sybase iQ multiplex-designed for analytics. *VLDB*, 2004.

[6] Malviya, N., Weisberg, A., Madden, S. and Stonebraker, M. Rethinking main memory OLTP recovery. *ICDE*, 2014.