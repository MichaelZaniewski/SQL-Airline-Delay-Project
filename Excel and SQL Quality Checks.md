## Excel and SQL Data Quality Checks

## Excel - Initial View and Formatting
Step 1: Add an ID column to act as a primary key for the table, allowing the user to uniquely identify each record. 
![excel1](https://github.com/user-attachments/assets/c4e38120-fde3-49b7-8a23-1f6e33712a8d)

Step 2: Format the date column by right clicking on the column and selecting "format cells." SQL only accepts dates in YYYY-MM-DD and will return an error upon import if the data structure does not match. 

Step 3: Apply a filter to all columns and search for blanks (NULLS). `tail_number` is the only column with blanks
![excel3](https://github.com/user-attachments/assets/e01f0f39-b43d-4df7-9306-d64e761e8b2a)

Step 4: Glance over and filter further for abnormalities such as rows where `actual_flt_time` = 0 and `taxi_out_time` = 0. 
![excel4](https://github.com/user-attachments/assets/fa111944-e6a0-40b9-be03-8fb52d1937c7)

Step 5: Repeat steps 1-4 for the second CSV. Take note of the last generated ID row and begin labeling IDs with the next sequential number in the second CSV to ensure all data is uniquely labeled with no duplicates.

## SQL - Importing and Cleaning
Step 1: Generate a table that matches the column names and datatypes of the CSV dataset.
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
 );
```
Note: Since there are NULLs in `tail_number`, the column cannot be constrained as NOT NULL in order for the data to import successfully. To test if all rows have been imported properly, select COUNT(id). Output should be 1,546,452

Step 2: Recall and review NULL cells and data abnormalities
- **NULL `tail_number`:**. Most likely due to a cancellation prior to being assigned an aircraft.
- **A `taxi_out_time` of 0:** Every aircraft has to have a taxi time of greater than zero as no aircraft can push back and takeoff within the same minute. This is a data input error.
- **An `actual_departure` time of 00:00:00 AND `wheels_up_time` of 00:00:00:** Cannot happen for the same reason previously stated.
-  **An `actual_flt_time` of 0:** Could possibly be a diversion that voids flight time.
  
To determine how many abnormal flights are in the dataset, this query was utilized:
```
SELECT COUNT(*)
FROM delay
WHERE  actual_flt_time = 0 
OR tail_number IS NULL
OR taxi_out_time = 0;
```
Output is 32K rows out of 1.5M

For the purposes of this project, only flights that have successfully departed and completed their route will be analyzed. Irregular flights will be removed using the same parameters specified for selecting them.
```
REMOVE FROM delay
WHERE  actual_flt_time = 0 
OR tail_number IS NULL
OR taxi_out_time = 0;
```
**Note:** The .CSVs in the repository will still include these flights.

