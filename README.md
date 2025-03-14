# SQL-Airline-Delay-Project
An analysis of American Airlines' departure statistics using PostgreSQL to determine what the airline can optimize to achieve more on-time departures.

## Introduction


## Exploratory Analysis
### How many planes did AA operate each year?
```
SELECT 
	EXTRACT(YEAR FROM date) AS year, 
	COUNT(DISTINCT tail_number) AS Total_Planes,
	COUNT(DISTINCT tail_number) - LAG(COUNT(DISTINCT tail_number),1) OVER () AS difference
FROM delay
GROUP BY EXTRACT(YEAR FROM date)
ORDER BY YEAR
```
### Airports ranked from quickest to slowest average taxi time
```
SELECT origin, ROUND(AVG(taxi_out_time),2) AS avg_taxi,
	RANK() OVER (ORDER BY ROUND(AVG(taxi_out_time),2)) AS RANK
FROM delay
GROUP BY origin
ORDER BY avg_taxi
```
### What was the maximum delay for each type of delay?
```
SELECT 	
		MAX(taxi_out_time) AS max_taxi,
		MAX(carrier_delay) AS max_carrier_dly,
		MAX(weather_delay) AS max_weather_dly,
		MAX(national_aviation_sys_delay) max_atc_dly,
		MAX(security_delay) AS max_security_dly,
		MAX(late_ac_arrival_delay) AS max_late_ac_dly
FROM dela
```

## Findings
### Average Delay by Base



## Reccomendations
- Being that late_ac_arrivals are the #1 cause of delayed flights, AA should focus on
