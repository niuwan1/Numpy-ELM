# Numpy-ELM

## Overview

<div align="center">
    <img src="https://i.imgur.com/GckJu86.png" width=600>
</div>

In this repository, we provide a numpy implementation of Extreme Learning Machine (ELM)
introduced by Huang et al. in this [paper](http://ieeexplore.ieee.org/document/1380068/?reload=true).

ELM is a neural-network-based new learning schieme which can learn fast.
The training will always converge to the global optimal solution,
while ordinary backpropagation-based neural networks have to deal with
the local minima problem.

## Dependencies

We tested our codes using the following libraries.

* Python 3.6.0
* Numpy 1.14.1
* Keras (tensorflow backend) 2.1.5
* Tensorflow 1.7.0

We used Keras only for downloading the MNIST dataset.

You don't have to use exactly the same version of the each library,
but we can not guarantee the codes work well in the case.

All the above libraries can be installed in the following command.

`$ pip install -U numpy Keras tensorflow`

## Usage

Here, we show how to train a OS-ELM module and predict on it.
For the sake of simplicity, we assume to train the model on MNIST, a
hand-written digits dataset.

```python
from keras.datasets import mnist
from keras.utils import to_categorical
from elm import ELM, load_model
import argparse
import os
import numpy as np

parser = argparse.ArgumentParser()
parser.add_argument('--n_hidden_nodes', type=int, default=1024)
parser.add_argument('--loss',
    choices=['mean_squared_error', 'mean_absolute_error'],
    default='mean_squared_error',
)
parser.add_argument('--activation',
    choices=['sigmoid', 'identity'],
    default='sigmoid',
)

def softmax(x):
    c = np.max(x, axis=-1)
    upper = np.exp(x - c)
    lower = np.sum(upper, axis=-1)
    return upper / lower

def main(args):
    # ===============================
    # Load dataset
    # ===============================
    n_classes = 10
    (x_train, t_train), (x_test, t_test) = mnist.load_data()

    # ===============================
    # Preprocess
    # ===============================
    x_train = x_train.astype(np.float32) / 255.
    x_train = x_train.reshape(-1, 28**2)
    x_test = x_test.astype(np.float32) / 255.
    x_test = x_test.reshape(-1, 28**2)
    t_train = to_categorical(t_train, n_classes).astype(np.float32)
    t_test = to_categorical(t_test, n_classes).astype(np.float32)

    # ===============================
    # Instantiate ELM
    # ===============================
    model = ELM(
        n_input_nodes=28**2,
        n_hidden_nodes=args.n_hidden_nodes,
        n_output_nodes=n_classes,
        loss=args.loss,
        activation=args.activation,
        name='elm',
    )

    # ===============================
    # Training
    # ===============================
    model.fit(x_train, t_train)
    train_loss, train_acc = model.evaluate(x_train, t_train, metrics=['loss', 'accuracy'])
    print('train_loss: %f' % train_loss)
    print('train_acc: %f' % train_acc)

    # ===============================
    # Validation
    # ===============================
    val_loss, val_acc = model.evaluate(x_test, t_test, metrics=['loss', 'accuracy'])
    print('val_loss: %f' % val_loss)
    print('val_acc: %f' % val_acc)

    # ===============================
    # Prediction
    # ===============================
    x = x_test[:10]
    t = t_test[:10]
    y = softmax(model.predict(x))

    for i in range(len(y)):
        print('---------- prediction %d ----------' % (i+1))
        class_pred = np.argmax(y[i])
        prob_pred = y[i][class_pred]
        class_true = np.argmax(t[i])
        print('prediction:')
        print('\tclass: %d, probability: %f' % (class_pred, prob_pred))
        print('\tclass (true): %d' % class_true)


if __name__ == '__main__':
    args = parser.parse_args()
    main(args)
```

## Notes

The following figure shows OS-ELM training formula.  

<div align="center">
    <img src="https://i.imgur.com/QjqaMcS.png" width=600>
</div>


* **important**: Since matrix inversion in OS-ELM update formula has a lot of conditional operations, even if it is executed on GPUs, the training is not necessarily accelerated.
* In OS-ELM, you can apply an activation function only to the hidden nodes.
* OS-ELM always finds the global optimal solution for the weight matrices at every training.
* If you feed all the training samples to OS-ELM in the initial training phase,
the computational procedures will be exactly the same as ELM. So, we can consider ELM is a special case of OS-ELM.
* OS-ELM does not need to train iteratively on the same data samples,
while backpropagation-based models usually need to do that.
* OS-ELM does not update 'alpha', the weight matrix connecting the input nodes
and the hidden nodes. It makes OS-ELM train faster.
* OS-ELM does not need to compute gradients. The weight matrices are trained by
computing some matrix products and a matrix inversion.
* The computational complexity for the matrix inversion is about O(batch\_size^3),
so take care for the cost when you increase batch\_size.
## Demo

You can execute the above sample code with the following command.

`$ python train_mnist.py`

## Todos

* support more activation functions
* support more loss functions
* provide benchmark results
