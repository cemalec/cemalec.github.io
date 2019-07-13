---
title: Gradients and You- Introduction and Notation
layout: default
permalink: /Gradients_Notation/
---

Finding analytic gradients can be a daunting task for many. The good news is that most analytic gradients used routinely in machine learning have already been solved and implemented at blazing fast speeds. However, if you want to 'get under the hood' of machine learning algorithms, you need to become familiar with how to calculate these objects. Between this and subsequent posts, I will try to walk you through the math behind the gradients.

### Notation

A few facts make it easier to translate mathematics into code. When calculating the gradient with respect to a matrix, you're really calculating the derivative of a function with respect to an arbitrary element of the matrix.  We refer to an arbitrary element of matrix $X$ as $X_{ij}$, which is the $i$th row and $j$th column.

Two common operations are element-wise multiplication and matrix multiplication. Both can be represented as operations on an arbitrary element of the matrix. For element-wise multiplication we write that

$C_{ij} = A_{ij}B_{ij}$  (element-wise product)

This obviously only works if all matrices have the same dimensions. Matrix multiplication is a type of inner product, meaning that the matrices share a common dimension which is summed over and eliminated.  We represent matrix multiplication by arbitrary elements like this

$C_{ij} = \sum\limits_k A_{ik}B_{kj}$ (matrix product)

Note that $C_{ij} \neq \sum\limits_k A_{ik}B_{jk}$. The columns of the first matrix must match the rows of the second. However, we can remedy a situation like this by using the transpose of a matrix, since $X_{ij} = X^T_{ji}$.

Finally, there are times when we must denote a single element of a matrix. A matrix that is one at row $i$ and column $j$ and zero elsewhere is written 

$\delta_{ij}$ (unit matrix)

### The Chain Rule

One of the most useful theorems in calculus is the chain rule. It allows you to calculate complicated derivatives by breaking it into pieces. Especially when computers are involved, breaking things into smaller pieces is almost always the right move. In addition, the chain rule features prominently in the back-propagation used in training neural networks.

Essentially, what the chain rule says is that we can treat partial differentials like fractions, so that

$\dfrac{\partial f}{\partial x} = \dfrac{\partial f}{\partial y}\dfrac{\partial y}{\partial x}$

So for example if we wanted to take the derivative of $f(x) = ln(x^2)$ could take the derivative of $f(y) = ln(y)$ and the derivative of $y = x^2$ so that

$\dfrac{\partial f}{\partial x} = \dfrac{\partial ln(x^2)}{\partial x^2}\dfrac{\partial x^2}{\partial x} = \dfrac{1}{x^2}2x = \dfrac{2}{x}$

### Numpy

Numpy is a huge package for dealing with arrays and vectors. For the purpose of finding gradients, I'm going to reduce numpy to four operations: indexing, summing, multiplying, and broadcasting.

Numpy indexing is fairly straightforward, you specify an element of a numpy array by specifying its row and column, for example the element $X_{ij}$ is represented by

```python
X[i,j]
```

An entire row or column is specified by

```python
#column
X[:,j]
#row
X[i,:]
```

An important note about indexing is that numpy vectors are _neither_ row nor column vectors. This has important implications for broadcasting.

Summations are also relatively straightforward. When summing arrays, it is important to specify the _axis_ you are summing over. If you are summing over the rows (leaving you with a single row) use

```python
np.sum(X,axis = 0)
```

In order to sum over the columns (leaving you with a single column) use
```python
np.sum(X,axis = 1)
```

And in order to sum over both, simply use

```python
np.sum(X)
```

Multiplication comes in two flavors. Element-wise multiplication is represented by the * operator. Matrix multiplication is achieved by the @ operator.

Finally, broadcasting is the powerful numpy tool that allows you to add or multiply incompatible arrays together. Normally, a nxm matrix A cannot be added to a nx1 vector B, however, numpy assumes that the column vector is actually m copies of B, thus 

```python
A + B
```
results in B being added to each column of A and

```python
A * B
```

results in B multiplying each column of A element-wise.

The same can be done with row vectors. Scalars will be added or multiplied to each element of an array.

Now earlier, I pointed out that numpy vectors were neither row nor column vectors, so sometimes it is necessary to force them to be row or column vectors by adding an axis of length 1.  To make the numpy vector y into a row vector input

```python
y[np.newaxis,:]
```

and to create a column vector input

```python
y[:, np.newaxis]
```

Now that that's out of the way, let's take a look at [SVM hinge loss](/Gradients_Hingeloss)
