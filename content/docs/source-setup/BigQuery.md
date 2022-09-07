---
title: Google BigQuery
weight: 13
bookHidden: false
---
# Source Google BigQuery

The extracted `replicant-cli` will be referred to as the `$REPLICANT_HOME` directory in the proceeding steps.

## I. Obtain the JDBC Driver for Google BigQuery

Replicant requires the JDBC driver for Google BigQuery as a dependency. To obtain the appropriate driver, follow the steps below:

- Go to the [JDBC drivers for BigQuery page](https://cloud.google.com/bigquery/docs/reference/odbc-jdbc-drivers#current_jdbc_driver).
- From there, download the [latest JDBC 4.2-compatible JDBC driver ZIP](https://storage.googleapis.com/simba-bq-release/jdbc/SimbaJDBCDriverforGoogleBigQuery42_1.2.25.1029.zip).
- From the downloaded ZIP, locate and extract the `GoogleBigQueryJDBC42.jar` file.
- Put the `GoogleBigQueryJDBC42.jar` file inside `$REPLICANT_HOME/lib` directory.

## II. Set up Connection Configuration

1. From `$REPLICANT_HOME`, navigate to the sample connection configuration file:
    ```BASH
    vi conf/conn/bigquery_src.yaml
    ```

2. Make the necessary changes as follows:
    ```YAML
    type: BIGQUERY

    host: https://www.googleapis.com/bigquery/v2
    port: 443
    project-id: <bigquery_projectID>
    auth-type: 0
    o-auth-service-acc-email: <your_service_account@your_project.iam.gserviceaccount.com>
    o-auth-pvt-key-path: <path_to_oauth_private_key>
    location: US
    timeout: 500

    username: "<your_username>"
    password: "<your_password>"

    max-connections: 20

    max-retries: 10
    retry-wait-duration-ms: 1000
    ```

## III. Set up Extractor Configuration

1. From `$REPLICANT_HOME`, navigate to the Extractor configuration file:

    ```BASH
    vi conf/src/bigquery.yaml
    ```
2. Currently, Arcion only supports snapshot mode for BigQuery as source. So make the necessary changes as follows in the `snapshot` section of the configuration file:

    ```YAML
    snapshot:
      threads: 32
      fetch-size-rows: 10_000

      min-job-size-rows: 1_000_000
    #  max-jobs-per-chunk: 32
      per-table-config:
        - schema: tpch
          tables:
            partsupp:
              split-key: ps_partkey
            supplier:
              split-key: s_suppkey
            orders:
              split-key: o_orderkey
            nation:
              split-key: n_regionkey
    ```

For a detailed explanation of configuration parameters in the Extractor file, read [Extractor Reference]({{< ref "/docs/references/extractor-reference" >}} "Extractor Reference").