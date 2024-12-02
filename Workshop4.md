# Workshop 4: Spanner Migration Tool

## Prerequisite
* Login to you Google Cloud Console

## Lab 4.1: Setup Spanner Migration Tool (SMT)
1. Install the Spanner Migration Tool (SMT) by using Cloud Shell
```
$ sudo apt-get install google-cloud-sdk-spanner-migration-tool
```
2. Open the SMT tool, then click "Authroize"
```
$ gcloud alpha spanner migrate web
```
3. In the Cloud Shell, click "Web Preview" icon, then click "Preview on port 8080"

## Lab 4.2: Setup Spanner target
1. At the top right, choose to configure/edit Spanner
2. Provide PROJECT_ID and SPANNER_INSTANCE_ID
3. Click "SAVE"

## Lab 4.3: Setup soruce database
1. Click "Connect to database"
2. Provide "Database Engine" as "PostgreSQL"
3. Provide Connection Detail
4. Choose Spanner Dialect