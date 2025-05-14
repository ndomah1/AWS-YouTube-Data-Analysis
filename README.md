# AWS YouTube Data Analysis
## Project Background

As a data engineer at a digital marketing consultancy, I was tasked with building an end-to-end data pipeline and dashboard to help clients understand YouTube trends in preparation for a major advertising campaign. The client, a multinational advertiser, wanted to optimize their YouTube spend by targeting high-performing categories and geographies.

I built a cloud-native, scalable pipeline using AWS services to ingest, clean, process, and analyze YouTube video data to support this. The output was an interactive dashboard that allowed stakeholders to explore which types of content gained the most traction, where, and why.

Insights and recommendations are provided on the following key areas:

- **Regional Viewership**
- **Content Categories**
- **Engagement Drivers**
- **Dashboard-Ready Reporting Layer**

> All transformations were orchestrated using AWS-native services like Lambda, Glue, S3, Athena, and QuickSight. Scripts and logic were managed using Python and Spark (via PySpark).

---

## Data Structure & Initial Checks

The dataset was sourced from Kaggle and included:

- **CSV files**: Trending video statistics by region (e.g., views, likes, comments).
- **JSON file**: Video metadata (category info, titles, tags).

Data was organized in Amazon S3 using Hive-style partitions, where each region had its own subfolder.
The project involved creating three distinct tables:

- **raw_statistics**: Raw CSV data by region, cataloged using AWS Glue.
- **cleaned_categories**: Normalized JSON metadata parsed by a Lambda function and stored in Parquet.
- **final_analytics**: Join of the two above, preprocessed and stored in a reporting-ready table.

The initial ingestion was automated with AWS CLI ([`s3_cli_command.sh`]()). JSON files were grouped together, and each CSV was placed into a partition-specific location.

---

# Executive Summary

### Overview of Findings

To support our client’s YouTube campaign, I built a fully automated data pipeline and interactive dashboard. From the cleansed dataset and Athena queries, I found that:

- **Entertainment, Music, and People & Blogs** were the most engaging content categories across all regions.
- **Great Britain had the highest viewership**, followed by Canada and the U.S.
- The dashboard allowed breakdowns of views and likes per category and region, helping guide the client's targeting strategy.

![Final Dashboard](../dashboard.jpg)

---

# Insights Deep Dive

### Regional Viewership

* **Great Britain recorded the most total views** in this dataset, followed by Canada and the U.S.
* Filtering by region in QuickSight revealed that content popularity varied: Canadian audiences leaned toward Music, while U.S. audiences favored Entertainment.
* Region-level partitioning in S3 made these queries efficient and scalable using AWS Athena.

---

### Content Categories

* **Music and Entertainment were the dominant video categories** in terms of both views and likes.
* **People & Blogs consistently ranked third**, with especially high traction in Canada.
* These patterns were identified by joining the `raw_statistics` with `cleaned_categories` via category ID.

---

### Engagement Drivers

* I interpreted `snippet_title` as the label for each video category (i.e., the cleaned title from the JSON metadata).
* Likes by snippet_title therefore reflect **engagement levels per content category**, not per video title length.
* This helped answer the question: “Which types of videos get the most engagement?”

---

### Dashboard-Ready Reporting Layer

To avoid complex joins during analysis, I built a final reporting layer using AWS Glue and PySpark:

- The script filtered only English-speaking regions (CA, GB, US) using `predicate_pushdown`.
- The cleaned data was mapped to appropriate types and written as **partitioned Parquet files** on S3.
- The final table (`final_analytics`) was registered in Glue for querying in Athena and visualization in QuickSight.

Key details:
- ETL logic was built in [`pyspark_code.py`]()
- JSON normalization was handled by [`lambda_function.py`]() using `awswrangler`
- All writing was done with partitioning by region and category

---

# Recommendations

For the client’s YouTube ad strategy, I recommended:

* **Prioritize Music and Entertainment categories**, which consistently performed across all regions.
* **Target region-specific interests**, such as People & Blogs in Canada or Entertainment in the U.S.
* **Deprioritize underperforming categories** like News or How-To unless niche targeting is intended.
* **Schedule weekly performance reviews** using the live QuickSight dashboard and Athena queries.
* **Scale this pipeline** to include streaming ingestion or expand to other platforms (e.g., TikTok or Instagram).

---

# Assumptions and Caveats

* Dataset is historical and reflects trends at the time of scraping; current YouTube behavior may differ.
* Region codes follow ISO standards (e.g., `gb` = Great Britain).
* UTF-8 encoding issues from multilingual content led to region filtering during ETL.
* The Lambda layer only processes new uploads; to reprocess all JSON, manual re-triggering or S3 batch ops are needed.
* Scripts were designed for batch processing; scaling to real-time would require tools like Kinesis or Kafka.
