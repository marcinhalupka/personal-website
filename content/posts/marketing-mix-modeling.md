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

The goal is to predict base sales (`X_ctrl`) as an input variable to MMM - this represents the baseline sales trend without any marketing activities.

![Plot 6](/posts/images/control-model.png)

We divide variables into two groups: $X_1$ and $X_2$.

$X_1$ - control variables positively related to sales  
$X_2$ - control variables that may have either positive or negative impact on sales.  
$y$ - target variable, sales

The variables are centralized.

**Priors**  

|Parameter|Prior|Description|
|---------|-----|-----------|
|$\beta_1$|half normal (0, 5)|Regression coefficients for control variables positively related to sales|
|$\beta_2$|normal (0, 5)|Regression coefficients for control variables positively or negatively related to sales|
|$\alpha$|[0, min(y)]|Intercept|
|Residual| inverse gamma (0.05, 0.05*0.01)|Residual variance|

**Python code**

```python
import pystan
import os
os.environ['CC'] = 'gcc-10'
os.environ['CXX'] = 'g++-10'

# mean-centralize: sales, numeric base_vars
df_ctrl, sc_ctrl = mean_center_trandform(df, ['sales']+me_cols+st_cols+mrkdn_cols)
df_ctrl = pd.concat([df_ctrl, df[hldy_cols+seas_cols]], axis=1)

# variables positively related to sales: macro economy, store count, markdown, holiday
pos_vars = [col for col in base_vars if col not in seas_cols]
X1 = df_ctrl[pos_vars].values

# variables may have either positive or negtive impact on sales: seasonality
pn_vars = seas_cols
X2 = df_ctrl[pn_vars].values

ctrl_data = {
    'N': len(df_ctrl),
    'K1': len(pos_vars), 
    'K2': len(pn_vars), 
    'X1': X1,
    'X2': X2, 
    'y': df_ctrl['sales'].values,
    'max_intercept': min(df_ctrl['sales'])
}

ctrl_code1 = '''
data {
  int N; // number of observations
  int K1; // number of positive predictors
  int K2; // number of positive/negative predictors
  real max_intercept; // restrict the intercept to be less than the minimum y
  matrix[N, K1] X1;
  matrix[N, K2] X2;
  vector[N] y; 
}

parameters {
  vector<lower=0>[K1] beta1; // regression coefficients for X1 (positive)
  vector[K2] beta2; // regression coefficients for X2
  real<lower=0, upper=max_intercept> alpha; // intercept
  real<lower=0> noise_var; // residual variance
}

model {
  // Define the priors
  beta1 ~ normal(0, 1); 
  beta2 ~ normal(0, 1); 
  noise_var ~ inv_gamma(0.05, 0.05 * 0.01);
  // The likelihood
  y ~ normal(X1*beta1 + X2*beta2 + alpha, sqrt(noise_var));
}
'''

sm1 = pystan.StanModel(model_code=ctrl_code1, verbose=True)
fit1 = sm1.sampling(data=ctrl_data, iter=2000, chains=4)
fit1_result = fit1.extract()
```

### Marketing Mix Model

Our goal is to:

* Find appropriate adstock parameters for media channels
* Decompose sales to media channels' contribution and baseline (non-marketing) contribution

![Plot 7](/posts/images/marketing-mix-model.png)

`L` - length of media impact  
`P` - peak of media impact  
`D` - decay of media impact  
`X` - adstocked media impression variables and base sales  
`y` - target variable, sales  

Variables are centralized.

**Priors**

|Parameter|Prior|Description|
|---------|-----|-----------|
|$L$|8|Length of media impact|
|$P$|uniform(0, max_lag/2)|Peak/delay of media impact|
|$D$|beta(3, 3)|Decay rate of media channel|
|$\beta$|half normal(0, 1)|Regression coefficients for media channels and control|
|$\tau$|normal(0, 5)|Intercept|
|Residual|inverse gamma(0.05, 0.05*0.01)|Residual variance|

**Python code**

