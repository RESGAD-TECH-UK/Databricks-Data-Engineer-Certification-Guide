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
This is the legacy service used for integrating your workspace with Git repositories. It has now been replaced by Git folders, which we cover in detail at the end of this chapter in “Creating Git Folders”.

**4. Trash**
This folder contains deleted items, which are retained for 30 days before being permanently removed.

## Clusters
Clusters in Databricks form the backbone of data processing and analytics on the platform. A cluster is essentially a collection of computers, often referred to as nodes, instances, or virtual machines, working together as a single entity. In the context of Apache Spark, which powers Databricks, a cluster comprises a master node known as the driver and several worker nodes, as illustrated in Figure below. The driver node is primarily responsible for orchestrating the activities of the worker nodes, which execute tasks in parallel, thereby enabling efficient processing of large-scale data.

<img width="301" height="348" alt="image" src="https://github.com/user-attachments/assets/a7769bd0-527c-402e-829f-1f0a4b078dc4" />

