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

Evaluating a model is a standard step in any training process. When we look at the metrics, we often wonder if the model is good enough or if there’s room for improvement. 

In time series forecasting, residual analysis can provide valuable insights to help refine the model.
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


It’s easy to notice that the dataset has seasonality and trend but let’s take a look on STL decomposition to prove it.
```python
stl = STL(data['x'], seasonal=13)
result = stl.fit()

result.plot()
plt.show()
```
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*OrW7qQXgU9jN7rK2MYFZeg.png)


Now let’s split dataset to train and test
```python
test_data = data.iloc[-12:]   # Last 12 rows - 12 months
data = data.iloc[:-12]
```
## Models training
To build an ARIMA model, we typically need to choose p, d, q, and other seasonal parameters by inspecting ACF and PACF plots. However, this time, let’s rely on auto_arima, which can automatically select these parameters. We’ll only specify the differencing order to speed up the auto_arima process.
```python
# Select order of differencing 
def adfuller_test(y):
    res = adfuller(y)

    print(f"ADF Statistic: {res[0]}")
    print(f"p-value: {res[1]}")
    if res[1] < 0.05:
        print("The series is stationary.")
    else:
        print("The series is non-stationary")
       print(" ")

adfuller_test(data['x'])

df_diff = np.diff(data['x'], n=1)
adfuller_test(df_diff)
```

![](https://miro.medium.com/v2/resize:fit:1096/format:webp/1*MwH4pqZm2yKWJzQxA9w7kA.png)

Differencing of order 1 is enough for this dataset to make it stationary.

Now let’s run auto_arima for a weak model — ARIMA. I intentionally put parameters m=1 and seasonal=False to make the model non seasonal.
```python
arima_model = auto_arima(data['x'], 
                      start_p=1, 
                      start_q=1,
                      test='adf',
                      tr=5, max_q=5, # maximum p and q
                      m=1, # frequency of series (if m==1, seasonal is set to FALSE automatically)
                      d=1,
                      seasonal=False, # No Seasonality for standard ARIMA
                      trace=True, 
                      error_action='warn',
                      suppress_warnings=True,
                      stepwise=False)
arima_model.summary()
```

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*pIX5fu_EyA7LnFsl4M_4FQ.png)

Now it’s time for SARIMA
```python
sarima_model = auto_arima(data["x"], start_p=2, start_q=2,
                         test='adf',
                         max_p=5, max_q=5, 
                         m=12, #12 monthly
                         start_P=1, start_Q=1,
                         max_P = 5, max_Q=5, 
                         seasonal=True, #set to seasonal
                         d=1, 
                         D=1,
                         trace=True,
                         error_action='ignore',  
                         suppress_warnings=True, 
                         stepwise=True)
sarima_model.summary()
```

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*gGBTzudfTWBgR3FZ5ID6ag.png)
Look at AIC and BIC for this two models, you already can spot the big difference!
Now we have to fitted models and time for residuals!

# Residuals analysis
Residuals in a time series model are the differences between the observed values and the fitted values after applying the model. They provide insights into how well the model has captured the underlying data.

A good model should resemble white noise with following properties:

1. Residuals should be uncorrelated. If correlations exist among the residuals, it suggests there is remaining information in the residuals that could be used for better forecasts.
2. Residuals should have a mean of zero. If the mean is not zero, the model’s forecasts may be biased.
If a forecasting method does not meet these conditions, it can likely be improved. However, even methods that satisfy these criteria can still be enhanced

Additionally, while not strictly necessary, it’s helpful for the residuals to exhibit two more properties:

3. Residuals should have constant variance, known as “homoscedasticity.”
4. Residuals should be normally distributed.

In order to explore these properties we can visualize residuals using method plot_diagnostics from ARIMA model. This method returns four plots and here what they mean:

**Standardized residuals over time.**

