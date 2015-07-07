---
title: Penalty in Statistical Learning
tags: analytics
---
Penalty terms are essential tools to manage regularities for statistical learning of high dimensional data. This post is to provide a quick summary for common penalty terms. 

<!-- more -->

The idea behind the penalty term is quite straightforward: more desirable models will be assigned a lower penalty in the cost function of statistical learning procedures; and vise versa for the less desirable ones . As a result, desirable models models are more likely to be selected by the learning algorithm.

Then what models are desirable? Among the thousands of predictors in high dimensional data, usually a few are relevant for analytical purposes and the rest are typically noise. A desirable model should include only the relevant predictors. Furthermore, for supervised learning, a desirable model should provide accurate and stable estimate for the effect of these predictors. 

Over past decades, many penalty terms have been discovered or designed to identify these desirable models. A few of them received more attention and are more commonly used: ridge, lasso, SCAD, adaptive lasso and elastic net. The original paper that introduce these penalty term can be readily found with google. So I will not going to details. Instead, I created the following table to summarize the important properties of these penalty terms.

| Penalties | Variable <br>Selection|Oracle <br> Property| Handle <br/>Correlated <br/> Variables | Algorithms |
| ----------|-------------| -----| ------ | ----- |
| Ridge | No | No | Yes | OLS|
| Lasso |Yes| No| No| LARS|
| SCAD  |Yes|Yes| No|Shooting Algorithm|
| Adaptive Lasso|Yes|Yes|No|LARS|
| Elastic Net|Yes|Yes|Yes|LARS|

Let's look at the columns one by one.

### Variable Selection
With penalty terms featuring variable selection, the learning algorithm will only include a few variables into the selected model. In supervised learning, this is usually done by setting the coefficient of the excluded variables to be exactly zero. Please note that being capable of variable selection does *NOT* mean the the right variables will be selected. 

### Oracle Property
First and foremost, oracle property has nothing to do with Larry Ellison. 

In statistical learning, oracle property means: with the sample size approaches to infinity,

* the probability of only selecting the relevant variables in the model also approaches to 100%
* the estimated effect of the selected variables 


### Handling Correlated Variables
In the high dimensional data, there are could be one or more groups of predictors that are relevant and are highly correlated with each other within the group. Some penalty terms, such as lasso, are very likely only pick one predictor within each group. This is of course less than ideal as we would lose information by ignoring the rest of the predictors within the same group. Convex penalties, such as elastic net, are able to overcome this and include all predictors.

### Caveats


