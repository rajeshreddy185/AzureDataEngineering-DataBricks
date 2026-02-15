Delta Live Tables (DLT) is the modern "Auto-Pilot" for Data Engineering.In standard PySpark (what we did before), 
YOU are the driver. You have to tell Spark: "Read this file, check for errors, wait for the previous step,
write to this folder, manage the checkpoint."
In DLT, you just tell Databricks: 
"I want a Silver Table that comes from Bronze and has clean data." DLT figures out the order, manages the dependencies, 
and handles the checkpoints for you.
Here is the breakdown of the three key concepts you asked about.
1. Declarative Pipelines ("The What, Not the How")
   In traditional engineering ("Imperative"), you write scripts that say HOW to do things step-by-step.
2. In DLT ("Declarative"), you simply declare WHAT the result should look like.
Old Way (Imperative):
```python # You have to manually manage the read/write logic
df = spark.read.load("/path/to/bronze")
df_clean = df.filter("id is not null")
df_clean.write.mode("append").save("/path/to/silver")
```
DLT Way (Declarative):
```python
import dlt

# You just define the end state. DLT handles the reading & writing.
@dlt.table
def silver_table():
    return (
        dlt.read("bronze_table").filter("id is not null")
    )
```
Why Industry Loves It: 
    You don't write "Orchestration" code anymore. DLT automatically builds the DAG (Directed Acyclic Graph) based on 
    your function dependencies.


2. Expectations ("The Data Bouncer")
    This is the Killer Feature of DLT.In standard Spark, data quality is hard. You have to write complex if/else logic
    to check for nulls or bad values.DLT has built-in Constraints called Expectations. You can define rules, and DLT 
    enforces them in 3 ways:

| Mode | Command | Behavior | Use Case |
| --- | --- | --- | --- |
| Track | @dlt.expect | Records the error as a metric but lets the data pass. | """I want to know if IDs are null, but don't stop the pipeline.""" |
| Drop | @dlt.expect_or_drop | Removes the bad row from the target table. | """If the 'amount' is negative, that data is garbage. Throw it away.""" |
| Fail | @dlt.expect_or_fail | Stops the entire pipeline immediately. | """If the 'Timestamp' is missing, the whole dataset is invalid. Stop!""" |

```python
@dlt.table
@dlt.expect_or_drop("valid_price", "price > 0")  # Drop rows where price is negative
@dlt.expect("valid_id", "id IS NOT NULL")        # Just warn me if ID is null
def sales_silver():
    return dlt.read("sales_bronze")
```
3. Auto Recovery & MaintenanceIn standard pipelines, if a job crashes halfway through, you have a mess. You might have
    partial data written, or corrupted checkpoints.DLT handles this automatically.
    **Managed Checkpoint**s: DLT tracks exactly which files it has processed. If the cluster crashes, it restarts and picks 
    up exactly where it left off (no duplicates, no missed data). 
    **Schema Evolution**: If the source data adds a new column, DLT can automatically add it to your target table without 
      crashing (if enabled).
    **Auto-Scaling**: DLT can automatically scale the cluster up during heavy loads and down when idle (Enhanced Autoscaling).
    **Hands-On**: The "Full" DLT PipelineTo run this, you cannot just click "Run Cell". You must create a Pipeline in the 
     Workflows tab.The Code (Save this as dlt_pipeline.py):
```python
import dlt
from pyspark.sql.functions import *

# ---------------------------------------------------------
# 1. BRONZE (Ingest Raw Data)
# ---------------------------------------------------------
@dlt.table(
    comment="Raw data from landing zone"
)
def sales_bronze():
    # In DLT, we usually use Auto Loader (cloud_files) for ingestion
    return (
        spark.readStream.format("cloudFiles")
        .option("cloudFiles.format", "csv")
        .load("abfss://raw-data@.../landing/")
    )

# ---------------------------------------------------------
# 2. SILVER (Clean & Validate)
# ---------------------------------------------------------
@dlt.table(
    comment="Clean data with valid prices"
)
@dlt.expect_or_drop("valid_amount", "amount > 0")
@dlt.expect_or_fail("has_timestamp", "transaction_date IS NOT NULL")
def sales_silver():
    return (
        dlt.read("sales_bronze")
        .select(
            col("transaction_id"),
            col("amount").cast("double"),
            col("transaction_date").cast("timestamp")
        )
    )

# ---------------------------------------------------------
# 3. GOLD (Aggregate)
# ---------------------------------------------------------
@dlt.table(
    comment="Daily sales aggregates"
)
def sales_gold_daily():
    return (
        dlt.read("sales_silver")
        .groupBy(to_date("transaction_date").alias("date"))
        .agg(sum("amount").alias("daily_total"))
    )
```
