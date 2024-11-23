---
layout: post
title: Decoding Residuals. The Key to Better Time Series Forecasts
description: >
  How to Perform In-Depth Analysis of Time Series Residuals with ARIMA and SARIMA Models
  
image: /assets/img/blog/resid_analysis.png

sitemap: false
hide_last_modified: true
tags: [time series analysis, time series forecasting]
---

Evaluating a model is a standard step in any training process. When we look at the metrics, we often wonder if the model is good enough or if there’s room for improvement. In time series forecasting, residual analysis can provide valuable insights to help refine the model.
  
Today, I’d like to show how to analyze residuals and extract meaningful insights. To achieve this, we will follow these steps:
  
- Build ARIMA and SARIMA models using a dataset “Monthly expenditure on corticosteroid drugs, Australia”.
- Compare model residuals visually and interpret the plots
- Compare residuals in more formal way
(!) The dataset exhibits a seasonal component, and I intentionally plan to build one weak model and one stronger model to highlight the differences in their residuals.

# Build models
## What is ARIMA and SARIMA?
There are plenty of detailed posts about ARIMA, SARIMA, and how to build them, so I’ll just provide a brief refresher here.

ARIMA(p,d,q) model consists of a few parts:

- Autoregression AR(p): predict the target variable by using a linear combination of its past values(lags), p — number of lags
- Moving average MA(q): use past forecast errors in a regression-like model, q — order of the model, number of errors to look at.
- Differencing I(d): apply differentiation to make a time series stationary, d — number of times a series should be differenced to become stationary.
Combining differencing(I) with autoregression(AR) and a moving average(MA) model results in a non-seasonal ARIMA model.

To capture seasonality, the same parameters are applied at the seasonal level, resulting in the SARIMA(p,d,q)(P,D,Q)m model:

- AR(P) — seasonal AR of order P
- MA(Q) — moving avearage of order Q
- D — order of seasonal differencing
- m — frequency, the number of periods after which you expect a pattern to repeat, for example m = 12 for monthly data suggests a yearly seasonal cycle
## Dataset
For this post I’m using a dataset “Monthly expenditure ($AUD) on corticosteroid drugs that the Australian health system had between 1991 and 2008”

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

from skforecast.datasets import fetch_dataset

import statsmodels.api as sm
from pmdarima import auto_arima

from statsmodels.stats.diagnostic import acorr_ljungbox
from statsmodels.tsa.statespace.sarimax import SARIMAX
from statsmodels.tsa.stattools import adfuller
from statsmodels.tsa.seasonal import STL

sns.set_theme()

data = fetch_dataset(name="h2o")

fig = plt.figure(figsize=(10, 5))
plt.plot(data.index, data.x)
plt.title("Monthly expenditure ($AUD) on corticosteroid drugs")
```
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*sWG0MAyX2vggg7P8uk-qzw.png)
