# Home Sales Analysis Report

## Introduction

This report presents an analysis of home sales data using PySpark and SparkSQL. The analysis includes various queries to determine average home prices based on different criteria. The data is processed and analyzed using Apache Spark, and the results are cached and compared for performance optimization.

## Dataset Overview

The dataset used for this analysis is stored in a CSV file named `home_sales_revised.csv`. It contains information about home sales, including the sale date, year built, price, number of bedrooms, number of bathrooms, square footage, lot size, number of floors, waterfront presence, and view rating.

## Analysis Questions

The analysis aims to answer the following questions:

1. What is the average price for a four-bedroom house sold for each year?
2. What is the average price of a home for each year the home was built, that has three bedrooms and three bathrooms?
3. What is the average price of a home for each year the home was built, that has three bedrooms, three bathrooms, two floors, and is greater than or equal to 2,000 square feet?
4. What is the average price of a home per "view" rating having an average home price greater than or equal to $350,000?

## Methodology

The analysis is performed using PySpark and SparkSQL. The following steps outline the process:

1. **Environment Setup**: Install Spark and Java, and create a Spark session.
2. **Data Loading**: Read the `home_sales_revised.csv` file into a Spark DataFrame.
3. **Temporary Table Creation**: Create a temporary table named `home_sales` for SQL queries.
4. **Query Execution**: Execute SQL queries to answer the analysis questions.
5. **Caching**: Cache the temporary table to improve query performance.
6. **Partitioning**: Partition the data by `date_built` and save it in parquet format for optimized querying.

## Analysis Results

### 1. Average Price for Four-Bedroom Houses Sold Per Year

```python
average_price_per_year = spark.sql("""
SELECT YEAR(date) as year,
ROUND(AVG(price), 2) as Average_Price
FROM home_sales
WHERE bedrooms = 4
GROUP BY YEAR(date)
ORDER BY YEAR(date)
""").show()
```

| Year | Average Price |
|------|----------------|
| 2019 | 300,263.70     |
| 2020 | 298,353.78     |
| 2021 | 301,819.44     |
| 2022 | 296,363.88     |

### 2. Average Price of Homes Built Each Year with Three Bedrooms and Three Bathrooms

```python
average_price_per_year_built = spark.sql("""
SELECT date_built as year_built,
ROUND(AVG(price), 2) as avg_price
FROM home_sales
WHERE bedrooms = 3 and bathrooms = 3
GROUP BY date_built
ORDER BY date_built
""").show()
```

| Year Built | Average Price |
|------------|----------------|
| 2010       | 292,859.62     |
| 2011       | 291,117.47     |
| 2012       | 293,683.19     |
| 2013       | 295,962.27     |
| 2014       | 290,852.27     |
| 2015       | 288,770.30     |
| 2016       | 290,555.07     |
| 2017       | 292,676.79     |

### 3. Average Price of Homes with Specific Criteria

Criteria: Three bedrooms, three bathrooms, two floors, and at least 2,000 square feet.

```python
avg_price_per_year_built_criteria = spark.sql("""
SELECT date_built as year_built,
ROUND(AVG(price), 2) as avg_price
FROM home_sales
WHERE bedrooms = 3 and bathrooms = 3 and floors = 2 and sqft_living >= 2000
GROUP BY date_built
ORDER BY date_built
""").show()
```

| Year Built | Average Price |
|------------|----------------|
| 2010       | 285,010.22     |
| 2011       | 276,553.81     |
| 2012       | 307,539.97     |
| 2013       | 303,676.79     |
| 2014       | 298,264.72     |
| 2015       | 297,609.97     |
| 2016       | 293,965.10     |
| 2017       | 280,317.58     |

### 4. Average Price per "View" Rating with Average Home Price ≥ $350,000

#### Uncached Data Query

```python
import time
start_time = time.time()

average_price_per_view = spark.sql("""
SELECT view,
ROUND(AVG(price), 2) as avg_price
FROM home_sales
GROUP BY view
HAVING AVG(price) >= 350000
ORDER BY view DESC
""").show()

print("--- %s seconds ---" % (time.time() - start_time))
```

| View | Average Price |
|------|----------------|
| 99   | 1,061,201.42   |
| 98   | 1,053,739.33   |
| 97   | 1,129,040.15   |
| ...  | ...            |

**Uncached Query Runtime**: ~1.49 seconds

#### Cached Data Query

```python
spark.catalog.cacheTable("home_sales")

start_time = time.time()
average_price_per_cached_view = spark.sql("""
SELECT view,
ROUND(AVG(price), 2) as avg_price
FROM home_sales
GROUP BY view
HAVING AVG(price) >= 350000
ORDER BY view DESC
""").show()

print("--- %s seconds ---" % (time.time() - start_time))
```

| View | Average Price |
|------|----------------|
| 99   | 1,061,201.42   |
| 98   | 1,053,739.33   |
| 97   | 1,129,040.15   |
| ...  | ...            |

**Cached Query Runtime**: ~3.99 seconds

### 5. Partitioned Data Query

```python
df.write.partitionBy("date_built").parquet("home_sales_partitioned")
parquet_df = spark.read.parquet("home_sales_partitioned")
parquet_df.createOrReplaceTempView("parquet_home_sales")

start_time = time.time()
spark.sql("""
SELECT view,
ROUND(AVG(price), 2) as avg_price
FROM parquet_home_sales
GROUP BY view
HAVING AVG(price) >= 350000
ORDER BY view DESC
""").show()

print("--- %s seconds ---" % (time.time() - start_time))
```

**Partitioned Data Query Runtime**: ~0.055 seconds

## Conclusion

This analysis demonstrates the power and efficiency of using PySpark and SparkSQL for large-scale data processing. By partitioning the data and using caching, we can significantly improve query performance. The results provide valuable insights into home prices based on various criteria, which can be useful for real estate analysis and decision-making.

## Repository Link

The complete notebook and data files are available in the GitHub repository:

[Home_Sales GitHub Repository](https://github.com/ajsidd/Home_Sales)

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
