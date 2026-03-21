🚀 Data Engineering Project: Medallion Architecture on Databricks

📌 Overview

This project implements a production-style data engineering pipeline using:
	•	Databricks (Lakehouse)
	•	Delta Lake
	•	AWS S3 (data source)
	•	Medallion Architecture (Bronze → Silver → Gold)

The system transforms raw data into clean, validated, and analytics-ready datasets with support for incremental processing and orchestration-ready pipelines.

⸻

🧠 Architecture

The project follows a domain-driven Medallion Architecture:

Domain → Bronze → Silver → Gold

🔷 Layers
	•	Bronze → Raw ingestion (no transformation)
	•	Silver → Cleaning, validation, standardization
	•	Gold → Business-ready data (facts & dimensions)

⸻

📁 Project Structure

Setup/
  ├── setup_catalog_schema
  ├── setup_dim_dates
  └── utilities

customer/
orders/
products/
product_pricing/

  ├── bronze/
  ├── silver/
  └── gold/

playground/


⸻

🔄 Data Flow

1. Bronze (Ingestion)
	•	Data ingested from AWS S3
	•	Stored in Delta format
	•	No transformations applied

2. Silver (Transformation)
	•	Data cleaning & normalization
	•	Schema enforcement
	•	Referential integrity checks
	•	Invalid records separated (quarantine pattern)

3. Gold (Modeling)
	•	Fact tables (orders)
	•	Dimension tables (customers, products)
	•	Aggregated marts (monthly metrics)

⸻

⚙️ Incremental Processing

Fact Tables
	•	Append or insert-only merge
	•	Grain: order_id
	•	Idempotent ingestion

Dimension Tables
	•	Built using latest records
	•	Uses ROW_NUMBER() for deduplication
	•	Supports upsert (MERGE)

⸻

🧩 Orchestration (Key Feature)
	•	Designed pipelines to support incremental ingestion using staging tables
	•	Ensures:
	•	Reliable processing
	•	Idempotency
	•	Controlled data movement

⸻

🛡️ Data Quality
	•	Referential integrity validation using left_anti joins
	•	Invalid records stored separately:
	•	invalid_customers
	•	invalid_products
	•	Enables debugging, monitoring, and auditing

⸻

🧱 Delta Lake Features
	•	ACID transactions
	•	Time travel
	•	Schema enforcement
	•	Change Data Feed (optional)

⸻

📊 Example Outputs

Fact Table
	•	fact_sb_orders
	•	Grain: order-level events

Aggregated Mart
	•	Monthly sales per customer & product

⸻

🛠️ Technologies Used
	•	Databricks
	•	PySpark
	•	Delta Lake
	•	AWS S3
	•	SQL

⸻

✅ Best Practices Implemented
	•	Domain-driven design
	•	Medallion architecture
	•	Incremental pipelines
	•	Idempotent processing
	•	Data validation & quarantine
	•	Separation of setup and pipelines

⸻

🚀 Future Improvements
	•	Add orchestration (Databricks Workflows / Airflow)
	•	Implement data quality framework
	•	Add monitoring & alerting
	•	Optimize performance (partitioning, Z-order)
	•	Introduce dbt for transformation layer

⸻

👤 Author

Saugat Siwakoti

⸻

🏁 Summary

This project demonstrates a modern data engineering system that:
	•	Scales across domains
	•	Ensures high data quality
	•	Supports incremental processing
	•	Produces analytics-ready datasets

It reflects real-world data engineering practices used in production systems.