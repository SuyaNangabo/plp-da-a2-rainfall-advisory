# plp-da-a2-rainfall-advisory

Rainfall analysis and planting advisory for five Kenyan counties, built with Python and NumPy.

## Project Overview

A farmer-support NGO working in Nairobi, Nakuru, Kisumu, Mombasa and Machakos sends farmers SMS advice on when to plant. The NGO has monthly rainfall readings from local weather stations, but the Machakos station data was typed in by hand and contains invalid entries.

This project:

1. Cleans the Machakos readings and records every value that had to be imputed
2. Combines all five counties into a single 5 × 12 NumPy array (counties × months)
3. Uses vectorised NumPy operations to measure rainfall patterns per county
4. Identifies the best two-month planting window for each county
5. Exports a summary table to CSV and turns the findings into a short advisory the NGO can use

## Business Questions

- How much rain does each county receive in a year?
- Which months are wettest on average across the five counties?
- What is the single wettest month in each county?
- How many dry months (below 50 mm) does each county have?
- Which two consecutive months give the most rainfall, making them the best planting window?

## Dataset

Monthly rainfall in millimetres, January to December, for one year.

| County | Source format | Notes |
|---|---|---|
| Nairobi | Clean integer list | No issues |
| Nakuru | Clean integer list | No issues |
| Kisumu | Clean integer list | No issues |
| Mombasa | Clean integer list | No issues |
| Machakos | Raw text strings | Contains `"N/A"`, an empty string and a value with extra spaces (`" 20 "`) |

## Methodology

### 1. Data Cleaning

The function `clean_rainfall()` tries to convert each Machakos value with `int()` inside a `try` block.

- A value that converts successfully is kept. `int()` strips surrounding whitespace, so `" 20 "` becomes `20`.
- A value that cannot be converted raises a `ValueError`, which is caught specifically so that other unexpected errors are not hidden.
- Invalid values are replaced with the rounded mean of the valid Machakos readings, and the imputed months are printed.

| Month | Raw value | Cleaned value | Action |
|---|---|---|---|
| April | `"N/A"` | 68 | Imputed |
| June | `" 20 "` | 20 | Whitespace stripped |
| September | `""` | 68 | Imputed |

The valid readings sum to 675 mm across 10 months, so the mean is 67.5 mm, which rounds to 68 mm.

### 2. Building the Array

The four clean rows and the cleaned Machakos row are combined into one array `R` with shape `(5, 12)` and dtype `int64`. Rows are counties and columns are months.

### 3. Analysis with NumPy

All calculations work on the whole array at once. Python loops are used only to print results.

| Metric | NumPy technique |
|---|---|
| Annual total per county | `R.sum(axis=1)` |
| Average per month across counties | `R.mean(axis=0)` |
| Wettest month per county | `np.argmax(R, axis=1)` used to index the month names array |
| Dry months per county | Boolean mask `R < 50` summed along `axis=1` |
| Planting window | Shifted arrays `R[:, :-1] + R[:, 1:]` give a 5 × 11 array of two-month totals, then `np.argmax(axis=1)` |

### 4. Output

`rainfall_summary.csv` is written with Python's built-in `csv` module and read back in the notebook to check that it has five data rows.

## Results

| County | Annual total (mm) | Wettest month | Dry months | Planting window | Window total (mm) |
|---|---|---|---|---|---|
| Nairobi | 1045 | April | 4 | April-May | 395 |
| Nakuru | 916 | April | 3 | April-May | 265 |
| Kisumu | 1371 | April | 0 | April-May | 420 |
| Mombasa | 1075 | May | 2 | April-May | 390 |
| Machakos | 811 | November | 6 | November-December | 278 |

### Key Findings

- **Kisumu** is the wettest county at 1371 mm and is the only county with no dry months.
- **Machakos** is the driest county at 811 mm and has the most dry months (six).
- Four of the five counties share the same planting window, April to May, during the long rains.
- **Machakos** is the exception. Its strongest period is the November to December short rains.
- Across all five counties, May has the highest average rainfall (171.2 mm), followed by April (164.4 mm). January has the lowest (47.0 mm).

## Planting Advisory

- Farmers in Nairobi, Nakuru, Kisumu and Mombasa should prepare land in March and plant at the start of the long rains so crops benefit from the April to May peak.
- Farmers in Machakos, the driest county with six dry months, should plant drought tolerant or early maturing crops with the short rains in late October to use the November to December peak.
- Machakos April and September readings were missing and were filled with the station average of 68 mm, so Machakos advice should be confirmed against station records before it is sent.

## Data Quality and Limitations

- **Imputation bias:** April falls in the long rains and September in the dry season, so filling both with an annual average probably makes April too low and September too high. The true Machakos April to May total is likely higher than 208 mm, which could change its planting window.
- **Single year:** each county has one year of readings, so the results describe that year and not a long-term climate pattern.
- **Fixed threshold:** a dry month is defined as below 50 mm for every county, even though crop water needs vary.
- **Ties:** `np.argmax` returns the first month if two values are equal. No ties affect the current results.

## Repository Structure

```
plp-da-a2-rainfall-advisory/
├── README.md                  Project documentation
├── rainfall_analysis.ipynb    Cleaning, analysis, advisory and data quality notes
├── rainfall_summary.csv       Summary table, one row per county
└── requirements.txt           Python dependencies
```

### rainfall_summary.csv Columns

| Column | Description |
|---|---|
| county | County name |
| annual_total_mm | Total rainfall for the year in millimetres |
| wettest_month | Month with the highest rainfall |
| dry_months | Number of months with less than 50 mm |
| planting_window | Two consecutive months with the highest combined rainfall |

## How to Run

### Option 1: Google Colab

1. Go to [Google Colab](https://colab.research.google.com) and choose **File > Upload notebook**.
2. Upload `rainfall_analysis.ipynb`. NumPy is already installed in Colab.
3. Choose **Runtime > Restart session and run all**.
4. Open the folder icon on the left to find and download `rainfall_summary.csv`.

### Option 2: Local Jupyter

```
git clone https://github.com/<SuyaNangabo>/plp-da-a2-rainfall-advisory.git
cd plp-da-a2-rainfall-advisory
pip install -r requirements.txt
pip install notebook
jupyter notebook rainfall_analysis.ipynb
```

In Jupyter, choose **Kernel > Restart & Run All**. The notebook writes `rainfall_summary.csv` to the project folder.

## Requirements

- Python 3.8 or later
- numpy
- csv (part of the Python standard library, no installation needed)

## Tools and Skills Demonstrated

- Data cleaning with specific exception handling (`try/except ValueError`)
- Documented mean imputation
- NumPy array creation, axis operations, boolean masking, `argmax` and fancy indexing
- Shifted-array technique for rolling two-month sums
- CSV export with the `csv` module
- Turning analysis into practical advice for a non-technical audience

## Possible Next Steps

- Replace the imputed Machakos values with the real station records or with long-term averages for April and September
- Add several years of data to measure how much rainfall varies from year to year
- Use crop-specific rainfall thresholds in place of a single 50 mm cutoff
- Add charts of monthly rainfall per county with Matplotlib

## Author

Suya Nangabo

PLP Data Analytics, Assignment 2
