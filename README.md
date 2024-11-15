# Snowflake-car-rental-pipeline-SCD2

## Table of Contents
1. [Introduction](#introduction)
2. [Technologies Used](#technologies-used)
3. [Project Workflow](#project-workflow)
4. [Setup Instructions](#setup-instructions)
5. [Challenges Faced](#challenges-faced)
6. [Outcome](#outcome)
7. [Real-World Usefulness](#real-world-usefulness)
8. [Future Improvements](#future-improvements)
9. [Steps to Run the Pipeline](#steps-to-run-the-pipeline)
10. [Conclusion](#conclusion)

## Introduction

This project implements an end-to-end data pipeline for processing car rental data. The pipeline ingests daily rental data, performs Slowly Changing Dimension (SCD2) merges on the customer dimension table, transforms the data, and loads it into Snowflake’s `rentals_fact` table. The pipeline leverages **PySpark** for data transformation, **GCP Dataproc** for cluster management, **Airflow** for orchestration, and **Snowflake** as the data warehouse.

## Technologies Used

- **Python**: For data processing and transformations.
- **PySpark**: For distributed data processing and transformation.
- **Airflow**: For scheduling and orchestrating the data pipeline.
- **GCP Dataproc**: For running Spark jobs in a managed Hadoop environment.
- **Snowflake**: For data storage, utilizing dimension and fact tables for car rental data.
- **Google Cloud Storage**: For storing daily car rental and customer data files.

## Project Workflow

### 1. **Ingest Raw Data**:
   - Raw car rental and customer data (in JSON format) is ingested from Google Cloud Storage (GCS).

### 2. **Dimension Tables Creation**:
   The following dimension tables are created in Snowflake:

   - **location_dim**: Contains location details for car rentals.
   - **car_dim**: Contains information about cars.
   - **date_dim**: Contains date-related information for time-based analysis.
   - **customer_dim**: A Slowly Changing Dimension (SCD2) table that tracks changes in customer data.
   
   The customer dimension table (`customer_dim`) uses the **SCD2 merge** strategy, which handles customer data changes over time.

### 3. **Data Transformation**:
   - The raw data is cleaned and transformed, including calculations of rental amounts, rental durations, and additional metrics.
   - The transformed data is joined with the Snowflake dimension tables to enrich the dataset.

### 4. **Fact Table Creation**:
   - **rentals_fact**: The fact table that contains detailed rental records, including customer, car, and rental information.

### 5. **Airflow DAG**:
   - The **Airflow DAG** orchestrates the entire pipeline, including triggering the SCD2 merge, inserting new records into the `customer_dim` table, and submitting the PySpark job on GCP Dataproc to transform and load data into the `rentals_fact` table.

### 6. **SCD2 Merge**:
   - The SCD2 merge handles customer data updates by closing the old records and inserting new records with updated customer information.

## Setup Instructions

### Prerequisites

- **Google Cloud**: GCP account with Dataproc and Cloud Storage permissions, and a Google Cloud Storage bucket to store the rental data.
- **Snowflake**: Snowflake account with a warehouse and schema setup. 
- **Apache Airflow**: Airflow instance configured with connections to Google Cloud and Snowflake.

### Steps to Set Up

#### Snowflake Setup:
1. Run `snowflake_dwh_setup.sql` in your Snowflake environment to create the necessary database, schema, and tables.
   
#### GCP Setup:
1. Create a GCS bucket (e.g., `car_rental_daily_data/`) to store the rental data.
2. Upload daily car rental data (e.g., `car_rental_20240802.json`) to the GCS bucket.

#### Airflow Setup:
1. Set up Airflow with the necessary connections for Google Cloud (`google_cloud_default`) and Snowflake (`snowflake_conn_v2`).
2. Place `airflow_dag.py` in the Airflow DAGs directory.
3. Ensure that the Airflow instance has access to the Dataproc cluster for running PySpark jobs.

### Running the Pipeline:
1. Trigger the Airflow DAG manually or schedule it to run daily.
2. Monitor the execution of the pipeline through Airflow’s web interface.

## Challenges Faced

- **Handling SCD2 Merges**: Implementing SCD2 merges on the `customer_dim` table required careful handling of historical records while ensuring that new customer records were inserted correctly without data duplication.
- **Large Data Volumes**: The volume of car rental data required efficient batch processing techniques to ensure that the data was processed within an acceptable timeframe.

## Outcome

This project automates the process of ingesting, transforming, and loading car rental data into Snowflake, making it ready for reporting and analysis. The pipeline efficiently processes data on a daily basis, performing transformations and handling SCD2 merges, and the data is now available in Snowflake’s `rentals_fact` table.

## Real-World Usefulness

This pipeline is beneficial for businesses in the car rental industry. It provides a scalable solution for managing and analyzing rental data, tracking customer behavior, calculating rental metrics, and ensuring data consistency over time. The integration of Snowflake, Dataproc, and Airflow also ensures the solution is scalable and reliable for enterprise-level operations.

## Future Improvements

- **Real-Time Data Processing**: Implementing a real-time data processing pipeline using **Apache Kafka** and **Apache Flink** could improve the timeliness of data availability for real-time reporting.
- **Data Quality Monitoring**: Adding automated data quality checks to ensure that the data being ingested and transformed meets the required standards.
- **Enhanced SCD2 Logic**: Incorporating additional logic for managing other slowly changing dimensions (e.g., rental pricing) in the pipeline.

## Steps to Run the Pipeline

1. Set up the prerequisites in **Google Cloud**, **Snowflake**, and **Airflow** as outlined in the [Setup Instructions](#setup-instructions).
2. Upload daily car rental data to GCS.
3. Trigger the **Airflow DAG** to run the pipeline manually or schedule it for daily execution.
4. Monitor the status of the pipeline through Airflow’s web interface.
5. Check Snowflake for the updated data in the **`rentals_fact`** and **`customer_dim`** tables.

### Running the PySpark Job Manually
You can also run the PySpark job manually on Dataproc using the following command:

```bash
gcloud dataproc jobs submit pyspark --cluster YOUR_CLUSTER_NAME \
  --region YOUR_REGION \
  --jars gs://snowflake_projects/snowflake_jars/spark-snowflake_2.12-2.15.0-spark_3.4.jar,gs://snowflake_projects/snowflake_jars/snowflake-jdbc-3.16.0.jar \
  gs://snowflake_projects/spark_job/spark_job.py -- --date '20240802'
