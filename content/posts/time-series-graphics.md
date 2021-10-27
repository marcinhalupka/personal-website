---
title: "Time Series Graphics"
date: 2021-10-26T23:36:17+02:00
draft: true
---

The first thing to do in any data analysis is to plot the data. Graphs enable many features of the data to be visualized, including patterns, unusual observations,
changes over time and relationships between the data. Just as the type of data determines what forecasting methods to use, it also determines what graphs are appropriate.

# Ts objects

A time series can be thought as a list of numbers, along with some information about what times those numbers were recorded. This information can be stored as a `ts` object in R.

Let's say we have annual observations for the last few years:

| Year | Obsevation |
| ---- | ---------- |
| 2016 | 123 |
| 2017 | 39 |
| 2018 | 78 |
| 2019 | 52 |
| 2020 | 110 |

We can turn this into a `ts` object using the `ts()` function:

```r
y <- ts(c(123, 39, 78, 52, 110), start=2015)
```

If the data is annual (one observation per year), it's enough to provide the starting year.

For observations that are more frequent than once per year, simply add a `frequency` argument. For example, if we store a monthly data as a numerical vector `z`, then we can
convert it to a `ts` object like this:

```r
z <- ts(y, start=2015, frequency=12)
```

While working with R, `ts` objects are really convenient  for handling time series data.

The `frequency` is the number of observations before the seasonal pattern repeats. You could argue that `period` would be a better name for the argument, but it is what it is.
When you're using the `ts()` function in R, you can use the following values for this parameter:

| Data | Obsevation |
| ---- | ---------- |
| Annual | 1 |
| Quarterly | 4 |
| Monthly | 12 |
| Weekly | 52 |

If you noticed that on average there are not 52 weeks in a year than you're right - we have a leap year every fourth year. But most functions which use `ts` objects require integer frequency.

If the frequency of observations is greater than once per week or different than presented options, you must decide what is the appropriate value. For example, data with daily observations may have a weekly
seasonality (`frequency = 7`), data observed every minute may have an hourly seasonality (`frequency = 60`) or a daily seasonality (`frequency = 24 * 60 = 1440`).

# Time plots

The most obvious graph to start with time series data is a time plot. That is the observations plotted against the time of observation, with consecutive observations joined by straight lines.

Figure below shows the weekly economy passenger load on Ansett Airlines between Australia's two largest cities.

```r
autoplot(melsyd[,"Economy.Class"]) +
  ggtitle("Economy class passengers: Melbourne-Sydney\n") +
  xlab("\nYear") + 
  ylab("Thousands\n") +
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5))
```

![Time plot 1](/posts/images/ts_economy_class_passengers.png)

In R you can use the command `autoplot()` to automatically produce an appropriate plot of whatever you pass to it as a first argument. In this case, a time series was recognized and a time plot was produced.


One simple plot and we immediately see some interesting features:
* There was a period in 1989 when no passengers were carried - caused by an industrial dispute
* There was a period of reduced load in 1992. This was caused by a trial in which some economy class seats were replaced by business class seats.
* A large increase in passenger load occurred in the second half of 1991.
* There are some large dips in load around the start of each year - caused by holiday effects.
* There is a long-term fluctuation in the level of the series which increases during 1987, decreases in 1989 and increases again through 1990 and 1991.
* There are some periods of missing observations.

Any model will need to take all these features into account in order to effectively forecast the passenger load into the future.

A simpler example is presented below.

```r
autoplot(a10) +
  ggtitle("Antidiabetic drug sales\n") +
  ylab("$ million\n") +
  xlab("\nYear") +
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5))
```

![Time plot 2](/posts/images/ts_antidiabetic_drug_sales.png)

The trend is clear and increasing here. You can also see a strong seasonal pattern that increases in size as the level of series increases. The sudden drop at the start of each year is
caused by a government subsidisation scheme that makes it cost-effective for patients to stockpile drugs at the end of calendar year. Forecasting model here would have to capture the seasonal pattern
and the slowly changing trend.

# Time series patterns

We've been using so far words like `trend` and `seasonal` which need to be defined precisely.

**Trend** exists then there is a long-term increase or decrease in the data. It doesn't have to be linear.

