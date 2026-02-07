# Module 3 Homework: Data Warehousing & BigQuery

##
BigQuery Setup

Create an external table using the Yellow Taxi Trip Records.
```SQL
CREATE EXTERNAL TABLE `ancient-edition-486309-k5.nytaxi.external_nycity_taxi_yellow`
OPTIONS (
    format = 'parquet',
    uris = ['gs://ny_taxi_yellow_2024/yellow_tripdata_2024-*.parquet'],
    autodetect = TRUE
);
```

Create a (regular/materialized) table in BQ using the Yellow Taxi Trip Records (NO partition).

```SQL
create or replace table `ancient-edition-486309-k5.nytaxi.nycity_taxi_yellow_non_partitioned`
as SELECT * FROM `ancient-edition-486309-k5.nytaxi.external_nycity_taxi_yellow` 

```

1. Question 1. Counting records
What is count of records for the 2024 Yellow Taxi Data?
```sql
select count(*) from `nytaxi.external_nycity_taxi_yellow`
```
Answer: 20332093

2. Question 2. Data read estimation
Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.

What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?

```sql
select distinct(PULocationID) from `nytaxi.external_nycity_taxi_yellow`

select distinct(PULocationID) from `nytaxi.nycity_taxi_yellow_non_partitioned`
```
Answer: 0 MB for the External Table and 155.12 MB for the Materialized Table

3. Question 3. Understanding columnar storage
Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table.

```sql
select PULocationID from `nytaxi.nycity_taxi_yellow_non_partitioned`;
select PULocationID,DOLocationID from `nytaxi.nycity_taxi_yellow_non_partitioned`;
```

BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.

4. Question 4. Counting zero fare trips
How many records have a fare_amount of 0?

```sql
select count(*) from `nytaxi.nycity_taxi_yellow_non_partitioned` where fare_amount = 0;
```
Answer: 8333

5. Question 5. Partitioning and clustering

What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)

```sql
create or replace table `nytaxi.nycity_taxi_yellow_partitioned_clustered`
partition by 
  Date(tpep_dropoff_datetime)
cluster by
  VendorID as
select * from `nytaxi.external_nycity_taxi_yellow`
```

Answer: Partition by tpep_dropoff_datetime and Cluster on VendorID

6. Question 6. Partition benefits
Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive)

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values?

```sql
select distinct(VendorID)
from `nytaxi.nycity_taxi_yellow_non_partitioned`
where date(tpep_dropoff_datetime) between '2024-03-01' and '2024-03-15';

select distinct(VendorID)
from `nytaxi.nycity_taxi_yellow_partitioned_clustered`
where date(tpep_dropoff_datetime) between '2024-03-01' and '2024-03-15';
```

Answer: 
310.24 MB for non-partitioned table and 26.84 MB for the partitioned table

7. Question 7. External table storage
Where is the data stored in the External Table you created?
Answer: GCP Bucket

8. Question 8. Clustering best practices
It is best practice in Big Query to always cluster your data:

Answer: False

9. Question 9. Understanding table scans

```sql
select count(*) from `nytaxi.nycity_taxi_yellow_non_partitioned`;
select count(*) from `nytaxi.nycity_taxi_yellow_partitioned_clustered`
```

Answer:  0 B. 

BigQuery can retrieve this information from the stored metadata without needing to scan the actual data in the table's storage. 