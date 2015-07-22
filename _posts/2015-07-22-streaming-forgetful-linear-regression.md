---
title: Streaming Forgetful Linear Regression
tags: [analytics, spark]
---

In supervised learning of streaming data, the relation between the label and the features can potentially change over time. For example, searches involving Apple is likely to increase as we approach WWDC. The current implementation of [streaming linear regression](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.regression.StreamingLinearRegressionWithSGD) in Spark treats all streaming data coming from the same source and thus is unable to reflect the changes in the relations. [streaming k-means](https://databricks.com/blog/2015/01/28/introducing-streaming-k-means-in-spark-1-2.html), on the other hand,  addresses the issue of changing data source by introducing a decay factor that intentionally "forget" data from previous batches. Inspired by streaming k-means, this post is a proposal for incorporating forgetfulness to streaming linear regression.

## Weighted Loss for Forgetfulness
In Spark, streaming data is handled through mini-batches. The data arrives between two time points are packaged in an RDD and pushed to the Spark engine for batch processing. Suppose we are currently at the 2nd window of the stream and \\(R_1\\) and \\(R_2\\) are the first and second RDD of the stream. For \\(i=1 ,2\\), \\(y_{ij}\\) are the labels and \\(x_{ij} \\) are the \\(p\\)-dimensional feature vectors for \\(R_i\\). In the usual linear regression, we minimize the square loss
$$\sum_j (x_{1j}\cdot \beta-y_{1j})^2+\sum_j (x_{2j}\cdot \beta-y_{2j})^2$$
To incorporate the forgetfulness, we can introduce a decay factor \\(a\\) and consider the weighted square loss
$$a\sum_j (x_{1j}\cdot \beta-y_{1j})^2+\sum_j (x_{2j}\cdot \beta-y_{2j})^2$$

* When \\(a=1\\), \\( R_1\\) is completely remembered and contributes equally to loss as in \\( R_2\\).
* When \\(a=0\\), \\( R_1\\) is completely forgotten and contributes nothing to the loss.

A value of \\(a\\) between 0 and 1 characterizes the level of forgetfulness. For easy interpretation, we can derive \\(a\\) using half life as in [streaming k-means](https://databricks.com/blog/2015/01/28/introducing-streaming-k-means-in-spark-1-2.html).

If we have access to \\(R_1\\) and \\(R_2\\)  simultaneously, the optimization of weighted loss can be easily solved by regular algorithm like [stochastic gradient descend](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.regression.LinearRegressionWithSGD). For streaming data,  however, we typically no longer have access to \\(R_1\\) when (R_2\\) arrives. So a more streaming friendly algorithm is needed.

## Math for Streaming Weighted Loss
Stacking the labels and features, we can form the vector \\(y_i \\) and matrix \\(X_i\\) for \\(i=1,2\\):

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

## Streaming Algorithm and Complexity

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

## Proposed Public API
We will introduce two public classes to implement the streaming forgetful linear regression.

A user will mostly interactive with the `StreamingForgetfulLinearRegression` class. It is designed to be mimic the existing `StreamingLinearRegressionWithSGD` and can be used as drop in replacement.

``` scala
class StreamingForgetfulLinearRegression
  extends StreamingLinearAlgorithm[LinearRegressionModel, StreamingForgetfulLinearRegressionAlgorithm] with Serializable {
  /**
   * The following methods are directly inherited
   * from StreamingLinearAlgorithm.
   * No coding is needed.
   */
  def latestModel(): LinearRegressionModel
  def trainOn(data: DStream[LabeledPoint]): Unit
  def trainOn(data: JavaDStream[LabeledPoint]): Unit
  def predictOn(data: DStream[Vector]): DStream[Double]
  def predictOn(data: JavaDStream[Vector]): JavaDStream[java.lang.Double]
  def predictOnValues[K: ClassTag](data: DStream[(K, Vector)]): DStream[(K, Double)]
  def predictOnValues[K](data: JavaPairDStream[K, Vector]): JavaPairDStream[K, java.lang.Double]

  /**
   * The following are setter methods for
   * fluent syntax. They are new code.
   */
  def setDecayFactor(a: Double): this.type
  def setHalfLife(halfLife: Double, timeUnit: String): this.type
  def setStepSize(stepSize: Double): this.type
  def setNumIterations(numIterations: Int): this.type
  def setInitialWeights(initialWeights: Vector): this.type
}
```

The actual learning algorithm is implemented in `StreamingForgetfulLinearRegressionAlgorithm`. We will override the `run` method with our algorithm implementation. The \\(cXX\\) and \\(cXY\\) will be persisted as private members of this class.

``` scala
class StreamingForgetfulLinearRegressionAlgorithm
  extends GeneralizedLinearAlgorithm[LinearRegressionModel] with Serializable {
  override def run(rdd: RDD[LabeledPoint], initialWeights: Vector): LinearRegressionModel
}
```
