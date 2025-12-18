# 🚀 Azure ETL Pipeline for Amazon Product Reviews Analytics

![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)
![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=for-the-badge&logo=apache-spark&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [Business Problem](#-business-problem)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Data Model](#-data-model)
- [Project Structure](#-project-structure)
- [Pipeline Workflow](#-pipeline-workflow)
- [Setup Instructions](#-setup-instructions)
- [Key Features](#-key-features)
- [Challenges & Solutions](#-challenges--solutions)
- [Future Enhancements](#-future-enhancements)
- [Learning Outcomes](#-learning-outcomes)
- [Acknowledgments](#-acknowledgments)

---

## 🎯 Project Overview

An end-to-end **Azure-based ETL data pipeline** that processes **571 million Amazon product reviews** and **48 million product metadata records** from 33 categories, transforming raw JSON data into an analytics-ready **Star Schema** data warehouse.

This project demonstrates modern data engineering best practices including:
- **Medallion Architecture** (Bronze → Silver → Gold)
- **Dimensional Modeling** (Fact and Dimension tables)
- **Cloud-native data processing** at scale
- **Infrastructure as Code** principles

### 🔑 Key Metrics
- **Data Volume:** 571M reviews, 48M products
- **Categories Processed:** 2 (Electronics, Books) - scalable to all 33
- **Pipeline Layers:** 3 (Bronze, Silver, Gold)
- **Storage Format:** Delta Lake (ACID-compliant)
- **Processing Engine:** Apache Spark (PySpark)

---

## 💼 Business Problem

### The Challenge
Amazon's product review data is stored as **unstructured JSON files** that are:
- ❌ **Difficult to query** (nested arrays and dictionaries)
- ❌ **Not optimized** for analytics (no indexing or partitioning)
- ❌ **Contains data quality issues** (duplicates, nulls, invalid values)
- ❌ **Scattered across categories** (33 separate datasets)

### The Solution
Build a **scalable ETL pipeline** that:
- ✅ **Extracts** data from source URLs via Azure Data Factory
- ✅ **Transforms** raw JSON into clean, normalized tables using PySpark
- ✅ **Loads** data into a Star Schema optimized for business intelligence
- ✅ **Enables** fast analytics queries for marketing, product, and data science teams

### Business Value
- 📊 **Marketing:** Analyze sentiment trends and customer satisfaction
- 🛍️ **Product Teams:** Identify top-rated products and improvement areas
- 🤖 **Data Science:** Build recommendation systems and predictive models
- 📈 **Executives:** Create dashboards showing KPIs and performance metrics

---

## 🏗️ Architecture

### Pipeline Architecture

<!-- PLACEHOLDER: Add your architecture diagram here -->
![Pipeline Architecture](./docs/architecture-diagram.png)

### Medallion Architecture Layers
```
┌─────────────────────────────────────────────────────────────┐
│                    🥉 BRONZE LAYER                          │
│                   (Raw Data - Immutable)                     │
├─────────────────────────────────────────────────────────────┤
│ • Format: Compressed JSON (.jsonl.gz)                       │
│ • Purpose: Single source of truth, audit trail              │
│ • Storage: Azure Data Lake Gen2                             │
│ • Size: ~50GB (2 categories)                                │
└─────────────────────────────────────────────────────────────┘
                           ↓
              PySpark Transformations (Databricks)
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                    🥈 SILVER LAYER                          │
│                (Cleaned & Validated Data)                    │
├─────────────────────────────────────────────────────────────┤
│ • Format: Parquet (columnar, compressed)                    │
│ • Purpose: Clean data for analytics                         │
│ • Transformations: Deduplication, null handling, flattening │
│ • Storage: Azure Data Lake Gen2                             │
│ • Size: ~5GB (90% compression)                              │
└─────────────────────────────────────────────────────────────┘
                           ↓
         Dimensional Modeling (Star Schema)
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                    🥇 GOLD LAYER                            │
│              (Business-Ready Star Schema)                    │
├─────────────────────────────────────────────────────────────┤
│ • Format: Delta Lake (ACID transactions)                    │
│ • Model: Star Schema (1 Fact + 4 Dimensions)               │
│ • Purpose: Optimized for BI tools (Power BI, Tableau)      │
│ • Storage: Azure Data Lake Gen2                             │
│ • Query Performance: < 5 seconds for complex aggregations   │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Orchestration** | Azure Data Factory | Pipeline scheduling and workflow management |
| **Storage** | Azure Data Lake Gen2 | Scalable data lake with hierarchical namespace |
| **Compute** | Azure Databricks | Distributed data processing with Apache Spark |
| **Processing** | PySpark | Large-scale data transformations |
| **Storage Format** | Delta Lake | ACID-compliant lakehouse storage layer |
| **Data Format** | Parquet | Columnar storage for analytics |
| **Language** | Python 3.9+ | Notebook development |
| **Version Control** | Git/GitHub | Code repository and collaboration |

### Why These Technologies?

**Azure Data Factory**
- Visual drag-and-drop interface
- Native integration with Azure services
- Cost-effective for batch processing
- Built-in monitoring and error handling

**Azure Data Lake Gen2**
- Hierarchical namespace for organized folder structures
- Low cost (~$0.018/GB/month)
- Seamless integration with Databricks
- Unlimited scalability

**Azure Databricks**
- Managed Spark clusters (no infrastructure management)
- Interactive notebooks for development
- Auto-scaling for cost optimization
- Delta Lake support out-of-the-box

**Delta Lake**
- ACID transactions prevent data corruption
- Time travel for auditing and debugging
- Schema enforcement and evolution
- 3-10x faster than plain Parquet

---

## 📊 Data Model

### Star Schema Design

The Gold layer implements a **Star Schema** with 1 Fact table and 4 Dimension tables:
```
        ┌─────────────────┐
        │  dim_products   │
        │─────────────────│
        │ product_key (PK)│
        │ parent_asin     │
        │ title           │
        │ price           │
        │ average_rating  │
        │ store           │
        │ brand           │
        └────────┬────────┘
                 │
                 │ 1:N
                 │
        ┌────────▼────────┐         ┌──────────────┐
        │  fact_reviews   │────────▶│  dim_users   │
        │─────────────────│   N:1   │──────────────│
        │ review_key (PK) │         │ user_key (PK)│
        │ product_key (FK)│         │ user_id      │
        │ user_key (FK)   │         │ total_reviews│
        │ date_key (FK)   │         │ avg_rating   │
        │ category_key(FK)│         └──────────────┘
        │ rating          │
        │ helpful_vote    │         ┌──────────────┐
        │ verified_purch. │────────▶│  dim_date    │
        │ has_images      │   N:1   │──────────────│
        └────────┬────────┘         │ date_key (PK)│
                 │                  │ year, month  │
                 │ N:1              │ quarter      │
                 │                  │ day_name     │
        ┌────────▼──────────┐       │ is_weekend   │
        │  dim_categories   │       └──────────────┘
        │───────────────────│
        │ category_key (PK) │
        │ category_name     │
        └───────────────────┘
```

### Table Descriptions

#### `fact_reviews` (Fact Table)
**Grain:** One row per customer review

| Column | Type | Description |
|--------|------|-------------|
| `review_key` | BIGINT | Surrogate primary key |
| `product_key` | BIGINT | FK to dim_products |
| `user_key` | BIGINT | FK to dim_users |
| `date_key` | INT | FK to dim_date (YYYYMMDD format) |
| `category_key` | INT | FK to dim_categories |
| `rating` | FLOAT | Star rating (1-5) - **MEASURE** |
| `helpful_vote` | INT | Number of helpful votes - **MEASURE** |
| `review_text_length` | INT | Character count of review - **MEASURE** |
| `verified_purchase` | BOOLEAN | Verified purchase flag |
| `has_images` | BOOLEAN | Review includes images |

#### `dim_products` (Product Dimension)
**Grain:** One row per unique product

| Column | Description |
|--------|-------------|
| `product_key` | Surrogate primary key |
| `parent_asin` | Amazon product identifier (natural key) |
| `title` | Product name |
| `price_cleaned` | Product price (0.0 if missing) |
| `average_rating` | Average rating from metadata |
| `rating_number` | Total number of ratings |
| `store` | Store/brand name |
| `main_category` | Product category |
| `brand` | Product brand |

#### `dim_users` (User Dimension)
**Grain:** One row per unique reviewer

| Column | Description |
|--------|-------------|
| `user_key` | Surrogate primary key |
| `user_id` | Amazon user identifier (natural key) |
| `first_review_date` | Date of first review |
| `total_reviews` | Total number of reviews by user |
| `avg_rating_given` | User's average rating across all products |

#### `dim_date` (Date Dimension)
**Grain:** One row per calendar date

| Column | Description |
|--------|-------------|
| `date_key` | Primary key (YYYYMMDD format: 20231015) |
| `full_date` | Full date value |
| `year`, `quarter`, `month` | Time hierarchies |
| `month_name`, `day_name` | Descriptive attributes |
| `is_weekend` | Weekend flag |

#### `dim_categories` (Category Dimension)
**Grain:** One row per product category

| Column | Description |
|--------|-------------|
| `category_key` | Surrogate primary key |
| `category_name` | Category name (e.g., "Electronics") |

---

## 📁 Project Structure
```
amazon-etl-pipeline/
├── databricks-notebooks/
│   ├── 01_Bronze_to_Silver_Reviews.py       # Reviews cleaning & transformation
│   ├── 02_Bronze_to_Silver_Metadata.py      # Metadata cleaning & transformation
│   └── 03_Silver_to_Gold_Dimensional.py     # Star schema creation
│
├── adf-pipelines/
│   ├── pipeline-config.json                  # ADF pipeline definition
│   └── README.md                             # ADF setup instructions
│
├── docs/
│   ├── architecture-diagram.png              # Architecture diagram
│   ├── erd-diagram.png                       # Entity-relationship diagram
│   └── data-dictionary.md                    # Complete data dictionary
│
├── tests/
│   └── data-quality-tests.py                 # Data validation tests
│
├── .gitignore
├── README.md                                 # This file
└── LICENSE
```

---

## 🔄 Pipeline Workflow

### Step-by-Step Process
```
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: Orchestration Setup (Azure Data Factory)            │
└─────────────────────────────────────────────────────────────┘
   │
   ├─ Pipeline Variable: all_categories = ["Books", "Electronics"]
   │
   └─ ForEach Activity: Loop through each category
      └─ Copy Activities:
         ├─ Download Reviews: bronze/reviews/{category}/
         └─ Download Metadata: bronze/metadata/{category}/

┌─────────────────────────────────────────────────────────────┐
│ STEP 2: Bronze → Silver Transformation (Databricks)         │
└─────────────────────────────────────────────────────────────┘
   │
   ├─ Notebook 1: 01_Bronze_to_Silver_Reviews.py
   │  ├─ Read compressed JSON from Bronze
   │  ├─ Apply explicit schema
   │  ├─ Remove duplicates (user_id + parent_asin + timestamp)
   │  ├─ Filter nulls in critical fields
   │  ├─ Convert Unix timestamp to datetime
   │  ├─ Derive features (review_length, has_images)
   │  ├─ Add audit columns (processing_timestamp)
   │  └─ Write to Silver as Parquet (partitioned by category)
   │
   └─ Notebook 2: 02_Bronze_to_Silver_Metadata.py
      ├─ Read compressed JSON from Bronze
      ├─ Apply explicit schema
      ├─ Flatten nested arrays (features, description)
      ├─ Clean prices (nulls → 0.0)
      ├─ Extract details fields (brand, color, size)
      ├─ Convert details map to JSON string
      └─ Write to Silver as Parquet (partitioned by category)

┌─────────────────────────────────────────────────────────────┐
│ STEP 3: Silver → Gold Dimensional Model (Databricks)        │
└─────────────────────────────────────────────────────────────┘
   │
   └─ Notebook 3: 03_Silver_to_Gold_Dimensional.py
      │
      ├─ Build dim_date
      │  └─ Extract unique dates → Generate date attributes → Write Delta
      │
      ├─ Build dim_categories
      │  └─ Extract unique categories → Assign surrogate keys → Write Delta
      │
      ├─ Build dim_products
      │  └─ Use metadata → Assign surrogate keys → Write Delta
      │
      ├─ Build dim_users
      │  └─ Aggregate user stats → Assign surrogate keys → Write Delta
      │
      └─ Build fact_reviews
         ├─ Join reviews with all dimensions (get surrogate keys)
         ├─ Select measures and foreign keys
         ├─ Validate data quality (check null FKs)
         └─ Write Delta (partitioned by category_key)

┌─────────────────────────────────────────────────────────────┐
│ STEP 4: Data Ready for Analytics                            │
└─────────────────────────────────────────────────────────────┘
   │
   └─ Connect BI Tools:
      ├─ Power BI
      ├─ Tableau
      └─ SQL queries via Databricks SQL
```

---

## 🚀 Setup Instructions

### Prerequisites

- **Azure Account** with active subscription ($200 free credit for students)
- **Azure CLI** installed
- **Git** installed
- Basic knowledge of Python, SQL, and cloud computing

### 1️⃣ Azure Resource Setup
```bash
# Login to Azure
az login

# Create Resource Group
az group create \
  --name rg-amazon-etl-pipeline \
  --location eastus

# Create Storage Account (Data Lake Gen2)
az storage account create \
  --name adlsamazonetl \
  --resource-group rg-amazon-etl-pipeline \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --hierarchical-namespace true

# Create Containers
az storage container create --name bronze --account-name adlsamazonetl
az storage container create --name silver --account-name adlsamazonetl
az storage container create --name gold --account-name adlsamazonetl

# Create Azure Data Factory
az datafactory create \
  --name adf-amazon-etl \
  --resource-group rg-amazon-etl-pipeline \
  --location eastus

# Create Databricks Workspace
az databricks workspace create \
  --name dbw-amazon-etl \
  --resource-group rg-amazon-etl-pipeline \
  --location eastus \
  --sku trial
```

### 2️⃣ Clone Repository
```bash
git clone https://github.com/yourusername/amazon-etl-pipeline.git
cd amazon-etl-pipeline
```

### 3️⃣ Configure Databricks

1. **Import Notebooks:**
   - Open Databricks workspace
   - Navigate to Workspace → Users → [your-email]
   - Import notebooks from `databricks-notebooks/` folder

2. **Create Cluster:**
   - Compute → Create Compute
   - **Cluster Mode:** Single Node (cost optimization)
   - **Runtime:** 13.3 LTS (includes Delta Lake)
   - **Node Type:** Standard_DS3_v2
   - **Auto-termination:** 30 minutes

3. **Set Storage Credentials:**
   - Get storage account access key from Azure Portal
   - Update `storage_account_key` in each notebook

### 4️⃣ Configure Azure Data Factory

1. **Create Linked Services:**
   - **HTTP Linked Service:** Connect to Amazon dataset source
   - **ADLS Gen2 Linked Service:** Connect to your storage account
   - **Databricks Linked Service:** Connect to Databricks workspace

2. **Import Pipeline:**
   - Import pipeline definition from `adf-pipelines/pipeline-config.json`

3. **Set Pipeline Variables:**
   - `all_categories`: `["Books", "Electronics"]`
   - Adjust based on which categories you want to process

### 5️⃣ Run the Pipeline
```bash
# Trigger ADF pipeline via Azure Portal
# OR via Azure CLI:
az datafactory pipeline create-run \
  --factory-name adf-amazon-etl \
  --name amazon-etl-pipeline \
  --resource-group rg-amazon-etl-pipeline
```

### 6️⃣ Verify Results
```python
# In Databricks notebook:
# Check Gold layer tables
display(spark.read.format("delta").load("abfss://gold@adlsamazonetl.dfs.core.windows.net/fact_reviews"))
display(spark.read.format("delta").load("abfss://gold@adlsamazonetl.dfs.core.windows.net/dim_products"))
```

---

## ✨ Key Features

### 1. **Parameterized Pipeline**
- Dynamic category selection via ADF variables
- Easy to scale from 2 categories to all 33
- No code changes required for different datasets

### 2. **Data Quality Framework**
- **Duplicate Detection:** Composite key deduplication
- **Null Handling:** Filtered critical fields, defaulted optional fields
- **Schema Validation:** Explicit schemas enforce data types
- **Audit Trail:** Processing timestamps for lineage tracking

### 3. **Performance Optimizations**
- **Partitioning:** Data partitioned by category for query pruning
- **Columnar Storage:** Parquet/Delta Lake for fast analytics
- **Predicate Pushdown:** Filters applied at storage layer
- **Compression:** 90% size reduction (JSON → Parquet)

### 4. **Cost Optimization**
- **Single Node Clusters:** 50% cheaper than multi-node for development
- **Auto-termination:** Clusters shut down after 30 minutes idle
- **LRS Storage:** Locally redundant storage (cheapest option)
- **Sample Data:** Processing 2 categories instead of all 33

### 5. **Production-Ready Patterns**
- **Medallion Architecture:** Industry-standard data lake organization
- **Idempotent Transformations:** Can re-run without side effects
- **Error Handling:** Try-catch blocks and validation checks
- **Monitoring:** Built-in ADF monitoring dashboard

---

## 🧩 Challenges & Solutions

### Challenge 1: Nested JSON Structures
**Problem:** Raw data has arrays and dictionaries 3-4 levels deep

**Solution:**
```python
# Flatten arrays to delimited strings
.withColumn("features_text", F.concat_ws(" | ", F.col("features")))

# Convert maps to JSON strings
.withColumn("product_details_json", F.to_json(F.col("details")))
```

### Challenge 2: Duplicate Column Names
**Problem:** `details` dictionary had keys like "Brand", "brand", "BRAND"

**Solution:**
- Define explicit schema to prevent auto-expansion
- Store full `details` as JSON string
- Extract only common fields with known keys

### Challenge 3: Schema Inference Performance
**Problem:** Spark spent 3-5 minutes inferring schema on large JSON files

**Solution:**
```python
# Define schema upfront (3-5x faster)
df = spark.read.schema(reviews_schema).json(path, multiLine=False)
```

### Challenge 4: Memory Issues with Large Datasets
**Problem:** Initial runs with all 571M reviews caused out-of-memory errors

**Solution:**
- Start with 2 categories for development (1.5M + 3M reviews)
- Use partitioning to process data incrementally
- Scale to larger datasets after optimization

### Challenge 5: ADF Parameter Type Mismatch
**Problem:** ADF passed array as object, Databricks expected string

**Solution:**
```python
# In ADF: Use @string(variables('all_categories'))
# In Databricks: Parse JSON string
categories = json.loads(categories_param)
```

---

## 🔮 Future Enhancements

### Phase 2: Scale to Production
- [ ] Process all 33 categories (571M reviews)
- [ ] Implement incremental loads (only new data)
- [ ] Add slowly changing dimensions (SCD Type 2)
- [ ] Set up automated testing (Great Expectations)

### Phase 3: Advanced Analytics
- [ ] Sentiment analysis using Azure Cognitive Services
- [ ] Product recommendation engine with collaborative filtering
- [ ] Time-series forecasting for review trends
- [ ] Natural language processing for review text

### Phase 4: Visualization & Reporting
- [ ] Power BI dashboard with key metrics
- [ ] Real-time monitoring with Azure Monitor
- [ ] Alerting for data quality issues
- [ ] Executive summary reports

### Phase 5: Infrastructure as Code
- [ ] Terraform scripts for Azure resource provisioning
- [ ] CI/CD pipeline with GitHub Actions
- [ ] Automated deployment to dev/staging/prod environments
- [ ] Infrastructure cost tracking and optimization

---

## 📚 Learning Outcomes

Through this project, I gained hands-on experience with:

### Technical Skills
- ✅ **Cloud Data Engineering:** Designed and implemented end-to-end Azure data pipeline
- ✅ **Big Data Processing:** Used Apache Spark (PySpark) to process millions of records
- ✅ **Data Modeling:** Built Star Schema with fact and dimension tables
- ✅ **ETL Development:** Created parameterized, reusable transformation logic
- ✅ **Data Quality:** Implemented validation, deduplication, and error handling

### Cloud Technologies
- ✅ **Azure Data Factory:** Pipeline orchestration and scheduling
- ✅ **Azure Data Lake Gen2:** Scalable data lake storage
- ✅ **Azure Databricks:** Managed Spark clusters and notebooks
- ✅ **Delta Lake:** ACID-compliant lakehouse architecture

### Best Practices
- ✅ **Medallion Architecture:** Organized data into Bronze/Silver/Gold layers
- ✅ **Dimensional Modeling:** Implemented Kimball methodology
- ✅ **Cost Optimization:** Minimized Azure spending with smart resource choices
- ✅ **Documentation:** Created comprehensive README and code comments

### Soft Skills
- ✅ **Problem-Solving:** Debugged complex data quality and performance issues
- ✅ **Project Management:** Broke down large project into manageable phases
- ✅ **Communication:** Documented technical decisions and trade-offs
- ✅ **Presentation:** Prepared for technical interviews and capstone defense

---

## 📊 Project Metrics

| Metric | Value |
|--------|-------|
| **Total Lines of Code** | ~800 |
| **Data Processed** | 4.5M reviews + 2.5M products |
| **Storage Used** | ~15GB (Bronze + Silver + Gold) |
| **Pipeline Runtime** | ~25 minutes end-to-end |
| **Cost per Run** | ~$0.50 (Databricks compute) |
| **Query Performance** | < 5 seconds for aggregations |
| **Data Quality** | 98% completeness after cleaning |

---

## 📖 References & Resources

### Documentation
- [Amazon Reviews 2023 Dataset](https://amazon-reviews-2023.github.io/)
- [Azure Data Factory Documentation](https://docs.microsoft.com/azure/data-factory/)
- [Azure Databricks Documentation](https://docs.microsoft.com/azure/databricks/)
- [Delta Lake Documentation](https://docs.delta.io/)
- [PySpark API Reference](https://spark.apache.org/docs/latest/api/python/)

### Learning Resources
- [Medallion Architecture](https://www.databricks.com/glossary/medallion-architecture)
- [Kimball Dimensional Modeling](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/)
- [Azure Data Engineering Best Practices](https://docs.microsoft.com/azure/architecture/data-guide/)

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

### How to Contribute
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Your Name**
- GitHub: [@yourusername](https://github.com/yourusername)
- LinkedIn: [Your Name](https://linkedin.com/in/yourprofile)
- Email: your.email@example.com

---

## 🙏 Acknowledgments

- **Amazon** for providing the open-source dataset
- **Microsoft Azure** for $200 in free credits for students
- **Databricks Community Edition** for free Spark clusters
- **My Mentor** for guidance throughout the project
- **Open Source Community** for PySpark and Delta Lake

---

## 📈 Project Status

✅ **Phase 1 Complete:** Core ETL pipeline with 2 categories  
🚧 **Phase 2 In Progress:** Scaling to all 33 categories  
📋 **Phase 3 Planned:** Advanced analytics and ML integration

---

## 💬 Questions or Feedback?

If you have questions or feedback about this project, please:
- Open an [Issue](https://github.com/yourusername/amazon-etl-pipeline/issues)
- Start a [Discussion](https://github.com/yourusername/amazon-etl-pipeline/discussions)
- Reach out via [email](mailto:your.email@example.com)

---

<div align="center">

**⭐ If you found this project helpful, please give it a star! ⭐**

Made with ❤️ and ☕ by [Your Name]

</div>
