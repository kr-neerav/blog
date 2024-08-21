+++
title = 'Query Optimization'
date = 2024-08-18T06:16:28-07:00
draft = true
+++
## Overview
* There are 3 steps to come up with alternate evaluation plans
* First generate relational expressions that are logically equivalent to the given expression.
* Second annotating the resultant expressions in alternate ways e.g. annotating about which parts are pipeline, which algorithms can be used
* Third estimate cost of each evaluation plan and choosing one with least estimated cost. It does so using statistics about relations.
* These 3 steps are interleaved in Optimizer.
* The first step is done by means of equivalence rules that specify how to transform an expression into logically equivalent one.
## Transformation of Relational Expressions
* 2 relational expressions are equivalent if they produce the same result set, but can be in different order.
### Equivalence Rules
* and filter conditions can be created as a sequence of individual filter conditions (pipeline)
* selection operation are commutative i.e. order of application does not matter.
* if there are multiple projects in sequence, only the final is needed.
* selection operation can be applied after join e.g if a join condition contains equip join and a non equi comparison then the non equi condition can be performed after join as a filter.
* equi joins are associative i.e. order of joining relations does not matter.
* projection after join can is equivalent to first projection and then join.
* selection after union/intersection can be done with first selection and then union/intersection
* it is not necessary to generate every equivalent expression using the rules. the optimizer can take into account cost to not generate some expressions.
### Join Ordering
* a good join ordering is important to reduce the size of temporary results.
* since joins are associative the join order does not impact the final output but can impact the performance.
## Estimating statistics of expression results
### Catalog Information
* type of statistics stored in catalog
* number of tuples in a relation
* number of blocks in a relation
* size of tuple in a relation
* blocking factor i.e. number of tuples in a bluck
* number of distinct values for each attribute or a set of attributes.
* histogram for continuous values. if historgram is not present optimizer assumes uniform distribution
### Selction Size Estimation
* single equality predicate: if the histogram is present the % of values in the range of it is assumed as the size of the predicate. If no histogram present, it is assumed the value is present in N/number of distinct values in the column.
* single compariso predicate: include non equi conditions like <= or >=. If historogram present it is estimated as the area under the range of values. if histogram not present it is assumes value is uniformly present and calculated using min/max values.
* if there are multiple predicates joined with AND their result is multiplied i.e. % of records expected in output is multiplied. hence multiple predicates reduces the estimated size of output.
* if there are multiple predicates joined with OR their result is expected to have more records.
* negation: the % of records is calculated by equality predicate and 1 - that is the output of negation.
### Join Size Estimation
* if there is no join condition then it is a cartesian product and it is easy to estimate the output size
* if one of the tables has a FK to other table then each record in the FK table will have atmost 1 record from primary table. Hence the size is estimated as the size of selection of FK table.
* if the join keys are not PKs in table then the estimate is decided by n1 * n2/number of distinct values of join keys in table 1 or 2. The higher of the denominotor is taken for estimate, as if a relation has more distinct values then the extra distinct values will not be part of output, unless outer join.
### Size estimation for other operations
* Project: same tuple count, operation does not change number of output records, it might change the number of blocks needs to store the output.
* aggregation: number of distinct values of the set of groupby columns
* set operations: if they are on the same table then we try to convert the operations into combination of filter conditions to get the size. if they are from different relations the estimate of union is Nr + Ns, minus r - s is Nr (assume no common record as we are estimating upper bound) and intersection r intersection r is Nr. The estimates are not accurate but provide an upper bound.
* outer join: r left outer join s is size of r join s + size of r. symetric for right outer join. full outer join is size of r join s + size of r + size of s.
## Choice of evaluation plan
* a cost based optimizer explores the space of all query evaluation plans that includes different relational expressions and algorithms used to execute those expressions.
* exploring all possible plans might be too expensive. the optimizer uses heuristics to reduce the search space for plans.
### Heuristics in Optimization
* A drawback of cost based optimizer is the cost of optimization itself. Heuristics are used to make it efficient. Heuristics work in most of the cases but not all. Sometimes they might end up increasing the cost.
* Perform selection operation as early as possible (filter pushdown). 
* Perform projections early. It is usually better to perform selection before projection.
### Nested subqueries
* coorelated subqueries are costly to execute as they can result in random disk I/O as relation needs to be accessed for each tuple. An optimization approach used is to convert this into joins and use efficient join algorithms. Where is not possible to convert to a join the query is exeucted in its original form (correlated)
* the process of replacing a correlated subquery with a join is called decorrelation.
* decorrelation is more complicated if nested subquery contains aggregation or is used as test for equality.
* given this complications it is recommended to avoid nested subqueries.
* 
