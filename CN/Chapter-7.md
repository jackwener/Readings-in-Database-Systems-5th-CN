# Chapter 7: Query Optimization

## Introduced by Joe Hellerstein

> Readings:
>
> [Goetz Graefe and William J. McKenna. The Volcano Optimizer Generator: Extensibility and Efficient Search. *ICDE*, 1993.](https://scholar.google.com/scholar?cluster=2304531151126477511)
>
> [Ron Avnur and Joseph M. Hellerstein. Eddies: Continuously Adaptive Query Processing. *SIGMOD*, 2000.](https://scholar.google.com/scholar?cluster=13049208738754012194)
>
> [Volker Markl, Vijayshankar Raman, David Simmen, Guy Lohman, Hamid Pirahesh, Miso Cilimdzic. Robust Query Processing Through Progressive Optimization. *SIGMOD*, 2004.](https://scholar.google.com/scholar?cluster=4929312332613352080)

查询优化是数据库技术的标志性组件之一——它是声明式语言(declarative languages)连接至高效执行(efficient execution)的桥梁。查询优化器被誉为 DBMS 中最难实现的部分之一，因此不同成熟的商业DBMS之间的优化器有着明显差异，这也就不足为奇了。对比之下，最好的开源关系数据库的优化器往往比较有限(limited)，有些优化器相对简单，只适用于最简单的查询。

很重要的一点，没有查询优化器能生成 "最佳 "计划。首先，优化器都会使用估计技术(estimation techniques)来估算实际的 plan cost，而且众所周知的是估计技术的错误可能会膨胀，在一些情况下甚至会和随机猜测(random guesses)一样差[7]。其次，因为优化器的优化问题是NP-hard，所以优化器使用启发式来限制可选的 plan 的搜索空间，[6]。最近被显著关注的一个假设是传统的两表 join operators 的使用；这已经被证明在理论上不如某些情况下的新的 multi-way join 算法[12]。

尽管有着这些限制，关系查询优化已被证明是成功的，并且它使关系数据库系统能够在实践中很好地用于广泛用例。数据库供厂商已经投入多年时间使得它们的优化器在一系列用例上能够可靠地执行。用户已经了解 join 数量的限制。大多数场景下，在优化器的作用下，声明式 SQL 查询成为比命令式(imperative code)代码更好的选择。

除了难以构建和调整之外，随着时间的推移，成熟的查询优化器也有变得越来越复杂的趋势，因为它们需要处理更大量的 workloads 和更多的 corner cases. 。关于数据库查询优化的研究文献实际上本身就是一个领域，充满了技术细节——IBM 和微软等与产品组密切合作的成熟供应商的研究人员已经在文献中讨论了其中的许多细节。在本书中只关注整体全局：面向查询优化的主要架构以及它们是怎样随着时间被重新评估。

## Volcano/Cascades

我们从最先进的技术开始。从数据库研究的早期开始，有两种用于查询优化的参考架构，它们涵盖了当今大多数重要的优化器实现。第一个是第 3 章中描述的 Selinger 等人的 System R 优化器。 System R 的优化器是教科书材料，已在许多商业系统中实现；每个数据库研究人员都应该详细了解它。第二个是 Goetz Graefe 和他的合作者在一系列研究项目中改进的架构：Exodus、Volcano 和 Cascades。 Graefe 的工作在研究文献或教科书中没有像 System R 工作那样频繁地出现，但它在实践中得到了广泛的应用，特别是在 Microsoft SQL Server 中，但据称也在许多其他商业系统中使用。 Graefe 关于这个主题的论文有一些内部人士的味道——面向那些了解并关心实现查询优化器的人。我们为本书选择了 Volcano 论文作为该工作最平易近人的代表，但爱好者也应该阅读 Cascades 论文 [5]——它不仅提出并解决了 Volcano 的许多详细缺陷，而且是最新的（并且因此标准）该方法的参考。最近，出现了两个开源的 Cascades 式优化器：Greenplum 的 Orca 优化器现在是 Greenplum 开源的一部分，Apache Calcite 是一个优化器，可以与多种后端查询执行器和语言一起使用，包括 LINQ。

Graefe 的优化器架构之所以引人注目，主要有两个原因。首先，它被明确设计为可扩展的。 Volcano 在探索数据流可用于广泛的数据密集型应用程序的想法方面具有前瞻性——早于 MapReduce 和大数据堆栈——值得称赞。因此，Graefe 优化器不仅仅用于将 SQL 编译成数据流迭代器计划。它们可以针对其他输入语言和执行目标进行参数化；近年来，随着专业数据模型和语言的兴起（参见第 2 章和第 9 章），另一方面是专门的执行引擎（第 5 章），这是一个高度相关的主题。这些优化器的第二个创新是使用自上而下或面向目标的搜索策略，以在可能的计划空间中找到最便宜的计划。这种设计选择与 Graefe 设计中的可扩展性 API 相关，但这不是内在的：Starburst 系统展示了如何为 Selinger 自底向上算法进行可扩展性 [9]。这种关于查询优化的“自上而下”与“自下而上”的争论，双方都有支持者，但没有明显的赢家；类似的自上而下/自下而上的辩论也或多或少地与递归查询处理文献有关 [13]。爱好者会感兴趣地注意到，文献递归查询处理和查询优化器搜索这两个主体在 Evita Raced 优化器中直接相连，该优化器通过使用递归查询作为语言来实现自上而下和自下而上的优化器搜索。实施优化器 [1]。

## Adaptive Query Processing

到 1990 年代后期，一些趋势表明查询优化的整体架构值得重新思考。这些趋势包括：

- 对流数据的连续查询。
- 数据探索的交互式方法，如在线聚合。
- 对 DBMS 之外的数据源的查询，不提供可靠的统计信息或性能。
- 不可预测的动态执行环境，包括弹性和多租户设置以及传感器网络等广泛分布的系统。
- 查询中的不透明数据和用户定义函数，只能通过观察行为来估计统计信息。

此外，对于多运营商查询的计划成本估算通常不稳定的理论事实一直存在实际问题 [7]。由于这些趋势，人们对处理查询的自适应技术产生了兴趣，其中执行计划可能会在查询中改变。我们在自适应查询处理的设计空间中提出了两个互补点；有一个更全面的概述 [4] 的长调查。

## Eddies

## Progressive Optimization

## Discussion

## References:

[1] Condie, T., Chu, D., Hellerstein, J.M. and Maniatis, P. Evita raced: Metacompilation for declarative networks. *Proceedings of the VLDB Endowment*. 1, 1 (2008), 1153-1165.

[2] Deshpande, A. An initial study of overheads of eddies. *ACM SIGMOD Record*. 33, 1 (2004), 44-49.

[3] Deshpande, A. and Hellerstein, J.M. Lifting the burden of history from adaptive query processing. *VLDB*, 2004.

[4] Deshpande, A., Ives, Z. and Raman, V. Adaptive query processing. *Foundations and Trends in Databases*. 1, 1 (2007), 1-140.

[5] Graefe, G. The cascades framework for query optimization. *IEEE Data Eng. Bull.* 18, 3 (1995), 19-29.

[6] Ibaraki, T. and Kameda, T. On the optimal nesting order for computing n-relational joins. *ACM Transactions on Database Systems (TODS)*. 9, 3 (1984), 482-502.

[7] Ioannidis, Y.E. and Christodoulakis, S. On the propagation of errors in the size of join results. *SIGMOD*, 1991.

[8] Kabra, N. and DeWitt, D.J. Efficient mid-query re-optimization of sub-optimal query execution plans. *SIGMOD*, 1998.

[9] Lohman, G.M. Grammar-like functional rules for representing query optimization alternatives. *SIGMOD*, 1988.

[10] Madden, S., Shah, M., Hellerstein, J.M. and Raman, V. Continuously adaptive continuous queries over streams. *SIGMOD*, 2002.

[11] Melton, J., Michels, J.E., Josifovski, V., Kulkarni, K. and Schwarz, P. SQL/MED: A status report. *ACM SIGMOD Record*. 31, 3 (2002), 81-89.

[12] Ngo, H.Q., Porat, E., Ré, C. and Rudra, A. Worst-case optimal join algorithms:[extended abstract]. *Proceedings of the 31st symposium on principles of database systems*, 2012, 37-48.

[13] Ramakrishnan, R. and Sudarshan, S. Top-down vs. bottom-up revisited. *Proceedings of the international logic programming symposium*, 1991, 321-336.

[14] Raman, V. and Hellerstein, J.M. Partial results for online query processing. *SIGMOD*, 2002, 275-286.

[15] Raman, V., Deshpande, A. and Hellerstein, J.M. Using state modules for adaptive query processing. *ICDE*, 2003.

[16] Urhan, T., Franklin, M.J. and Amsaleg, L. Cost-based query scrambling for initial delays. *ACM SIGMOD Record*. 27, 2 (1998), 130-141.

[17] Wilschut, A.N. and Apers, P.M. Dataflow query execution in a parallel main-memory environment. *Parallel and distributed information systems, 1991., proceedings of the first international conference on*, 1991, 68-77.

[18] Wong, E. and Youssefi, K. Decomposition—a strategy for query processing. *ACM Transactions on Database Systems (TODS)*. 1, 3 (1976), 223-241.