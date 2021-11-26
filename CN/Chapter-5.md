# Chapter 5: Large-Scale Dataflow Engines

## Introduced by Peter Bailis

> Readings:
>
> [Jeff Dean and Sanjay Ghemawat. MapReduce: Simplified Data Processing on Large Clusters. *OSDI*, 2004.](https://scholar.google.com/scholar?cluster=10940266603640308767)
>
> [Yuan Yu, Michael Isard, Dennis Fetterly, Mihai Budiu. DryadLINQ: A System for General-Purpose Distributed Data-Parallel Computing Using a High-Level Language. *OSDI*, 2008.](https://scholar.google.com/scholar?cluster=3662601067977846800)

在过去十年数据管理的众多发展中，MapReduce 和随后的大规模数据处理系统是最具颠覆性和最具争议性的。廉价的商业存储和不断增长的数据量使许多互联网服务厂商放弃了传统的数据库系统和数据仓库，转而建立定制的、自制的引擎。谷歌关于其大规模系统的一连串 paper ，包括 GFS[11]、MapReduce、Chubby[6]和BigTable[7] (Google 老三篇)，可能是该领域上最著名和最具影响力的。这些新的自制的系统基本上都实现了传统数据库中的一小部分功能，包括高级语言、查询优化器和高效的执行器。这些系统和由此产生的开源 Hadoop 生态系统非常受开发者欢迎。这导致了对这些平台的大量投资、营销、研究兴趣和开发，今天，这些平台正处于变化之中，但是，作为一个生态系统，已经开始类似于传统的数据仓库--有了一些重要的修改。我们在此对这些趋势进行反思。

## History and Successors

第一篇论文是 Google 2014 年提出的 MapReduce 论文。MapReduce 是用于简化谷歌规模的分布式数据的并行、分布式计算的一个库，特别是用于从抓取的页面中批量重建网络搜索索引。在当时，传统的数据仓库不太可能处理这种工作量。然而，与传统的数据仓库相比，MapReduce 提供了很底层的接口（两阶段数据流），与容错的执行策略（两阶段数据流之间的中间物化）紧密相连。同样重要的是，MapReduce被设计成一个用于并行编程的库，而不是一个端到端的数据仓库解决方案；例如，MapReduce将存储委托给 GFS。当时，数据库领域认为该结构过于简单，低效且用途有限[8]。

虽然最初的 MapReduce 论文是在 2003 年发布的，但在谷歌之外的影响较少，直到 2006 年雅虎开源 Hadoop 这一 MapReduce 实现。随后，人们的兴趣大增：在一年之内，包括Dryad（微软）[15]、Hive（Facebook）[26]、Pig（雅虎）[22]等一系列项目都在开发之中。这些系统，我们称之为后 MapReduce 系统，吸引了大量硅谷的开发者，同时也获得了大量的风投。许多研究跨越了系统、数据库和网络，研究的问题包括调度、减少掉队、实现容错、UDF查询优化和替代编程模型[5]。

几乎在同一时间，后MapReduce系统扩展了其接口和功能，包括更复杂的声明性接口、查询优化策略和高效的运行时。今天的后 MapReduce 系统已经覆盖了传统 RDBMS 的越来越多的特性。最新一代的数据处理引擎，如Spark[27]、F1[24]、Impala[16]、Tez[3]、Naiad[21]、Flink/Stratosphere[1]、AsterixDB[2]和Drill[14]经常i）有更高级的查询语言，如SQL，ii）更高级的执行策略，包括处理一般图的运算符的能力，以及iii）尽可能使用结构化输入数据源的索引和其他功能。在Hadoop生态系统中，数据流引擎已经成为一套更高级别的功能和声明性接口的底层，包括SQL[4, 26]、图处理[13, 19]和机器学习[12, 25]。人们对流处理功能的兴趣也越来越大，重新审视2000年代在数据库社区开创的许多概念。一个不断增长的商业和开源生态系统已经开发了与各种结构化和半结构化数据源的 "连接器"、目录功能（如HCatalog）以及数据服务和有限的交易能力（如HBase）。这些功能中的大部分，例如这些框架中的典型查询优化器，与许多成熟的商业数据库相比是很初级的，但正在迅速发展。

DryadLING 是本节的第二篇论文，它的接口也许是最有趣的：一组用于数据处理的嵌入式语言，和微软的.NET LINQ深度绑定，提供一个并行集合库。DryadLINQ通过早期的Dryad系统[15]执行查询，该系统使用基于重放的容错实现了任意数据流图的运行时间。虽然DryadLINQ仍然限制程序员进行一系列无副作用的数据集转换（包括 "类似SQL "的操作），但它提供了一个比Map Reduce高得多的接口。DryadLINQ的语言集成、轻量级容错和基本的查询优化技术被证明对后来的数据流系统很有影响，包括Apache Spark[27]和微软的Naiad[21]。

## Impact and Legacy

MapReduce现象至少有三个持久的影响，否则可能不会发生。这些想法--就像分布式数据流本身--不一定是新颖的，但后MapReduce数据流和存储系统的生态系统广泛地增加了它们的影响。

1. 模式的灵活性。

也许最重要的是，传统的数据仓库系统是有围墙的花园：摄入的数据是原始的，经过策划的，并且有结构。相比之下，MapReduce系统可以处理任意结构的数据，不管是干净的还是脏的，不管是否经过策划。没有加载步骤。这意味着用户可以先存储数据，然后再考虑如何处理它。再加上存储（如在Hadoop文件系统中）比传统的数据仓库要便宜得多，用户可以负担得起保留数据的时间越来越长。这是对传统数据仓库的一个重大转变，也是 "大数据 "兴起和聚集的一个关键因素。越来越多的存储格式（例如，Avro、Parquet、RCFile）将半结构化数据和列式布局等存储方面的进展结合起来。与XML相比，这一最新的半结构化数据浪潮甚至更加灵活。因此，提取-转换-加载（ETL）任务是后MapReduce引擎的主要工作负荷。从分析师到程序员和分析厂商，模式的灵活性对现代数据管理实践的影响怎么强调都不为过，而且我们相信它在未来会变得更加重要。然而，这种异质性并不是免费的：策划这样的 "数据湖 "是很昂贵的（比存储要贵得多），这也是我们在第12章中深入考虑的一个话题。

2. 界面的灵活性。

如今，大多数用户都用类似SQL的语言与大数据引擎进行互动。但是，这些引擎也允许用户使用多种范式组合进行编程。例如，一个组织可能会使用指令性代码来执行文件解析，使用SQL来预测一个列，使用机器学习子程序来对结果进行聚类，所有这些都在一个单一的框架内。像DryadLINQ那样严密的、习惯性的语言整合是很常见的，这进一步提高了可编程性。传统的数据库引擎历来支持用户定义的函数，而这些新引擎的接口使用户定义的计算更容易表达，也使用户定义的计算结果与使用传统关系结构（如SQL）表达的查询结果更容易整合。接口的灵活性和集成性是数据分析产品的一个强有力的卖点；在一个系统中结合ETL、分析和后处理的能力对于程序员来说是非常方便的--但对于使用传统JDBC接口的传统BI工具的用户来说就不一定了。

3. 架构的灵活性。

对RDBMS的一个常见的批评是它们的架构过于紧密耦合：存储、查询处理、内存管理、事务处理等等都紧密地交织在一起，在实践中它们之间缺乏明确的接口。相比之下，由于自下而上的发展，Hadoop生态系统已经有效地将数据仓库建成了一系列的模块。今天，企业可以针对原始文件系统（如HDFS）、任何数量的数据流引擎（如Spark）、使用高级分析包（如GraphLab[18]、Parameter Server[17]）或通过SQL（如Impala[16]）编写和运行程序。这种灵活性增加了性能开销，但混合和匹配组件和分析包的能力在这种规模下是前所未有的。这种架构上的灵活性可能对系统建设者和供应商最感兴趣，他们在设计他们的基础设施产品时有更多的自由度。

总而言之，当今分布式数据管理基础设施的一个主要主题是灵活性和异质性：存储格式、计算范式和系统实现。其中，存储格式的异质性可能是影响最大的一个数量级，甚至更多，因为它对新手、专家和建筑师都有影响。相反，计算范式的异质性对专家和建筑师的影响最大，而系统实现的异质性对建筑师的影响最大。所有这三者都是数据库研究的相关和令人兴奋的发展，但关于市场影响和寿命的问题却挥之不去。

## Looking Ahead

从某种意义上说，MapReduce是一个短暂的、极端的架构，它吹开了一个设计空间。该架构简单，可扩展性强，它在开源领域的成功让很多人意识到，人们对替代性解决方案和它所体现的灵活性原则有需求（更不用说基于开源的更便宜的数据仓库解决方案的市场机会）。由此产生的兴趣仍然让许多人感到惊讶，这是由许多因素造成的，包括社区的潮流、巧妙的营销、经济和技术的转变。考虑这些新系统和RDBMS之间的哪些差异是根本性的，哪些是由于工程上的改进，是很有意思的。

今天，关于大规模数据处理的适当架构仍然存在争议。作为一个例子，Rasmussen等人提供了一个强有力的论据，说明为什么除了在非常大的（100多个节点）集群中，中间的容错是没有必要的[23]。另一个例子是，McSherry等人彩色地说明了许多工作负载可以使用单个服务器（或线程！）有效地处理，完全不需要分配[20]。最近，GraphLab项目[18]等系统提出，特定领域的系统对于性能来说是必要的；后来的工作，包括Grail[9]和GraphX[13]，认为情况不一定是这样。最近的另一波提议也提出了用于流处理、图处理、异步编程和通用机器学习的新接口和系统。是否真的需要这些专门的系统，或者一个分析引擎可以统治所有的系统？时间会证明这一点，但我认为这是对整合的推动。

最后，不能不提到 Spark，它只有6年的历史，但却越来越受到开发者的欢迎，并且得到了风险投资支持的初创公司（如 Databricks）和 Cloudera 和 IBM 等成熟公司的大力支持。尽管由于 DryadLINQ 的历史意义和技术深度，我们将其作为后 MapReduce 系统的一个例子，但写于项目早期的 Spark 论文[27]和最近的扩展，包括SparkSQL[4]，都值得额外阅读。像 Hadoop 一样，Spark 在相对早期的成熟阶段就唤起了人们的兴趣。今天，在其功能集与传统数据仓库相媲美之前，Spark 仍有一段路要走。然而，它的功能集正在迅速增长，人们对Spark作为MapReduce在Hadoop生态系统中的继承者的期望很高；例如，Cloudera正在努力用Spark在Hadoop生态系统中取代MapReduce[10]。时间会证明这些期望是否准确；与此同时，传统仓库和后MapReduce系统之间的差距正在迅速缩小，从而产生了与传统系统一样擅长数据仓库的系统，但也远远不止如此。

## References:

[1] Alexandrov, A., Bergmann, R., Ewen, S., Freytag, J.-C., Hueske, F., Heise, A., Kao, O., Leich, M., Leser, U., Markl, V. and others The Stratosphere platform for big data analytics. *The VLDB Journal*. 23, 6 (2014), 939-964.

[2] Alsubaiee, S., Altowim, Y., Altwaijry, H., Behm, A., Borkar, V., Bu, Y., Carey, M., Cetindil, I., Cheelangi, M., Faraaz, K. and others Asterixdb: A scalable, open source bDMS. *VLDB*, 2014.

[3] Apache Tez: Apache Tez. https://tez.apache.org/.

[4] Armbrust, M., Xin, R.S., Lian, C., Huai, Y., Liu, D., Bradley, J.K., Meng, X., Kaftan, T., Franklin, M.J., Ghodsi, A. and others Spark SQL: Relational data processing in spark. *SIGMOD*, 2015.

[5] Babu, S. and Herodotou, H. Massively parallel databases and MapReduce systems. *Foundations and Trends in Databases*. 5, 1 (2013), 1-104.

[6] Burrows, M. The chubby lock service for loosely-coupled distributed systems. *OSDI*, 2006.

[7] Chang, F., Dean, J., Ghemawat, S., Hsieh, W.C., Wallach, D.A., Burrows, M., Chandra, T., Fikes, A. and Gruber, R.E. Bigtable: A distributed storage system for structured data. *OSDI*, 2006.

[8] DeWitt, D. and Stonebraker, M. MapReduce: A major step backwards. *The Database Column*. (2008).

[9] Fan, J., Gerald, A., Raj, S. and Patel, J.M. The case against specialized graph analytics engines. *CIDR*, 2015.

[10] Forbes: Why Cloudera is saying ’Goodbye, MapReduce’ and ’Hello, Spark’: Forbes: Why Cloudera is saying ’Goodbye, MapReduce’ and ’Hello, Spark’. 2015. http://fortune.com/2015/09/09/cloudera-spark-mapreduce/.

[11] Ghemawat, S., Gobioff, H. and Leung, S.-T. The google file system. *SOSP*, 2003.

[12] Ghoting, A., Krishnamurthy, R., Pednault, E., Reinwald, B., Sindhwani, V., Tatikonda, S., Tian, Y. and Vaithyanathan, S. SystemML: Declarative machine learning on mapReduce. *ICDE*, 2011.

[13] Gonzales, J.E., Xin, R.S., Crankshaw, D., Dave, A., Franklin, M.J. and Stoica, I. GraphX: Unifying data-parallel and graph-parallel analytics. *OSDI*, 2014.

[14] Hausenblas, M. and Nadeau, J. Apache Drill: Interactive ad-hoc analysis at scale. *Big Data*. 1, 2 (2013), 100-104.

[15] Isard, M., Budiu, M., Yu, Y., Birrell, A. and Fetterly, D. Dryad: Distributed data-parallel programs from sequential building blocks. *EuroSys*, 2007.

[16] Kornacker, M., Behm, A., Bittorf, V., Bobrovytsky, T., Ching, C., Choi, A., Erickson, J., Grund, M., Hecht, D., Jacobs, M. and others Impala: A modern, open-source sQL engine for hadoop. *CIDR*, 2015.

[17] Li, M., Andersen, D.G., Park, J.W., Smola, A.J., Ahmed, A., Josifovski, V., Long, J., Shekita, E.J. and Su, B.-Y. Scaling distributed machine learning with the parameter server. *OSDI*, 2014.

[18] Low, Y., Bickson, D., Gonzalez, J., Guestrin, C., Kyrola, A. and Hellerstein, J.M. Distributed graphLab: A framework for machine learning and data mining in the cloud. *VLDB*, 2012.

[19] Malewicz, G., Austern, M.H., Bik, A.J., Dehnert, J.C., Horn, I., Leiser, N. and Czajkowski, G. Pregel: A system for large-scale graph processing. *SIGMOD*, 2010.

[20] McSherry, F., Isard, M. and Murray, D.G. Scalability! But at what COST? *HotOS*, 2015.

[21] Murray, D.G., McSherry, F., Isaacs, R., Isard, M., Barham, P. and Abadi, M. Naiad: A timely dataflow system. *SOSP*, 2013.

[22] Olston, C., Reed, B., Srivastava, U., Kumar, R. and Tomkins, A. Pig latin: A not-so-foreign language for data processing. *SIGMOD*, 2008.

[23] Rasmussen, A., Lam, V.T., Conley, M., Porter, G., Kapoor, R. and Vahdat, A. Themis: An i/O-efficient mapReduce. *SoCC*, 2012.

[24] Shute, J., Vingralek, R., Samwel, B., Handy, B., Whipkey, C., Rollins, E., Oancea, M., Littlefield, K., Menestrina, D., Ellner, S. and others F1: A distributed sQL database that scales. *VLDB*, 2013.

[25] Sparks, E.R., Talwalkar, A., Smith, V., Kottalam, J., Pan, X., Gonzalez, J., Franklin, M.J., Jordan, M., Kraska, T. and others MLI: An aPI for distributed machine learning. *ICDM*, 2013.

[26] Thusoo, A., Sarma, J.S., Jain, N., Shao, Z., Chakka, P., Anthony, S., Liu, H., Wyckoff, P. and Murthy, R. Hive: A warehousing solution over a map-reduce framework. *VLDB*, 2009.

[27] Zaharia, M., Chowdhury, M., Das, T., Dave, A., Ma, J., McCauley, M., Franklin, M.J., Shenker, S. and Stoica, I. Resilient distributed datasets: A fault-tolerant abstraction for in-memory cluster computing. *NSDI*, 2012.

## Comments

**Michael Stonebraker**

**26 October 2015**

最近，作为 "大数据 "这一营销名称的一部分，人们对数据分析产生了极大的兴趣。历史上，这意味着商业智能（BI）分析，并由BI应用程序（Cognos、Business Objects等）与关系型数据仓库（如Teradata、Vertica、Red Shift、Greenplum等）进行对话。最近，它已经与 "数据科学 "联系起来。在这种情况下，让我们从十年前的Map-Reduce开始，它是由谷歌专门为支持他们的网络抓取数据库而建立的。然后，营销人员接管了基本论点。"谷歌很聪明；Map-Reduce是谷歌的下一个大东西，所以它一定很好"。Cloudera、Hortonworks和Facebook是宣传Map-Reduce（及其开源的外观相似的Hadoop）的急先锋。几年前，市场上出现了喝着Map-Reduce酷饮的声音。大约在同一时间，谷歌停止将Map-Reduce用于其专门设计的应用，转而使用Big Table。拖延了大约5年之后，世界上的其他国家都看到了谷歌早先发现的问题；Map-Reduce并不是一个具有广泛适用性的架构。

实际上，Map-Reduce存在以下两个问题。

1. 作为建立数据仓库产品的平台，它是不合适的。任何商业数据仓库产品中都没有类似Map-Reduce的接口，这是有原因的。因此，DBMS并不希望有这样的平台。

2. 作为建立分布式应用的平台，它是不合适的。不仅Map-Reduce的接口对分布式应用来说不够灵活，而且使用文件系统的消息传递系统也太慢了，没有什么意思。

当然，这并没有阻止Map-Reduce供应商。他们只是把自己的平台重新命名为HDFS（一个文件系统），并在HDFS的基础上建立了不包括Map-Reduce的产品。例如，Cloudera最近推出了Impala，它是一个SQL引擎，而不是建立在Map-Reduce之上。事实上，Impala也没有真正使用HDFS，而是选择钻过该层，直接读写底层的本地Linux文件。HortonWorks和Facebook也在进行类似的项目。因此，Map-Reduce人群已经变成了SQL人群，而Map-Reduce作为一种接口，已经成为历史。当然，HDFS在被SQL引擎使用时有很严重的问题，所以不清楚它是否会有什么发展，但这还有待观察。无论如何，Map-Reduce-HDFS市场将与SQL-数据仓库市场合并；并希望最好的系统能够获胜。总之，Map-Reduce作为一个分布式系统平台已经失败了，厂商们正在使用HDFS作为数据仓库产品下的一个文件系统。

这就把我们带到了Spark。Spark的最初论点是它是Map-Reduce的一个更快的版本。它是一个具有快速消息传递接口的主内存平台。因此，在用于分布式应用时，它不应该受到Map-Reduce的性能问题的影响。然而，根据Spark的主要作者Matei Zaharia的说法，超过70%的Spark访问是通过SparkSQL进行的。实际上，Spark正被用作一个SQL引擎，而不是一个分布式应用平台! 在这种情况下，Spark有一个身份问题。如果它是一个SQL平台，那么它需要一些持久性、索引、用户之间共享主内存、元数据目录等机制，以便在SQL/数据仓库领域具有竞争力。看来，Spark很可能会变成一个数据仓库平台，跟随Hadoop走同样的道路。

另一方面，30%的Spark访问不是到SparkSQL，主要来自Scala。据推测，这是一种分布式计算负载。在这种情况下，Spark是一个合理的分布式计算平台。然而，有几个问题需要考虑。首先，普通的数据科学家做的是数据管理和分析的混合工作。更高的性能来自于两者的紧密耦合。在Spark中没有这种耦合，因为Spark的数据格式在这两个任务中不一定通用。第二，Spark是纯内存的（至少目前是这样）。随着时间的推移，可扩展性的要求可能会使这个问题得到解决。因此，看看Spark如何向未来发展将是很有趣的。
综上所述，我想提供以下几点启示。

- 仅仅因为谷歌认为某个东西是个好主意，并不意味着你应该采用它。
- 不相信所有的营销策略，并弄清楚任何特定的产品实际上有什么好处。这一点应该特别适用于性能声明。
- 程序员群体对 "下一个闪亮的东西 "情有独钟。这可能会在你的组织中造成 "流失"，因为闪亮物体的 "半衰期 "可能很短。
