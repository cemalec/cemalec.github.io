---
title: Gradients and You - Softmax
layout: post
permalink: /Gradients_Softmax/
---

### The Gradient for Softmax Loss

This is the second in a series on gradients, check out an introduction [here]() and also the [hinge loss gradient]().
The softmax function appears in a number of disciplines and goes by many names. I first saw it in physics class in the context of thermodynamics. We called the bottom half the partition function and its related to many thermodynamic variables like free energy and entropy.

The loss function used for softmax svm is stated like this for a single training example:

$$L_i = -log\left (\dfrac{e^{s_{iy_i}}}{\sum\limits_j e^{s_{ij}}}\right )$$

where again $$s_{ij} = \sum\limits_k X_{ik}W_{kj}$$. This is not conventional notation, but for this article, I will be calling $$e^{s_{ij}}$$ the *score factor* in analogy with the Boltzmann factor in thermodynamics (which is written $$e^{-\beta E}$$. This loss is relatively straightforward to calculate compared to the hinge loss. 

We will calculate the scores, and then the matrix of score factors. We will produce both a selection of the correct score factors and a sum over all the classes of the score factors. Finally, we divide the two vectors element-wise and take the negative logarithm. 

A trick often suggested is to subtract off the maximum score from all elements in the score matrix so that the score factors don't become too large. Since the score factors are normalized to always be less than one, this is more a practical consideration than a mathematical one. It doesn't affect the outcome because

$$\dfrac{e^{s_{iy_i}-s_{max}}}{\sum\limits_j e^{s_{ij}-s_{max}}} = \dfrac{e^{-s_{max}}e^{s_{iy_i}}}{e^{-s_{max}}\sum\limits_j e^{s_{ij}}} = \dfrac{e^{s_{iy_i}}}{\sum\limits_j e^{s_{ij}}}$$

So let's translate this loss function into numpy:

```python
#Find number of training examples
num_train = X.shape[0]

#Useful for indexing over all rows
i = np.arange(num_train)

#Calculate Scores

scores = X @ W

#subtract off the maximum score, do this for each training example
#keepdims keeps the 1D matrix as a column vector

score_max = np.amax(scores, axis = 1, keepdims = True) 
scores -= score_max

#calculate score factors

score_factors = np.exp(scores)

#Create sums to normalize the score factors

score_factor_sums = np.sum(score_factors,axis = 1, keepdims = True)

#Calculate the softmax function
softmax = score_factors/score_factor_sums

#Calculate log Loss

Loss = np.sum(-log(softmax[i,y]))
Loss /= num_train
Loss += reg*np.sum(W*W)
```

Now we would like to calculate the gradient. As often happens when the exponential function and logarithm get into the mix, we will end up with a derivative that resembles the original function, allowing us to save ourselves some computation. For convenience, I'm going to abbreviate the softmax function as *m*

Using the chain rule, we first take the derivative of the logarithm (in this case it is a natural logarithm) with respect to the softmax function. 

- Derivative of the Loss with respect to softmax: $$\dfrac{\partial L}{\partial m_{iy_i}} = - \dfrac{1}{N}\sum\limits_i \dfrac{1}{m_{iy_i}}$$


- Derivative of softmax with respect to the scores: $$\dfrac{\partial m_{iy_i}}{\partial s_{ij}} =  \dfrac{e^{s_{iy_i}}}{\sum\limits_j e^{s_{ij}}}\delta_{iy_i} - \dfrac{e^{s_{iy_i}}e^{s_{ij}}}{(\sum\limits_j e^{s_{ij}})^2}\delta_{ij}$$


- In terms of the softmax function: $$\dfrac{\partial m_{iy_i}}{\partial s_{ij}} = m_{iy_i}\delta_{iy_i} - m_{iy_i}m_{ij}\delta_{ij}$$

- Derivative of the scores with respect to the weights: $$\dfrac{\partial s_{ij}}{\partial W_{kj}} = X_{ik}\delta_{kj}$$

And we put it all together to get

$$\dfrac{\partial L}{\partial W_{kj}} = -\dfrac{1}{N}\sum\limits_i \dfrac{1}{m_{iy_i}}(m_{iy_i}\delta_{iy_i} - m_{iy_i}m_{ij}\delta_{ij})X_{ik} = \dfrac{1}{N}\sum\limits_i(m_{ij}\delta_{ij} - \delta_{iy_i})X_{ik} = \dfrac{1}{N}\sum\limits_iX_{ki}^T(m_{ij}\delta_{ij} - \delta_{iy_i})$$

Where the regularization loss and its derivative are the same as the previous example. Now let's translate this into numpy:

```python
#Calculate softmax, oh wait, you already did, just subtract one from each correct class
softmax[i,y] -= 1

#Multiply by feature vectors
dLdW = X.T @ softmax

#Normalize and add regulatory term
dLdW /= num_train
dLdW += 2*reg*W
```

The derivative of the sigmoid function used for neural nets has a similarly nice property which allows for very efficient computation of complex gradients. We'll get to that later. 
