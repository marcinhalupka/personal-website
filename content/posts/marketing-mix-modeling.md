---
title: "Marketing Mix Modeling"
date: 2021-11-02T12:05:26+01:00
draft: false
---

Marketing Mix Model or Media Mix Model (MMM) is used by advertisers to measure how their media spending contributes to sales. As a result,
the future budget can be allocated in a more efficient way. `ROAS` - return on ad spend and `mROAS` - marginal return on ad spend are the key
metrics to look at. High ROAS indicates the channel is efficient, high mROAS means that increasing spend in the channel will yield a high
return based on current spending level.

# Model training procedure

1. Fit a regression model with priors on coefficients using media channels' impressions (or spendings) and 
control variables to predict (explain) sales.

2. Decompose sales to each media channel's contribution. Channel contribution is calculated by comparing original sales and predicted sales
upon removal of the channel.

3. Compute ROAS and mROAS using channel's contribution and spendings.

# Intuition of MMM

* Offline channel's influence is hard to track (example: a customer saw a TV ad and made a purchase at store)

* Media channels' influences are interwined

**Actual Customer Journey: multiple touchpoints** 

A customer:

1. Saw a product on TV
2. Clicked on a display ad
3. Clicked on a paid search ad
4. Made a purchase of $30

![Plot 1](/posts/images/multiple-touchpoints.png)

In this case 3 touchpoints contributed to the conversion and they should all get credits for the conversion.

**What's trackable: last digital touchpoint**

Usually, only the last digital touchpoint can be tracked. In this case SEM will get all credits form this conversion.

![Plot 2](/posts/images/last-digital-touchpoint.png)

So a good attribution model should take into account all the relevant variables leading to conversion.

## Multiplicative MMM

Since media channels work interactively, a multiplicative model structure is adopted:

$$
y = \beta_0 \cdot x_{TV}^{\beta_{TV}} \cdot x_{SEM}^{\beta_{SEM}} \cdot \dots \cdot x_{ctrl}^{\beta_{ctrl}}
$$

Where `$y$` - sales, `$x_i$` - input media variables and `$\beta_i$` - parameters.

Taking log of both sides we get the linear form (log-log model).

$$
\log y = \beta_0  + \beta_{TV} \log {x_{TV}} + \beta_{SEM} \log {x_{SEM}} + \dots + \beta_{ctrl} \log {x_{ctrl}}
$$

Typical constraints on coefficients:

* Media coefficients are non-negative
* If some control variables like discounts, macroeconomy, holidays are expected to have positive impact on sales, their coefficients also should be limited to positive or non-negative values.

## Adstock function

Media effect on sales may lag behind the original exposure and extend several weeks. The carry-over effect is modeled by an adstock function given by the following mathematical formula:

$$\begin{eqnarray}
w_{t-l} &=& D^{{(l - P)}^2}, \\; \text{for each} \\; l  \\;\text{in} \\; \[0, L) \\\\\\
x_t^* &=& \text{Adstock}(x_t, \dots, x_{t-(L-1)}; L, P, D) = \frac{\sum_{l=0}^{L-1} w_{t-l} \cdot x_{t-l}}{\sum_{l=0}^{L-1} w_{t-l}}
\end{eqnarray}$$

* `L` - length of the media effect  
* `P` - peak/delay of the media effect - how many units of time it's lagging behind first exposure  
* `D` - dekay/retention rate of the media channel - concentration of the effect  

The media effect of current weeks is a weighted average of current week and previous (L-1) weeks.

**Adstock example**

Task: compute $x_3^*$ for $L = 4$, $P = 1$ and $D = 0.8$
 
We need to obtain values for $w_i$:

||||||
|-|-|-|-|-|
|$l$|$0$|$1$|$2$|$3$|
|$x$|$x_3$|$x_2$|$x_1$|$x_0$|
|$w$|$0.8^1$|$0.8^0$|$0.8^1$|$0.8^4$|

Then we receive:

