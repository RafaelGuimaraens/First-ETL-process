# Automotive Workshop Network — ETL & Exploratory Data Analysis

![Python](https://img.shields.io/badge/Python-3.13-3776AB?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-ETL-150458?logo=pandas&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualization-11557C)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)

An end-to-end **ETL (Extract, Transform, Load) and exploratory analysis** of 50,000 service orders from a network of automotive repair shops. The project takes a deliberately "dirty" Excel dataset, cleans and standardizes it with **pandas**, and answers business questions through visualizations built with **Matplotlib**.

> [!NOTE]
> **Academic context.** This notebook was produced as a graded exam for the course *Desenvolvimento para Ciência de Dados* (Development for Data Science) in my undergraduate program in Data Science and Machine Learning. The task statements were provided by the instructor; the code and the analysis are my own.
>
> **Language.** The task descriptions, markdown cells, chart labels, and conclusions inside the notebook are written in **Portuguese**, as required by the exam. This README is in English and summarizes everything in the notebook.
>
> **Data.** The dataset is **entirely fictional**. It was synthetically generated for the exam and does not represent any real company, customer, or transaction.

---

## Table of Contents

- [Business Scenario](#business-scenario)
- [Dataset](#dataset)
- [ETL Pipeline](#etl-pipeline)
- [Analysis & Key Findings](#analysis--key-findings)
- [Repository Structure](#repository-structure)
- [How to Run](#how-to-run)
- [Possible Improvements](#possible-improvements)
- [Skills Demonstrated](#skills-demonstrated)

---

## Business Scenario

A chain of automotive workshops located in the Federal District of Brazil (Brasília and surrounding administrative regions) records every service order in a spreadsheet. The data was entered manually and contains typical real-world quality problems. The goal is to make the data reliable and then answer questions such as:

- Which service categories and individual services are most in demand?
- Where are most services performed?
- How do customers prefer to pay?
- Which vehicle types and which workshops account for the largest volume?

## Dataset

**File:** `Dataset_Prova_Oficinas_50mil.xlsx` — 50,000 rows × 11 columns (fictional data).

| Column (PT) | Meaning (EN) | Example |
|---|---|---|
| `id_ordem` | Service order ID | `OS000002` |
| `data_servico` | Service date | `05/09/2026` |
| `cliente` | Customer (anonymized) | `Cliente 000002` |
| `cidade` | City / administrative region | `Ceilândia` |
| `oficina` | Workshop branch | `Auto Center Central` |
| `categoria` | Service category | `Manutenção Preventiva` (Preventive Maintenance) |
| `servico` | Specific service | `Troca de Óleo` (Oil Change) |
| `tipo_veiculo` | Vehicle type | `Hatch`, `Sedan`, `SUV`, `Picape` (Pickup) |
| `valor_servico` | Service price (BRL) | `R$ 99,43` |
| `quantidade` | Quantity | `2` |
| `forma_pagamento` | Payment method | `PIX`, `Cartão de Crédito` (Credit Card) |

### Data quality issues found

| Issue | Example |
|---|---|
| Completely empty rows | ~300 rows with all fields missing |
| Inconsistent casing and stray whitespace in city names | `Brasília`, `brasília`, `BRASÍLIA`, `" Brasília"` |
| Prices stored as text in Brazilian currency format | `R$ 99,43`, `1.088,14` |
| Dates stored as text, including invalid values | `05/09/2026`, unparseable entries |
| Duplicate records | 645 exact duplicates |
| Negative quantities | 581 rows with values such as `-2` |

## ETL Pipeline

| Step | What was done | pandas technique |
|---|---|---|
| **Extract** | Loaded the Excel sheet and inspected shape, columns and first records | `pd.read_excel`, `head`, `shape`, `columns` |
| Remove empty rows | Dropped rows where every field is null | `dropna(how="all")` |
| Standardize cities | Trimmed whitespace and normalized to title case, collapsing 40 spelling variants into 8 cities | `str.strip`, `str.lower`, `str.title` |
| Clean prices | Removed the `R$` symbol and thousands separator, converted the decimal comma to a dot | `str.replace` |
| Parse dates | Converted day-first text dates to date objects, with invalid dates coerced to null (800 records) | `pd.to_datetime(dayfirst=True, format="mixed", errors="coerce")` |
| Deduplicate | Counted and removed exact duplicate rows | `duplicated().sum()`, `drop_duplicates()` |
| Fix quantities | Treated negative quantities as sign-entry errors and converted them to their absolute value | string replacement, then cast to `float` |
| **Load** | Final clean DataFrame (49,055 rows) used for the analysis | — |

## Analysis & Key Findings

The exam required four specific charts plus two additional charts of my own choosing (the "challenge" section). All charts are bar charts built with Matplotlib and are rendered directly inside the notebook; open it on GitHub or in Jupyter to see them. All counts below refer to the cleaned dataset.

| Question | Answer | Orders |
|---|---|---|
| Most frequent service category | Preventive Maintenance | ~17.6k |
| Most performed service | Oil Change | ~8.0k |
| City with the most services | Brasília | ~16.8k |
| Most used payment method | PIX | ~18.7k |
| Most serviced vehicle type | Hatchback | ~16.7k |
| Workshop with the most orders | Auto Center Central | ~17.4k |

### 1. Services by category
**Preventive Maintenance** (*Manutenção Preventiva*) is by far the most frequent category (~17.6k orders), followed by Mechanical (*Mecânica*) and Electrical (*Elétrica*).

### 2. Most performed services
**Oil Change** (*Troca de Óleo*) leads (~8.0k orders), ahead of General Inspection (*Revisão Geral*) and Brake Pad Replacement (*Troca de Pastilhas de Freio*). This is consistent with the dominance of preventive maintenance.

### 3. Services by city
**Brasília** concentrates the highest volume (~16.8k orders), followed by **Taguatinga** and Ceilândia. This chart only became meaningful after city names were standardized. Before cleaning, Brasília's orders were split across several spelling variants.

### 4. Payment methods
**PIX**, Brazil's instant payment system, is the most used payment method (~18.7k orders), followed by credit and debit cards.

### 5. Challenge: vehicle types
**Hatchbacks** and **sedans** are the most frequently serviced vehicle types.

### 6. Challenge: orders by workshop
**Auto Center Central** registers the most orders (~17.4k), significantly ahead of the other four branches.

## Repository Structure

```
.
├── Estudo_de_Caso_Oficinas.ipynb        # Main notebook (in Portuguese)
├── data/
│   └── Dataset_Prova_Oficinas_50mil.xlsx   # Fictional dataset
└── README.md
```

## How to Run

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/<repository-name>.git
   cd <repository-name>
   ```
2. Install the dependencies:
   ```bash
   pip install pandas matplotlib openpyxl jupyter
   ```
3. Open the notebook:
   ```bash
   jupyter notebook Estudo_de_Caso_Oficinas.ipynb
   ```
4. In the first code cell, update the file path so it points to the dataset inside the repository:
   ```python
   df = pd.read_excel("data/Dataset_Prova_Oficinas_50mil.xlsx")
   ```

The notebook was developed with Python 3.13 and pandas 3.x.

## Possible Improvements

The notebook is kept exactly as it was submitted for the exam. Looking back, these are the changes I would make in a production version:

- **Explicit numeric conversion of prices.** Add `pd.to_numeric(df["valor_servico"], errors="coerce")` after the string cleaning, so the column becomes `float` and supports revenue analysis (e.g., total revenue per workshop or category).
- **Keep dates as `datetime64`.** Calling `.dt.date` converts the column to generic Python objects; keeping the pandas datetime type enables time-series grouping (monthly volume, seasonality).
- **More deliberate treatment of negative quantities.** Taking the absolute value assumes a data-entry error; an alternative is to flag or remove those rows and document the decision.
- **Reusable pipeline.** Wrap the ETL steps in functions and export the clean data (e.g., to CSV or Parquet) to separate the *Load* stage from the analysis.
- **Chart refinements.** Use horizontal bars for long category names, add value labels to the bars, and export the figures as image files for easier sharing.

## Skills Demonstrated

Data cleaning and standardization · Handling of missing, duplicate and invalid data · Type conversion (text → numeric/date) · Working with Brazilian number and date formats · Exploratory data analysis · Data visualization · Translating business questions into analyses

---

*Academic project developed for the course "Desenvolvimento para Ciência de Dados". All data is fictional.*
