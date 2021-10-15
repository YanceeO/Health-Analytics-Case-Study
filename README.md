# Health-Analytics-Case-Study

This case study is a part of the [Serious SQL Couse](www.datawithdanny.com) by [Danny Ma](https://www.linkedin.com/in/datawithdanny/).

## Objective:

The main objective of this study is to help the General Manager of Health Co derive insights from the `health.user_logs` data set.

Basically, we only need to answer the following questions: 
1. How many unique users exist in the logs dataset?
2. How many total measurements do we have per user on average?
3. What about the median number of measurements per user?
4. How many users have 3 or more measurements?
5. How many users have 1,000 or more measurements?
6. How many have logged blood glucose measurements?
7. How many have at least 2 types of measurements?
8. How many have all 3 measures - blood glucose, weight and blood pressure?
9. For users that have blood pressure measurements, what is the median systolic/diastolic blood pressure values?

## Solutions:
To answer the questions, we first inspect the data set. The data set is composed of 6 columns namely `id`, `log_date`, `measure`, `measure_value`, `systolic`, and `diastolic`. This was determined by running the query below 

```sql
SELECT *
FROM health.user_log
```
*Result [column header only]:*

<img width="676" src="https://user-images.githubusercontent.com/91449927/137304855-2afda314-70fa-45b6-b2fa-a5b6a83b84b4.png">

#### Question 1: **How many unique users exist in the logs dataset?**

```sql
SELECT
  COUNT(DISTINCT id)
FROM health.user_logs;
```

*Result:*

<img width="143" alt="Screen Shot 2021-10-14 at 7 10 06 PM" src="https://user-images.githubusercontent.com/91449927/137306498-87bd9f5a-771f-49a4-b790-7b88896a5c82.png">

** *Note: For questions 2-8, a temporary table is created first.* **

I ran a code `DROP TABLE IF EXISTS` to clear out any existing tables. Then, I created a new temporary table using the query below.

```sql
DROP TABLE IF EXISTS user_measure_count
CREATE TEMP TABLE user_measure_count as(
SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) as unique_measures
FROM health.user_logs
GROUP BY id);
```

#### Question 2: **How many total measurements do we have per user on average?**

```sql
select 
  ROUND(AVG(measure_count)) as  average_value
from user_measure_count;
```

*Result:*

<img width="144" src="https://user-images.githubusercontent.com/91449927/137312075-43ff97fa-67e3-4a3b-9526-0c98272a3df6.png">

#### Question 3: **What about the median number of measurements per user?**

```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count;
```

*Result:*

<img width="126" src="https://user-images.githubusercontent.com/91449927/137312275-32273c94-1f52-4c88-9da4-c5b48df01f16.png">

#### Question 4: **How many users have 3 or more measurements?**

```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 3;
```
*Result:*

<img width="80" alt="Screen Shot 2021-10-14 at 7 53 41 PM" src="https://user-images.githubusercontent.com/91449927/137312333-10fc92be-9258-44b1-964c-2582dbbaf8ba.png">

#### Question 5: **How many users have 1,000 or more measurements?**

```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 1000;
```

*Result:*

<img width="80" src="https://user-images.githubusercontent.com/91449927/137312411-fbfdad4b-10a2-4d30-85e1-afb190ed50cf.png">


#### Question 6: **How many have logged blood glucose measurements?**

```sql
SELECT
 COUNT(DISTINCT id)
FROM health.user_logs
WHERE measure = 'blood_glucose';
```
*Result:*

<img width="97" src="https://user-images.githubusercontent.com/91449927/137312499-3c4fcad2-4015-475d-b50d-33dbdca79e54.png">


#### Question 7: **How many have at least 2 types of measurements?**

```sql
SELECT
 COUNT(*)
FROM user_measure_count
WHERE unique_measures >= 2;
```
*Result:*

<img width="85" src="https://user-images.githubusercontent.com/91449927/137312551-342e8968-a1dc-43b2-b5c0-6b0c7d3a242a.png">

#### Question 8: **How many have all 3 measures - blood glucose, weight and blood pressure?**

```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE unique_measures = 3;
```
*Result:*

<img width="86" src="https://user-images.githubusercontent.com/91449927/137312631-2f00af14-d7bf-4a8b-af89-bacdbaa5bed6.png">

#### Question 9: **For users that have blood pressure measurements, what is the median systolic/diastolic blood pressure values?**

```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN  GROUP (ORDER BY systolic) AS median_systolic,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure='blood_pressure';
```
*Result:*

<img width="279" src="https://user-images.githubusercontent.com/91449927/137312768-2786c7e9-49e3-4843-8abc-7389b9338743.png">
