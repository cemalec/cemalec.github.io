---
layout: post
title: Numpy Neural Nets Models
permalink: /neural_net_5/
---

## Another Numpy Neural Net: Model

Finally, we get to make a model. The model is essentially a bunch of layers which are in turn arrays of weights and biases. It is also the method by which the weights and biases are optimized. It is thirdly the data that gives the optimization method fuel to optimize.
So to define our model, we will need
1. A list of layers
1. An optimizer
1. A loss function

It's not common practice to carry the training data around with the model, but it is terribly important to keep track of what data was used to train the model. In our mnist set this is done by using the 'train' file only for training.

```python
class Model:
    layers: List[DenseLayer]
    loss: DifferentiableFunction
    optimizer: Optimizer

    def __init__(
        self,
        layers: List[DenseLayer],
        loss: DifferentiableFunction,
        optimizer: Optimizer,
    ):
        self.layers = layers
        self.loss = loss
        self.optimizer = optimizer
        for layer in self.layers:
            if layer.name is None:
                layer.name = f"Layer_{self.layers.index(layer)}"

    def forward(self, x: np.ndarray) -> np.ndarray:
        for layer in self.layers:
            x = layer.forward(x)
        return x

    def backward(self, y_true: np.ndarray, y_pred: np.ndarray):
        loss_grad = self.loss.derivative(y_true, y_pred)
        grad_dict = {"inputs": loss_grad}
        for layer in reversed(self.layers):
            grad_dict = layer.backward(grad_dict["inputs"])
            self.optimizer.step(layer, grad_dict)

    def predict(self, x: np.ndarray) -> np.ndarray:
        return self.forward(x)

    def compute_loss(self, y_true: np.ndarray, y_pred: np.ndarray) -> float:
        return np.mean(self.loss.function(y_true, y_pred))
```
