# DCGAN-grayscale-images

Use deep convolutional generative adversarial networks (DCGAN) to generate images in grayscale


## Results at a glance:

Below are image examples generated by my DCGAN model:

**MNIST** (16,000 episodes)

[![DCGAN-MNIST-result.png](https://i.postimg.cc/7ZddCYt8/DCGAN-MNIST-result.png)](https://postimg.cc/3kjnV7Hn)

**Fashion-MNIST** (9,000 episodes)

[![DCGAN-Fashion-MNIST-result.png](https://i.postimg.cc/9fcXnbVQ/DCGAN-Fashion-MNIST-result.png)](https://postimg.cc/Yvyw41h5)

**Kuzushiji-MNIST** (16,000 episodes)

[![DCGAN-KMNIST-result.png](https://i.postimg.cc/bvvRq52B/DCGAN-KMNIST-result.png)](https://postimg.cc/BjdFgMpT)

**Food-101** (episodes)


**Cursive calligraphy** (episodes)





### References

1. Original paper

Radford et al. (2015) **Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks** [[arXiv](https://arxiv.org/abs/1511.06434)]

2. Tutorial on immplementation using MXNet [[link](https://gluon.mxnet.io/chapter14_generative-adversarial-networks/dcgan.html)]

3. Lazy Programmer's GAN (with fully connected layers) Colab Notebook: https://colab.research.google.com/drive/1NGi0HyEuR8cMWyzmGzBiv06PDqmxZddY

4. These posts have some useful information:

- https://machinelearningmastery.com/practical-guide-to-gan-failure-modes/

- https://towardsdatascience.com/gan-ways-to-improve-gan-performance-acf37f9f59b

# Datasets

In this project, 2 datasets were tested:

1. MNIST [[link](https://www.tensorflow.org/datasets/catalog/mnist)]
2. Fashion-MNIST [[link](https://www.tensorflow.org/datasets/catalog/fashion_mnist)]
3. Kuzushiji-MNIST [[link](https://www.tensorflow.org/datasets/catalog/kmnist)]
4. Food-101 [[link](https://www.kaggle.com/kmader/food41)]
5. Cursive calligraphy [[link](https://github.com/nccuviplab/CursiveChineseCalligraphyDataset)]

**MNIST**

MNIST dataset consists of 70,000 (60,000 training and 10,000 testing) images of hand-written digits (0-9). The dataset can be loaded by:

``` python3
## Load the mnist dataset

print("Using MNIST dataset")
print("60,000 images, each image with size 28 x 28 (grayscale)")

train_data, _ = tf.keras.datasets.mnist.load_data(path = "mnist.npz")

orig_data = np.array(train_data[0])

orig_data.shape
```

In the code above, I only loaded the 60,000 training data to be my dataset.


**Fashion-MNIST**

Fashion-MNIST dataset consists of 70,000 (60,000 training and 10,000 testing) images of Zalando's articles (T-shirts, Trousers, Pullovers, ...). The dataset can be loaded by:

``` python3
## Load the fashion-mnist dataset

print("Using Fashion-MNIST dataset")
print("60,000 images, each image with size 28 x 28 (grayscale)")

train_data, _ = tf.keras.datasets.fashion_mnist.load_data()

orig_data = np.array(train_data[0])

orig_data.shape
```

In the code above, I only loaded the 60,000 training data to be my dataset.

**Kuzushiji-MNIST**

Kuzushiji-MNIST dataset consists of 70,000 (60,000 training and 10,000 testing) images of Japanese Hiragana characters. The dataset can be loaded by:

``` python3
## Load the Kuzushiji-mnist dataset
import tensorflow_datasets as tfds

print("Using Kuzushiji-MNIST dataset")
print("60,000 images, each image with size 28 x 28 (grayscale)")
print("\n")

orig_data, _ = tfds.as_numpy(tfds.load('kmnist', split = 'train', batch_size = -1, as_supervised = True))


orig_data.shape
```

In the code above, I only loaded the 60,000 training data to be my dataset.



**Cursive Calligraphy**

Calligraphy dataset: https://github.com/nccuviplab/CursiveChineseCalligraphyDataset




# Hyperparameters

### General principles

These are the general features of DCGAN I used for both (MNIST & Fashion-MNIST) datasets

#### Leaky-Relu activation
The original paper suggested using ReLU for the generator, while using LeakyRelu activation in the discriminator. However, I used LeakyRelu (`alpha = 0.2`) in **both generator and discriminator**.

#### Batch normalization
Contrary to the original paper, I only used batch normalizations in **hidden Conv2DTranspose layers** in the **generator**. I didn't use batch normalization in the discriminator.

#### Weight initialization
As recommended by the original DCGAN paper, I initialized the wieghts by normal distribution (stddev = 0.02)

``` python3
from tensorflow.keras.initializers import RandomNormal

w_init = RandomNormal(mean = 0.0, stddev = 0.02)
```

#### Batch size
In each epoch, the generator created 32 fake images. These 32 fake images and 32 real images sampled from the dataset will be used to train the discriminator. After then, a new batch of 32 fake images will be used to train the generator.

#### Learning rate
I used learning rate = 1e-5 for both the generator and the discriminator. 

#### Label smoothing
Label smoothing can be applied by setting `label_smoothing` during model compiling:

``` python3
discriminator.compile(loss = tf.keras.losses.BinaryCrossentropy(label_smoothing = 0.1), optimizer = Adam(learning_rate = 1e-5))
DCGAN.compile(loss = tf.keras.losses.BinaryCrossentropy(label_smoothing = 0.1), optimizer = Adam(learning_rate = 1e-5))
```

# Model architectures

## MNIST and Fashion-MNIST

**Generator**

``` python3
def build_generator_DC(hidden_dim = 100):
    
    ## weight initialization
    w_init = RandomNormal(mean = 0.0, stddev = 0.02)

    ## input vector
    z = Input(shape = (hidden_dim, )) # (None, hidden_dim)
    
    ## Project and reshape
    n = H // 4 # H = height of the image
    x = Dense(n*n*128, kernel_initializer = w_init)(z)
    x = LeakyReLU(alpha = 0.2)(x)
    x = Reshape((n, n, 128))(x)

    ## Conv2D-T
    x = Conv2DTranspose(256, 4, 2, 'same', kernel_initializer = w_init)(x)
    x = BatchNormalization()(x)
    x = LeakyReLU(alpha = 0.2)(x)

    ## Conv2D-T
    x = Conv2DTranspose(256, 4, 2, 'same', kernel_initializer = w_init)(x)
    x = BatchNormalization()(x)
    x = LeakyReLU(alpha = 0.2)(x)
        
    ## last Conv2D (no batch norm!)
    img = Conv2D(1, 4, padding = 'same', activation = 'tanh', kernel_initializer = w_init)(x)
    
    # generator model
    model = Model(inputs = z, outputs = img, name = 'generator')

    return model
```

**Discriminator**

``` python3
def build_discriminator_DC():
    
    ## weight initialization
    w_init = RandomNormal(mean = 0.0, stddev = 0.02)

    ## input image (img)
    img = Input(shape = (H, H, 1)) # H = height of the image
    
    ## Conv
    x = Conv2D(256, 4, 2, padding = 'same', kernel_initializer = w_init)(img)
    x = LeakyReLU(alpha = 0.2)(x)

    ## Conv
    x = Conv2D(256, 4, 2, padding = 'same', kernel_initializer = w_init)(x)
    x = LeakyReLU(alpha = 0.2)(x)

    ## Conv
    x = Conv2D(256, 4, 2, padding = 'same', kernel_initializer = w_init)(x)
    x = LeakyReLU(alpha = 0.2)(x)

    ## final layer
    x = Flatten()(x)
    y = Dense(1, activation = 'sigmoid')(x)

    # generator model
    model = Model(inputs = img, outputs = y, name = 'discriminator')

    return model
```


### Related papers and articles

**DCGAN**

[[link1](http://doi.org/10.23915/distill.00003)]\
[[link2](https://arxiv.org/abs/1511.06434)]\


**Wasserstein GAN** [[link1](https://arxiv.org/abs/1701.07875)], [[link2](https://arxiv.org/abs/1704.00028)], [[link3](https://lilianweng.github.io/lil-log/2017/08/20/from-GAN-to-WGAN.html)]

**Spectral Normalization** [[link](https://arxiv.org/abs/1802.05957)]

**Conditional GAN** Conditional Generative Adversarial Nets (Mirza and Osindero, 2014) [[link](https://arxiv.org/abs/1411.1784)]

**Controllable GAN** Interpreting the Latent Space of GANs for Semantic Face Editing (Shen, Gu, Tang, and Zhou, 2020)[[link](https://arxiv.org/abs/1907.10786)]

**Face Editting** Interpreting the Latent Space of GANs for Semantic Face Editing (Shen, Gu, Tang, and Zhou, 2020) [[link](https://arxiv.org/abs/1907.10786)]

**Machine Bias** [[link](https://www.propublica.org/article/machine-bias-risk-assessments-in-criminal-sentencing)]

### Interesting datasets

**CelebFaces** http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html

