# Workshop 3: Run Full-Text-Search (FTS) functions on Spanner

## Prerequisite
* Login to you Google Cloud Console
* Make sure that the Spanner APIs is enabled

## Lab 3.1: Provision a Spanner Instance
1. Go to "Spanner" main page
2. Click "CREATE A PROVISIONED INSTANCE"
3. Choose "Enterprise" Edition, then click "Continue"
4. Put your instance name "workshop-spanner-01" (The console will automatic put your instance id with the same name)
5. Choose "Regional" and "Singapore" configure.
6. Adjust number of nodes to be "3" nodes.
7. Click "CREATE"

## Lab 3.2: Create a Spanner Database
1. Click "CREATE DATABASE"
2. Put "workshopdb" as a name
3. Select "Google Standard SQL" as the dailect for this database.
4. Click "CREATE"
5. View over database pages to see what's changes.

## Lab 3.3: Create the "toy" table
1. Click "Spanner Studio" on the left panel
2. Choose "Editor 1" tab to open the database studio.
3. Create a table by using SQL as below.
```
CREATE TABLE toy(
	product_id STRING(MAX),
	crawl_timestamp TIMESTAMP,
	product_url STRING(MAX),
	product_name STRING(MAX),
	description STRING(MAX),
	list_price FLOAT64,
	sale_price FLOAT64,
	brand STRING(500),
	item_number FLOAT64,
	gtin INT64,
	package_size STRING(MAX),
	category STRING(MAX),
	postal_code STRING(MAX),
	available BOOL
) PRIMARY KEY (product_id)
;
```
4. Click "Run"
5. View information by using the explorer panel.

## Lab 3.4: Upload "toy.csv" data to BigQuery
1. Go to BigQuery by searching "bigquery" in the search box.
2. At "Explorer", click "+ ADD"
3. Download "toy.csv" to your laptop
4. Choose "Local file"
5. At "Select file *", Browse the "toy.csv"
6. Change "File format" to be "CSV"
7. At "Dataset", choose to create a new one, provide "Dataset ID" as "workshopds", ensure that the region is "asia-southeast1 (Singapore)", then click "CREATE DATASET"
8. At "Table", put "toy"
9. At "Schema", check the "Auto detect"
10. Click "CREATE TABLE"
11. Open a editor by clicking "+" at the middle console
12. Put the query
```
SELECT * 
-- `<PROJECT_ID.DATASET_ID.TABLE_ID>`
FROM `qwiklabs-gcp-04-9b3626472aa1.workshopds.toy` 
LIMIT 10
;
```

## Lab 3.5: Load data into Spanner by using BigQuery Reverse-ETL
1. Run Reverse ETL from BigQuery to Spanner at "BigQuery Editor"
```
-- uri="https://spanner.googleapis.com/projects/<PROJECT_ID>/instances/<SPANNER_INSTANCE_ID/databases/<SPANNER_DATABASE_ID>"
EXPORT DATA OPTIONS (
  uri="https://spanner.googleapis.com/projects/qwiklabs-gcp-04-9b3626472aa1/instances/workshop-spanner-01/databases/workshopdb",
  format='CLOUD_SPANNER',
  spanner_options="""{ "table": "toy" }"""
)
AS SELECT * 
-- `<PROJECT_ID.DATASET_ID.TABLE_ID>`
FROM `qwiklabs-gcp-04-9b3626472aa1.workshopds.toy`
;
```
2. Go back to "Spanner Studio"
3. Check the data
```
SELECT *
FROM toy
LIMIT 20
;
```

## Lab 3.6: Prepare tokenization columns and search indexes
1. Add tokenization columns into Spanner
```
ALTER TABLE toy
ADD COLUMN description_tokens TOKENLIST
AS (TOKENIZE_FULLTEXT(description)) HIDDEN
;
ALTER TABLE toy
ADD COLUMN product_name_tokens TOKENLIST
AS (TOKENIZE_FULLTEXT(product_name)) HIDDEN
;
ALTER TABLE toy
ADD COLUMN general_tokens TOKENLIST
AS (TOKENIZE_FULLTEXT(CONCAT(product_name,description))) HIDDEN
;
```
2. Create search indexes on top of the tokenization columns
```
CREATE SEARCH INDEX descriptionSearchIndex ON toy(description_tokens)
;
CREATE SEARCH INDEX productNameSearchIndex ON toy(product_name_tokens)
;
CREATE SEARCH INDEX generalSearchIndex ON toy(general_tokens)
;
```

## Lab 3.7: Run full-text-search (FTS) queries on the data
1. Run a full-text-search query on the "description" column
```
SELECT product_id, product_name, description
FROM toy
WHERE SEARCH(description_tokens, 'bubble guns')
;
```
2. Try to change the search keyword to be
* swimming pool
* BioGuard
* game card
* BioGuard OR swimming pool -- use OR logic in the rquery
3. Try another FTS query on the "product_name" column
```
SELECT product_id, product_name, description
FROM toy
WHERE SEARCH(product_name_tokens, 'Gorilla')
;
```
4. Try to change the search keyword to be
* Gorilla Blue
* 3-wheel Tricycle
* Water Wheel
5. Try another FTS query on both "description" and "product_name" columns
```
SELECT product_id, product_name, description
FROM toy
WHERE SEARCH(general_tokens, 'This is test')
;
```
6. Try to change the search keyword to be
* Swimming pool
* Unicorn -- show both searches
* Frog

## Lab 3.8: Explore more on Spanner Full-Text-Search (FTS)
1. Checking Spell correction with Fuzzy matching. Try to run the statement below. It should return "No results".
```
SELECT product_id, product_name, description
FROM toy
WHERE SEARCH(description_tokens, 'swiming pool')
;
```
2. Change the statement to be
```
SELECT product_id, product_name, description
FROM toy
WHERE SEARCH(description_tokens, 'swiming pool', enhance_query=>true)
;
```
3. View the score of the search. Try to run the statement below.
```
SELECT SCORE(description_tokens, 'swimming pool') AS search_score,
product_id, product_name, description
FROM toy
WHERE SEARCH(description_tokens, 'swimming pool')
ORDER BY search_score DESC
;
```
4. View snippet of the search result. Try to run the statement below.
```
SELECT SNIPPET(description, 'swimming') AS snipper,
product_id, product_name, description
FROM toy
WHERE SEARCH(description_tokens, 'swimming')
;
```

## Lab 3.9: Insert a new record and see how no-ETL work
1. Run a FTS query
```
SELECT product_id, product_name, description
FROM toy
WHERE SEARCH(general_tokens, 'google')
;
```
2. Make an insert statement
```
INSERT INTO toy (product_id, product_name, description)
VALUES ('60564492639169878ba6723fce384319', 'Google T-shirt', 'This is Gooooooooogle T-Shirt')
;
```
3. Re-run the FTS query again
```
SELECT product_id, product_name, description
FROM toy
WHERE SEARCH(general_tokens, 'google')
;
```