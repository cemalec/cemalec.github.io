---
layout: default
title: Numpy Neural Nets Layers
permalink: /neural_net_2/
---

## Another Numpy Neural Net: Layers

Deep Learning is built on layers, you input numbers on one side and the crank turns forward from layer to layer until it spits numbers out on the other side. Then the crank turns backwards and a loss out the other side spits a derivative on one side.  This incredible modularity is both necessary (you have to introduce a non-linearity periodically or all your operations can be combined into one linear transformation) and probably one reason why the field moves so fast. So let's start with the classic, fully connected layer, we'll loop back and do a couple more layers once we have the rest of the machinery in place.

## A Base Class

But first, a Base Class:

```python
from DifferentiableFunction import DifferentiableFunction
from typing import Dict
from abc import abstractmethod

class Layer:
    def __init__(self):
        self.name = None
        self.type = 'Layer'
        self.weights = None
        self.biases = None
        self.last_input = None
        self.last_z = None
        self.weights_initialized = False

    @abstractmethod
    def initialize_weights(self):
        pass

    @abstractmethod
    def forward(self, input_data: np.ndarray) -> np.ndarray:
        if self.weights_initialized is False:
            self.initialize_weights()
            self.weights_initialized = True

    @abstractmethod
    def backward(self, output_gradient: np.ndarray) -> Dict[str,np.ndarray]:
        pass

    @abstractmethod
    def to_dict(self) -> Dict:
        pass
    @classmethod
    @abstractmethod
    def from_dict(cls, data: Dict) -> 'Layer':
        pass
```
The Layer Class defines what we would like all layers to have, and since layers are generally used by other objects, certain methods we need the layer to have.  So all layers have certain attributes specified in the `__init__` method, we can call this from subclasses and not wonder if we forgot something later. The point of the `@abstractmethod` decorator is that if a subclass does not implement the method underneath it, python will throw a `NotImplemented` error. There's a number of them, because if a layer is missing any of these, it won't work! It needs a forward pass, a backward pass, a way to initialize weights and a to/from_dict method. The `@classmethod` decorator means that the method can be called from the class rather than an instance (`Layer.from_dict(kwargs)` instead of `Layer(kwargs).from_dict(kwargs)`), this makes a lot of sense for 'load' type functions, where is seems silly to try to make some kind of dummy instance so you can instantiate the real instance.

The last two 'dict' methods are really so that we can specify these models as a yaml or json file instead of inline. This can be very useful when you want to separate the code to run/train your model from the actual model architecture. You don't want to be pushing code to the repo just to try out a different number of hidden nodes.

## Subclass

So now we're ready to code up our Dense layer:
```python
class DenseLayer(Layer):
    """
    A fully connected neural network layer.
    
     Parameters:
        input_size (int): The number of input features.
        output_size (int): The number of output features (neurons).
        activation_function (DifferentiableFunction): The activation function to apply.
        name (str, optional): Name of the layer. Defaults to None.
    """

    def __init__(self, 
                 input_size:int,
                 output_size: int, 
                 activation_function: DifferentiableFunction,
                 name: str = None):
        super().__init__()
        self.name = name
        self.type = 'Dense'
        self.input_size = input_size
        self.output_size = output_size
        self.activation_function = activation_function

    def initialize_weights(self):
        """
        initialize weights with He initialization.
        """
        self.weights = np.random.randn(self.input_size, self.output_size) * (np.sqrt(2./self.input_size))
        self.biases = np.zeros(self.output_size)
        logger.info(f"Weights and biases initialized for layer {self.name}")

    def forward(self, input_data: np.ndarray) -> np.ndarray:
        """
        Performs the forward pass through the layer, supporting batched input.
        Parameters:
            input_data (np.ndarray): Input data to the layer. Shape: (batch_size, input_size)

        Returns:
            np.ndarray: Output after applying weights, biases, and activation function. Shape: (batch_size, output_size)
        """
        super().forward(input_data)
        self.last_input = input_data
        self.last_z = np.dot(input_data, self.weights) + self.biases  # (batch_size, output_size)
        logger.debug(f"Forward pass in layer {self.name}: input shape {input_data.shape}, z shape {self.last_z.shape}")
        return self.activation_function.function(self.last_z)

    def backward(self, output_gradient: np.ndarray) -> Dict[str,np.ndarray]:
        """
        Performs the backward pass through the layer, updating weights and biases.
        Parameters:
            output_gradient (np.ndarray): Gradient of the loss with respect to the layer's output. Shape: (batch_size, output_size)
            learning_rate (float): Learning rate for weight updates.
        Returns:
            np.ndarray: Gradient of the loss with respect to the layer's input. Shape: (batch_size, input_size)
            """
        activation_gradient = self.activation_function.derivative(self.last_z)  # (batch_size, output_size)
        delta = output_gradient * activation_gradient  # (batch_size, output_size)

        weight_gradient = np.dot(self.last_input.T, delta) / self.last_input.shape[0]  # (input_size, output_size)
        bias_gradient = np.sum(delta, axis=0) / self.last_input.shape[0] # (output_size,)

        input_gradient = np.dot(delta, self.weights.T)  # (batch_size, input_size)
        logger.debug(f"Backward pass in layer {self.name}: output_gradient shape {output_gradient.shape}, input_gradient shape {input_gradient.shape}") 
        grad_dict = {'inputs': input_gradient,
                     'weights': weight_gradient,
                     'biases': bias_gradient}
        
        return grad_dict

        # Update weights and biases will be handled by the optimizer in Model.py

    def to_dict(self) -> Dict:
        return {
            'name': self.name,
            'type': self.type,
            'input_size': self.input_size,
            'output_size': self.output_size,
            'activation_function': self.activation_function.__class__.__name__,
        }
    
    @classmethod
    def from_dict(cls, data: Dict) -> 'DenseLayer':
        activation_function = getattr(__import__('DifferentiableFunction'), data['activation_function'])()
        layer = cls(input_size=data['input_size'],
                    output_size=data['output_size'],
                    activation_function=activation_function,
                    name=data['name'])
        return layer
```

The call to `super().__init__()` initializes all our default variables we'll need to do forward/backward passes. We update `self.type` to the new layer type, and implement all the abstract methods.

`forward` simply does a matrix multiplication of the weights with the inputs and adds the biases, then applies the activation function.  The first time forward pass is called, it initializes the weights by calling the base class' forward method

`backward` calculates the relevant gradients and packages them up into a dictionary to be used by the optimizer. Note we don't actually update the weights on the forward or backward pass partly to decrease the number of objects the layer needs to depend on.

Note that I also through some logging in here. Putting logging inside your classes is usually doing yourself a real solid later when some indeterminate layer is ruining your life come training time. It's also way better than a `print` statement because you don't have to remember to erase it (I feel like a n00b every time a colleague points out a print I missed in a PR review). To put logging in just throw this on top of any file:

```python
import logging

logger = logging.getLogger(__name__)
logger.INFO("This is a log message at loglevel INFO")
```

Finally, the to/from dict methods make it possible to save/load a layer from a dictionary like form such as json or yaml.

Next up we'll look at Optimizers and then we'll be ready to put it all together into a model.