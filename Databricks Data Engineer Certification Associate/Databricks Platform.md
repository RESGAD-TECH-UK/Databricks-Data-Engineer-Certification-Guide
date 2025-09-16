# Introduction to Databricks Platform
The Databricks Data Intelligence Platform is an AI-powered data lakehouse platform built on Apache Spark. 

## High-Level Architecture of the Databricks Platform
<img width="400" height="600" alt="image" src="https://github.com/user-attachments/assets/be600579-bf3c-4506-b2fe-7172c282db06" />

#### **1. Databricks workspace**:
At the top of the architecture is the Databricks workspace, which serves as the user interface for interacting with the platform. It provides an interactive
environment where users can perform data engineering, analytics, and AI workloads using a variety of languages, such as Python, SQL, R, and Scala. The workspace
offers a range of services, including notebooks for development, dashboards for visualizing data, and workflow management tools for orchestrating data pipelines.

#### **2. Data governance with Unity Catalog**:
At the core of the Databricks lakehouse architecture is Unity Catalog, which provides a centralized data governance solution across all data and AI assets. Unity
Catalog is designed to secure and manage data access across the Databricks environment, ensuring that sensitive information is accessible only to authorized users.
This layer is crucial for maintaining data security, integrity, and compliance across the lakehouse platform.

#### **3. Databricks Runtime**:
Databricks Runtime is a pre-configured virtual machine image optimized for use within Databricks clusters. It includes a set of core components, such as Apache
Spark, Delta Lake, and other essential system libraries. Delta Lake enhances traditional data lakes by providing transactional guarantees similar to those found in
operational databases, thereby ensuring improved data reliability and consistency.

#### **4. Cloud infrastructure**:
At the foundation of the Databricks lakehouse architecture lies the cloud infrastructure layer. Databricks is a multi-cloud platform, meaning it is available on
major cloud service providers, including Microsoft Azure, Amazon Web Services (AWS), and Google Cloud Platform (GCP). This layer is responsible for providing the
underlying hardware resources that Databricks accesses on behalf of users. It enables the provisioning of essential components, such as storage, networking, and the
virtual machines (VMs) or nodes that form the backbone of a computing cluster running Databricks Runtime.

## Deployment of Databricks Resources
When deploying Databricks resources within your cloud provider’s environment, the architecture is divided into two high-level components: the control plane and the data plane.

<img width="400" height="500" alt="image" src="https://github.com/user-attachments/assets/8d36333f-cf5d-4f62-84f7-7735ebf4885b" />


#### **1. Control plane**: 
The control plane is managed by Databricks and hosts various platform services within the Databricks account. When you create a Databricks workspace, it is deployed within the control plane, along with essential services such as the Databricks user interface (UI), cluster manager, workflow service, and notebooks. Thus, the control plane handles tasks such as workspace management, cluster provisioning, and job scheduling. It also provides the interface through which users interact with the platform, including the web-based notebooks, the Databricks REST API, and the command-line interface (CLI).

#### **2. Data plane**:
The data plane, on the other hand, resides within the user’s own cloud subscription. This is where actual storage and classic compute resources (non-serverless) are provisioned and managed. When a user sets up a Spark cluster, the virtual machines that comprise the cluster are deployed in the data plane, within the user’s cloud account. Similarly, storage resources, such as those used by the Databricks File System (DBFS) or Unity Catalog, are also deployed in the data plane.

## Apache Spark

| Feature                   | Traditional Data Processing Engine                                           | Apache Spark on Databricks                                                                             |
| ------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| **Processing Model**      | Processes data **sequentially** or with limited parallelism                  | Processes data in **parallel across distributed cluster nodes**                                        |
| **Scalability**           | Scaling is hardware-bound (vertical scaling → add more power to one machine) | Cloud-native **horizontal scaling** → add/remove nodes dynamically                                     |
| **Data Storage/Access**   | Relies heavily on **disk I/O** (slow for large/iterative tasks)              | Optimized for **in-memory processing**, reducing read/write overhead                                   |
| **Programming Languages** | Often limited to **SQL** or vendor-specific languages                        | Supports **Python, SQL, Scala, R, Java** seamlessly                                                    |
| **Workload Types**        | Primarily **batch-oriented** (historical analysis only)                      | Handles both **batch** (historical) and **streaming** (real-time)                                      |
| **Data Types Supported**  | Best with **structured data** (tables, rows, columns)                        | Handles **structured, semi-structured, and unstructured data** (CSV, JSON, images, video, nested data) |
| **Performance**           | Slower for complex analytics (due to repeated disk reads/writes)             | Faster due to **distributed + in-memory execution**                                                    |
| **Integration**           | Typically tied to on-premise systems                                         | Deeply integrated with **cloud-based clusters and Delta Lake** for modern data pipelines               |

