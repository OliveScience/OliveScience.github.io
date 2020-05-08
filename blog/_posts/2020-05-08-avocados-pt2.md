---
title: "2017 Spike in Avocado Prices"
layout: post
subtitle: Examining Average Prices of Avocados using Prophet
tags: [prophet, rstan, SeanJTaylor, RStudio, linear modeling, splines, prediction, forecasting, dplyr, lineplot, Avocados, Analysis, Visualization]
bigimg: null
---




# Introduction

In my last [post](https://9olive.github.io/blog/2020/04/20/avocado-pt1.html) I explored the average prices of avocados from 2015 - 2018. In this follow up post I answer the following question: *Can the trend in avocado prices be explained?* To answer this, I examine the data statistically, treat the data as a time series analysis, and conclude with action item to be taken in a hypothetical situation. 

## Contents of this post

  * Basic linear modeling in R
  * Measuring collinearity between predictors
  * Applying Prophet modeling and forecasting
  * Conclusion
  
### Basic Linear Modeling in R

A 14th century theologian once stated that *"Entities should not be multiplied without necessity"*. This idea, born from William of Ockham, is often implemented as a guiding principle stated as, the simplest idea is most likely the right one. Ockham's Razor is not a pillar of logical reasoning, however, it serves as a well endured rule of thumb. 

So can the prices of avacados simply be explained by the few straight forward variables from the avocado dataset? The dataset has the following variables

{% highlight text %}
## # A tibble: 6 x 14
##   Week_no Date       AveragePrice `Total Volume` `4046` `4225` `4770`
##     <dbl> <date>            <dbl>          <dbl>  <dbl>  <dbl>  <dbl>
## 1      52 2015-12-27         1.33         64237.  1037. 5.45e4   48.2
## 2      51 2015-12-20         1.35         54877.   674. 4.46e4   58.3
## 3      50 2015-12-13         0.93        118220.   795. 1.09e5  130. 
## 4      49 2015-12-06         1.08         78992.  1132  7.20e4   72.6
## 5      48 2015-11-29         1.28         51040.   941. 4.38e4   75.8
## 6      47 2015-11-22         1.26         55980.  1184. 4.81e4   43.6
## # ... with 7 more variables: `Total Bags` <dbl>, `Small Bags` <dbl>, `Large
## #   Bags` <dbl>, `XLarge Bags` <dbl>, type <chr>, year <dbl>, region <chr>
{% endhighlight %}

  * Week_no: Weeks of the year enumerated
  * **Date**: Indexed weekly
  * **Average Price**: Our response varaible, <img src="https://render.githubusercontent.com/render/math?math=y">
  * **Total Volume**: Weekly number of avocados sold
  * 4046, 4225, 4770: Count for different PLUs of avocados sold
  * Total Bags: For avoacods that come in bagged units
  * Small - XLarge Bags: Count for different sized bags
  * **type**: Conventional avocados vs Organic
  * year: Year component of the date
  * **region**: Different US regions at the scope of cities (some regions are compose of cities commonly examined together, i.e.: CincinnatieDayton)
  
The relevant variables are highlighted. The rationale behind removing variables is based on the fact that the ones removed can be represented by another variable. Week_no and year is represented by date, and the count of bags or PLUs of avocados is represented by Total volume and type. 

I'll be fitting a model that uses date, weekly volume of avocados sold, type of avocado, and region to predict the average price. Type and region will be treated as categorical variables. Additionally, I'll be leaving out the last 52 weeks of data to assess the fit later on. So how does the simplest linear model perform?


#### Summary:
  
  * Coefficent of Determination: 0.632. About 63% the variance in our data is explained by this model.
  * F-Statistic P-Value is effectively 0, therefor our model is better than just using the average price to explain the data. 
  
Looking more closely at the plots some key attributes are observed:

  * Our residuals do not have a consistent spread in the Residuals vs Fitted plot. Observe the larger spread of error around \$ 1.60 that is not present around \$ 1.00. This isn't surprising, as the average prices of avocados experienced a lot of volatility in the range of \$ 1.50 - \$ 2.00.
  * There is moderate skewness in the tails of our QQ plot. The QQ plot compares our error to a normal bell curve. If our error is normally distributed, than the points should remain close to the dotted line. 
  * Our Residuals vs Leverage suggest that we take a look at outliers in both the x-direction and y-direction. 
  
![testing](/blog/addons/2020-05-08-avocados pt2/plot_summary-1.png)

The simplest model does not confirm the basic assumptions of linear modeling. These assumptions are

  1. Our predictor matrix is comprised of fully independent variables.
  2. Our error follows a normal distribution with constant variance.
  3. Our error is independently and identically distributed.

The first assumption pertains to the structure of our model. If the predictor variables fail to meet this assumption, then another modeling approach will have to be considered. From the initial linear model, there is evidence to suggest that some our predictor data (Total Volume, type, region, and date) do not affect average price in a way that is independent of one another. 

### Measuring Collinearity in our Predictors

The model can only be as good as the data collected. One decent way to look at collinearity in a predictor matrix is to consider the pair-wise scatter plot matrix. It is visually intuitive to digest the relationship between the pairs of predictor factors. Though the option does no do when one's data is high dimensional, or, in my case, the data consist of many factors. Therefore, I'll rely on a numerical method of determining collinearity. 

A condition number, <img src="https://render.githubusercontent.com/render/math?math=\kappa">, can be calculated to determine how colllinear our observations are. The value of <img src="https://render.githubusercontent.com/render/math?math=\kappa"> is derived from eigenvalues. A large <img src="https://render.githubusercontent.com/render/math?math=\kappa">, typcially <img src="https://render.githubusercontent.com/render/math?math=\geq 30"> indicates high collinearity. The exact relationship is shown below.

<img src="https://render.githubusercontent.com/render/math?math=\kappa = (\lambda _1 / \lambda _p)^{1/2} && ">
<img src="https://render.githubusercontent.com/render/math?math=\text{Where the eigenvalues are computed and ranked: } \lambda_1 \geq \lambda_2 \geq ...\geq \lambda_p \geq 0">
```



The condition number for the predictor matrix is 63,202,332, which is well beyond our threshold. What does an acceptable condition number look like under an ideal circumstance? And ideal circumstance would be where the variables are generated by different processes. An example has been created below to demonstrate. 

In the code below, two predictor variables are generated from a random uniform process and a random gamma process. A response variable is generated using the following equation: <img src="https://render.githubusercontent.com/render/math?math=y = 3x_1 {+} 7x_2 + \epsilon"> where <img src="https://render.githubusercontent.com/render/math?math=\epsilon"> is noise generated from a random normal process. A simple linear model is fit, and the eigen values are calculated from the predictor matrix. The summary of the fit is printed, and condition number is below.


{% highlight r %}
set.seed(5683)

# Generating data
x <- c(runif(100, 2, 5), rgamma(100, 10, 5))
x <- matrix(x, ncol = 2)

# Parameters
b <- matrix(c(3, 7), nrow = 2)

# Response variable
y <- (x %*% b) + rnorm(100)

simple_lm <- lm(y ~ x)

# Eigenvalue calc
simple_eigen <- eigen(t(model.matrix(simple_lm)[,-1]) %*% model.matrix(simple_lm)[,-1])
cond_num <- sqrt(max(simple_eigen$values) / min(simple_eigen$values))

# Outputting the summary of our linear model for comparison
summary(simple_lm)
{% endhighlight %}



{% highlight text %}
## 
## Call:
## lm(formula = y ~ x)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -2.36052 -0.60411 -0.02918  0.52837  1.99711 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  -0.6225     0.5959  -1.045    0.299    
## x1            3.0558     0.1164  26.244   <2e-16 ***
## x2            7.1312     0.1522  46.843   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9666 on 97 degrees of freedom
## Multiple R-squared:  0.9628,	Adjusted R-squared:  0.962 
## F-statistic:  1255 on 2 and 97 DF,  p-value: < 2.2e-16
{% endhighlight %}

Condition number: 5.747


With the condition number being so large in the avocado price model, there is evidence to suggest that a model with fewer predictor parameters can be used. At this point, the diligence of investigating a simple approach is done. In my previous avocado post, visualizing the data as a time series distribution seemed most appropriate. The analysis, so far, is consistent with the exploratory work initially done. 

### Apply time series... Prophet?

Traditional time series methods attempt to model the variance of data by approximating the structure of the process that generates the data. I've chosen to experiment with the Prophet package released by the Facebook open source team. In essence, Prophet decomposes the time series. 

To asses the potential of Prophet, I'll limit the data to just Organic avocado prices in the Raleigh-Greensboro area. If it the Prophet were to show potential, then a time series model can be generated for each category (a trivial task for a computer).

#### Organic Avocados in Raleigh-Greensboro

I'll start this by limiting the data to the average price of organic avocados in Raleigh-Greensboro. The time series of this specific segment has some seasonality over the period of 2015 - 2016, however in 2017 there are spikes in the average price. The surge in price will be difficult to model without additional data, so I don't expect the prophet model to capture this. Though, perhaps it can capture the upward trend. The fit of the first simple linear model (in red) is pasted over the actual data. 

![testing](/blog/addons/2020-05-08-avocados pt2/Avocado Average Price over time form 2015 to 2018-1.png)

In black is the average price of avocados for Raleigh-Greensboro. In red is are the fitted values from the initial simple linear model. It's visually clear here that the inital model may not be appropriate. It's not smooth due to the model being of categorical nature. It also does a poor job modeling the volatility in mid-2017. 


![testing](/blog/addons/2020-05-08-avocados pt2/Prophet-1.png)


Here is the Prophet model. The black points indicate the data used to form the model. I purposely left out the last ~52 weeks of data as in the initial simple linear model. 

Next, I lay over the actual data below (in green) and the previous simple linear model (in red).

![testing](/blog/addons/2020-05-08-avocados pt2/Prophet vs ANOVA-1.png)

It's clear that Prophet does a better job modeling than the simple linear model approach. Though it's worth noting a few things:

  * At the first spike in price, price was forecasted to drop. This was due to seasonal affects from 2015 and 2016. With context this makes sense, but more on that after noting a few more observations.
  * There are two spikes in price, and then a severe drop at the start of 2018.
  * The spikes in average price, at least for Organic avocados in the Raleigh-Greensboro region, appear anomolous.
  * The simple linear model is essentially a piecewise mean model.
  
### Conclusion

The Prophet model performs better It's conceivable that that anomalous behavior of average price in 2017 could have been anticipated by buyers in the produce market, restaurant or cafe managers, and inventory managers. Suppose that you find yourself in this role or managing this role. If a future contract needs to be finalized for an order that is fulfilled in 12 weeks, then is it possible to anticipate the sale price of avaocados than? Using this model, it is possible. Though, our model is only as good as our data, and we need more data. It would be still unhelpful if one blindly collects more data.

The average avocado prices will be subject to the same uncertainty that any good on the market is. Alas, the risk can be culled through making the data available for anyone. 

Check out the interactive forecase model in action:


