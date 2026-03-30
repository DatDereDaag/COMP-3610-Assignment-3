# COMP 3610: Big Data Analytics — Assignment 3

## LLM-Powered Applications & Distributed Computing

---

## Overview

This project builds a complete intelligent data analytics system that combines:

- **Apache Spark** for distributed processing of NYC Yellow Taxi trip data
- **RAG (Retrieval-Augmented Generation)** pipeline over a curated corpus of NYC transportation policy documents
- **An integrated query router** that classifies natural language questions and routes them to the appropriate backend (structured data, documents, or both)

---

## Repository Structure

```
├── assignment3.ipynb               # Main Jupyter notebook (all code, outputs, analysis)
├── docs/                           # Curated PDF document corpus
│   ├── 2020-tlc-factbook.pdf
│   ├── annual_report_2024.pdf
│   ├── annual_report_2025.pdf
│   ├── commuter_van_safety_study_2024.pdf
│   ├── driver_expense_report.pdf
│   ├── electrification_in_motion_report_2024.pdf
│   ├── license-pause-report-2025-02.pdf
│   ├── mobility-report-singlepage-2019.pdf
│   └── nyc-rules-of-the-city.pdf
├── requirements.txt                # Python dependencies
├── README.md                       # This file
└── .gitignore                      # Excludes data files, chroma_db/, __pycache__/
```

> **Note:** The NYC Taxi Parquet data file and ChromaDB vector store are **not** committed to this repository. The notebook downloads the data automatically and rebuilds the vector store on first run.

---

## Document Corpus

| File                                        | Description                                                                                        |
| ------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `2020-tlc-factbook.pdf`                     | NYC TLC statistical factbook covering trip volumes, vehicle counts, and industry trends up to 2020 |
| `annual_report_2024.pdf`                    | NYC TLC 2024 annual report — licensing, enforcement, accessibility, and industry metrics           |
| `annual_report_2025.pdf`                    | NYC TLC 2025 annual report — latest regulatory updates and for-hire transportation data            |
| `commuter_van_safety_study_2024.pdf`        | TLC safety standards and regulations for commuter vans                                             |
| `driver_expense_report.pdf`                 | Analysis of operating costs faced by TLC-licensed drivers                                          |
| `electrification_in_motion_report_2024.pdf` | TLC report on EV fleet adoption, targets, and sustainability goals                                 |
| `license-pause-report-2025-02.pdf`          | Report on the TLC licensing pause policy and its impact on driver supply                           |
| `mobility-report-singlepage-2019.pdf`       | NYC citywide transit patterns, taxi/FHV usage, and urban mobility trends                           |
| `nyc-rules-of-the-city.pdf`                 | Official NYC rules governing TLC-licensed drivers and vehicles                                     |

---

## Setup Instructions

### Prerequisites

- Python 3.10+
- Java 8 or Java 11 (required for PySpark)
- Jupyter Notebook or JupyterLab

Verify Java is installed:

```bash
java -version
```

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd <repo-folder>
```

### 2. Create a Virtual Environment (Recommended)

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS/Linux
source venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Set Up Environment Variables

Create a `.env` file in the root directory with your LLM API credentials:

```
LLM_API_KEY=your_api_key_here
LLM_BASE_URL=your_llm_server_base_url_here
```

The notebook loads these automatically via `python-dotenv`.

### 5. Run the Notebook

Run all cells from top to bottom. The notebook will:

1. Automatically download the NYC Yellow Taxi January 2024 Parquet file (~50 MB)
2. Build the Spark session and perform all data processing
3. Load the PDF corpus from `docs/` and build the ChromaDB vector store
4. Run the full RAG pipeline and integrated query system

---

## How to Run

The notebook is self-contained and designed to be run **top to bottom**. Key things to be aware of:

- **Data download** happens automatically in Task 1.1 — no manual download needed
- **ChromaDB** is rebuilt each run (the existing `chroma_db/` directory is cleared first to avoid duplication)
- **LLM calls** require a valid API key and base URL in `.env`
- The cleaned Parquet data is written to `data/cleaned/` partitioned by `pickup_hour`

---

## Project Components

### Part 1: Distributed Data Processing with Spark

- SparkSession configured with adaptive query execution and 4GB driver memory
- Data cleaning: null removal, invalid trip filtering, feature engineering (duration, speed, pickup hour, tip percentage)
- 5 Spark SQL analytical queries including window functions
- Performance optimisation: caching, partitioned Parquet writes, and execution plan analysis

### Part 2: RAG Pipeline over Transportation Documents

- 9 PDF documents loaded, extracted, and quality-checked
- Text chunked with `RecursiveCharacterTextSplitter` (chunk size 1000, overlap 200)
- Embeddings generated with `sentence-transformers/all-MiniLM-L6-v2` and stored in ChromaDB
- Chunk size experiment comparing sizes 200, 1000, and 2000
- RAG pipeline tested on 5 diverse questions with source citations
- 10-question evaluation set with retrieval and answer quality metrics

### Part 3: Integrated Analytics Application

- LLM-powered query router classifying questions as DATA / DOCUMENT / HYBRID
- NL-to-SQL handler with retry logic on failure
- End-to-end demo processing 6 queries across all three categories
- HYBRID queries synthesise structured data and document findings into a unified response

---

## Tools & Technologies

| Category              | Tool                                                     |
| --------------------- | -------------------------------------------------------- |
| Distributed Computing | PySpark 3.3+ (local mode)                                |
| Document Processing   | LangChain `PyPDFDirectoryLoader`, pypdf                  |
| Text Splitting        | LangChain `RecursiveCharacterTextSplitter`               |
| Embeddings            | `sentence-transformers/all-MiniLM-L6-v2` via HuggingFace |
| Vector Database       | ChromaDB (persisted to disk)                             |
| LLM Access            | OpenAI-compatible API (`llama3.3-70b-instruct`)          |
| Visualisation         | Matplotlib, Seaborn                                      |
| Data Processing       | PySpark DataFrame API, Spark SQL                         |

---

## AI Tool Usage Disclosure

AI tools (Claude, ChatGPT) were used during the development of this assignment to assist with:

- Debugging PySpark errors and understanding execution plans
- Drafting and refining LLM prompt templates
- Checking logic in the RAG evaluation pipeline
- Creating readme and verifying requirements.txt

All submitted code and analysis reflects the author's own understanding. No AI-generated code was submitted without review and comprehension.

---

## Data Source

NYC Yellow Taxi Trip Records (January 2024) from the NYC Taxi and Limousine Commission:  
https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-01.parquet
