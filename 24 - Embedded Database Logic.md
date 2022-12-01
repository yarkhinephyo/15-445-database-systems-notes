**User-defined function**

Treated as functions to be used inside the database (E.g - how concat() is used). Takes in input arguments and return a result.

If multiple applications use the same logic, there is less room for error. Some logic is difficult to convey with SQL too.

There are <u>SQL Functions</u> and <u>External Programming Languages.</u>

<u>Pros</u> - For the programmers, some logic is easier to implement in UDF (Iterative programming).

<u>Cons</u> - If a query uses the functions, the DBMS cannot optimize well because the functions are treated as a black box. DBMS also only uses a single thread due to the same reason so there will be less parallelization.

![](images/Pasted%20image%2020221201123251.png)

**Stored procedure**: Performs more complex logic inside a DBMS. Not normally used inside a SQL query. Some DBMS distinguish UDFs vs Stored Procedures but not all.

**Trigger**: If some event occurs, DBMS invokes a UDF.

1. Event to monitor.
2. Scope of the event.
3. When the UDF should fire.

```sql
CREATE TRIGGER foo_updates
	BEFORE UPDATE ON foo FOR EACH ROW 
	EXECUTE PROCEDURE log_foo_updates();
```

**Change notification**

Publisher-subscriber pattern to send message to an external entity. Keywords used are LISTEN and NOTIFY.

![](images/Pasted%20image%2020221201125659.png)

**User-defined types**: Data types that is defined by the application developer that DBMS can store natively.

**View**: Virtual table containing the output from a SELECT query. It can be accessed similar to a real table. It executes the view query each time so the data is always updated.

**Materialized view**: The output from SELECT query is retained so that it does not run every time. DBMS automatically updates materialized views when the underlying tables change.