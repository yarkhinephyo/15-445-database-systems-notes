**Database seminars**: Mondays @ 4:30 pm https://db.cs.cmu.edu/seminar2022

**Database**: Collection of inter-related data.

**Database systems**: Software that stores and analyzes information in a database.

**Early DBMS**: Expensive computers mean tight coupling between physical and logical layers.

**Relational Model**: Database abstraction based on relations with physical implemention left up to DBMS. Relational model is independent of any query language implementation. SQL is the de facto standard.

**Relation**: Unordered set that contain relationship of attibutes. No duplicate tuples.

**Tuple**: A set of attribute values. Identified by a relation's *primary key*. In practice, there is auto-generation of unique integer keys.

**Relational calculus**: Query specifies only what data is wanted.

**Relational algebra**: Query specifies high level strategy to find the desired results.

1. SELECT - Relation with a subset of tuples that satisfies a selection predicate.
2. PROJECTION - Generate a relation with only specified attributes.
3. UNION - Relation that contains all tuples that appear in either one or both.
4. INTERSECTION - Relation with tuples that appear in both.
5. DIFFERENCE - Relation with tuples that appear in first but not second relation.
6. PRODUCT - Relation with all possible combinations of tuples.
7. JOIN - Relation that contains all tuples that contains all tuples that are a combination of two tuples with a common value for one or more attributes. This is different from INTERSECTION because in JOIN, schemas do not have to be the same.