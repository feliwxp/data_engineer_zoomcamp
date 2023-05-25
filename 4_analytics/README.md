# 4_analytics

## What is dbt?

***dbt*** stands for ***data build tool***. It's a _transformation_ tool: it allows us to transform process _raw_ data in our Data Warehouse to _transformed_ data which can be later used by Business Intelligence tools and any other data consumers.

dbt also allows us to introduce good software engineering practices by defining a _deployment workflow_:
1. Develop models
1. Test and document models
1. Deploy models with _version control_ and _CI/CD_.

## How does dbt work?

dbt works by defining a ***modeling layer*** that sits on top of our Data Warehouse. The modeling layer will turn _tables_ into ***models*** which we will then transform into _derived models_, which can be then stored into the Data Warehouse for persistence.

A ***model*** is a .sql file with a `SELECT` statement; no DDL or DML is used. dbt will compile the file and run it in our Data Warehouse.

## How to use dbt?

dbt has 2 main components: _dbt Core_ and _dbt Cloud_:
* ***dbt Core***: open-source project that allows the data transformation.
    * Builds and runs a dbt project (.sql and .yaml files).
    * Includes SQL compilation logic, macros and database adapters.
    * Includes a CLI interface to run dbt commands locally.
    * Open-source and free to use.
* ***dbt Cloud***: SaaS application to develop and manage dbt projects.
    * Web-based IDE to develop, run and test a dbt project.
    * Jobs orchestration.
    * Logging and alerting.
    * Intregrated documentation.
    * Free for individuals (one developer seat).

For integration with BigQuery, the dbt Cloud IDE will be used. 

![dbt](/images/03.png)

## Setting up dbt

1. Create 2 new empty datasets in BigQuery: a _development_ dataset and a _production_ dataset. 
2. Create a user account on [dbt homepage](https://www.getdbt.com/).
3. In the IDE windows, press the green _Initilize_ button to create the project files.

Note: dbt models are mostly written in SQL but they also make use of the [Jinja templating language](https://jinja.palletsprojects.com/en/3.0.x/) for templates. 

Example of dbt model:

```sql
{{
    config(materialized='table')
}}

SELECT *
FROM staging.source_table
WHERE record_state = 'ACTIVE'
```

dbt will compile this code into the following SQL query:

```sql
CREATE TABLE my_schema.my_model AS (
    SELECT *
    FROM staging.source_table
    WHERE record_state = 'ACTIVE'
)
```

After the code is compiled, dbt will run the compiled code in the Data Warehouse.

Additional model properties are stored in YAML files. 

## Defining a source and creating a model

We will now create our first model.

We will begin by creating 2 new folders under our `models` folder:
* `staging` will have the raw models.
* `core` will have the models that we will expose at the end to the BI tool, stakeholders, etc.

Under `staging` we will add 2 new files: `sgt_green_tripdata.sql` and `schema.yml`:

* We define our ***sources*** in the `schema.yml` model properties file.
* We are defining the 2 tables for yellow and green taxi data as our sources.

Run the model with the `dbt run` command, either locally or from dbt Cloud.

## Testing dbt models

In dbt, tests are essentially a `SELECT` query that will return the amount of records that fail because they do not follow the assumption defined by the test.

Here's an example test:

```yaml
models:
  - name: stg_yellow_tripdata
    description: >
        Trips made by New York City's iconic yellow taxis. 
    columns:
        - name: tripid
        description: Primary key for this table, generated with a concatenation of vendorid+pickup_datetime
        tests:
            - unique:
                severity: warn
            - not_null:
                severrity: warn
```
* There are 2 tests in this YAML file: `unique` and `not_null`. Both are predefined by dbt.

You may run tests with the `dbt test` command.

## Documentation

Run `dbt docs generate` to generate docs
Run `dbt docs serve` to host docs in dbt Cloud as well or on any other webserver with .

## Deployment

dbt projects are usually deployed in the form of ***jobs***:
* A ***job*** is a collection of _commands_ such as `build` or `test`. A job may contain one or more commands.
* Jobs can be triggered manually or on schedule.
    * dbt Cloud has a scheduler which can run jobs for us, but other tools such as Airflow or cron can be used as well.
* Each job will keep a log of the runs over time, and each run will keep the logs for each command.
* A job may also be used to generate documentation, which may be viewed under the run information.
* If the `dbt source freshness` command was run, the results can also be viewed at the end of a job.

## Continuous Integration

dbt makes use of GitHub/GitLab's Pull Requests to enable CI via [webhooks](https://www.wikiwand.com/en/Webhook). When a PR is ready to be merged, a webhook is received in dbt Cloud that will enqueue a new run of a CI job. This run will usually be against a temporary schema that has been created explicitly for the PR. If the job finishes successfully, the PR can be merged into the main branch, but if it fails the merge will not happen.

CI jobs can also be scheduled with the dbt Cloud scheduler, Airflow, cron and a number of additional tools.

## Deployment using dbt Cloud

Now, create a new _Production_ environment of type _Deployment_ using the latest stable dbt version (`v1.0` at the time of writing these notes). By default, the environment will use the `main` branch of the repo but you may change it for more complex workflows. If you used the JSON credentials when setting up dbt Cloud then most of the deployment credentials should already be set up except for the dataset. For this example, we will use the `production` dataset (make sure that the `production` dataset/schema exists in your BigQuery project).

The dbt Cloud scheduler is available in the _Jobs_ menu in the sidebar. We will create a new job with name `dbt build` using the _Production_ environment, we will check the _Generate docs?_ checkbox. Add the following commands:

1. `dbt seed`
1. `dbt run`
1. `dbt test`

In the _Schedule_ tab at the bottom we will check the _Run on schedule?_ checkbox with a timing of _Every day_ and _every 6 hours_. Save the job. You will be shown the job's run history screen which contains a _Run now_ buttom that allows us to trigger the job manually; do so to check that the job runs successfully.

You can access the run and check the current state of it as well as the logs. After the run is finished, you will see a _View Documentation_ button at the top; clicking on it will open a new browser window/tab with the generated docs.

Under _Account settings_ > _Projects_, you may edit the project in order to modify the _Documentation_ field under _Artifacts_; you should see a drop down menu which should contain the job we created which generates the docs. After saving the changes and reloading the dbt Cloud website, you should now have a _Documentation_ section in the sidebar.

## Data visualization using Google Data Studio

[Google Data Studio](https://datastudio.google.com/) (GDS) is an online tool for converting data into ***reports*** and ***dashboards***.

First, create a ***Data Source*** with BigQuery and connect to `production.fact_trips` schema.

Widget 1: _Time Series Chart_ to show the amount of trips per day

![time series chart](images/04.png)

Vast majority of trips are concentrated in a small interval; this is due to dirty data which has bogus values for `pickup_datetime`. We can filter out these bogus values by adding a _Date Range Control_, which we can drag and drop anywhere in the report, and then set the start date to January 1st 2019 and the end date to December 31st 2020.

![date range control](images/05.png)

Widget 2:  _Scorecard With Compact Numbers_ with the total record count in `fact_trips`
Widget 3: _Pie chart_ displaying the `service_type` dimension using the record count metric
Widget 4: _Table With Heatmap_ using `pickup_zone` as its dimension
Widget 5: _Stacked Column Bar_ showing trips per month. 
Widget 6: _Drop-Down List Control_ and drag the `service_type` dimension to _Control field_. The drop-down control will now allow us to choose yellow, green or both taxi types. 

![final report](images/06.png)
