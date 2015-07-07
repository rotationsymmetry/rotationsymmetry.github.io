---
title: Penalty in Statistical Learning
tags: analytics
---
Over the years, many penalty terms have been discovered or designed to have various features and capabilities. I think it is very nice to do a quick summary of these penalty terms.

<!-- more -->

The idea behind the penalty term is quite straightforward: Less desirable models will be assigned a higher penalty in the cost function. As a result, these models are less likely to be selected by the training algorithm.

For a quick summary, I would like to fos

| Penalties | Variable <br>Selection|Oracle <br> Property| Correlated <br/> Group Var | Algorithms |
| ----------|-------------| -----| ------ | ----- |
| Ridge | No | No | Yes | ?|
| Lasso |Yes| No| No| LARS|
| SCAD  |Yes|Yes| No|Shooting Algorithm|
| Adaptive Lasso|Yes|Yes|No|LARS|
| Elastic Net|Yes|Yes|Yes|LARS|
