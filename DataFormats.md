#### Data Formats


1. **Delta Lake** "Parquet files + A Transaction Log."

Delta Lake is not a file format itself; it is a storage layer that sits on top of Parquet files. 
It solves the biggest problem with data lakes: reliability.
It stores your data as Parquet files, but it adds a _delta_log folder. This log tracks every insert, delete, and update.

**Properties:**
* ACID Transactions: Prevents corrupted data if a job crashes halfway.
* Time Travel: You can query SELECT * FROM table VERSION AS OF 1 to see what data looked like yesterday.
* Speed: Uses "Z-Ordering" and data skipping to run queries 10x–100x faster than raw Parquet.
* Best For: Everything inside Databricks (Silver & Gold layers)

2. **Parquet** "The Engine under the Hood."

If Delta Lake is the car, Parquet is the engine. It is an open-source, columnar file format.

**Columnar Storage**: Instead of storing data row-by-row (like CSV), it stores it column-by-column.
This means if you only select the price column, it doesn't even read the name or date columns from the disk.
**Schema Enforcement**: It knows that id is an Integer and price is a Double. You cannot put text into a number column.
**Compression**: It is highly compressed (Snappy by default), saving huge amounts of storage space.
**Best For**: Storing large datasets efficiently when you don't need the transaction features of Delta 
(e.g., archiving old data).

3. **CSV** (Comma Separated Values) "The Excel of Data."

**Row-Based**: Reads line by line. Slow for analytics because to find one column, you have to read the whole row.
**No Schema**: Everything is just text. "100" could be a number or a string; the system has to guess.
**No Compression**: Bloated file sizes.
**Best For**: Data Exchange. It is the universal language. If you are sending data to a non-technical client or 
              receiving a dump from a legacy system, it will be CSV.

4. **JSON** (JavaScript Object Notation) "The Language of the Web."

**Hierarchical**: Can store nested data (lists inside lists). Great for representing complex objects like a "Customer"
who has multiple "Addresses."
**Flexible**: You can add new fields to any record without breaking the others.
**Slow**: It is text-based and very verbose (it repeats the column name for every single row).
**Best For**: API Responses and NoSQL databases (like MongoDB or CosmosDB). 
              In Databricks, you usually ingest JSON (Bronze) and immediately convert it to Delta (Silver).


**5. Avro** "The Streaming Specialist."

**Row-Based (Binary)**: Unlike CSV/JSON, it is binary (not human-readable) and compressed.
**Schema Evolution**: The schema is stored with the file. It is famous for handling schema changes (e.g., adding a
                        field) very gracefully.
**Best For**: Kafka and Streaming. Because it is row-based, it is very fast at writing one record at a time 
              (perfect for a stream of events), but slower for reading large analytical queries (where Parquet wins).


| Format | Storage Type | Human Readable? | Schema Support | Best Use Case |
|--------|--------------|-----------------|----------------|---------------|
| Delta Lake | Columnar | No | Enforced + Evolution | Primary Storage (Silver/Gold layers) |
| Parquet | Columnar | No | Enforced | Long-term Archive / Raw Storage |
| Avro | Row-Based | No | Strong (Stored in file) | Kafka / Streaming Ingestion |
| CSV | Row-Based | Yes | None (Inferred) | Manual Uploads / Excel Exports |
| JSON | Row-Based | Yes | Flexible | Web APIs / NoSQL Data |