**Seasonal** pattern occurs when a time series is affected by seasonal factors like the time of the year or the day of the week. It's always of a fixed and known frequency.

**Cyclic** behaviour (often confused with seasonal behaviour) occurs when the data exhibit rises and falls that are not of a fixed frequency. These happen often due to economic conditions and are related to the *business cycle*

The examples below show different combinations of described components.

![Time plot 3](/posts/images/ts_pattern_examples.png)

1. The monthly housing sales (top left) show strong seasonality within each year and some cyclic behaviour with a period of about 6-10 years. No trend is visible here.

2. The US treasury bill contracts (top right) show results from the Chicage market for 100 consecutive trading days in 1981. Here there is no seasonality but an obvious downward trend. If we had much longer series,
we would see that this is actually part of a long cycle, but when viewed over only 100 days it appears to be a trend.

3. The Australian quarterly electricity production (bottom left) shows a strong increasing trend with strong seasonality. No signs of any cyclic behaviour here.

4. The daily change in the Google closing stock price (bottom right) has no trend, seasonality and cyclic behaviour. It looks like random fluctuations - not very predictable.

# Seasonal plots

Seasonal plot is similar to a time plot but the data is plotted against the individual *seasons*. An example will be presented using antidiabetic drug sales data.

```r
ggseasonplot(a10, year.labels=TRUE, year.labels.left=TRUE) +
  ylab("$ million\n") +
  xlab("\nMonth") +
  ggtitle("Seasonal plot: antidiabetic drug sales\n") +
  theme_minimal() +
  theme(plot.title = element_text(hjust=0.5))
```

![Time plot 4](/posts/images/ts_plot_seasonal.png)

The data is the same as before, we just presented each season separately so we can see the pattern more clearly.

