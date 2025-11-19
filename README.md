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


## 🥈 Silver Layer - Transforming Raw Bronze Data into Analytics-Ready Tables with ADF Data Flows
Once my Bronze layer was ready and the raw data had safely landed in ADLS, it was time to turn that raw data into something analytics-friendly.
This is where the Silver Layer comes in.
In this stage, my goal was clear:
- 👉 Clean, transform, standardize, and load dimension + fact tables into Delta format.
- 👉 Apply business rules and ensure schema consistency.
- 👉 Support incremental upserts using Delta Lake.
  
And I did all of this using Azure Data Factory Mapping Data Flows, which behave like PySpark transformations but with a visual interface.
### 🔹 Designing the Silver Layer Data Flow
I created a data flow that contained five parallel transformation streams - each representing one table:
- ✈️ DimAirline
- 🧍 DimPassenger
- 🛫 DimFlight
- 🛬 DimAirport
- 📒 FactBookings

Instead of creating five separate pipelines, I built one unified data flow with five transformation streams running side-by-side.


### ✨ Step 1: Reading the Bronze Data (Source)
Each stream starts with a Source block:
- Input type: Delta or CSV/Parquet
- Source linked service: ADLS Gen2

This simply loads the raw - or previously ingested - records.
### ✨ Step 2: Bringing the Data to Life (Transformations)
And this is where the real magic happened.
ADF's transformation blocks feel like small LEGO pieces - you pick the right ones and snap them together until the structure looks exactly the way you want.
Here's what I used the most:
#### 🔧 derivedColumn
Create or update fields, such as:
Extracting the first name from a full name
Creating new business logic attributes
Standardizing(LoweCase/UpperCase) field values

#### 💡 cast
Fixing data types, for example :ticket_cost → decimal, booking_date → date, flight_duration_mins → integer
#### 🚫 filter
Filtering is where I remove data that shouldn't move forward. It's the difference between clean datasets and noisy, unpredictable ones. The filter stage ensures that only "good data" moves to the next step.
### ✨ Step 3: Preparing for Incremental Loads (AlterRow)
Now, this is where the Silver layer becomes powerful. To support incremental ingestion, I told ADF exactly how to treat each row:
- upsert() for updates & insert
- insert() for new values
- (delete/update rules optional for future enhancement)

It's basically me telling Delta:
"Match rows using the primary key, update them if anything changed, insert them if they're new."

### ✨ Step 4: Writing Everything into the Silver Layer as Delta Tables (The Polished Output)
Once all the transformations were done, it was time for the most rewarding part - loading the enriched, cleaned data into my Silver layer as Delta tables. This is where raw Bronze data finally becomes analytics-ready.

Delta supports ACID transactions, schema evolution, and time-travel, which ensures that every write operation is consistent and traceable. Most importantly, it allows MERGE (upsert) operations, enabling my Silver tables to update incrementally without rewriting the entire dataset.

For each table stream inside my data flow, I configured a Delta sink:
- I chose Inline Dataset → Delta so the sink writes directly into ADLS.
- Enabled Allow Schema Drift, because data in the real world loves surprises.
- Used Snappy compression to keep the files light and query-efficient.

But the real magic happened under the Settings tab:
 I enabled Upsert, selected the correct primary key column (airline_id, passenger_id, booking_id, etc.), and let Delta's MERGE capabilities handle the rest.

Here's what this gave me - automatically, every time the pipeline runs:
- ✨ New rows are inserted
- ✨ Existing rows are updated (MERGE)
- ✨ Silver stays clean and consistent
  
By the end of this step, every dataset - airlines, passengers, flights, bookings, airports - flowed seamlessly into its own Delta table. And just like that, my Silver layer became a structured, trustworthy, analytics-ready foundation for the Gold layer dashboards.
 - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

## 🥇Crafting the Gold Layer - Turning Clean Data Into Insight
It started in the Bronze layer as raw files, travelled through the Silver layer for cleansing and standardization, and now it was ready for the final transformation - the part that directly supports business decisions. This layer is where data stops being "data" and starts becoming insight.

For my project, the business question was simple:
"Which airlines generate the highest total sales?"

The Gold layer is where I finally answered it.

Step-by-Step Process:
#### ✈️ Bringing the Silver Data Together
I began by pulling in two polished Delta tables from the Silver layer:
- FactBookings (Silver): transaction-level booking data
- DimAirlines (Silver): airline reference information

At this point, both datasets were clean and structured, making them perfect for business modeling.
#### 🔗 Joining Facts With Dimensions
To build a meaningful view, I performed a Left Outer Join between bookings and airline details.
This join enriched every booking record with airline attributes - things like airline name and country.
#### 🧹 Shaping the Data
Next, I used a Select transformation to pick and rename only the columns I needed. 
#### 📊 Aggregating Airline Sales
With clean, joined data ready, it was time to build the insight.

I grouped the data by airline and calculated:
        Total Sales = SUM(ticket_cost)

In just one step, raw bookings turned into an aggregated revenue view.
#### 🏅 Ranking the Airlines
To make the insight more actionable, I applied a ranking transformation using a window function. Each airline received a rank based on its total sales (highest first).
#### ✂️ Filtering the Top 5
From here, I filtered the dataset to only keep: Rank ≤5
This turned a long list of airlines into a concise Top 5 Airlines by Total Sales report.
#### 🗂️ Writing to the Gold Layer
Finally, the refined, aggregated, business-ready dataset was written into the Gold layer using Delta format.


Ultimately, implementing the Medallion Architecture with Azure Data Factory allowed me to build a structured, resilient, and scalable data pipeline - one that reliably converts raw inputs into high-quality analytical outputs.


### How I Integrated Azure Data Factory With GitHub (and Prepared for CI/CD)
After building the full Medallion Architecture pipeline, the final step was making my Data Factory solution production-ready. To do that, I integrated Azure Data Factory with GitHub, enabled proper version control, and set the foundation for future CI/CD deployments.

Here’s a quick summary of how I set it up:

- ✨ Connected ADF to GitHub
From the Manage → Git Configuration section, I linked my Data Factory to GitHub, selected the main branch as my collaboration branch, and let ADF create the adf_publish branch for deployment templates.

- ✨ Worked inside a development branch
To understand the recommended workflow, I created a new branch directly from ADF, added a demo pipeline activity, committed the changes, and kept development isolated from the main branch.

- ✨ Created a Pull Request directly from ADF
ADF automatically generated a PR with all my updates, redirected me to GitHub with everything pre-filled, and I merged the changes into main after reviewing the diffs.

- ✨ Clicked “Publish” to generate deployment templates
Publishing converted the contents of my main branch into ARM templates and committed them to the adf_publish branch. This branch now contains deployment-ready artifacts for Azure DevOps pipelines.

This setup ensures my entire ADF project is version-controlled, traceable, and ready for CI/CD.

I’ve written a detailed, step-by-step breakdown of the entire Git integration and Publish workflow here:

https://medium.com/@sakibul1605/integrating-azure-data-factory-with-github-741180475b8f
