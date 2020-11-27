# Introduction to a modern data tech stack 


## Why Snowflake?
Our SQL infrastructure is starting to strain around the seams, especially for reporting needs, and especially in the Risk environment. Analysis Services cubes help to some extent, but top out at 2 billion unique values. Risk has tables with upwards of 100 billion rows!

Snowflake is a modern data warehouse, and is accessible to anyone who can write SQL. It scales horizontally (I'll explain this below!) and when you need to increase resources, this is basically instant. You just pay a cost for each warehouse size increase.

## How does Snowflake speed up queries?
Firstly, Snowflake separates storage from compute. This is really a rather neat trick - your data sits inside S3 buckets, cheap storage essentially, which are accessible by multiple compute nodes ("warehouses" in Snowflake terms)
This means that your data load process can be totally separated from your querying process.
How does this work?

The storage is a series of S3 buckets, each one with approx. 10K rows. When you load new data, new buckets are created, not interfering with currently running queries.

![storage](/Snowflake/Diagrams/Architecture.Speedup-Storage.png)

Each bucket contains statistics about the max and min values in each column.
At query time, only the buckets needed to fulfil the query are queried - and these are then loaded into a nearline SSD cache on the compute node. This also has the effect of massively parallelising the IO load.

![Query](/Snowflake/Diagrams/Architecture.Speedup-FirstQuery.png)

The compute node then performs aggregations, calculations, etc.

On subsequent queries, the cache will be hit exclusively

![Cache](/Snowflake/Diagrams/Architecture.Speedup-Cache.png)

In addition to this, there exist multiple "warehouses" (defined in your connection), each of which doesn't compete with other warehouses.

![Scale-out](/Snowflake/Diagrams/Architecture.Speedup-Scale-out.png)

![test](/Snowflake/Diagrams/airflow.drawio.svg)