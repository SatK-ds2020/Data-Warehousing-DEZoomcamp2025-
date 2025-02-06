# Module 3 Homework

For this homework we will be using the Yellow Taxi Trip Records for January 2024 - June 2024 NOT the entire year of data Parquet Files from the New York City Taxi Data found here:
https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
If you are using orchestration such as Kestra, Mage, Airflow or Prefect etc. do not load the data into Big Query using the orchestrator.
Stop with loading the files into a bucket.

<b>GCP SETUP:</b></br>
- service account created with roles: storage admin, storage object create,storage object admin, bigquery admin  
- Key Credentials were downloaded as json file
- Google storage Bucket("dezoomcamp_skhw3_2025") created with permissions granted for service account principle for storage object create, storage admin, storage object admin. 

```bash
export GOOGLE_APPLICATION_CREDENTIALS="D:/DEZommcamp2025/gcp-serviceaccount/gcp-warehouse.json"
```
```bash
$ python load_yellow_taxi_data.py
Downloading https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-01.parquet...
Downloading https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-02.parquet...
Downloading https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-03.parquet...
Downloading https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-04.parquet...
Downloaded: .\yellow_tripdata_2024-02.parquet
Downloading https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-05.parquet...
Downloaded: .\yellow_tripdata_2024-04.parquet
Downloading https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-06.parquet...
Downloaded: .\yellow_tripdata_2024-06.parquet
Downloaded: .\yellow_tripdata_2024-05.parquet
Downloaded: .\yellow_tripdata_2024-03.parquet
Downloaded: .\yellow_tripdata_2024-01.parquet
Uploading .\yellow_tripdata_2024-01.parquet to dezoomcamp_skhw3_2025 (Attempt 1)...   
Uploading .\yellow_tripdata_2024-02.parquet to dezoomcamp_skhw3_2025 (Attempt 1)...   
Uploading .\yellow_tripdata_2024-03.parquet to dezoomcamp_skhw3_2025 (Attempt 1)...   
Uploading .\yellow_tripdata_2024-04.parquet to dezoomcamp_skhw3_2025 (Attempt 1)...   
Uploaded: gs://dezoomcamp_skhw3_2025/yellow_tripdata_2024-01.parquet
Uploaded: gs://dezoomcamp_skhw3_2025/yellow_tripdata_2024-02.parquet
Uploaded: gs://dezoomcamp_skhw3_2025/yellow_tripdata_2024-04.parquet
Verification successful for yellow_tripdata_2024-02.parquet
Uploading .\yellow_tripdata_2024-05.parquet to dezoomcamp_skhw3_2025 (Attempt 1)...   
Verification successful for yellow_tripdata_2024-01.parquet
Uploading .\yellow_tripdata_2024-06.parquet to dezoomcamp_skhw3_2025 (Attempt 1)...   
Verification successful for yellow_tripdata_2024-04.parquet
Uploaded: gs://dezoomcamp_skhw3_2025/yellow_tripdata_2024-03.parquet
Verification successful for yellow_tripdata_2024-03.parquet
Uploaded: gs://dezoomcamp_skhw3_2025/yellow_tripdata_2024-05.parquet
Verification successful for yellow_tripdata_2024-05.parquet
Uploaded: gs://dezoomcamp_skhw3_2025/yellow_tripdata_2024-06.parquet
Verification successful for yellow_tripdata_2024-06.parquet
All files processed and verified.
```
Load Script: You can manually download the parquet files and upload them to your GCS Bucket or you can use the linked script [here](./load_yellow_taxi_data.py):<br>

<b>BIG QUERY SETUP:</b></br>
Create an external table using the Yellow Taxi Trip Records. </br>
Create a (regular/materialized) table in BQ using the Yellow Taxi Trip Records (do not partition or cluster this table). </br>
</p>

## Question 1:
Question 1: What is count of records for the 2024 Yellow Taxi Data?
- 65,623
- 840,402
- 20,332,093
- 85,431,289
### Answer: 20,332,093
```sql
CREATE OR REPLACE EXTERNAL TABLE `yellow_taxi.yellowtaxi_tripdata_2024`
OPTIONS (
  format = 'parquet',
  uris = ['gs://dezoomcamp_skhw3_2025/yellow_tripdata_2024-*.parquet']
);


SELECT COUNT(*) FROM `de-zoomcamp2025-448100.yellow_taxi.yellowtaxi_tripdata_2024`;
```

## Question 2:
Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.</br> 
What is the **estimated amount** of data that will be read when this query is executed on the External Table and the Table?

- 18.82 MB for the External Table and 47.60 MB for the Materialized Table
- 0 MB for the External Table and 155.12 MB for the Materialized Table
- 2.14 GB for the External Table and 0MB for the Materialized Table
- 0 MB for the External Table and 0MB for the Materialized Table
- 
### Answer:  0 MB for the External Table and 155.12 MB for the Materialized Table (262 Distinct count)
```sql
CREATE OR REPLACE TABLE `de-zoomcamp2025-448100.yellow_taxi.yellow_nonpartitioned_tripdata2024`
AS SELECT * FROM `de-zoomcamp2025-448100.yellow_taxi.yellowtaxi_tripdata_2024`;

SELECT * FROM `de-zoomcamp2025-448100.yellow_taxi.yellow_nonpartitioned_tripdata2024` LIMIT 10;

# MATERIALIZED TABLE
SELECT COUNT(DISTINCT(PULocationID)) FROM `de-zoomcamp2025-448100.yellow_taxi.yellow_nonpartitioned_tripdata2024`;

# EXTERNAL TABLE
SELECT COUNT(DISTINCT(PULocationID)) FROM `de-zoomcamp2025-448100.yellow_taxi.yellowtaxi_tripdata_2024`;
```


