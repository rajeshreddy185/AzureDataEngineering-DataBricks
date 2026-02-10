#### Databricks

**What?**
Databricks is a cloud-based Unified Data Analytics Platform. It provides a collaborative environment for Data Engineers,
Data Scientists, and SQL Analysts to work together in one place.

Databricks takes the raw power of Apache Spark, adds the reliability of Delta Lake, the speed of Photon, and the 
collaboration of Notebooks, and wraps it all in a managed cloud service.

Architecturally, it sits on top of your cloud provider (AWS, Azure, or GCP) and manages your clusters, storage, and 
security. It is the birthplace of the "Lakehouse" architecture, a hybrid that combines the best parts of a 
Data Warehouse (fast, structured) and a Data Lake (cheap, flexible).

**Why ?**
Before Databricks, if you wanted to use Spark, you had to be a "plumber" as much as a 
"data person." You had to:

* Manually set up EC2 instances or Virtual Machines.
* Configure networking between nodes.
* Install Spark and all its dependencies.
* Manage security and access controls.
* Scale the cluster manually when the data got too big

**What problems is it solving?**
**A. The "Data Silo" Problem**
In many companies, the Data Engineers use one tool, the Data Scientists use another (Python/R), 
and the Business Analysts use a third (SQL).

The Solution: Databricks provides Collaborative Notebooks. Multiple people can work in the same notebook simultaneously
using Python, SQL, Scala, or R.

**The "Small File" and Reliability Problem**
Standard Data Lakes are messy. If a Spark job fails halfway through, your data becomes corrupted. If you write millions
of tiny files, your queries become slow.

The Solution: Delta Lake. Databricks created Delta Lake to bring ACID transactions to Big Data. It ensures that if a job
fails, nothing is written, and it automatically compacts small files into large, efficient ones.

**The Performance Gap**
Raw Apache Spark is written in Scala/Java and runs on the JVM. This can be slow for certain operations.

The Solution: Photon. Databricks built a completely new execution engine called Photon, written in C++, which is 
significantly faster than standard Spark for SQL-heavy workloads.


**Governance and Security**
Tracking who has access to what data in a giant cloud storage bucket is a nightmare.

The Solution: Unity Catalog. This provides a single layer of governance where you can grant permissions 
(e.g., GRANT SELECT ON TABLE...) across your entire organization, regardless of whether the data is a file or a table.




