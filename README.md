🏥 **Hospital Operations & Medication Adherence Analysis**   
Tooling: SQLite, DB Browser, Power BI (Visualizations)   

Dataset: Healthcare Dataset (Kaggle)   

📌 Project Overview
This project evaluates hospital performance and patient outcomes by analyzing medication adherence and operational efficiency. I transformed raw, inconsistently formatted healthcare data into a dynamic dashboard that allows stakeholders to track patient recovery times, billing trends, and drug compliance across various medical conditions.   

🛠️ Data Cleaning & ETL
To prepare the data for visualization, I performed the following steps in Power Query and SQLite:    

**Name Normalization**: Fixed inconsistent casing (e.g., "andrEw waTtS" → "Andrew Watts") using Capitalize Each Word transformations.   
**Temporal Logic**: Reformatted DD-MM-YYYY date strings into standard ISO formats to calculate Length of Stay.   
**Drug Classification**: Grouped individual medications into clinical categories (Pain Relief, Antibiotics, Cardiovascular) using conditional logic.   

🔍 Key SQL Queries
I used SQLite to extract key metrics before building the dashboard.   
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
2. Operational Efficiency (Average Length of Stay) <br>
Identifying which admission types (Emergency vs. Elective) occupy beds the longest / experience the longest hospital stays. <br>

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
📊 Power BI Dashboard   

![Dashboard](https://github.com/anushkashajith/DA_Projects/blob/main/Screenshot.png "Visualisation")
The dashboard consists of three main sections:   

**Executive KPIs**: Real-time tracking of Total Patients, Average Billing, and Overall Adherence.  
**Adherence Trends**: A breakdown of which drug classes see the highest patient compliance.  
**Patient Demographics**: A deep dive into medical conditions and test results to identify high-risk groups.  

💡 Insights & Conclusions

**Compliance Gap**: Cardiovascular medications (Lipitor) show a lower adherence rate than Antibiotics, suggesting a need for better patient follow-up.  
**Billing Patterns**: Patients with Urgent admissions incur 15% higher billing amounts on average than Elective admissions.   
**Data Quality**: By normalizing the "Name" field, I improved the usability of the patient lookup table for administrative staff.  
