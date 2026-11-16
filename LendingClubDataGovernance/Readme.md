# Data Quality and Governance Framework (Lending Club Case Study)

## Context

Regulatory reporting and credit risk models depend on a handful of fields in lending data. This project shows how a governance team gets from a raw dataset to rules that can be tested. I built a data dictionary, picked the critical data elements, wrote data quality rules for each one, and coded those rules in Python so each one returns a score.

**Scope:** 8 critical data elements, 22 scored data quality rules, run on the first 500,000 loans in the dataset.

## Contents

- [Dataset](#dataset)
- [Tools and technologies](#tools-and-technologies)
- [Project structure](#project-structure)
- [Governance process](#governance-process)
- [Results](#results)
- [Proposed production architecture](#proposed-production-architecture)
- [Future work](#future-work)

## Dataset

Lending Club accepted and rejected loans from 2007 to 2018, from Kaggle.

- Source: [Lending Club dataset on Kaggle](https://www.kaggle.com/datasets/wordsforthewise/lending-club)
- The input CSV is not stored in this repository because of its size. Download the accepted loans file from Kaggle and save it as `Datasets/accepted_2007_to_2018Q4.csv` next to the notebook to run it.

## Tools and technologies

- **Python** with pandas and numpy for the data quality checks
- **Google Sheets** for the data dictionary and rule definitions
- **Canva** for the flowchart

## Project structure

| File | What it contains |
| :--- | :--- |
| `LendingClub Data Dictionary.xlsx` | Field metadata for 54 fields, critical data element selection and justification, and the data quality rules |
| `LoanData_DataCleaningProject.ipynb` | The Python code that scores each data quality rule |
| `AzureDataFlow.png` | A proposed cloud data flow. This is a design only and was not built |

## Governance process

1. **Data dictionary.** Documented metadata for 54 fields.
2. **Critical data element selection.** Picked 8 fields because they identify a loan or feed the target variables of credit risk models: `addr_state`, `grade`, `id`, `int_rate`, `loan_amnt`, `loan_status`, `term` and `total_rec_prncp`.
3. **Rule definition.** Wrote business rules for each critical data element across seven dimensions: accuracy, completeness, consistency, integrity, timeliness, uniqueness and validity.
4. **Operationalization.** Coded the testable rules in Python and scored each one as the share of records that pass.

Examples of the rules:

- **Integrity:** Principal received (`total_rec_prncp`) must not exceed the loan amount (`loan_amnt`).
- **Accuracy:** The loan amount must be the same across the whole account history.
- **Accuracy:** The month over month change in the grade mix must stay within 5%.
- **Integrity:** A loan status cannot return to Current after Charged Off or Fully Paid.

## Results

Every scored rule passed for more than 98.9% of records. The findings that matter:

- **Two records caused most failures.** The same 2 rows failed nearly every completeness and validity check, which points to 2 invalid rows rather than many separate problems. They should be removed during cleaning.
- **Loan status integrity had the most failures.** 15 records broke the rule that a status cannot return to Current after Charged Off or Fully Paid.
- **The lowest score was a grade accuracy check at 98.98%.** One monthly shift in the grade mix went past the 5% threshold.
- **Two fields have the wrong data type.** `id` and `term` are stored as text and need converting to numbers.
- **Some rules cannot be tested on this data.** Timeliness rules, real time accuracy rules and cross system consistency need transaction history, modification dates or a second system. This dataset is a snapshot, so I documented the logic for these rules without running them.

## Proposed production architecture

This design was not built. It shows how the checks would run at scale.

| Component | Tool | Role |
| :--- | :--- | :--- |
| Code | Python, pandas | Vectorized data quality rules |
| Compute | Azure Databricks | Runs the script against data in the data lake |
| Data flow | ADLS, Azure Databricks, Azure SQL Database, Power BI | Moves data from storage to a data quality monitoring report |
| Output | Data quality scorecard | One table with the score for every rule |

## Future work

- Add the timeliness and accuracy checks once the dataset has the fields they need.
- Run generic data quality checks on the non-critical fields, such as `issue_d` and `earliest_cr_line`, which are stored as text instead of dates.
