![BannerSkinny](https://github.com/user-attachments/assets/0227b5ef-e000-4477-b5fd-357d38fb2937)

# SQL-Airline-Delay-Project

An analysis of American Airlines' departure statistics to determine what and how the airline can optimize to achieve more on-time departures while balancing an enhanced customer experience.

## Background and Overview
American Airlines, established in 1926, is the worlds largest Airline in terms of quantity of flights and passangers carried daily. This project is focused on American's nine largest hubs: DFW, CLT, MIA, PHX, ORD, PHL, LAX, DCA, and JFK.

The goal is to determine the top causes for departure delays and provide business reccomendations utilizing actionable insights to mitigate delays in accordance with the airline's future endeavors of enhancing the customer experience on-board. 

Insights and reccomendations are provided on the following key areas:
- Matching type of delay to scheduled departure time to test time-of-day influence
- Median length of delays per category per hub
- A/B testing on PHL base 

There are various types of delays, some controllable, some not. Those that are not directly controllable by the airline include weather, ATC, and security issues. Those that are directly controllable include carrier delays (maintinance issues, catering discrepancies, staffing shortages, etc) and late aircraft arrivals. These are the type of delays that will be primarily analyzed.


Excel strategies and SQL queries used to inspect, clean, and perform quality checks can be found HERE (HYPERLINK)
 
Targeted SQL queries used to answer business questions and extract insights can be found here (HYPERLINK)

A long-form written report can be found HERE (HYPERLINK)

## DATASET
The dataset for this project was gathered from the [Bureau of Transportation Statistics](https://www.transtats.bts.gov/ontime/departures.aspx) for all AA departure metrics in years 2022-2024. Statistics are generated per origin airport. Compiled datasets are uploaded to the repository [here](url).

Columns in this dataset include: **INCLUDE ERD HERE**

|        id       |        date      |  departure_delay            |        
|   ------------- |   -------------  | ---------------             |
|   flight_number |    tail_number   |  carrier_delay              |
|      origin     |    destination   |  weather_delay              |
| sched_departure | actual_departure | national_aviation_sys_delay |
| sched_flt_time  | actual_flt_time  | security_delay              |
| wheels_up_time  |   taxi_out_time  | late_ac_arrival_delay       |

Prior to beginning the analysis, a variety of checks were conducted for quality control and familiarization with column relationships. The SQL queries used to inspect, clean, and perform quality checks can be found HERE (HYPERLINK)

## Executive Summary
- Friday saw the highest total delay times with Tueday being the lowest (Figure 6)
- There are signficantly more departures in the afternoon than the morning (Figure 7)
- 31.19% of flights in the morning were delayed, while 50.44% in the afternoon were delayed, a ratio of 1:2.7 morning to evening late departures (Figures 7 & 8)
- Out of the delayed flights in the morning, most are attributed to carrier delays, while delayed flights in the afternoon are heavily swayed towards late aircraft arrivals (Figure 8)
- Late ac arrivals are the #1 cause for controllable late departues and have the highest maximum delay length of any delay, being the most detrimental advresary to on-time goals (Figure 5 and 3)


- AA should look to implement a split-priority tactic of ensuring on-time departures in the first half of the operational day, while shifting to enhance cabin appearance in the latter half




## RECCOMENDATIONS
- Being that late_ac_arrivals are the #1 cause of delayed flights , AA should focus on
- Focus on on-time departures in the morning so the planes can operate on-time for the later departures then shift priority to customer experience during times when there are the most customers.

- A/B testing using PHL base as the experimental group, the base that stands to improve the most from delays (figure 5)


EXTRA 
Are late_ac_delays more prevalent in the second half of the day? YES
Are carrier_delays more prevalent in the first half of the day? YES
If both are true, AA should implement a priority shift tactic depending on the time of day. 