- A simple time series plot of the residuals.
- Helps identify patterns or trends in the residuals that may indicate model inadequacy.
- Ideally, residuals should appear as white noise (randomly scattered around zero with no discernible patterns).

**Histogram plus estimated density.**  
- A smoothed histogram of the residuals, overlaid with a normal distribution curve.
- Indicates whether the residuals are approximately normally distributed.
- Significant deviations suggest a potential bias or issue with the model.

**Normal Q-Q plot, with Normal reference line**
- Compares the quantiles of the residuals against a standard normal distribution.
- Points should align along the diagonal line if residuals are normally distributed.
- Deviations from the line indicate non-normality.

**Correlogram (ACF plot)**  
- Displays the autocorrelation of residuals at various lags.
- Useful for checking the independence of residuals.
- Ideally, all correlations should fall within the 95% confidence bounds, indicating no significant autocorrelation.

Notice: In case if you don’t use ARIMA model, such plots can be reproduced with matplotlib or seaborn.

```python
arima_model.plot_diagnostics(figsize=(10,7))
plt.show()
```

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*QjDac2iv5YGL3qRMfS08Lw.png)

## ARIMA Results:

1. Standardized residuals over time — a pattern can be clearly seen hence it’s not a white noise.
2. Histogram plus estimated density — residuals are not normally distributed
3. Normal Q-Q plot — points strongly deviate from normal distribution
4. Correlogram (ACF plot) — autocorrelaction on lag 10
5. Overall this means that ARIMA model is not well fitted and should be improved.

Now SARIMA! 
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*gakRsgey433ow9UgUuFEjg.png)

## SARIMA Results:
1. Standardized residuals over time — looks like a white noise.
2. Histogram plus estimated density — residuals are very close to normally distributed
3. Normal Q-Q plot — close to a normal distribution
4. Correlogram (ACF plot) — no autocorrelaction
5. Overall this means that SARIMA model is well fitted (again, it doesn’t mean that model can’t be improved)

# Formal test, Ljung-Box test
In addition to looking at the ACF plot, we can also do a more formal test for autocorrelation by considering a whole set of residual values as a group, rather than treating each one separately.

The Ljung-Box test is a statistical test used to determine whether a set of residuals from a time series model exhibits significant autocorrelation. It is commonly used in time series analysis to check the adequacy of a fitted model, such as ARIMA or SARIMA, by testing whether the residuals are independent.

*Hypotheses.*

Null Hypothesis (H₀): The residuals are independently distributed (no autocorrelation).
Alternative Hypothesis (H₁): The residuals exhibit autocorrelation.

Rejecting the null hypothesis implies that the model’s residuals are not independent, suggesting the model may need further refinement.

## How the Ljung-Box test works

*Inputs:*  
- A time series of residuals from a model.
- A lag 𝑘, up to which autocorrelation is tested.
- Significance level (commonly 0.05 or 0.01).
*Test Statistic:*  
The Ljung-Box test uses the following statistic:
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*dH9hX1gh1N59hLDX43guAA.png)

𝑛 — Number of observations in the residuals. 𝜌𝑘 — Sample autocorrelation at lag 𝑘. 𝑚 — Number of lags being tested.

Distribution: The test statistic 𝑄 is approximately chi-squared (𝜒2) distributed with 𝑚 degrees of freedom.

P-value: The p-value indicates the probability of observing the data if the null hypothesis is true. If 𝑝 -value < significance level, reject H₀

### Important:

Choosing the number of lags for the Ljung-Box test (or other time series diagnostics) is crucial to balance between detecting autocorrelation patterns and maintaining statistical power. Here’s a few recomendation how to choose an appropriate number of lags:
1. Length of the Time Series:
The number of lags should generally be much smaller than the number of observations in the series. Rule of thumb: Use √𝑁, where 𝑁 is the number of observations.
2. Frequency of the Data:
Consider the natural periodicity of the data. For monthly data, test lags around 12 (one year). For daily data with weekly seasonality, test lags around 7.
3. Domain Knowledge:
Use domain-specific insights to focus on meaningful lag ranges. E.g., in sales, consider lags that align with known business cycles or seasonality.

