###### Cluster Policies (now often called Compute Policies)
Cluster Policies (now often called Compute Policies) are the "Guardrails" of a Databricks workspace.
In a organisation, you won't have the freedom to pick any massive VM you want—you will be assigned a policy that 
limits your choices to save money and ensure security.

Think of a Cluster Policy as a Template with rules. 
Instead of seeing 50 different settings when you create a cluster, you might only see 2 or 3.

**Why ?**

Cost Control: Prevents a developer from accidentally starting a 100-node cluster that costs ₹50,000/hour.
Enforced Governance: Automatically attaches mandatory tags (like Project: LendingClub or Owner: Finance) so the billing
                    team knows who to charge.

Simplicity: Reduces the "too many buttons" problem. Most users just want a "Small" or "Medium"
            cluster without worrying about RAM or Disk types.

**How ?**
Admins write these policies using a JSON definition. Here is a simplified version of a policy that forces a cluster to
shut down after 20 minutes and limits workers to a maximum of 5.
```json
{
  "autotermination_minutes": {
    "type": "fixed",
    "value": 20,
    "hidden": true
  },
  "autoscale.max_workers": {
    "type": "range",
    "maxValue": 5,
    "defaultValue": 2
  },
  "spark_version": {
    "type": "regex",
    "pattern": "14\\..*",
    "defaultValue": "14.3.x-scala2.12"
  },
  "custom_tags.Team": {
    "type": "fixed",
    "value": "DataEngineering"
  }
}
```

fixed: The user cannot change this.
range: The user can choose a value between the Min and Max.
hidden: The user can't even see this setting in the UI; it's applied automatically.

Organisations are moving away from managing VMs entirely. Instead of a "Cluster Policy," they use Serverless Budget Policies.
Instead of picking nodes, you set a Credit Limit (e.g., "This project cannot spend more than 10 DBUs per day").
If the limit is reached, Databricks automatically pauses the compute for that specific user.
