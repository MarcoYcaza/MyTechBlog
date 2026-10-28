---
title: "Vaccum functionality and deletion vectors"
date: 2026-10-28
description: "We'll see briefly how the certain writting operations have an unexpected impact on the vaccum operation"
tags: [Databricks, Delta Lake, VACUUM]
categories: [Data Engineering]
draft: true
---

>This was the instruction: Let's enforce a  7 day retetion policy on our tables. For GDPR purposes no file containing any rows can persist in our storage.  


First things first was to check the state of the tables , with a describe command it was evident that some tables had different window retention. The fix was simple , to set the deleteFileRetentionDuration to 7 days.  

This means when a table experiments an modification for example a deletion , its parquet file that compose this version is recreated without the deleted row, making the original file obsolete. 

But being obsolete doesn't make it automatically dissapear , it needs to be programatically vaccumed. So when running the cleaning, spark checks if the file is eligible to be deleted if its older than the retention period. 


>This is the ideal scenario! But something strange happens when some operations have a default feature enabled called **`Deletion Vecors`**

## Deletion Vectors

Spark has introduced since the version <state version> this feacture that  gets triggered when running certain operations under the MERGE operation. UPSERT and DELETE clauses affect the table but  when ocurr instead of creating a new parquet file containing the change , spark marks the deleted rows in a bitmap (which is another file with a different extention ".dv") so when reconstructing the current state of the table it refers quickly to this bitmap to filter out the rows. This operation doesn't alter the original file, this generally to avoid recreating files when the amount of rows to be dropped is few. 

But this comes with a catch , because if the original file is not obsolete after the deletion , then it never becomes a candidate for vaccuming. 

### Inspect first

On Databricks, use a DESCRIBE EXTENDED on a table

```sql
DESCRIBE EXTENDED catalog.schema.mytable;
```

Then, you'll see in the property column if it has this feature enabled.

[PASTE IMAGE from databricks]


## How to deal with them

If we want to be strict with this, on our vaccum process we'll need to purge the deletion vectors and materialize the changes.   We do that using this command:

```sql
REORG catalog.schema.table PURGE 
```

What this does is to force spark is to take the deltion vectors and apply them on the parquet files, hence the parquet file is recreated and the original now becomes obsolete and the clock starts ticking.

> So I do a REORG + PURGE and then the vaccum cleans it right?

We'll actually no, the PURGE makes spark reflect the changes but its at that very moment where the file becomes a candidate to be eligible for deletion so it needs to wait still the `DurationRetention` until it could be dropped.

## Quick Fix

I see two quick options:

### A. Turn off the deletion vectors feature
This can be done configuring the spark at the cluster, notebook level. Also you can set this as a property for the table itself.  

### B. Set the WindowDuration to half the desired threshold

If I need the obsolete files to be deleted after 7 days , we can restrict this to 3 days (which is more or less half) and now if a file is PURGED on day 3 just after the VACCUM it will still be deleted on the next VACCUM run that will occur still after 3 days which is less than the 7 days but safely no more than that.

My recommendation is to try to turn off the deletion vectors and test the runtime , if there is no evident change on the duration then we do  not actually need them and we can safely continue with our 7 day retention period. But if the performance degrades , then adjusting the retention is the safest and quick way to fix the issue.


Seguimos!