---
layout: default
title: Numpy Neural Nets
permalink: /neural_net_1/
---
# Another Numpy Neural Net
There's a number of resources on creating a neural net "from scratch" out there. I put the from scratch in quotes because numpy is a lot of machinery to start with, but I think it's a fair label because numpy *looks* about as close to the abstract math that powers these things as you can get. My goal here is to demonstrate development fundamentals I've picked up as a scientist-turned-dev, and another was to see just how much co-pilot could do for me on a task that I know is well represented in the training set. The answer short answer is, quite a bit, but you really need to know what you're doing or it'll get you into trouble fast.

# The Bedrock of Deep Learning: Differentiable Functions
A well known but under-appreciated fact about most machine learning models is that the very important loss/activation functions have to be differentiable. So I could have made an Activation class and a Loss class first, but for pedagogical reasons, I made a DifferentiableFunction class first. The first subclass is of course our friend softmax.  I just typed in the class names, and tabbed my way to success on these, including type hints and function signatures for the callables (highly recommended).

```python
import numpy as np
from typing import Callable
class DifferentiableFunction:
    function: Callable[[np.ndarray], np.ndarray]
    derivative: Callable[[np.ndarray], np.ndarray]

    def __init__(self, function: Callable[[np.ndarray], np.ndarray], derivative: Callable[[np.ndarray], np.ndarray]):
        self.function = function
        self.derivative = derivative

class SoftMax(DifferentiableFunction):
    def __init__(self):
        def softmax(x: np.ndarray) -> np.ndarray:
            e_x = np.exp(x - np.max(x, axis=-1, keepdims=True))
            return e_x / e_x.sum(axis=-1, keepdims=True)

        def softmax_derivative(x: np.ndarray) -> np.ndarray:
            s = softmax(x)
            return s * (1 - s)

        super().__init__(softmax, softmax_derivative)
```

the `super` that appears in SoftMax calls the method from the parent class, so after defining the softamx function and its derivative, the class instantiates an instance of DifferentiableFunction with those two functions.

I also tried to get copilot to write me some tests, which for this task were pretty all right, but testing numerical functions is very straightforward in that you can give it some inputs (though I should include 0) and see if it outputs what it should. Also, though I see plenty of devs with more years under their belt than me do this, I feel like it could have made a fixture to house the test data since it's going to be used over and over again. However, there is something to be said for having it *right there* in the test.

```python
import numpy as np
from DifferentiableFunction import DifferentiableFunction, SoftMax

def test_softmax_function():
    sm = SoftMax()
    x = np.array([1.0, 2.0, 3.0])
    result = sm.function(x)
    expected = np.exp(x - np.max(x)) / np.sum(np.exp(x - np.max(x)))
    np.testing.assert_allclose(result, expected, rtol=1e-6)

def test_softmax_derivative_shape():
    sm = SoftMax()
    x = np.array([1.0, 2.0, 3.0])
    deriv = sm.derivative(x)
    assert deriv.shape == x.shape
```

Another note on the tests, this is an area where you need to pay for the better copilot plan to get more than a tiny number of tests written for you, but I might. Even though as we will see later, the tests become less impressive as our tasks become more abstract, the fact that the computer did the gruntwork of writing some test functions and later a test class, including throwing some sample data in there is really nice. It needs to be done, it's not hard to do, and I hate doing it, so fine Microsoft, take my money.

In the next section, we will make a Layer class!