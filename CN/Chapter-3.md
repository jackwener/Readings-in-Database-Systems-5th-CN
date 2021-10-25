# Chapter 3: Techniques Everyone Should Know

## Introduced by Peter Bailis

> Readings:
>
> [Patricia G. Selinger, Morton M. Astrahan, Donald D. Chamberlin, Raymond A. Lorie, Thomas G. Price. Access path selection in a relational database management system. *SIGMOD*, 1979.](https://scholar.google.com/scholar?cluster=102545501597608314)
>
> [C. Mohan, Donald J. Haderle, Bruce G. Lindsay, Hamid Pirahesh, Peter M. Schwarz. ARIES: A Transaction Recovery Method Supporting Fine-Granularity Locking and Partial Rollbacks Using Write-Ahead Logging. *ACM Transactions on Database Systems*, 17(1), 1992, 94-162.](https://scholar.google.com/scholar?cluster=2142924814045750364)
>
> [Jim Gray, Raymond A. Lorie, Gianfranco R. Putzolu, Irving L. Traiger. Granularity of Locks and Degrees of Consistency in a Shared Data Base. , IBM, September, 1975.](https://scholar.google.com/scholar?cluster=15730220590995320737)
>
> [Rakesh Agrawal, Michael J. Carey, Miron Livny. Concurrency Control Performance Modeling: Alternatives and Implications. *ACM Transactions on Database Systems*, 12(4), 1987, 609-654.](https://scholar.google.com/scholar?cluster=9784855600346107276)
>
> [C. Mohan, Bruce G. Lindsay, Ron Obermarck. Transaction Management in the R* Distributed Database Management System. *ACM Transactions on Database Systems*, 11(4), 1986, 378-396.](https://scholar.google.com/scholar?cluster=6135007404184895390)

在这一章中，我们介绍了数据库系统设计中几个最重要的核心概念(查询计划、并发控制、数据库恢复和分布式)的主要/接近主要的来源。本章中的概念对于现代数据库系统来说是非常基本的，几乎每一个成熟数据库系统的实现都包含了这些概念。本章中的三篇论文是各自主题的经典参考文献。此外与前一章相比，本章的重点不是整个系统，而是广泛应用的技术和算法。

## Query Optimization

查询优化在关系型数据库的架构中非常重要，因为它是实现数据独立查询处理的核心。Selinger 等人关于 System R 的基础性论文通过将问题分解为三个不同的子问题来实现查询优化：成本估计（cost estimation）、定义搜索空间的等价关系表达式（relational equivalences that define a search space），基于成本的搜索（cost-based search）。

优化器提供了一个执行查询的每个组件的成本估计，以 I/O 和 CPU 成本来衡量。为了做到这一点，优化器既依赖关于每个关系内容的预先计算的统计数据（存储在系统目录中），也依赖一套启发式算法来确定查询输出的 cardinality（大小）（例如，基于估计的谓词选择性）。作为一项练习，详细考虑这些启发式方法：它们在什么时候有意义，在什么输入上会失败？如何改进它们？

使用这些成本估计，优化器使用动态规划来构建一个查询计划。优化器定义了一组物理算子，以实现给定的逻辑算子（例如，使用完整的 "分段 "扫描与索引来查找元组）。使用这组运算符，优化器迭代地构建了一棵 "左深 "运算符树，反过来使用成本启发式方法来最小化运行运算符所需的估计工作总量，同时考虑到上游消费者所需的 "interesting orders"。这避免了考虑所有可能的运算符排序，但仍然是计划大小的指数级；正如我们在第7章中所讨论的，现代查询优化器在处理大型计划（例如，多向连接）时仍然很困难。此外，虽然Selinger等人的优化器提前进行了编译，但其他早期的系统，如Ingres[25]解释了查询计划--实际上，是在逐个元组的基础上。

像几乎所有的查询优化器一样，Selinger 等人的优化器实际上不是最优的--不能保证优化器选择的计划是最快或成本最低的。关系优化器在精神上更接近于现代语言编译器中的代码优化程序（即，将执行最佳努力搜索），而不是数学优化程序（即，将找到最佳解决方案）。然而，今天的许多关系型引擎都采用了本文的基本方法，包括使用两阶段算子（binary operators）和成本估算。

## Concurrency Control

我们关于事务的第一篇论文，来自 Gray 等人，介绍了两个经典的想法：多粒度锁和多锁模式。这篇论文实际上是两篇独立的论文。

首先，这篇论文提出了多粒度锁的概念。这里的问题很简单：给定一个具有分层结构的数据库，我们应该如何执行互斥？什么时候我们应该在粗粒度（如整个数据库）与细粒度（如单个记录）之间进行锁定，以及我们如何支持同时对层次结构的不同部分进行并发访问？虽然Gray等人的分层布局（由数据库、区域、文件、索引和记录组成）与现代数据库系统的布局略有不同，但除了最初级的数据库锁系统外，今天所有的数据库都适应他们的建议。

第二，本文提出了多度隔离的概念。正如Gray等人提醒我们的那样，并发控制的一个目标是保持数据的 "一致性"，即它服从一些逻辑断言。传统上，数据库系统使用可序列化的事务作为执行一致性的手段：如果单个事务都让数据库处于 "一致 "状态，那么可序列化的执行（相当于事务的某些序列执行）将保证所有事务都观察到数据库的 "一致 "状态[5]。Gray等人的 "程度3 "协议描述了经典的（严格的）"两阶段锁定"（2PL），它保证了可序列化的执行，是事务处理的一个主要概念。

然而，可序列化通常被认为是太昂贵了，无法执行。为了提高性能，数据库系统经常使用不可序列化的隔离来执行事务。在本文中，持有锁是很昂贵的：在发生冲突的情况下等待锁需要时间，而在发生死锁的情况下，可能需要永远的等待（或者导致中止）。因此，早在1973年，IMS和System R等数据库系统就开始尝试使用非可序列化策略。在一个基于锁的并发控制系统中，这些策略是通过在较短的时间内保持锁来实现的。这允许更大的并发性，可能导致更少的死锁和系统引起的中止，并且在分布式环境中，可能允许更大的操作可用性。
在本文的后半部分，Gray等人对这些基于锁的策略的行为进行了初步的形式化。今天，它们已经很普遍了；正如我们在第6章中所讨论的，在大多数商业和开源RDBMS中，不可序列化的隔离是默认的，而且一些RDBMS根本不提供序列化。程度2现在通常被称为可重复读隔离，程度1现在被称为读承诺隔离，而程度0则不常被使用[1]。本文还讨论了重要的可恢复性概念：在这种政策下，一个事务可以被中止（或 "撤销"）而不影响其他事务。除了Degree 0交易外，所有的交易都满足这个属性。

在Gray等人关于基于锁的可序列化的开创性工作之后，出现了广泛的替代性并发控制机制。随着硬件、应用需求和访问模式的变化，并发控制子系统也发生了变化。然而，并发控制的一个属性仍然是近乎肯定的：在并发控制中没有单方面的 "最佳 "机制。最佳策略是取决于工作负载的。为了说明这一点，我们包括了Agrawal、Carey和Livny的研究。虽然过时了，但这篇论文的方法和广泛的结论仍然是有针对性的。这是一个深思熟虑的、与实施无关的性能分析工作的好例子，可以随着时间的推移提供宝贵的经验。

从方法论上讲，进行所谓的 "信封式 "计算的能力是一种有价值的技能：用粗略的算术快速估计一个感兴趣的指标，得出一个与正确值相差一个数量级的答案，可以节省数小时甚至数年的系统实施和性能分析。这在数据库系统中是一个长期而有用的传统，从 "五分钟规则"[12]到谷歌的 "每个人都应该知道的数字"[4]。虽然从这些估计中得出的一些教训是短暂的[8, 10]，但往往这些结论提供了长期的教训。

然而，对于复杂系统的分析，如并发控制，模拟可以是一个有价值的中间步骤，介于包络法和全面的系统基准测试。Agrawal的研究是这种方法的一个例子：作者使用一个精心设计的系统和用户模型来模拟锁定、基于重启和乐观的并发控制。

该评估的几个方面特别有价值。首先，几乎每张图中都有一个 "交叉点"：没有明显的赢家，因为表现最好的机制取决于工作负载和系统配置。相反，几乎所有没有交叉点的性能研究都可能是无趣的。如果一个方案 "总是赢"，研究应该包含一个分析性的分析，或者，最好是证明为什么会这样。第二，作者考虑了广泛的系统配置；他们调查并讨论了其模型的几乎所有参数。第三，许多图形表现出非单调性（即不总是向上和向右走）；这是惊动和资源限制的产物。正如作者所说明的，假设资源是无限的，就会得出截然不同的结论。一个不那么仔细的模型，如果隐含了这个假设，那么它的作用就会小得多。

最后，该研究的结论是合理的。基于重启的方法的主要成本是在发生冲突时 "浪费 "了工作。当资源充足时，投机是有意义的：浪费的工作成本较低，而且，在资源无限的情况下，它是免费的。然而，在资源比较有限的情况下，阻断策略会消耗更少的资源，并提供更好的整体性能。同样，也不存在单方面的最优选择。然而，该论文的结论被证明是有先见之明的：计算资源仍然是稀缺的，事实上，今天很少有商品系统采用完全基于重启的方法。然而，随着技术比例--磁盘、网络、CPU速度--的不断变化，重新审视这种权衡是有价值的。

## Database Recovery

事务处理的另一个主要问题是保持持久性：事务处理的效果应该在系统故障后仍然存在。一种近乎普遍的保持耐久性的技术是进行日志记录：在交易执行期间，交易操作被存储在容错介质（如硬盘或SSD）的日志中。每个在数据系统工作的人都应该了解写前日志的工作原理，最好是了解一些细节。

实现基于 WAL 的恢复管理器的经典算法是 IBM 的 ARIES 算法，这是我们下一篇论文的主题。(资深的数据库研究人员可能会告诉你，非常类似的想法是在同一时间在Tandem和Oracle等地方发明的)。在 ARIES 中，数据库不需要在提交时将脏页写入磁盘（"No Force"），而数据库可以在任何时候将脏页刷入磁盘（"Steal"）[15]；这些策略可以实现高性能，几乎所有的商业 RDBMS 产品中都有这些策略，但反过来也增加了数据库的复杂性。ARIES的基本思想是分三个阶段进行崩溃恢复。首先，ARIES执行一个分析阶段，通过重放日志，以确定在崩溃时哪些交易正在进行。第二，ARIES 执行重做阶段，（再次）重放日志，（这次）执行在崩溃时正在进行的任何事务的效果。第三，ARIES 通过反向播放日志和撤销未提交事务的影响来执行撤销阶段。因此，ARIES的关键思想是 "重复历史 "来执行恢复；事实上，撤销阶段可以执行与正常操作中用于中止交易的相同逻辑。

ARIES 应该是一篇相当简单的论文，但它可能是本论文集中最复杂的论文。在研究生数据库课程中，这篇论文是一种仪式。然而，这个材料是基本的，所以理解它是很重要的。幸运的是，Ramakrishnan 和Gehrke的本科生教材[22]和Michael Franklin的调查论文[7]分别提供了一个比较温和的处理方法。我们在这里收录的ARIES论文全文，由于其对沿途替代设计决定的缺点的转移性讨论而变得非常复杂。首先，我们鼓励读者忽略这些材料，只关注 ARIES 方法。替代方案的缺点很重要，但应该留到更仔细的第二或第三遍阅读。除了它的组织结构之外，ARIES 协议的讨论还因为对管理内部状态的讨论而变得更加复杂，比如索引（即嵌套的顶部操作和逻辑撤销记录--后者也被用于像 Escrow 事务[21]这样的奇特方案）和在恢复期间尽量减少停机时间的技术。在实践中，重要的是恢复时间看起来越短越好；这一点实现起来很棘手。

## Distribution

本章的最后一篇论文涉及分布式环境中的事务执行。这个话题在今天尤为重要，因为越来越多的数据库是分布式的--要么是复制的，在不同的服务器上有多个数据副本，要么是分区的，数据存储在不相干的（disjoint）服务器上（或者两者都是）。尽管分布式数据库有益于容量（capacity）、持久性（durability）和可用性（availability）等方面，但也带来了一系列新的问题。服务器可能发生故障，网络连接可能不可靠。在没有故障的情况下，网络通信可能也成本高昂。

我们专注于分布式交易处理的核心技术之一：原子保证（atomic commitment，AC）。非正式地讲，给定一个在多个服务器上执行的事务（无论是多个副本、多个分区，还是两者），AC确保该事务在所有服务器上提交或中止。实现 AC 的经典算法可以追溯到20世纪70年代中期，被称为两阶段提交（2PC；不要与上面的2PL混淆！）[9, 18]。除了对 2PC 以及提交协议和 WAL 之间的相互作用提供了一个很好的概述，本文还包含了 AC 的两个变体，以提高其性能。推定中止变体允许进程避免将中止决定强加给磁盘或确认中止，从而减少磁盘利用率和网络流量。推定提交的优化也是类似的，当更多的事务提交时，优化了空间和网络流量。注意 2PC 协议、本地存储和本地事务管理器之间相互作用的复杂性；细节很重要，正确实现这些协议可能具有挑战性。

失败的可能性使 AC （以及分布式计算中的大多数问题）大大复杂化。例如，在 2PC 中，如果一个协调者和参与者在所有参与者都发送了他们的投票之后，但在协调者听到失败的参与者的消息之前都失败了，会发生什么？剩下的参与者将不知道是提交还是中止交易：失败的参与者是投了 "是 "还是 "否"？参与者无法安全地继续下去。事实上，当在一个不可靠的网络上运行时，任何 AC 的实现都可能会阻塞，或无法取得进展[2]。再加上可序列化的并发控制机制，阻塞的交流意味着吞吐量可能会停滞。因此，一组相关的交流算法在关于网络（例如，通过假设同步网络）[24]和服务器可用信息（例如，通过利用确定节点故障的 "故障检测器"）[14]的宽松假设下研究交流。

最后，许多读者可能熟悉与之密切相关的共识问题，或者听说过诸如 Paxos 算法这样的共识实现方式。在共识中，任何提议都可以被选择，只要所有进程最终都会同意它。(相比之下，在 AC 中，任何单个参与者都可以投反对票，之后所有参与者都必须放弃）。) 这使得共识成为一个比 AC 更 "容易 "的问题[13]，但是，和 AC 一样，任何共识的实现在某些情况下也会受阻[6]。在现代分布式数据库中，共识经常被用作复制的基础，以确保复制体以相同的顺序应用更新，这是状态机复制的一个实例（见 Schneider 的教程[23]）。AC 经常被用来执行跨越多个分区的事务。Lamport 的 Paxos [17]是最早的（也是最有名的，部分原因是它的演示在复杂性上可以与 ARIES 相媲美）共识的实现。然而，Viewstamped Replication[19] 和 Raft[20]、ZAB[16] 和 Multi-Paxos[3] 算法在实践中可能更有帮助。这是因为这些算法实现了一个分布式日志抽象（而不是像Paxos原始论文中的 "共识对象"）。

不幸的是，数据库和分布式计算社区在某种程度上是分开的。尽管在复制数据方面有共同的兴趣，但多年来两者之间的思想交流是有限的。在云和互联网规模的数据管理时代，这种差距已经缩小了。例如，Gray 和 Lamport 在2006年合作开发了 Paxos Commit[11]，这是一个结合了 AC 和 Lamport 的 Paxos 的有趣算法。在这个交叉领域还有很多工作要做，这个领域的 "每个人都应该知道的技术 "的数量也在增加。

## References:

[1] Berenson, H., Bernstein, P., Gray, J., Melton, J., O’Neil, E. and O’Neil, P. A critique of ANSI SQL isolation levels. *SIGMOD*, 1995.

[2] Bernstein, P., Hadzilacos, V. and Goodman, N. Concurrency control and recovery in database systems. Addison-Wesley New York. 1987.

[3] Chandra, T.D., Griesemer, R. and Redstone, J. Paxos made live: An engineering perspective. *PODC*, 2007.

[4] Dean, J. Designs, lessons and advice from building large distributed systems (keynote). *LADIS*, 2009.

[5] Eswaran, K.P., Gray, J.N., Lorie, R.A. and Traiger, I.L. The notions of consistency and predicate locks in a database system. *Communications of the ACM*. 19, 11 (1976), 624-633.

[6] Fischer, M.J., Lynch, N.A. and Paterson, M.S. Impossibility of distributed consensus with one faulty process. *Journal of the ACM (JACM)*. 32, 2 (1985), 374-382.

[7] Franklin, M.J. Concurrency control and recovery. *The Computer Science and Engineering Handbook*. (1997), 1-58-1077.

[8] Graefe, G. The five-minute rule twenty years later, and how flash memory changes the rules. *DaMoN*, 2007.

[9] Gray, J. Notes on data base operating systems. *Operating systems: An advanced course*. Springer Berlin Heidelberg. 393-481.

[10] Gray, J. and Graefe, G. The five-minute rule ten years later, and other computer storage rules of thumb. *ACM SIGMOD Record*. 26, 4 (1997), 63-68.

[11] Gray, J. and Lamport, L. Consensus on transaction commit. *ACM Transactions on Database Systems (TODS)*. 31, 1 (Mar. 2006), 133-160.

[12] Gray, J. and Putzolu, F. The 5 minute rule for trading memory for disc accesses and the 10 byte rule for trading memory for cPU time. *SIGMOD*, 1987.

[13] Guerraoui, R. Revisiting the relationship between non-blocking atomic commitment and consensus. *WDAG*, 1995.

[14] Guerraoui, R., Larrea, M. and Schiper, A. Non blocking atomic commitment with an unreliable failure detector. *SRDS*, 1995.

[15] Haerder, T. and Reuter, A. Principles of transaction-oriented database recovery. *ACM Computing Surveys (CSUR)*. 15, 4 (1983), 287-317.

[16] Junqueira, F.P., Reed, B.C. and Serafini, M. Zab: High-performance broadcast for primary-backup systems. *DSN*, 2011.

[17] Lamport, L. The part-time parliament. *ACM Transactions on Computer Systems (TOCS)*. 16, 2 (1998), 133-169.

[18] Lampson, B. and Sturgis, H. Crash recovery in a distributed data storage system. Xerox PARC Technical Report. 1979.

[19] Liskov, B. and Cowling, J. Viewstamped replication revisited. MIT; MIT-CSAIL-TR-2012-021. 2012.

[20] Ongaro, D. and Ousterhout, J. In search of an understandable consensus algorithm. *USENIX ATC*, 2014.

[21] O’Neil, P.E. The escrow transactional method. *ACM Transactions on Database Systems*. 11, 4 (1986), 405-430.

[22] Ramakrishnan, R. and Gehrke, J. Database management systems. McGraw Hill. 2000.

[23] Schneider, F.B. Implementing fault-tolerant services using the state machine approach: A tutorial. *ACM Computing Surveys (CSUR)*. 22, 4 (1990), 299-319.

[24] Skeen, D. Nonblocking commit protocols. *SIGMOD*, 1981.

[25] Stonebraker, M., Held, G., Wong, E. and Kreps, P. The design and implementation of iNGRES. *ACM Transactions on Database Systems (TODS)*. 1, 3 (1976), 189-222.