## Databricks File System (DBFS)
The DBFS acts as an abstraction layer that simplifies file management across the distributed environment. It allows users to interact with cloud files as if they were stored on a local file system. When a file is created in a Databricks cluster and stored in the DBFS, it is actually persisted in the underlying cloud storage associated with your cloud provider. This design ensures that data remains durable and accessible, even after the Spark cluster is terminated.

## Databricks Workspace Essentials
**1. Home directory**:
The Home directory is your default location within the workspace. It is personalized to each user’s personal directory, providing a semi-private space where you can store your files and folders.

**2. Workspace directory**:
This is the root folder that contains all users’ personal directories. From here, you can also access your Home directory by going to Users >user_name.

**3. Repos**:
This is the legacy service used for integrating your workspace with Git repositories. It has now been replaced by Git folders.

**4. Trash**
This folder contains deleted items, which are retained for 30 days before being permanently removed.

## Clusters
Clusters in Databricks form the backbone of data processing and analytics on the platform. A cluster is essentially a collection of computers, often referred to as nodes, instances, or virtual machines, working together as a single entity. In the context of Apache Spark, which powers Databricks, a cluster comprises a master node known as the driver and several worker nodes, as illustrated in the Figure below. The driver node is primarily responsible for orchestrating the activities of the worker nodes, which execute tasks in parallel, thereby enabling efficient processing of large-scale data.

<img width="301" height="348" alt="image" src="https://github.com/user-attachments/assets/a7769bd0-527c-402e-829f-1f0a4b078dc4" />

Databricks offers two primary types of clusters: all-purpose clusters and job clusters. Each serves distinct purposes and use cases, tailored to different stages of the data engineering and analytics lifecycle.

| Feature             | All-purpose Cluster                         | Job Cluster                                |
| ------------------- | ------------------------------------------- | ------------------------------------------ |
| **Usage**           | Interactive development and data analysis   | Automated job execution                    |
| **Management**      | Manually created and managed by the user    | Automatically created by the job scheduler |
| **Termination**     | Manual or auto-termination after inactivity | Automatic termination upon task completion |
| **Cost Efficiency** | Comes at a higher expense                   | Less expensive                             |

### Databricks Pools
In addition to offering various types of **clusters**, Databricks provides **cluster pools** to further optimize **resource usage** and reduce **operational latency**. Cluster pools are a powerful tool for users who need to minimize the **time** it takes to **spin up clusters**, especially in environments where job execution speed is critical.

A **cluster pool** in Databricks is essentially a **group of pre-configured, idle virtual machines** that are ready to be assigned to clusters as needed. The primary advantage of using a **cluster pool** is the **reduction** in both **cluster start time** and **autoscaling time** whenever there are **available nodes** in the pool. This can be particularly beneficial in scenarios where time is a critical factor, such as in **automated report generation** and **real-time data processing** tasks.

While cluster pools offer significant **operational benefits**, they come with important **cost considerations**. It’s essential to understand that even though Databricks itself does not charge for the **idle instances** in a pool, your cloud provider does. This is because these instances, although **idle**, are actively **running** on your cloud infrastructure, and as such, they incur **standard compute costs**.

### Creating All-Purpose Clusters
This is done in the compute tab from the left siderbar in your databricks. 

**Configuring the cluster: Single-node versus multi-node**

