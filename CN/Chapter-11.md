# Chapter 11: A Biased Take on a Moving Target: Complex Analytics

## by Michael Stonebraker

在过去的5-10年里，出现了比典型的商业智能(BI)用例更复杂的新的分析场景。 例如，互联网广告商可能想知道“在过去四天里买了苹果电脑的女性与在同一时期买了福特皮卡的女性在统计数字上有什么不同?”下一个问题可能是:“在我们所有的广告中，根据女性消费者的点击可能性，向她们展示哪一个最赚钱?”这些是当今数据科学家提出的问题，它们代表了一个与商业智能专家运行的传统SQL分析非常不同的用例。人们普遍认为，在未来10年或20年，数据科学将完全取代商业智能，因为它代表了一种更复杂的方法，可以从数据仓库中挖掘新的见解。因此，本文主要关注数据科学家的需求。

我将在这一节开始描述我所认为的数据科学家的工作内容。在清理和处理他的数据后，目前消耗了他绝大部分的时间，这在数据整合一节中讨论过，他一般会进行以下的迭代。

```
Until (tired) {

Data management operation(s); Analytic operation(s);

}
```

换句话说，他有一个迭代发现过程，他隔离感兴趣的数据集，然后对其执行一些分析操作。这通常表明要么使用不同的数据集尝试相同的分析，要么对同一数据集进行不同的分析。总的来说，数据科学与商业智能的区别在于分析是预测建模、机器学习、回归……而不是 SQL 分析。

换句话说，他有一个迭代的发现过程，在此过程中，他分离出感兴趣的数据集，然后对其进行分析操作。这通常建议使用不同的数据集来尝试相同的分析，或者在相同的数据集上使用不同的分析。总的来说，数据科学与商业智能的区别在于，分析是预测建模、机器学习、回归……而不是SQL分析。

通常，有一个组成分析的计算管道。例如，Tamr有一个模块，它对一个记录集合(比如N个)执行实体合并(重复数据删除)。避免N * * 2蛮力算法的复杂性,Tamr标识“特性”的集合,将他们分为范围不可能同时发生,计算(可能是多个)“垃圾箱”为每一个记录基于这些范围,重组并行所以分区的数据本,每本删除处理,合并结果,最后构造复合各种集群的重复记录。 这个管道部分是面向sql的(分区)，部分是面向数组的分析。Tamr似乎是典型的数据科学工作负载，因为它是一个包含6个步骤的管道。

## Commentary: Joe Hellerstein

6 December 2015

在这方面我有Mike有较大不同的看法，不管是从商业角度还是研究计划角度。基本上，我推荐一个“大帐篷”来靠近这个领域。DB家族有太多可以贡献的，但是我们如果我和其他领域合作愉快，我们会做得更多。

让我们看看工业界。首先，我们这里讨论的先进分析不会像Mike说的取代BI。BI届非常健康而且在不断成长。更基本的是，如统计学家John Tukey在他们的探索性数据分析报告中指出的，图标常常比复杂的统计模型更有价值。向图标致敬！

文中提到，先进分析和数据科学市场确实在发展和不断变化。但是和BI市场不一样的是，在其中数据库技术现在并没有扮演重要角色。而这个角色现在是SAS，一个每年收益几十亿美元的公司，不是一个数据库公司。当VC在这个领域寻找公司时，他们在寻找下一个’SAS’。SAS用户不是数据库用户。而且使用开源替代方案如R的用户也不是数据库用户。如果假设Mike说的“数据科学家将使用数据库技术”—一个专项巨大的“分析型数据库”—那你就是在激流中逆流而上。

先进分析市场更自然的实现路线是什么，问自己：SAS真正的威胁是什么？谁能从企业目前在这个方面的现金支出中分得一杯羹呢?以下是思考起点：

1.开源编程：包括R和python数据科学生态（NumPy,SciKit-Learn,IPython Notebook）。这些解决方案目前都不能很好地扩展，但是面向这些限制的努力在进行中。这个生态进化比SAS快。

2.大数据平台的松耦合。当数据足够大，处于性能要求迫使用户选择新的平台—在组织中已经存储了这些数据的平台。于是，他们致力于构建平台的数据框架如Spark/MLLib, PivotalR/MADlib,和 Ver- tica dplyr。注意先进分析社区更偏向于开源。在云环境也对这块有兴趣，而不是SAS占据优势的领域。

3.分析服务。这里指的是使用分析手段的在线的交互服务：推荐系统，实时欺诈分析，设备预测保护等。这些领域都有强烈的系统响应时间需求，要求扩展，故障容错和SAS产品不具备的持续进化能力。今天，这些服务大量由客户代码构建。这些并没有跨行业扩展--大部分企业没有招聘到能在这个层面工作的开发者。因此，从表面上看，对于大多数用例来说，这是一个将这项技术商品化的机会。但是这还是市场早期—还要看分析服务平台是否可能做到足够简单，以用于二次开发。如果基础进化，基于云的服务可能有机会中断这个机会。

