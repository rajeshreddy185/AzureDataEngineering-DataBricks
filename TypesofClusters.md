#### Types Of Clusters

In Databricks, "clusters" are the engines that run your code. In 2026, the platform has simplified these into
two main categories: Classic Compute (where you manage the hardware) and 
Serverless Compute (where Databricks manages everything).

Here are the primary types of clusters you will encounter:
1. **All-Purpose Clusters** (Development)
   1. These are your "daily drivers." You use them while writing code in notebooks or exploring data.
   2. Best for: Ad-hoc analysis, developing pipelines, and testing machine learning models.
   3. Shared vs. Dedicated: You can create a cluster for just yourself (Single User) or one that a whole team can use 
      simultaneously (Shared).
   4. Billing: Usually charged at a higher rate because they stay "on" even when you aren't running a cell 
      (unless you set auto-termination).

2. **Job Clusters (Production)**
   1. These are "ephemeral" clusters—they only exist for the duration of a specific task.
   2. Best for: Automated workflows, scheduled ETL jobs (Bronze) --> Silver --> (Gold), and production pipelines.
   3. How it works: When a scheduled job starts, Databricks creates the cluster; 
   4. when the job finishes, the cluster is immediately destroyed.
   5. Cost Efficiency: They are roughly 50% cheaper than All-Purpose clusters because they don't support
      interactive notebook features.
   
#### 3. SQL Warehouses (Analytics)

Previously called SQL Endpoints, these are clusters optimized specifically for SQL queries and BI tools 
(like Power BI or Tableau).

**Best for**: Data Analysts and BI Engineers who don't need Python or Scala.
**T-Shirt Sizing**: Instead of picking RAM and CPU, you pick a size like 2X-Small, Small, Medium, or Large.
**Photon Engine**: They always run on Photon, a vectorized engine that makes SQL queries run 10x–20x faster.

#### 4. Serverless Compute (Modern Standard)

In 2026, this is the recommended "no-ops" option. You don't pick VMs or scaling rules; you just run your code.

**Start-up Time**: Starts in 2–6 seconds (compared to 3–5 minutes for Classic clusters).
**Management**: No need to configure auto-scaling or termination. Databricks handles the 
                "warm pool" of instances behind the scenes.
Usage: Available for Notebooks, SQL Warehouses, and Workflows (Jobs).

| Cluster Type | Use Case | Lifecycle | Scalability |
|--------------|----------|-----------|-------------|
| All-Purpose  | Interactive Dev | Persistent (Manual start/stop) | Manual/Auto-scale |
| Job          | Production ETL | Ephemeral (Start/Stop with Job) | Fixed or Auto-scale |
| SQL Warehouse | BI & Dashboards | Persistent or Serverless | Rapid Auto-scale |
| Serverless   | Everything | On-Demand (Instant) | Fully Automated |


##### Which one should you use ?

**For Writing Code**: Use a Serverless Notebook or a small Single Node All-Purpose Cluster.

**For Running the Pipeline**: Once your notebooks are ready, schedule them as a Workflow using a Job Cluster.