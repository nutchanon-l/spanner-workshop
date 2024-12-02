# Workshop 1: Deploy your first Spanner on Google Cloud

## Prerequisite
* Login to you Google Cloud Console
* Make sure that the Spanner APIs is enabled
* Prefer to use "Challenge Lab: Administering a Spanner Database" for 4 hours access.

## Lab 1.1: Provision a Spanner Instance
1. Go to "Spanner" main page
2. Click "CREATE A PROVISIONED INSTANCE"
3. Choose "Enterprise" Edition, then click "Continue"
4. Put your instance name "workshop-spanner-01" (The console will automatic put your instance id with the same name)
5. Choose "Regional" and "Singapore" configure.
6. Leave "1" node as a default value.
7. Click "CREATE"

## Lab 1.2: Walkthrough the Spanne console
1. See the middle layer for the configuration
    - Instance ID, Edition, Region, Auto scaling
    - Instance summary
    - Databases
2. View the left panel
    - Overview, Export/Import, Backup/Restore, Partitions
    - System Insights, Query Insights, Lock Insights, Transaction Insights, Hotspot Insights

## Lab 1.3: Create a Spaner Database
1. Click "CREATE DATABASE"
2. Put "workshopdb" as a name
3. Select "Google Standard SQL" as the dailect for this database.
4. Click "CREATE"
5. View over database pages to see what's changes.

## Lab 1.4: Create a table in the database 
1. Click "Spanner Studio" on the left panel
2. Choose "Editor 1" tab to open the database studio.
3. Create a table by using SQL as below.
```
CREATE TABLE Singers (
  SingerId   INT64 NOT NULL,
  FirstName  STRING(1024),
  LastName   STRING(1024),
  SingerInfo BYTES(MAX),
  BirthDate  DATE
) PRIMARY KEY(SingerId)
;
```
4. Click "Run"
5. View information by using the explorer panel.

## Lab 1.5: Perform read and write operations on Spanner.
1. Run a query
```
SELECT * from Singers;
```
2. Insert a row
```
INSERT INTO
  Singers (SingerId,
    BirthDate,
    FirstName,
    LastName,
    SingerInfo)
VALUES
  (1, -- type: INT64
    NULL, -- type: DATE
    'Marc', -- type: STRING(1024)
    'Richards', -- type: STRING(1024)
    NULL -- type: BYTES(MAX)
    )
;
```
3. Insert another row
```
INSERT INTO Singers  (SingerId, BirthDate, FirstName, LastName, SingerInfo) VALUES (2, "2024-12-01", "Google", "Cloud", NULL);
```
4. Run a query again
```
SELECT * from Singers;
```
5. Run another query
```
SELECT * from Singers WHERE SingerId = 2;
```
6. Update a row
```
UPDATE Singers SET FirstName = "Goooogle" WHERE SingerId = 2;
```
7. Run a query again
```
SELECT * from Singers;
```
8. Delete a row
```
DELETE FROM Singers WHERE SingerId = 1;
```
9. Run a query again
```
SELECT * from Singers;
```

## Lab 1.6: Modify Spanner Configuration
1. Back to Spanner Instanc main page by clicking "INSTANCE" at the top
2. Click "EDIT INSTANCE"
3. Scoll down to "Configure compute capacity" Section
4. Choose a scaling mode to be "Autoscaling"
5. Put minimum as "3" and Maximum as "10"
6. Leave CPU threshold as "65" % and Storage threshold as "95" % as default values.
7. Click "SAVE"

# Optional Labs
## Lab 1.7: Create a Spanner Instance by using gcloud
1. Open Cloud Shell
2. Provide project id and login, click "Authorize"
```
$ gcloud auth list
$ gcloud config set project <GOOGLE_CLOUD_PROJECT_ID>
```
3. Check existing instances
```
$ gcloud spanner instances list
```
4. Create a Spanner instance
```
$ gcloud spanner instances create workshop-spanner-02 \
--description workshop-spanner-02 \
--config regional-asia-southeast1 \
--edition ENTERPRISE \
--nodes 1
5. Check the Spanner instance in the console
