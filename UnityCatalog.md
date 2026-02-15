#### `Data governance and secure data sharing`


Unity Catalog provides a unified data governance model for the data lakehouse. Cloud administrators configure and 
integrate granular level access control permissions for Unity Catalog, and then Azure Databricks administrators can 
manage permissions for teams and individuals. Privileges are managed with access control lists (ACLs) through either 
user-friendly UIs or SQL syntax, making it easier for database administrators to secure access to data without 
needing to scale on cloud-native identity access management (IAM) and networking.


The lakehouse makes data sharing within your organization as simple as granting query access to a table or view. 
For sharing outside of your secure environment, Unity Catalog features a managed version of Delta Sharing.

While it does handle access (RBAC), its value comes from four other major pillars that are 
essential for modern data engineering

1. The Hierarchy: Catalogs & Schemas
Unity Catalog uses a three-level namespace to organize data. This is much cleaner than the old "Hive Metastore" days.

Catalog: The highest level of isolation (e.g., lending_club_prod). It usually represents a business unit or environment.
Schema (Database): A logical grouping inside a catalog (e.g., bronze, silver, gold).
Table/Volume: The actual data objects.

2. Automated Data Lineage (The "GPS" of your data)
Unity Catalog automatically tracks where data comes from and where it goes.

**How it works**: If you create a Delta table in Databricks, Unity Catalog "watches" the Spark code. It builds a visual 
map showing that Table A was used to create Table B, If a report in regional office shows wrong numbers, a data engineer
uses Lineage to trace back through 10 tables to find exactly which source file had the error.

2. Data Discovery & Search
Without Unity Catalog, you have to ask colleagues "where is the customer table?" With it, you get a "Google for your data."
You can search for keywords like "Sales" or "Revenue" and find all relevant tables across all your workspaces.
It stores descriptions, tags (like PII for sensitive data), and column types so you understand the data before you even query it.

3. Delta Sharing (Open Sharing)
It allows you to share data outside of your Databricks workspace without copying it.
If your company needs to share a Delta table with a partner, who uses Snowflake or Power BI, you can use Delta Sharing 
through Unity Catalog. They get access to the live data securely without you ever having to send them a CSV file

4. System Tables (Operational Auditing)
Unity Catalog provides built-in "System Tables" that tell you everything about your environment:

5. The Plumbing: Storage Credentials & External Locations
In UC, you don't use raw Access Keys. You use a structured "handshake."

Storage Credentials: This is the "Identity." It stores an Azure Managed Identity or Service Principal that has physical 
                     access to your ADLS Gen2 bucket.
External Locations: This is the "Path + Identity." It links a Storage Credential to a specific URL
                    (e.g., abfss://container@account.../data/).
Pro Tip: Once you have an External Location, you can grant users the CREATE TABLE permission on that location without
         ever giving them the storage key.

6. Advanced Security: Row-Level & Column Masking
This allows you to share the same table with different people while showing them different data.

Row-Level Security (Filters)
You can restrict which rows a user sees based on their group.

Example: A "Karnataka Manager" only sees loans where state = 'KA', while a "National Manager" sees all rows.

```SQL

CREATE FUNCTION state_filter(addr_state STRING)
RETURN IF(is_account_group_member('admin'), true, addr_state = 'KA');

ALTER TABLE lending_club.gold.loans SET ROW FILTER state_filter ON (addr_state);

```

Column Masking
You can hide sensitive data like PII (Personally Identifiable Information).

Example: Mask the member_id or email so only authorized users see the full value.

```SQL

CREATE FUNCTION email_mask(email STRING)
RETURN IF(is_account_group_member('hr_group'), email, '****@****.com');

ALTER TABLE lending_club.silver.members ALTER COLUMN email SET MASK email_mask;
```
**Billing**: You can see exactly which user or job is spending the most money on your clusters.

**Auditing**: It records every single query ever run—who ran it, when, and if it was successful. This is mandatory for 
banking and healthcare companies.

UC follows an Explicit Grant model. By default, users have access to nothing.

**USE CATALOG / USE SCHEMA**: Mandatory first step. Without these, a user can't even "see" that a table exists.
**SELECT**: Standard read access.
**MODIFY**: Permission to update/delete data.
**BROWSE** : Allows users to find a table in the UI and see its metadata/lineage even if they don't have permission 
               to read the actual rows yet. This is great for data discovery.