## Question 3:
Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table. Why are the estimated number of Bytes different?
- BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires 
reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.
- BigQuery duplicates data across multiple storage partitions, so selecting two columns instead of one requires scanning the table twice, 
doubling the estimated bytes processed.
- BigQuery automatically caches the first queried column, so adding a second column increases processing time but does not affect the estimated bytes scanned.
- When selecting multiple columns, BigQuery performs an implicit join operation between them, increasing the estimated bytes processed

### Answer:  BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.
```sql
SELECT PULocationID FROM `de-zoomcamp2025-448100.yellow_taxi.yellow_nonpartitioned_tripdata2024`;

SELECT PULocationID,DOLocationID FROM `de-zoomcamp2025-448100.yellow_taxi.yellow_nonpartitioned_tripdata2024`;
```

## Question 4:
How many records have a fare_amount of 0?
- 128,210
- 546,578
- 20,188,016
- 8,333
## Answer:  8,333 
```sql
SELECT COUNT(*) FROM `de-zoomcamp2025-448100.yellow_taxi.yellow_nonpartitioned_tripdata2024` 
WHERE fare_amount = 0;
```
## Question 5:
What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)
- Partition by tpep_dropoff_datetime and Cluster on VendorID
- Cluster on by tpep_dropoff_datetime and Cluster on VendorID
- Cluster on tpep_dropoff_datetime Partition by VendorID
- Partition by tpep_dropoff_datetime and Partition by VendorID
## Answer:  Partition by tpep_dropoff_datetime and Cluster on VendorID
```sql
CREATE OR REPLACE TABLE `yellow_taxi.yellowtaxi2024_partitioned_clustered`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS (
  SELECT * FROM `yellow_taxi.yellowtaxi_tripdata_2024`
);
```

## Question 6:
Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime
2024-03-01 and 2024-03-15 (inclusive)</br>

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values? </br>

Choose the answer which most closely matches.</br> 

- 12.47 MB for non-partitioned table and 326.42 MB for the partitioned table
- 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table
- 5.87 MB for non-partitioned table and 0 MB for the partitioned table
- 310.31 MB for non-partitioned table and 285.64 MB for the partitioned table
## Answer: 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table (VendorIDs: 6,1,2)
```sql
SELECT DISTINCT(VendorID) FROM  `yellow_taxi.yellow_nonpartitioned_tripdata2024`
WHERE DATE(tpep_dropoff_datetime) BETWEEN '2024-03-01' AND '2024-03-15';

SELECT DISTINCT(VendorID) FROM  `yellow_taxi.yellowtaxi2024_partitioned_clustered`
WHERE DATE(tpep_dropoff_datetime) BETWEEN '2024-03-01' AND '2024-03-15';
```


## Question 7: 
Where is the data stored in the External Table you created?

- Big Query
- Container Registry
- GCP Bucket
- Big Table
## Answer: The data for an External Table in Google BigQuery is stored in a GCP Bucket.

External tables allow BigQuery to query data stored outside of BigQuery’s own storage, such as in Google Cloud Storage (GCS), which is commonly referred to as GCP Bucket. This allows us to keep our data in GCS while still being able to analyze it using BigQuery's powerful query engine.

## Question 8:
It is best practice in Big Query to always cluster your data:
- True
- False
## Answer: False.

While clustering the data can improve query performance and reduce costs in many scenarios, it is not always necessary or beneficial. Whether to cluster the data depends on the nature of the queries and the data. Clustering is most useful for tables with large amounts of data and frequently queried columns with high cardinality.
Here are some scenarios where clustering can be beneficial:

Frequent Filtering: If we frequently filter on certain columns, clustering those columns can speed up query performance. For example, if we often filter by date, clustering on the date column can help BigQuery scan only the relevant partitions.

High Cardinality Columns: Clustering works best on columns with high cardinality (many unique values). For example, columns like user IDs or timestamps.

Large Datasets: Clustering is most effective for large datasets where optimizing the query performance can lead to substantial time and cost savings.

Combining with Partitioning: When combined with partitioning, clustering can further enhance performance. For instance, we can partition a table by date and then cluster it by user ID.

Query Patterns: Analyze the query patterns. If our queries benefit from data being physically sorted by specific columns, then clustering is a good choice. For example, analytical queries that perform aggregations or joins on clustered columns.

Storage Savings: Although clustering primarily aims to improve query performance, in some cases, it can also reduce storage costs by optimizing how data is stored.

Therefore, before deciding to cluster, it's important to analyze the query patterns and data characteristics to ensure clustering will provide the desired benefits.

## (Bonus: Not worth points) Question 9:
No Points: Write a `SELECT count(*)` query FROM the materialized table you created. How many bytes does it estimate will be read? Why?
## Answer: byte processed is 0 B however byte shuffled is 459 B
"bytes processed" refers to the total amount of data read by the query, while
"bytes shuffled" refers to the amount of data transferred between different stages of the query execution
In our case When we see 0 B for bytes processed and 459 B for bytes shuffled, it means that no data was read from storage (perhaps because the query was optimized to use cached data or metadata), but there was still some data movement (shuffling) happening between the query stages(reading ,writing).
In this scenerio query is leveraging cached results or intermediate results from previous stages, leading to minimal data read but some data movement.