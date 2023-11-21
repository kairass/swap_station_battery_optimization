# Data Description/ Data Preprocessing

[](https://github.com/kairass/swap_station_battery_optimization/blob/main/Preparing_data.ipynb)

# 1. Scooter Data

| Number of observations | 2,078,866 |
| --- | --- |
| Scooter code | ID number of scooters |
| Longitude | Longitude information for a scooter |
| Latitude | Latitude information  for a scooter |
| ODO | Total travel distance |
| Speed | Speed of a scooter |
| Battery_code | In-use Battery code for a scooter |
| soc | Remaining battery (%) for battery in use for a scooter |
| Battery_temperature | Temperature for a battery in use. |
| Create_time | The time at which the data is created (basically the time variable) |

wow… 2,078,866… We can see that the data has 2 million observations in the Scooter data CSV file, but luckily we only have 9 variables. Let’s check the Station Data

# 2. Station Data

| Number of Observations | 245,889 |
| --- | --- |
| Station_code | ID for a station |
| Device_type | type of battery charging machine that a station has. |
| longitude | Longitude information for a station |
| latitude | latitude information for a station |
| battery_count | number of batteries that a station has |
| batter_0{x}_code | ID for batteries stored in a station (up to 8) |
| battery_0{x}_soc | remaining charges of batteries stored in a station (up to 8) |
| create_time | The time at which the data is created (basically the time variable) |

Good. It has 245,889 observations. 0.2 million is still a lot but compared to 2 million, it looks quite small. 

Since this data contains data on scooters concerning time (Panel data), let’s how many scooters the data keeps track of (# of unique scooters) and how many time frames the data has. 

# 3. Check the number of unique scooters, the total number of stations, and the number of time frames.

![Screenshot 2023-11-21 at 10.05.57 AM.png](Data%20Description%20Data%20Preprocessing%201caa68615e5d494ebb28e4f8b333b0f7/Screenshot_2023-11-21_at_10.05.57_AM.png)

 we are starting to understand the data. One thing to note is that the number of time frames for scooter data and station data **does not match** (904 and 905). If we are going to combine those two data files, we need to do something to match them correctly.  Let’s see how the scooters and stations are distributed on Jakarta at time 0 (That is at 2023-05-06 15:43:37)

# 4. Draw stations and scooters at time 0 (2023-05-06 15:43:37)

![Screenshot 2023-11-21 at 10.27.21 AM.png](Data%20Description%20Data%20Preprocessing%201caa68615e5d494ebb28e4f8b333b0f7/Screenshot_2023-11-21_at_10.27.21_AM.png)

This looks awesome (I love folium). blue dots represent scooter location and blue circles represents station location (we used latitude and longitude location to do so). Note that there exist scooters that are real far away from stations in the data.

# 5. Movement of scooters from time 0 to time 1

![Screenshot 2023-11-21 at 10.33.06 AM.png](Data%20Description%20Data%20Preprocessing%201caa68615e5d494ebb28e4f8b333b0f7/Screenshot_2023-11-21_at_10.33.06_AM.png)

This shows how scooters move from time from time 0 to time 1. Here… we can see some scooters are not moving at all. Since we are interested in scooters that swap batteries (not charging), we don’t need scooters information that does not move (because it is not going to swap the battery in the next period). For this reason, we remove the scooters that does not move. The algorithm is that we eliminate the scooter at time 0 if the location information (latitude and longitude) is the same for time 1. Don’t worry we are not removing the entire scooter information. we only remove the data at the particular time when they do not move (remember it is a panel data).

# 6. Removing scooters that do not move

![Screenshot 2023-11-21 at 10.21.46 AM.png](Data%20Description%20Data%20Preprocessing%201caa68615e5d494ebb28e4f8b333b0f7/Screenshot_2023-11-21_at_10.21.46_AM.png)

You can see that all dots (none-moving scooters) have disappeared. This will leave 362,390 observations (From 2 million to 0.36 million…).  Now we also remove scooters that are too far away. Too far I mean, for some scooters, the distance between their location at t and the closest swap station location is too far for them to physically come to the swap station in the next time frame (which is 10 minuets). Since the maximum speed of the scooter is around 80km, we remove scooters that are 13km away (80/6=1.333). However, to do so, we need to calculate the distance between scooters and stations to find the closest swap station location…. (We will do it.. but not now!, a bit more step later)

For next, let us check how many scooters have actually swapped the batteries.

# 7. Check the scooters that swap batteries

The scooter data has variable called “battery_code” for each time frame for scooter *i*. That is, if a scooter swaps a battery from a station, between time t and t+1, the battery code would be changed for t+1. Using this information, we can identify the scooters that have changed the battery by comparing the battery code data at different times for each scooter *i*.

This leaves 34,150 observations (among 2 million scooters, only 34,150 scooters actually swaps batteries!). 

![Screenshot 2023-11-21 at 11.38.31 AM.png](Data%20Description%20Data%20Preprocessing%201caa68615e5d494ebb28e4f8b333b0f7/Screenshot_2023-11-21_at_11.38.31_AM.png)

The below shows the number of total scooters (top panel) and compares floating scooters vs swapped scooters (floating scooters are those are actually moving at time t and swapped scooters are floating scooters that swap batteries at time t). There are two things to notice. The first is the the number of total scooters are not completely fixed it started with 2299 but later three more scooters are added in the total pool. This is not surprising and we are not going to remove added scooters from total scooters. Another thing to notice is that there are two jumps in the data. This happens because some time frame data are missing and they are all summed in a none desirable way. We are going to remove those data points

![Screenshot 2023-11-21 at 1.58.58 PM.png](Data%20Description%20Data%20Preprocessing%201caa68615e5d494ebb28e4f8b333b0f7/Screenshot_2023-11-21_at_1.58.58_PM.png)

As you can see, we have floating scooters and swapped scooters at different time frame. As we expected, we can see that the number changes with respect to time. The number picks around 19:00 pm and the number plunges around 3:00 am. The below graph shows the remaining percentage of batteries (charged percentage) when swap takes place. That is, it shows the effectiveness of the swapped batteries.

![Screenshot 2023-11-21 at 2.47.00 PM.png](Data%20Description%20Data%20Preprocessing%201caa68615e5d494ebb28e4f8b333b0f7/Screenshot_2023-11-21_at_2.47.00_PM.png)

We can see that the number of swapped scooters and the charged percentage of batteries are inversely related. This result is also quite intuitive because the more scooters are swapping batteries, the less charged batteries scooters will receive.

[](https://github.com/kairass/swap_station_battery_optimization/blob/main/Preparing_data.ipynb)

Now we have 362,390 observation of scooters that moves around Jakarta and 34,150 observation of scooters that actually swaps batteries. 

 For next, let’s see whether we can consider ways to match the station data and the scooter data.

export