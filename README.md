![BannerSkinny](https://github.com/user-attachments/assets/0227b5ef-e000-4477-b5fd-357d38fb2937)

# SQL-Airline-Delay-Project

An analysis of American Airlines' departure statistics to determine what and how the airline can optimize to achieve more on-time departures while balancing an enhanced customer experience.

## Background and Overview
American Airlines, established in 1926, is the worlds largest Airline in terms of quantity of flights and passangers carried daily. This project is focused on American's nine largest hubs: DFW, CLT, MIA, PHX, ORD, PHL, LAX, DCA, and JFK.

The goal is to determine the top causes for departure delays and provide business reccomendations utilizing actionable insights to mitigate delays in accordance with the airline's future endeavors of enhancing the customer experience on-board. 

Insights and reccomendations are provided on the following key areas:
- Analyzing departure count to determine relative presence at bases
- Total delay per flight to extrapolate information such as total delay time per day of week
- Matching type of delay to scheduled departure time to test time-of-day influence
- A/B testing split-prioritiy tactic at DCA
- Increase percentage of maintenance, catering, and cabin cleaner staff in the morning or reposition shift changes to overlap in the morning
  

There are various types of delays, some controllable, some not. Those that are not directly controllable by the airline include weather, ATC, and security issues. Those that are directly controllable include carrier delays (maintinance issues, catering discrepancies, staffing shortages, etc) and late aircraft arrivals. These are the type of delays that will be primarily analyzed.


Excel strategies and SQL queries used to inspect, clean, and perform quality checks can be found HERE (HYPERLINK)
 
Targeted SQL queries used to answer business questions and extract insights can be found here (HYPERLINK)

A long-form written report can be found HERE (HYPERLINK)

## Data Structure and Info
The dataset for this project was gathered from the [Bureau of Transportation Statistics](https://www.transtats.bts.gov/ontime/departures.aspx) for all AA departure metrics in years 2022-2024. Statistics are generated per origin airport. Compiled datasets are uploaded to the repository [here](url).

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


Prior to beginning the analysis, a variety of checks were conducted for quality control and familiarization with column relationships. The SQL queries used to inspect, clean, and perform quality checks can be found HERE (HYPERLINK)

## Executive Summary
### Overview of findings


SQL queries used to extract insights can be found here (HYPERLINK)


- Friday saw the highest total delay times with Tueday being the lowest (Figure 6)
- There are signficantly more departures in the afternoon than the morning (Figure 7)
- 31.19% of flights in the morning were delayed, while 50.44% in the afternoon were delayed, a ratio of 1:2.7 morning to evening late departures (Figures 7 & 8)
- Out of the delayed flights in the morning, most are attributed to carrier delays, while delayed flights in the afternoon are heavily swayed towards late aircraft arrivals (Figure 8)
- Late ac arrivals are the #1 cause for controllable late departues and have the highest maximum delay length of any delay, being the most detrimental adversary to on-time goals (Figure 5 and 3)


- AA should look to implement a split-priority tactic of ensuring on-time departures in the first half of the operational day, while shifting to enhance cabin appearance in the latter half




## RECCOMENDATIONS
- Being that late_ac_arrivals are the #1 cause of delayed flights , AA should focus on
- Focus on on-time departures in the morning so the planes can operate on-time for the later departures then shift priority to customer experience during times when there are the most customers.

- A/B testing using DCA base as the experimental group, (Have a high median carrier delay but significantly less departures per year relative to other bases. Easier to impliment change which standing to benefit the most. RECCOMEND A DAY OF WEEK AS WELL


