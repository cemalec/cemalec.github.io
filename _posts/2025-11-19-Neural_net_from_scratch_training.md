---
layout: post
title: Numpy Neural Nets Training
permalink: /neural_net_6/
---

## Another Numpy Neural Net: Training Loop

Finally, we need to actually train this model. You usually have two choices, which is to either make a training function, or supply a .train() method to your model. Either is fine, I will say that for the purposes of this exercise, it's a little easier to deal with a training function. You don't have to keep track of which things your model has inside of it and which it doesn't.

First we have some configuration global variables and logging setup
```python
# Model configuration
INPUT_SIZE = 28 * 28
HIDDEN_SIZE1 = 64
HIDDEN_SIZE2 = 32
OUTPUT_SIZE = 10

# Create logger
logger = logging.getLogger(__name__)
```

Next we define the training function, it takes in a Model and a Dataset, as well as values for epochs, batch_size, and learning_rate. Each epoch effectively reshuffles the dataset and runs through all the examples again. Within each epoch, for each batch there is a:
1. Forward pass
1. Loss computation
1. Backward pass (to compute gradients and update weights/biases)
1. Metric computation

Each epoch I log the current log_loss and accuracy averaged over the batches.

```python
def training_loop(
    model: Model, dataset: Dataset, epochs: int, batch_size: int, learning_rate: float
):
    for epoch in range(epochs):
        i = 0
        batch_accuracies = []
        batch_losses = []
        for x_train, y_train in dataset.get_batches(batch_size):
            # Forward pass
            y_pred = model.forward(x_train)

            # Compute loss
            loss = model.compute_loss(y_train, y_pred)
            batch_losses.append(loss)

            # Backward pass (weights update as part of the optimizer step in Model.backward)
            model.backward(y_train, y_pred, learning_rate)

            # Compute accuracy
            batch_acc = accuracy(y_train, y_pred)
            batch_accuracies.append(batch_acc)
            i += 1
        avg_acc = np.mean(batch_accuracies)
        avg_loss = np.mean(batch_losses)
        logger.info(
            f"Epoch {epoch+1}/{epochs}, Loss: {avg_loss:.4f}, Avg Accuracy: {avg_acc:.4f}"
        )
```

Which seems a lot simpler than everything else, but that is in part because the model has a few different important methods like .forward(), .backward(), and .calculate_loss() that make everything at this stage very compact. However, you may be asking 'when did we actually construct the model.' Well the training_loop.py file is not done yet, there's also a main function that executes when we call the script.

This allows arguments to be parsed from the command line so you can type something like
```sh
python training_loop.py --epochs 200 --learning_rate 5e-6 --log-level DEBUG
```

which we will take advantage of in the next post.  After the arguments are parsed, we load the data in, construct our model, and call the model. After that, we save the model and log the training/test losses and accuracies.

```python
if __name__ == "__main__":
    argparser = argparse.ArgumentParser(
        description="Train a simple neural network on MNIST"
    )
    argparser.add_argument(
        "--epochs", type=int, default=20, help="Number of epochs to train"
    )
    argparser.add_argument(
        "--batch_size", type=int, default=32, help="Batch size for training"
    )
    argparser.add_argument(
        "--learning_rate", type=float, default=1e-3, help="Learning rate for optimizer"
    )
    argparser.add_argument("--log_level", type=str, default="INFO")
    # Parse command line arguments
    args = argparser.parse_args()
    epochs = args.epochs
    batch_size = args.batch_size
    learning_rate = args.learning_rate

    # Configure logging
    logging.basicConfig(
        level=getattr(logging, args.log_level.upper(), None),
        format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    )
    logger.info(
        f"Starting training for {epochs} epochs with batch size {batch_size} and learning rate {learning_rate}"
    )

    # Define Model
    basic_model = Model(
        layers=[
            DenseLayer(
                input_size=INPUT_SIZE,
                output_size=HIDDEN_SIZE1,
                activation_function=ReLU(),
            ),
            DenseLayer(
                input_size=HIDDEN_SIZE1,
                output_size=HIDDEN_SIZE2,
                activation_function=ReLU(),
            ),
            DenseLayer(
                input_size=HIDDEN_SIZE2,
                output_size=OUTPUT_SIZE,
                activation_function=SoftMax(),
            ),
        ],
        loss=CrossEntropyLoss(),
        optimizer=Adam(learning_rate=learning_rate),
    )

    # Load Dataset
    train_dataset = MNISTDataset(split="train")
    test_dataset = MNISTDataset(split="test")
    X_train, y_train = train_dataset.X, train_dataset.y
    X_test, y_test = test_dataset.X, test_dataset.y
    print("Training data shape:", X_train.shape, y_train.shape)
    print("Testing data shape:", X_test.shape, y_test.shape)

    # Train Model
    training_loop(
        model=basic_model,
        dataset=train_dataset,
        epochs=epochs,
        batch_size=batch_size,
        learning_rate=learning_rate,
    )

    # Save Model
    basic_model.save("models/bigger_model.npz")

    # Evaluate on training set
    y_train_pred = basic_model.predict(X_train)
    train_loss = basic_model.compute_loss(y_train, y_train_pred)
    logging.info(f"Train Loss: {train_loss:.4f}")
    logging.info(f"Train Accuracy: {accuracy(y_train, y_train_pred):.4f}")

    # Evaluate on test set
    y_test_pred = basic_model.predict(X_test)
    test_loss = basic_model.compute_loss(y_test, y_test_pred)
    test_accuracy = accuracy(y_test, y_test_pred)
    logging.info(f"Test Loss: {test_loss:.4f}")
    logging.info(f"Test Accuracy: {test_accuracy:.4f}")
```

And there you have it, I can run this and get a relatively accurate mnist model.