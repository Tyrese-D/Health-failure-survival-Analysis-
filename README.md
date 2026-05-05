# Health-failure-survival-Analysis-

## Project Overview
This project focuses on transforming raw healthcare transactional data into a
clean, analysis-ready dataset by identifying and resolving data quality issues
across 55,500 patient records before loading into a data warehouse or BI tool.


## The Data Analysis Process
I followed a 3-stage ETL framework to move from raw data to a clean, warehouse-ready dataset:

1. **Extract:** Loaded the raw CSV into a pandas DataFrame as a staging environment,
   confirmed the shape as 55,500 rows and 15 columns, and flagged that
   `Date of Admission` and `Discharge Date` were loaded as plain strings instead
   of datetime objects, and that `Billing Amount` needed validation for negative
   values before any financial analysis could begin
2. **Transform:** Applied all cleaning operations across six categories — null
   validation, duplicate removal, name standardization, whitespace correction,
   date conversion and logical validation, and negative billing handling
3. **Load:** Enforced schema-level data types, hard constraints, composite key
   deduplication logic, and admission date partitioning in the target warehouse

## Data Cleaning Steps

### Null & Missing Values
- **Null Check:** Ran `.isnull().sum()` across all 15 columns — returned zero
  nulls across the entire dataset
- **Verification:** Did not accept the result at face value — at 55,500 records,
  a fully clean null report often means missing values were filled upstream with
  placeholder text like "N/A" or "Unknown" before the file was delivered
- **Whitespace Audit:** Applied `.str.strip()` across all text columns because a
  value like `"Cancer "` with a trailing space passes a null check but is treated
  as a completely different category by SQL, pandas, and every BI tool
- **Forward-Looking Rules:** Established imputation rules for future loads —
  median imputation for Age, flagging rather than imputing for Billing Amount,
  and standardized `"Unknown"` labels for categorical columns rather than
  dropping the row

### Duplicate Records
- **Discovery:** Ran `df.duplicated().sum()` and found 534 exact duplicate rows
  matching another record in all 15 columns — same name, age, admission date,
  billing amount, and discharge date
- **Root Causes:** Identified likely causes as an ETL pipeline that ran more than
  once without checking for existing records, a manual form submitted twice, or a
  source system bug that logged the same admission event twice
- **Resolution:** Removed all 534 duplicates using `drop_duplicates()`, reducing
  the dataset from 55,500 rows to 54,966 rows
- **Impact:** Leaving duplicates in would have overstated every count-based
  metric — total admissions, average billing per condition, patient volume per
  hospital, and any model trained on this data would have learned from false
  observations

### Inconsistent Name Formatting
- **Discovery:** Inspected the Name column and found severe random casing across
  all 55,500 records — examples included `Bobby JacksOn`, `LesLie TErRy`,
  `DaNnY sMitH`, `EMILY JOHNSOn`, and `CHrisTInA MARtinez`
- **Root Cause:** Manual data entry across multiple systems with no input
  validation enforcing a consistent format at the point of capture
- **Functional Impact:** `Bobby Jackson`, `BOBBY JACKSON`, and `BoBby jACKson`
  are treated as three different people by SQL, pandas, and every BI tool —
  breaking readmission tracking, fragmenting longitudinal patient records, and
  causing name-based duplicate detection to produce false negatives
- **Resolution:** Standardized all 55,500 names to Title Case using `.str.title()`
  in a single operation and applied `.str.strip()` to remove any surviving
  leading or trailing whitespace

### Date Type Conversion & Logical Validation
- **Type Conversion:** Converted both `Date of Admission` and `Discharge Date`
  from plain string objects to proper datetime objects using `pd.to_datetime()`
  — string-stored dates cannot be filtered by range, aggregated by month, or
  used in time-series calculations without conversion
- **LOS Derivation:** Derived a new Length of Stay column calculated as
  `Discharge Date − Date of Admission` in days to validate date logic
- **Overlap Check:** Checked for any record where discharge date fell before
  admission date — returned zero violations across all records
