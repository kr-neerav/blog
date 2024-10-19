+++
title = 'Lakehouse'
date = 2024-10-18T05:52:57-07:00
draft = true
+++
## Introduction
* Lakehouse is characterized by 1) open direct access data format such as parquet and orc 2) first class support for ML 3) state of art performance
* Evolution of analytics/DWH
* first generation systems which were DWH had challenges like 1) coupled storage and compute thus increasing cost with data growth 2) no support for unstructured data 3) ineffective data flow for ML over ODBC
* second generation of systems were two tier to include data lake and DWH. Raw data sits in data lake and a subset of data is curated and pushed to DWH. This addresses challenges in first generation but created new challenges like 1) additional staleness in DWH due to an additional ETL step to move data from data lake to DWH 2) additional complexity of maintainng data in sync between data lake and DWH i.e. moving data in both directions DWH -> Data lake for ML 3) no data management capabilities like transactions, caching, indexes in data lake.
### Existing steps towards Lakehouse
* major DWH added support for external tables in parquet and orc format. this allows warehouse to query datalake using the same sql engine but does not make data lake tables easier to manage.
* investment in SQL engines like Spark SQL, Presto, Hive allows to run SQL queries directly on data lake, however this does not address data management challenges with data lake.
## Lakehouse architecture
* lakehouse are especially good fit for cloud environments due to decoupled storage and compute and integration with different computing engines.
* lakehouse stores data in low cost object store using an open data format like parquet or orc
* implements a metadata layer on top of objects to define which objects are part of which table version.
* this allows lakehouse to implement transactional features like transactions, table versioning in metadata layer while keeping all data in object store. new open table formats have come up for this like Delta Lake, Apache Hudi, Apache Iceberg
* An initial attempt to do this was with Hive ACID which tracks metadata in OLTP database, thus allowing transactions. New systems like Delta Lake store the metadata information in the object store itself, thus supports tables with billions of objects. This however limits the rate of transactions due to high latency of object stores.
* The open table formats allow transactions, time travel. They are also easy to adopt to existing data lakes, given the data is stored in open data formats already in data lake.
* Caching: processing nodes can cache frequently used data locally in faster storage like SDD, given experience of caching provided by DWH. The dataframe APIs used in ML and other data processing allow this.
* Auxilary data: store statistics about objects in the metadata layer thus allowing compute engine to skip files e.g min/max value for a given column in the object. using bloom filter based index to identify if a key is present in an object. 