```python
def ljung_box_test_print(resid):
    ljung_box_test = acorr_ljungbox(resid, lags=[12], return_df=False)
    if ljung_box_test.lb_pvalue.values < 0.05:
        print("Reject H0. Autocorrelation in the residuals! Model can be improved")
    else:
        print("Great! Failed to Reject 𝐻0. Resudials are a white noise")
    print()
    print(ljung_box_test)

# Null Hypothesis (H₀): The residuals are independently distributed (no autocorrelation)
print("ARIMA model")
ljung_box_test_print(arima_model.resid())
```
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*DwtYCkOrHyjCrihP2FhHCg.png)

```python
print("SARIMA model")
ljung_box_test_print(sarima_model.resid())
```
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*w-k1mz2JVmoPdBBoAyqEVg.png)

So, the formal test also proves that SARIMA model is better fitted than ARIMA!

We have finished residuals analysis. Now lets make a few final things: plot forecasted values against observed and calculate MAPE

## Observed vs forecasted values and MAPE

```python
arima_preds = arima_model.predict(n_periods=12, return_conf_int=False)
sarima_preds = sarima_model.predict(n_periods=12, return_conf_int=False)

arima_preds = arima_preds.reset_index()
sarima_preds = sarima_preds.reset_index()

arima_preds.columns = ['date', 'x']
sarima_preds.columns = ['date', 'x']

arima_in_sample = arima_model.predict_in_sample(start=1, end=data["x"].shape[0])
sarima_in_sample = sarima_model.predict_in_sample(start=1, end=data["x"].shape[0])

arima_in_sample = arima_in_sample.reset_index()
arima_in_sample.columns = ['date', 'x']

sarima_in_sample = sarima_in_sample.reset_index()
sarima_in_sample.columns = ['date', 'x']

data = data.reset_index()
data.columns = ['date', 'x']

test_data = test_data.reset_index()
test_data.columns = ['date', 'x']

plt.figure(figsize=(12, 6))
plt.plot(data['date'], data['x'], label='Observed', color='blue', alpha = 0.7)
plt.plot(test_data['date'], test_data['x'], label='Test observed', color='blue', alpha = 0.7)

plt.plot(arima_in_sample['date'], arima_in_sample['x'], label='In-sample ARIMA', color='green', alpha = 0.7)
plt.plot(sarima_in_sample['date'], sarima_in_sample['x'], label='In-sample SARIMA', color='red', alpha = 0.7)

plt.plot(arima_preds.date, arima_preds['x'], label='Out-sample ARIMA', color='green', linestyle = '--')
plt.plot(sarima_preds.date, sarima_preds['x'], label='Out-sample SARIMA', color='red', linestyle = '--')

plt.legend()
plt.title('Fitted vs Observed values') 
```
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ea8G1R-5r0Qsvf3uIaJIdA.png)

```python
mean_absolute_percentage_error(y_true, y_pred):
    return np.mean(np.abs((y_true - y_pred) / y_true)) * 100

print("MAPE ARIMA on train", mean_absolute_percentage_error(data['x'], arima_in_sample['x']))
print("MAPE SARIMA on train ", mean_absolute_percentage_error(data['x'], sarima_in_sample['x']))
print()
print("MAPE ARIMA on test", mean_absolute_percentage_error(test_data['x'], arima_preds['x']))
print("MAPE SARIMA on test ", mean_absolute_percentage_error(test_data['x'], sarima_preds['x']))
```
![](https://miro.medium.com/v2/resize:fit:1120/format:webp/1*X26pTugHlwotQFYfqsg40A.png)

Thanks for reading!

Sources:

https://otexts.com/fpp3/diagnostics.html   
https://learn.saylor.org/mod/book/tool/print/index.php?id=55332&chapterid=40915

