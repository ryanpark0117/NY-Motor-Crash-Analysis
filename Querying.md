## Motor Crash Analytics 

### Q1: How many distinct counties are represented in the dataset?

```sql
SELECT 
   COUNT(DISTINCT county) num_of_counties
FROM crash_setting;
```
**Answer:**

![Screenshot](/Users/pantsu/Desktop/Screenshot 2023-07-18 at 11.59.08 AM.png)


### Q2: What percentage of injury related accidents are fatal? 

```sql
SELECT 
   ROUND(100 * SUM(CASE WHEN crash_descriptor = 'Fatal Accident' THEN 1 ELSE 0 END) / COUNT(*), 2) as percentage
FROM crash_conditions 
WHERE crash_descriptor != 'Property Damage Accident';
```
**Answer:**

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
