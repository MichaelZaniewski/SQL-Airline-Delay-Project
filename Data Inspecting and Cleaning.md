## Excel - Data Inspecting and Formatting
1) Add ID column primary key but first order by date ASC
2) Format date column
3) Apply filter to all rows, search for blanks
(Notice blanks in tail number)
4) Notice abnormality in first few rows where actual flight time 0
5) filter for actual flight time 0 
6) Last step: take note of how many rows are in the dataset and what the last id number is 

### Creating the table
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

Note: There are NULLs in tail_number so it cannot be NOT NULL for the data to import successfully 
## SQL - Data Importing and Cleaning
