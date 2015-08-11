---
title: Sample from the Unsamplable
tags: [analytics]
---

What is the mean size of a family in a region? If we could sample the families in this region, we can estimate the mean by the sample average. However, we are often unable to do so because the regional government does not keep records on families. So instead of sampling the families, we can make random calls to individuals and ask for the size of their family. Suppose the answer is 1, 2, 3, 3 from 4 responders. (1+2+3+3)/4 will obviously overestimate the mean family size because families with more members will be oversampled. What is the right way to do it?

<!-- more -->

Let \\(x\\) denote a family from the region with distribution \\(p(x)\\). \\(f(x)\\) is the size of a family. \\(y\\) is an individual with distribution \\(q'(y)\\). We can identify an individual \\(y\\) to his/her family and induce a distribution on \\(x\\) by \\(q(x)=\sum_{\\text{y in x}} q'(y)\\). So the family size problem becomes how to estimate \\(E_p (f(x))\\) when we can only sample \\(x\\) through \\(q(x)\\)?

By definition of expectation
$$E_p (f(x))=\int f(x)p(x)dx=\int \frac{f(x)p(x)}{q(x)}\cdot q(x)dx=E_q (\frac{f(x)p(x)}{q(x)})$$

This sounds promising as we have switched the expectation from \\(p\\) to \\(q\\). However, we still have one missing piece \\(\frac{p(x)}{q(x)}\\). Very often we don't know the exact values of the ratio. Instead we only can evaluate a function \\(h(x)\\) that is related to the ratio by an unknown multiplicative factor \\(k\\), i.e.  \\(h(x)=k \cdot \frac{p(x)}{q(x)}\\). For the family size problem, we have \\(h(x)=1/f(x)\\) as a family \\(x\\) will be oversampled \\(f(x)\\) times through its members.

There is a simple trick to get rid of \\(k\\). Let \\(g(x)=k\\) be a constant function of \\(x\\) and then we have

$$k=E_p (g(x))=E_q (\frac{g(x)p(x)}{q(x)})=E_q (k \cdot \frac{p(x)}{q(x)})=E_q(h(x))$$

leading to
$$E_p (f(x))=E_q (\frac{f(x)p(x)}{q(x)})=\frac{E_q(f(x)kp(x)/q(x))}{k}=\frac{E_q(f(x)h(x))}{E_q(h(x))}$$

So we have arrived at the final algorithm to estimate \\(E_p (f(x))\\): Sample \\(x_i\\) with \\(q\\), then evaluate

$$\frac {\sum_i f(x_i)h(x_i)}{\sum_i h(x_i)}$$

It is as simple as a weighted average! But I also hope you can appreciate all the rigorous math behind it.

## Black Box Warnings
Now you might have a question: Can I always calculate the expectation through sampling from a totally different distribution? As you might expect, the answer is no.

In the derivation, we have made use of the ratio \\(\frac{p(x)}{q(x)}\\). So the math will fall apart due to division by zero if there is some \\(x\\) that \\(p(x)>0\\) but \\(q(x)=0\\). This should not be a surprise. If members from certain families have no phone numbers, we will be unable to reach them. Such families will be excluded from calculation, resulting in biased estimate of the family size. Therefore, for an unbiased estimate, the alternative sampling distribution should cover all the possible values for original distribution.

In addition, the estimation variance using original distribution \\(p\\) is

$$Var_p(f(x))\cdot \frac{1}{n}$$

while the estimation variance using the alternative distribution \\(q\\) is

$$Var_p(f(x))\cdot \sum_{i=1}^{n} \frac{h(x_i)^2}{(\sum_{j=1}^n h(x_j))^2}$$

By [Cauchy–Schwarz inequality](https://en.wikipedia.org/wiki/Cauchy%E2%80%93Schwarz_inequality)

$$\sum_{i=1}^{n} \frac{h(x_i)^2}{(\sum_{j=1}^n h(x_j))^2} \ge  \frac{1}{n}$$

So the estimation variance using alternative distribution \\(q\\) is always greater than using the original distribution unless \\(q\\) and \\(p\\) are equivalent. I think this observation is quite intuitive and is consistent with the theme in statistics: *It is always better to use the original data and avoid missingness; if you have to use something else as substitute, you pay a price in either variance or bias*.