通过前面的研究，我认为站在数据库之外思考很重要，然后积极合作。对我来说这是不言而喻的。近期计算领域的每个子域都在蹭热点处理大数据分析，来自不同领域的聪明人都在快速学习他们自己关于数据和等级的课程。我们可以和这些人一起玩得很开心，也可以忽视他们。

那数据库研究在哪里可以造成大的冲击？在我看来有如下方面：

1.扩展能力的新方法。我们已经提供了并行数据流—考虑MADlib，MILlib或Teradata在Ordonez的工作—将带给你如何实现可扩展的分析而不对系统架构做出破坏。这很有用。进一步看，我们是否可以做点什么来实现比并行数据流更快和扩展性更强的方案？是否有必要Hogwild !已经在这里产生了一些最大的兴奋;请注意，这是跨越数据库和ML社区的工作。

2.用于分析服务的分布式基础设施。如上面提到的，分析服务是革新的机会。系统基础设施问题非常广泛。分析服务架构的主要组件是什么？他们如何组织在一起？这些组件之间的数据一致性要求如何？所谓的参数服务器是现在的主题，但支出了很小一部分。在在线服务、模型的演化和部署方面已经有了一些初步的工作。我希望会有更多。

3.分析生命周期和元数据管理。这一点我同意Mike的观点。分析通常是一项人为密集的工作，除了核心的统计模型构建外，还包括数据探索和转换。在此过程中，需要管理大量的上下文，以了解如何跨一系列工具和系统开发模型和数据产品。数据库社区在这个领域有高度相关的观点，包括工作流管理、数据沿袭和物化视图维护。visi - trails是该领域研究的一个例子，目前正在应用于实践。这也是工业界迫切需要的一个领域——尤其是考虑到该领域中分析工具和系统的实际多样性的工作。

## References:

[1] Baumann, P., Dehmel, A., Furtado, P., Ritsch, R. and Widmann, N. The multidimensional database system rasDaMan. *SIGMOD*, 1998.

[2] Chilimbi, T., Suzue, Y., Apacible, J. and Kalyanaraman, K. Project adam: Building an efficient and scalable deep learning training system. *OSDI*, 2014.

[3] Choi, J. and others ScaLAPACK: A portable linear algebra library for distributed memory computers—Design issues and performance. *Applied parallel computing computations in physics, chemistry and engineering science*. Springer. 95-106.

[4] Dean, J., Corrado, G., Monga, R., Chen, K., Devin, M., Mao, M., Senior, A., Tucker, P., Yang, K., Le, Q.V. and others Large scale distributed deep networks. *Advances in neural information processing systems*, 2012, 1223-1231.

[5] Duggan, J. and Stonebraker, M. Incremental elasticity for array databases. *Proceedings of the 2014 aCM sIGMOD international conference on management of data*, 2014, 409-420.

[6] Elmore, A., Duggan, J., Stonebraker, M., Balazinska, M., Cetintemel, U., Gadepally, V., Heer, J., Howe, B., Kepner, J., Kraska, T. and others A demonstration of the BigDAWG polystore system. *VLDB*, 2015.

[7] Gonzales, J.E., Xin, R.S., Crankshaw, D., Dave, A., Franklin, M.J. and Stoica, I. GraphX: Unifying data-parallel and graph-parallel analytics. *OSDI*, 2014.

[8] Hellerstein, J.M., Ré, C., Schoppmann, F., Wang, D.Z., Fratkin, E., Gorajek, A., Ng, K.S., Welton, C., Feng, X., Li, K. and others The MADlib analytics library: or MAD skills, the SQL. *VLDB*, 2012.

[9] Jindal, A., Rawlani, P., Wu, E., Madden, S., Deshpande, A. and Stonebraker, M. Vertexica: Your relational friend for graph analytics! *VLDB*, 2014.

[10] Kepner, J. and others Dynamic distributed dimensional data model (D4M) database and computation system. *Acoustics, speech and signal processing (iCASSP), 2012 iEEE international conference on*, 2012, 5349-5352.

[11] Low, Y., Bickson, D., Gonzalez, J., Guestrin, C., Kyrola, A. and Hellerstein, J.M. Distributed graphLab: A framework for machine learning and data mining in the cloud. *VLDB*, 2012.

[12] Recht, B., Re, C., Wright, S. and Niu, F. Hogwild: A lock-free approach to parallelizing stochastic gradient descent. *Advances in neural information processing systems*, 2011, 693-701.

[13] Siva, N. 1000 genomes project. *Nature biotechnology*. 26, 3 (2008), 256-256.

[14] Stonebraker, M., Madden, S. and Dubey, P. Intel big data science and technology center vision and execution plan. *ACM SIGMOD Record*. 42, 1 (2013), 44-49.

[15] The SciDB Development Team Overview of SciDB: Large scale array storage, processing and analysis. *SIGMOD*, 2010.
