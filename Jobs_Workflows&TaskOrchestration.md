#### **Notebooks vs workflows**

**1. The Mental Model**
* Think of a restaurant kitchen.
* The Notebook is the Recipe (The instructions: "Chop onions, fry meat").
* The Workflow (Job) is the Kitchen Manager (The coordinator: "First chop onions. Wait until onions are done. 
* Then fry meat. If meat burns, throw it away and restart.").

**2. Notebooks vs. Workflows**

| Feature | Notebooks | Workflows (Jobs) |
|---------|-----------|-----------------|
| Primary Goal | Interactive Coding & Exploration. | Orchestration & Scheduling. |
| State | Stateful: Variables stay in memory (good for debugging). | Stateless: Starts fresh every time (good for reliability). |
| Dependency | Manual execution (Cell 1 → Cell 2). | Defined DAG (Task A → Task B → Task C). |
| Compute | "All-Purpose Cluster: Expensive, always on." | "Job Cluster: Cheap, spins up for the job, dies after." |
| Best For | "Development, EDA, prototyping." | "Production pipelines, nightly runs." |


**3. The "Notebook Orchestration" Trap (Anti-Pattern)**


```python
# Master_Notebook
dbutils.notebook.run("notebook_A")
dbutils.notebook.run("notebook_B") # Wait for A to finish

```

Why this is bad in Production:

* Hidden Logic: The Databricks UI sees this as 1 Task. If notebook_B fails, the UI just says "Master Notebook Failed."
* You have to dig into logs to find where.
* **wasted Money:** The cluster stays active waiting for notebook_A to finish before it even looks at notebook_B.
* No Visuals: You lose the nice green/red boxes graph that shows you exactly what broke.
* The Fix: Use Databricks Workflows to define these dependencies explicitly in the UI.

**4. Jobs Orchestration: The Industry Standard**
In a real company, you rarely click "Run Now" in the UI. Here is the maturity curve of orchestration:

Level 1: The UI Scheduler (Beginner)
You go to the "Workflows" tab.

* You manually click "Create Job."
* You point to your Notebooks.
* **Pros**: Fast, easy.
* **Cons**: Hard to version control. If someone accidentally deletes the job, it's gone forever.


4. The Wheel Pattern
Imagine you have 50 different notebooks for 50 different data pipelines.You have a function clean_currency_column(df) 
that fixes dollar signs.You copy-paste this function into all 50 notebooks.ou find a bug in the function. 
Now you have to open 50 notebooks and fix it manually. Instead of pasting the code, you write it once in a Python 
file (e.g., in VS Code or PyCharm), package it into a library (a .whl file), and install that library on your cluster.

```
Repository: src/common_utils/cleaning.pyBuild: python setup.py bdist_wheel $\rightarrow$ my_library-1.0.whl
Cluster: Installs my_library-1.0.whl.Notebook:Pythonfrom my_library.cleaning import clean_currency_column.

# If we update the library, this code gets the fix automatically!
df = clean_currency_column(df)
```
4. The "Modern" Evolution: Databricks Repos (Git Folders)
While the Wheel Pattern is still the gold standard for shared libraries (code used by multiple different teams),
Databricks introduced a feature called "Files in Repos" that makes this easier for single teams.

The Hybrid Approach (Most Common Today):
You don't always need to build a .whl file anymore.

You put your .py files in the same Git Folder as your notebook.

Databricks now allows you to import them directly!

```
The Structure:

/Workspace/Users/you/my-project/
├── ingestion_pipeline.ipynb   <-- Your Notebook
├── utils/
│   ├── __init__.py
│   └── cleaning.py            <-- Your Python Module
Inside ingestion_pipeline.ipynb:

Python

# No wheel build needed! It just works now.
from utils.cleaning import clean_currency_column
```

Jobs schedule Azure Databricks notebooks, SQL queries, and other arbitrary code. Databricks Asset Bundles allow you to
define, deploy, and run Databricks resources such as jobs and pipelines programmatically. Git folders let you sync Azure
Databricks projects with a number of popular git providers.

