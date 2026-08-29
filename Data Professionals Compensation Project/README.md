# 📊 Data Professionals Compensation Analysis

## 📌 Project Overview

This project analyzes **compensation and professional characteristics of data professionals** using survey data collected from individuals working in the data field.

The project focuses on preparing a messy compensation dataset for analysis by performing data cleaning and standardization using **Python and Pandas**.

The raw dataset contains information about professionals' roles, career levels, years of experience, industries, gender, technology stacks, salaries, benefits, employer types, and work setups.

The primary objective is to transform the raw survey data into a clean, structured dataset that can be used for further **data analysis and Power BI visualization**.

---

## 🎯 Project Objectives

The project aims to:

* Clean and standardize raw survey data
* Standardize job roles and professional levels
* Categorize industries into consistent groups
* Clean and standardize salary information
* Convert salaries from different currencies into Kenyan Shillings (KES)
* Handle missing and invalid salary responses
* Standardize work setup categories
* Consolidate technology-stack information
* Produce a clean dataset ready for analysis and visualization

---

## 🗂️ Dataset

The original dataset is stored as:

```text
compensation.csv
```

The dataset was imported into Python using Pandas:

```python
df = pd.read_csv("compensation.csv")
```

The original `Timestamp` column was removed because it was not required for the analysis.

---

## 📋 Dataset Variables

The cleaned dataset contains the following fields:

| Column               | Description                           |
| -------------------- | ------------------------------------- |
| `role`               | Current professional role             |
| `other_role`         | Other role provided by respondents    |
| `gender`             | Gender of respondent                  |
| `level`              | Professional/career level             |
| `year_of_experience` | Years of experience in the data field |
| `industry`           | Industry of employer                  |
| `employer_type`      | Type of employer                      |
| `work_setup`         | Work arrangement/setup                |
| `tech_stack`         | Main technology/tool stack            |
| `gross_salary`       | Monthly gross salary in KES           |
| `other_benefits`     | Additional benefits received          |

The original survey questions were renamed to shorter, analysis-friendly column names such as `role`, `level`, `year_of_experience`, `industry`, `gross_salary`, and `work_setup`.

---

# 🧹 Data Cleaning

The raw survey data required several cleaning and transformation steps before it could be used for analysis.

## 1. Column Cleaning

The `Timestamp` column was removed because it was not required for the analytical dataset.

Long survey-question column names were also renamed into concise, descriptive names.

Example:

```python
df = df.rename(columns={
    'What is your Current Role?': 'role',
    'What is your Level?': 'level',
    'How many years of experience do you have in the data Field?': 'year_of_experience',
    'What industry is your Employer? eg Fintech, Utilities, HR, Gaming, Health etc.': 'industry',
    'What is your gender': 'gender',
    'What is your main Tech stack?': 'tech_stack',
    'What is your monthly Gross Salary in Kes per month?': 'gross_salary'
})
```

---

## 2. Role Cleaning

Responses in the `other_role` column were standardized using title case.

Missing values were replaced with empty strings to make the field easier to work with later.

```python
df["other_role"] = df["other_role"].str.title().fillna("")
```

---

## 3. Professional Level Standardization

Different descriptions of professional levels were mapped into consistent categories.

For example:

| Raw Value                           | Clean Value |
| ----------------------------------- | ----------- |
| Mid-Level eg Data Analyst           | Mid-level   |
| Junior eg Junior Data Analyst       | Junior      |
| Manager eg Manager of Analytics     | Manager     |
| Senior Level eg Senior Data Analyst | Senior      |
| Lead eg Lead Analyst                | Lead        |

This makes it easier to compare compensation across career levels.

---

# 🏢 4. Industry Standardization

One of the major cleaning steps involved standardizing employer industries.

The raw survey contained different spellings and descriptions for the same industry.

For example:

```text
education
edtech
school
research/education
```

were mapped to:

```text
Education
```

Similarly, variations such as:

```text
fintech
finetech
agrifintech
```

were standardized as:

```text
Fintech
```

Other industry categories included:

* Banking
* Finance
* Insurance
* Pensions
* Technology
* Telecommunications
* FMCG
* Manufacturing
* Retail
* E-commerce
* Government
* NGO / Non-profit
* NGO / Development
* Healthcare
* Environment & Conservation
* Energy
* Agriculture
* Consulting
* Professional Services
* Marketing & Advertising
* Logistics & Transport
* Automotive
* Engineering & Construction
* Media & Entertainment
* Research
* Tourism
* Religious
* Human Resources
* Other
* Unknown

Before mapping, the industry values were cleaned by:

1. Converting values to string
2. Removing leading/trailing whitespace
3. Converting values to lowercase
4. Applying the industry mapping

