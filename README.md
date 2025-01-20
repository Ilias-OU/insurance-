# 🏥 Healthcare Insurance Dataset SQL Analysis

![Healthcare](https://today.uconn.edu/wp-content/uploads/2017/04/shutterstock_373492012-health-insurance-e1491415000969.jpg)

## 📄 Overview

This project analyzes the **Healthcare Insurance dataset** to uncover trends in insurance costs, demographics, and other key factors. Using SQL, we explore patterns in healthcare insurance data, demonstrating skills in data manipulation, aggregation, and advanced SQL techniques to generate actionable insights.

---

## 📂 Dataset

Access the dataset here: [Healthcare Insurance Dataset on Kaggle](https://www.kaggle.com/datasets/willianoliveiragibin/healthcare-insurance).

## 🔍 SQL Query Analysis

#### 1. Retrieve All Data
```
SELECT *
FROM [insurance];

```

#### 2.Change Data Types for Various Columns
```
ALTER TABLE [insurance]
ALTER COLUMN age INT;

ALTER TABLE [insurance]
ALTER COLUMN bmi FLOAT;

ALTER TABLE [insurance]
ALTER COLUMN children INT;

ALTER TABLE [insurance]
ALTER COLUMN charges FLOAT;

```
#### 3. View Table Schema
```
EXEC sp_help [insurance];
```

#### 4. Find the Average Charges for Each Region
```
SELECT 
    AVG(charges) AS avgcha, 
    region
FROM [insurance]
GROUP BY region;
```

#### 5. Find the Total Number of Children for Smokers vs. Non-Smokers
```
SELECT smoker, SUM(children) AS totalchildren
FROM [insurance]
GROUP BY smoker;
```

#### 6. List the Top 5 Individuals with the Highest Charges
```
SELECT TOP 5 *
FROM [insurance]
ORDER BY charges DESC;
```

#### 7. Identify Individuals Whose Charges Are Above the Average for Their Region
```
WITH Cte1 AS (
    SELECT *,
           AVG(charges) OVER (PARTITION BY region) AS avgcharges
    FROM [insurance]
)
SELECT *
FROM Cte1
WHERE charges > avgcharges;
```

#### 8.Pair Individuals with Identical BMI Values
```
WITH Ranked AS (
    SELECT age, sex, ROW_NUMBER() OVER (ORDER BY bmi) AS rn, charges, bmi
    FROM [insurance]
)
SELECT a.age, a.sex, a.charges
FROM Ranked a
JOIN Ranked b
ON a.bmi = b.bmi AND a.rn < b.rn;
```

#### 9.Calculate the Rank of Individuals Based on Charges Within Each Region
```
SELECT *,
       ROW_NUMBER() OVER (PARTITION BY region ORDER BY charges) AS ranking
FROM [insurance];
```

#### 10.Find the Maximum Charges for Smokers in Each Region
```
SELECT region, MAX(charges)
FROM [insurance]
GROUP BY region;
```

#### 11.Retrieve Records for Individuals Aged Between 25 and 40 With More Than 2 Children in the Northwest Region
```
SELECT *
FROM [insurance]
WHERE age BETWEEN 25 AND
 40
  AND children > 2
  AND region = 'northwest';
```

#### 12.Create a CTE to Identify Smokers Whose BMI Exceeds the Average for Smokers
```
WITH AvgBMI AS (
    SELECT AVG(bmi) AS avg_bmi, smoker
    FROM [insurance]
    GROUP BY smoker
)
SELECT *
FROM [insurance] a
JOIN AvgBMI b
ON a.smoker = b.smoker AND a.bmi > b.avg_bmi;
```

#### 13.Simulate Cumulative Charges Incrementally for Each Individual (Ordered by ID)
```
WITH RecursiveCTE AS (
    SELECT id, charges
    FROM [insurance]
    WHERE id = 1
    UNION ALL
    SELECT a.id, a.charges + r.charges
    FROM [insurance] a
    JOIN RecursiveCTE r
    ON a.id = r.id + 1
)
SELECT *
FROM RecursiveCTE;
```

#### 14.Retrieve Individuals Whose Charges Are in the Top 10% for Their Region
```
SELECT *
FROM [insurance] a
WHERE charges > (
    SELECT PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY charges)
    FROM [insurance] b
    WHERE a.region = b.region
);
```

#### 15.Create a Pivot Table Showing Average Charges for Smokers and Non-Smokers in Each Region
```
SELECT region,
       AVG(CASE WHEN smoker = 'yes' THEN charges END) AS avg_charges_smokers,
       AVG(CASE WHEN smoker = 'no' THEN charges END) AS avg_charges_non_smokers
FROM [insurance]
GROUP BY region;
```

#### 16.Classify Individuals as High Risk, Moderate Risk, or Low Risk
```
SELECT *,
       CASE 
           WHEN bmi > 30 AND smoker = 'yes' THEN 'High Risk'
           WHEN bmi > 30 AND smoker = 'no' THEN 'Moderate Risk'
           ELSE 'Low Risk'
       END AS risk_level
FROM [insurance];
```

#### 17.Nested Aggregations: Calculate Region-Wise Average Charges and Find the Region With the Highest Average
```
WITH RegionAvg AS (
    SELECT region, AVG(charges) AS avg_charges
    FROM [insurance]
    GROUP BY region
)
SELECT region, avg_charges
FROM RegionAvg
WHERE avg_charges = (
    SELECT MAX(avg_charges)
    FROM RegionAvg
);
```

