# Hospital Analytics

Analyze hospital **patient encounters**, **costs**, and **behavior trends** to uncover operational and financial insights.

---

## Objectives Overview

1. **Encounters Overview**  
   Explore the encounters table by slicing data by **year**, **encounter class**, and **encounter length**.

2. **Cost & Coverage Insights**  
   Examine payer coverage, top procedures, and claim cost trends.

3. **Patient Behavior Analysis**  
   Analyze quarterly admissions and readmission patterns.

---

## Objective 1: Encounters Overview

### Task 1 — Yearly and Class-Based Insights

#### 1️⃣ How many total encounters occurred each year?
```sql
SELECT * FROM encounters; 

SELECT YEAR(START) AS Year , COUNT(Id) AS Total_encounters FROM encounters
GROUP BY Year
ORDER BY Year; 
````

#### 2️⃣ For each year, what percentage of all encounters belonged to each encounter class?
```sql
SELECT DISTINCT ENCOUNTERCLASS FROM encounters
# ambulatory | outpatient | wellness | urgentcare | emergency | inpatient

-- number of encounters for each encounter class
SELECT YEAR(START) AS Year,
	SUM(CASE WHEN ENCOUNTERCLASS = 'ambulatory' THEN 1 ELSE 0 END) AS ambulatory,
	SUM(CASE WHEN ENCOUNTERCLASS = 'outpatient' THEN 1 ELSE 0 END) AS outpatient,
	SUM(CASE WHEN ENCOUNTERCLASS = 'wellness' THEN 1 ELSE 0 END) AS wellness,
	SUM(CASE WHEN ENCOUNTERCLASS = 'urgentcare' THEN 1 ELSE 0 END) AS urgent_care,
	SUM(CASE WHEN ENCOUNTERCLASS = 'emergency' THEN 1 ELSE 0 END) AS emergency,
	SUM(CASE WHEN ENCOUNTERCLASS = 'inpatient' THEN 1 ELSE 0 END) AS inpatient
 FROM encounters
 GROUP BY Year; 

SELECT 
  YEAR(`START`) AS Year,
  SUM(CASE WHEN ENCOUNTERCLASS = 'ambulatory' THEN 1 ELSE 0 END) AS ambulatory,
  SUM(CASE WHEN ENCOUNTERCLASS = 'outpatient' THEN 1 ELSE 0 END) AS outpatient,
  SUM(CASE WHEN ENCOUNTERCLASS = 'wellness' THEN 1 ELSE 0 END) AS wellness,
  SUM(CASE WHEN ENCOUNTERCLASS = 'urgentcare' THEN 1 ELSE 0 END) AS urgent_care,
  SUM(CASE WHEN ENCOUNTERCLASS = 'emergency' THEN 1 ELSE 0 END) AS emergency,
  SUM(CASE WHEN ENCOUNTERCLASS = 'inpatient' THEN 1 ELSE 0 END) AS inpatient
FROM encounters
GROUP BY YEAR(`START`);


-- percentage of encounters
SELECT YEAR(START) AS Year, 
	ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'ambulatory' THEN 1 ELSE 0 END)/   COUNT(*) * 100, 1) AS ambulatory,
	ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'outpatient' THEN 1 ELSE 0 END)/   COUNT(*) * 100, 1) AS outpatient,
	ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'wellness' THEN 1 ELSE 0 END)/   COUNT(*) * 100, 1)  AS wellness,
	ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'urgentcare' THEN 1 ELSE 0 END)/   COUNT(*) * 100, 1) AS urgent_care,
	ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'emergency' THEN 1 ELSE 0 END)/   COUNT(*) * 100, 1) AS emergency,
	ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'inpatient' THEN 1 ELSE 0 END)/   COUNT(*) * 100, 1)  AS inpatient,
    COUNT(*) AS total_num_of_encounters
 FROM encounters
 GROUP BY Year
 ORDER BY Year; 