- **LOS Statistics:** Confirmed the distribution was internally consistent with
  a minimum of 1 day, a maximum of 30 days, and a mean of approximately 15.5 days

### Negative Billing Amounts
- **Discovery:** Ran `.describe()` on Billing Amount and found a minimum value
  of -$2,008.49 — a negative charge in a field meant to represent patient billing
- **Root Causes:** Identified likely causes as a refund or credit adjustment
  stored in the wrong field, a data entry error with an accidental minus sign, or
  a financial correction not separated from original charges at the source system
- **Resolution:** Did not delete those rows — the rest of the patient record was
  still valid — instead flagged negative values with a boolean column
  `is_billing_adjustment = True`, moved them into a dedicated Adjustments
  category, and excluded them from all standard billing dashboards and averages

### Data Type Enforcement
- **Dates:** Enforced `Date of Admission` and `Discharge Date` as DATE types,
  not VARCHAR
- **Billing:** Enforced `Billing Amount` as DECIMAL for financial precision,
  not FLOAT
- **Numerics:** Enforced `Age` and `Room Number` as INTEGER
- **Categoricals:** Enforced all text columns as standardized string categories
  with controlled vocabularies

## Load & Pipeline Design
- **Schema Constraints:** Defined hard constraints at the warehouse schema level —
  `Billing Amount >= 0` in the main table, `Discharge Date >= Date of Admission`,
  `Age` between 0 and 130, and controlled vocabularies for all categorical columns
- **Deduplication at Load:** Built a composite key check using
  `Patient Name + Date of Admission + Hospital` so only net-new records are
  inserted on each pipeline run — preventing duplicate accumulation from recurring
- **Partitioning:** Enforced partitioning by admission date in the warehouse
  schema so date-range queries and monthly aggregations run efficiently without
  full table scans

## Business Impact

### Removing 534 Duplicate Records
- **Risk Prevented:** Inflated admission counts would have overstated patient
  volume, skewed average billing per condition, and corrupted any machine learning
  model trained on this data
- **Operational Impact:** For a healthcare organization, inflated admission counts
  directly affect staffing decisions, bed allocation planning, and insurance
  reimbursement reporting — where overcounted admissions can trigger compliance
  audits or financial penalties

### Standardizing Patient Name Formatting
- **Risk Prevented:** Without consistent casing, the same patient under different
  name formats would appear as multiple distinct individuals — breaking readmission
  rate analysis, fragmenting longitudinal records, and causing patient outreach
  campaigns to send duplicate communications to the same person
- **Operational Impact:** Every name-based join, deduplication check, and patient
  lookup now behaves consistently across every tool and every analyst accessing
  the data

### Resolving Negative Billing Amounts
- **Risk Prevented:** Negative values summed alongside positive charges would
  have understated total revenue, distorted average billing per condition, and
  produced unreliable financial forecasts and insurance reimbursement calculations
- **Operational Impact:** Finance teams can now trust that billing figures in the
  main dataset represent actual patient charges — not a mix of charges and
  unprocessed credits sitting in the wrong field

### Validating Date Logic & Enforcing Date Types
- **Risk Prevented:** A single inverted date pair would corrupt the entire Length
  of Stay distribution — directly affecting bed turnover analysis, staffing ratios,
  and cost-per-episode calculations built on LOS data
- **Operational Impact:** Converting dates from strings to DATE types eliminated
  an entire class of query inconsistency — different analysts running the same
  report now get the same result regardless of how they format the date filter

### Enforcing Schema Constraints at Load
- **Risk Prevented:** Without constraints, future data loads could silently
  introduce the same quality issues — negative billing, inverted dates, invalid
  categories — and corrupt the warehouse months after the initial cleanup
- **Operational Impact:** Any future record violating a constraint is rejected
  and logged for review, creating an auditable trail of data quality incidents
  rather than letting bad data accumulate undetected

