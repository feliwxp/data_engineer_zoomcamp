# 2_data_ingestion

- Ingesting data to local Postgres with Airflow
- Ingesting data to GCP with Airflow 
- Transfer data from multiple sources to GCS using GCP's transfer service

## Data Lake vs Data Warehouse

* Data Processing:
  * DL: The data is **raw** and has undergone minimal processing. The data is generally unstructured.
  * DW: the data is **refined**; it has been cleaned, pre-processed and structured for specific use cases.
* Size:
  * DL: Data Lakes are **large** and contains vast amounts of data, in the order of petabytes. Data is transformed when in use only and can be stored indefinitely.
  * DW: Data Warehouses are **small** in comparison with DLs. Data is always preprocessed before ingestion and may be purged periodically.
* Nature:
  * DL: data is **undefined** and can be used for a wide variety of purposes.
  * DW: data is historic and **relational**, such as transaction systems, etc.
* Users:
  * DL: Data scientists, data analysts.
  * DW: Business analysts.
* Use cases:
  * DL: Stream processing, machine learning, real-time analytics...
  * DW: Batch processing, business intelligence, reporting.

In essence, Data Lake was born to collect any potentially useful data that could later be used in later steps from the very start of any new projects.

## ETL vs ELT

When ingesting data, DWs use the ***Export, Transform and Load*** (ETL) (data transformed before storing) model whereas DLs use ***Export, Load and Transform*** (ELT) (data stored before transformation).

## Airflow

Airflow pipeline (Download, parquetize and ingest in the same pipeline):

```
(web)
  ↓
DOWNLOAD
  ↓
(csv)
  ↓
PARQUETIZE
  ↓
(parquet) ------→ UPLOAD TO S3
  ↓
UPLOAD TO GCS
  ↓
(parquet in GCS)
  ↓
UPLOAD TO BIGQUERY
  ↓
(table in BQ)
```
Airflow architecture: 
![airflow architecture](/images/02.png)

## Setting up Airflow with Docker

### Full version

1. In `airflow` subdirectory, download the official Docker-compose YAML file:
```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.2.3/docker-compose.yaml'
```

2. Alter the `x-airflow-common` service definition inside the `docker-compose.yaml` file to point to custom Docker image, google credentials.

### Lite version

1. Remove the `redis`, `airflow-worker`, `airflow-triggerer` and `flower` services.
1. Change the `AIRFLOW__CORE__EXECUTOR` environment variable from `CeleryExecutor` to `LocalExecutor` .
1. At the end of the `x-airflow-common` definition, within the `depends-on` block, remove these 2 lines:
    ```yaml
    redis:
      condition: service_healthy
    ```
1. Comment out the `AIRFLOW__CELERY__RESULT_BACKEND` and `AIRFLOW__CELERY__BROKER_URL` environment variables.

### Execution
1. Build the image:
    ```bash
    docker-compose build
    ```
2. Initialize configs:
    ```bash
    docker-compose up airflow-init
    ```
3. Run Airflow
    ```bash
    docker-compose up -d
    ```
Access the Airflow GUI using `localhost:8080`. Username and password are both `airflow` .

## Creating a DAG

```python
with DAG(dag_id="my_dag_name") as dag:
    op1 = DummyOperator(task_id="task1")
    op2 = DummyOperator(task_id="task2")
    op1 >> op2
```

## Ingesting data to local Postgres with Airflow

This DAG will download the NYC taxi trip data and ingest it to local Postgres.

- `ingest_data.py` file: connect to database and ingest NYC taxi trip data to local Postgres
- `data_ingestion_local.py` DAG file: DAG that will download NYC taxi data and call the `ingest_data.py` script
- `docker-compose.yaml` file: build the Airflow image with `docker-compose build` and initialize the Airflow config with `docker-compose up airflow-init`. Start Airflow by using `docker-compose up` and on a separate terminal, find out which virtual network it's running on with `docker network ls`. It most likely will be something like `airflow_default`.
- `docker-compose-lesson2.yaml` file: add the network info here and comment away the pgAdmin service to reduce the amount of resources we will consume. Run `docker-compose -f docker-compose-lesson2.yaml up`.

Select the DAG from Airflow's dashboard and trigger it. 

Use `docker-compose down` to shut down both Airflow and Postgres terminals.

## Ingesting data to GCP with Airflow

This DAG will download the NYC taxi trip data, convert it to parquet, upload it to a GCP bucket and ingest it to GCP's BigQuery.

- `data_ingestion_gcs_dag.py` DAG file

Select the DAG from Airflow's dashboard and trigger it. 

## GCP's Transfer Service

To transfer data from multiple sources to Google's Cloud Storage. This is useful for Data Lake purposes.

Transfer Service _jobs_ can be created via 2 ways: 
- GCP UI: using _Transfer Service | cloud_ submenu
- Terraform: `transfer_service.tf` file in `1_intro` folder
