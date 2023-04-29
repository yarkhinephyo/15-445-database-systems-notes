**Physical operator**: Define an execution strategy using an access path. Not always 1:1 mapping from logical operator. For example, inner join (logical) is mapped to hash join (physical).

**Architecture of DBMS that converts query to physical plan**

1. SQL Rewriter (Rare): Convert SQL to SQL string.
2. SQL Parser spits out abstract syntax tree.
3. Binder maps AST to database objects (For example: Table name to table object found in system catalog), and produces a logical plan.
4. Tree Rewriter (Common): Rewrite logical plan.
5. Optimizer estimates cost with cost model and produces a physical plan.

**Logical query optimization steps**

Transform logical plan with pattern matching rules. Increases the likelihood of enumerating the optimal plan in the search.

1. <ins>Split conjunctive predicates</ins> - Decompose predicates into their simplest forms.

![](images/Pasted%20image%2020221026224907.png)

2. <ins>Predicate pushdown</ins> - Move predicate to the lowest applicable point to ensure the DBMS applies the most selective one first.

3. <ins>Replace cartesian products</ins> - Replace cartesian products with inner joins using the join predicates.

![](images/Pasted%20image%2020221026225633.png)

4. <ins>Projection pushdown</ins> - Eliminate redundant attributes before the pipeline breakers, as pipeline breakers disrupt efficient streaming of iterators and are expensive.

![](images/Pasted%20image%2020221026225916.png)

**Nested subqueries**

Rewrite to flatten them or decompose and store result to temporary table.

Correlated queries gets rewritten as a join.

![](images/Pasted%20image%2020221026230839.png)

Uncorrelated queries have results stored in a temporary location.

![](images/Pasted%20image%2020221026231235.png)k

**Expression rewriting**

1. Remove impossible/unnecessary predicates.
2. Merging predicates. (Combining range of values)

**Cost-based search to evaluate equivalent plans**

Enumerate multiple equivalent plans and produce the one with lowest cost. Cost can be physical costs (CPU cycles, cache misses), logical costs (estimate output size per operator), algorithmic costs (complexity of the operator algorithm). 

**Selection cardinality**

Selection of a predicate is the fraction of tuples that qualify.  Assumptions are -

1. Distribution of values is uniform.
2. Predicates are independent.
3. Every tuple in inner table is guaranteed to exist in outer table during join.

The assumptions does not always hold true. For example, a model of a car may only be made by a brand. If there are 10 models and 10 brands, the selectivity is 0.01 with assumption 1 and 2, but it is not true in practice.

<ins>Ways to estimate selectivity</ins>

1. Equai-width or equi-depth histograms can be used to maintain counts instead of each unique key.
2. Sketches are probabilistic data structures that approximate statistics about a data set. Histograms can be replaced with sketches to improve selectivity estimate accuracy.
3. Sampling can be done to create smaller tables to estimate selectivities.

**Single-relation query planning**

Choose access method (sequentital scan, binary search, index scan). Order predicate evaluation. Simple heuristics are good enough for OLTP.

Query planning for <ins>OLTP</ins> are easy because "sargable" (Search Argument Able). Usually just picking the best index. Joins are almost always on foreign keys with low cardinality.

**Multi-relation query planning**

<ins>Bottom-up optimization</ins>

Use static rules to perform initial optimization. Then use dynamic programming to determine the best join order for tables using divide-and-conquer.

For each logical operator, come up with access paths and evaluate for the best access path.

![](images/Pasted%20image%2020221027211500.png)

![](images/Pasted%20image%2020221027211605.png)

![](images/Pasted%20image%2020221027211634.png)

<ins>Top-down optimization</ins>

Invoke rules to create new nodes and traverse down the tree. Keep track of the global best plan and do not search if the current access path has more cost than the global plan.

Treat properties of the data as first class entities during planning (For example - Input must be sorted means hash join should not be used).

![](images/Pasted%20image%2020221027213102.png)