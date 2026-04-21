# Yelp Dataset Processor: JSON to CSV & Restaurant Classification
This project provides a high-performance pipeline for extracting the Yelp Academic Dataset, identifying restaurant-related businesses using a local Large Language Model (LLM), and converting the results into structured CSV files.

## Overview
Processing 150k+ records manually is slow. This pipeline uses a "Filter-First" strategy to categorize businesses in minutes instead of hours by combining keyword pre-filtering with batched inference via LM Studio.

## Prerequisites
- Python 3.10+
- Jupyter Notebook / Lab
- LM Studio (for local LLM inference)
  - Recommended Model: qwen/qwen3-1.7b (or any small, fast SLM)
- Dataset: Yelp Academic Dataset (Place yelp_dataset.tar in the data/ folder).

## Instructions
### 1. Data Extraction
The dataset is provided as a compressed .tar file. Use the initial cells in Yelp_Restaurant_Classifier.ipynb to extract the JSON files.

- **Input**: `data/yelp_dataset.tar`
- **Output**: `data/json/yelp_academic_dataset_business.json`, etc.

### 2. Optimized Restaurant Classification
Instead of classifying every row with an LLM, we use a multi-stage approach to ensure accuracy and eliminate false positives (like Target or Temples).

- Pre-Filter: Automatically flags businesses with "Restaurants" in categories and blacklists retail/religious keywords.
- Batching: Sends "ambiguous" cases (e.g., businesses tagged only with "Food") to **LM Studio** in batches of 20 to maximize throughput.
- Local Inference: Uses **Qwen3-1.7B** for fast, local classification without OpenAI API costs.

Run the classification in `Yelp_Restaurant_Classifier.ipynb` to generate `data/csv/restaurants_id.csv`.

### 3. JSON to CSV Conversion

Once the relevant restaurant IDs are identified, use `Yelp_JSON2CSV.ipynb` to transform the massive NDJSON files into manageable CSVs.

The script performs the following:

- Reads `business`, `review`, `user`, and `tip` JSON files.
- Filters records based on the `restaurants_id` generated in Step 2.
- Exports clean CSVs to `data/csv/`.

## Project Structure
    ├── data/
    │   ├── yelp_dataset.tar               # Original download
    │   ├── json/                          # Extracted .json files
    │   └── csv/                           # Final processed .csv files
    ├── Yelp_Restaurant_Classifier.ipynb   # Extraction & Smart Filtering
    ├── Yelp_JSON2CSV.ipynb                # Conversion logic
    └── README.md

## Key Optimizations
- **Keyword Blacklisting**: Prevents false positives by filtering out known non-restaurant categories before the LLM sees them.
- **Batch Requesting**: Reduces API overhead by 20x by grouping multiple classification queries into a single prompt.
- **Low-Latency SLM**: Utilizing Qwen3-1.7B on local GPU for near-instant classification of "gray-area" businesses.