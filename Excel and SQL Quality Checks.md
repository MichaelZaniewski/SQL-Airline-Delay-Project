## Excel and SQL Data Quality Checks

## Excel - Initial View and Formatting
Step 1: Add an ID column to act as a primary key for the table, allowing the user to uniquely identify each record. 

Step 2: Format the date column by right clicking on the column and selecting "format cells." SQL only accepts dates in YYYY-MM-DD and will return an error upon import if the data structure does not match. 

Step 3: Apply a filter to all columns and search for blanks (NULLS). `tail_number` is the only column with blanks

Step 4: Glance over and filter further for abnormalities such as rows where `actual_flt_time` = 0 and `taxi_out_time` = 0. 

Step 5: Repeat steps 1-4 for the second CSV. Take note of the last generated ID row and begin labling IDs with the next sequential number in the second CSV to ensure all data is uniquely labeled with no duplicates.

## SQL - Importing and Cleaning
Step 1: Generate a table within PostgreSQL that matches the column names and datatypes of the dataset.
The query used to generate the table is as follows:
```
CREATE TABLE delay(
  id SERIAL PRIMARY KEY,
  date DATE NOT NULL,
  flight_number VARCHAR(20) NOT NULL,
  tail_number VARCHAR(45),
  origin VARCHAR(4) NOT NULL,
  destination VARCHAR(4) NOT NULL,
  sched_departure TIME NOT NULL,
  actual_departure TIME NOT NULL,
  sched_flt_time INTEGER NOT NULL,
  actual_flt_time INTEGER NOT NULL,
  departure_delay INTEGER NOT NULL,
  wheels_up_time TIME NOT NULL,
  taxi_out_time INTEGER NOT NULL,
  carrier_delay INTEGER NOT NULL,
  weather_delay INTEGER NOT NULL,
  national_aviation_sys_delay INT NOT NULL,
  security_delay INTEGER NOT NULL,
  late_ac_arrival_delay INTEGER NOT NULL
 )
```
Note: Since there are NULLs in `tail_number`, the column cannot be constrained as NOT NULL in order for the data to import successfully 

To test if all rows imported properly, select COUNT(id). Output should be 1,546,452






The analysis is only considering flights that have successfully taken off. All flights with an `actual_flight_time = 0` OR `taxi_out_time = 0` OR `tail_number IS NULL` have been removed (count of 32074 out of 1.5 million) to consider only the flights that operated under routine conditions. The .CSVs in the repository will still include these flights.

Flights that have been removed include:
- Cancelled flights
- Diverted flights
- Abnormalities attributed to data input
