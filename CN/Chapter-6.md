# Chapter 6: Weak Isolation and Distribution

## Introduced by Peter Bailis

> Readings:
>
> [Atul Adya, Barbara Liskov, and Patrick O'Neil. Generalized Isolation Level Definitions. *ICDE*, 2000.](https://scholar.google.com/scholar?cluster=12975897967422539576)
>
> [Giuseppe DeCandia, Deniz Hastorun, Madan Jampani, Gunavardhan Kakulapati, Avinash Lakshman, Alex Pilchin, Swaminathan Sivasubramanian, Peter Vosshall, and Werner Vogels. Dynamo: Amazon's Highly Available Key-Value Store. *SOSP*, 2007.](https://scholar.google.com/scholar?cluster=5432858092023181552)
>
> [Eric Brewer. CAP Twelve Years Later: How the "Rules" Have Changed. *IEEE Computer*, 45, 2 (2012).](https://scholar.google.com/scholar?cluster=17642052422667212790)

传统的数据库经验表明可串行化事务（serializable transactions）是并发编程问题的标准答案，但在现实世界的数据库中很少出现这种情况。在实践中，数据库系统反而压倒性地实现了不可序列化的并发控制，这让用户有可能他们的事务似乎不会以某种串行顺序执行。在本章中，我们将讨论为什么这种所谓的“弱隔离”的使用如此广泛，这些不可序列化的隔离模式实际上做了什么，以及为什么对它们进行推理如此困难。

## Overview and Prevalence（概况和流行情况）

即使是在数据库系统的最早期（见第3章），系统建设者也意识到，实现可序列化是很昂贵的。要求事务看起来是按顺序执行的，这对数据库所能达到的并发程度有深刻的影响。如果事务访问数据库中不相干的数据项集，可序列化实际上是 "免费 "的：在这些不相干的访问模式下，可序列化的时间表承认数据并行。然而，如果事务在相同的数据项上竞争，在最坏的情况下，系统不能以任何的并行性来处理它们。这个属性是可序列化的基础，与实际的实现无关：因为在所有的工作负载下，事务不能安全地独立取得进展（也就是说，它们必须协调），任何可序列化的实现实际上都可能要求串行执行。在实践中，这意味着事务可能需要等待，在增加延迟的同时降低了吞吐量。事务处理专家Phil Bernstein认为，在单节点数据库上，与最常见的弱隔离级别（称为 "读承诺"）相比，可序列化通常会产生三倍的性能损失[12]。根据不同的实现，可序列化还可能导致更多的中止、重启事务和/或死锁。在分布式数据库中，这些成本会增加，因为网络通信是昂贵的，增加了执行串行关键部分（例如，持有锁）所需的时间；我们已经观察到在不利条件下的多个数量级的性能惩罚[7]。

因此，数据库系统设计者没有实现可序列化，而是经常实现较弱的模型。在弱隔离下，事务不能保证观察到可序列化的行为。相反，事务会观察到一系列的异常现象（或 "现象"）：在串行执行中不可能出现的行为。确切的异常现象取决于所提供的模型，但是异常现象的例子包括读取另一个事务产生的中间数据、读取中止的数据、在同一事务的执行过程中为同一项目读取两个或多个不同的值，以及由于对同一项目的并发写入而 "丢失 "一些事务的效果。

这些弱隔离模式出人意料地普遍存在。在最近对18个SQL和 "NewSQL "数据库的调查中[5]，我们发现18个数据库中只有3个默认提供了可序列化，8个（包括Oracle和SAP的旗舰产品）根本不提供可序列化 这种状况因术语的使用往往不准确而进一步复杂化：例如，Oracle的 "可序列化 "隔离保证实际上提供了快照隔离，一种弱隔离模式[14]。供应商之间也存在着竞争。据传闻，当交易处理市场上的主要厂商A将其默认的隔离模式从可序列化转换为读承诺时，仍然默认为可序列化的厂商B开始在与厂商A的比试中失去销售合同。厂商B的数据库显然更慢，那么为什么客户会选择B而不是A呢？不出所料，供应商B现在也默认提供了读承诺隔离。

## The Key Challenge: Reasoning about Anomalies

弱隔离存在问题的主要原因是，根据应用的不同，弱隔离的异常情况会导致应用层面的不一致：每个事务在可序列化执行中保留的不变性在弱隔离下可能不再成立。例如，如果两个用户试图同时从一个银行账户中提款，而他们的交易在允许并发写入同一数据项的弱隔离模式下运行（例如，常见的读承诺模式），用户可能会成功地提取比账户中包含的更多的钱（即，每个人读取当前的金额，每个人计算他们提款后的金额，然后每个人将 "新 "总数写入数据库中）。这并不是一个假设的场景。在最近一个丰富多彩的例子中，一个攻击者系统地利用了Flexcoin比特币交易所的弱隔离行为；通过反复和程序性地触发Flexcoin应用程序中的非交易性的读-修改-写行为（在读承诺隔离下的漏洞，以及在更复杂的访问模式下的快照隔离），攻击者能够提取比她应该提取的更多的比特币，从而使交易所破产[15]。

也许令人惊讶的是，与我交谈过的关于使用事务的开发者中，很少有人意识到他们是在不可序列化的隔离下运行的。事实上，在我们的研究中，我们发现许多开源的ORM支持的应用程序假设是可序列化的隔离，当部署在商品数据库引擎上时，会导致一系列可能的应用程序完整性违规[6]。意识到弱隔离的开发者倾向于在应用层面采用一系列替代技术，包括明确获取锁（例如，SQL "SELECT FOR UPDATE"）和引入虚假冲突（例如，在快照隔离下写入一个假键）。这是很容易出错的，而且否定了事务概念的许多好处。

不幸的是，弱隔离的规范往往是不完整的、含糊的，甚至是不准确的。这些规范有很长的历史，可以追溯到1970年代。虽然随着时间的推移，它们已经有所改进，但仍然存在问题。

最早的弱隔离模式是在操作上指定的：正如我们在第3章中看到的，像 "已读承诺 "这样的流行模式最初是通过修改读锁的持续时间而发明的[17]。读取承诺的定义是。"在短时间内保持读锁，在长时间内保持写锁。"

后来，ANSI SQL标准试图为几种弱隔离模式提供与实现无关的描述，这些模式不仅适用于基于锁的机制，而且也适用于多版本和乐观的方法。然而，正如Gray等人在[10]中所描述的那样，SQL标准既含糊不清，又不够规范：对英文描述有多种可能的解释，而且形式化并没有捕捉到基于锁的实现的所有行为。此外，ANSI SQL标准并没有涵盖所有的隔离模式：例如，在Gray等人在1995年的论文中定义它之前，供应商已经开始提供生产数据库的快照隔离（并将其标记为可序列化！）。(遗憾的是，截至2015年，ANSI SQL标准仍然没有改变）。)

使问题进一步复杂化的是，Gray等人1995年修订的形式主义也有问题：它侧重于与锁有关的语义，排除了在多版本并发控制系统中可能被认为是安全的行为。因此，在他1999年的博士论文[1]中，Atul Adya介绍了迄今为止我们拥有的最好的弱隔离形式主义。Adya的论文将多版本序列化图[11]的形式主义适应于弱隔离领域，并根据对这些图的限制来描述异常情况。我们包括Adya相应的ICDE 2000论文，但隔离爱好者应该查阅论文全文。不幸的是，Adya的模型在某些情况下仍然没有得到充分的说明（例如，如果不涉及读，G0到底是什么意思？），而且这些保证的实现在不同的数据库中是不同的。

即使有一个完美的规范，弱隔离仍然是一个真正的推理挑战。为了决定弱隔离是否 "安全"，程序员必须在头脑中把他们应用层面的一致性问题转化为低级别的读写行为[2]。即使对于经验丰富的并发控制专家来说，这也是非常困难的。事实上，人们可能会想，如果可序列化被破坏了，事务还有什么好处？为什么推理 "已读承诺 "隔离比没有隔离更容易？鉴于像Oracle这样的许多数据库引擎是在弱隔离下运行的，那么现代社会是如何运作的呢--无论用户是在预订机票、管理医院，还是在进行股票交易？文献中没有提供什么线索，对今天实践中部署的事务概念的成功提出了严重的质疑。

对于为什么弱隔离在实践中似乎是 "好的"，我遇到的最有说服力的论据是，今天很少有应用程序经历高度的并发性。在没有并发性的情况下，大多数弱隔离的实现都能提供可序列化的结果。这反过来又导致了一系列富有成效的研究结果。即使在分布式环境中，弱隔离数据库也能提供 "一致的 "结果：例如，在Facebook，从其最终一致的存储中返回的结果只有0.0004%是 "过时的"[19]，其他人也发现了类似的结果[9, 25]。然而，虽然对于许多应用程序来说，弱隔离显然不是问题，但它可能是：正如我们的Flexcoin例子所说明的那样，考虑到错误的可能性，应用程序的编写者必须警惕地考虑（或明确地忽略）与并发性有关的异常情况。

## Weak Isolation, Distribution, and “NoSQL”

随着互联网规模的服务和云计算的兴起，弱隔离已经变得更加普遍了。正如我前面提到的，分布式加剧了可序列化的开销，而且，在部分系统故障的情况下（例如服务器崩溃），事务可能会无限期地停滞。随着越来越多的程序员开始编写分布式应用和使用分布式数据库，这些问题成为主流。

在过去的十年中，出现了一系列为分布式环境优化的新数据存储，统称为 "NoSQL"。不幸的是，"NoSQL "这个标签已经超载了，它指的是这些存储的许多方面，从缺乏字面的SQL支持到更简单的数据模型（例如，键值对）以及几乎没有交易支持。今天，正如类似MapReduce的系统（第5章），NoSQL存储正在增加许多这些功能。然而，一个值得注意的根本区别是，这些NoSQL存储经常专注于通过较弱的模型提供更好的操作可用性，并明确关注容错。(有点讽刺的是，虽然NoSQL存储通常与使用不可序列化的保证有关，但经典的RDBMS默认也不提供可序列化的功能。)

作为这些NoSQL存储的一个例子，我们包括一篇关于Dynamo系统的论文，该系统来自亚马逊，在2007年SOSP上发表。Dynamo的推出是为了给亚马逊的购物车提供高可用和低延迟的操作。这篇论文在技术上很有趣，因为它结合了几种技术，包括法定人数复制、Merkle树反熵、一致散列和版本向量。该系统完全是非交易性的，不提供任何种类的原子操作（例如，比较和交换），并依靠应用程序编写者来协调不同的更新。在极限情况下，任何节点都可以更新任何项目（在暗示的交接下）。

通过使用合并函数，Dynamo采用了一种 "乐观的复制 "策略：先接受写入，以后再调和不同的版本[16, 21]。一方面，向用户展示一组有分歧的版本比简单地丢弃一些并发的更新更友好，就像在读承诺隔离中那样。另一方面，程序员必须对合并函数进行推理。这就提出了许多问题：对于一个应用程序来说，什么是合适的合并？我们如何避免丢弃已提交的数据？如果一个操作一开始就不应该被并发执行怎么办？一些开源的Dynamo克隆，如Apache Cassandra，不提供合并运算符，只是根据数字时间戳来选择一个 "获胜 "的写入。其他的，像Basho Riak，已经采用了像计数器一样的可自动合并的数据类型的 "库"，称为Commutative Replicated Data Types [22] 。

Dynamo也没有对读取的频率做出承诺。相反，它保证，如果停止写入，最终一个数据项的所有副本将包含相同的写入集合。这种最终一致性是一个非常弱的保证：从技术上讲，一个最终一致的数据存储可以在不确定的时间内返回陈旧的（甚至是垃圾）数据[4]。在实践中，数据存储的部署经常返回最近的数据[9, 25]，但是，尽管如此，用户必须对非序列化的行为进行推理。此外，在实践中，许多存储提供中间形式的隔离，称为 "会话保证"，确保用户读取他们自己的写（但不是其他用户的写）；有趣的是，这些技术是在20世纪90年代初作为移动计算的Bayou项目的一部分发展起来的，最近又开始崭露头角[23, 24]。

## Trade-offs and the CAP Theorem

我们还包括布鲁尔对CAP定理的12年回顾。布鲁尔的CAP定理最初是在布鲁尔建立Inktomi（最早的可扩展搜索引擎之一）之后提出的，它精辟地描述了协调性（或 "可用性"）要求与可序列化等强保障之间的折衷。虽然早期的结果描述了这种权衡[13，18]，但CAP成为了2000年中期开发者的集结号，并具有相当大的影响。Brewer的文章简要地讨论了CAP的性能影响，以及在不依赖协调的情况下保持一些一致性标准的可能性。

## Programmability and Practice

正如我们所看到的，弱隔离是一个真正的挑战：它的性能和可用性优势意味着它在部署中非常流行，尽管我们对它的行为了解甚少。即使有一个完美的规范，现有的弱隔离的表述仍然是一个极难推理的问题。为了决定弱隔离是否 "安全"，程序员必须在头脑中把他们应用层面的一致性问题转化为低级别的读写行为[2]。即使对经验丰富的并发控制专家来说，这也是非常困难的。

因此，我相信存在着研究语义的重大机会，这些语义不受可序列化的性能和可用性开销的影响，但比现有的保证更加直观、可用和可编程。从历史上看，弱隔离对于推理来说是非常具有挑战性的，但这并不一定是事实。我们和其他人已经发现，一些高价值的用例，包括索引和视图维护、约束维护和分布式聚合，实际上经常不需要协调 "正确 "的行为；因此，对于这些用例，可序列化是多余的[3，8，20，22]。也就是说，通过为数据库提供关于其应用的额外知识，数据库用户可以得到他们的蛋糕，并把它吃掉。进一步识别和利用这些用例是一个已经成熟的研究领域。

## Conclusions

综上所述，弱隔离之所以盛行，是因为它有许多好处：更少的协调、更高的性能和更大的可用性。然而，人们对它的语义、风险和用法了解甚少，甚至在学术背景下也是如此。考虑到对可序列化事务处理的大量研究，这一点尤其令人费解，许多人认为可序列化事务处理是一个 "已解决的问题"。可以说，弱隔离更应该得到如此彻底的处理。正如我所强调的，许多挑战仍然存在：现代系统甚至如何工作，以及用户应该如何对弱隔离进行编程？就目前而言，我提供了以下几点启示。

- 非序列化隔离在实践中非常普遍（在经典的RDBMS和最近的NoSQL新秀中），因为它具有与并发有关的好处。
- 尽管这种现象很普遍，但许多现有的非序列化隔离的表述都没有得到很好的说明，而且难以使用。
- 对新形式的弱隔离的研究表明，如何在不牺牲可序列化的前提下保留有意义的语义并提高可编程性。

## References:

[1] Adya, A. *Weak consistency: A generalized theory and optimistic implementations for distributed transactions*. Ph.D. Thesis, MIT, 1999

[2] Alvaro, P., Bailis, P., Conway, N. and Hellerstein, J.M. Consistency without borders. *SoCC*, 2013.

[3] Bailis, P. *Coordination avoidance in distributed databases*. Ph.D. Thesis, University of California at Berkeley, 2015.

[4] Bailis, P. and Ghodsi, A. Eventual consistency today: Limitations, extensions, and beyond. *ACM Queue*. 11, 3 (2013).

[5] Bailis, P., Davidson, A., Fekete, A., Ghodsi, A., Hellerstein, J.M. and Stoica, I. Highly Available Transactions: Virtues and limitations. *VLDB*, 2014.

[6] Bailis, P., Fekete, A., Franklin, M.J., Ghodsi, A., Hellerstein, J.M. and Stoica, I. Feral Concurrency Control: An empirical investigation of modern application integrity. *SIGMOD*, 2015.

[7] Bailis, P., Fekete, A., Franklin, M.J., Hellerstein, J.M., Ghodsi, A. and Stoica, I. Coordination avoidance in database systems. *VLDB*, 2015.

[8] Bailis, P., Fekete, A., Ghodsi, A., Hellerstein, J.M. and Stoica, I. Scalable atomic visibility with RAMP transactions. *SIGMOD*, 2014.

[9] Bailis, P., Venkataraman, S., Franklin, M.J., Hellerstein, J.M. and Stoica, I. Probabilistically Bounded Staleness for practical partial quorums. *VLDB*, 2012.

[10] Berenson, H., Bernstein, P., Gray, J., Melton, J., O’Neil, E. and O’Neil, P. A critique of ANSI SQL isolation levels. *SIGMOD*, 1995.

[11] Bernstein, P., Hadzilacos, V. and Goodman, N. Concurrency control and recovery in database systems. Addison-Wesley New York. 1987.

[12] Bernstein, P.A. and Das, S. Rethinking eventual consistency. *SIGMOD*, 2013.

[13] Davidson, S., Garcia-Molina, H. and Skeen, D. Consistency in partitioned networks. *ACM CSUR*. 17, 3 (1985), 341-370.

[14] Fekete, A., Liarokapis, D., O’Neil, E., O’Neil, P. and Shasha, D. Making snapshot isolation serializable. *ACM TODS*. 30, 2 (Jun. 2005), 492-528.

[15] Flexcoin: The Bitcoin Bank: Flexcoin: The Bitcoin Bank. 2014. http://www.flexcoin.com/; originally via Emin Gün Sirer.

[16] Gray, J., Helland, P., O’Neil, P. and Shasha, D. The dangers of replication and a solution. *SIGMOD*, 1996.

[17] Gray, J., Lorie, R., Putzolu, G. and Traiger, I. Granularity of locks and degrees of consistency in a shared data base. IBM. 1976.

[18] Johnson, P.R. and Thomas, R.H. RFC 667: The maintenance of duplicate databases. 1975.

[19] Lu, H., Veeraraghavan, K., Ajoux, P., Hunt, J., Song, Y.J., Tobagus, W., Kumar, S. and Lloyd, W. Existential consistency: Measuring and understanding consistency at Facebook. *SOSP*, 2015.

[20] Roy, S., Kot, L., Bender, G., Ding, B., Hojjat, H., Koch, C., Foster, N. and Gehrke, J. The homeostasis protocol: Avoiding transaction coordination through program analysis. *SIGMOD*, 2015.

[21] Saito, Y. and Shapiro, M. Optimistic replication. *ACM Comput. Surv.* 37, 1 (Mar. 2005).

[22] Shapiro, M., Preguica, N., Baquero, C. and Zawirski, M. Shapiro Marc. Preguica Nuno. Baquero Carlos. Zawirski Marek. A comprehensive study of convergent and commutative replicated data types. INRIA TR 7506. 2011.

[23] Terry, D. Replicated data consistency explained through baseball. *Communications of the ACM*. 56, 12 (2013), 82-89.

[24] Terry, D.B., Demers, A.J., Petersen, K., Spreitzer, M.J., Theimer, M.M. and others Session guarantees for weakly consistent replicated data. *PDIS*, 1994.

[25] Wada, H., Fekete, A., Zhao, L., Lee, K. and Liu, A. Data consistency properties and the trade-offs in commercial cloud storage: The consumers’ perspective. *CIDR*, 2011.