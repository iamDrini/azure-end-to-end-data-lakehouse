# Azure End-to-End Data Lakehouse

This repository demonstrates an end-to-end cloud data pipeline that migrates on-premises SQL data into a modern Azure Data Lakehouse. The architecture leverages Azure Databricks for data processing following the Medallion architecture (Bronze, Silver, Gold layers) and Azure Synapse Analytics for serving the data to BI tools like Power BI.

## Project Architecture

The pipeline follows a multi-layered approach to ingest, process, and serve data:

1.  **Data Ingestion**: Data from an on-premises SQL Server database (specifically, the `SalesLT` schema) is first extracted and landed in a `bronze` container within Azure Data Lake Storage (ADLS) Gen2. This raw data is stored in Parquet format.

2.  **Bronze to Silver Layer (Cleansing & Conforming)**: The `bronze to silver.ipynb` notebook reads the raw Parquet files from the Bronze layer. It performs initial data cleansing tasks, such as standardizing date/time formats across all tables. The processed data is then written to the `silver` container in Delta Lake format, providing ACID transactions and schema enforcement.

3.  **Silver to Gold Layer (Modeling & Business Logic)**: The `silver to gold.ipynb` notebook reads the conformed Delta tables from the Silver layer. It applies final transformations to create a business-ready, curated dataset. In this project, this includes standardizing all column names to `snake_case` for consistency. The resulting high-quality data is saved to the `gold` container, also in Delta Lake format. The Gold layer serves as the single source of truth for analytics and reporting.

4.  **Serving Layer (Analytics & Reporting)**: An Azure Synapse Analytics serverless SQL pool is used to create a relational layer on top of the Delta files in the Gold layer. The `createView.sql` stored procedure dynamically generates T-SQL views over the data in ADLS, making it directly queryable using standard SQL without needing to move the data.

5.  **Visualization**: Downstream tools like Power BI can connect to the Azure Synapse serverless SQL endpoint to query the views and build interactive reports and dashboards.

## Repository Components

This repository contains the core scripts for data transformation and serving:

*   **`df_lookup.sql`**: A SQL query to identify all tables within the `SalesLT` schema on the source SQL Server. This is used to define the scope of the data migration.

*   **`bronze to silver.ipynb`**: An Azure Databricks notebook that:
    *   Reads raw `.parquet` files from the `bronze` ADLS container.
    *   Casts and standardizes all columns containing "Date" in their name to a `yyyy-MM-dd` format.
    *   Writes the cleaned data as Delta tables into the `silver` container.

*   **`silver to gold.ipynb`**: An Azure Databricks notebook that:
    *   Reads cleaned Delta tables from the `silver` container.
    *   Applies a function to convert all `CamelCase` column names to `snake_case`.
    *   Writes the final, curated data as Delta tables into the `gold` container.

*   **`createView.sql`**: A stored procedure for an Azure Synapse Analytics serverless SQL pool. It accepts a table name and dynamically creates a queryable view on top of the corresponding Delta files in the `gold` container using `OPENROWSET`.

## Technologies Used

*   **Azure Data Lake Storage Gen2**: Scalable storage for the Bronze, Silver, and Gold layers of the lakehouse.
*   **Azure Databricks**: A collaborative analytics platform used for running the PySpark notebooks for data transformation.
*   **Delta Lake**: An open-source storage framework that brings reliability, performance, and ACID transactions to data lakes.
*   **Azure Synapse Analytics (Serverless SQL Pool)**: A query service for data in the data lake, enabling T-SQL-based analytics.
*   **Power BI**: A business analytics service for visualizing data and sharing insights.
