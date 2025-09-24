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

### Streaming Query Configurations

#### Trigger Intervals

When setting up a streaming write operation, the trigger method defines how often the system should process incoming data. This timing mechanism is referred to as the trigger interval. There are two primary trigger modes: continuous and triggered.

* **Continuous mode: Near-real-time processing**
  
In this mode, the streaming query will continuously run to process data in micro-batches at regular intervals. By default, if no specific trigger interval is provided, the data will be processed every half a second. This is equivalent to using the option `processingTime="500ms"`. Alternatively, you have the flexibility to specify another fixed interval according to your requirements. This mode ensures a continuous flow of data, enabling near-real-time data processing.

* **Triggered mode: Incremental batch processing**
  
In contrast to continuous mode, the triggered mode offers a batch-oriented approach known as incremental batch processing.  In this mode, the streaming query processes all available data since the last trigger and then stops automatically. This mode is suited for scenarios where data arrival is not constant, eliminating the need for continuously running resources. Under the triggered mode, two options are available: `Once and availableNow`:

  > 1. **Trigger.Once**
  >
  >   With this option, the stream processes the currently available data, all at once, in a single micro-batch. However, this can introduce challenges related to       scalability when dealing with large volumes of data, as it may lead to out-of-memory (OOM) errors.

  > 2. **Trigger.availableNow**
  >  
  >    Similarly, the availableNow option also facilitates batch processing of all currently available data. However, it addresses scalability concerns by        allowing data to be processed in multiple micro-batches until completion. This option offers flexibility in handling large data batches while ensuring efficient resource utilization.

>NOTE: Since Databricks Runtime 11.3 LTS, the Trigger.Once setting has been deprecated. However, it’s possible that you may encounter references to it in the current exam version or in older documentation. Databricks now recommends using Trigger.AvailableNow for all incremental batch processing workloads.

#### Output Modes
When writing streaming data, you can specify the output mode to define how the data is written to the target. There are primarily two output modes available: `append mode and complete mode`.


* **Append (default)**
  
`.outputMode("append")`
Only newly received rows are appended to the target table with each batch.  This mode is suitable for scenarios where the target sink needs to maintain a continuously growing dataset based on the incoming streaming data.

* **Complete**
  
`.outputMode("complete")`
The target table is overwritten with each batch. It replaces the entire contents of the output sink with the latest computed results with each batch. This mode is commonly used for updating summary tables with the latest aggregates.

#### Checkpointing
Checkpointing is a mechanism for saving the progress information of the streaming query. The checkpoints are stored in a reliable storage system, such as the DBFS or cloud storage like Amazon S3 or Azure Storage. This approach ensures that if the streaming job crashes or needs to be restarted, it can resume processing from the last checkpointed state rather than starting from scratch.

One important aspect to note about checkpoints in Apache Spark is that they cannot be shared between multiple streaming jobs. Each streaming write operation requires its own separate checkpoint location. This separation ensures that each streaming application maintains its own processing guarantees and doesn’t interfere with or rely on the checkpoints of other jobs.
