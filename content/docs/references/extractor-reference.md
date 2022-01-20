---
title: Extractor Configuration
weight: 1
---

# Extractor Configuration

The extractor configuration file contains all the parameters that Replicant uses while extracting data from the source database. While it is not necessary to modify most of the  parameters, they can be adjusted to optimize the extraction as necessary. The sample extractor configuration file is located in the Replicant release download folder. The path to the sample extraction configuration file in the release folder is: `conf/src/source_database_name.yaml` The sample file is broken up into two sections- snapshot and realtime. The extraction configurations for the initial data capture must be specified under `snapshot` and the configurations for realtime replication must be specified under `realtime`

## Snapshot Mode

Replicant can run on the default extractor configurations for the data snapshot. Thus, changing the snapshot extraction configurations is not required. However, depending on the replication job, adjusting or specifying the parameters explained below may help optimize Replicant.

**snapshot**
  1. **threads**: Maximum number of threads replicant will use for data extraction from source

  2. **fetch-size-rows**: Maximum number of records/documents fetched by replicant at once from the source system

  3. **lock**: This is the option to perform object locking on the source database. By default, object locking is disabled.
     * **enable**: false #"false" is the default
     * **scope**: table #"table" is the default
     * **force**: false #"false" is the default
     * **timeout-sec**: 5 #"5" is the default
  * **Note**: This is parameter is irrelevant for source databases which do not support object locking such as Mongo and Cassandra.

  4. **min-job-size-rows**: Replicant chunks tables/collections into multiple jobs for replication. This configuration specifies the minimum size for each such job. The   minimum size specified has a positive correlation with the memory footprint of Replicant.

  5. **max-jobs-per-chunk**: The maximum number of jobs in a chunk.  

  6. **split-key**: Replicant uses this configuration to split the table/collection into multiple jobs in order to perform parallel extraction. The specified split key  column must be of numeric or timestamp type. Splitting the work for source data extraction using the split-key provided here significantly optimizes Replicant if:
          * The split-key has uniform data distribution in the table (and there is minimal data skew in the values of split-key)  .
          * An index is present on split-key on the source system.

  7. **split-method**: Replicant supports two split methods.
        * RANGE: Replicant splits the work in a range-based manner by uniformly distributing the split-key value ranges between the MIN and MAX values.
        * MODULO: The split key must be of numeric type for this split-method. Each extraction job is assigned a JobID of 0 to JOB-CNT -1. Each job then pulls data from the source where MOD(split-key, JOB-CNT) = JobID.

  8. **extraction-method**: Replicant supports two extraction methods: QUERY and TPT ((Default method is QUERY). TPT stands for Teradata Parallel Transporter utility (TPT) and is currently only supported for Teradata as a source.

  9. **tpt-num-files-per-job**:  Tpt-num-files-per-job is only applicable when the extraction-method is TPT. This parameter indicates how many CSV files should be exported by each TPT job (default  value is 16).

  10. **fetch-PK**: Option to fetch (and replicate) primary key constraints for tables (By default this is true)

  11. **fetch-UK**: Option to fetch (and replicate) primary key constraints for tables (By default this is true)

  12. **fetch-FK**: Option to fetch (and replicate) unique key constraints for tables (By default this is true)

  13. **fetch-Indexes**: Option to fetch (and replicate) indexes for table. (By default this is false)

  14. **fetch-user-roles**: Option to fetch (and replicate) user/roles. (The default is true for homogeneous pipelines, but false otherwise)

  15. **normalize**: This parameter is only supported for Mongo Database as a source. The configuration is used to configure the normalization of data.
      * **enable**: true/false (The default is false)
      * **de-duplicate**: true/false (Default is false) To de-duplicate data during normalization
      * **extract-upto-depth**: The depth upto which the mongoDB document should be extracted (Default is INT_MAX)

  14. **fetch-schemas-from-system-tables**: Option to use system tables to fetch schema information. By default, the value is true, and this option is enabled. If disabled, schemas need to be provided using --src-schemas.

  15. **per-table-config**: This section can be used to override certain configurations in specific tables if necessary.
      * **catalog**: <catalog_name>
      * **schema**: <shema_name>

      * **tables**: Multiple tables can be specified using the following format
        1. **<table_name>**:
            * **max-jobs-per-chunk**:
            * **split-key**: Used to specify split-key for the specified table
            * **split-method**: Used to override the global split method in the specific table
            * **extraction-method**: Used to override the global extraction method in the specified table
            * **tpt-num-files-per-job**: Used to override the global Num files per TPT job in the specified table
            * **row-identifier-key**: Here you can specify a list of columns which uniquely identify a row in this table. If a table does not have a PK/UK defined and if the table has a subset of columns that can uniquely identify rows in the table, it is strongly recommended to specify that subset of columns as a row-identifier key. Specifying an identifier can significantly improve the performance of incremental replication of this table.
            * **extraction-priority**: Priority for scheduling extraction of table. Higher value is higher priority. Both positive and negative values are allowed. Default priority is 0 if unspecified.
            * **normalize**: This parameter is only supported for Mongo Database as a source. The configuration is used to configure the normalization of data.
              * **de-duplicate**: true/false (Default set to false) To de-duplicate data during normalization
              * **extract-upto-depth**:Used to specify the depth upto which the mongoDB document should be extracted.(Default is INT_MAX)

         As many tables as necessary can be configured for and specified under each other using the format above. For example configuring two tables would look as follows:
         ```YAML
          <table_name>:
             max-jobs-per-chunk:
             split-key:
             split-method:  
             extraction-method:
             pt-num-files-per-job:
             row-identifier-key:
             extraction-priority:
             normalize: #Only for Mongo Database as source
               de-duplicate:
               extract-upto-depth:

          <table_name>:
            max-jobs-per-chunk:
            split-key:
            split-method:
            extraction-method:
            pt-num-files-per-job:
            row-identifier-key:
            extraction-priority:
            normalize: #Only for Mongo Database as source   
              de-duplicate:
              extract-upto-depth:               
         ```

## Heartbeat table

For real-time replication, you must create a heartbeat table. Replicant periodically updates this table at a configurable frequency. This table helps forcefully flush the cdc log for all committed transactions so that Replicant can Replicate them. A few things to note:
    * The table must be created in the catalog/schema which is to be replicated by replicant.
    * The user configured for Replicant must be granted INSERT, DELETE, and UPDATE privileges to the heartbeat table.
    * For simplicity, it is recommended to use the exact DDL provided in the extractor configuration set up under your source database's instructional documentation to create the heartbeat table

The configurations for the heartbeat table must be specified under the ```realtime``` section of the Extractor Configuration as explained in the proceeding section.

## Realtime mode

Unless you have given your heartbeat table different table and column names than the ones used in the provided command to create the table, the extractor configurations under the ```realtime``` section do not need to be changed. However, changing certain parameters may improve replication performance depending on the use case. Each configuration that can be altered is explained below.

**realtime**
  1. **threads**: Maximum number of threads to be used by replicant for real-time extraction

  2. **fetch-size-rows**: Maximum number of records/documents fetched by replicant at once from the source system

  3. **Fetch-duration-per-extractor-slot-s**: Number of seconds a thread should continue extraction from a given replication channel/slot. (E.g. For MongoDB source, a mongo shard is one replication channel). After a thread spends extracting from a particular replication channel, it gets scheduled to process another replication channel. This option is relevant and important to avoid starvation from any replication channel when the number of threads provided is less than the number of replication channels.

  4. **Heartbeat**:
     * **enable**: Enable heartbeat mechanism (required for realtime replication)
     * **catalog**: Catalog of the heartbeat table
     * **schema**: Schema of the heartbeat table
     * **table-name[20.09.14.3]**: Name of the heartbeat table
     * **column-name[20.10.07.9]**: Name of column in heartbeat table (Has only 1 column)
     * **Interval-ms**: Interval at which heartbeat table should be updated with the latest timestamp (milliseconds since epoch) by replicant

  5. **fetch-interval-s[20.07.16.1]**: Interval in seconds after which Replicant will try to fetch the CDC log
      * **Note**: Not all realtime sources currently support fetch-interval-s option.

  6. **start-position[20.09.14.1]**: This config is used in real-time mode to specify the starting log position for real-time replication. The form for providing start position for different databases is as follows:
      * **DB2**:
        1. commit-time: timestamp from source Db2 in UTC. E.g. Following query will give the timestamp in UTC: SELECT CURRENT TIMESTAMP - CURRENT TIMEZONE AS UTC_TIMESTAMP FROM SYSIBM.SYSDUMMY1

      * **MySQL**:
        1. log: log file from where replication should start
        2. position: position within the log file from where replication should start

      * **MongoDB**:
        1. timestamp-ms: timestamp in milliseconds from where replication should start. This corresponds to the timestamp field “ts” in oplog. Note that “ts” contains timestamp in seconds which needs to be multiplied by 1000.
        2. Increment: (Optional) The increment for the given timestamp. This corresponds to the increment section in the “ts” filed of oplog

      * **Oracle**:
        1. start-scn: The scn from which replication should start
      * **Others**:
        1. timestamp-ms: Timestamp from which replication should start

   7. **idempotent-replay[20.09.14.1]**: The possible values are ALWAYS/ NONE/ NEVER. ALWAYS means always do INSERT as REPLACE. NEVER means publish operation as is. NONE means default replicant behavior.

   8. **normalize[20.09.14.12]**: This parameter is only supported for Mongo Database as a source. The configuration is used to configure the normalization of data.
        * **enable**: true/false (Default set to false) To enable normalization
        * **de-duplicate**: true/false (The default is false) To de-duplicate data during normalization
        * **extract-upto-depth**: To specify depth upto which mongoDB document should be extracted (Default is INT_MAX)

   9. **per-table-config**: This section can be used to override certain configurations in specific tables if necessary.
        * **catalog**: <catalog_name>
        * **schema**: <shema_name>

        * **tables**: Multiple tables can be specified using the following format
           1. **<table_name>**:
               * **normalize[20.09.14.12]**: This parameter is only supported for Mongo Database as a source. The configuration is used to configure the normalization of data.
               * **de-duplicate**: true/false (The default is false) To de-duplicate data during normalization
               * **extract-upto-depth**: true/false (The default is false) To de-duplicate data during normalization

               As many tables as necessary can be configured for and specified under each other using the format above. For example configuring two tables would look as follows:
               ```YAML
                <table_name>:
                   normalize: #Only for Mongo Database as source
                     de-duplicate:
                     extract-upto-depth:

                <table_name>:
                  normalize: #Only for Mongo Database as source     
                  de-duplicate:
                  extract-upto-depth:                
               ```

## Delta snapshot mode

Blitzz supports a third mode of replication called delta-snapshot. Delta-snapshot is required when the source database does not provide access to CDC log but the customer is interested in real-time replication (for example Teradata). The delta snapshot is a recurring snapshot which replicates the *delta* of the records which have been inserted/updated since the previous delta snapshot iteration. The following describes parameters of the `delta-snapshot` section of the Extractor configuration file.

**delta-snapshot**
  1. **threads**: Maximum number of threads replicant should use for data extraction from source.

  2. **fetch-size-rows**: Maximum number of records/documents Replicant fetches at once from the source system.

  3. **lock**: Option to do object locking on source. No locking is done on MongoDB. Below are the parameters and example values:

      i. **enable**: false

      ii. **scope**: table

      iii. **force**: false

      iv. **timeout-sec**: 5

  4. **min-job-size-rows**: Replicant chunks Tables/collections into multiple jobs for replication. This configuration specifies a minimum size for each such job. This has a positive correlation with the memory footprint of replicant.

  5. **max-jobs-per-chunk**: Determines the maximum number of jobs Replicant should create for each source table/collection.

  6. **split-key**: Replciant uses this configuration to split a table into multiple jobs in order to do parallel extraction. It lets you specify a global `split-key`. Replicant will use this column to perform parallel data extraction from each table that has this column (unless this configuration is overridden in `per-table-config` for this table).
  
  7. **split-method**[20.05.12.3] : Replicant supports two split methods:

      i. **RANGE** : Performs a range based splitting of work by uniformly distributing the split-key value ranges between the MIN and MAX values.

      ii. **MODULO** : Split key must be only of numeric type for this split-method. Each extraction job is assigned a `JobID` of `0` to `JOB-CNT -1`. Each job then pulls data from the source where `MOD(split-key, JOB-CNT) =  JobID`.
  
  8. **extraction-method**[20.07.02.1]: Replicant supports following extraction methods (default set to QUERY):
      * QUERY

      * TPT (Teradata Parallel Transporter utility; currently supported only for Teradata as source)

      * CSVLOAD (Supported only for Cassandra as source)

      * DSBULK (Supported only for Cassandra as source)

      * COPY (Supported only for Greenplum as source)

9. **tpt-num-files-per-job**[20.07.02.1]: Relevant when extraction-method is TPT. This config indicates how many CSV files each TPT job should export (default value set to 16).    

10. **delta-snapshot-key**: Tables requiring incremental replication must have a NON-NULL numeric/timestamp column which is updated on each insert/update on each row of that table. The value of this column must be monotonically increasing and non-repeatable. We call such a column a `delta-snapshot-key`. This configuration lets you specify that key. Replicant uses this column to perform its incremental replication for each table being replicated that has this column (unless this configuration is overridden in `per-table-config` for this table). 

  {{< hint "warning" >}}
  `delta-snapshot-key` is deprecated. Please use `delta-snapshot-keys` instead.
  {{< /hint >}}

11. **delta-snapshot-keys**[21.12.02.1]: This config lets you specify one or more `delta-snapshot-keys`. Tables requiring incremental replication must have a NON-NULL numeric/timestamp column which is updated on each insert/update on each row of that table. The value of this column must be monotonically increasing and non-repeatable. We call such a column a `delta-snapshot-key`. This configuration lets you specify multiple `delta-snapshot-key`s. Replicant uses this column to perform its incremental replication for each table being replicated that has this column (unless this configuration is overridden in per-table-config for this table).

    Each column in this list  acts as an individual `delta-snapshot-key` and updating on any of the columns in this list will trigger replication. For example:

    ```YAML
    delta-snapshot-keys: [col1, col2, col3]`  # A list of monotonic increasing numeric/timestamp columns which gets new value on each INSERT/UPDATE
    ```

12. **row-identifier-key**: If a table does not have a PK/UK defined on it, then we strongly recommend that you specify a `row-identifier-key`: a single column or a group of columns that are ensured to be unique in the table by your applications. Blitzz leverages this `row-identifier-key` to achieve a much better overall performance for incremental replication (in the absence of PK/UK). This configuration lets you specify a global `row-identifier-key`. Blitzz Replicant makes use of this key to perform certain replication (unless this configuration is overridden in per-table-config for this table).

13. **update-key**: This lets users specify a key that Replicant should use to perform DELETES/UPDATES on the target system under the following scenarios: 
    * PK/UK does not exist.
    * There's no unique `row-identifier-key` present in the table.

    This specifies a single column or a group of columns to be used by replicant.  We strongly recommended that you create an index on `update-key` on the target table explicitly to get better replication performance.

14. **delta-snapshot-interval**: The interval (in seconds) between two incremental replication cycles. This configuration allows you to specify how frequently Blitzz replicant should query the source database and pull incremental changes.

15. **replicate-deletes**: This configuration allows you to disable delete replication. 

16. **delta-snapshot-delete-interval**: This configuration allows you to specify a different interval for delete replication that the one specified for insert/update incremental replication.

17. **native-extract-options**:

    i. **charset**: This configuration allows users to specify the charset to be used with native extraction method. Supported values are ASCII and UTF8.

    ii. **compression-type**: This configuration allows users to specify the compression type to be used with native extraction method. Supported value is GZIP.
  
18. **per-table-config**: Use this section if you want to override certain configuration on a per table (/view/query) basis.

    i. **catalog**: <catalog_name>

    ii. **schema**: <shema_name>

    iii. **tables**:
      1. **<table_name>**:

          a. **split-key**: This configuration allows you to specify table specific `split-key`. If specified, it overrides the global `split-key` configuration.

          b. **split-method**[20.05.12.3]: You can override `split-method` on a per-table basis using this config.
          
          c. **extraction-method**[20.07.02.1]: You can override `extraction-method` on a per-table basis using this config.

          d. **num-jobs**: Number of parallel jobs Replicant will use to extract the rows from a table. This value will override the number of jobs Replicant internally calculates.
          
          e. **num-delete-jobs**[21.02.01.8]: Number of parallel delete jobs that Replicant will use to replicate deletes for a table. This value will override the number of jobs Replicant internally calculates. If you don't specify any value, then Replicant will use the value of `num-jobs` that you specified.

          f. **tpt-num-files-per-job**[20.07.02.1]: Num files per TPT job can be overridden on a per-table basis using this config. 
          
          g. **delta-snapshot-key**: This configuration allows you to specify table specific delta-snapshot-key. If specified, it overrides the global delta-snapshot-key.

           {{< hint "warning" >}} `delta-snapshot-key` is deprecated. Please use `delta-snapshot-keys` instead.{{< /hint >}}

          h. **delta-snapshot-key**s: This configuration allows you to specify table specific delta-snapshot-keys. If specified, it overrides the global delta-snapshot-keys
          
          i. **row-identifier-key**: This configuration allows you to specify table specific `row-identifier-key`. If specified, it overrides the global `row-identifier-key`.
          
          j. **update-key**: This configuration allows you to specify a table specific `update-ke`y. If specified, it overrides the global `update-key`.

          k. **replicate-deletes**: This configuration allows you to enable/disable delete replication individually for a table.

          l. **native-extract-options**:
            1. **charset**: This configuration allows users to specify the charset to be used with native extraction method. Supported values are ASCII and UTF8.

            2. **compression-type**: This configuration allows users to specify the compression type to be used with native extraction method. Supported value is GZIP.

            3. **column-size-map**[20.08.13.9]: This lets users specify column size/length to use while extracting data.