# Automotive Workshop Network — ETL & Exploratory Data Analysis

![Python](https://img.shields.io/badge/Python-3.13-3776AB?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-ETL-150458?logo=pandas&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualization-11557C)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![Context](https://img.shields.io/badge/Context-Timed%20exam%2C%20closed%20book-critical)

An ETL (Extract, Transform, Load) and exploratory analysis of 50,000 service orders from a fictional network of car repair shops. The raw Excel file is deliberately "dirty" — empty rows, inconsistent spellings, prices stored as text, invalid dates, duplicates and negative quantities — and the notebook cleans it with **pandas** and answers business questions with **Matplotlib** charts.

> [!IMPORTANT]
> **This was a timed, closed-book exam.** The notebook was written as the practical exam for the course *Desenvolvimento para Ciência de Dados* (Development for Data Science) in my undergraduate program in Data Science and Machine Learning. I had **a little over one hour** to complete it, **with no access to documentation, the internet, notes or AI assistants**. Every pandas and Matplotlib call was written from memory.
>
> The notebook is published **exactly as submitted**, without later edits, so it reflects what I could do under those conditions — including the shortcuts and rough edges a time limit produces. The [Retrospective](#retrospective-what-i-would-change-with-more-time) section lists what I would do differently with more time.

The task statements (in Portuguese, kept as markdown cells in the notebook) were written by the instructor; the code and the conclusions are mine. The dataset is entirely synthetic and was generated for the exam — it does not describe any real company, customer or transaction.

---

## Contents

- [The scenario](#the-scenario)
- [The dataset](#the-dataset)
- [ETL pipeline](#etl-pipeline)
- [Findings](#findings)
- [Retrospective: what I would change with more time](#retrospective-what-i-would-change-with-more-time)
- [Repository structure](#repository-structure)
- [How to run](#how-to-run)

---

## The scenario

A chain of five workshops in the Federal District of Brazil (Brasília and surrounding administrative regions) records every service order in a spreadsheet filled in by hand. The exam asked for the data to be loaded and inspected, cleaned through a ten-step ETL, and then used to answer four fixed questions — which service categories and services are most common, where most services happen, and how customers pay — plus two additional questions of the student's own choosing.

## The dataset

`Dataset_Prova_Oficinas_50mil.xlsx` has 50,000 rows and 11 columns.

| Column | Meaning | Example |
|---|---|---|
| `id_ordem` | Service order ID | `OS000002` |
| `data_servico` | Service date (text, day first) | `05/09/2026` |
| `cliente` | Customer (anonymized) | `Cliente 000002` |
| `cidade` | City / administrative region | `Ceilândia` |
| `oficina` | Workshop branch (5) | `Auto Center Central` |
| `categoria` | Service category (6) | `Manutenção Preventiva` (preventive maintenance) |
| `servico` | Specific service (24) | `Troca de Óleo` (oil change) |
| `tipo_veiculo` | Vehicle type | `Hatch`, `Sedan`, `SUV`, `Picape` (pickup) |
| `valor_servico` | Price in BRL, stored as text | `R$ 1.118,50`, `144,84` |
| `quantidade` | Quantity | `2` |
| `forma_pagamento` | Payment method | `PIX`, `Cartão de Crédito` (credit card) |

The quality problems planted in the file were 300 completely empty rows; 40 spelling variants of 8 city names, mixing case (`BRASÍLIA`, `brasília`) and stray whitespace; prices as Brazilian-formatted text, sometimes with the `R$` symbol and sometimes without; 800 dates that cannot be parsed; 645 exact duplicate rows; and 581 negative quantities (after deduplication).

## ETL pipeline

| Step | What the notebook does | pandas |
|---|---|---|
| Extract | Reads the sheet and shows the first rows, the shape and the column names | `pd.read_excel`, `head`, `shape`, `columns` |
| Empty rows | Drops rows in which every field is null (300) | `dropna(how="all")` |
| Cities | Strips whitespace and normalizes case, collapsing 40 variants into 8 cities | `str.strip`, `str.lower().str.title()` |
| Prices | Removes `R$` and the thousands separator, swaps the decimal comma for a dot | `str.replace(..., regex=False)` |
| Dates | Parses day-first text; invalid dates become null (800) | `pd.to_datetime(dayfirst=True, format="mixed", errors="coerce")` |
| Duplicates | Counts and removes exact duplicates (645) | `duplicated().sum()`, `drop_duplicates()` |
| Quantities | Treats negatives as sign-entry errors and keeps their absolute value | string replace, cast to `float` |
| Load | Clean DataFrame of **49,055 rows** used for the charts | — |

## Findings

Every chart is a Matplotlib bar chart rendered inside the notebook (GitHub displays them when you open the `.ipynb`). The counts below come from re-running the notebook's own cleaning steps on the dataset.

| Question | Answer | Orders |
|---|---|---|
| Most frequent service category | Preventive maintenance | 17,634 |
| Most performed service | Oil change | 7,989 |
| City with the most services | Brasília | 16,615 |
| Most used payment method | PIX | 18,651 |
| Most serviced vehicle type *(my question)* | Hatchback | 16,733 |
| Busiest workshop *(my question)* | Auto Center Central | 17,357 |

Preventive maintenance dominates the categories, well ahead of mechanical (12,215) and electrical (6,975) work, and the service ranking tells the same story: oil change leads, followed by general inspection (5,238) and brake pad replacement (4,297). Brasília concentrates about a third of all orders, followed by Taguatinga (9,355) and Ceilândia (7,355) — a ranking that only became trustworthy after the city cleaning, since before it Brasília's orders were spread across several spellings. PIX, Brazil's instant payment system, is the preferred payment method, ahead of credit (14,281) and debit cards (8,309).

For the two open "challenge" charts I chose vehicle type and workshop. Hatchbacks and sedans (14,275) are the most frequently serviced vehicles, and Auto Center Central handles far more orders than any other branch, with Auto Center Norte a distant second (11,622).

## Retrospective: what I would change with more time

Writing this README gave me the chance to review the notebook calmly, without the clock and with documentation at hand. The points below are things I noticed afterwards; the notebook itself was left untouched so the exam stays an honest record.

**Finish the numeric conversion of prices.** The task asked for `valor_servico` to be converted to a numeric type, and under time pressure I stopped after cleaning the strings, so the column is still text. One extra line — `pd.to_numeric(df["valor_servico"], errors="coerce")` — would turn it into `float` and open the door to revenue analysis, such as total and average ticket per workshop or category.

**Make the cleaning cells safe to re-run.** The saved output of the price cell shows values like `14484` where `144.84` was expected. The code is correct on a fresh run, but running that cell a second time removes the decimal point that the first run had just inserted, because it strips every `.`. Cleaning steps should be idempotent — for instance, by always starting from the raw column or wrapping the ETL in a function applied once — and a notebook should be checked with *Restart & Run All* before it is saved.

**Keep dates as `datetime64`.** Appending `.dt.date` turns the column into plain Python objects; keeping the pandas datetime type would allow monthly volumes or seasonality to be computed with `resample` or `dt.to_period`.

**Handle negative quantities numerically and document the choice.** Removing the `-` character via string conversion works, but `df["quantidade"].abs()` is simpler and safer. It is also worth stating explicitly that taking the absolute value assumes a typing error; flagging or excluding those rows would be an equally defensible decision.

**Separate Load from analysis and polish the charts.** A reusable pipeline would export the clean data to CSV or Parquet, replace the hard-coded Windows path with a relative one, and use horizontal bars with value labels for long names such as the 24 services.

## Repository structure

```
.
├── Estudo de Caso - Desenvolvimento para Ciencia de Dados.ipynb   # Notebook (Portuguese), as submitted
├── Dataset_Prova_Oficinas_50mil.xlsx                               # Synthetic dataset
└── README.md
```

## How to run

Clone the repository and install the dependencies:

```bash
git clone https://github.com/RafaelGuimaraens/First-ETL-process.git
cd First-ETL-process
pip install pandas matplotlib openpyxl jupyter
jupyter notebook
```

The notebook reads the dataset from the absolute path used on the exam machine (`C:\Users\Public\...`). Before running it, change the first code cell to point to the file in the repository:

```python
df = pd.read_excel("Dataset_Prova_Oficinas_50mil.xlsx")
```

The notebook was written with Python 3.13 and pandas 3.x.

---

*Practical exam for "Desenvolvimento para Ciência de Dados" — written in about one hour, closed book. All data is synthetic.*
