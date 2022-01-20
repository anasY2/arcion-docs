---
title: Apache Spark
weight: 14
bookHidden: false
---

# Destination Apache Spark

The extracted `replicant-cli` will be referred to as the `$REPLICANT_HOME` directory in the proceeding steps.

## I. Setup Connection Configuration

1. From `$REPLICANT_HOME`, navigate to the sample connection configuration file:

   ```BASH
   vi conf/conn/spark.yaml
   ```

2. Make the necessary changes as follows:

   ```YAML
   type: SPARK

   host: local #Replace local with your Apache Spark host

   storage-location: "/tmp/parquet"
    storage-type: PARQUET

    max-retries: 10 #Enter the maximum number of times Replicant can re-attempt a failed operation
    retry-wait-duration-ms: 1000 #Enter the time Replicant should wait between each re-try of a failed operation

    ```

## II. Setup Applier Configuration

1. From `$REPLICANT_HOME`, navigate to the applier configuration file:
    ```BASH
    vi conf/dst/spark.yaml
    ```
2. Make the necessary changes as follows:

    ```YAML
    snapshot:
      threads: 16 #Maximum number of threads Replicant should use for writing to the targe
      # 
      batch-size-rows: 5_000
      txn-size-rows: 1_000_000

      #If bulk-load is used, Replicant will use the native bulk-loading capabilities of the target database
      bulk-load:
        enable: true
        type: FILE   # PIPE, FILE
        serialize: true #Set to true if you want the generated files to be applied in serial/parallel fashion
    ```

For a detailed explanation of configuration parameters in the applier file, read [Applier Reference]({{< ref "/docs/references/applier-reference" >}} "Applier Reference").