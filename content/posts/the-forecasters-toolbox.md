---
title: "The Forecaster's Toolbox"
date: 2021-12-04T11:55:25+01:00
draft: true
---

Some general tools are useful for many forecasting situations. We can think about some benchmark forecasting methods, ways of making
the forecasting task simpler using transformations and adjustments, methods for checking whether a forecasting method has adequately
utilised the available information and techniques for computing prediction intervals. Each of the tools discussed here will be used
later as we explore a range of forecasting methods.

# Some simple forecasting methods

Some forecasting methods are exremely simple and surprisingly effective. It's a good idea to use the following four forecasting methods
as benchmarks.

**Average method**

Here the forecasts of all future values are equal to the average (or "mean") of the historical data. If we let the historical data be
denoted by $y_{1},\dots,y_{T}$, then we can write the forecasts as:

$$
\hat{y}_  {T+h|T} = \bar{y} = (y_{1}+\dots+y_{T})/T.
$$

The notation $\hat{y}_  {T+h|T}$ is a short-hand for the estimate of $y_{T+h}$ based on the data $y_{1},\dots,y_{T}$.

```r
meanf(y, h)
# y contains the time series
# h is the forecast horizon
```

**Naive method**

For naive forecasts, we simply set all forecasts to be the value of last observation:

$$
\hat{y}_ {T+h|T} = y_{T}.
$$

This method works remarkably well for many economic and financial time series.

```r
naive(y, h)
rwf(y, h) # Equivalent alternative
```

A naive forecast is optimal when the data follow a random walk these are also called `random walk forecasts`.

**Seasonal naive method**

A similar method is useful for highly seasonal data. In this case, we set each forecast to be equal to the last observed value from the same
season of the year (e.g., the same month of the previous year). Formally, the forecast for time $T+h$ is written as:

$$
\hat{y}_ {T+h|T} = y_{T+h-m(k+1)},
$$

where $m$ - the seasonal period and $k$ - the integer part of $(h-1)/m$ (the number of complete years in the forecast period prior to time
$T+h$). This looks more complicated than it really is. For example, with monthly data, the forecast for all future February values is equal to
the last observed February value. With quarterly data, the forecast of all future Q2 values is equal to the last observed Q2 value (where Q2
means the second quarter). Similar rules apply for other months and quarters, and for other seasonal periods.

```r
snaive(y, h)
```

**Drift method**

A variation of the naive method is to allow the forecasts to increase or decrease over time, where the amount of change over time (called `drift`)
is set to be the average change seen in the historical data:

$$
\hat{y}_ {T+h|T} = y_{T} + \frac{h}{T-1}\sum_{t=2}^T (y_{t}-y_{t-1}) = y_{T} + h \left( \frac{y_{T} -y_{1}}{T-1}\right).
$$

This is equivalent to drawing a line between the first and last observations and extrapolating it into the future.

```r
rwf(y, h, drift=TRUE)
```

**Examples**

The graph below presents the first three methods applied to the quarterly beer production data.

```r
# Set training data from 1992 to 2007
beer2 <- window(ausbeer,start=1992,end=c(2007,4))
# Plot some forecasts
autoplot(beer2) +
  autolayer(meanf(beer2, h=11),
            series="Mean", PI=FALSE) +
  autolayer(naive(beer2, h=11),
            series="Naïve", PI=FALSE) +
  autolayer(snaive(beer2, h=11),
            series="Seasonal naïve", PI=FALSE) +
  ggtitle("Forecasts for quarterly beer production\n") +
  xlab("\nYear") + ylab("Megalitres\n") +
  guides(colour=guide_legend(title="Forecast")) +
  theme_minimal() +
  theme(plot.title = element_text(hjust=0.5))
```

![Plot 1](/posts/images/forecasting_example_1.png)

In the second graph, the non-seasonal methods are applied to a series of 200 days of the Google daily closing stock price.

```r
autoplot(goog200) +
  autolayer(meanf(goog200, h=40),
            series="Mean", PI=FALSE) +
  autolayer(rwf(goog200, h=40),
            series="Naïve", PI=FALSE) +
  autolayer(rwf(goog200, drift=TRUE, h=40),
            series="Drift", PI=FALSE) +
  ggtitle("Google stock (daily ending 6 Dec 2013)\n") +
  xlab("\nDay") + ylab("Closing Price (US$)\n") +
  guides(colour=guide_legend(title="Forecast")) +
  theme_minimal() +
  theme(plot.title = element_text(hjust=0.5))
```

![Plot 1](/posts/images/forecasting_example_2.png)

Sometimes one of these simple methods will be the best forecasting method available but in many cases these methods will serve as benchmarks
rather than the method of choice. That is, any forecasting method we develop will be compared to these simple methods to ensure that the new
method is better than these simple alternatives. If not, the new method is not worth considering.

# Transormations and adjustments

Adjusting the historical data can often lead to a simpler forecasting task. The purpose of these adjustments and transformations is to simplify the patterns
in the historical data by removing known sources of variation or by making the pattern more consistent across the whole data set. We present four kinds of adjustements:
calendar adjustments, population adjustments, inflation adjustments and mathematical transformations.

**Calendar adjustments**

Some of the variation seen in seasonal data may be due to simple calendar effects. For example, if you are studying the monthly milk production on a farm, there will be variation between the months
simply because of the different numbers of days in each month, in addition to the seasonal variation across the year.

<TO BE COMPLETED>

**Population adjustments**

Any data that are affected by population changes can be adjusted to give per-capita data. That is, consider the data per person (or per thousand people, or per milion people) rather than the total.
For example, if you are studying the number of hospital beds in a particular region over time, the results are much easier to interpret if you remove the effect of population changes by considering the number
of beds per thousand people. Then you can see whether there have been real increases in the number of beds, or whether the inreases are due entirely to population increases.

