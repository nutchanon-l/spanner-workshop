# Workshop 2: Run applications on Spanner

## Prerequisite
* You have run "Workshop1.md"
* You have created a Spanner Instance
* You have created a Spanner Database
* You have created "Singers" table in the database

## Lab 2.1: Run a python application
1. Copy the code below to Cloud Shell as "insert-spanner.py"
```
cat << EOF > insert-spanner.py
#!/usr/bin/python3

from google.cloud import spanner

INSTANCE_ID = "<SPANNER_INSTANCE_ID>" # e.g. workshop-spanner-01
DATABASE_ID = "<SPANNER_DATABASE_ID>" # e.g. workshopdb

spanner_client = spanner.Client()
instance = spanner_client.instance(INSTANCE_ID)
database = instance.database(DATABASE_ID)

def insert_singers(transaction):
    row_ct = transaction.execute_update(
        "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES "
        "(10, 'Melissa', 'Garcia'), "
        "(11, 'Russell', 'Morales'), "
        "(12, 'Jacqueline', 'Long'), "
        "(13, 'Dylan', 'Shaw')"
    )
    print("{} record(s) inserted.".format(row_ct))
	
database.run_in_transaction(insert_singers)
EOF
```
2. Change "INSTANCE_ID" and "DATABASE_ID" for your instance on "insert-spanner.py"
* INSTANCE_ID is "workshop-spanner-01"
* DATABASE_ID is "workshopdb"
3. Run a python program. It should return "4 record(s) inserted."
```
$ python3 insert-spanner.py
```
4. Copy the code below to Cloud Shell as "select-spanner.py"
```
cat << EOF > select-spanner.py
#!/usr/bin/python3

from google.cloud import spanner

INSTANCE_ID = "<SPANNER_INSTANCE_ID>" # e.g. workshop-spanner-01
DATABASE_ID = "<SPANNER_DATABASE_ID>" # e.g. workshopdb

spanner_client = spanner.Client()
instance = spanner_client.instance(INSTANCE_ID)
database = instance.database(DATABASE_ID)

def query_data():

    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            "SELECT SingerId, FirstName, LastName FROM singers"
        )

        for row in results:
            print("{} {} {}".format(*row))
	
query_data()
EOF
```
5. Change "INSTANCE_ID" and "DATABASE_ID" for your instance on "select-spanner.py"
* INSTANCE_ID is "workshop-spanner-01"
* DATABASE_ID is "workshopdb"
6. Run a python program. It should return every rows in the table.
```
$ python3 select-spanner.py
```