### Building Deduplication Into the Pipeline
- **Risk Prevented:** Removing the 534 existing duplicates was a one-time fix —
  without a structural change to the load process, the same problem would recur
  on every future pipeline run
- **Operational Impact:** The composite key check at load time means the warehouse
  can never accumulate duplicates from repeated loads, even if the upstream system
  delivers the same file twice — eliminating an entire category of recurring risk

## Repository Structure
- `/data`: Raw CSV file used for analysis
- `/notebooks`: Python notebook containing all cleaning and validation steps
- `/outputs`: Cleaned dataset exported after all transformations were applied

Outcome Variable: Death Event (Survival vs. Mortality).
<img width="1920" height="877" alt="Screenshot 2026-04-27 at 11 57 46 AM" src="https://github.com/user-attachments/assets/46b25a9d-d9f7-44f8-89dc-8cc3db400594" />


🛠️ Tech Stack
Analysis: Tableau Desktop / Tableau Public.

Data Prep: Excel (Cleaning and categorical encoding).

Statistical Visualization: Box-and-Whisker Plots, Stacked Histograms, Scatter Plot Matrices.

Live interactive dashboard: https://public.tableau.com/app/profile/tyrese.dieudonne/viz/HeartFailurePatientSurvivalAnalysis/Dashboard1
<img width="1920" height="1080" alt="Screenshot 2026-05-05 at 4 35 15 AM" src="https://github.com/user-attachments/assets/01004c63-d889-44b2-9391-9ed7dae3565e" />



Machine Learning Integration: Predictive Modeling
While the Tableau dashboard provides a visual diagnostic tool, I implemented a Python-based machine learning pipeline to automate risk assessment.

  1. Models Evaluated: Logistic Regression, Random Forest, Support Vector Machine (SVM), and Decision Trees.
 
  2. Top Performer: The Random Forest Classifier achieved the highest accuracy (approx. 85-90%), effectively handling the non-linear relationships between clinical biomarkers.

 3. Feature Engineering: Utilized StandardScaler to normalize continuous variables like Platelets and Creatinine Phosphokinase, ensuring high-variance features didn't bias the model.
4. Evaluation: Leveraged Confusion Matrices and Accuracy scores to validate model reliability against the test dataset.


Evaluation: Leveraged Confusion Matrices and Accuracy scores to validate model reliability against the test dataset.



The Model vs. The Dashboard: I learned that while a model gives a "prediction," the Tableau dashboard provides the "explanation." Combining both allows a doctor to see a high-risk score and the specific biomarkers causing it.

Data Imbalance Challenges: In clinical datasets, "Death Events" are often fewer than "Survival Events." I learned how to evaluate models using more than just accuracy—focusing on Precision and Recall to ensure the model doesn't miss high-risk patients (False Negatives).

Algorithm Selection: I discovered that while Logistic Regression is great for interpretability, Ensemble methods like Random Forest are superior for capturing the complex, interconnected nature of human health data

Predictive Triage System
1. By deploying the Random Forest model into a clinical workflow, hospitals could implement an Automated Triage Alert. When a patient's lab results are entered, the system can instantly flag those with a high "Mortality Probability," allowing staff to prioritize care before symptoms escalate.

2. Consumer & Patient Trust
A major concern in "Black Box" AI is trust.

Recommendation: Use the Tableau visualizations as a "Translation Layer." When the ML model flags a patient, show the patient the Feature Importance (e.g., "Your Serum Creatinine is significantly above the median") to make the AI's decision transparent and actionable.


 Strategic Recommendations
1. Automated Clinical Triage
Implementation: Hospitals can use this Random Forest model as a "second set of eyes" to automatically flag patients with a mortality probability >60%.

Goal: Allow medical staff to prioritize high-risk specialist reviews before symptoms escalate.

2. Transparent AI (XAI)
Trust Factor: To address patient concern over "Black Box" algorithms, use the Tableau visuals to show patients why they were flagged.

Actionable Insights: Turning a "Risk Score" into a visual representation of their biomarkers makes the data a conversation starter for lifestyle changes rather than a scary number.
