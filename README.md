
# 📊 End-to-End Netflix Data Pipeline (Azure + Databricks)
This repository contains a highly scalable, automated, and governed data pipeline, designed to ensure efficient ingestion, processing, and transformation of Netflix data. It combines modern technologies such as Azure Data Factory, Databricks, Delta Live Tables (DLT), and Unity Catalog, providing a high-performance, reliable, and well-governed data environment.

## 🧰 Tech Stack

- Azure Data Factory (ADF)
- Azure Databricks
- PySpark
- Delta Lake
- Delta Live Tables (DLT)
- Unity Catalog
- ADLS Gen2


🚀 Project Highlights and Benefits

- ✅ Centralized Data Governance: Uses Unity Catalog for granular access control.
  
- ✅ Data Quality & Observability: Delta Live Tables (DLT) ensures data quality through built-in validation rules and continuous monitoring.
  
- ✅ Full Automation: Databricks Jobs orchestrate all stages of the pipeline.
  
- ✅ Scalability and Efficiency: The use of AutoLoader and Delta Lake optimizes ingestion and storage, reducing costs and processing time.
  
- ✅ Flexibility and Reliability: The pipeline supports batch and streaming processing, enabling near real-time ingestion.

## 📌 Project Architecture
The pipeline follows the **Bronze - Silver - Gold** architecture, using modern tools such as **Azure Data Factory (ADF)** for ingestion, **AutoLoader for incremental processing**, **Databricks Jobs** for orchestration, and **DLT for automation and data quality control**.

![image](https://github.com/user-attachments/assets/82981dad-da4d-4b56-b2f9-b63ecfebab4b)

### 🔄 Data Flow

1. **Azure Data Factory (ADF)** collects data from GitHub and stores it in **Azure Data Lake Gen2**.
2. **AutoLoader in Databricks** reads and processes new data incrementally into the **Bronze** layer.
3. **Transformation in the Silver layer**: data cleansing, standardization, and enrichment.
4. **Delta Live Tables (DLT)** structures and validates data in the **Gold** layer, ensuring quality and compliance.
5. **Unity Catalog** manages tables and access for centralized data governance.
6. **Processed data is made available for analysis** in **Power BI** and **Azure Synapse Analytics**.

---

## 🔄 **Integration with Azure Data Factory**

- **Ingestion:** ADF retrieves CSV files from GitHub and loads them into **Data Lake Gen2**.
- **Orchestration:** Triggers Databricks Jobs to process the data.
- **Monitoring:** Configured for error alerts via email.
- **Pipeline automation**, ensuring optimized execution.

---
## 🏗️ **Unity Catalog and External Locations**

- **Unity Catalog** centralizes data governance and provides unified access control.
- All tables are managed within the `netflix_unity_metastore`.
- External locations are configured to store data in **Azure Data Lake Gen2**, ensuring security and traceability.

![image](https://github.com/user-attachments/assets/da428e91-2694-4991-ac8d-82378e3e628d)


## 🚀 Notebooks and Data Processing

### 1️⃣ **Bronze Layer - AutoLoader**

File: `1_autoloader.ipynb`

- **AutoLoader** performs automatic and incremental ingestion of CSV files.
- Enables scalability for large data volumes and reduces operational costs.
- Automatically detects new files without the need for manual monitoring.
- Data is stored in the **Bronze** layer in Data Lake Gen2.

```python
checkpoint_location = "abfss://container@storageaccountl.dfs.core.windows.net/checkpoint"
df = spark.readStream\
    .format('cloudFiles')\
    .option('cloudFiles.format', 'csv')\
    .option('cloudFiles.schemaLocation', checkpoint_location)\
    .load('abfss://raw@storageaccountl.dfs.core.windows.net')
```

### 2️⃣ **Silver Layer - Transformations**

File: `2_silver.ipynb`

- Performs data cleansing, null handling, and data type adjustments.
- Creation of derived columns (`Shorttitle`, `type_flag`, etc.).
- Stores refined data in the **Silver** layer in **Delta** format.

```python
df = df.withColumn('Shorttitle',split(col('title'),':')[0])
df = df.withColumn('type_flag',when(col('type') == 'Movie',1)\
        .when(col('type') == 'TV Show',2).otherwise(0))
df.write.format('delta')\
     .mode('overwrite')\
     .option('path', 'abfss://container@storageaccountl.dfs.core.windows.net/silver/netflix_titles')\
     .save()
```
## 🏗️ **Databricks Jobs**

**Jobs** in Databricks ensure the automation of the data pipeline.

### 🔹 **Job 1 - Silver Processing** (`job_silver.json`)
- Executes `3_lookupnotebook.ipynb` to retrieve metadata.
- Uses `2_silver.ipynb` to process different **Silver** tables with dynamic parameters.
- Executes all tables from the `my_arr` array.

![image](https://github.com/user-attachments/assets/e1e5c00b-5880-4568-872f-48517bd75789)

### 🔹 **Job 2 - Conditional Check**
- Executes `5_lookupNotebook.ipynb` to check the date.
- Depending on the day of the week, decides which notebook to execute (**4_Silver.ipynb** or **6_false_notebook.ipynb**).

![image](https://github.com/user-attachments/assets/39d092bb-1fdc-471e-a4e9-2ade7e638d86)

### 3️⃣ **Gold Layer - Delta Live Tables (DLT)**

![image](https://github.com/user-attachments/assets/c810bd87-00bf-47a6-bc81-3e2ff1652239)

File: `7_DLT_Notebook.ipynb`

- **DLT transforms the Silver layer into Gold, applying validations and ensuring Data Quality.**
- **Quality rules are automatically applied**, rejecting invalid data and ensuring consistency.
- **Automatic scalability** for large data volumes.
- **Data governance** with change tracking.


The use of Delta Live Tables (DLT) makes this pipeline highly efficient, ensuring data quality from ingestion to the consumption layer.

#### **Advantages of Delta Live Tables (DLT):**
- ✅ **Full automation of the data pipeline** – no need to manually manage tasks.
- ✅ **Built-in data validation and quality checks** using `@dlt.expect_all_or_drop()`.
- ✅ **Complete history of changes** – simplifies auditing and compliance.
- ✅ **Optimized and scalable execution**, reducing operational costs.


The scalability and optimized processing of Databricks reduce execution time and optimize storage and compute costs. With full traceability of transformations, the pipeline maintains a complete history of data changes, facilitating auditing and analysis.
