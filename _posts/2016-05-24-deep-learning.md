---
layout: post
title:  Deep learning
date:   2016-05-24 23:43:08 -0500
categories: [machine learning]
comments: true
description: Notes on deep learning
tags: [neural network]
---

## Introduction

Deep learning (DL) is a buzzword nowadays. 
It has found great success in object recognition, speech 
recognition, image segmentation, machine translation, etc
(see the [nature review paper][nature] in 2015).
In this post I will give a short overview of it.

For the avid learner, a comprehensive list of deep learning 
resources can be found on the github pages of [ChristosChristofidis][awesome]
and [Awesome RNN](https://github.com/kjw0612/awesome-rnn).

## What it is

DL is built upon the older field of neural networks (NN). 
Roughly speaking, it is a NN with many layers. 
Typically it is used as supervised learning. 
Like NN, its structure encodes a nonlinear input-output relationship
and its parameters are to be fitted using known input-output data 
(the so-called training data). 
After parameter fitting (training), generalization to unseen input 
(the so-called test data) is (hopefully) possible. 
Although one is still solving an optimization problem in DL or NN, 
the explicit parametrization of the cost function is too cumbersome to write down. 
There are proofs saying that given enough neurons any function can be
approximated by a NN to arbitrary accuracy, even with only one hidden
layer (The input layer does not contain neurons while the output layer
contains neurons, thus with one hidden layer of neurons, there are
two layers of neurons in total for the NN). 
An intuitive proof of this universality theorem can be found in 
Michael Nielsen's [book][1] (Yes, the same [Michael Nielsen][MN] who 
wrote the [quantum computing book][qc] with [Isaac Chuang][IC]).
Of course more layers will help approximate the desired function more
efficiently with less neurons. 

Due to the complex structure of NN, fancy numerical optimization
methods usually do not apply. Gradient descent and its variants 
are commonly used.
The cost function naturally has layered structure which allows 
gradient descent updates to be done in a layer by layer fashion (back propagation).

Compared to more conventional machine learning methods, 
DL has the advantage that data features are not defined manually, 
thus allowing the processing of natural data in their raw form.
Also it has been demonstrated many times that 
hand crafted features do not work as well as learned ones.

## A brief history

According to the [book][2] by Ian Goodfellow, Yoshua Bengio and Aaron Courville, 
there have been three major waves of development of deep learning. 

1. cybernetics 1940s - 1960s
    * idea: intelligence may be modeled by reverse engineering 
            the brain function using artificial NN
    * hindrance: too little is know about how the brain works
2. connectionism 1980s - 1990s
    * idea: intelligence can emerge when a large number of simple
            computational units are networked together
    * hindrance: computational cost is too high
3. deep learning since 2006
    * idea: depth of NN can be important in the performance
    * hindrance: understanding is poor of why things work 

## The building block 

The basic computational unit of NN is a single neuron, which takes a vector input $$x$$ and scalar output $$y$$. 
The input-output relationship of a single neuron is called 
activation function and there are quite a few choices of them.
One possible choice is the sigmoid neuron with $$a=\sigma(wx+b)$$, 
where $$\sigma(z)=\frac{1}{1+\exp(-z)}$$ is the sigmoid function, 
$$w$$ is a row vector called weight 
and $$b$$ is a scalar called bias. 
Another popular choice is the linear rectifier neuron with 
$$\sigma(z) = \max(0, z)$$.
It is also known as rectified linear unit (ReLU).
Note that activation function needs to be nonlinear function of the 
input $$z$$. Otherwise the layered structure of NN becomes futile
since the composition of linear functions is still linear.


This setup is motivated by the biological neuron, 
which receives inputs from other neurons and passes its output to 
others. 
The sigmoid function is a smoothed version of the step function, 
thus it characterizes the neuron as being either activated 
(1) or deactivated (0). 
The linear rectifier neuron is either deactivated (0), 
or activated with different intensities ($$z>0$$).

## The network

In a DL NN there are layers of neurons and there is freedom how 
to connect the neurons. 
The outputs of previous layers are fed into the succeeding layer as inputs. 
A succinct way to write the input-output relationship for one layer 
is  $$\mathbf a'=\sigma(W\mathbf a+\mathbf b)$$, 
where $$\mathbf a'$$ is the output vector, $$\mathbf a$$ is the input vector (or the output vector of the previous layer), 
$$W$$ is a matrix whose rows correspond to the weights to the neurons of the current layer, 
and $$\mathbf b$$ is the bias vector. 
Note $$\mathbf a'$$ and $$\mathbf b$$ are of the same size while $$\mathbf a$$ may not have the same size 
since the number of neurons does not need to agree from layer to layer.

Thus the parameters of the NN are the weights and biases of the 
neurons. 
The purpose of the training data is to fit out these parameters. 
Note the number of parameters grows rapidly with the number of neurons and the number of layers. 
Thus a large training data set is needed for a sophisticated DL NN
otherwise the system is under-determined. 
Regularization may help in these cases. 
Also data augmentation can be used to increase the training data.
In general, more data is always better than more tricks. 

