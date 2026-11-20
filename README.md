# Amazon Athena – CTAS, Parquet Conversion and Data Partitioning

## 1. Create Database

```sql
CREATE DATABASE IF NOT EXISTS sales_db;
```

## 2. Create Raw CSV Table

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS sales_db.raw_sales (
    order_id STRING,
    order_date STRING,
    customer_id STRING,
    product STRING,
    category STRING,
    quantity INT,
    unit_price INT,
    total_amount INT,
    city STRING,
    payment_mode STRING,
    year INT,
    month INT
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
    'separatorChar' = ',',
    'quoteChar' = '"',
    'escapeChar' = '\\'
)
LOCATION 's3://mse2-bucket-073276525111-us-east-1-an/raw/'
TBLPROPERTIES ('skip.header.line.count'='1');
```

## 3. Verify Raw Table

```sql
SELECT *
FROM sales_db.raw_sales
LIMIT 10;
```

## 4. Query Sales for November 2025 (CSV)

```sql
SELECT *
FROM sales_db.raw_sales
WHERE year = 2025 AND month = 11;
```

## 5. Convert CSV to Parquet using CTAS and Partitioning

```sql
CREATE TABLE sales_db.parquet_sales
WITH (
    format = 'PARQUET',
    external_location = 's3://mse2-bucket-073276525111-us-east-1-an/converted/',
    partitioned_by = ARRAY['year', 'month']
)
AS
SELECT
    order_id,
    order_date,
    customer_id,
    product,
    category,
    quantity,
    unit_price,
    total_amount,
    city,
    payment_mode,
    year,
    month
FROM sales_db.raw_sales;
```

## 6. Verify Parquet Table

```sql
SELECT *
FROM sales_db.parquet_sales
LIMIT 10;
```

## 7. Verify Partitions

```sql
SELECT DISTINCT year, month
FROM sales_db.parquet_sales
ORDER BY year, month;
```

## 8. Run the Same Monthly Query on Parquet

```sql
SELECT *
FROM sales_db.parquet_sales
WHERE year = 2025 AND month = 11;
```

## Performance Comparison

| Metric | CSV | Parquet |
|---|---|---|
| Format | CSV | Parquet |
| Partitioning | None | Year and Month |
| Data Scanned | 1.54 MB | 11.03 KB |
| Execution Time | 628 ms | 1.214 sec |

## Conclusion

The raw sales data was stored in CSV format in Amazon S3 and queried using Amazon Athena. Using CTAS (Create Table As Select), the data was converted into Parquet format and partitioned by year and month.

The same monthly query was executed on both tables. The data scanned decreased from 1.54 MB to 11.03 KB after conversion to Parquet and partitioning, demonstrating more efficient data scanning and the potential to reduce Athena query costs.

Although the Parquet query had a higher observed execution time in this small experiment, the reduction in scanned data demonstrates the storage and query-efficiency benefits of Parquet and partitioning.


073276525111