```python
df_mmm, sc_mmm = mean_log1p_trandform(df, ['sales', 'base_sales'])
mu_mdip = df[mdip_cols].apply(np.mean, axis=0).values
max_lag = 8
num_media = len(mdip_cols)
# padding zero * (max_lag-1) rows
X_media = np.concatenate((np.zeros((max_lag-1, num_media)), df[mdip_cols].values), axis=0)
X_ctrl = df_mmm['base_sales'].values.reshape(len(df),1)
model_data2 = {
    'N': len(df),
    'max_lag': max_lag, 
    'num_media': num_media,
    'X_media': X_media, 
    'mu_mdip': mu_mdip,
    'num_ctrl': X_ctrl.shape[1],
    'X_ctrl': X_ctrl, 
    'y': df_mmm['sales'].values
}

model_code2 = '''
functions {
  // the adstock transformation with a vector of weights
  real Adstock(vector t, row_vector weights) {
    return dot_product(t, weights) / sum(weights);
  }
}
data {
  // the total number of observations
  int<lower=1> N;
  // the vector of sales
  real y[N];
  // the maximum duration of lag effect, in weeks
  int<lower=1> max_lag;
  // the number of media channels
  int<lower=1> num_media;
  // matrix of media variables
  matrix[N+max_lag-1, num_media] X_media;
  // vector of media variables' mean
  real mu_mdip[num_media];
  // the number of other control variables
  int<lower=1> num_ctrl;
  // a matrix of control variables
  matrix[N, num_ctrl] X_ctrl;
}
parameters {
  // residual variance
  real<lower=0> noise_var;
  // the intercept
  real tau;
  // the coefficients for media variables and base sales
  vector<lower=0>[num_media+num_ctrl] beta;
  // the decay and peak parameter for the adstock transformation of
  // each media
  vector<lower=0,upper=1>[num_media] decay;
  vector<lower=0,upper=ceil(max_lag/2)>[num_media] peak;
}
transformed parameters {
  // the cumulative media effect after adstock
  real cum_effect;
  // matrix of media variables after adstock
  matrix[N, num_media] X_media_adstocked;
  // matrix of all predictors
  matrix[N, num_media+num_ctrl] X;
  
  // adstock, mean-center, log1p transformation
  row_vector[max_lag] lag_weights;
  for (nn in 1:N) {
    for (media in 1 : num_media) {
      for (lag in 1 : max_lag) {
        lag_weights[max_lag-lag+1] <- pow(decay[media], (lag - 1 - peak[media]) ^ 2);
      }
     cum_effect <- Adstock(sub_col(X_media, nn, media, max_lag), lag_weights);
     X_media_adstocked[nn, media] <- log1p(cum_effect/mu_mdip[media]);
    }
  X <- append_col(X_media_adstocked, X_ctrl);
  } 
}
model {
  decay ~ beta(3,3);
  peak ~ uniform(0, ceil(max_lag/2));
  tau ~ normal(0, 5);
  for (i in 1 : num_media+num_ctrl) {
    beta[i] ~ normal(0, 1);
  }
  noise_var ~ inv_gamma(0.05, 0.05 * 0.01);
  y ~ normal(tau + X * beta, sqrt(noise_var));
}
'''

sm2 = pystan.StanModel(model_code=model_code2, verbose=True)
fit2 = sm2.sampling(data=model_data2, iter=1000, chains=3)
fit2_result = fit2.extract()
```

**Distribution of Media Coefficients**

Red line - mean, green line - median.

![Plot 7](/posts/images/distribution-media-coefficients.png)

**Decompose sales to media channels' contribution**

Each media channel's contribution = total sales - sales upon removal of the channel. In the previous model fitting step, parameters of the log-log model have been found:

![Plot 7](/posts/images/log-log-model.png)

Now we can plug them into the multiplicative mode:

![Plot 7](/posts/images/multiplicative-model.png)

Baseline = $1000$

$x_1 = 100, \\; \beta_1 = 0.05, \\; x_1^{\beta_1} = 1.26$  
$x_2 = 500, \\; \beta_2 = 0.03, \\; x_2^{\beta_2} = 1.20$  
$y = 1000 \cdot 1.26 \cdot 1.20 = 1512$  
Contribution of media 1: $C_1 = 1512 - 1512/1.26 = 312$  
Contribution of media 1: $C_1 = 1512 - 1512/1.20 = 252$  
Sum of media contribution is slightly different from total:  
$V_{pred} = C_1 + C_2 = 564, \\; V_{true} = 512$  
So we can scale each factor's contribution by removing the delta volume proportionally:  
$C_1^\prime = 312 - (564-512) \cdot 312/564 = 283.23$  
$C_2^\prime = 252 - (564-512) \cdot 252/564 = 228.77$  
Total colume: $V_{total}^\prime = 1000 + 283.23 + 228.77 = 1512$

**Python code**

```python
# decompose sales to media contribution
mc_df = mmm_decompose_media_contrib(mmm, df, y_true=df['sales_ln'])
adstock_params = mmm['adstock_params']
mc_pct, mc_pct2 = calc_media_contrib_pct(mc_df, period=52)  
```

**Adstock parameters**

```python
{'dm': {'L': 8, 'P': 0.8147057071636012, 'D': 0.5048365638721349},
 'inst': {'L': 8, 'P': 0.6339321363933637, 'D': 0.40532404247040194},
 'nsp': {'L': 8, 'P': 1.1076944292039324, 'D': 0.4612905130128658},
 'auddig': {'L': 8, 'P': 1.8834110997525702, 'D': 0.5117823761413419},
 'audtr': {'L': 8, 'P': 1.9892680621155827, 'D': 0.5046141055524362},
 'vidtr': {'L': 8, 'P': 0.05520253973872224, 'D': 0.0846136627657064},
 'viddig': {'L': 8, 'P': 1.862571613911107, 'D': 0.5074553132446618},
 'so': {'L': 8, 'P': 1.7027472358912694, 'D': 0.5046386226501091},
 'on': {'L': 8, 'P': 1.4169662215350334, 'D': 0.4907407637366824},
 'em': {'L': 8, 'P': 1.0590065753144235, 'D': 0.44420264450045377},
 'sms': {'L': 8, 'P': 1.8487648735160152, 'D': 0.5090970201714644},
 'aff': {'L': 8, 'P': 0.6018657109295106, 'D': 0.39889023002777724},
 'sem': {'L': 8, 'P': 1.34945185610011, 'D': 0.47875793676213835}}
```

