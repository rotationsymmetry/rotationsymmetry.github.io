---
title: Streaming Forgetful Linear Regression
tags: [analytics, spark]
---

In supervised learning streaming data, the relation between the label and the predictors can potentially change over time. For example ... The current implementation of [streaming linear regression]() in [Spark]() treat all streaming data coming from the same source with constant relation. Inspired by the decay factor introduced in [streaming k-means](https://databricks.com/blog/2015/01/28/introducing-streaming-k-means-in-spark-1-2.html), this post is a proposal for incorporating forgetfulness to streaming linear regression.

## Forgetfulness
In Spark, streaming data is handled through mini-batches. The data arrives between two time points are packaged in an RDD and pushed to the spark engine for batch processing. Suppose we are currently at the 2nd window of the stream with \\(R_1\\) and \\(R_2\\) being the first and second RDD of the stream. For \\(i=1 ,2\\), \\(y_{ij}\\) are the labels and \\(x_{ij} \\) are the \\(p\\)-dimensional feature vectors for \\(R_i\\). In the usual linear regression, we minimize the square loss
$$\sum_j (x_{1j}\cdot \beta-y_{1j})^2+\sum_j (x_{2j}\cdot \beta-y_{2j})^2$$
To incorporate the forgetfulness, we we can introduce a decay factor \\(a\\) and consider the weighted loss
$$a\sum_j (x_{1j}\cdot \beta-y_{1j})^2+\sum_j (x_{2j}\cdot \beta-y_{2j})^2$$

* When \\(a=1\\), \\( (X_1, Y_1)\\) is completely remembered and contribute equally to loss as in \\( (X_2, Y_2)\\).
* When \\(a=0\\), \\( (X_1, Y_1)\\) is completely forgotten and contributes nothing to the loss.

A value of \\(a\\) between 0 and 1 characterizes the level of forgetfulness. For easy interpretation, we can derive \\(a\\) using half life as in [streaming k-means](https://databricks.com/blog/2015/01/28/introducing-streaming-k-means-in-spark-1-2.html).

If we have access to \\(R_1\\) and \\(R_2\\)  simultaneously, the optimization of weighted loss can be easily solved by regular algorithm like ??? or ???. For streaming data,  however, we typically no longer have access to \\(R_1\\) when (R_2\\) arrives. So a more streaming friendly algorithm is needed.


## Streaming Algorithm for Weighted Loss
Stacking the labels and features, we can form the vector \\(y_i \\) and matrix \\(X_i\\) for \\(i=1,2\\):

$$
y_i =
\left( \begin{array}{c}
y_{i1} \\
\vdots \\
y_{ij_i}  \\
\end{array} \right),
X_i =
\left( \begin{array}{c}
x_{i1} \\
\vdots \\
x_{ij_i}  \\
\end{array} \right)
$$
Then weighted loss can be written as
$$a\cdot (X_1\cdot \beta -y_1)^t(X_1\cdot \beta -y_1)+(X_2\cdot \beta -y_2)^t(X_2\cdot \beta -y_2)$$
This is a convex function with respect to \\(\beta\\) and we can find the global minimum by gradient descend. The gradient with respect to \\(\beta\\):
$$(a\cdot X_1^tX_1+X_2^tX_2)\beta-(a\cdot X_1^ty_1+X_2^ty_2)$$
A critical observation is that we only need  \\(X_i^tX_i\\) and  \\(X_i^ty_i\\) to evaluate the gradient. Since  the dimensions of \\(X_i^tX_i\\) and  \\(X_i^ty_i\\) are only \\(p\times p\\) and \\(p\\), they are trivial to persist in storage compared to the entire RDD's. 



## Computation Complexity

At time point \\(i\\), the RDD \\(R_i\\) has \\(n_i\\) records and the feature vectors are of dimension \\(p\\). We have \\(k\\) executioners and we perform \\(q\\) iteration for the gradient descend. The following are the algorithm and computation complexity. 

Steps | Time Complexity
------|----------------
Process \\(R_i\\) in one pass to get \\(X_i^tX_i\\) and  \\(X_i^ty_i\\)  | \\(O(p^2\cdot n / k)\\)
Set \\(curXX=a \cdot curXX+X_i^tX_i\\) | \\(O(p^2 )\\)
Set \\(curXY=a \cdot curXY+X_i^ty_i\\) | \\(O(p)\\)
Use gradient descend to find \\(\beta\\) | \\(O(p^2\cdot q)\\)

If 



## Proposed Public API
We will introduce two public classes to implement the streaming forgetful linear regression. 
class 
