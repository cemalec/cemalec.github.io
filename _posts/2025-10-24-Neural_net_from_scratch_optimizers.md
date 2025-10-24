---
layout: default
title: Numpy Neural Nets Optimizers
permalink: /neural_net_3/
---

## Another Numpy Neural Net: Optimizers

We have layers, but currently they consist of a weight and bias matrix with randomly chosen values. If we send an input through this, it will produce an essentially random output. So we need a way to improve the values of the layer weights, this process is called training and training affects the values of the weights and biases in our model through data and optimization.  Usually when one begins their machine learning journey, they begin with gradient descent, and it's nearly identical cousin stocastic gradient descent (using random batches to optimize). But there are other optimization algorithms out there that have proven even more successful than our pal SGD, but they are all more or less variations on [Newton's Method](https://en.wikipedia.org/wiki/Newton%27s_method). If you know where you are and where you're going you can figure out where you should be, at least if you take a small enough step.

## Base class

Not a whole lot to this one, just need to enforce the methods we'll need because like layers, the optimizer will be used by the model, so we have to make sure that all optimizers have certain common functionalities, expect certain inputs, and produce certain outputs.

```python
from abc import abstractmethod
from typing import Dict,Any
import numpy as np

class Optimizer:
    def __init__(self):
        self.name = None
        self.type = 'Optimizer'

    @abstractmethod
    def step(self, layer: Any, grads: Dict[str,np.ndarray]) -> np.ndarray:
        pass
    
    @abstractmethod
    def to_dict(self) -> Dict[str,Any]:
        pass
    
    @classmethod
    @abstractmethod
    def from_dict(cls, config: Dict[str,Any]) -> 'Optimizer':
        pass

```

You see this looks remarkably like the Layer base class, but the main method we need to implement in subclasses is `step`, which is where the weights and biases of the layer passed to it are updated using the gradients calculated in the layer's backward pass. Now we make the SGD class:

```python
class SGD(Optimizer):
    def __init__(self, learning_rate: float):
        super().__init__()
        self.learning_rate = learning_rate
        self.type = 'SGD'
    
    def step(self, layer, grads: np.ndarray) -> np.ndarray:
        params={'weights': layer.weights,'biases': layer.biases} 
        for key in ['weights','biases']:
            params[key] -= self.learning_rate * grads[key]
    
    def to_dict(self) -> dict:
        return {'learning_rate': self.learning_rate}
    
    @classmethod
    def from_dict(cls, data: dict):
        return cls(learning_rate=data['learning_rate'])
```
As we'll see this is dead simple compared to other optimizers, and is special in that it is *stateless*. No matter what step you are in in the training process or what the previous step was, the optimization algorithm proceeds the same. For each layer, we get the weights and biases, we subtract the relevant gradient multiplied by the learning rate, and return the new weights and biases. The optimizer itself also only has one parameter, the learning rate.

But there are others! Notably, Adam, which is many people's go-to deep learning optimizer. Adam adds a *momentum* term that smooths out rapid changes in the gradient. In practice this leads to much faster training for many neural nets.

```python
class Adam(Optimizer):
    def __init__(self, 
                 learning_rate: float, 
                 beta1: float = 0.9, 
                 beta2: float = 0.999, 
                 epsilon: float = 1e-8):
        super().__init__()
        self.type = 'Adam'
        self.learning_rate = learning_rate
        self.beta1 = beta1
        self.beta2 = beta2
        self.epsilon = epsilon
        self.m = dict()
        self.v = dict()
        self.t = 0
    
    def initialize_state(self,layer: Any):
        initial_dict = dict(weights = np.zeros_like(layer.weights),
                            biases = np.zeros_like(layer.biases))
        self.m[layer.name] = initial_dict.copy()
        self.v[layer.name] = initial_dict.copy()

    def step(self,
             layer: Any,
             grads: Dict[str,np.ndarray]) -> np.ndarray:
        if self.m.get(layer.name) is None:
            self.initialize_state(layer)
        params = {'weights': layer.weights, 'biases': layer.biases}
        if self.t < 1000:
            self.t += 1
        for key in ['weights', 'biases']:
            self.m[layer.name][key] = self.beta1 * self.m[layer.name][key]  + (1 - self.beta1) * grads[key]
            self.v[layer.name][key]  = self.beta2 * self.v[layer.name][key]  + (1 - self.beta2) * (grads[key] ** 2)
            m_hat = self.m[layer.name][key] / (1 - self.beta1 ** self.t)
            v_hat = self.v[layer.name][key] / (1 - self.beta2 ** self.t)
            params[key] -= self.learning_rate * m_hat / (np.sqrt(v_hat) + self.epsilon)
    
    def to_dict(self) -> dict:
        return {'beta1': self.beta1, 
                          'beta2': self.beta2, 
                          'epsilon': self.epsilon,
                          'm': self.m,
                          'v': self.v,
                          't': self.t}
    
    @classmethod
    def from_dict(cls, data: dict):
        obj = cls(learning_rate=data['learning_rate'],
                  beta1=data.get('beta1', 0.9),
                  beta2=data.get('beta2', 0.999),
                  epsilon=data.get('epsilon', 1e-8))
        obj.m = data.get('m', None)
        obj.v = data.get('v', None)
        obj.t = data.get('t', 0)
        return obj
```

The big difference here is that the optimizer has a *state* which is important when it comes time to save the model and train it further later. I've been bitten many times by not properly saving the optimizer state and going to train a model for additional epochs only to find it quickly turn into garbage and I have to waste time retraining from scratch.  The m (momentum) and v (variance) terms have values associated both with the weights and with the biases and are updated for each layer at each step, and furthermore early steps (t) in the training process are attenuated by a factor of `(1-beta1**t)` for m and `(1-beta2**t)` for v. It's a little more complicated, but once you have your optimizer, it's really no harder to use than any other one, and it often works a lot better.

Well that's the optimizer. I lied in the last post, there's actually one more incredibly important component before the model. It's the Dataset! Then we'll make a model, I promise.