You configure your cluster to a **single-node cluster**, it will operate with just a driver node, eliminating the need for additional worker nodes. In this configuration, the driver handles both driver and worker responsibilities, executing all Spark jobs on a single machine. This setup is more cost effective as it consumes fewer resources. While **Multi-node cluster** is used when you want to handle larger datasets or more complex processing tasks, you can opt for a multi-node cluster, which includes one driver node and multiple worker nodes. This setup allows parallel processing, making it suitable for heavier workloads.

**Configuring the access mode**

Databricks clusters offer different access modes depending on how the cluster is intended to be used: **Shared access mode**, this allows multiple users to share the cluster simultaneously but restricts workloads to SQL and Python only. Shared clusters are useful in collaborative environments where several users need to access the same cluster. **Single user mode**, this mode is appropriate if you are the only one using the cluster. It ensures that the cluster resources are dedicated solely to your tasks, potentially improving performance and efficiency.

**Configuring Performance:**

>**1. Selecting the Databricks Runtime version:**
>The Databricks Runtime version is a critical choice, as it determines the software environment in which your clusters will operate. Databricks Runtime is a pre->configured virtual machine image that includes specific versions of Apache Spark, Scala, and various other libraries essential for data processing.
>
>**2. Enabling Photon:**
>Photon is an optional feature you can enable to further enhance your cluster’s performance.  Photon is a high-speed query engine developed in C++, designed to accelerate the execution of SQL queries in Spark. Enabling Photon is particularly beneficial for workloads that involve heavy SQL processing or operations with many files, as it can significantly reduce query execution times and enhance overall performance. However, it’s essential to consider the additional costs associated with this feature.
>
>**3. Configuring worker nodes:**
>Worker nodes are the backbone of a multi-node cluster, responsible for processing the distributed tasks assigned by the driver node. **VM size selection**
>Databricks allows you to choose from various virtual machine types and sizes provided by your cloud provider (e.g., Azure). These differ in terms of CPU cores,
>memory, and storage options, which should be selected based on the specific demands of your workloads. For simplicity, you may choose to keep the default VM
>size. **Number of workers** Databricks offers an autoscaling feature, which dynamically adjusts the number of workers based on the cluster’s workload. Enabled
>by default, the “Enable autoscaling” option allows you to specify a minimum and maximum range for the number of workers. Databricks will automatically increase
>or decrease the number of worker nodes within this range based on demand. Alternatively, you can disable autoscaling and set a fixed number of workers, such as
>3, ensuring that the cluster always operates with the specified resources regardless of changes in workload.
>
>**4. Configuring the driver node:**
>After configuring the worker nodes, you can set the configuration for the driver node, which coordinates all tasks across the cluster. You can either choose a
>different configuration for the driver or simply match it with the worker nodes, depending on your workload requirements.
>
>**5. Enabling auto-termination:**
>To manage costs and optimize resource usage, Databricks provides an auto-termination feature, which is also enabled by default. By setting a specific duration
>of inactivity (e.g., 30 minutes), you can ensure that the cluster automatically shuts down if it remains idle for that period. This feature is particularly
>useful in preventing unnecessary charges for clusters that are no longer in use.

### Managing Your Cluster
Once your cluster in Databricks is provisioned and running,  indicated by a fully green circle next to its name, you have several options for managing and monitoring it. To access your cluster at any time, simply navigate to the Compute tab in the left sidebar of your Databricks workspace. Effective cluster management goes beyond just starting and stopping the cluster. Databricks provides tools to monitor the cluster’s activity and troubleshoot any issues.  
>**1. Event log:**
>The “Event log” records all significant actions related to the cluster, such as when the cluster was created, terminated, edited, or encountered any errors.
>This detailed tracking enables effective monitoring and troubleshooting of cluster activities.
>
>**2. Spark UI:**
>The Spark UI provides a comprehensive interface for monitoring and debugging Apache Spark applications. It provides detailed insights into job execution,
>stages, and tasks that enable you to easily track performance and identify bottlenecks.
>
>**3. Driver logs:**
>“Driver logs” contains logs generated by the driver node within the cluster. This log captures output from the notebooks and libraries running on the cluster,
>making it an essential tool for diagnosing and resolving issues during development.


