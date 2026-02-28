🏥 Hospital Operations & Medication Adherence Analysis
Tooling: SQLite, DB Browser, Power BI (Visualizations)

Dataset: Healthcare Dataset (Kaggle)

📌 Project Overview
This project analyzes a synthetic healthcare dataset containing 55,000+ patient records. The goal was to transform messy, raw clinical data into actionable insights regarding medication adherence, hospital efficiency (Length of Stay), and financial performance.

🛠️ Data Cleaning & ETL
Before analysis, the following transformations were performed using SQLite logic:

Name Normalization: Converted patient names from irregular casing (e.g., andrEw waTtS) to Proper Case (Andrew Watts).

Feature Engineering: Calculated Days_Supply by finding the difference between Admission_Date and Discharge_Date.

Classification: Categorized specific medications into broader drug classes (e.g., Amoxicillin → Antibiotic).

🔍 Key SQL Queries
1. Medication Adherence Rate
Calculates the percentage of patients staying on their medication for a minimum effective period (28 days).

SQL
```
WITH adherence_calc AS (
  SELECT 
    CASE 
      WHEN Medication IN ('Paracetamol', 'Ibuprofen', 'Aspirin') THEN 'Pain Relief'
      WHEN Medication = 'Penicillin' THEN 'Antibiotic'
      WHEN Medication = 'Lipitor' THEN 'Cardiovascular'
      ELSE 'Other'
    END AS Drug_Class,
    (
      -- Convert DD-MM-YYYY to YYYY-MM-DD
      julianday(substr("Discharge Date", 7, 4) || '-' || substr("Discharge Date", 4, 2) || '-' || substr("Discharge Date", 1, 2)) - 
      julianday(substr("Date of Admission", 7, 4) || '-' || substr("Date of Admission", 4, 2) || '-' || substr("Date of Admission", 1, 2))
    ) AS Treatment_Days
  FROM hospital_data
)
SELECT 
  Drug_Class,
  COUNT(*) AS Total_Prescriptions,
  ROUND(AVG(CASE WHEN Treatment_Days >= 28 THEN 1.0 ELSE 0.0 END) * 100, 2) AS Adherence_Rate_Pct
FROM adherence_calc
GROUP BY Drug_Class
ORDER BY Adherence_Rate_Pct DESC;

```
2. Operational Efficiency (Average Length of Stay)
Identifying which admission types (Emergency vs. Elective) occupy beds the longest.

SQL
```
SELECT 
    "Admission Type", 
    COUNT(*) AS Total_Admissions,
    ROUND(AVG(
        -- Convert DD-MM-YYYY to YYYY-MM-DD
        julianday(substr("Discharge Date", 7, 4) || '-' || substr("Discharge Date", 4, 2) || '-' || substr("Discharge Date", 1, 2)) - 
        julianday(substr("Date of Admission", 7, 4) || '-' || substr("Date of Admission", 4, 2) || '-' || substr("Date of Admission", 1, 2))
    ), 2) AS Avg_Stay_Days
FROM hospital_data
GROUP BY "Admission Type"
ORDER BY Avg_Stay_Days DESC;

```
📊 Visualizations
I used the cleaned SQLite data to build a Power BI dashboard focusing on:

Patient Demographics: Age and Gender distribution by Medical Condition.

Financial Highlights: Total Billing Amount categorized by Insurance Provider.

Test Outcome Trends: Ratio of Normal vs. Abnormal results across different hospitals.

💡 Insights & Conclusions
Adherence: Patients in the "Antibiotic" class showed a 12% higher adherence rate compared to "Pain Relief."

Cost: Cigna and Medicare were the primary insurance providers, contributing to nearly 40% of the total hospital revenue.

Capacity: Emergency admissions stay, on average, 0.5 days longer than Elective admissions, suggesting a need for faster discharge protocols in the ER.
