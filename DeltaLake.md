#### Delta Lake

Delta Lake is the "brain" of the storage. It’s essentially a layer on top of your Parquet files that adds a Transaction 
Log (the _delta_log folder).

**1. The Reliability Layer (Data Integrity)**

* ACID Transactions: Delta Lake ensures your operations are Atomic (all or nothing), Consistent, Isolated, and Durable.
* Real-world scenario: If your cluster crashes halfway through writing the 1.5 million rows,
                       Delta Lake won't leave you with a "half-written" mess. It simply won't "commit" the transaction,
                       and the table remains in its last healthy state.
* Schema Enforcement: This is the "gatekeeper." If you try to insert a string into a column that is defined as an 
                      integer, Delta Lake will throw an error and stop the write. This prevents the "Data Swamp" problem 
                      where messy data ruins your pipelines.
* Merge (UPSERT): In standard Spark, updating a single row means rewriting the whole table. In Delta, you use MERGE.
* Logic: "If the loan_id matches, update the status; if it doesn't match, insert it as a new record."


**2. The Performance Layer (Speed)**

* OPTIMIZE: Over time, your storage gets cluttered with thousands of tiny files (the "Small File Problem"). 
            Running OPTIMIZE takes those tiny files and "compacts" them into large, efficient 1GB files. It makes 
            reading your data significantly faster.
* Data Skipping: Delta Lake automatically stores the Min/Max values for the first 32 columns of every file.
                 How it works: If you query WHERE colm_val > 50000, Databricks looks at the metadata first. 
                 If a file's Max is 40,000, it skips that entire file without reading it.
* Z-ORDER: This is a physical "clustering" of data. If you frequently filter by address State and grade, 
           you run OPTIMIZE ... ZORDER BY (addrState, grade).
          The result: It physically moves rows with the same state and grade into the same files,
                      making Data Skipping incredibly powerful.

**3. The "Time Machine" (Auditing)**
* Time Travel: Every change to a Delta table creates a new Version.
               Why use it? If you accidentally delete a table, you can query VERSION AS OF 5 or TIMESTAMP AS OF 
              '2026-01-01' to see the data exactly as it was back then.
* VACUUM: This is the "Trash Collector." While Time Travel is cool, keeping every old version forever costs money in 
           storage. VACUUM deletes data files that are no longer referenced by the current version of the table and 
           are older than a retention threshold (default is 7 days).
* **Warning**: Once you VACUUM, you can no longer "Time Travel" back to those deleted versions.

**4. Advanced Data Flow**
* Change Data Feed (CDF): Think of this as "Log Streaming." It records every Insert, Update, and Delete at a row level.

* Use case: If your data "Silver" table gets updated, the "Gold" table can use CDF to see only the specific 
            rows that changed, rather than re-scanning the whole Silver table. It's the secret to high-efficiency 
            pipelines.


| Feature | Keywords | Business Benefit |
| --- | --- | --- |
| ACID | All-or-nothing | No corrupted data. |
| Z-ORDER | High Cardinality | Faster filters (e.g., searching by member_id). |
| OPTIMIZE | Compaction | Fixes the "Small File" slow-down. |
| VACUUM | Clean-up | Saves money on ADLS storage costs. |
| CDF | Row-level changes | Incremental updates to downstream tables. |