![BannerSkinny](https://github.com/user-attachments/assets/0227b5ef-e000-4477-b5fd-357d38fb2937)

# SQL-Airline-Delay-Project

An analysis of American Airlines' departure statistics to determine what and how the airline can implement to achieve more on-time departures while balancing an enhanced customer experience.

## Background and Overview
American Airlines, established in 1926, is the world's largest Airline in terms of quantity of flights and passengers carried daily. This project is focused on American's nine largest hubs: DFW, CLT, MIA, PHX, ORD, PHL, LAX, DCA, and JFK.

The goal is to determine the top causes for departure delays and provide business recommendations utilizing actionable insights to mitigate delays in accordance with the airline's future endeavors of enhancing the customer experience on-board. 

Insights and recommendations are provided on the following key areas:
- Analyzing departure count to determine relative presence at bases
- Total delay per flight to extrapolate information such as total delay time per day of week
- Matching type of delay to scheduled departure time to test time of day influence
- Implementing a minimum-spare-parts list
- A/B testing split-priority tactic depending on the time of day at DCA airport
  
  
Excel strategies and SQL queries used to inspect, clean, and perform quality checks can be found [here](https://github.com/MichaelZaniewski/SQL-Airline-Delay-Project/blob/main/Excel%20and%20SQL%20Quality%20Checks.md)
 
Targeted SQL queries used to answer business questions and extract insights can be found [here](https://github.com/MichaelZaniewski/SQL-Airline-Delay-Project/blob/main/SQL%20Analysis%20Queries.md) 

A long-form written report can be found [here](https://docs.google.com/document/d/1FbklT3wubcDVVe8UjOMpISShDmbYRQFbfnnLsYlv4DM/edit?usp=sharing)

## Data Structure and Info
The dataset for this project was gathered from the [Bureau of Transportation Statistics](https://www.transtats.bts.gov/ontime/departures.aspx) for all AA departure metrics in years 2022-2024. Statistics are generated per origin airport. Compiled datasets can be downloaded from a google drive folder [here](https://drive.google.com/drive/folders/149eeRoGHqdNVELTDj48WXwkjbDq19Gq3?usp=drive_link).

|   column name       |     data type     |     column name     | data type           |   
|  -------------------| ------------------| ------------------- |---------------------|           
|         id          |     integer       |  actual_flt_time    | time                |
|       date          |       date        |  departure_delay    | integer             |
|   flight_number     |character varying  |    wheels_up_time   | time                |
|    tail_number      | character varying |     taxi_out_time   | integer             |
|      origin         |character varying  |     carrier_delay   | integer             |
|   destination       | character varying | weather_delay       |  integer            |
| scheduled_departure |       time        | national_aviation_sys_delay| integer      |
| actual_departure    |       time        | security_delay      | integer             |
| sched_flt_time      |       integer     | late_ac_arrival_delay| integer            |


Prior to beginning the analysis, a variety of checks were conducted for quality control and familiarization with column relationships. The SQL queries used to inspect, clean, and perform quality checks can be found [here](https://github.com/MichaelZaniewski/SQL-Airline-Delay-Project/blob/main/Excel%20and%20SQL%20Quality%20Checks.md).

## Executive Summary
### Overview of Findings
American Airlines' departure dependability struggles most in the afternoon due to rolling delays caused by late aircraft arrivals stemming from carrier delays on morning flights. For years 2022-2024, 31.19% of morning flights were delayed, climbing to 50.44% in the afternoon, a ratio of 1:2.7 morning to evening late departures. Digging deeper, there is a correlation with this data and that of delays caused specifically by controllable metrics. In the mornings, carrier delays are most prevalent while the afternoon points to late aircraft arrivals as the number one cause of late departures. 

### Findings
- American has instated 39 aircraft in 2023 and 17 in 2024 for a current fleet total of 971
- Friday saw the highest total delay times with Tuesday being the lowest
- There are significantly more departures in the afternoon than in the morning
- Of controllable delays, most are due to carrier delays in the mornings while flights in the afternoon are heavily swayed towards late aircraft arrivals
- Late aircraft arrivals are the #1 cause of controllable late departures and have the highest maximum delay length of any delay, being the most detrimental adversary to on-time goals

Targeted SQL queries used to answer business questions and extract insights can be found [here](https://github.com/MichaelZaniewski/SQL-Airline-Delay-Project/blob/main/SQL%20Analysis%20Queries.md) 
  
### Recommendations
- AA should look to implement a split-priority tactic of ensuring on-time departures in the first half of the operational day to prevent rolling delays, while shifting to enhancing cabin appearance and consumer experience in the latter half of the day to impact the majority of customers
- Longer carrier delays are typically due to extensive maintenance issues, therefore, a minimum-spare-parts list should be implemented to ensure each airport has the equipment available to repair an aircraft, eliminating wait time for shipping
- Create an A/B test using DCA base as the experimental group. DCA is a smaller base with less traffic making it easier to implement a testing strategy. It is also one of the bases struggling most with controllable delays, allowing results to be readily apparent

## Assumptions, Limitations, and a Caveat
### Assumptions
- All recorded data is accurate and no additional flights are missing after removing nulls and obvious data input errors
- The most frequent occurrences of carrier_delays are due to maintenance, catering, and cleaning. Other factors such as crew legalities are lesser.

### Limitations
- Maximum available data ends in october 2024, does not include the entire year
- The inability to hone-in on specific aircraft through an entire day to see how each delay affected its subsequent routes and actual departure times
  
### Caveat
- Scheduled departures of major bases are not indicative of out-station schedules. Including out-stations in the analysis would impact percentage of flights departing in the mornings vs afternoons 
