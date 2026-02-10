#### Uploading Your First Dataset

Find a small CSV on your computer (or just save a few rows from Excel as a .csv). 

**The Upload Process:**
  * On the sidebar, click Catalog.
  * Look for the "+" (Add) button at the top or a button that says Create Table / Add Data.
  * Choose Create or modify table.
  * You’ll see a "Drop files to upload" box. Drag your CSV there.
  * Wait! Once it uploads, Databricks will show you a "File Path" (it usually looks like /Volumes/main/default/... or
    /FileStore/tables/...). Copy that path.

**Reading the File with Spark**

Go back to your Notebook. Create a new cell and paste this code (replace the path with the one you copied):

```python
# We tell Spark: "Go to this file, it has a header, and please figure out the data types"
path = "/FileStore/tables/your_filename.csv" # <--- Paste your path here

df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load(path)
```
# Let's see if it worked
```
    display(df)
    Why these options?
    header="true": Tells Spark the first row is names, not data.
    inferSchema="true": Tells Spark to guess if a column is a Number or a String. Without this,
                        Spark treats everything like text.
```

**The "Catalog" Magic (Permanent Tables)**
    Right now, your data is just floating in your notebook's memory as a DataFrame (df). 
    If you turn off your cluster, you have to run that read code again.

To make it a permanent table that you can query with SQL anytime, run this:

```python

# This saves the data into the Databricks Catalog
df.write.mode("overwrite").saveAsTable("my_first_table")
```

**Switching to SQL**


Databricks is famous because you can switch languages instantly. Create a new cell, type %sql at the very top, 
and run this:


```sql
%sql
SELECT * FROM my_first_table WHERE city = 'New York'
```

