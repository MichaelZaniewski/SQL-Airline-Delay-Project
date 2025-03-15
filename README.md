![Banner](https://github.com/user-attachments/assets/0beb29af-f4ad-4711-b705-2bd8072e2e1d)

# SQL-Airline-Delay-Project
An analysis of American Airlines' departure statistics using PostgreSQL to determine what and how the airline can optimize to achieve more on-time departures while balancing an enhanced customer experience.

## INTRODUCTION
This project is focused on American's nine largest hubs: DFW, CLT, MIA, PHX, ORD, PHL, LAX, DCA, and JFK.

The goal is determine the top causes for departure delays and provide a business reccomendation to reduce those delays in accordance with the Airline's future endeavor of enhancing the customer experience on-board. 

There are many types of delays, some controllable, some not. Those that are not directly controllable by the airline include weather, ATC, and security issues. Those that are directly controllable include carrier delays (maintinance issues, catering discrepancies, staffing shortages, etc) and late aircraft arrivals, typically due to carrier delay. These are the type of delays that will be primarily focused on.

## Dataset
The dataset for this project was gathered from the [Bureau of Transportation Statistics](https://www.transtats.bts.gov/ontime/departures.aspx) for all AA departure metrics in years 2022-2024. Statistics are generated per origin airport. Compiled datasets are uploaded to the repository.

Columns in this dataset include:

|        id       |        date      |  departure_delay            |        
|   ------------- |   -------------  | ---------------             |
|   flight_number |    tail_number   |  carrier_delay              |
|      origin     |    destination   |  weather_delay              |
| sched_departure | actual_departure | national_aviation_sys_delay |
| sched_flt_time  | actual_flt_time  | security_delay              |
| wheels_up_time  |   taxi_out_time  | late_ac_arrival_delay       |

The analysis is only considering flights that have successfully taken off. All flights with an actual_flight_time = 0 OR taxi_out_time = 0 OR tail_number IS NULL have been removed (count of 32074 out of 1.5 million) to consider only the flights that operated under routine conditions. The raw datasets in the repository will still include these flights.

Flights that have been removed can include:
- Cancelled flights
- Diverted flights
- Abnormalities attributed to data input


## EXPLORATORY ANALYSIS
### 1) How many planes did AA operate each year?
![Figure1](https://github.com/user-attachments/assets/3593d1cd-d31d-4db6-8076-bd3da590cec7)
```
SELECT 
	EXTRACT(YEAR FROM date) AS year, 
	COUNT(DISTINCT tail_number) AS Total_Planes,
	COUNT(DISTINCT tail_number) - LAG(COUNT(DISTINCT tail_number),1) OVER () AS difference
FROM delay
GROUP BY EXTRACT(YEAR FROM date)
ORDER BY YEAR
```
### 2) How many departures from each base did AA operate in 2023? 
![Figure2](https://github.com/user-attachments/assets/af14b2fe-0a6c-482f-866f-3c39e10ce529)
```
SELECT origin, COUNT(*) as total_departures,
	RANK() OVER (ORDER BY COUNT(*) DESC) as RANK		
FROM    (SELECT origin, id,
	EXTRACT(YEAR FROM date) AS year
	FROM delay)
WHERE year = '2023'
GROUP BY year, origin
ORDER BY total_departures DESC
```
### 3) What was the maximum time for each category of delay?
![Figure3](https://github.com/user-attachments/assets/65c2495e-f904-4094-8542-d18de3cd35ce)
```
SELECT 	
		MAX(taxi_out_time) AS max_taxi,
		MAX(carrier_delay) AS max_carrier_dly,
		MAX(weather_delay) AS max_weather_dly,
		MAX(national_aviation_sys_delay) max_atc_dly,
		MAX(security_delay) AS max_security_dly,
		MAX(late_ac_arrival_delay) AS max_late_ac_dly
FROM delay
```
- Note that taxi_time is not denoted as a delay because every aircraft **has** to have a taxi time, and excess taxi time is always recored under a delay category. Regardless, it is still an interesting metric to pull.

## DIGGING DEEPER
### 4) What were the top 5 most delayed flights?
![Figure4](https://github.com/user-attachments/assets/8375620a-68cf-4851-bbd6-109f7aa779b5)
```
SELECT id, origin, sched_departure, actual_departure, 	
	CASE
		WHEN departure_delay > 0 OR departure_delay > almost_total_delay 
		THEN SUM(almost_total_delay + pos_difference)
		WHEN departure_delay < 0 OR departure_delay < almost_total_delay 
		THEN SUM(almost_total_delay + neg_difference)
		ELSE 0
	END AS total_delay
FROM (SELECT *, COALESCE(SUM(almost_total_delay - departure_delay) FILTER(WHERE departure_delay >0), departure_delay) AS neg_difference,
		COALESCE(ABS(SUM(almost_total_delay - departure_delay) FILTER(WHERE departure_delay >0)), ABS(departure_delay)) AS pos_difference		
	FROM 		(SELECT id, origin, sched_departure, actual_departure, departure_delay, carrier_delay, weather_delay, national_aviation_sys_delay, security_delay, late_ac_arrival_delay,
			SUM (carrier_delay +
				weather_delay +
				national_aviation_sys_delay +
				security_delay +
				late_ac_arrival_delay) AS almost_total_delay
			FROM delay
			GROUP BY id) AS z
	 GROUP BY id, origin, sched_departure, actual_departure, departure_delay, carrier_delay, weather_delay, national_aviation_sys_delay, security_delay, late_ac_arrival_delay, z.almost_total_delay) AS y
GROUP BY id, origin, sched_departure, actual_departure, departure_delay, y.late_ac_arrival_delay, y.almost_total_delay, y.neg_difference, y.pos_difference
ORDER BY total_delay DESC, late_ac_arrival_delay DESC
LIMIT 5
```
### 5) What was the median delay length for each delay category per base for only situations where there was a delay? Bases are ranked from most to least delayed.
![Figure5](https://github.com/user-attachments/assets/00b29b96-4ffd-43d5-9008-7a9e35e774d9)
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
ORDER BY total_mdn_dly DESC)
```
### 6) What day of the week saw the longest delays?
![Figure6](https://github.com/user-attachments/assets/d6b3171c-ed14-4322-92e2-6f7e12be7e66)
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
ORDER BY total_delay_time DESC
```
### 7) Of total flights, how many left in the morning vs afternoon? What percent of morning and afternoon flights departed on time vs late?
![Figure7](https://github.com/user-attachments/assets/7c15d4d4-447b-45d7-a2b9-f61f34a3606f)
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
ORDER BY total_flights ASC
```

### 8) Of the flights that departed late, what percentage were attributed to late_ac delays and carrier delays for morning and afternoon departures?
![Figure8](https://github.com/user-attachments/assets/c46c364e-e4b5-49f8-a643-3b8a79a2f65c)
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
ORDER BY count_delayed_departures ASC
```


## Findings - prerequisit for understanding reccomendations
- 


- Friday saw the highest total delay times with Tueday being the lowest (Figure 6)
- There are signficantly more departures in the afternoon than the morning (Figure 7)
- 31.19% of flights in the morning were delayed, while 50.44% in the afternoon were delayed (Figure 7)
- For every flight that departed late in the morning, there were 2.7 flights departing late in the afternoon (extrapolated from figure 8)
- Out of the delayed flights in the morning, most are attributed to carrier delays, while delayed flights in the afternoon are heavily swayed towards late aircraft arrivals (Figure 8)
- Late ac arrivals are the #1 cause for controllable late departues and have the highest maximum delay length of any delay, being the most detrimental advresary to on-time goals (Figure 5 and 3)

## RECCOMENDATIONS
- Being that late_ac_arrivals are the #1 cause of delayed flights , AA should focus on
- Focus on on-time departures in the morning so the planes can operate on-time for the later departures then shift priority to customer experience during times when there are the most customers.

- A/B testing using PHL base as the experimental group, the base that stands to improve the most from delays (figure 5)


EXTRA 
Are late_ac_delays more prevalent in the second half of the day? YES
Are carrier_delays more prevalent in the first half of the day? YES
If both are true, AA should implement a priority shift tactic depending on the time of day. 


