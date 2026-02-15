#### Data Quality


Data quality is no longer just a "check" at the end of a script; 
it is a declarative part of the architecture. For any project, you will primarily choose between Delta Live Tables (DLT) 
Expectations (built-in) and Great Expectations (GX) (external/heavyweight).


1. Delta Live Tables (DLT) ExpectationsThis is the "Databricks-Native" way. 
   You define rules directly in your SQL or Python code, and the platform handles the enforcement and 
   reporting automatically.

| Action | Keyword | Behavior | Use Case |
| --- | --- | --- | --- |
| Warning | expect | Keeps the record but logs the failure. | "Non-critical fields (e.g., desc is empty)." |
| Drop | expect_or_drop | Deletes the record from the target table. | "Cleaning data (e.g., loan_amnt is negative)." |
| Fail | expect_or_fail | Stops the entire pipeline. | "Critical integrity (e.g., member_id is null)." |

```python


@dlt.table
@dlt.expect_or_drop("valid_interest_rate", "int_rate > 0 AND int_rate < 100")
@dlt.expect_or_fail("has_member_id", "member_id IS NOT NULL")
def silver_loans():
    return dlt.read("bronze_loans")
```

2. Great Expectations (GX)GX is a massive open-source framework. Think of it as the "Scientific Grade" validation tool.
   It is often used outside of the main pipeline for deep auditing or compliance. 
Expectation Suite: A JSON file containing hundreds of "expectations" (e.g., expect_column_mean_to_be_between).
Data Docs: Automatically generates a clean HTML website showing your team exactly what passed and what failed.
Industry Standard: If your company needs to prove to a regulator that the data is 99.9% accurate, 
GX is the tool you use.

3.  Strategy: 
    "Quarantine Pattern"In a real project, you don't just "drop" bad data—you Quarantine it. 
     This allows you to fix the data and re-insert it later.
      How to build it in Databricks:Ingest to Bronze.
      Split the data:
           Silver_Table: Where expectations == True.
           Quarantine_Table: Where expectations == False.
   Alert: If the Quarantine_Table gets more than 5% of the daily records, send a Slack/Email alert.
   Innovation: Lakehouse MonitoringDatabricks now has Lakehouse Monitoring, which sits on top of Unity Catalog.
   Auto-Profiling: It automatically watches your tables. 

5. Drift Detection: It alerts you if the average interest rate suddenly jumps from 12% to 40% (which might mean a source
   system bug).No Code: You don't write rules; you just click "Enable Monitor" in the UI.
 
| Feature | DLT Expectations | Great Expectations (GX) |
| ------- | ---------------- | ----------------------- |
| Setup | Easy (inside your code) | Complex (requires separate config) |
| Reporting | Integrated Dashboard | "HTML 'Data Docs'" |
| Performance | High (built into Spark) | Medium (extra processing step) |
| Best For | Daily ETL Pipelines | Auditing & Compliance |