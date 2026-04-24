# 🛒 Products Data Pipeline

> An end-to-end ETL pipeline that scrapes product data from the web, ingests from a REST API, cleans and merges both datasets, and loads the result into a MySQL database.

![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python&logoColor=white)
![Selenium](https://img.shields.io/badge/Selenium-Web%20Scraping-green?logo=selenium&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Processing-150458?logo=pandas&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-Database-4479A1?logo=mysql&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Pipeline Architecture](#-pipeline-architecture)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [Pipeline Steps](#-pipeline-steps)
- [Output](#-output)

---

## 🔍 Overview

This project implements a fully automated data pipeline that:

1. **Scrapes** book titles and prices from [Books to Scrape](https://books.toscrape.com/) using Selenium
2. **Ingests** product data from the [DummyJSON](https://dummyjson.com/products) REST API
3. **Cleans & merges** both datasets using Pandas (deduplication, type casting, null handling)
4. **Exports** the final dataset to a CSV file
5. **Loads** the cleaned data into a MySQL database

---

## 🏗 Pipeline Architecture

```
┌─────────────────────┐     ┌──────────────────────┐
│   Books to Scrape   │     │   DummyJSON REST API │
│  (Selenium Scraper) │     │   (requests library) │
└────────┬────────────┘     └──────────┬───────────┘
         │  df_scraped                 │  df_api
         └──────────────┬──────────────┘
                        ▼
             ┌─────────────────────┐
             │  Data Cleaning &    │
             │  Transformation     │
             │  (Pandas)           │
             └────────┬────────────┘
                      │  final_df
           ┌──────────┴──────────┐
           ▼                     ▼
  ┌─────────────────┐   ┌─────────────────┐
  │  cleaned_data   │   │  MySQL Database │
  │     .csv        │   │  (products tbl) │
  └─────────────────┘   └─────────────────┘
```

---

## 📁 Project Structure

```
products_pipeline/
├── products_pipeline.ipynb   # Main pipeline notebook
├── cleaned_data.csv          # Output CSV (generated after run)
└── README.md
```

---

## ✅ Prerequisites

- Python 3.8+
- Google Chrome browser
- ChromeDriver (matching your Chrome version)
- MySQL Server running locally

---

## 🚀 Installation

**1. Clone the repository**

```bash
git clone https://github.com/your-username/products-pipeline.git
cd products-pipeline
```

**2. Install dependencies**

```bash
pip install pandas requests selenium mysql-connector-python jupyter
```

**3. Set up MySQL**

Create the target database before running the pipeline:

```sql
CREATE DATABASE products_scrape_db;
```

---

## ⚙️ Configuration

All settings are defined at the top of the notebook under **Section 0 – Imports & Configuration**:

| Variable | Default | Description |
|---|---|---|
| `SCRAPE_URL` | `https://books.toscrape.com/` | Target URL for web scraping |
| `SCRAPE_LIMIT` | `10` | Max number of books to scrape |
| `API_URL` | `https://dummyjson.com/products` | REST API endpoint |
| `OUTPUT_CSV` | `cleaned_data.csv` | Output file path |
| `DB_CONFIG.host` | `localhost` | MySQL host |
| `DB_CONFIG.user` | `root` | MySQL username |
| `DB_CONFIG.password` | `••••••••` | MySQL password |
| `DB_CONFIG.database` | `products_scrape_db` | Target database name |

> ⚠️ **Security Note:** Avoid committing credentials to version control. Consider using environment variables or a `.env` file in production.

---

## 🧑‍💻 Usage

Launch the notebook and run all cells:

```bash
jupyter notebook products_pipeline.ipynb
```

Or run it non-interactively from the terminal:

```bash
jupyter nbconvert --to notebook --execute products_pipeline.ipynb
```

---

## 🔄 Pipeline Steps

### Step 1 — Web Scraping
Uses **Selenium** in headless mode to scrape book titles and prices from Books to Scrape. Returns a DataFrame with `title` and `price` columns.

### Step 2 — API Ingestion
Sends a GET request to the **DummyJSON** products endpoint and parses the JSON response into a DataFrame, selecting only `title` and `price`.

### Step 3 — Data Cleaning & Transformation
- Strips the `£` currency symbol from scraped prices and casts to `float`
- Ensures API prices are numeric
- Concatenates both DataFrames
- Strips whitespace from titles
- Drops duplicate titles
- Fills missing prices with `0`

### Step 4 — Export to CSV
Saves the final cleaned DataFrame to `cleaned_data.csv`.

### Step 5 — Load into MySQL
Creates the `products` table (if it does not exist) and bulk-inserts all rows using `executemany` for efficiency.

### Step 6 — Summary Report
Prints a summary of the pipeline run including record counts and price range statistics.

---

## 📊 Output

**CSV schema:**

| Column | Type | Description |
|---|---|---|
| `title` | string | Product or book title |
| `price` | float | Price in GBP (scraped) or USD (API) |

**MySQL table — `products`:**

```sql
CREATE TABLE products (
    id    INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    price FLOAT        NOT NULL
);
```

**Sample pipeline summary output:**

```
Pipeline Summary
========================================
  Books scraped    : 10
  API products     : 30
  After dedup      : 40
  CSV output       : cleaned_data.csv
  Price range      : £13.99 – £499.99
========================================
```
