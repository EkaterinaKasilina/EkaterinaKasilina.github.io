---
layout: post
title: Discovering Time Series Patterns with Seasonal Plots
description: >
  Exploring time series patterns with seasonal plots
  
image: /assets/img/blog/ts_plots_seasonal.png

sitemap: false
hide_last_modified: true
tags: [data analysis, data visualization]
---


Time series data, by nature, captures trends, seasonality, and anomalies over time, making it essential to use the right visual tools to reveal these insights. 

The full text and code also can be found in my [Medium blog](https://medium.com/@katykas/discovering-time-series-patterns-seasonal-plots-1be96b3fd239)

## Introduction
In this article, I’ll explore the crucial role of effective visualization in uncovering and understanding seasonal patterns in time series data.

So let’s dive in!

## General line plot
For these visualisations I’m using dataset Half-hourly electricity demand for Victoria, Australia. It’s can be accessed from skforecast.datasets library.
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

from skforecast.datasets import fetch_dataset

vic_elec = fetch_dataset(name="vic_electricity")
vic_elec = vic_elec.reset_index()
vic_elec["Date"] = pd.to_datetime(vic_elec["Date"])
vic_elec['Time'] = vic_elec['Time'] + pd.Timedelta(hours=11)

sns.lineplot(x="Date", y="Demand", data=vic_elec)
plt.xticks(rotation=45)
plt.title("Victoria electricity demand by date")

plt.grid(True)
plt.tight_layout()
plt.show()
```
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*NfV78Xh2FEY7Wx1Ht-5L3w.png)

The monthly seasonality is clear: electricity demand peaks during the Australian winter months (June to August), is moderate in spring and autumn, and rises significantly during the summer months.

But what else we can find in this dataset?

## Weekly seasonality
Now we can take a look on the weekly pattern
```python
vic_elec['day_of_week'] = vic_elec['Date'].dt.dayofweek
vic_elec['week'] = vic_elec['Date'].dt.isocalendar().week
vic_elec['day_of_week'] = vic_elec['Time'].dt.dayofweek + (vic_elec['Time'].dt.hour / 24)  # To show fractions of a day
vic_elec['week_str'] = vic_elec['week'].astype(str)

sns.lineplot(x="day_of_week", y="Demand", data=vic_elec, hue="week_str")

# Customize the plot
plt.title('Weekly seasonality plot')
plt.xlabel('Day of the week')
plt.ylabel('Demand')
plt.xticks(ticks=range(7), labels=['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'])
plt.legend([],[], frameon=False)
plt.grid(True)
plt.tight_layout()
plt.show()
```
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*8ZHDnyMR2JPv4nvV4dGLtw.png)

Now, it’s evident that the dataset also exhibits distinct daily and weekly patterns. These recurring cycles highlight how demand fluctuates not just across months but within shorter time frames, with variations that likely align with typical human activity and business operations throughout the week.

## Daily seasonality
How demand changes during a day we can track with daily seasonality plot
```python
vic_elec['hours'] = vic_elec['Time'].dt.hour + vic_elec['Time'].dt.minute / 60
vic_elec['date_from_time'] = vic_elec['Time'].dt.date 

sns.lineplot(x="hours", y="Demand", data=vic_elec, hue="date_from_time")

plt.title('Hourly seasonality plot')
plt.xlabel('Hour')
plt.ylabel('Demand')
plt.legend([],[], frameon=False)
plt.grid(True)
plt.tight_layout()
plt.show()
```
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*UACylb-5RrM84sYzVNYwwQ.png)

This plot provides a clearer view of the patterns observed earlier, where the fluctuations in demand can be attributed to human activities.

## Seasonal subseries plots
It can often be helpful to compare values for the same month across different years to identify trends, seasonal shifts, or long-term changes.
```python
vic_elec['year'] = vic_elec['Date'].dt.year
vic_elec['month'] = vic_elec['Date'].dt.month
vic_elec_monthly = vic_elec.groupby(['month', 'year'])['Demand'].sum().reset_index()
fig, axes = plt.subplots(1, 12, figsize=(12, 4), sharey=True)
axes = axes.flatten()  

# Define month names for labels
months = ['January', 'February', 'March', 'April', 'May', 'June', 
          'July', 'August', 'September', 'October', 'November', 'December']

# Plot each month on a separate subplot
for i, month in enumerate(range(1, 13)):
    ax = axes[i]
    
    # Filter data for the current month
    df_month = vic_elec_monthly[vic_elec_monthly['month'] == month]
    
    # Create the scatter plot for each month, with year as hue
    sns.lineplot(x='year', y='Demand', data=df_month, ax=ax)
    mean_value = df_month['Demand'].mean()
    ax.axhline(mean_value, color='red', linestyle='--')
    
    # Customize the subplot
    ax.set_title(months[month-1])  # Set month name as title
    ax.tick_params(axis='x', rotation=90)

    ax.set_xlabel('Year')
    ax.set_ylabel('Value')
    ax.grid(True)

# Adjust layout and show the plot
plt.tight_layout()
plt.show()
```
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*iiEb37fO9qk_lkBmUHD4FA.png)

## Scatter plots
Often timeseries data also have dependencies from external features. We can study the relationship between demand and temperature by plotting one series against the other.
```python
sns.scatterplot(data=vic_elec, x='Temperature', y='Demand', palette={2014: 'red', 2013: 'blue', 2012: 'green'}, hue = 'year', s = 10)

plt.title('Temperature vs Demand')
plt.xlabel('Temperature')
plt.ylabel('Demand')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*-8YI87roPDiStW6H9PQSrQ.png)

This scatterplot helps us to visualise the relationship between the variables. It is clear that high demand occurs when temperatures are high due to the effect of air-conditioning. But there is also a heating effect, where demand increases for very low temperatures.
  

In this article, I’ve shared just a few examples of how valuable insights can be drawn from a time series dataset. Feel free to share your thoughts on other types of plots that could be useful!

