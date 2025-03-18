## Excel and SQL Data Quality Checks

## Overview 
The analysis is only considering flights that have successfully taken off. All flights with an `actual_flight_time = 0` OR `taxi_out_time = 0` OR `tail_number IS NULL` have been removed (count of 32074 out of 1.5 million) to consider only the flights that operated under routine conditions. The .CSVs in the repository will still include these flights.

Flights that have been removed include:
- Cancelled flights
- Diverted flights
- Abnormalities attributed to data input

## Excel
Step 1:



## SQL
Step 1:
