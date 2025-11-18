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
