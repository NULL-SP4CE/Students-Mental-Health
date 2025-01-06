# Super-market Sales Data Analysis using PostgreSQL

## Overview
This project involves a comprehensive analysis of students mental health data using PostgreSQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, problems and solutions.


## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Students_mental_health](https://www.kaggle.com/datasets/hopesb/student-depression-dataset/code)


## Problems and Solutions 

### (1) Determining which gender of students has the highest depression count and the corresponding average age.

```sql
SELECT total_dcount.gender, Total_student, average_age
FROM (SELECT gender, COUNT(*) AS Total_student
      FROM student_mental_health 
          WHERE depression = 1
          GROUP BY 1) AS total_dcount
INNER JOIN (SELECT gender, ROUND(AVG(age), 2) AS average_age
       FROM student_mental_health
          WHERE depression = 1
          GROUP BY 1 ) AS age_range 
ON total_dcount.gender = age_range.gender
    ORDER BY 2 DESC ;

```


### (2)Determining whether students suicidal thoughts depend on eating habits and sleep duration.

```sql
SELECT sleep_duration, dietary_habits, suicidal_thoughts, COUNT(suicidal_thoughts) AS suicidal_thoughts
FROM student_mental_health
    WHERE NOT dietary_habits = 'Others'
    GROUP BY 1, 2, 3
    ORDER BY 2 DESC, 4 DESC ;
```


### (3) Finding degrees where the average study and work time is more than 7 hours in a day.

```sql
WITH CTE_degree_avg AS (
    SELECT degree,
        ROUND(AVG(work_study_hours), 2) AS average_work_study_hours
    FROM student_mental_health
        GROUP BY 1 
        ORDER BY 2 DESC )
SELECT *
FROM CTE_degree_avg 
    WHERE average_work_study_hours > 7 ;
```


### (4) Finding duplicate on table using window fuction

```sql
SELECT *
FROM (
    SELECT 
        COUNT(1) OVER(PARTITION BY id) AS finding_duplicate
    FROM student_mental_health
        ORDER BY 1 DESC )
WHERE finding_duplicate > 1 :
```


### (5)Finding and ranking all males with financial stress greater than or equal 5(change all NULL value to 0)
    
```sql
SELECT * 
FROM (
    SELECT id, gender, financial_stress,
        Rank() OVER (PARTITION BY gender ORDER BY COALESCE( financial_stress, 0)DESC ) AS ranking_stress
    FROM student_mental_health ) AS ranking_gender
WHERE gender = 'Male' AND financial_stress >= 5 ;

```


### (6)Finding students who sleep less than 5 hours, have unhealthy dietary habits, and also experience suicidal thoughts and financial stress.

```sql
WITH CTE_sleep_duration_average AS (
    SELECT *,
        CASE  
            WHEN sleep_duration = '5-6 hours' THEN 5.30
            WHEN sleep_duration = 'More than 8 hours' THEN 9
            WHEN sleep_duration = 'Less than 5 hours' THEN 4
            WHEN sleep_duration = '7-8 hours' THEN 7.30
        END AS sleep_duration_approx_hrs
    FROM student_mental_health)
SELECT * FROM CTE_sleep_duration_average
    WHERE (sleep_duration_approx_hrs <5 AND dietary_habits = 'Unhealthy') AND (suicidal_thoughts = 'Yes' AND financial_stress >4);

```


### (7) ranking students cgpa above 7.99 and finicial stress more than 4 by genders

```sql
SELECT id, gender, cgpa,
    DENSE_RANK() OVER(PARTITION BY gender ORDER BY cgpa DESC) AS ranking
FROM student_mental_health 
    WHERE financial_stress >4 AND cgpa >= 8.00;
```

### (8) finding students total work, study hours according to degree by gender
	
```sql
SELECT degree, gender,
    SUM(work_study_hours) AS total_work_study_hrs
FROM student_mental_health
    GROUP BY degree, gender
    ORDER BY 1, 3 DESC;

```


## Conclusion

This analysis provides a comprehensive view of Students Mental Health content and can help inform content strategy and decision-making.

