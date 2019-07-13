---
title: Gradients and You- SVM hinge loss
layout: default
permlink: /Gradients_Hingeloss/
---
This is the first gradient in a series, for general notes on notation read this [post](https://cemalec.github.io/Gradients_Notation)

### Hinge Loss for SVM

The hinge loss is often used for support vector machines. To help keep indices straight, I'll use the following conventions:

_i_ will be used for training examples

_j_ will be used for classes to be categorized

_k_ will be used as the feature dimensions

Each training example accumulates loss according to how much weight is given to mis-categorized examples. The hinge in hinge loss comes from the fact that loss is accumulated only if the wrong score is greater than the right score by a predetermined margin. The full expression for the loss of one training example is

$$L_i = \sum\limits_{j\neq y_i}max(0,s_{ij} - s_{iy_i} + \Delta)$$ where $$s_{ij} = \sum\limits_k X_{ik}W_{kj}$$

For coding purposes, we break down the loss into a series of smaller computations.

- Scores: $$s_{ij} = \sum\limits_{k} X_{ik}W_{kj}$$

- Margins: $$m_{ij} = s_{ij} - s_{iy_i} + 1$$



- Hinge: $$h_{ij} = max(0,m_{ij})$$


- Loss: $$L_{hinge} = \frac{1}{N}\sum\limits_i\sum\limits_{j\neq y_i} h_{ij}$$

- Loss with Regularization: $$L = L_{hinge} + \lambda\sum\limits_k\sum\limits_jW_{kj}^2$$

This is what it looks like translated into numpy

```python
#Find number of training examples
num_train = X.shape[0]

#Useful for indexing over all rows
i = np.arange(num_train)

#Calculate scores (i x j)
s = X @ W

#Calculate Margins (i x j), some cool numpy indexing and Broadcasting here
m = s - s[i,y][:,np.newaxis] + 1

#Calculate the hinge function (i x j)
h = max(0,m)

#Select only elements of the hinge function that are not in the correct class (i x j)
h[i,y] = 0

#Sum up the loss for all examples and divide by the number of training examples (scalar)
L = (1/num_train)*np.sum(h)

#Add the regulatory loss (scalar)
L += reg*np.sum(W*W)
```

### The Gradient for Hinge Loss

The hinge loss can be an intimidating function at first, but it can be broken down into relatively easy to compute chunks. Similarly, the complicated gradient of the hinge loss can be broken down into easy to compute chunks via the chain rule.

It is important to note that the hinge function $max(0,x)$ does not have a derivative at zero. If SVM were the subject of pure mathematics we would have to reckon with this fact, but it is not. The algorithm has been shown to work as expected in a wide range of applications, possibly because the margin values are almost never exactly zero.

Away from zero, the hinge function has a derivative of zero if its argument is less than zero, and one if it is greater. Here, I choose the derivative to be zero at zero. We denote this function as $\mathbb{1}(x)$.

- Derivative of Loss with respect to margins: $$\dfrac{\partial L_{hinge}}{\partial m_{ij}} = \dfrac{1}{N}\sum\limits_i\sum\limits_{j'\neq y_i}\mathbb{1} (m_{ij} > 0)$$


- Derivative of margins with respect to scores: $$\dfrac{\partial m_{ij}}{\partial s_{ij}} = \delta_{ij\neq y_i} - \delta_{ij=y_i}$$


- Derivative of scores with repect to Weights: $$\dfrac{\partial s_{ij}}{\partial W_{jk}} = X_{ik}$$


- Derivative of the regularization loss: $$\dfrac{\partial}{\partial W_{jk}}\lambda\sum\limits_j\sum\limits_k W_{jk}^2= 2\lambda W_{jk}$$



And finally, we put it all together to obtain the derivative of the loss with respect to the weights

$$\dfrac{\partial L}{\partial W_{jk}} = \dfrac{1}{N}\sum\limits_i\sum\limits_{j'\neq y_i}\left (\dfrac{\partial s_{ij}}{\partial W_{jk}}\right )^T\dfrac{\partial L_{hinge}}{\partial m_{ij}}\dfrac{\partial m_{ij}}{\partial s_{ij}} + \dfrac{\partial}{\partial W_{jk}}\lambda\sum\limits_j\sum\limits_k W_{jk}^2$$

$$\dfrac{\partial L}{\partial W_{jk}} = \dfrac{1}{N}\sum\limits_i\sum\limits_{j'\neq y_i} X_{ki}^T \mathbb{1}(m_{ij} > 0)(\delta_{ij\neq y_i} - \delta_{ij=y_i}) + 2\lambda W_{jk}$$

To implement this in code it is useful to first calculate the matrix where the positive elements of the margins matrix are set equal to one. The elements that are not the correct class remain one, while the elements of the correct class are set equal to $$-\sum\limits_{j\neq y_i}\mathbb{1}(m_{ij} > 0)$$. This matrix is then multiplied by the transpose of the feature matrix.

The reason for the sum term (called class_sums in the code below) is that every margin function includes the scores $$s_{iy_i}$$, and so they all must be summed.

```python
#Derivative of Loss with respect to margin (i x j)
dLdm = np.zeros(m.shape)
dLdm[m > 0] = 1

#Derivative of margin with respect to scores (i x j)
dmds = np.ones(m.shape)
class_sums = -np.sum(dLdm,axis = 1) + 1
dmds[i,y] = class_sums

#Derivative of scores with respect to weights (i x k)
dsdW = X

#Derivative of Loss with respect to weights
dLdW = dfdW.T @ (dLdm*dmds)/num_train
dLdW += 2*reg*W
```