```
#### 3️⃣  What percentage of encounters were **over 24 hours** versus **under 24 hours**?
```sql
SELECT * FROM encounters; 

SELECT TIMESTAMPDIFF(HOUR, START, STOP)
FROM encounters
WHERE TIMESTAMPDIFF(HOUR, START, STOP) >= 24 ; 

SELECT COUNT(TIMESTAMPDIFF(HOUR, START, STOP))
FROM encounters
WHERE TIMESTAMPDIFF(HOUR, START, STOP) >= 24 ;

SELECT COUNT(TIMESTAMPDIFF(HOUR, START, STOP))
FROM encounters
WHERE TIMESTAMPDIFF(HOUR, START, STOP) < 24 ;

SELECT 
SUM(CASE WHEN TIMESTAMPDIFF(HOUR, START, STOP) >= 24 THEN 1 ELSE 0 END) AS  over_24_hours,
SUM(CASE WHEN TIMESTAMPDIFF(HOUR, START, STOP) < 24 THEN 1 ELSE 0 END) AS  under_24_hours 
FROM encounters; 

-- percentage of encounterss over & under 24 hours
SELECT 
ROUND(SUM(CASE WHEN TIMESTAMPDIFF(HOUR, START, STOP) >= 24 THEN 1 ELSE 0 END)/   COUNT(*) * 100, 1)  AS  over_24_hours,
ROUND(SUM(CASE WHEN TIMESTAMPDIFF(HOUR, START, STOP) < 24 THEN 1 ELSE 0 END)/   COUNT(*) * 100, 1)  AS  under_24_hours,
COUNT(*)
FROM encounters; 
```
---

## Objective 2: Cost & Coverage Insights

### Task 2 — Financial & Payer Analysis

#### 1️⃣ How many encounters had zero payer coverage, and what percentage of total encounters does this represent?

```sql
SELECT * FROM encounters; 

SELECT * FROM encounters WHERE PAYER_COVERAGE = 0;

SELECT COUNT(*) AS total_num_of_encounters FROM encounters ;

SELECT COUNT(*) AS total_num_of_encounters FROM encounters WHERE PAYER_COVERAGE = 0;

SELECT  SUM(CASE WHEN PAYER_COVERAGE = 0 THEN 1 ELSE 0 END) AS Zero_payer_coverage,
COUNT(*) AS total_num_of_encounters,
ROUND(SUM(CASE WHEN PAYER_COVERAGE = 0 THEN 1 ELSE 0 END) / COUNT(*) * 100,1) AS Percentage
FROM encounters; 


SELECT COUNT(*) AS total_num_of_encounters, PAYER_COVERAGE FROM encounters
GROUP BY PAYER_COVERAGE
ORDER BY PAYER_COVERAGE; 
```

#### 2️⃣ What are the **top 10 most frequent procedures** and their **average base cost**?
```sql
SELECT * FROM procedures; 

SELECT DISTINCT DESCRIPTION, AVG(BASE_COST) FROM procedures
GROUP BY DESCRIPTION; 

SELECT CODE, DESCRIPTION, COUNT(*) AS number_of_procedures,  AVG(BASE_COST) AS Avg_base_cost FROM procedures
GROUP BY CODE, DESCRIPTION
ORDER BY number_of_procedures DESC
LIMIT 10;
```

#### 3️⃣ What are the **top 10 most expensive procedures** on average, and how many times were they performed?
```sql
SELECT CODE, DESCRIPTION,  AVG(BASE_COST) AS Avg_base_cost, COUNT(*) AS number_of_procedures FROM procedures
GROUP BY CODE, DESCRIPTION
ORDER BY Avg_base_cost DESC
LIMIT 10;
```

#### 4️⃣ What is the **average total claim cost per payer**?
```sql
SELECT * FROM encounters; -- payer 

SELECT * FROM payers; -- id

