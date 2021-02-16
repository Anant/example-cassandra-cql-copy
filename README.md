# cql-copy
We cover how we can use CQL Copy for data operations. A step-by-step demo is shown below to walkthrough how you can do data operations such as exporting a Cassandra table as a CSV and then importing data from a CSV into a Cassandra table.

## Prerequisites
- [Docker](https://www.docker.com/)

## Demo

1. We will begin by downloading / cloning this GitHub repository. Once extracted / cloned, we will need to `cd` into it.

```bash
cd cql-copy
```

2. Now that we are in the cql-copy directory, we can start our Apache Cassandra docker container. We will also be mounting our cql-copy directory to the docker container.

```bash
docker run -d -it --name cassandra -v "$(pwd)":/cql-copy cassandra:latest
```

3. We can confirm that the directory was added by opening a new terminal tab / terminal and running the following:

```bash
docker exec -it cassandra bash
```
```bash
ls
```

4. Now we can open another terminal tab / terminal and run the following to start a CQLSH terminal in our docker container:

```bash
docker exec -it cassandra cqlsh
```

5. Once loaded, we can use the cql file that is in our cql-copy directory when we mounted it.
```bash
source '/cql-copy/spacecraft_journey_catalog.cql'
```

In this cql file, we create 2 tables: spacecraft_journey_catalog and duration_by_journey_summary. We also insert 1000 records into spacecraft_journey_catalog. As a sneak-peek of the scenario, we will then export this table as CSV, do some “transformations” on it, and load the “transformed” data CSV into the duration_by_journey_summary table.

6. To confirm that our data is in our Cassandra instance, we can run the following code block. We should get back 1000 rows.
```bash
use demo ;
describe tables ;
select count(*) from spacecraft_journey_catalog ;
```

7. As mentioned above, we can now export this data as CSV using CQL Copy. We will be using COPY TO and only export 4 of the columns. We want to take the summary and journey_id of each journey and calculate the duration in days of each journey versus just having the end time and start time. We can then load that new data into duration_by_journey_summary, which is partitioned by the summary with a clustering column of journey_id. The use case of this kind of table could be for BI to analyze discrepancies between duration lengths of journies that had the same purpose / summary. We will point the CSV to be created at the root level of the docker container.

```bash
copy spacecraft_journey_catalog (summary, journey_id, end, start) to'../spacecraft_journey_catalog.csv' with header = true ;
```

8. We can then confirm that the CSV was created by running the following in the terminal / terminal tab that is running Docker bash:
```bash
ls
```

9. Then, we will need to export the CSV to do some “transformations” on it. We will need to run the following in the first terminal or a new terminal tab to get our Docker container ID:

```bash
docker ps
```

10. Then we can run the following command with your ID substituted for the placeholder id:

```bash
docker cp container_ID:/spacecraft_journey_catalog.csv .
```

This will export the CSV to our local cql-copy directory. You can open your file manager to visualize this.

That covers how to export your Cassandra data using CQL COPY TO. Now we will cover how to import data using CQL COPY FROM.

As mentioned before, we will “transform” the CSV that we exported. However, to speed up the demo, I have already updated and calculated the duration in days for you. The “transformed” data is in the CSV file called duration_by_journey_summary.csv. You can see this in the cql-copy directory. Technically, we have already imported the file when we mounted it, but we will show you how to import it from the local directory into the root level of the docker container.

11. Make sure you are in the cql-copy directory on your local machine. Then run the following command using the Docker container ID that we got before:
```bash
docker cp duration_by_journey_summary.csv container_ID:/
```

12. To confirm that the file was added to the root level of the Docker container, run the following in the terminal / terminal tab running Docker bash:

```bash
ls
```

13. Once you have confirmed it, we can now move onto importing the data from the CSV into our duration_by_journey_summary table. Use the following command to do so:

```bash
copy duration_by_journey_summary (summary, journey_id, duration_in_days) from '../duration_by_journey_summary.csv' WITH HEADER = TRUE ;
```

14. We can now move the terminal / terminal tab running CQLSH and confirm that the data was transferred. We should get back 1000 rows.
```bash
select count(*) from duration_by_journey_summary ;
```

And with that we have done CQL Copy for data operations by exporting and importing CSV data from and to our Cassandra tables.

## Resources
- [Accompanying Blog]()
- [DataStax CQL Copy Docs](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_reference/cqlshCopy.html)
- [Apache Cassandra Docs](https://cassandra.apache.org/doc/latest/tools/cqlsh.html#copy-to)
