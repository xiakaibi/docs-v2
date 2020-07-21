---
title: Analyze autocorrelation
description: >
  Use ... to analyze seasonality.
menu:
  v2_0:
    parent: Analyze data
weight: 101
---

Autocorrelation represents the degree of correlation (similarity) between the same time series across successive time intervals. Autocorrelation helps us to identify patterns in a data set, which can help when selecting a forecasting method.

## How to detect autocorrelation

For this exercise, I’m using InfluxDB and the InfluxDB Python CL. I am using available data from the National Oceanic and Atmospheric Administration’s (NOAA) Center for Operational Oceanographic Products and Services. Specifically, I will be looking at the water levels and water temperatures of a river in Santa Monica. 

Dataset: ``curl https://s3.amazonaws.com/noaa.water-database/NOAA_data.txt -o NOAA_data.txt
influx -import -path=NOAA_data.txt -precision=s -database=NOAA_water_database``

This analysis and code is included in a jupyter notebook in this repo. 

First, I import all of my dependencies.

```import pandas as pd
import numpy as np
import matplotlib
import matplotlib.pyplot as plt
from influxdb import InfluxDBClient
from statsmodels.graphics.tsaplots import plot_pacf
from statsmodels.graphics.tsaplots import plot_acf
from scipy.stats import linregress
```

Next I connect to the client, query my water temperature data, and plot it.
  
```
client = InfluxDBClient(host='localhost', port=8086)
h2O = client.query('SELECT mean("degrees") AS "h2O_temp" FROM "NOAA_water_database"."autogen"."h2o_temperature"  GROUP BY time(12h) LIMIT 60')
h2O_points = [p for p in h2O.get_points()]
h2O_df = pd.DataFrame(h2O_points)
h2O_df['time_step'] = range(0,len(h2O_df['time']))
h2O_df.plot(kind='line',x='time_step',y='h2O_temp')
plt.show()
```

Fig 1. H2O temperature vs. timestep 

From looking at the plot above, it’s not obviously apparent whether or not our data will have any autocorrelation. For example, I can’t detect the presence of seasonality, which would yield high autocorrelation. 

