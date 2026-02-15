#### Integrating Azure Key Vault


Integrating Azure Key Vault with Databricks using a Secret Scope is a way to manage credentials. This ensures your 
storage keys are never hardcoded in notebooks and are protected by enterprise-grade security.


Step 1: Store the Key in Azure Key Vault

First, you need to store your ADLS Gen2 Access Key as a secret.
Navigate to your Azure Key Vault in the Azure Portal.
Go to Secrets > Generate/Import.
Name: Give it a clear name (e.g., adls-access-key).
Value: Paste your ADLS Storage Account Key.
Click Create.

Step 2: Configure Key Vault Permissions
Databricks needs permission to "read" from your Key Vault.
RBAC Model: In the Key Vault Access Control (IAM) tab, assign the Key Vault Secrets User role to the "AzureDatabricks"
            enterprise application.
Access Policy Model (Legacy): Go to Access Policies > Create, select "Get" and "List" permissions, and select the 
                              AzureDatabricks service principal.

Step 3: Create the Secret Scope in Databricks
This acts as a "bridge" between Databricks and the Key Vault.
Launch your Databricks workspace.
In your browser's address bar, append #secrets/createScope to your workspace URL:
https://adb-xxxx.xx.azuredatabricks.net/#secrets/createScope
Scope Name: Enter a name (e.g., keyvault-scope).
DNS Name & Resource ID: Go to your Key Vault’s Properties tab in the Azure Portal to copy these.
Click Create.

Step 4: Use the Secret in your Code
Now you can fetch the key securely using dbutils.secrets.get(). 
Databricks will automatically mask the value in your notebook outputs with [REDACTED].
Method A: Batch/Interactive (Notebook Level)
```python

# Fetch the secret from the scope
storage_key = dbutils.secrets.get(scope="keyvault-scope", key="adls-access-key")

# Use it in Spark configuration
spark.conf.set(
    "fs.azure.account.key.mystorageaccount.dfs.core.windows.net", 
    storage_key
)

# Read data
df = spark.read.format("delta").load("abfss://container@mystorageaccount.dfs.core.windows.net/data")

```


Method B: Cluster-Level (Production Best Practice)

To avoid running authentication code in every notebook, put the secret reference directly into the Cluster Spark Config.
Go to Compute > Cluster > Edit > Advanced Options.

In Spark Config, add:

```

fs.azure.account.key.mystorageaccount.dfs.core.windows.net {{secrets/keyvault-scope/adls-access-key}}
Note: The {{secrets/scope/key}} syntax tells Databricks to fetch the secret automatically during cluster startup.

```

Once Method B is set up in the Cluster settings, your notebook code becomes incredibly simple. 
You do NOT need dbutils.secrets.get or spark.conf.set.


 **Unity Catalog (UC)**
 If we uses Unity Catalog (the modern "Lakehouse" governance), you don't even use Secret Scopes or Spark Configs for 
 data access anymore.
 How it works for Multiple Teams:
    Storage Credentials: The Admin creates a "Storage Credential" for the whole workspace.
    External Locations: The Admin creates two paths:abfss://lending@storage...  Name it lending_location
    abfss://risk@storage... Name it risk_location
    Permissions: * GRANT READ ON EXTERNAL LOCATION lending_location TO lending_team_group
                    GRANT READ ON EXTERNAL LOCATION risk_location TO risk_team_group
    The Result: A user from the Lending team just writes spark.read.load(path). 
                Databricks checks their Identity (who they are), not the Cluster Config, to see if they have access. 
                This is the most secure method.