---
title: Neural Network and Decision Tree
tags: analytics
---
Neural network deep learning is probably the hottest buzzword for machine learning recently. What are its benefits when comparing to good old methods like decision tree? 


<!-- more -->
It has been proved that both decision trees and neural networks can represent (or approximate):

* Any boolean function
* Any continuous function 

So on paper, a neural network does not appear to be better than the decision. However, it is worth pointing out that what a statistical learning model can represent does not always equal to what it can learn well from empirical data. The architectural difference between neural network and decision tree can lead to disparity in learning efficiency.

## Representation Redundancy 
In a decision tree, the data flows from the root,  branches out at an inner node depending on a single condition corresponding to the node, and repeat the process until it reaches a  leaf node. The decision tree approach is simple to implement and interpret. However, the simplicity also introduces representation redundancy. To see this, we can consider a simple the boolean function \\(R=(A \cap B) \cup (C \cap D)\\). The decision tree representation of this boolean function is illustrated in the figure below:

The two subtrees involving \\(C\\) and \\(D\\) are identical and duplicated. This is because the logics with \\(C\\) and \\(D\\) runs in *parallel* with the logics of \\(A\\) and \\(B\\). As a result, we need to repeatedly consider \\(C\\) and \\(D\\) on both branches of the inner node of \\(A\\), leading the the duplication. 

The neural network, on the other hand, does not suffer from the same issue when evaluating \\(R=(A \cap B) \cup (C \cap D)\\). As illustrated below, a neural network will simply have two neurons in the hidden layer evaluating \\(A \cap B\\) and \\(C \cap D\\) respectively; and then the combine the the results at the final output neuron. 

The representation redundancy in decision tree can negatively impact its learning efficiency. The decision tree algorithms is not aware that the two subtrees involving \\(C\\) and \\(D\\) are identical. Among these two subtrees, the one on the left will be trained using only the data with \\(A= True\\) and \\(B=False\\) while the one on the right using the only data with \\(A= False\\). There is no borrowing of information across the two subtrees. Consequently the learning efficiency of decision tree is reduced. In contrast, in the neural network, there is a single neuron representing \\(C \cap D \\), which will be trained using all available data. So the neural network will remain efficiency with parallel logics.

For simple examples like \\((A \cap B) \cup (C \cap D)\\), both the decision tree and the neural network will learn the boolean function equally well from empirical data. For learning problems featuring more complex parallel logics, the advantage of neural network will be more prominent, as we will show in the LED signal detection problem.

## LED Signal Detection
Suppose \\(p\\) LED lights are along a horizontal line. They are controlled by a hidden random signal generator. The generator will set \\(q\\) consecutive LED to be on and the rest are off. The value of \\(q\\) and the locations of the \\(q\\) consecutive lights will change every second. For simplicity, \\(q\\) can take on two values \\(g\\) and \\(r\\), at 50-50 chance. In addition, we can observe a beacon. With 90% probability, the beacon will be red if \\(q=r\\) and green if \\(q=g\\). 

The figure below depicts a few realization of the LED lights and the beacon for \\(p=16\\), \\(r=5\\) and \\(g=3\\). The circles are the the LED lights with yellow being "on" and black being "off". The triangle is the beacon. In the last one, 5 LED's are on but the beacon is green, as the beacon matches the LED only 90% of the time. 
 
The learning goal is to predict the color of the beacon with the on-off status of the \\(q\\) LED lights. Please note the learning algorithms only have access to the color of the beacon and the on-off status of the individual lights. The learning algorithm are not aware of the mechanism that sets the lights on or off. Otherwise, the learning problem will be too easy: Simple count how many lights are on and you are done.

Next we training the learning algorithms. To keep it simple, we use neural network with a single hidden layer. The number of neurons in the hidden layer is \\(p\\). The activations between layers are modeled by sigmoid functions. For decision tree, we consider gini impurity and information entropy as criterion for branch splitting. The neural network is implemented with [Scikit Neural Network](http://scikit-neuralnetwork.readthedocs.org) and the decision tree is implemented with [Scikit Learn](http://scikit-learn.org/stable/modules/tree.html). Both are python packages for fair comparison. 

The following table summarizes the prediction errors under various scenarios. 

The neural network is leading the decision tree by a healthy margin in all of the simulation scenarios. Why? The LED signal detection problem is set up to have many parallel logics: The consecutive \\(q\\) "on" LED's can be placed at any location along the line. So the decision tree will contains many identical subtrees and those deep at the bottom will have very little data to learn from. 

## When to Use What
At this point, you might be tempted to ditch decision trees for good. Not so fast! Instead, I would probably advocate this strategy: try decision tree first; unless it is has been well established that neural network is better fit for your learning problem. 

In my experience, not every learning problem features complex parallel logics. So the prediction accuracy of decision trees could be very good already. In addition, decision trees are much much quicker to train. For the simulation in the LED signal detection problem, it takes no more than 1 second to complete a set of training data with decision trees but several minutes with neural networks. The advantage of decision tree can only be more obvious when we scale up the size of the data. Computational speed is really critical for the initial exploration of data. It allows the data scientist much more time to experiment with various formulations of the learning problem, which could bring in more significant insight of the data than a few percentage point decrease in prediction error. 



