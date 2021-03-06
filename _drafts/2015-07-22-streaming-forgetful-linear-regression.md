---
title: Streaming Forgetful Linear Regression
tags: [analytics, spark]
---

In supervised learning of streaming data, the relation between the label and the features can potentially change over time. For example, searches involving Apple is likely to increase as we approach WWDC. The current implementation of [streaming linear regression] (https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.regression.StreamingLinearRegressionWithSGD) in Spark treats all streaming data coming from the same source and thus is unable to reflect the changes in the relations. [streaming k-means] (https://databricks.com/blog/2015/01/28/introducing-streaming-k-means-in-spark-1-2.html), on the other hand,  addresses the issue of changing data source by introducing a decay factor that intentionally "forget" data from previous batches. This post will discuss two approaches incorporate the forgetfulness of streaming k-means to streaming linear regression.

# Streaming Linear Regression
In Spark, streaming data is handled through mini-batches. The data arrives between two time points are packaged in an RDD and pushed to the Spark engine for batch processing. Suppose we are currently at the kth window of the stream and \\(R_k\\) is corresponding RDD. In \\(R_k\\), \\(y_{kj}\\) are the labels and \\(x_{kj} \\) are the \\(p\\)-dimensional feature vectors. Then the streaming linear regression will process \\(R_k\\) to generate the model. The linear regression algorithm should be very fast and induce minimal latency. Otherwise, data backlog could occur. Stochastic gradient descend is currently employed. The computational complexity of SGD is \\(O(npk)\\) where \\(n\\) is the number of records in the RDD and \\(k\\) is the number of iterations.
After \\(R_k\\) is processed, Spark will move on to the next RDD and \\(R_k\\) is not persisted.

# Exact Approach
Suppose we are currently at the 2nd window of the stream and \\(R_1\\) and \\(R_2\\) are the first and second RDD of the stream. In the usual linear regression, we minimize the square loss
$$\sum_j (x_{1j}\cdot \beta-y_{1j})^2+\sum_j (x_{2j}\cdot \beta-y_{2j})^2$$
To incorporate the forgetfulness, we can introduce a decay factor \\(a\\) and consider the weighted square loss
$$a\sum_j (x_{1j}\cdot \beta-y_{1j})^2+\sum_j (x_{2j}\cdot \beta-y_{2j})^2$$

* When \\(a=1\\), \\( R_1\\) is completely remembered and contributes equally to loss as in \\( R_2\\).
* When \\(a=0\\), \\( R_1\\) is completely forgotten and contributes nothing to the loss.

A value of \\(a\\) between 0 and 1 characterizes the level of forgetfulness. For easy interpretation, we can derive \\(a\\) using half life as in [streaming k-means](https://databricks.com/blog/2015/01/28/introducing-streaming-k-means-in-spark-1-2.html).

If we have access to \\(R_1\\) and \\(R_2\\)  simultaneously, the optimization of weighted loss can be easily solved. For streaming data, however, we typically no longer have access to \\(R_1\\) when (R_2\\) arrives. we can form the vector \\(y_i \\) and matrix \\(X_i\\) for \\(i=1,2\\):

\\[
y_i =
\left( \begin{array}{c}
y_{i1} \\newline
\vdots \\newline
y_{ij_i}  
\end{array} \right),
X_i =
\left( \begin{array}{c}
x_{i1} \\newline
\vdots \\newline
x_{ij_i}
\end{array} \right)
\\]

Then weighted loss can be written as
$$a\cdot (X_1\cdot \beta -y_1)^t(X_1\cdot \beta -y_1)+(X_2\cdot \beta -y_2)^t(X_2\cdot \beta -y_2)$$
This is a convex function with respect to \\(\beta\\) and we can find the global minimum by gradient descend. The gradient with respect to \\(\beta\\):
$$(a\cdot X_1^tX_1+X_2^tX_2)\beta-(a\cdot X_1^ty_1+X_2^ty_2)$$

A critical observation is that we only need  \\(X_i^tX_i\\) and  \\(X_i^ty_i\\) to evaluate the gradient. All the information in \\(R_i\\) that is relevant to the linear regression is summarized in these two statistics. They are considered as [sufficient statistics](https://en.wikipedia.org/wiki/Sufficient_statistic).

This observation provides a shortcut for optimizing the weighted loss in streaming: After processing the data \\(R_1\\), we can persist \\(X_1^tX_1\\) and  \\(X_1^ty_1\\) for use in processing \\(R_2\\). Because the dimensions of \\(X_i^tX_i\\) and  \\(X_i^ty_i\\) are only \\(p\times p\\) and \\(p\\), they are trivial to persist in storage compared to the entire RDD's.

We can extend this idea to implement a streaming forgetful linear regression algorithm.

### Algorithm and Complexity

At time point \\(i\\), the RDD \\(R_i\\) has \\(n_i\\) records and the feature vectors are of dimension \\(p\\). The cluster has \\(k\\) worker nodes and we perform \\(q\\) iteration for the gradient descend. The matrix \\(XX\\) and vector \\(XY\\) persist the weighted sum of \\(X_j^tX_j\\) and \\(X_j^ty_j\\) from previous time points.

The following are the algorithm and time complexity.

Steps | Time Complexity
------|----------------
Process \\(R_i\\) in one pass to get \\(X_i^tX_i\\) and  \\(X_i^ty_i\\)  | \\(O(p^2\cdot n_i / k)\\)
Set \\(XX=a \cdot XX+X_i^tX_i\\) | \\(O(p^2 )\\)
Set \\(XY=a \cdot XY+X_i^ty_i\\) | \\(O(p)\\)
Use gradient descend to find \\(\beta\\) | \\(O(p^2\cdot q)\\)

\\(p\\) is much less than \\(n_i\\) in general. So the time complexity of algorithm is expected to scale linearly with the size of the RDD.

The space complexity is constant in \\(O(p^2)\\).

# Fast Approach
