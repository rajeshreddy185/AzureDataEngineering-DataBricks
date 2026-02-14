Lakehouse


Lakehouse is combination of data lake , data warehouse to simplify, accelerate and unify enterprise data solutions.
Data engineers, data scientists, analysts, and production systems can all use the data lakehouse as their single source 
of truth, providing access to consistent data and reducing the complexities of building, maintaining, and syncing 
many distributed data systems


A data lakehouse provides scalable storage and processing capabilities for modern organizations that want to avoid 
isolated systems for processing different workloads, like machine learning (ML) and business intelligence (BI). 

Data lakehouses often use a data design pattern that incrementally improves, enriches, and refines data as it moves 
through layers of staging and transformation. Each layer of the lakehouse can include one or more layers. This pattern 
is frequently referred to as a medallion architecture.


The Databricks lakehouse uses two additional key technologies:

Delta Lake: an optimized storage layer that supports ACID transactions and schema enforcement.
Unity Catalog: a unified, fine-grained governance solution for data and AI.


Data ingestion
At the ingestion layer, batch or streaming data arrives from a variety of sources and in a variety of formats. (CSV, JSON, Images) like S3 or ADLS buckets to /mnt/raw. I
This first logical layer provides a place for that data to land in its raw format. As you convert those files to Delta
tables, you can use the schema enforcement capabilities of Delta Lake to check for missing or unexpected data, converting 
files to Delta tables is the he gold standard for this is Auto Loader. It is the bridge between your "Raw" files and your
"Bronze" Delta table.
Schema Inference: Auto Loader doesn't just "guess" the schema; it samples the data and builds a schema for you.
Schema Evolution: If a new column appears in your raw JSON tomorrow, Auto Loader can automatically add it to your Delta table without crashing the job.
You can use Unity Catalog to register tables according to your data governance model and required data isolation 
boundaries. Unity Catalog allows you to track the lineage of your data as it is transformed and refined, as well as 
apply a unified governance model to keep sensitive data private and secure.



