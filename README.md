# Architecture

![architecture diagram](/images/01.png)

* [New York's Taxi and Limousine Corporation's Trip Records Dataset](https://github.com/DataTalksClub/nyc-tlc-data/releases/): the dataset used.
* [Spark](https://spark.apache.org/): analytics engine for large-scale data processing (distributed processing).
* [Google BigQuery](https://cloud.google.com/products/bigquery/): serverless _data warehouse_ (central repository of integrated data from one or more disparate sources).
* [Airflow](https://airflow.apache.org/): workflow management platform for data engineering pipelines. In other words, a pipeline orchestration tool.
* [Kafka](https://kafka.apache.org/): unified, high-throughput,low-latency platform for handling real-time data feeds (streaming).

# Data

Yellow and green cabs:

| Columns               | Definition | Example             |
| --------------------- | ---------- | ------------------- |
| VendorID              |            | 2                   |
| lpep_pickup_datetime  |            | 2021-01-01 00:15:56 |
| lpep_dropoff_datetime |            | 2021-01-01 00:19:52 |
| store_and_fwd_flag    |            | N,                  |
| RatecodeID            |            | 1                   |
| PULocationID          |            | 43                  |
| DOLocationID          |            | 151                 |
| passenger_count       |            | 1                   |
| trip_distance         |            | 1.01                |
| fare_amount           |            | 5.5                 |
| extra                 |            | 0.5                 |
| mta_tax               |            | 0.5                 |
| tip_amount            |            | 0                   |
| tolls_amount          |            | 0                   |
| ehail_fee             |            |                     |
| improvement_surcharge |            | 0.3                 |
| total_amount          |            | 6.8                 |
| payment_type          |            | 2                   |
| trip_type             |            | 1                   |
| congestion_surcharge  |            | 0                   |

Taxi Zone: 

| Columns      | Definition | Example        |
| ------------ | ---------- | -------------- |
| LocationID   |            | 1              |
| Borough      |            | EWR            |
| Zone         |            | Newark Airport |
| service_zone |            | EWR            |


[Download link](https://github.com/DataTalksClub/nyc-tlc-data/releases/)