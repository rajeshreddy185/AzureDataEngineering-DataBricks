
**1. Schema Evolution** "The ability to adapt to changing data without crashing."

By default, if you try to append data with new columns to an existing Delta table, Databricks will throw an error to 
protect the table's integrity. Schema Evolution allows you to automatically update the table schema to include these
new columns.

How to use it:
```python
Add the .option("mergeSchema", "true") option to your write command.
```

**2. Compression** "Shrinking data size to save storage and speed up I/O."

In Databricks, compression happens at two levels: File Level and Table Layout Level.

**Level 1**: File Compression (Automatic)
Delta Tables use Parquet files under the hood. By default, these are compressed using Snappy (fast to read/write).

**Level 2**: Layout Optimization (Manual)
Even if individual files are compressed, you might have thousands of tiny files ("small file problem").

The Fix: Run the OPTIMIZE command. This performs Bin-Packing (merging small files into 1GB files).
Z-Ordering: Colocates related data to make filtering faster.

```python
-- Compacts small files AND sorts data by 'region' for faster lookup
OPTIMIZE sales_gold ZORDER BY (region);
```

**3. Column Pruning** "Reading only the columns you need."

This is an automatic optimization by the Catalyst Optimizer. If your Parquet file has 100 columns (A, B, C... Z) but 
your query only asks for column A, Spark will only read column A from the disk. It physically ignores the 
other 99 columns.

How to trigger it: Just select what you need.
Parquet is a Columnar Storage Format. Data is stored by column, not by row.
```python
# Spark sees you only need 'id' and 'amount'
# It will NOT read 'address', 'customer_name', 'logs', etc. from the disk.
df = spark.read.table("sales_gold").select("id", "amount") 
display(df)
```

**4. Predicate Pushdown** "Filtering data before it enters memory."

This is the most critical optimization for large datasets. Instead of reading all the data into Spark's memory and 
then filtering it, Spark "pushes" the filter down to the storage layer.
Parquet files have a "Footer" (metadata) that stores the Min and Max values for every column in that file.
Query: WHERE id = 500
File 1 Footer: min_id=1, max_id=100 --> SKIP (Spark doesn't even open this file).
File 2 Footer: min_id=400, max_id=600 --> READ.

How to verify it:Use .explain() and look for PushedFilters.
```python
df = spark.read.table("sales_gold").filter("amount > 1000")
df.explain()
```

Output to look for:

== Physical Plan ==
... PushedFilters: [IsNotNull(amount), GreaterThan(amount, 1000)]


How to Break it (Don't do this):

If you cast a column inside the filter, Spark often cannot push it down.
Bad: filter("cast(amount as string) = '1000'") --> Reads all data, then casts, 
then filters.Good: filter("amount = 1000") --> Pushes filter to disk.

Concept,        Action,                                         Benefit
Schema          Evolution,"option(""mergeSchema"", ""true"")"   Prevents pipeline failures when upstream data adds columns.
Compression     OPTIMIZE ... ZORDER BY                          Reduces storage cost and speeds up queries by skipping irrelevant files.
Column          Pruning,.select(...)                            Saves I/O by reading only required columns (Vertical slicing).
Predicate       Pushdown,.filter(...)                           Saves I/O by skipping files that don't match criteria (Horizontal slicing).