### Magic Commands
Magic commands in Databricks notebooks are special cell instructions that provide additional functionality in the notebook environment. These commands, which are prefixed with a %, allow you to execute tasks that go beyond standard code execution. Let’s explore these commands in detail, highlighting their benefits and how to use them effectively. For example, if your notebook’s default language is Python, but you need to run a SQL query, you would use the %sql magic command. This command instructs the notebook to interpret and execute the cell as SQL code:
```sql
%sql
SELECT "Hello world from SQL!"
```
**Run magic command**
The %run magic command in Databricks notebooks is a powerful tool that allows you to execute another notebook within the current notebook. This feature is particularly useful for supporting code modularity and reusability.
```bash
%run ./other_notebook
```
>E.g., If `School-Setup` contains:
```python
school_name = "Greenwood High"
def greet():
    print(f"Welcome to {school_name}!")
```
Then executes `%run` another notebook as if its contents were copied into the current one.
It’s often used for setup notebooks that contain shared functions, variables, or environment configuration.
```python
%run ../Includes/School-Setup
```
`../Includes/School-Setup` is the relative path to another notebook.
When this line runs, all the code inside School-Setup (probably definitions, imports, variables, maybe data mount configs) gets executed in your current notebook’s context.
That means any variables, functions, or Spark configs defined there will be available to the rest of your notebook.
```python
greet()  # → Welcome to Greenwood High!
```


**FS magic command**
The %fs magic command provides a simple way to execute file system operations directly within your notebook cells. This command allows you to perform various tasks, such as copying, moving, and deleting files and directories within your cloud storage. One of the most common uses of the %fs magic command is listing the contents of a directory. For instance, if you want to explore your datasets directory, you can use the following command:
```bash
%fs ls '/your-datasets'
```

**Databricks Utilities**
Databricks Utilities (dbutils) provides a range of utility commands for interacting with different services and tools within Databricks, including the file system (dbutils.fs). If you’re interested in a specific utility within dbutils, you can request detailed help for that particular module. For example, if you want to learn more about the file system commands provided by dbutils.fs, you can use this:
```pyspark
dbutils.fs.help()
```
To achieve similar functionality as the %fs command using dbutils, you can list the contents of the same directory with the following code:

 ```pyspark
files = dbutils.fs.ls("/databricks-datasets/")
```
This command not only lists the files but also stores the output in a variable (files), which can be further manipulated within your code.

>Note: Choosing between the %fs magic command and dbutils depends on the complexity and requirements of your task. If you need to perform a quick, one-off file
>system operation, the %fs magic command is straightforward and easy to use. For more complex tasks, especially when you need to manipulate the output
>programmatically, dbutils is the better choice. It allows you to store the results in variables, apply conditional logic, loop through files, and more—all within your Python code.

### Versioning Databrick Notebook

**Accessing version history:**

To access the **version history** of a **notebook**, look for the **Version history icon** located in the **right sidebar** of the **notebook editor**. The **version history panel** shows a **chronological list** of all changes made to the notebook, with each entry representing an **auto-saved version** that captures the notebook’s state at that point in time. To **restore a previous version**, simply select the desired version from the list and click **Restore this version**, which reverts the current notebook to the selected state and undoes any changes made since then. While this feature is useful for **tracking changes**, it has **limitations**, especially in **complex** or **collaborative projects**, since it lacks advanced capabilities such as **merging** or **branching**, and the history can be **deleted**. For a more robust solution, Databricks offers **Git integration**, providing enhanced **version control** capabilities.

**Versioning with Git:**

Databricks offers **Git integration**, allowing users to manage their data projects using familiar **Git workflows** such as **branching**, **merging**, **committing**, and **pushing changes** to **remote repositories**. This feature is particularly beneficial for users who need to manage **complex projects**, **collaborate** with team members, or maintain a **history of changes** in a more controlled and secure manner than what the basic **notebook versioning** can offer. This seamless integration is facilitated through **Git folders** (formerly known as **Databricks Repos**), which enable **source control** directly into your **Databricks workspace**. With Git folders, you can **synchronize your code** with remote repositories and perform common **Git operations**. You can only create the Git folder(Git repo) in your Git acoount, Databricks acts like a **local working copy** of the **Git repo** — you’re not creating a standalone folder that exists independently in Databricks; it’s always tied to a Git repository.


