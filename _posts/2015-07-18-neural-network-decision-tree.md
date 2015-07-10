---
title: Neural Network and Decision Tree
tags: analytics
---
Neural network deep learning is definitely the hot buzzword for machine learning. What are its benefits when comparing to good old methods like decision tree? 

<!-- more -->


## The Theory
It has been proved in the theory that the a neural network with enough neurons can represent or approximate

* Any boolean function
* Any continuous function 

How about decision tree? It can be shown that with enough leaf nodes, a decision tree can represent or approximate 

* Any boolean function
* Any continuous function 

So on paper, a neural network does not appear to be better than the decision. But a statistical learning model can represent or approximate something does not always mean it can learn it well from empirical data. There are deep architectural difference between neural network and decision trees. The prediction capability of the two approaches in practical scenarios are not always the same.   

## Subtree Duplication
The defining feature of a decision tree is that from the root, the business logics recursively branches out at an inner node based on a single condition until a leaf node is reached. The approach is simple to implement and interpret. However, the simplicity also introduces potential redundancy. For example, we can consider evaluating the boolean function \\(R=(A \cap B) \cup (C \cap D)\\). The figure below illustrates the decision tree representation of this boolean function. 

The subtrees involving \\(C\\) and \\(D\\) are obviously duplicated. The logics with \\(C\\) and \\(D\\) are in *parallel* with the logics of \\(A\\). As a result, we need to repeatedly consider \\(C\\) and \\(D\\) on both branches of the inner node of \\(A\\), leading the the duplication. 

On the other hand, a neural network will not suffer from the same duplication problem when evaluating \\(R=(A \cap B) \cup (C \cap D)\\). As illustrated below, a neural network will simply have two neurons in the hidden layer evaluating \\(A \cap B\\) and \\(C \cap D\\) respectively; and then the combine the the results at the final output neuron. 

 

For simple examples like \\(R=(A \cap B) \cup (C \cap D)\\), the duplication do not pose any problems and both the decision tree and the neural network will get learn the boolean function equally well from empirical data. For larger and more complex problem, however, we will begin to see the difference. 

## Signal Detector
Using the knowledge we gain from analyzing \\(R=(A \cap B) \cup (C \cap D)\\), let's construct an more interesting learning problem. 

Suppose \\(p\\) LED lights are along a horizontal line. They are controlled by a hidden random signal generator. The generator will set \\(q\\) consecutive light on and the rest are off. The value of \\(q\\) and the locations of the \\(q\\) consecutive lights will change every second. For simplicity, \\(q\\) can take on two values \\(g\\) and \\(r\\), at 50-50 chance. In addition, we can observe a beacon. With 90% probability, the beacon will be red if \\(q=r\\) and green if \\(q=g\\). 

The figure below depicts a few realization of the LED lights and the beacon for \\(p=16\\), \\(r=5\\) and \\(g=3\\). The circles are the the LED lights with yellow being "on" and black being "off". The triangle is the beacon. In the last one, 5 LED's are on but the beacon is green, as the beacon matches the LED only 90% of the time. 
 
The learning goal is to predict the color of the beacon with the on-off status of the \\(q\\) LED lights. Please note the learning algorithms only have access to the color of the beacon and the on-off status of the individual lights. The learning algorithm are not aware of the mechanism that sets the lights on or off. Otherwise, the learning problem will be too easy: Simple count how many lights are on and you are done.

Next we training the learning algorithms. To keep it simple, we use neural network with a single hidden layer. The number of neurons in the hidden layer is \\(p\\). The activations between layers are modeled by sigmoid functions. For decision tree, we use gini impurity as criterion for branch splitting. 

The following table summarizes the prediction errors under various scenarios. 

The advantage of neural network in the signal detection problem is similar to evaluating \\(R=(A \cap B) \cup (C \cap D)\\). The consecutive lights that are on can be placed on any location along the line. For each location, the neural network can use one neuron in the hidden layer to record whether \\(r\\) or \\(g\\) consecutive lights are on near that location; then summarize the information of all locations at the output layer. The decision tree, however, can only consider 

## When to Use What