**Inflation adjustments**

Data which are affected by the value of money are best adjusted before modelling. For example, the average cost of a new house will have increased over the last few decades due to inflation.
A \\$200,000 house this year is not the same as \\$200,000 house twenty years ago. For this reason, financial time series are usually adjusted so that all values are stated in dollar values from a particular year.
For example, the house price data may be stated in year 2000 dollars.

To make these adjustments, a price index is used. If $z_t$ denotes the price index and $y_t$ denotes the original house price in year $t$, then $x_t = y_t / z_t * z_{2000}$ gives the adjusted house price at year 2000
dollar values. Price indexes are often constructed by government agencies. For consumer goods, a common price index is Consumer Price Index (or CPI).

**Mathematical transformations**

If the data show variation that increases or decreases with the level of the series, then a transformation can be useful. For example, a logarithmic transformation is often useful. If we denote original observations
as $y_{1},\dots,y_{T}$ and the transformed observations as $w_{1},\dots,w_{T}$, then $w_t = \log(y_t)$. Logarithms are useful because they are interpretable: changes in a log value are relative (or percentage) changes
on the original scale. So if log base 10 is used, then an increase of 1 on the log scale corresponds to a multiplication of 10 on the original scale. Another useful feature of log transformations is that they constrain
the forecasts to stay positive on the original scale.

Sometimes other transformations are also used (although they usually are not so interpretable). For example, square roots and cube roots can be used. These are called `power transformations` because they can be written in
the form $w_t = y_t^p$.

A useful family of transformations, that includes both logarithms and power transformations, is the family of `Box-Cox transformations` which depend on the parameter $\lambda$ and are defined as follows:

$$
w_t  =
    \begin{cases}
      \log(y_t) & \text{if $\lambda=0$};  \\\\\\
      (y_t^\lambda-1)/\lambda & \text{otherwise}.
    \end{cases}
$$

The logarithm in a Box-Cox transformation is always a natural logarithm. so if $\lambda = 0$, natural logarithms are used, but if $\lambda\ne0$, a power transformation is used, followed by some simple scaling.

If $\lambda = 1$, then $w_t = y_t - 1$, so the transformed data is shifted downwards but there is no change in the shape of the time series. But for all other values of $\lambda$, the time series will change shape.

The question that comes to mind is what is the good value of $\lambda$?

The one which makes the size of the seasonal variation about the same across the whole series, as that makes the forecasting model simpler.

The `BoxCox.lambda()` function will choose a value of lambda for you.

<TO BE COMPLETED>

The `BoxCox()` command actually implements a slight modification of the Box-Cos transformation, which allows for negative values of $y_t$ when $\lambda > 0$:

$$
w_t  =
    \begin{cases}
      \log(y_t) & \text{if $\lambda=0$};  \\\\\\
      \text{sign}(y_t)(|y_t|^\lambda-1)/\lambda & \text{otherwise}.
    \end{cases}
$$

For positive values of $y_t$, this is the same as the original Box-Cox transformation.

Having chosen a transformation, we need to forecast the transformed data. Then, we need to reverse the transformation (or back-transform) to obtain forecasts on the original scale. The reverse Box-Cox transformation is
given by:

$$
\begin{equation}
\tag{3.1}
  y_{t} =
    \begin{cases}
      \exp(w_{t}) & \text{if $\lambda=0$};\\\\\\
      \text{sign}(\lambda w_t + 1)|\lambda w_t+1|^{1/\lambda} & \text{otherwise}.
    \end{cases}
\end{equation}
$$

**Features of power transformations**

* Choose a simple value of $\lambda$. It makes explanations easier.
* The forecasting results are relatively insensitive to the value of $\lambda$
* Often no transformation is needed.
* Transformations sometimes make little difference to the forecasts but have a large effect on prediction intervals.

**Bias adjustments**

One issue with using mathematical transformations such as Box-Cox transformations is that the back-transformed point forecast will not be the mean of the forecast distribution. In fact, it will usually be the median of
the forecast distribution (assuming that the distribution on the transformed space is symmetric). For many purposes, this is acceptable, but occasionally the mean forecast is required. For example, you may wish to
add up sales forecasts from various regions to form a forecast for the whole country. But medians do not add up, whereas means do.

For a Box-Cox transformation, the back-transformed mean is given by:

$$
\begin{equation}
\tag{3.2}
y_t =
  \begin{cases}
     \exp(w_t)\left[1 + \frac{\sigma_h^2}{2}\right] & \text{if $\lambda=0$;}\\\\\\
     (\lambda w_t+1)^{1/\lambda}\left[1 + \frac{\sigma_h^2(1-\lambda)}{2(\lambda w_t+1)^{2}}\right] & \text{otherwise;}
  \end{cases}
\end{equation}
$$

where $\sigma_h^2$ is the $h$-step variance. The larger the forecast variance, the bigger the difference between the mean and the median.

The difference between the simple back-transformed forecast and the mean is called the `bias`. when we use the mean, rather than the median, we say the point forecasts have been `bias-adjusted`.

To see how much difference this bias-adjustmend makes, consider the following example, where we forecast average annual price of eggs using the drift method with a log transformation ($\lambda = 0$).
The log transformation is useful in this case to ensure the forecasts and the prediction intervals stay positive.

<TO BE COMPLETED>

The blue line on the graph shows the forecast medians while the red line shows the forecast means. Notice how the skewed forecast distribution pulls up the point forecast when we use the bias adjustment.

Bias adjustment is not done by default in the `forecast` package. If you want your forecasts to be means rather than medians, use the argument `biasadj=TRUE` when you select your Box-Cox transformation parameter.
