# Incremental Data Processing
## Streaming Data with Apache Spark
A data stream represents an unbounded flow of data, often originating from various sources such as sensors, log files, or social media platforms. As new data is generated, it is appended to the stream, making it a dynamic and constantly changing dataset.

### Spark Structured Streaming
Spark Structured Streaming is a scalable stream processing engine that enables querying of infinite data sources, automatically detecting new data as it arrives and persisting results incrementally into target data sink. A sink is often a durable storage system such as files or tables that serves as the destination for the processed data. In Structured Streaming, the key idea is to handle live data streams as unbounded, continuously growing tables, where each incoming data item is appended as a new row. This design allows you to apply familiar SQL and DataFrame operations on streaming data in the same way you would with batch data.

**The append-only requirement of streaming sources**

One fundamental prerequisite for a data source to be considered valid for streaming is that it must adhere to the append-only requirement in Structured Streaming. This condition implies that data can only be added to the source, and existing data cannot be modified. If a data source allows data to be updated, deleted, or overwritten, it is then considered no longer streamable.

**Delta Lake as streaming source**

Spark Structured Streaming seamlessly integrates with various data sources, including directories of files, messaging systems like Kafka, and Delta Lake tables as well. Delta Lake is well-integrated with Spark Structured Streaming using the **DataStreamReader** and **DataStreamWriter** APIs in PySpark.

1. **DataStreamReader**
In Python, the `spark.readStream` method allows you to query a Delta Lake table as a streaming source. This functionality enables processing both existing data in the table and any new data that arrives subsequently. The result is a “streaming” DataFrame, which allows for applying transformations just like one would on a static DataFrame:
```sql
streamDF = spark.readStream.table("source_table")
```

2. **DataStreamWriter**
Once the necessary transformations have been applied, the results of the streaming DataFrame can be persisted using its writeStream method:
```sql
streamDF.writeStream.table("target_table")
```
This method enables configuring various output options to store the processed data into durable storage. For example, we have two Delta Lake tables, `Table_1` and `Table_2`. The goal is to continuously stream data from `Table_1` to `Table_2`, appending new records into `Table_2` every two minutes.To achieve this, we use the following Python code. This code sets up a Structured Streaming job in Spark that continuously monitors `Table_1` for new data, processes it at regular intervals of two minutes, and appends the new records to `Table_2`:
```sql
streamDF = spark.readStream
                .table("Table_1")

streamDF.writeStream
        .trigger(processingTime="2 minutes")
        .outputMode("append")
        .option("checkpointLocation", "/path")
        .table("Table_2")
```
>In this code snippet, we start by defining a streaming DataFrame streamDF against the Delta table Table_1 using the spark.readStream method. Whenever a new version of the table is written, a new micro-batch containing the new data will come in through this readStream.
>
>Next, we use the writeStream method to define the streaming write operation on the streamDF. Here, we specify the processing trigger interval using the trigger function, indicating that Spark should check for new data every two minutes. This means that the streaming job will be triggered at regular intervals of two minutes to process any new incoming data in the source.
>
>We then set the output mode to “append” using the outputMode function. This mode ensures that only newly received records since the last trigger will be appended to the output sink, which in this case is the Delta table Table_2.
>
>Additionally, we specify the checkpoint location using the checkpointLocation option. Spark uses checkpoints to store metadata about the streaming job, including the current state and progress. By providing a checkpoint location, Spark can recover the streaming job from failures and maintain its state across restarts.
