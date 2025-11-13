---
layout: post
title: Numpy Neural Nets Datasets
permalink: /neural_net_4/
---

## Another Numpy Neural Net: Datasets

The Data is the most important part of training an machine learning model, and also the part that's most difficult to automate. It seems no matter how many 'data pipelines' and 'etl solutions' are out there, at some point you end up having to have an if/elif/else statement that's about 50 clauses long to handle all the different formats and entire-pipeline-wrecking abnormalities that the data might contain. 

I think this is fundamentally because you don't have total control over data the way you do with the rest of the process. Data, if it is useful, comes from the 'real world', the part of the world which is not contained as a bunch of charges and magnetic domains inside a computer. Therefore, as the world is infinite in its complexity, so too are the number of ways that the data can crush your dreams. Anyone who has scanned an excel doc with a column containing missing values has seen something along the lines of

| Important column which is never null | Optional column |
| :-----------------------------------:|:---------------:|
|0|N/A|
|1|NULL|
|2|null|
|3|blank|
|NULL||
|4||

Important note: the last two entries in the right-hand column are blank and a space (' ') and require different handling.

I could go on, these are pretty mundane problems. Most data scientists will probably get some grizzled sea captain look on their face if you ask them to recount the worst data problem they've seen, they're lucky to be alive. All this is to say, here, like most pedagogical projects, I will be using an extremely-battle-tested dataset: MNIST. There's a few other good options, but who doesn't like reading hand written digits?

## Today's Base Class

I tried a few fancier, more generalizable things for this class, but honestly, I have found you almost always have to have an extremely specific set of functions at some point to read your data. So here I'm kind of trying to outline the important bits and things you'll have to think about for your particular application, but things like how you load, get it into some kind of dataframe/array, preprocess, are all going to be very specific to your particular dataset and your particular application.

```python
class Dataset:
    def __init__(self, X: np.ndarray, y: np.ndarray,split: Literal['train','test','validation']='train'):
        self.X = X
        self.y = y
        self.split = split

    def __len__(self):
        return len(self.X)

    def __getitem__(self, idx: int):
        return self.X[idx], self.y[idx]

    def get_batches(self, batch_size: int, shuffle: bool = True):
        indices = np.arange(len(self.X))
        if shuffle:
            np.random.shuffle(indices)
        for start_idx in range(0, len(self.X), batch_size):
            batch_indices = indices[start_idx:start_idx + batch_size]
            logger.debug(f"Yielding batch from index {start_idx} to {start_idx + batch_size}")
            yield self.X[batch_indices], self.y[batch_indices]

    @abstractmethod
    def preprocess_data(self):
        pass
    @abstractmethod
    def preprocess_labels(self):
        pass

    def preprocess(self):
        self.X = self.preprocess_data()
        self.y = self.preprocess_labels()

    def split(self, train_ratio: float):
        split_idx = int(len(self.X) * train_ratio)
        X_train, y_train = self.X[:split_idx], self.y[:split_idx]
        X_val, y_val = self.X[split_idx:], self.y[split_idx:]
        return self.__class__(X_train, y_train, split='train'), self.__class__(X_val, y_val, split='validation')
```
So here we make a Dataset class that takes in an array of data and an array of labels as well as a indication as to whether it is to be considered a train or test set. Now already here's something that is affected by how you get your data. The MNIST set I downloaded from [here](https://huggingface.co/datasets/ylecun/mnist) comes with a train file and a test file. The way I've set up this class makes a little less sense if you just have a big chunk of data that you need to split yourself. I have included a split method for doing a train/val split. It returns two copies of the Dataset that called it but with different slices of the data. You'd need to make it a little different if you wanted k-fold cross validation.

Importantly, the `preprocess_data` and `preprocess_labels` method is applicable to most applications. I haven't encountered a whole lot of data you can just shove into a model raw, in this case it comes as a PIL (python image library) object and you have to turn it into an array of pixels. The labels are the numbers themselves, and no, you can't use those raw either, you have to one-hot-encode them. So that:

```
0 = [1,0,0,0,0,0,0,0,0,0]
1 = [0,1,0,0,0,0,0,0,0,0]
...
9 = [0,0,0,0,0,0,0,0,0,1]
```
Then use `np.argmax` to turn the output of softmax back into a number when you make predictions. But first we need to read a parquet file:

```python
class ParquetDataset(Dataset):
    def __init__(self, file_path: str, feature_cols: list, label_col: str):
        df = pd.read_parquet(file_path)
        X = df[feature_cols].values
        y = df[label_col].values
        super().__init__(X, y)
```