### Diminishing return model

The goal for each channel is to find the relationship (fit a Hill function) between spending and contribution so `ROAS` and `mROAS` can be calculated.

![Plot 7](/posts/images/diminishing-return-model.png)

`x` - adstocked media channel spending  
`K` - half saturation  
`S` - shape  
`y` - target variable, the media channel's contribution

Variables are centralized

**Priors**

|Parameter|Prior|Description|
|---------|-----|-----------|
|$K$|beta(2, 2)|Half saturation point|
|$S$|gamma(3, 1)|Shape|
|$\beta$|half normal(0, 1)|Coefficient|
|Residual|inverse gamma(0.05, 0.05*0.01)|Residual variance|

**Python code**

```python
def create_hill_model_data(df, mc_df, adstock_params, media):
    y = mc_df['mdip_'+media].values
    L, P, D = adstock_params[media]['L'], adstock_params[media]['P'], adstock_params[media]['D']
    x = df['mdsp_'+media].values
    x_adstocked = apply_adstock(x, L, P, D)
    # centralize
    mu_x, mu_y = x_adstocked.mean(), y.mean()
    sc = {'x': mu_x, 'y': mu_y}
    x = x_adstocked/mu_x
    y = y/mu_y
        
    model_data = {
        'N': len(y),
        'y': y,
        'X': x
    }
    return model_data, sc

model_code3 = '''
functions {
  // the Hill function
  real Hill(real t, real ec, real slope) {
  return 1 / (1 + (t / ec)^(-slope));
  }
}

data {
  // the total number of observations
  int<lower=1> N;
  // y: vector of media contribution
  vector[N] y;
  // X: vector of adstocked media spending
  vector[N] X;
}

parameters {
  // residual variance
  real<lower=0> noise_var;
  // regression coefficient
  real<lower=0> beta_hill;
  // ec50 and slope for Hill function of the media
  real<lower=0,upper=1> ec;
  real<lower=0> slope;
}

transformed parameters {
  // a vector of the mean response
  vector[N] mu;
  for (i in 1:N) {
    mu[i] <- beta_hill * Hill(X[i], ec, slope);
  }
}

model {
  slope ~ gamma(3, 1);
  ec ~ beta(2, 2);
  beta_hill ~ normal(0, 1);
  noise_var ~ inv_gamma(0.05, 0.05 * 0.01); 
  y ~ normal(mu, sqrt(noise_var));
}
'''

# train hill models for all media channels
sm3 = pystan.StanModel(model_code=model_code3, verbose=True)
hill_models = {}
to_train = ['dm', 'inst', 'nsp', 'auddig', 'audtr', 'vidtr', 'viddig', 'so', 'on', 'sem']
for media in to_train:
    print('training for media: ', media)
    hill_model = train_hill_model(df, mc_df, adstock_params, media, sm3)
    hill_models[media] = hill_model
```

**Diminishing Return Model (Fitted Hill Curve)**

![Plot 7](/posts/images/fitted-hill-curve.png)

## Calculating overall ROAS and weekly ROAS

`Overall ROAS` = total media contribution / total media spending  
`Weekly ROAS` = weekly media contribution / weekly media spending

**Distribution of weekly ROAS (weekly data, last year)**  

Red line - mean, green line - median.

![Plot 7](/posts/images/distribution-weekly-roas.png)

## Calculating mROAS

Marginal ROAS represents the return of incremental spending based on current spending. Example - we've already spent $100 on SEM, how much will the next $1 bring?

We can calculate mROAS by increasing current spending by 1%, estimating incremental channel contribution and dividing it by incremental channel spending.

1. Current spending level `cur_sp` is an array of weekly spending in a given period.
2. We create next spending level `next_sp` by increasing `cur_sp` by 1%.
3. We plug both `cur_sp` and `next_sp` into the Hill function:
	* Current media contribution: `cur_mc` = beta * Hill(`cur_sp`)
	* Next-level media contribution: `next_mc` = beta * Hill (`next_sp`)
4. mROAS = (sum(`next_mc`) - sum(`cur_mc`)) / sum(0.01 * `cur_sp`)

# Results and marketing budget optimization

Few visualization ideas that can help the marketing team understand efficiency of current budget allocation.

**Media channel contribution**

![Plot 7](/posts/images/media-channel-contribution.png)

![Plot 7](/posts/images/media-contribution-percentage.png)

**ROAS**

![Plot 7](/posts/images/media-channel-contribution-roas.png)

**mROAS**

![Plot 7](/posts/images/media-channel-roas-mroas.png)