SELECT PAYER,  AVG(TOTAL_CLAIM_COST) FROM encounters
GROUP BY PAYER; 

SELECT PAYER, TOTAL_CLAIM_COST FROM payers AS p
LEFT JOIN encounters AS e ON p.id = e.payer ; 

SELECT p.name, AVG(e.TOTAL_CLAIM_COST) AS Avg_total_claim_cost FROM payers AS p
LEFT JOIN encounters AS e ON p.id = e.payer 
GROUP BY p.name
ORDER BY Avg_total_claim_cost DESC; 
```

---

## Objective 3: Patient Behavior Analysis

### Task 3 — Admission & Readmission Patterns

#### 1️⃣ How many unique patients were admitted each quarter over time?
```sql
SELECT * FROM patients;
SELECT * FROM encounters;
 
SELECT PATIENT, START, STOP FROM encounters;
SELECT PATIENT, QUARTER(START) FROM encounters;

SELECT COUNT( DISTINCT PATIENT) AS Unique_patients, YEAR(START) AS YEAR, QUARTER(START) AS Quarter FROM encounters
GROUP BY YEAR, Quarter 
ORDER BY YEAR, Quarter;
```

---

#### 2️⃣ How many patients were **readmitted within 30 days** of a previous encounter?
```sql
SELECT * FROM encounters;
SELECT PATIENT, START, STOP FROM encounters;

SELECT PATIENT, START, STOP ,
LEAD(START) OVER(PARTITION BY PATIENT ORDER BY START) AS next_start_date
FROM encounters
ORDER BY PATIENT, START;

WITH cte AS 
(SELECT PATIENT, START, STOP ,
LEAD(START) OVER(PARTITION BY PATIENT ORDER BY START) AS next_start_date
FROM encounters)
SELECT * FROM cte; 

-- total number of patients
WITH cte AS 
(SELECT PATIENT, START, STOP ,
LEAD(START) OVER(PARTITION BY PATIENT ORDER BY START) AS next_start_date
FROM encounters)
SELECT COUNT(DISTINCT PATIENT) FROM cte; 

-- patients readmitted over time
WITH cte AS 
(SELECT PATIENT, START, STOP ,
LEAD(START) OVER(PARTITION BY PATIENT ORDER BY START) AS next_start_date
FROM encounters)
SELECT * FROM cte 
WHERE DATEDIFF(next_start_date, STOP) < 30 ; 

-- number of patients within 30 days
WITH cte AS 
(SELECT PATIENT, START, STOP ,
LEAD(START) OVER(PARTITION BY PATIENT ORDER BY START) AS next_start_date
FROM encounters)
SELECT COUNT(DISTINCT PATIENT) AS number_of_patients FROM cte 
WHERE DATEDIFF(next_start_date, STOP) < 30 ; 
```

#### 3️⃣ Which patients had the **most readmissions**?
```sql
WITH cte AS 
(SELECT PATIENT, START, STOP ,
LEAD(START) OVER(PARTITION BY PATIENT ORDER BY START) AS next_start_date
FROM encounters)
SELECT PATIENT, COUNT(*) AS number_of_readmissions FROM cte 
WHERE DATEDIFF(next_start_date, STOP) < 30 
GROUP BY PATIENT
ORDER BY number_of_readmissions DESC ; 

-- detailed look at patients though id
SELECT * FROM encounters WHERE PATIENT = '1712d26d-822d-1e3a-2267-0a9dba31d7c8'; 
```

---

## 🧾 Summary

* **Data Sources:** `encounters`, `procedures`, `payers`
* **Techniques Used:** `YEAR()`, `CASE WHEN`, `TIMESTAMPDIFF()`, window functions (`LAG`, `LEAD`)
* **Goal:** Generate insights into hospital operations, costs, and patient behavior trends.

---

Author: [Jeseena Parveen K]  
Database: hospital_analytics_db  
Purpose: Hospital encounter and trend analysis project

---



