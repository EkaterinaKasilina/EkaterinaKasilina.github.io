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

The full text and code can be found in my [Medium blog](https://medium.com/@katykas/discovering-time-series-patterns-seasonal-plots-1be96b3fd239)

## Introduction
In this article, I’ll explore the crucial role of effective visualization in uncovering and understanding seasonal patterns in time series data.

So let’s dive in!

## General line plot
For these visualisations I’m using dataset Half-hourly electricity demand for Victoria, Australia. It’s can be accessed from skforecast.datasets library.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*NfV78Xh2FEY7Wx1Ht-5L3w.png)

The monthly seasonality is clear: electricity demand peaks during the Australian winter months (June to August), is moderate in spring and autumn, and rises significantly during the summer months.

But what else we can find in this dataset?

## Weekly seasonality
Now we can take a look on the weekly pattern
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*8ZHDnyMR2JPv4nvV4dGLtw.png)

Now, it’s evident that the dataset also exhibits distinct daily and weekly patterns. These recurring cycles highlight how demand fluctuates not just across months but within shorter time frames, with variations that likely align with typical human activity and business operations throughout the week.

## Daily seasonality
How demand changes during a day we can track with daily seasonality plot

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*UACylb-5RrM84sYzVNYwwQ.png)

This plot provides a clearer view of the patterns observed earlier, where the fluctuations in demand can be attributed to human activities.

## Seasonal subseries plots
It can often be helpful to compare values for the same month across different years to identify trends, seasonal shifts, or long-term changes.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*iiEb37fO9qk_lkBmUHD4FA.png)

## Scatter plots
Often timeseries data also have dependencies from external features. We can study the relationship between demand and temperature by plotting one series against the other.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*-8YI87roPDiStW6H9PQSrQ.png)

This scatterplot helps us to visualise the relationship between the variables. It is clear that high demand occurs when temperatures are high due to the effect of air-conditioning. But there is also a heating effect, where demand increases for very low temperatures.
  

In this article, I’ve shared just a few examples of how valuable insights can be drawn from a time series dataset. Feel free to share your thoughts on other types of plots that could be useful!

