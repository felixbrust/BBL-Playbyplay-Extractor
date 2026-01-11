# 🏀 German BBL Play-by-play Data Pipeline

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Data](https://img.shields.io/badge/Data-BasketballBundesliga-orange)
![Status](https://img.shields.io/badge/Status-Maintained-green)

## 📄 Overview
A robust data engineering pipeline designed to digitize, clean, and analyze historical Play-by-Play data from the German Basketball Bundesliga (BBL).

This repository contains the source code for transforming unstructured PDF game reports into structured, queryable CSV datasets using **LLMs (Gemini 3.0 Flash)**, as well as my master dataframe containing all events from the first 100 games of the 25/26 season ready for analysis. Included in the dataframe are columns denoting all players who are on the court during each event, enabling you to calculate on-off stats and more.

**🔗 Read the full technical breakdown on my Substack: _____**

## 🏗 Architecture
![Pipeline Diagram](./workflow.jpg)
*Figure 1: The "Human-in-the-Loop" extraction workflow.*


## 🤖 Extraction Methodology
The raw data is sourced from the BBL API (`api.basketball-bundesliga.de/reports/...`). These PDFs contain complex visual layouts that standard OCR tools fail to parse.

To solve this, I utilized **Gemini 3.0 Flash** as an extraction engine. The model is guided by a strict system prompt to enforce structural consistency (CSV formatting) before the data hits the Python validation layer.

### System Prompt

Read this pdf and return it as a txt. The columns in the pdf and in the txt are:
quarter, time, home action, score, diff, away action. 
Include the quarter starters row with all 5 players for both teams at the beginning of each quarter.
Include the flag "Starters: " in front of both entries and separate the starters using commas.
Return a txt file using pipe delimiters (|) to use in pandas. 
Keep in mind that every player has a jersey number in front of him e.g. 99 SMITH J. 
Include all events, including substitutions.

⚙️ Validation & Cleaning Pipeline

Raw LLM output is rarely perfect. The PDFconversion module runs a series of integrity checks to ensure data quality:

Temporal Consistency: Automatically detects and fills missing quarter values to prevent row-shifts.

Duration Anomaly Detection: Calculates Duration_Seconds between events. Outliers (impossible time gaps) can easily be detected using queries for this column.

Entity Resolution (Player Names): The most common extraction error is hallucinated name variations (e.g., 34 BEAN I vs 34 BEAN J). The pipeline uses a clean_player_name function with a dictionary map to normalize these entities against the official roster. These anomalies can be picked up on later by checking for nan values in the player columns.

Source Data Correction: Occasional errors exist in the official BBL PDFs (e.g., wrong jersey numbers). These also have to be found out using queries with nan values as the error is in the original data.

⚠️ Known Limitations
Possession Attribution: In rare edge cases, an "Away" event may be visually misaligned in the PDF and transcribed into the "Home" column.

Impact: This affects <1% of total rows. However, if this misalignment hits a substitution event, a player may be briefly assigned to the wrong team for that stint. This has been corrected for the master dataset.

Mitigation: For aggregate analysis (Season-long investigations), this noise is statistically negligible. For single-game rotation analysis, manual verification is recommended.