There is a large jump in sales in January each year. Actually, these are probably sales in late December as customers stockpile before the end of calendar year but the sales are not registered
with the government until a week or two later. The graph also shows that there was an unusually small number of sales in March 2008 (for other years there's an increase between February and March). The small amount of sales in
June 2008 may be caused by incomplete counting at the time of data collection.

A useful variation of the seasonal plot uses polar coordinates. Setting `polar=TRUE` makes the time series axis circular.

```r
ggseasonplot(a10, polar=TRUE) +
  ylab("$ million\n") +
  xlab("\nMonth") +
  ggtitle("Seasonal plot: antidiabetic drug sales\n") +
  theme_minimal() +
  theme(plot.title = element_text(hjust=0.5))
```

![Time plot 5](/posts/images/ts_seasonal_plot_polar.png)


# Seasonal subseries plots

Another visualization idea that can be useful for seasonal patterns analysis is to plot the data for each season in separate mini time plots.

```r
ggsubseriesplot(a10) +
  ylab("$ million\n") +
  xlab("\nMonth") +
  ggtitle("Seasonal subseries plot: antidiabetic drug sales\n") +
  theme_minimal() +
  theme(plot.title = element_text(hjust=0.5))
```

![Time plot 6](/posts/images/ts_seasonal_subseries_plot.png)

The horizontal lines present the means for each month. This form of plot shows the changes of seasonality over time. It can be especially useful in identifying changes within particular seasons.

# Scatterplots

We know how to visualize individual time series. It's also useful to explore relationships *between* time series.

the next figure shows two time series: half-hourly electricity demand (in gigawatts) and temperature (in degrees Celsius) for 2014 in Victoria, Australia. the temperatures are for Melbourne, the largest
city in Victoria, while demand values are for the entire state.

```r
date_transform <- function(x) {format(date_decimal(x), "%b %Y")}

autoplot(elecdemand[,c("Demand","Temperature")], facets=TRUE) +
  xlab("\nYear: 2014") +
  scale_x_continuous(labels=date_transform) +
  ylab("") +
  ggtitle("Half-hourly electricity demand: Victoria, Australia\n") +
  theme_minimal() +
  theme(plot.title = element_text(hjust=0.5))
```

![Time plot 7](/posts/images/ts_multiple_facet.png)

We can study the relationship between demand and temperature by plotting one series against the other.

```r
qplot(Temperature,
      Demand,
      data=as.data.frame(elecdemand)) +
  ylab("Demand (GW)\n") +
  xlab("\nTemperature (Celsius)") +
  theme_minimal()
```

![Time plot 8](/posts/images/ts_scatterplot.png)

Now it's clear that high demand occurs when temperatures are high due to effect of air-conditioning. But there's also a heating effect where demand increases for very low temperatures.

## Correlation

It's hard to skip the correlation when talking about the scatterplots. Correlation coefficient measures the strength of the relationsip between two variables.
The mathematical formula for a correlation between variables $x$ and $y$ is given by:

$$
r = \frac{\sum (x_{t} - \bar{x})(y_{t}-\bar{y})}{\sqrt{\sum(x_{t}-\bar{x})^2}\sqrt{\sum(y_{t}-\bar{y})^2}}.
$$

The value of $r$ is normalized - it always lies between -1 and 1 with negative values indicating a negative relationship and positive values indicating a positive relationship. The graphs below
show examples of datasets with varying levels of correlation.

![Time plot 9](/posts/images/ts_correlation_examples.png)

Important remark - correlation coefficient only measures the strength of the *linear* relationship and can sometimes be misleading. Correlation between the electricity demand and temperature data
presented earlier is 0.28 but the non-linear relationsjip is stronger than that.

The plots below present various datasets and for each a correlation coefficient was calculated.

![Time plot 10](/posts/images/ts_correlation_caveats.png)

You can see that in some cases despite correlation coefficient equal to zero, the relationship is clear. That shows how important
is to look at the graphs and not simply rely on correlation values.

## Scatterplot matrices

When there are several potential predictor variables, it's useful to plot each variable against remaining ones. Let's have a look at five time series presenting quarterly visitor numbers for five regions of
New South Wales, Australia.

```r
autoplot(visnights[,1:5], facets=TRUE) +
  ylab("Number of visitor nights each quarter (millions)\n") +
  xlab("") +
  theme_minimal()
```

![Time plot 11](/posts/images/ts_five_series.png)

To see relationship between these five time series, we can plot each time series against remaining ones. These plots can be arranged in a scatterplot matrix.

```r
GGally::ggpairs(as.data.frame(visnights[,1:5]),
                lower=list(continuous=wrap("points", size=0.8))) +
  theme_minimal()
```

![Time plot 12](/posts/images/ts_scatterplot_matrix.png)

For each panel, the variable on the vertical axis is given by the variable name in that row and the variable on the horizontal axis is given by the variable name in that column. There are many options available
to produce different plots within each panel. In the default version the correlation coefficients are shown in the upper right half of the plot, the scatterplots are shown in the lower left half and the
density plots are shown on the diagonal.

This visualization enables a quick view of the relationships between all pairs of variables. In our example, the second column of plots shows there is a strong positive relationship between visitors to the
NSW North Coast and visitors to the NSW South Coast but no detectable relationship between visitors to the NSW North Coast and visitors to the NSW South Inland. We can see some outliers and unusually high quarter for the
NSW Metropolitan region, corresponding to the 2000 Sydney Olympics. This is the most easily seen in the first two plots in the left column, the largest value for NSW Metro is separate from the main cloud of obsevations.

# Lag plots

The next graph displays scatterplots of quarterly Australian beer production where the horizontal axis shows lagged values of the time series. Each graph shows $y_t$ plotted agains $y_{t-k}$ for different values of $k$.

```r
beer2 <- window(ausbeer, start=1992)

gglagplot(beer2, do.lines=FALSE) +
  theme_minimal()
```

![Time plot 13](/posts/images/ts_lagplot.png)

Here the colours indicate the quarter of the variable on the vertical axis. The relationship is strongly positive at lags 4 and 8, reflecting the strong seasonality in data. The negative relationship seen for lags 2 and 6
occurs because peaks (in Q4) are plotted against troughs (in Q2).

the `window()` function used here is useful when extracting a portion of a time series. In this case, we have extracted the data from `ausbeer`, beginning in 1992.

# Autocorrelation

Just as correlation measures the extent of a linear relationship between two variables, autocorrelation measures the linear relationship between lagged values of a time series.

There are several autocorrelation coefficients, corresponding to each panel in the lag plot. For example $r_1$ measures the relationship between $y_t$ and $y_{t-1}$, $r_2$ measures the relationship between
$y_t$ and $y_{t-2}$ and so on.

The value of $r_k$ can be written as:

$$
r_{k} = \frac{\sum\limits_{t=k+1}^T (y_{t}-\bar{y})(y_{t-k}-\bar{y})}
 {\sum\limits_{t=1}^T (y_{t}-\bar{y})^2},
$$

where $T$ is the length of the time series.

The first nine autocorrelation coefficients for the beer production data are given in the following table.

| $r_1$ | $r_2$ | $r_3$ | $r_4$ | $r_5$ | $r_6$ | $r_7$ | $r_8$ | $r_9$ | 
|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|-0.102	|-0.657	|-0.060	|0.869	|-0.089	|-0.635	|-0.054	|0.832	|-0.108 |

These numbers correspond to previous nine scatterplots. The autocorrelation coefficients are plotted to show the `autocorrelaction function` or `ACF`. The plot is known as a `correlogram`.

```r
ggAcf(beer2) +
  ylab("ACF\n") +
  xlab("\nLag") +
  ggtitle("Series: beer2\n") +
  theme_minimal() +
  theme(plot.title = element_text(hjust=0.5))
```

![Time plot 14](/posts/images/ts_acf.png)

In this graph:

* $r_4$ is higher than values for other lags. This is due to the presence of seasonal pattern in the data - peaks tend to be four quarters apart, same as troughs.

* $r_2$ is smaller than values fot other lags - troughs tend to be two quarters behind peaks.

* The dashed blue lines indicate whether the correlations are significantly different from zero.

## Trend and seasonality in ACF plots

Then data have a trend, the autocorrelations for small lags tend do be large and positive because observations nearby in time are also nearby in size. So the ACF of a time series with trend
tend to have positive values that slowly decrease as the lags increase.

When data are seasonal, the autocorrelations will be larger for the seasonal lags (at multiples of the seasonal frequency) than for other lags.

When data are both trended and seasonal, you can see a combination of these effects. The monthly Australian electricity demand series plotted below shows both trend and seasonality. It's followed by its
ACF.

```r
aelec <- window(elec, start=1980)

autoplot(aelec) +
  xlab("\nYear") +
  ylab("GWh\n") +
  ggtitle("Monthly Australian electricity demand 1980-1995\n") +
  theme_minimal() +
  theme(plot.title = element_text(hjust=0.5))
```

![Time plot 15](/posts/images/ts_trend_seasonality.png)

```r
ggAcf(aelec, lag=48) +
  ylab("ACF\n") +
  xlab("\nLag") +
  ggtitle("ACF of monthly Australian electricity demand\n") +
  theme_minimal() +
  theme(plot.title = element_text(hjust=0.5))
```

![Time plot 16](/posts/images/ts_acf_aelec.png)

The slow decrease in the ACF as the lags increase is due to the trend while the *bumps* are due to the seasonality.

# White noise

Time series that show  no autocorrelation are called `whie noise`. The figure below presents an example of a white noise series.

```r
set.seed(42)
y <- ts(rnorm(50))

autoplot(y) +
  ylab("y\n") +
  xlab("\nTime") +
  ggtitle("A white noise time series\n") +
  theme_minimal() +
  theme(plot.title = element_text(hjust=0.5))
```

![Time plot 17](/posts/images/ts_white_noise.png)

```r
ggAcf(y) +
  ylab("ACF\n") +
  xlab("\nLag") +
  ggtitle("ACF for the white noise series\n") +
  theme_minimal() +
  theme(plot.title = element_text(hjust=0.5))
```

![Time plot 18](/posts/images/ts_white_noise_acf.png)

For the white noise we expect each autocorrelation to be close to zero (not equal, as there is some random variation). We expect 95% of the spikes in the ACF to lie within $\pm 2/\sqrt{T}$ where $T$ is the length of the
time series. It's common to plot these bounds on a graph of the ACF (blue dashed lines). If one or more large spikes are outside these bounds or if substantially more than 5% of spikes are outside, then the series is
probably not the white noise.
