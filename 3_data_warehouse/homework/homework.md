# Other tasks
- Ingest yellowtaxis, greentaxis, fhv and zones data into GCS
- Create BigQuery tables from GCS files
- Run SQL analysis 

## SQL Analysis
**What is count for fhv vehicles data for year 2019**  

```sql
-- Create a table with only the 2019 data for simplicity
CREATE OR REPLACE EXTERNAL TABLE `dtc-de-382406.trips_data_all.fhv_tripdata`
OPTIONS (
  format = 'CSV',
  uris = ['gs://dtc_data_lake_dtc-de-382406/fhv/fhv_tripdata_2019-*.csv']
);

-- Count of fhv trips
SELECT count(*) FROM `dtc-de-382406.trips_data_all.fhv_tripdata`;
```

**How many distinct dispatching_base_num we have in fhv for 2019**  
```sql
SELECT COUNT(DISTINCT(dispatching_base_num)) FROM `dtc-de-382406.trips_data_all.fhv_tripdata`;
```

**Best strategy to optimise if query always filter by dropoff_datetime and order by dispatching_base_num**  
```
Partition by dropoff_datetime and cluster by dispatching_base_num
```

**What is the count, estimated and actual data processed for query which counts trip between 2019/01/01 and 2019/03/31 for dispatching_base_num B00987, B02060, B02279**  

```sql
-- This code assumes that fhv_tripdata only contains 2019 data
-- Non-partitioned table
CREATE OR REPLACE TABLE `animated-surfer-338618.trips_data_all.fhv_nonpartitioned_tripdata`
AS SELECT * FROM `animated-surfer-338618.trips_data_all.fhv_tripdata`;

-- Partitioned and clustered table
CREATE OR REPLACE TABLE `animated-surfer-338618.trips_data_all.fhv_partitioned_tripdata`
PARTITION BY DATE(dropoff_datetime)
CLUSTER BY dispatching_base_num AS (
  SELECT * FROM `animated-surfer-338618.trips_data_all.fhv_tripdata`
);

SELECT count(*) FROM  `animated-surfer-338618.trips_data_all.fhv_nonpartitioned_tripdata`
WHERE dropoff_datetime BETWEEN '2019-01-01' AND '2019-03-31'
  AND dispatching_base_num IN ('B00987', 'B02279', 'B02060');
```

**What will be the best partitioning or clustering strategy when filtering on dispatching_base_num and SR_Flag**  

```
Cluster by SR_Flag and dispatching_base_num

The SR_Flag field appears as a String rather than an Integer which means that we could not use it for partitioning. 
```

**What improvements can be seen by partitioning and clustering for data size less than 1 GB**  

```
No improvements
Can be worse as partitioning and clustering also creates extra metadata.  

```

