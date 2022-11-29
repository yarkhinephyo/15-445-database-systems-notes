**Physical operator**: Define an execution strategy using an access path. Not always 1:1 mapping from logical operator.

**Architecture**

1. SQL Rewriter (Rare): Convert SQL to SQL string.
2. SQL Parser spits out abstract syntax tree.
3. Binder maps AST to database objects (For example: Table name to table object found in system catalog), and produces a logical plan.
4. Tree Rewriter (Common): Rewrite logical plan.
5. Optimizer estimates cost with cost model and produces a physical plan.

**Heuristics**: Rewrite the query to remove inefficiency without examining the data.

**Cost based search**: Enumerate multiple equivalent plans and produce the one with lowest cost.

**Logical plan optimization**: Transform logical plan with pattern matching rules. Increases the likelihood of enumerating the optimal plan in the search.

<u>1. Split conjunctive predicates</u> - Decompose predicates into their simplest forms.

![](images/Pasted%20image%2020221026224907.png)

<u>2. Predicate pushdown</u> - Move predicate to the lowest applicable point.

<u>3. Replace cartesian products</u> - Replace cartesian products with inner joins using the join predicates.

![](images/Pasted%20image%2020221026225633.png)

<u>4. Projection pushdown</u> - Eliminate redundant attributes before the pipeline breakers.

![](images/Pasted%20image%2020221026225916.png)

**Nested subqueries**: Rewrite to flatten them or decompose and store result to temporary table.

Correlated queries gets rewritten as a join.

![](images/Pasted%20image%2020221026230839.png)

Uncorrelated queries have results stored in a temporary location.

![](images/Pasted%20image%2020221026231235.png)k

**Expression rewriting**

1. Remove impossible/unnecessary predicates.
2. Merging predicates. (Combining range of values)

**Cost-based search**

Cost can be physical costs (CPU cycles, cache misses), logical costs (estimate output size per operator), algorithmic costs (complexity of the operator algorithm). 

**Selection cardinality**

Selection of a predicate is the fraction of tuples that qualify.

<u>Assumption 1</u>: Distribution of values is uniform.

<u>Assumption 2</u>: Predicates are independent.

<u>Assumption 3</u>: Every tuple in inner table is guaranteed to exist in outer table during join.

The assumptions does not always hold true. For example, a model of a car may only be made by a brand. If there are 10 models and 10 brands, the selectivity is 0.01 with assumption 1 and 2 but it is not true.

Equai-width histogram can be used to maintain counts instead of each unique key.

Sketches are probabilistic data structures that approximate statistics about a data set. Histograms can be replaced with sketches to improve selectivity estimate accuracy.

Sampling can be done to create smaller tables to estimate selectivities.

**Single-relation query planning**

Choose access method (sequentital scan, binary search, index scan). Order predicate evaluation. Simple heuristics are good enough for OLTP.

Query planning for <u>OLTP</u> are easy because "sargable" (Search Argument Able). Usually just picking the best index. Joins are almost always on foreign keys with low cardinality.

**Multi-relation query planning**

<u>Bottom-up optimization</u>

Use static rules to perform initial optimization. Then use dynamic programming to determine the best join order for tables using divide-and-conquer.

For each logical operator, come up with access paths and evaluate for the best access path.

![](images/Pasted%20image%2020221027211500.png)

![](images/Pasted%20image%2020221027211605.png)

![](images/Pasted%20image%2020221027211634.png)

<u>Top-down optimization</u>

Invoke rules to create new nodes and traverse down the tree. Keep track of the global best plan and do not search if the current access path has more cost than the global plan.

Treat properties of the data as first class entities during planning (For example - Input must be sorted means hash join should not be used).

![](images/Pasted%20image%2020221027213102.png)