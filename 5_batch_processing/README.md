# 5_batch_processing

## Batch vs Streaming

There are 2 ways of processing data:
* ***Batch processing***: processing _chunks_ of data at _regular intervals_.
    * Example: processing taxi trips each month.
        ```mermaid
        graph LR;
            a[(taxi trips DB)]-->b(batch job)
            b-->a
        ```
* ***Streaming***: processing data _on the fly_.
    * Example: processing a taxi trip as soon as it's generated.
        ```mermaid
        graph LR;
            a{{User}}-. gets on taxi .->b{{taxi}}
            b-- ride start event -->c([data stream])
            c-->d(Processor)
            d-->e([data stream])
        ```

## Pyspark
`pyspark.ipynb`: Create spark session, read csv files, create spark cluster, create spark dataframes

## Actions vs Transformations

Some Spark methods are "lazy", meaning that they are not executed right away. 

These lazy commands are called ***transformations*** and the eager commands are called ***actions***. Computations only happen when actions are triggered.
```python
df.select(...).filter(...).show()
```

List of transformations (lazy):
* Selecting columns
* Filtering
* Joins
* Group by
* Partitions
* ...

List of actions (eager):
* Show, take, head
* Write, read
* ...

## Resilient Distributed Datasets (RDDs)

***Resilient Distributed Datasets*** (RDDs) are the main abstraction provided by Spark and consist of collection of elements partitioned accross the nodes of the cluster.

Dataframes are actually built on top of RDDs and contain a schema as well, which plain RDDs do not.

## Running Spark in the Cloud

Upload parquet files to Cloud Storage:

```bash
gsutil -m cp -r <local_folder> gs://<bucket_name/destination_folder>
```

Go to the [Google's Cloud Storage Connector page](https://cloud.google.com/dataproc/docs/concepts/connectors/cloud-storage) and download the corresponding version of the connector. The version tested for this lesson is version 2.5.5 for Hadoop 3; create a `lib` folder in your work directory and run the following command from it:

```bash
gsutil cp gs://hadoop-lib/gcs/gcs-connector-hadoop3-2.2.5.jar gcs-connector-hadoop3-2.2.5.jar
```

This will download the connector to the local folder.

`pyspark_gcs.ipynb` file: Configure Spark, read in remote data and create local spark cluster.

## Create Spark cluster in [Standalone Mode](https://spark.apache.org/docs/latest/spark-standalone.html) 

Go to your Spark install directory from a terminal and run:

```bash
./sbin/start-master.sh
```
Open the main Spark dashboard using `localhost:8080`

At the very top of the dashboard the URL for the dashboard should appear; copy it and use it in your session code like so:

```python
spark = SparkSession.builder \
    .master("spark://<URL>:7077") \
    .appName('test') \
    .getOrCreate()
```
* Note that we used the HTTP port 8080 for browsing to the dashboard but we use the Spark port 7077 for connecting our code to the cluster.

Run a worker from the command line by running the following command from the Spark install directory:

```bash
./sbin/start-worker.sh <master-spark-URL>
```
## Submitting Spark jobs with Spark submit
`spark_sql.py` file: submit with ->
```bash
spark-submit \
    --master="spark://<URL>" \
    my_script.py \
        --input_green=data/pq/green/2020/*/ \
        --input_yellow=data/pq/yellow/2020/*/ \
        --output=data/report-2020
```

## Dataproc Cluster

[Dataproc](https://cloud.google.com/dataproc) is Google's cloud-managed service for running Spark and other data processing tools such as Flink, Presto, etc.

We would normally choose a `standard` cluster, but you may choose `single node` if you just want to experiment and not run any jobs.

Leave all other optional settings with their default values. After you click on `Create`, it will take a few seconds to create the cluster. You may notice an extra VM instance under VMs; that's the Spark instance.

### Running a job with the web UI

In Dataproc's _Clusters_ page, choose your cluster and un the _Cluster details_ page, click on `Submit job`. Under _Job type_ choose `PySpark`, then in _Main Python file_ write the path to your script (you may upload the script to your bucket and then copy the URL).

Now press `Submit`. 

### Running a job with the gcloud SDK

Allow `Dataproc Administrator` role in IAM.

Submit a job from the command line:

```bash
gcloud dataproc jobs submit pyspark \
    --cluster=<your-cluster-name> \
    --region=europe-west6 \
    gs://<url-of-your-script> \
    -- \
        --param1=<your-param-value> \
        --param2=<your-param-value>
```
