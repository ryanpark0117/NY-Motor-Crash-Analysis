## Motor Crash Analytics 

### Q1: How many distinct counties are represented in the dataset?

```sql
SELECT 
   COUNT(DISTINCT county) num_of_counties
FROM crash_setting;
```
**Answer:**

<img width="145" alt="Screenshot 2023-07-18 at 5 59 56 PM" src="https://github.com/ryanpark0117/NY-Motor-Crash-Analysis/assets/135900740/dc68aa4f-7f96-440f-82ae-3cc146ea6f08">

***

### Q2: What percentage of injury related accidents are fatal? 

```sql
SELECT 
   ROUND(100 * SUM(CASE WHEN crash_descriptor = 'Fatal Accident' THEN 1 ELSE 0 END) / COUNT(*), 2) as percentage
FROM crash_conditions 
WHERE crash_descriptor != 'Property Damage Accident';
```
**Answer:**

<img width="111" alt="Screenshot 2023-07-19 at 12 56 17 AM" src="https://github.com/ryanpark0117/NY-Motor-Crash-Analysis/assets/135900740/970226e9-d728-4b03-85e8-e3f26706db0c">

***

### Q3: What are the average number of vehicles involved in injury vs non-injury related cases? 

```sql
SELECT 
   DISTINCT case_type,
   ROUND(AVG(num_of_vehicles_involved) OVER(PARTITION BY case_type), 2) AS avg_vehicles_involved
FROM (SELECT 
	CASE 
	   WHEN crash_descriptor = 'Property Damage Accident' THEN 'Non-Injury'
        ELSE 'Injury'
           END AS case_type,
           num_of_vehicles_involved
	FROM crash_conditions) c;
```
**Answer:**

<img width="269" alt="Screenshot 2023-07-19 at 12 58 12 AM" src="https://github.com/ryanpark0117/NY-Motor-Crash-Analysis/assets/135900740/ad36df1b-1bb1-45d0-be3a-8fc08cadd7e8">

***

### Q4: What percentage of motor crashes are collision vs non-collision? 

```sql
WITH collision AS (
	SELECT
	   CASE
	      WHEN event_descriptor LIKE '%, Non-Collision' THEN 'Non_Collision'
	   ELSE 'Collision'
	      END AS event_type
	   FROM crash_conditions) 

SELECT 
   event_type,
   ROUND(100 * (COUNT(*) / SUM(COUNT(*)) OVER()), 2) AS percentage
FROM collision 
GROUP BY event_type;
```
**Answer:**

<img width="255" alt="Screenshot 2023-07-19 at 12 59 17 AM" src="https://github.com/ryanpark0117/NY-Motor-Crash-Analysis/assets/135900740/334b59d6-b7f7-42dd-ac5d-885b0d0828c6">
***

### Q5: Which 10 counties accounted for the most amount of accidents? 

First, create variable referencing count of all motor crashes in database

```sql
SET @total_accidents := (SELECT 
      			   COUNT(*) FROM crash_setting);

SELECT 
   county,
   COUNT(*) AS num_of_accidents,
   ROUND(100 * COUNT(*) / @total_accidents, 2) AS percentage
FROM crash_setting
GROUP BY county
ORDER BY num_of_accidents DESC
LIMIT 10;
```
**Answer:**

<img width="350" alt="Screenshot 2023-07-18 at 12 01 13 PM" src="https://github.com/ryanpark0117/images/assets/135900740/e426e67f-3ffa-43f5-ba37-090d2308c676">

***

### Q6: Which county accounted for the most amount of fatal accidents? 

```sql
WITH fatal AS (
	SELECT
	   s.county,
           COUNT(*) AS fatal_accident_count
	FROM crash_setting AS s
        JOIN crash_conditions AS c
	   ON s.ID = c.ID
	WHERE c.crash_descriptor = 'Fatal Accident'
        GROUP BY s.county)

SELECT
   COUNTY,
   fatal_accident_count
FROM fatal
WHERE fatal_accident_count = (SELECT 
				MAX(fatal_accident_count) FROM fatal);
```
**Answer:**              

<img width="240" alt="Screenshot 2023-07-18 at 12 01 32 PM" src="https://github.com/ryanpark0117/images/assets/135900740/bd8074e4-8365-4f51-be87-0b48ef96052d">

***

### Q7: In which hour did the most amount of accidents happen? 

```sql
SELECT
   EXTRACT(HOUR FROM time) AS hour,
   COUNT(*) AS num_of_accidents
FROM crash_setting
GROUP BY hour
HAVING COUNT(*) = (SELECT MAX(accidents) FROM 
                     (SELECT COUNT(*) AS accidents
		        FROM crash_setting 
			GROUP BY EXTRACT(HOUR FROM time)) a);
```
**Answer:**      

<img width="200" alt="Screenshot 2023-07-18 at 12 03 12 PM" src="https://github.com/ryanpark0117/images/assets/135900740/11ee5a43-abba-4208-831e-4ef1d7b77078">

***
                        
### Q8: What is the average time that fatal crashes take place in?

```sql
SELECT
   CONVERT(SEC_TO_TIME(AVG(TIME_TO_SEC(s.time))), CHAR(5)) AS avg_time
   FROM crash_setting AS s
   JOIN crash_conditions AS c
      ON s.ID = c.ID
   WHERE c.crash_descriptor = 'Fatal Accident';
```
**Answer:**

<img width="80" alt="Screenshot 2023-07-18 at 12 03 35 PM" src="https://github.com/ryanpark0117/images/assets/135900740/3aa87eaf-ddba-492d-b5f9-3b4b0ca4ac5d">

***
