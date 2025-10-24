---
layout: post
title: Gradients and You - Backpropagation and Neural Nets
permalink: /Gradients_NN/
---
This is a post in a series about gradients.

Let's start with a two layer network. The workflow is 

1) A weight matrix multiplies the feature vectors ($$input_1 = XW_1 + b_1$$)

2) The weighted features go through an activation function to create hidden features ($$hidden = \alpha_1(input_1)$$) 

3) A second weight matrix multiplies the hidden features ($$input_2 = hW_2 + b_2$$)

4) The second layer goes through an activation function to create scores ($$scores = \alpha_2(input_2)$$)

5) A loss is computed from the scores.

6) The gradient of the loss is calculated and used to optimize the weight matrices.

For an example, we use an activation function of $$max(0,x)$$ for the hidden layer and the softmax function for the output layer. The loss is calculated as the log loss. My convention for indices is

- $$i$$: examples


- $$j$$: classes


- $$k$$: feature dimensions


- $$l$$: hidden classes

The *forward pass* then looks like so:

- $$f^{(1)}_{il} = \sum\limits_k X_{ik} W^{(1)}_{kl} + b^{(1)}_{l}$$ 1st layer weighted features

- $$h_{il} = max(0, f^{(1)}_{il})$$ hidden classes from the activation function


- $$s_{ij} = \sum\limits_l h^{(1)}_{il}W^{(2)}_{lj} + b^{(2)}_j$$ 2nd layer scores

- $$L_i = -log(softmax(s_{iy_i})$$ log loss


- $$L = \dfrac{1}{N}\sum\limits_i L_i + \lambda\left(\sum\limits_{k,l}(W^{(1)}_{kl})^2 + \sum\limits_{l,j}(W^{(2)}_{lj})^2 + \sum\limits_l (b^{(1)}_{l})^2+ \sum\limits_j (b^{(2)}_{j})^2\right )$$ loss with regularization


In numpy, it looks like this:
```python
num_train = X.shape[0]
i = np.arange(num_train)

#inputs to hidden layer
inputs1 = X @ W1 + b1[np.newaxis,:]
#hidden layer after activation function
h = np.maximum(0, inputs1)
#inputs to output layer
inputs2 = h1 @ W2 + b2[np.newaxis,:]
#output layer activation function (softmax)        
scores = inputs2 - np.amax(inputs2, axis = 1, keepdims = True)
score_factors = np.exp(scores)
sum_score_factors = np.sum(score_factors, axis = 1, keepdims = True)
softmax = score_factors/sum_score_factors #softmax function for each observation and class
        
#Calculate loss (log loss with regularization)
data_loss = -np.sum(np.log(softmax[i,y]))
data_loss /= num_train
reg_loss = reg*(np.square(W1)).sum() + reg*(np.square(W2)).sum()
reg_loss += reg*(np.square(b1)).sum() + reg*(np.square(b2)).sum()
loss = data_loss + reg_loss
```

Finding the gradient of this isn't much worse than finding the gradient of the SVM examples. However, it should be obvious that calculating the gradient will get rapidly harder as you add layers to the neural net. An important development for neural nets was therefore the concept of backpropagation, or using the values calculated in the loss to calculate the gradient using the chain rule.

Since we need to optimize for two weight matrices and two bias vectors, our gradient will branch off in several places.

The *backward pass* looks like this:

Second layer gradients, all the summation over *i* is reused for the different gradients:
* $$\dfrac{\partial L}{\partial s_{ij}} = \dfrac{1}{N}\sum\limits_i(softmax(s_{ij})-\delta_{iy_i})$$ see softmax gradient


* $$\dfrac{\partial L}{\partial W^{(2)}_{lj}} = \dfrac{\partial L}{\partial s_{ij}} \dfrac{\partial s_{ij}}{\partial W^{(2)}_{lj}}=\sum\limits_ih^T_{li}\dfrac{\partial L}{\partial s_{ij}}$$ gradient with repect to second weight matrix (l x j)


* $$\dfrac{\partial L}{\partial b^{(2)}_j} = \dfrac{\partial L}{\partial s_{ij}} \dfrac{\partial s_{ij}}{\partial b^{(2)}_{j}}=\sum\limits_i\dfrac{\partial L}{\partial s_{ij}}$$ gradient with repect to second bias vector (row vector length j)


First layer gradients:
* $$\dfrac{\partial L}{\partial h_{il}} = \dfrac{\partial L}{\partial s_{ij}}\dfrac{\partial s_{ij}}{\partial h_{il}} = \sum\limits_j\dfrac{\partial L}{\partial s_{ij}}W^{(2)T}_{jl}$$ gradient with respect to hidden classes (l x j)


* $$\dfrac{\partial L}{\partial f_{il}} = \dfrac{\partial L}{\partial h_{il}}\dfrac{\partial h_{il}}{\partial f_{il}}=\dfrac{\partial L}{\partial h_{il}}\mathbb{1}(f_{il} > 0)$$ gradient with respect to weighted features (i x l) 


* $$\dfrac{\partial L}{\partial W^{(1)}_{kl}} = \dfrac{\partial L}{\partial f_{il}}\dfrac{\partial f_{il}}{\partial W^{(1)}_{kl}}=\sum\limits_iX^T_{ki}\dfrac{\partial L}{\partial f_{il}}$$ gradient with respect to first weight matrix (k x l)


* $$\dfrac{\partial L}{\partial b^{(1)}_l} = \dfrac{\partial L}{\partial f_{il}}\dfrac{\partial f_{il}}{\partial b^{(1)}_{l}}=\sum\limits_i\dfrac{\partial L}{\partial f_{il}}$$ gradient with respect to first bias vector (row vector length l)

```python
#gradient of loss with respect to the scores
softmax[i,y] -= 1
dLds = softmax/num_train
#gradient of loss with respect to second weights and bias
grads['W2'] = h.T @ dLds
grads['b2'] = np.sum(dLds,axis=0)
#gradient of loss with respect to hidden features        
dLdh = dLds @ W2.T
#gradient of loss with respect to inputs
dLdi = dLdh*(inputs > 0)
#gradient of loss with respect to first weights and bias
grads['W1'] = X.T @ dLdi
grads['b1'] = np.sum(dLdi,axis=0)
#add gradients due to regularization
grads['W1'] += 2*reg*W1
grads['W2'] += 2*reg*W2
grads['b1'] += 2*reg*b1
grads['b2'] += 2*reg*b2
```

And that's it! An important point here is that we could speed up our computation because *we already calculated a bunch of values we used in the loss*. There is no fundamental reason I know of that it has to be this way. Not all derivatives look like their original function, $$e^x$$ is the only function with this explicit property. Also, the chain rule does not tell you exactly how to break up a derivative into smaller parts. Sometimes there's one way that is obviously easier, but this may not be true with complex derivatives.

This is all to say, that back-propagation in and of itself does not calculate gradients efficiently. You have to make good choices! Your goal should be to use gradients where you use as many of the forward pass values as possible to calculate the backward-pass. 

Another goal is to multiply matrices by vectors instead of matrices by matrices, but I need to learn more to be clear on how often that goal can be achieved.
