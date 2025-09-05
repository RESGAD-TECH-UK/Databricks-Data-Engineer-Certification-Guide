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