I can calculate the autocorrelation with Pandas.Series.autocorr() function which returns the value of the Pearson correlation coefficient. The Pearson correlation coefficient is a measure of the linear correlation between two variables. The Pearson correlation coefficient has a value between -1 and 1, where 0 is no linear correlation, >0 is a  positive correlation, and <0  is a negative correlation. Positive correlation is when two variables change in tandem while a negative correlation coefficient means that the variables change inversely. I compare the data with a lag=1 (or data(t) vs. data(t-1)) and a lag=2 (or data(t) vs. data(t-2). 

```shift_1 = h2O_df['h2O_temp'].autocorr(lag=1)
print(shift_1)
-0.07205847740103073
0.17849760131784975
```
These values are very close to 0, which indicates that there is little to no correlation. However, calculating individual autocorrelation values might not tell the whole story. There might not be any correlation at lag=1, but maybe there is a correlation at lag=15. It’s a good idea to make an autocorrelation plot to compare the values of the autocorrelation function (AFC) against different lag sizes. It’s also important to note that the AFC becomes more unreliable as you increase your lag value. This is because you will compare fewer and fewer observations as you increase the lag value. A general guideline is that the total number of observations (T) should be at least 50, and the greatest lag value (k) should be less than or equal to T/k. Since I have a total of 60 observations, I will only consider the first 20 values of the AFC. 

```plot_acf(h2O_df['h2O_temp'], lags=20)
plt.show()
```

Fig 2. Autocorrelation plot for H2O temperatures

From this plot, we see that values for the ACF are within 95% confidence interval (represented by the solid gray line) for lags > 0, which verifies that our data doesn’t have any autocorrelation. At first, I found this result surprising, because usually the air temperature on one day is highly correlated with the temperature the day before. I assumed the same would be true about water temperature. This result reminded me that streams and rivers don’t have the same system behavior as air. I’m no hydrologist, but I know spring fed streams or snowmelt can often be the same temperature year-round. Perhaps they exhibit a stationary temperature profile day to day where the mean, variance, and autocorrelation are all constant (where autocorrelation is = 0). 

Uncovering seasonality with autocorrelation in time series data

The ACF can also be used to uncover and verify seasonality in time series data. Let’s take a look at the water levels from the same dataset. 

```client = InfluxDBClient(host='localhost', port=8086)
h2O_level = client.query('SELECT "water_level" FROM "NOAA_water_database"."autogen"."h2o_feet" WHERE "location"=\'santa_monica\' AND time >= \'2015-08-22 22:12:00\' AND time <= \'2015-08-28 03:00:00\'')
h2O_level_points = [p for p in h2O_level.get_points()]
h2O_level_df = pd.DataFrame(h2O_level_points)
h2O_level_df['time_step'] = range(0,len(h2O_level_df['time']))
h2O_level_df.plot(kind='line',x='time_step',y='water_level')
plt.show()
```

Fig 3. H2O level vs. timestep 

Just by plotting the data, it’s fairly obvious that seasonality probably exists, evident by the predictable pattern in the data. Let’s verify this assumption by plotting the ACF.

```plot_acf(h2O_level_df['water_level'], lags=400)
plt.show()```

Fig. 4: Autocorrelation plot for H2O levels

From the ACF plot above, we can see that our seasonal period consists of roughly 246 timesteps (where the ACF has the second largest positive peak). While it was easily apparent from plotting time series in Figure 3 that the water level data has seasonality, that isn’t always the case. In Seasonal ARIMA with Python, author Sean Abu shows how he must add a seasonal component to his ARIMA method in order to account for seasonality in his dataset. I appreciated his dataset selection because I can’t detect any autocorrelation in the following figure. It’s a great example of how using ACF can help uncover hidden trends in the data.  

Fig. 5: Monthly Ridership vs. Year. Source: Seasonal ARIMA with Python



Examining trend with autocorrelation in time series data  

In order to take a look at the trend of time series data, we first need to remove the seasonality. Lagged differencing is a simple transformation method that can be used to remove the seasonal component of the series. A lagged difference is defined by: 

difference(t) = observation(t) - observation(t-interval)2,

where interval is the period. To calculate the lagged difference in the water level data, I used the following function:

```def difference(dataset, interval):
    diff = list()
    for i in range(interval, len(dataset)):
        value = dataset[i] - dataset[i - interval]
        diff.append(value)
    return pd.DataFrame(diff, columns = ["water_level_diff"])
h2O_level_diff = difference(h2O_level_df['water_level'], 246)
h2O_level_diff['time_step'] = range(0,len(h2O_level_diff['water_level_diff']))
h2O_level_diff.plot(kind='line',x='time_step',y='water_level_diff')
plt.show()```


Fig. 6: Lagged difference for H2O levels

We can now plot the ACF again. 

```plot_acf(h2O_level_diff['water_level_diff'], lags=300)
plt.show()
```

Fig. 7: ACF of lagged difference for H2O levels

It might seem that we still have seasonality in our lagged difference. However, if we pay attention to the y-axis in Figure 5, we can see that the range is very small and all the values are close to 0. This informs us that we successfully removed the seasonality, but there is a polynomial trend. I used seasonal_decompose to verify this. 

```
from statsmodels.tsa.seasonal import seasonal_decompose
from matplotlib import pyplot
result = seasonal_decompose(h2O['water_level'], model='additive', freq=250)
result.plot()
pyplot.show()
```


Fig. 8.  Seasonal Decomposition of H2O levels

Conclusion

Autocorrelation is important because it can help us uncover patterns in our data, successfully select the best prediction model, correctly evaluate the effectiveness of our model. I hope this introduction to autocorrelation is useful to you.
