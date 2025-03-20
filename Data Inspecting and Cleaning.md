## Excel - Data Inspecting and Formatting
1) Add ID column primary key but first order by date ASC
2) Format date column
3) Apply filter to all rows, search for blanks
(Notice blanks in tail number)
4) Notice abnormality in first few rows where actual flight time 0
5) filter for actual flight time 0 
6) Last step: take note of how many rows are in the dataset and what the last id number is 


 
## SQL - Data Importing and Cleaning
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

- See if all data imported properly by selecting MAX(id)

### Cleanup:
Reviewing the data at face value uncovered some abnormalities  
- **NULL tail numbers**. Most likely due to a  cancellation prior to being assigned an aircraft 
- **A `Taxi_out_time of 0`**. Every aircraft has to have a taxi time of greater than zero as no aircraft can push back and takeoff within the same minute. This is a data input error.
- **An `actual_departure` of 00:00:00 AND `wheels_up_time` of 00:00:00.** Cannot happen for the same reason as previously stated. 
- **An `actual_flt_time` of 0**. Could possibly be a diversion that voids the flight time

To determine how many abnormal flights are in the dataset, this query was utilized:
SELECT COUNT(*)
FROM delay
WHERE  actual_flt_time = 0 
OR tail_number IS NULL
OR taxi_out_time = 0;

Abnormalities consist of 32K rows out of 1.5M

For the purposes of this project, only flights that have successfully departed and completed their route will be analyzed. 

Irregular flights will be removed using the same parameters specified for selecting them:

```DELETE FROM delay
WHERE  actual_flt_time = 0 
OR tail_number IS NULL
OR taxi_out_time = 0```
