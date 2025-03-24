## EXPLORATORY ANALYSIS
### 1) How many planes did AA operate each year?
![Figure1](https://github.com/user-attachments/assets/3593d1cd-d31d-4db6-8076-bd3da590cec7)
- **Functions:** Utilized COUNT() function, DISTINCT clause to aggregate a count of total planes. Then, created a LAG() window function to show changes over time and improve readability
- **Insights Gained:** AA instated 39 planes in 2023 and 17 in 2024 
```
SELECT 
	EXTRACT(YEAR FROM date) AS year, 
	COUNT(DISTINCT tail_number) AS Total_Planes,
	COUNT(DISTINCT tail_number) - LAG(COUNT(DISTINCT tail_number),1) OVER () AS difference
FROM delay
GROUP BY EXTRACT(YEAR FROM date)
ORDER BY YEAR;
```
### 2) How many departures from each base did AA operate in 2023? 
![Figure2](https://github.com/user-attachments/assets/af14b2fe-0a6c-482f-866f-3c39e10ce529)
- **Methodology:** Aggregated a year column using EXTRACT() function, COUNT() to see total_departures, allowing viewers to gain a relative understanding of airline presence per airport. Ranked for easy viewing ability. 
- **Insights Gained:** AA has the largest presence in DFW by far, and the smallest in JFK.  
```
SELECT origin, COUNT(*) as total_departures,
	RANK() OVER (ORDER BY COUNT(*) DESC) as RANK		
FROM    (SELECT origin, id,
	EXTRACT(YEAR FROM date) AS year
	FROM delay)
WHERE year = '2023'
GROUP BY year, origin
ORDER BY total_departures DESC;
```
### 3) What was the maximum time for each category of delay?
![Figure3](https://github.com/user-attachments/assets/65c2495e-f904-4094-8542-d18de3cd35ce)
- **Methodology:** MAX() function to return information on maximum recorded delay to determine how comparing controllable and uncontrollable delays affect departure dependability
- **Insights Gained:** The highest recorded delay is attributed to a late aircraft arrival, followed very closely by carrier delay
- **NOTE:** `taxi_time` is not denoted as a delay. Every aircraft **has** to have a taxi time, and excess taxi time is recored under a delay category. Regardless, it is still an interesting metric to pull
```
SELECT 	
		MAX(taxi_out_time) AS max_taxi,
		MAX(carrier_delay) AS max_carrier_dly,
		MAX(weather_delay) AS max_weather_dly,
		MAX(national_aviation_sys_delay) max_atc_dly,
		MAX(security_delay) AS max_security_dly,
		MAX(late_ac_arrival_delay) AS max_late_ac_dly
FROM delay;
```
## DIGGING DEEPER
### 4) What were the top 5 most delayed flights and their primary cause?
![Image](https://github.com/user-attachments/assets/025a1219-b698-4f95-a34e-c9b2d335cd8a)
- **Methodology:** Utilized subqueries, CASE statements along with SUM(), ABS(), FILTER(), and COALESCE() functions to aggregate a `total_delay` column, accounting for categorically unlisted delays before gate-pushback. Then, applied ORDER BY and LIMIT statements, allowing a viewer to see details of most delayed flights in the dataset
- **Insights Gained:** The highest recorded delay is attributed to a late aircraft arrival, followed very closely by carrier delay
```
SELECT date, origin, sched_departure, actual_departure, departure_delay, carrier_delay, weather_delay, national_aviation_sys_delay, security_delay, late_ac_arrival_delay, 
	CASE
		WHEN departure_delay > 0 OR departure_delay > almost_total_delay 
		THEN SUM(almost_total_delay + pos_difference)
		WHEN departure_delay < 0 OR departure_delay < almost_total_delay 
		THEN SUM(almost_total_delay + neg_difference)
		ELSE 0
	END AS total_delay
FROM (SELECT *, COALESCE(SUM(almost_total_delay - departure_delay) FILTER(WHERE departure_delay >0), departure_delay) AS neg_difference,
		COALESCE(ABS(SUM(almost_total_delay - departure_delay) FILTER(WHERE departure_delay >0)), ABS(departure_delay)) AS pos_difference		
	FROM 		(SELECT id, date, origin, sched_departure, actual_departure, departure_delay, carrier_delay, weather_delay, national_aviation_sys_delay, security_delay, late_ac_arrival_delay,
			SUM (carrier_delay +
				weather_delay +
				national_aviation_sys_delay +
				security_delay +
				late_ac_arrival_delay) AS almost_total_delay
			FROM delay
			GROUP BY id) AS z
	 GROUP BY id, date, origin, sched_departure, actual_departure, departure_delay, carrier_delay, weather_delay, national_aviation_sys_delay, security_delay, late_ac_arrival_delay, z.almost_total_delay) AS y
GROUP BY date, origin, sched_departure, actual_departure, departure_delay, departure_delay, carrier_delay, weather_delay, national_aviation_sys_delay, security_delay, late_ac_arrival_delay, y.late_ac_arrival_delay, y.almost_total_delay, y.neg_difference, y.pos_difference
ORDER BY total_delay DESC, late_ac_arrival_delay DESC
LIMIT 5;
```
### 5) What was the median delay length for each delay category per base for only situations where there was a delay? Bases are ranked from most to least delayed.
![Figure5](https://github.com/user-attachments/assets/00b29b96-4ffd-43d5-9008-7a9e35e774d9)
- **Methodology:** PERCENTILE_CONT() function to aggregate the median while using FILTER to remove zero values, ensure accurate and insightful data is returned
- **Insights Gained:** Most bases, with the exceptions of PHL and LAX, see the highest median delays attributed to late aicraft arrivals
```
SELECT RANK() OVER (ORDER BY total_mdn_dly DESC) AS top_mdn_dlyd_base_rank, * 
FROM (SELECT 	origin, 
		SUM(mdn_dept_dly+mdn_carrier_dly+mdn_weather_dly+mdn_atc_dly+mdn_security_dly+mdn_late_ac_dly) AS total_mdn_dly,
		mdn_dept_dly,
		mdn_carrier_dly,
		mdn_weather_dly,
		mdn_atc_dly,
		mdn_security_dly,
		mdn_late_ac_dly	
	FROM 	(SELECT  origin,
			PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY departure_delay) FILTER(WHERE departure_delay > 0) AS mdn_dept_dly,
			PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY carrier_delay) FILTER(WHERE carrier_delay <> 0) AS mdn_carrier_dly,
			PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY weather_delay) FILTER(WHERE weather_delay <> 0) AS mdn_weather_dly,
			PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY national_aviation_sys_delay) FILTER(WHERE national_aviation_sys_delay <> 0) AS mdn_atc_dly,
			PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY security_delay) FILTER(WHERE security_delay <> 0) AS mdn_security_dly,
			PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY late_ac_arrival_delay) FILTER(WHERE late_ac_arrival_delay <> 0) AS mdn_late_ac_dly
		FROM delay
		GROUP BY origin)
GROUP BY origin, mdn_dept_dly, mdn_carrier_dly, mdn_weather_dly, mdn_atc_dly, mdn_security_dly, mdn_late_ac_dly
ORDER BY total_mdn_dly DESC);
```
### 6) What day of the week saw the longest delays?
![Figure6](https://github.com/user-attachments/assets/d6b3171c-ed14-4322-92e2-6f7e12be7e66)
- **Methodology:** Querying off the aggregated `total_delay` column generated previously, use TO_CHAR to compose `day of week` and GROUP BY it to illustrate which days of the week saw the longest delays
- **Insights Gained:** Friday saw the longest total delays while tuesday saw the least
```
SELECT day_of_week, SUM(total_delay) AS total_delay_time
FROM (SELECT TO_CHAR(date, 'DAY') AS day_of_week,
		CASE
			WHEN departure_delay > 0 OR departure_delay > almost_total_delay 
			THEN SUM(almost_total_delay + pos_difference)
			WHEN departure_delay < 0 OR departure_delay < almost_total_delay 
			THEN SUM(almost_total_delay + neg_difference)
			ELSE 0
		END AS total_delay
	FROM (SELECT *, COALESCE(SUM(almost_total_delay - departure_delay) FILTER(WHERE departure_delay >0), departure_delay) AS neg_difference,
			COALESCE(ABS(SUM(almost_total_delay - departure_delay) FILTER(WHERE departure_delay >0)), ABS(departure_delay)) AS pos_difference					
		FROM (SELECT date, id, origin, departure_delay, carrier_delay, weather_delay, national_aviation_sys_delay, security_delay, late_ac_arrival_delay,
				SUM (carrier_delay +
					weather_delay +
					national_aviation_sys_delay +
					security_delay +
					late_ac_arrival_delay) AS almost_total_delay
			FROM delay
			GROUP BY id) AS z
	GROUP BY date, id, origin, departure_delay, carrier_delay, weather_delay, national_aviation_sys_delay, security_delay, late_ac_arrival_delay, z.almost_total_delay) AS y
	GROUP BY day_of_week, id, origin, departure_delay, carrier_delay, weather_delay, national_aviation_sys_delay, security_delay, late_ac_arrival_delay, y.almost_total_delay, y.neg_difference, y.pos_difference)
GROUP BY day_of_week
ORDER BY total_delay_time DESC;
```
### 7) Of total flights, how many left in the morning vs afternoon? What percent of morning and afternoon flights departed on time vs late?
![Figure7](https://github.com/user-attachments/assets/7c15d4d4-447b-45d7-a2b9-f61f34a3606f)
- **Methodology:** Created a subquery CASE statment to categorize `sched_departure` into 'MORNING' and 'AFTERNOON', then COUNT() flights on-time and late, column division by total, and GROUP BY `time_of_day` to get percentages per time period. Allows for easy data interpretation of departure statistics 
- **Insights Gained:** 30.19% of flights depart late in the morning, quickly increasing to 50.44% by the afternoon
```
SELECT time_of_day, 
		COUNT(*) AS total_flights,
		TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE departure_delay <=0) / CAST(COUNT(*) AS numeric)),2),'999D99%') AS on_time,
		TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE departure_delay >0) / CAST(COUNT(*) AS numeric)),2),'999D99%') AS delayed	
FROM(SELECT *, 
	CASE
		WHEN sched_departure < '12:00:00' THEN 'MORNING'
		WHEN sched_departure >= '12:00:00' THEN 'AFTERNOON'
		ELSE NULL
	END AS time_of_day
FROM delay)
GROUP BY time_of_day 
ORDER BY total_flights ASC;
```
### 8) Of the flights that departed late, what percentage were mostly attributed to late aircraft delays and carrier delays for morning and afternoon departures?
![Figure8](https://github.com/user-attachments/assets/c46c364e-e4b5-49f8-a643-3b8a79a2f65c)
- **Methodology:** Calling back the same subquery, COUNT rows where either `carrier_delay` or `late_ac_arrival_delay` are greater and divide by total delayed flights and use TO_CHAR TO convert to percentage to achieve actionable insights on controllable delays
- **Insights Gained:** Digging deeper into percentages of controllable departure metrics, AA struggles most with carrier delays in the mornings and late aircraft arrivals in the afternoon. These numbers insiuate that early carrier delays play a part in creating late afternoon arrivals as the plane attempts to continues its route through the day.
```
SELECT time_of_day, 
	COUNT(*) FILTER(WHERE departure_delay > 0) AS count_delayed_departures,
	TO_CHAR(100*(COUNT(*) FILTER(WHERE carrier_delay > late_ac_arrival_delay)) / CAST(COUNT(*) FILTER(WHERE departure_delay >0) AS NUMERIC), '999D99%') AS carrier_delayed,
	TO_CHAR(100*(COUNT(*) FILTER(WHERE late_ac_arrival_delay > carrier_delay)) / CAST(COUNT(*) FILTER(WHERE departure_delay >0) AS NUMERIC), '999D99%') AS late_ac_delayed,
	TO_CHAR(100*(COUNT(*) FILTER(WHERE carrier_delay = 0 AND late_ac_arrival_delay = 0 AND departure_delay >0)) / CAST(COUNT(*) FILTER(WHERE departure_delay >0) AS NUMERIC), '999D99%') AS other_delayed
FROM	(SELECT *, 
		CASE
			WHEN sched_departure < '12:00:00' THEN 'MORNING'
			WHEN sched_departure >= '12:00:00' THEN 'AFTERNOON'
			ELSE NULL
		END AS time_of_day
	FROM delay)
GROUP BY time_of_day 
ORDER BY count_delayed_departures ASC;
```

