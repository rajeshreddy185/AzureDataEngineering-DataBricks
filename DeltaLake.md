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
When you enable CDF on a table, Delta Lake begins recording a version of every change in a private folder (_change_data).
When you query this feed, Spark provides four special metadata columns:
```bash
_change_type: Tells you if the row was an insert, update_preimage (the old value), update_postimage 
              (the new value), or delete.

_commit_version: The version of the table when the change happened.

_commit_timestamp: Exactly when the change was committed.
```
* Without CDF, if you wanted to know what changed between Version 5 and Version 6 of a table, you had to perform a 
   massive diff (comparing the two versions), which is slow and compute-heavy.
* Downstream apps only pull the "delta" (the changes), not the whole table.tells what exactly changed
* Use case: If your data "Silver" table gets updated, the "Gold" table can use CDF to see only the specific 
            rows that changed, rather than re-scanning the whole Silver table. It's the secret to high-efficiency 
            pipelines.
  * Example: 
    Lets take an example of an appstore which want to process the current status of the packages in each and every device
    status can be installed or updated or uninstalled.
    Data flows through three stages. CDF is the "bridge" that makes the hop from Silver to Gold incredibly efficient.
    Bronze (Raw Inbox): Stores every raw log as it arrives (e.g., "Device D1 sent a 'heartbeat' for Spotify at 10:00 AM").
    Silver (Cleaned State): A MERGE cleans the Bronze data to show the current status of every device.
    Gold (Business Value): Uses CDF to pull only the updates from Silver to calculate metrics, like "How many users 
    uninstalled Spotify today?"

    Step 1: Ingesting to Silver (The MERGE)
            Every hour, you take new records from Bronze and merge them into Silver. This keeps your Silver 
            table up-to-date.
    
      ```SQL
    
      -- Step 1: Clean raw logs into a 'Current Status' table
      MERGE INTO silver_device_data AS target
      USING bronze_raw_logs AS source
      ON target.device_id = source.device_id AND target.package = source.package
      WHEN MATCHED THEN
        UPDATE SET target.status = source.status, target.last_updated = source.timestamp
      WHEN NOT MATCHED THEN
        INSERT (device_id, package, status, last_updated) 
        VALUES (source.device_id, source.package, source.status, source.timestamp);
      ```

      Step 2: Enabling the "Hidden Camera" (CDF)
        You must enable CDF on the Silver table. This tells Delta Lake to start recording the specific rows that were 
        touched by the MERGE above.
      ``` ALTER TABLE silver_device_data SET TBLPROPERTIES (delta.enableChangeDataFeed = true); ```
      ```python
          # Read only the 'receipts' of the last changes
            changes_df = (spark.read.format("delta")
                .option("readChangeFeed", "true")
                .option("startingVersion", version) # Version where last job stopped
                .table("silver_device_data"))

                # Filter to get only the final 'New' status of the device
                updates_for_gold = changes_df.filter("_change_type IN ('insert', 'update_postimage')")

    ```
NOTE: 
    When you use foreachBatch, you are telling Spark: "I know this is a stream, but for the next 30 seconds, pretend
    this small chunk of data is a regular, static DataFrame so I can run a MERGE on it."

**5. Liquid Clustering**
Issues  with Z-order:

* We have to define Z-order columns upfront, with the change in query patterns we have to update z-order column which will
  essentially requires re write of entire data in table.
* Usually had to combine it with Hive-style partitioning (e.g., /year/month/day/), which leads to "small file" problems 
  if you over-partition.So, If data wasn't evenly distributed, Z-Order struggled.
* With Z-Ordering, Spark cannot just write what it has. It must ensure that data points which are "close" to each other 
  in the multidimensional space (like language and timestamp) end up in the same physical file.Scan all the data you 
  are optimizing means a massive Network Shuffle.Spilling to disk if the data being sorted is larger than the worker's RAM.
* Z-Ordering is not incremental. If you have a table with 1,000 files and you add 10 new files, running OPTIMIZE ... ZORDER 
  BY doesn't just sort the 10 new files. To maintain, it often has to rewrite the existing files to incorporate the new 
  data into the sorted structure.


* You can cluster by any column (up to 4) regardless of cardinality.
* It is Implemented Delta Lake Liquid Clustering to replace legacy Z-Ordering, moving from fixed, manual partition 
  strategies to a dynamic, self-tuning data layout that adapts to changing query patterns.
* Liquid Clustering will eliminate "partition evolution" challenges, enabling high-performance data skipping on 
  high-cardinality columns without the overhead of manual re-partitioning.
* It clusters data more locally and incrementally.


**6. DELETION VECTOR**

* To understand the importance of deletion vector we need to know how delete works in parquet or delta log
* first it reads all the rows
* then find the row to delete
* delete that row 
* write this to entire new file (mark old file deleted in delta log)

This is called copy on write this is expensive and slow.

Deletion Vector 
It follows merge on read, which does instead rewriting whole data, it creates tiny auxiliary files, which acts as a notes
to the original files.
At write time, when you delete instead writing all the data, it just adds a bit mask which tell this rows are gone.
At read time, when we read the data the file which reads data file, deletion vector simultaneously which tells the rows to skip.

You can't leave Deletion Vectors forever. Eventually, if a file has too many "ignored" rows, it becomes inefficient to 
keep reading the dead weight.

This is where your REORG TABLE or OPTIMIZE command comes back in. When you run these, Databricks finally does the heavy 
lifting: it takes the original file + the Deletion Vector and collapses them into a fresh, clean file with the rows 
permanently removed.

**how deletion vector is fast when we have to eventually do reorg table or optimize?**

The old "Copy-on-Write" (CoW) world, if you were deleting a single row from a 1GB file, that file was "locked" for 
the duration of the rewrite. If another job tried to update that same file, you’d get a Concurrent Modification Exception.
With Deletion Vectors: The "delete" is just a tiny metadata write (the bitmask). It takes milliseconds. This minimizes the 
"window of conflict," allowing multiple jobs to hit the same table without crashing each other.

By using REORG or OPTIMIZE later, you are choosing when to pay the "compute tax."OPTIMIZE once a week (or during off-peak 
hours) to "squash" those vectors.


| Feature | Keywords | Business Benefit |
| --- | --- | --- |
| ACID | All-or-nothing | No corrupted data. |
| Z-ORDER | High Cardinality | Faster filters (e.g., searching by member_id). |
| OPTIMIZE | Compaction | Fixes the "Small File" slow-down. |
| VACUUM | Clean-up | Saves money on ADLS storage costs. |
| CDF | Row-level changes | Incremental updates to downstream tables. |