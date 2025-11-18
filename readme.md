# Mastering the Medallion Architecture: From Bronze Ingestion to Silver Transformations and Gold-Level Insights in Azure Data Factory

Every strong data platform needs a backbone - something that quietly organizes the chaos and turns scattered information into something meaningful. In my case, that backbone was the Medallion Architecture. The Medallion Architecture turns chaotic raw data into trusted insights by moving it through three layers: Bronze, where data lands untouched; Silver, where it's cleaned and refined; and Gold, where it becomes business-ready intelligence.

## 🛠 The Medallion Architecture: My Blueprint: 
 Medallion Architecture, consisting of three layers:
- 🥉 Bronze - The raw data landing zone
- 🥈 Silver - The cleaned, structured, transformed data
- 🥇 Gold - The business-ready, insight-rich datasets

## 🥉 Bronze Layer - Collecting the Raw, Messy Data
The Bronze layer was my project's first destination - the place where every piece of raw data, in all its messy forms, safely landed.
And because my data sources were diverse, I ended up building three different ingestion paths, all feeding into this single zone.

### 1️⃣ Dynamic File Ingestion (CSV Files → ADLS)
My first challenge was handling multiple CSV files without building a separate pipeline for each table.
So, I made the ingestion flexible by designing a pipeline that could scale itself:
- A files array parameter listed every source file.
- Mapping objects defined column-level transformations.
- ForEach loop processed each file dynamically.
  
https://medium.com/@sakibul1605/building-a-dynamic-ingestion-pipeline-in-azure-data-factory-881d4e89ff2f

### 2️⃣ API Ingestion (GitHub Raw URL → ADLS)
Next came an API endpoint hosted on GitHub.
I built a simple  pipeline:
- A Web Activity checked if the API was reachable.
- A REST/HTTP dataset pulled CSV/JSON content.
- A Copy Activity landed the response directly into ADLS.

https://medium.com/@sakibul1605/how-i-ingested-api-data-into-azure-using-azure-data-factory-adf-795ec0249662

### 3️⃣ Incremental SQL Ingestion (Watermark Pattern)
This was the most exciting part of Bronze: building an ingestion flow that only loads new records from Azure SQL DB- no duplicates, no reprocessing.
Here's how I did it:
- Read the previous load timestamp from lastload.json
- Use that timestamp in a dynamic SQL query to fetch only new rows
- Load the incremental records into the Bronze layer
- Update lastload.json with the latest timestamp after a successful run

https://medium.com/@sakibul1605/how-i-built-an-incremental-sql-ingestion-pipeline-in-azure-data-factory-9fcd7ce11efc



### 4️⃣ Orchestration - One Pipeline to Rule Them All
After building three separate ingestion pipelines, I needed a conductor.
So I created one parent orchestration pipeline that:
- Runs the CSV ingestion
- Runs the API ingestion
- Runs the SQL incremental ingestion
- On failure → Triggers Logic Apps to send me an instant email alert

With ingestion fully automated and monitored, Bronze stood strong as the raw source of data, perfectly prepared for the transformations waiting in the Silver layer.
