---
title: Neural Network and Decision Tree
tags: analytics
---
Neural network deep learning is probably the hottest buzzword for machine learning recently. What are its benefits when comparing to good old methods like decision tree? 

<!-- more -->


## The Theory
It has been proved in the theory that the a neural network with enough neurons can represent or approximate

* Any boolean function
* Any continuous function 

How about decision tree? It can be shown that with enough leaf nodes, a decision tree can represent or approximate 

* Any boolean function
* Any continuous function 

So on paper, a neural network does not appear to be better than the decision. But a statistical learning model can represent or approximate something does not always mean it can learn it well from empirical data. There are deep architectural difference between neural network and decision trees, which leads to difference in learning efficiency under certain scenarios.

## Representation Redundancy 
In a decision tree, the data flows from the root,  branches out at an inner node depending on a single condition corresponding to the node, and repeat this until it reaches a  leaf node. The decision tree approach is simple to implement and interpret. However, the simplicity also introduces representation redundancy. To see this, we can consider a simple the boolean function \\(R=(A \cap B) \cup (C \cap D)\\). The decision tree representation of this boolean function is illustrated in the figure below:

The two subtrees involving \\(C\\) and \\(D\\) are identical and they are obviously duplicated. The logics with \\(C\\) and \\(D\\) are in *parallel* with the logics of \\(A\\) and \\(B\\). As a result, we need to repeatedly consider \\(C\\) and \\(D\\) on both branches of the inner node of \\(A\\), leading the the duplication. 

The neural network, on the other hand, does not suffer from the same issue when evaluating \\(R=(A \cap B) \cup (C \cap D)\\). As illustrated below, a neural network will simply have two neurons in the hidden layer evaluating \\(A \cap B\\) and \\(C \cap D\\) respectively; and then the combine the the results at the final output neuron. 

The representation redundancy in decision tree can negatively impact its learning efficiency. The decision tree algorithms is not aware that the two subtrees involving \\(C\\) and \\(D\\) are identical. Among these two subtrees, the one on the left will only be trained using data with \\(A= True\\) and \\(B=False\\) while the one on the right will only be trained using data with \\(A= False\\). There is no borrowing of information across the two subtrees. Consequently the learning efficiency of decision tree is reduced. In contrast, in the neural network, there is a single neuron representing \\(C \cap D \\), which will be training using all available data. So the neural network will remain efficiency with paralle logics.

For simple examples like \\((A \cap B) \cup (C \cap D)\\), both the decision tree and the neural network will learn the boolean function equally well from empirical data. For learning problems featuring complex parallel logics, the advantage of neural network will be more prominent, as we will show in the LED signal detection problem.

## LED Signal Detection
Suppose \\(p\\) LED lights are along a horizontal line. They are controlled by a hidden random signal generator. The generator will set \\(q\\) consecutive LED to be on and the rest are off. The value of \\(q\\) and the locations of the \\(q\\) consecutive lights will change every second. For simplicity, \\(q\\) can take on two values \\(g\\) and \\(r\\), at 50-50 chance. In addition, we can observe a beacon. With 90% probability, the beacon will be red if \\(q=r\\) and green if \\(q=g\\). 

The figure below depicts a few realization of the LED lights and the beacon for \\(p=16\\), \\(r=5\\) and \\(g=3\\). The circles are the the LED lights with yellow being "on" and black being "off". The triangle is the beacon. In the last one, 5 LED's are on but the beacon is green, as the beacon matches the LED only 90% of the time. 
 
The learning goal is to predict the color of the beacon with the on-off status of the \\(q\\) LED lights. Please note the learning algorithms only have access to the color of the beacon and the on-off status of the individual lights. The learning algorithm are not aware of the mechanism that sets the lights on or off. Otherwise, the learning problem will be too easy: Simple count how many lights are on and you are done.

Next we training the learning algorithms. To keep it simple, we use neural network with a single hidden layer. The number of neurons in the hidden layer is \\(p\\). The activations between layers are modeled by sigmoid functions. For decision tree, we consider gini impurity and information entropy as criterion for branch splitting. The neural network is implemented with [Scikit Neural Network](http://scikit-neuralnetwork.readthedocs.org) and the decision tree is implemented with [Scikit Learn](http://scikit-learn.org/stable/modules/tree.html). Both are python packages for fair comparison. 

The following table summarizes the prediction errors under various scenarios. 

The neural network is leading the decision tree by a healthy margin in all of the simulation scenarios. Why? The LED signal detection problem is set up to have many parallel logics: The consecutive \\(q\\) "on" LED's can be placed at any location along the line. So the decision tree will contains many identical subtrees and those deep at the bottom will have very little data to learn from. 

## Beyond LED Signal Detection


## When to Use What