$$
x_3^* = \frac{0.8 \cdot x_3 + 1 \cdot x_2 + 0.8 \cdot x_1 + 0.8^4 \cdot x_0}{0.8 + 1 + 0.8 + 0.8^4}
$$

**Adstock with varying decay**  

The larger the decay, the more scattered the effect.

![Plot 3](/posts/images/adstock-varying-decay.png)

**Adstock with varying length**  

As our model assumes geometrical decay when we're moving from the peak, the impact of length can be relatively minor in most cases.
During training the length could be fixed to a period long enought for the media effect to finish - for example 8 weeks.

![Plot 4](/posts/images/adstock-varying-length.png)

**Example code in python:**

```python
import numpy as np
import pandas as pd

def apply_adstock(x, L, P, D):
    '''
    params:
    x: original media variable, array
    L: length
    P: peak, delay in effect
    D: decay, retain rate
    returns:
    array, adstocked media variable
    '''
    x = np.append(np.zeros(L-1), x)
    
    weights = np.zeros(L)
    for l in range(L):
        weight = D**((l-P)**2)
        weights[L-1-l] = weight
    
    adstocked_x = []
    for i in range(L-1, len(x)):
        x_array = x[i-L+1:i+1]
        xi = sum(x_array * weights)/sum(weights)
        adstocked_x.append(xi)
    adstocked_x = np.array(adstocked_x)
    return adstocked_x
```

## Diminishing return

After a certain saturation point, increasing spend will yield diminishing marginal return - the channel will be losing efficiency as you keep overspending on it.

The diminishing return is modeled by Hill function:

$$
\text{Hill} (x; K, S) = \frac{1}{1 + {(x/K)}^{-S}}
$$

`K` - half saturation point  
`S` - slope

**Hill function with varying K and S**

![Plot 5](/posts/images/hill-function.png)

**Example code in python:**

```python
def hill_transform(x, ec, slope):
    return 1 / (1 + (x / ec)**(-slope))
```

# Model specification and implementation

We'll use a sample dataset to go through all the steps of training a model.

## Example dataset

Four years (209 weeks) covered - sales, media impressions and media spendings at weekly level

1. Media variables:
	* Media impressions (prefix = `mdip_`): impressions of 13 media channels: direct mail, insert, newspaper, digital audio, radio, TV, digital video, social media,
	  online display, email, SMS, affiliates, SEM.
	* Media spending (prefix = `mdsp_`): spendings of media channels.
2. Control variables:
	* Macroeconomy (prefix = `me_`): CPI, gas price.
	* Markdown (prefix = `mrkdn_`): markdown/discount.
	* Store count (`st_ct`)
	* Retail Holidays (prefix = `hldy_`): one-hot encoded
	* Seasonality (prefix = `seas_`): month, with Nov and Dec further broken into weeks. One-hot encoded.
3. Sales variables (`sales`)

```python
df = pd.read_csv('data.csv')

# 1. media variables
# media impression
mdip_cols=[col for col in df.columns if 'mdip_' in col]
# media spending
mdsp_cols=[col for col in df.columns if 'mdsp_' in col]

# 2. control variables
# macro economics variables
me_cols = [col for col in df.columns if 'me_' in col]
# store count variables
st_cols = ['st_ct']
# markdown/discount variables
mrkdn_cols = [col for col in df.columns if 'mrkdn_' in col]
# holiday variables
hldy_cols = [col for col in df.columns if 'hldy_' in col]
# seasonality variables
seas_cols = [col for col in df.columns if 'seas_' in col]
base_vars = me_cols+st_cols+mrkdn_cols+va_cols+hldy_cols+seas_cols

# 3. sales variables
sales_cols =['sales']
```

## Model architecture

The model is built in a stacked way. Three models are trained:
1. Control Model
2. Marketing Mix Model
3. Diminishing Return Model

![Plot 6](/posts/images/model-architecture.png)

### Control model

The goal
# Results and marketing budget optimization