```python
df["industry"] = (
    df["industry"]
    .astype("string")
    .str.strip()
    .str.lower()
)

df["industry"] = df["industry"].map(industry_mapping).fillna("")
```

---

# 💰 5. Salary Cleaning

Salary cleaning is one of the most important parts of the project because respondents entered salary information in different formats and currencies.

The raw salary field could contain values using:

* KES
* KSh
* USD
* EUR
* `K` notation
* Million notation
* Commas
* Currency symbols
* Hourly salary responses
* Missing values
* Non-salary responses

A separate `gross_salary_clean` column was initially created for the transformation process.

---

## USD to KES Conversion

USD salary responses were converted to KES using:

```text
$1 = KES 130
```

The script identifies USD responses, extracts the numerical value, removes commas and converts the result to KES.

---

## EUR to KES Conversion

EUR salaries were converted using:

```text
€1 = KES 150
```

The numerical salary value was extracted and multiplied by the exchange rate.

---

## Million Notation

Salary responses such as:

```text
2.5 million
```

were converted to:

```text
2,500,000
```

The script multiplies the extracted value by `1,000,000`.

---

## KES and KSh Formatting

Currency formatting was removed from salary values.

Examples of cleaned values include:

```text
KES 120,000
KSh 120,000
120,000/-
```

The cleaning process removes:

* Commas
* `/-`
* `KSh`
* `Ksh`
* `KES`

---

## K Notation

Salary values such as:

```text
80K
120K
```

were converted to:

```text
80,000
120,000
```

The script identifies values ending in `K` and multiplies the numerical component by `1,000`.

---

## Hourly Salaries

Responses identified as hourly salaries were excluded from the monthly salary analysis by setting them to `NULL`.

```python
df.loc[hourly_mask, "gross_salary_clean"] = pd.NA
```

---

## Missing and Invalid Salary Values

Non-salary responses such as:

```text
Nill
Nil
Unemployed
Prefer not to say
```

were treated as missing values.

The cleaned salary field was then converted to numeric format using:

```python
pd.to_numeric(
    df["gross_salary_clean"],
    errors="coerce"
)
```

Zero and negative salary values were also converted to missing values.

Finally, the cleaned salary was converted to Pandas' nullable integer type and written back to the `gross_salary` column.

---

# 🏠 6. Work Setup Cleaning

Work setup responses were standardized to make different hybrid-work descriptions easier to analyze.

For example:

```text
Hybrid- I need to be in the office specific days in a week
```

was standardized to:

```text
Hybrid- Specific office days
```

While:

```text
Hybrid-I choose my days to go the office
```

was standardized to:

```text
Hybrid- Choose office days
```

---

# 💻 7. Technology Stack Consolidation

The project combines the respondent's main technology stack with any additional tools they provided.

```python
df["tech_stack_all"] = df["tech_stack"] + "," + df["other_stack"]
```

The original technology-stack columns were then removed and the combined column was renamed to:

```text
tech_stack
```

---

# 📐 8. Final Dataset Structure

After cleaning, the dataset was rearranged into a logical analytical structure:

```python
df = df[[
    'role',
    'other_role',
    'gender',
    'level',
    'year_of_experience',
    'industry',
    'employer_type',
    'work_setup',
    'tech_stack',
    'gross_salary',
    'other_benefits'
]]
```

---

# 📁 Output Dataset

The final cleaned dataset is exported as:

```text
data_professionals_compensation.csv
```

using:

```python
df.to_csv(
    "data_professionals_compensation.csv",
    index=False
)
```

The exported dataset is then loaded again to verify the cleaned output.

---




# 📂 Repository Structure

A recommended GitHub structure for this project is:

```text
data-professionals-compensation/
│
├── README.md
│
├── data/
│   ├── compensation.csv
│
├── data_cleaning/
│   └── data_compensation.ipynb
```

---

# 🔄 Project Workflow

```text
                RAW DATA
                   │
                   ▼
          compensation.csv
                   │
                   ▼
          DATA CLEANING
                   │
        ┌──────────┴──────────┐
        │                     │
   Standardize           Clean Salary
   Categories            & Currency
        │                     │
        └──────────┬──────────┘
                   │
                   ▼
          CLEAN DATASET
                   │
                   ▼
data_professionals_compensation.csv
                   │
                   ▼

---

# 🎯 Key Skills

Through this project, the following practical data analytics skills are demonstrated:

* Data ingestion using Python
* Exploratory data inspection
* Data cleaning
* Data standardization
* Missing-value handling
* String manipulation
* Data transformation
* Currency conversion
* Data type conversion
* Categorical mapping
* Dataset restructuring
* CSV export
* Preparation of datasets for BI tools
* Business-oriented data visualization

---
