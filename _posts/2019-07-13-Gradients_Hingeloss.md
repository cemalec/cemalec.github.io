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

<p align="center"><img src="/_posts/tex/5446c83d569034c121408830fdb8e61f.svg?invert_in_darkmode&sanitize=true" align=middle width=228.43132125pt height=39.26959575pt/></p> where <p align="center"><img src="/_posts/tex/8d10f903f1a93f7146698c3522959577.svg?invert_in_darkmode&sanitize=true" align=middle width=122.93763735pt height=37.03214955pt/></p>

For coding purposes, we break down the loss into a series of smaller computations.

- Scores: <p align="center"><img src="/_posts/tex/e1129db95e7361f17bacec00e5e51252.svg?invert_in_darkmode&sanitize=true" align=middle width=122.93763735pt height=37.03214955pt/></p>

- Margins: <p align="center"><img src="/_posts/tex/f368324a59ce5c8bed8275bd5d28fc89.svg?invert_in_darkmode&sanitize=true" align=middle width=140.66478195pt height=15.296782050000001pt/></p>



- Hinge: <p align="center"><img src="/_posts/tex/8308c4a42d41f664e1a5f9cc7d67d72f.svg?invert_in_darkmode&sanitize=true" align=middle width=129.80417835pt height=17.031940199999998pt/></p>


- Loss: <p align="center"><img src="/_posts/tex/d506d3cb3815a31ed5faac655c511818.svg?invert_in_darkmode&sanitize=true" align=middle width=166.66753619999997pt height=43.7234787pt/></p>

- Loss with Regularization: <p align="center"><img src="/_posts/tex/0167d8db502ca0c07c774035505b70a2.svg?invert_in_darkmode&sanitize=true" align=middle width=192.93340274999997pt height=38.89287435pt/></p>

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

It is important to note that the hinge function <img src="/_posts/tex/17dbb49fa54774c628dc5b96668c94d8.svg?invert_in_darkmode&sanitize=true" align=middle width=70.22275589999998pt height=24.65753399999998pt/> does not have a derivative at zero. If SVM were the subject of pure mathematics we would have to reckon with this fact, but it is not. The algorithm has been shown to work as expected in a wide range of applications, possibly because the margin values are almost never exactly zero.

Away from zero, the hinge function has a derivative of zero if its argument is less than zero, and one if it is greater. Here, I choose the derivative to be zero at zero. We denote this function as <img src="/_posts/tex/7fd3f88f4852072eb1cecaa18566849a.svg?invert_in_darkmode&sanitize=true" align=middle width=34.05260099999999pt height=24.65753399999998pt/>.

- Derivative of Loss with respect to margins: <p align="center"><img src="/_posts/tex/cc6f837fead7a5bd3f3c813eb36b3b9c.svg?invert_in_darkmode&sanitize=true" align=middle width=243.30522645000002pt height=44.5453998pt/></p>


- Derivative of margins with respect to scores: <p align="center"><img src="/_posts/tex/781ad77e6655ebd64f4c491d035506bb.svg?invert_in_darkmode&sanitize=true" align=middle width=159.6858978pt height=38.5152603pt/></p>


- Derivative of scores with repect to Weights: <p align="center"><img src="/_posts/tex/53b21d22450f5666a45825dd971c4c36.svg?invert_in_darkmode&sanitize=true" align=middle width=88.78385175pt height=38.5152603pt/></p>


- Derivative of the regularization loss: <p align="center"><img src="/_posts/tex/41b5981d7510b6109d39b82e856c37b0.svg?invert_in_darkmode&sanitize=true" align=middle width=204.96641714999998pt height=44.1686784pt/></p>



And finally, we put it all together to obtain the derivative of the loss with respect to the weights

<p align="center"><img src="/_posts/tex/58cae7829dab2e106c6f7655520be204.svg?invert_in_darkmode&sanitize=true" align=middle width=483.58564484999994pt height=49.64412915pt/></p>

<p align="center"><img src="/_posts/tex/c18016a124d12baeae92e4a1e76515fa.svg?invert_in_darkmode&sanitize=true" align=middle width=435.20463194999996pt height=44.5453998pt/></p>

To implement this in code it is useful to first calculate the matrix where the positive elements of the margins matrix are set equal to one. The elements that are not the correct class remain one, while the elements of the correct class are set equal to <p align="center"><img src="/_posts/tex/88c868b0b89f47b675afd9eec55c86f2.svg?invert_in_darkmode&sanitize=true" align=middle width=127.13958015pt height=39.26959575pt/></p>. This matrix is then multiplied by the transpose of the feature matrix.

The reason for the sum term (called class_sums in the code below) is that every margin function includes the scores <p align="center"><img src="/_posts/tex/5ab506490ff70944678b4d80b392f26a.svg?invert_in_darkmode&sanitize=true" align=middle width=23.408644049999996pt height=11.780795399999999pt/></p>, and so they all must be summed.

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