## The solver
The cost function can take on various forms, 
as long as it aims to characterize the difference between ground truth and model output. 
Due to the complex structure of NN and the nonlinearity of the 
activation function, the minimization of the cost function is 
difficult and global minimum is not guaranteed. 
The nice methods from convex optimization are not applicable 
and one is left with gradient descent or its variations 
to do the minimization. 
Thus a practical concern for choosing the cost function 
is that its gradient should be large and its form should 
be tractable 
for the gradient descent based methods to converge fast.
Note that the choice of the activation function also affects the 
size of the gradient. The sigmoid neuron is considered inferior 
than the ReLU as its gradient is small for most inputs.
It is also worth noting that considering the complexity of the 
whole NN, it is not clear why gradient descent works so well in 
practice.

A common choice of the cost function is the cross-entropy function, i.e.,

$$C_x = -[y(x)\log a + (1-y(x))\log(1-a)] $$

where $$y(x)$$ is the ground truth result for the input $$x$$ and $$a$$ is the NN output. 
The overall cost function is $$C = \sum_x C_x$$. 
To minimized the cost function, one needs to calculate $$\nabla C_x$$ 
where the partial derivatives are with respect to the weights and biases. 
In theory it is possible to write down explicit expression of these derivatives for each neuron. 
But such approach would make the coding extremely tedious and any change in the NN structure would require big change in the code. 
Fortunately, the layered structure of NN enables a layer-by-layer update of all the partial derivatives of $$C$$, 
starting from the output layer towards the input layer. 
This technique is called back propagation, which is essentially the chain rule of calculus. 
At each layer, it is helpful to introduce an intermediate variable $$\delta_j^k\equiv\frac{\partial C}{\partial z_j^k}$$ 
where $$j$$ refers to the neuron index and $$z^k\equiv Wa^{k-1}+b$$. 
Here the superscript denotes the layer index. 
Then starting from the output layer, all the partial derivatives can be easily derived.

For a long time, NN with many layers cannot be solve in reasonable time due to a so-called vanishing gradient problem. 
This is because when there are many layers, the partial derivatives with respect to the early layers become too small. 
This problem limited the early NN research to shadow network structures. 
From 2006, progress has been made to efficiently train deep NNs,
 mostly due to the novel structural designs. 
Two examples are convolutional neural network (CNN) and recurrent neural network (RNN).

## CNN and RNN

CNN plays an important role in computer vision. 
It was proposed in the early 90's and was motivated by the mechanisms in the visual neuroscience. 
There are four key ideas behind CNN: 

* local receptive fields
* shared weights and biases
* pooling
* the use of many layers

A typical CNN is composed of convolutional layers, pooling layers and fully-connected layers.
Neurons in the convolutional and pooling layers are not fully 
connected to the neurons in the previous layer. Instead, each 
neuron is only connected a group of neighboring neurons in the 
previous layer, thus the name of local receptive fields. 
Furthermore, all weights and biases are the same for the neurons
in the convolutional layer, which greatly reduces the number of 
parameters. The shared weights and biases also reflect the intuition
that the same feature is to be searched. 
In practice, there are multiple layers of convolutional layers and 
pooling layers. This hierarchical structure encodes the idea that 
higher-level features are abstracted from lower-level features.

RNN is commonly used for tasks with sequential inputs, such as
natural language processing (NLP) and speech recognition. 
Unlike CNN, where the outputs of the earlier layers
are fed to later layers (known as feedforward NN or FNN), 
RNN contains feedback structure. 
This feedback structure provides more capability but also causes
new problems to the gradient descent solver. 
These self loops make the gradient more likely to vanish or 
explode. 
To control the gradient in a reasonable range, some kind of gating
is needed and the two most common ones are 
long short term memory (LSTM) or Gated Recurrent Unit (GRU).
A good introduction to LSTM can be found on Christopher Olah's [blog][rnn].

It is also possible to combine CNN and RNN together as modules and 
do interesting applications such as captioning images. 

## The future

According to the [nature review paper][nature], the future of 
DL resides in unsupervised learning. 
In the 2015 ICML panel discussion, NLP was considered to have 
potential for big improvement with DL, 
particularly the natural language dialog systems. 
Aslo medicine/healthcare was 
identified as the next big thing for DL, for example, 
medical image analysis and drug discovery.

Another future direction is about hardware, the ones that are more
suitable for NN computations.

## References

* [Neural Networks and Deep Learning][1] by [Michael Nielsen][MN]
* [Deep learning book][2] by Ian Goodfellow, Yoshua Bengio and Aaron Courville
* [Deep learning][nature] by [Yann LeCun][LeCun], Yoshua Bengio 
and Geoffrey Hinton, Nature 521, 436 (2015)
* Stanford deep learning [tutorial][3]
* [Machine learning packages][pack]
* [U Toronto CSC2535](http://www.cs.toronto.edu/~hinton/csc2535/)

[MN]: http://michaelnielsen.org/
[1]: http://neuralnetworksanddeeplearning.com/
[2]: http://www.deeplearningbook.org/
[3]: http://deeplearning.stanford.edu/tutorial/
[pack]: https://github.com/showcases/machine-learning
[awesome]: https://github.com/ChristosChristofidis/awesome-deep-learning
[nature]: http://www.nature.com/nature/journal/v521/n7553/full/nature14539.html
[LeCun]: http://yann.lecun.com/
[IC]: http://feynman.mit.edu/ike/homepage/index.html
[qc]: http://www.amazon.com/Quantum-Computation-Information-10th-Anniversary/DL/1107002176
[rnn]: http://colah.github.io/posts/2015-08-Understanding-LSTMs/