Which is a simple enough embellishment, I could use this for any data that comes to me in a parquet file with a label column. It's not everything, but it's a lot of things. Finally, MNIST:

```python
class MNISTDataset(ParquetDataset):
    def __init__(self, 
                 split: Literal['train','test','validation'] = 'train',
                 feature_cols: List[str] = [ 'image' ],
                 label_col: str = 'label'):
        file_path: str = f'/workspaces/NumpyNN/data/mnist/{split}-00000-of-00001.parquet'
        super().__init__(file_path, 
                         feature_cols,
                         label_col)
        self.split = split
        self.preprocess()

    def preprocess_data(self):
        # Convert byte arrays to numpy arrays
        imgs = self.X[:,0]
        img_arrays = [Image.open(io.BytesIO(img['bytes'])) for img in imgs]
        # Flatten
        flattened_imgs = [np.array(img).reshape(-1) for img in img_arrays]
        # Normalize
        normalized_imgs = np.array(flattened_imgs) / 255.0
        # Return an array of shape (num_samples, 784)
        return np.array(normalized_imgs)
        
    def preprocess_labels(self):
        return np.eye(10)[self.y]  # One-hot encode labels
```

I've got the file in my repo because it's small. You really shouldn't keep them in your repo in general, but [DVC](https://dvc.org/) is the subject of a totally different post. The data and the label both have preprocessing functions and the init method calls them both with `preprocess`. These arrays must be normalized and flattened, which is not true for all datasets and all applications, so it'll need to be changed for a different model architecture.

## A Note on Tests

So the last few posts, copilot and autocomplete have for the most part come through. I more or less write the name of a class and I get large chunks of it taken care of for me, went even faster once I found that I sometimes get two options (like make a placeholder method or actually fill it in). If I feel it's missing a method or a method is missing a parameter, I just type it in, and the rest updates seemlessly. However, I got to this dataset, and it was a bit of a struggle. The stuff that copilot output that worked was kind of a mess because it was trying to account for every possible situation, even after I tried to chat to it a bit about what the situation was. Trying to add the datafile to the context isn't super helpful because in general it's too big of a file, so you need to read the data and make a smaller version to help the llm out, and you might as well just do it yourself at that point.

Which comes around to my point, I love that this thing writes tests for me, but the fact is that they are only good tests if I
1. read them and
1. run them

So here's me reading them. This is a good test of copilot, because I always find trying to write data tests to be particularly annoying for all the reasons mentioned earlier:

```python
@pytest.fixture
def sample_parquet(tmp_path):
    # Create a sample DataFrame
    df = pd.DataFrame({
        'image': [np.random.rand(28*28) for _ in range(5)],
        'label': [0, 1, 2, 3, 4]
    })
    file_path = tmp_path / "sample.parquet"
    df.to_parquet(file_path)
    return str(file_path), df

def test_mnist_train_dataset_init(sample_parquet):
    file_path, df = sample_parquet
    dataset = ParquetDataset(file_path=file_path, feature_cols=['image'], label_col='label')
    assert isinstance(dataset.X, np.ndarray)
    assert isinstance(dataset.y, np.ndarray)
    assert dataset.X.shape[0] == len(df)
    assert dataset.y.shape[0] == len(df)
    # Check that the labels match
    np.testing.assert_array_equal(dataset.y, df['label'].values)
```

First off, good for copilot, using a fixture. It must have read my earlier post. Unlike when it was trying to write the class, it's being super specific about the one we're actually using, down to assuming a 28x28 image. This seems appropriate, this dataset doesn't have surprises in image sizing. It uses the tmp_path builtin pytest fixture so we can test that the `__init__` method is actually loading parquet files, but won't slowly fill up our repo with junk parquet files. The assert statements are reasonable in that it's testing that the X and y attributes are the right type and have the right shape. However, since this is operating on the ParquetDataset, we aren't really getting a test of the `MNISTDataset` class.

I had to ask copilot special to make me something, and a couple different ways, but it did it:

```python
def test_mnist_dataset_pil_images(sample_parquet):
    file_path, df = sample_parquet
    dataset = MNISTDataset(file_path=str(file_path), split='train')
    assert isinstance(dataset.X, np.ndarray)
    assert dataset.X.shape == (5, 28 * 28)
    assert np.issubdtype(dataset.X.dtype, np.floating)
    assert dataset.X.max() <= 1.0 and dataset.X.min() >= 0.0

    assert dataset.y.shape == (5, 10)
    np.testing.assert_array_equal(np.argmax(dataset.y, axis=1), df['label'].values)

```
There you have it, now we can train a model. Again, I find it somewhat annoying to mix up different types of assert statements, but it works!