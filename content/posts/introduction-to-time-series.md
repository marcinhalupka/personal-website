---
title: "Introduction to Time Series"
date: 2021-10-26T23:35:17+02:00
draft: false
---

Good forecasts can seem almost magical, while bad forecasts may be dangerous. We can look at three famous predictions about computing:
* *I think there is a world market for maybe five computers.* (Chairman of IBM, 1943)
* *Computers in the future may weigh no more than 1.5 tons.* (Popular Mechanics, 1949)
* *There is no reason anyone would want a computer in their home.* (President, DEC, 1977)

The last of these was made three years before IBM produced the first personal computer. Not surprisingly, you can no longer buy a DEC computer.

Forecasting is obviously a difficult activity and businesses that do it well have a big advantage over those whose forecasts fail.

# What can be forcast?

Forecasting is required in many situations:
* deciding whether to build another power generation plant in the next five years requires forecasts of future demand,
* scheduling staff in a call centre next week requires forecasts of call volumes,
* stocking an inventory requires forecasts of stock requirements.

And these are just examples. Forecasts can be required several years in advance (some capital investments) or only a few minutes beforehand
(telecommunication routing) - no matter the circumstances and time horizon, the benefits are clear.

Some things are easier than others. The time of the sunrise tomorrow morning can be forecast quite precisely. On the other hand, tomorrow's lotto
numbers can't be forecast with any accuracy. The predictability of an event or a quantity depends on several factors, key ones are:
1. How well we understand the factors that contribute to it?
2. How much data is available?
3. Can the forecasts affect the thing we are trying to forecast?

Often in forecasting, a key step is knowing when something can be forecast accurately and when forecasts will be no better than tossing a coin. Good forecasts capture the patterns
and relationships which exist in the historical data, but do not replicate past events that will not occur again. Many people wrongly assume that forecasts are not possible in a
changing environment. Every environment is changing and a good forecasting model captures the way in which things are changing. Forecasts rarely assume that the environment is unchanging - rather that
the way in which the environment is changing will continue into the future. The highly volatile environment will continue to be highly volatile, a business with fluctuating sales will continue to have
fluctuating sales, en economy that has gone through booms and busts will continue to go through booms and busts.


# Forecasting, planning and goals

Forecasting is a common statistical task in business but often done poorly and confused with planning and goals.

**Forecasting** is about predicting the future as accurately as possible, given all the information available.

**Goals** are what you would like to have happen. They should be linked to forecasts and plans but this doesn't always occur.

**Planning** is a response to forecasts and goals - creating a set of actions required to make your forecasts match your goals.

# Determining what to forecast

In the early stages of a forecasting project, it's easy to throw yourself into work and forget about some key questions.

If forecasts are required for items in a manufacturing environment, it's nevessary to ask if forecasts are needed for:
1. Every product lines or some groups?
2. Every sales outlet, outlets grouped by region, or maybe only total sales?
3. Weekly, monthly or annual data?

A quick checklist:
1. What is the forecasting horizon?
2. How frequently are forecasts required?
3. Is it clear what we want to have as an output?
4. Should we segment the data somehow?
5. How the forecasts will be used?
7. Do we have the data? And if we don't can we collect it?

# Forecasting data and methods

The appropriate method depends largely on that data is available.

If there is no data or if available data is not relevant to the forecasts, then `qualitative forecasting` methods must be used. These methods are not purely guesswork,
they are well-developed structured approaches to obtaining good forecasts without using historical data.

`Quantitative forecasting` can be applied when two conditions are satisfied:
1. Numerical information about the past is available
2. It is reasonable to assume that some aspects of the past patterns will continue into the future

Most quantitative forecasting methods use either `time series data` (collected at regular intervals over time) or `cross-sectional data` (collected at single point in time).

Examples of time series data include:
* Daily Tesla stock prices
* Monthly rainfall
* Quarterly sales results for Amazon
* Annual Google profits

Anything that is observed sequentially over time is a time series and usual assupmtion says thay time series are observed at regular intervals of time (e.g. hourly, daily, weekly, monthly, annually).

The simplest time series forecasting methods use only information on the variable to be forecast and make no attempt to discover other factors that affect its behaviour - they will extrapolate trend
and seasonal patterns but ignore all other information such as marketing activities, competitor activity, changes in economic conditions and so on.

If we define $ED_t$ as an electricity demand at time $t$, we may think about a typical `time series model`:

$$
ED_{t+1} = f(ED_{t}, ED_{t-1}, ED_{t-2}, ED_{t-3},\dots) + \text{error}
$$

Prediction is based on past values of a variable but not on external variables which may affect the system. The error term on the right allows for random variation ant the effects of the relevant variables
that are not included in the model.

On the other hand, we can notice that there is a dependency between an electricity demand and other external variables. We could think about an`explanatory model`:

$$\begin{eqnarray} 
  ED_{t+1} = f(\text{temperature, population, time of day}) + error.
\end{eqnarray}$$

The relationship will not be exact as there can always be changes in electricity demand that cannot be accounted for by the predictor variables.

If each of the approach explains some variability visible in our data but different portion than another one, we could combine them together to receive a `mixed model`:

$$\begin{eqnarray} 
  ED_{t+1} = f(ED_{t}, \text{temperature, population, time of day}) + error.
\end{eqnarray}$$

So is the mixed model the best one? Well, it depends. Sometimes the only thing we care about is the accuracy (or other success metric), sometimes about the explainability of the prediction and sometimes about
something totally different. We also have to take into account the resources and data available and the way in which the forecasting model is to be used. In each scenario different model can turn out to be the best.
It's always a trade-off and the better we understand the goals, the better decisions we make.



# The basic steps in a forecasting task

The framework usually involves five basic steps, common for modeling.

**Step 1: Problem definition**
Often the most difficult part - who requires the forecasts, how the forecasting function fits within the organisation requiring the forecasts.
This step needs some time spend on talking to everyone who will be involved in collecting data, maintaining databases and using forecasts for future planning.

**Step 2: Gathering information**
We need the data and accumulated expertise of the people who collect it and use the forecasts. Often it's difficult to obtain enough historical data to be able
to fit a good model. In that case, judgmental forecasting methods can be considered. Occasionally, old data can be less useful due to structural changes in the system being forecast -
we may choose then to use only the most recent data. However good models may handle evolutionary changes in the system so don't throw away good data unnecessarily.

**Step 3: Exploratory analysis**
Always start by graphing the data. Are there consistent patterns? Is there a trend? Is seasonality visible/important? Is there a presence of business cycles? Are there any outliers that need to
be explained by subject matter experts?

**Step 4: Choosing and fitting models**
 Each model is an artificial construct that is based on a set of assumptions. The best model to use depends on the availability of historical data, the strength of relationship between the forecast variable and
 explanatory variables and the way in which the forecasts are to be used. It's common to compare multiple ones with varying underlying mechanisms

**Step 5: Using and evaluating a forecasting model**
Once a model has been selected and trained, it's used to make forecasts. The performance of the model has to be continuously monitored and evaluated.
There may be multiple challenges waiting - from practical ones such as missing values and outliers to organizational like acting on the forecasts.
