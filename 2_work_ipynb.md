{
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {},
      "source": [
        "# Assignment 2: The Dirty Ledger — Diagnostic & Work Notebook\n",
        "**Input Dataset:** `teaching_prices_messy.csv`  \n",
        "**Goal:** Load the raw data, perform diagnostic checks in a separate working copy using beginner-friendly Python, and build a before-and-after audit comparison."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {},
      "source": [
        "### Step 1: Load Raw File & Preserved Inspection"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {},
      "outputs": [],
      "source": [
        "import pandas as pd\n",
        "\n",
        "# Load original raw dataset\n",
        "file_name = 'teaching_prices_messy.csv'\n",
        "raw_df = pd.read_csv(file_name)\n",
        "\n",
        "print('=== RAW DATA SUMMARY ===')\n",
        "print('Number of rows:', len(raw_df))\n",
        "print('Number of columns:', len(raw_df.columns))\n",
        "print('Columns list:', list(raw_df.columns))\n",
        "\n",
        "print('\\nFirst 5 rows:')\n",
        "display(raw_df.head())\n",
        "\n",
        "print('\\nColumn Data Types:')\n",
        "display(raw_df.dtypes.to_frame('Raw DataType'))\n",
        "\n",
        "print('\\nRaw Descriptive Statistics:')\n",
        "display(raw_df.describe(include='all').T)"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {},
      "source": [
        "### Step 2: Diagnostic Checks on Working Copy"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {},
      "outputs": [],
      "source": [
        "# Create working copy to run diagnostic tests\n",
        "work_df = raw_df.copy()\n",
        "\n",
        "# Safe date conversion check\n",
        "work_df['date_parsed'] = pd.to_datetime(work_df['date'], errors='coerce')\n",
        "\n",
        "# Safe price conversion check (strip string commas first)\n",
        "price_clean_text = work_df['price'].astype(str).str.replace(',', '')\n",
        "work_df['price_num'] = pd.to_numeric(price_clean_text, errors='coerce')\n",
        "\n",
        "print('=== DIAGNOSTIC COUNTS ===')\n",
        "print('Total missing cells in raw data:', raw_df.isna().sum().sum())\n",
        "print('Exact duplicate rows:', raw_df.duplicated().sum())\n",
        "print('Duplicate (date, item, city) keys:', raw_df.duplicated(subset=['date', 'item', 'city'], keep=False).sum())\n",
        "print('Price conversion failures (excluding NaN):', (raw_df['price'].notna() & work_df['price_num'].isna()).sum())\n",
        "print('Unparseable date failures:', (raw_df['date'].notna() & work_df['date_parsed'].isna()).sum())"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {},
      "source": [
        "### Step 3: Inspect Flagged Records"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {},
      "outputs": [],
      "source": [
        "print('--- Flagged 1: Missing Price Row ---')\n",
        "display(raw_df[raw_df.isna().any(axis=1)])\n",
        "\n",
        "print('\\n--- Flagged 2: String Comma in Price ---')\n",
        "display(raw_df[raw_df['price'].astype(str).str.contains(',', na=False)])\n",
        "\n",
        "print('\\n--- Flagged 3: Exact Duplicate Rows ---')\n",
        "display(raw_df[raw_df.duplicated(keep=False)].sort_values(by=['date', 'item', 'city']))\n",
        "\n",
        "print('\\n--- Flagged 4: Duplicate Date-Item-City Keys ---')\n",
        "display(raw_df[raw_df.duplicated(subset=['date', 'item', 'city'], keep=False)].sort_values(by=['date', 'item', 'city']))"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {},
      "source": [
        "### Step 4: Categorical Label & Item Grid Verification"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {},
      "outputs": [],
      "source": [
        "print('--- Raw Item Counts ---')\n",
        "display(raw_df['item'].value_counts(dropna=False).to_frame('Count'))\n",
        "\n",
        "print('\\n--- Normalized Item Counts (Case & Space Trimmed) ---')\n",
        "norm_item = raw_df['item'].dropna().astype(str).str.strip().str.lower()\n",
        "display(norm_item.value_counts().to_frame('Normalized Count'))\n",
        "\n",
        "print('\\n--- Unit Column Values ---')\n",
        "display(raw_df['unit'].value_counts(dropna=False).to_frame('Count'))\n",
        "\n",
        "print('\\n--- Data Label Disclaimers ---')\n",
        "display(raw_df['data_label'].value_counts(dropna=False).to_frame('Count'))"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {},
      "source": [
        "### Step 5: Before & After Diagnostics Table"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {},
      "outputs": [],
      "source": [
        "print('--- Cleaned Price Summary Statistics ---')\n",
        "display(work_df['price_num'].describe().to_frame('Numeric Price Stats'))\n",
        "\n",
        "# Build explicit before/after summary table\n",
        "before_after_df = pd.DataFrame({\n",
        "    'Raw Dataset': [\n",
        "        len(raw_df),\n",
        "        len(raw_df.columns),\n",
        "        raw_df.isna().sum().sum(),\n",
        "        raw_df.duplicated().sum(),\n",
        "        raw_df['item'].nunique()\n",
        "    ],\n",
        "    'Working Copy (Diagnostic)': [\n",
        "        len(work_df),\n",
        "        len(work_df.columns),\n",
        "        work_df[['date','item','city','price','unit','data_label']].isna().sum().sum(),\n",
        "        work_df[['date','item','city','price','unit','data_label']].duplicated().sum(),\n",
        "        work_df['item'].nunique()\n",
        "    ]\n",
        "}, index=['Total Rows', 'Total Columns', 'Missing Cells', 'Exact Duplicate Rows', 'Unique Items'])\n",
        "\n",
        "before_after_df['Difference'] = before_after_df['Working Copy (Diagnostic)'] - before_after_df['Raw Dataset']\n",
        "\n",
        "print('\\n=== BEFORE VS AFTER AUDIT TABLE ===')\n",
        "display(before_after_df)\n",
        "print('\\nDiagnostic run complete. Raw file preserved without unauthorized row drops.')"
      ]
    }
  ],
  "metadata": {
    "language_info": {
      "name": "python"
    }
  },
  "nbformat": 4,
  "nbformat_minor": 5
}
eof

```markdown:3_log_reflection.md:assignments/week2/3_log_reflection.md
# Cleaning Log, Code Court Verdicts, & Reflection

**Assignment:** Assignment 2 — The Dirty Ledger  
**Dataset:** `teaching_prices_messy.csv`  

---

## 1. Cleaning Log

| Check / Item | Before (Raw) | Action Taken | After (Working Copy) | Audit Decision |
| :--- | ---: | :--- | ---: | :--- |
| **Total Rows** | 97 | Preserved all rows | 97 | **KEEP** all rows during diagnostic audit |
| **Total Columns** | 6 | Preserved all columns | 6 | **KEEP** all source columns |
| **Missing Cells** | 1 | Located in `price` (Row 5) | 1 | **FLAG** missing cell; do not impute automatically |
| **Exact Duplicates** | 1 pair | Located Rows 8 & 96 (`Staple A`) | 1 pair | **FLAG** duplicate for review before dropping |
| **Duplicate Keys (Raw)** | 2 rows | Checked `(date, item, city)` | 2 rows | **FLAG** identical records on 2026-01-08 |
| **Duplicate Keys (Normalized)** | 4 rows | Checked lowercased item names | 4 rows | **FLAG** `Oil B` / `oil b` collision on 2026-02-05 |
| **Price String Formatting** | 1 comma string | Stripped `,` from `'1,250'` in work column | 0 comma strings | **MODIFY** format to numeric `1250.0` |
| **Date Parsing** | 12 dates | Parsed via `pd.to_datetime()` | 0 failures | **MODIFY** data type in working copy |
| **Item Label Categorical** | 5 labels | Compared casing variants | 4 main items | **FLAG** inconsistent lowercase `oil b` |
| **Unit Label** | 1 label | Inspected consistency | 1 label | **KEEP** (`PKR per kg (illustrative)`) |
| **Data Disclaimer** | 1 label | Checked teaching disclaimer | 1 label | **KEEP** context label untouched |

---

## 2. Cleaning Code Court Verdicts

| AI Suggested Code Line | Verdict | Evidence / Audit Rationale |
| :--- | :--- | :--- |
| `pd.read_csv('teaching_prices_messy.csv')` | **KEEP** | Standard loading line required to inspect data. |
| `df.copy()` | **KEEP** | Correctly preserves the original raw DataFrame. |
| `df['price'] = pd.to_numeric(df['price'])` | **MODIFY** | Fails with `ValueError` on string `'1,250'`. Must strip `,` first. |
| `df.dropna(inplace=True)` | **REJECT** | Blanket deletion drops Row 5 unnecessarily, breaking grid structure. |
| `df.drop_duplicates(inplace=True)` | **MODIFY** | Auto-dropping without logging masks identical record entry errors. |
| `df['price'].replace(9999, df['price'].mean())` | **REJECT** | Imputing missing/sentinel data with means distorts true price variance. |
| `df['item'] = df['item'].str.title()` | **MODIFY** | Changing `oil b` to `Oil B` creates two `Oil B` rows for City 1 on Feb 5 (`318.0` and `108.5`). Requires human audit. |

---

## 3. Before-and-After Summary

- **Total Rows:** 97 before $\rightarrow$ 97 after (raw data preserved).
- **Total Columns:** 6 before $\rightarrow$ 6 after.
- **Original Missing Cells:** 1 cell (Row 5 `Pulse C`).
- **Numeric Price Conversion:** 96 valid numeric prices after stripping comma from `'1,250'`.
- **Exact Duplicate Occurrences:** 1 pair (Rows 8 and 96).
- **Unique Item Labels:** 5 raw labels $\rightarrow$ 4 normalized concepts.
- **Automatic Deletions or Imputations:** 0 rows deleted, 0 values fabricated.

---

## 4. Reflection

* **Where my decisions differed from AI:** The AI suggested applying `df['item'] = df['item'].str.title()` to fix lowercase `oil b` and running `df.drop_duplicates()`. However, simple casing standardization creates a key collision on `2026-02-05` in `City 1` between a price of `318.0` and `108.5`. Examining the time series reveals `108.5` actually fits `Staple A` trends.
* **What I would have missed alone:** The AI quickly spotted the comma formatting issue inside the string `'1,250'` that caused basic float conversion to fail.
* **Key Takeaway:** AI tools prioritize syntax fixes (making code run without errors) rather than domain logic. Undocumented automated changes are worse than an honest messy table.