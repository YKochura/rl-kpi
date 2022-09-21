class: middle, center, title-slide

# Навчання з підкріпленням

Лекція 2: Згорткові мережі

<br><br>
Кочура Юрій Петрович<br>
[iuriy.kochura@gmail.com](mailto:iuriy.kochura@gmail.com) <br>
<a href="https://t.me/y_kochura">@y_kochura</a> <br>


---
class: middle,
# Сьогодні

Розуміння згорткових нейронних мереж (*convnets* або *СNNs*)

- Повнозв'язна vs згорткова мережа
- Операція згортки
- Крок згортки
- Ефект доповнення (padding)
- Операція агрегації (pooling)

---

class: blue-slide, middle, center
count: false

.larger-xx[Повнозв'язна мережа]

---

class: middle

# MNIST: приклади

.center.width-100[![](figures/lec2/mnist-samples.png)]

.success[**Примітка!** У машинному навчанні для задачі класифікації категорія даних називається *класом*. Кожна одиниця даних називається
прикладом. Клас, який пов’язаний із певним прикладом називається *міткою (label)*.]

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
We’re about to dive into the theory of what convnets are and why they have been so successful at computer vision tasks. But first, let’s take a practical look at a densely connected network example that classifies MNIST digits. Don’t worry if some steps seem arbitrary or look like magic to you! We’ve got to start somewhere.

The problem we’re trying to solve here is to classify grayscale images of handwritten digits (28 × 28 pixels) into their 10 categories (0 through 9). We’ll use the MNIST dataset, a classic in the machine learning community, which has been around almost as long as the field itself and has been intensively studied. It’s a set of 60,000 training images, plus 10,000 test images, assembled by the National Institute of Standards and Technology (the NIST in MNIST) in the 1980s. You can think of “solving” MNIST as the “Hello World” of deep learning &mdash; it’s what you do to verify that your algorithms are working as expected. As you become a machine learning practitioner, you’ll see MNIST come up over and over again in scientific papers, blog posts, and so on. You can see some MNIST samples
on this slide.

---

class: middle

# Імпортування набору даних MNIST у Keras

.center.width-100[![](figures/lec2/load-mnist-keras.png)]

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

- *train_images* та  *train_labels* &mdash; навчальний набір даних (дані на яких
модель буде навчатись)

-  *test_images* та *test_labels* &mdash; тестовий набір (дані на яких буде оцінено продуктивність моделі)

???
The MNIST dataset comes preloaded in Keras, in the form of a set of four NumPy arrays.

---

class: middle

# Навчальний набір

.center.width-100[![](figures/lec2/train-data.png)]

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]


???
The images are encoded as NumPy arrays, and the labels are an array of digits, ranging from 0 to 9. The images and labels have a one-to-one correspondence.

---

class: middle

# Тестовий набір

.center.width-100[![](figures/lec2/test-data.png)]

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

---

class: middle

# Архітектура мережі

.center.width-90[![](figures/lec2/dense-net-arch.png)]

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

Робочий процес буде таким: спочатку ми передамо нейронній мережі навчальні дані *train_images* та *train_labels*. Таким чином мережа навчиться пов’язувати зображення з мітками. Потім ми попросимо мережу створити прогнози для *test_images* та перевіримо, чи відповідають ці прогнози міткам з *test_labels*.

???
The core building block of neural networks is the *layer*. You can think of a layer as a filter for data: some data goes in, and it comes out in a more useful form. Specifically, layers extract representations out of the data fed into them &mdash; hopefully, representations that are more meaningful for the problem at hand. Most of deep learning consists of chaining together simple layers that will implement a form of progressive data distillation. A deep learning model is like a sieve for data processing, made of a succession of increasingly refined data filters &mdash; the layers.

Here, our model consists of a sequence of two *Dense* layers, which are densely connected (also called *fully connected*) neural layers. The second (and last) layer is a 10-way *softmax classification* layer, which means it will return an array of 10 probability scores (summing to 1). Each score will be the probability that the current digit image belongs to one of our 10 digit classes.

---

class: middle

# Готуємо мережу до навчання

Щоб підготувати мережу до навчання, нам потрібно визначити на етапі *компіляції*:

- *Оптимізатор* &mdash; алгоритм за допомогою якого модель оновлюватиметься на основі навчальних даних, які надаються моделі для покращеня свої продуктивності.

- *Функція втрат* &mdash; спосіб виміру втрат моделі. Оптимізатор намагається мінімізувати втрати моделі.

- *Метрики для моніторингу під час навчання та тестування* &mdash; у цій задачі ми слідкуватимо за точністю (відсоток зображень, які були правильно класифіковані).

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

---

class: middle

# Компіляція моделі

.center.width-100[![](figures/lec2/dense-model-compile.png)]

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

---


class: middle

# Підготовка даних

.center.width-90[![](figures/lec2/image-preparation.png)]

Раніше наші навчальні зображення зберігалися в масиві форми *(60000, 28, 28)* типу **uint8** зі значеннями в інтервалі *[0, 255]*. Ми
перетворимо його на масив **float32** форми *(60000, 28 ∗ 28)* зі значеннями від *0* до *1*.

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
Before training, we’ll preprocess the data by reshaping it into the shape the model expects and scaling it so that all values are in the $[0, 1]$ interval.

We’re now ready to train the model, which in Keras is done via a call to the model’s *fit()* method &mdash; we fit the model to its training data. 

---

class: middle

# Навчання моделі

.center.width-100[![](figures/lec2/fiting.png)]

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
Two quantities are displayed during training: the loss of the model over the training data, and the accuracy of the model over the training data. We quickly reach an accuracy of 0.989 (98.9%) on the training data. Now that we have a trained model, we can use it to predict class probabilities for new digits &mdash; images that weren’t part of the training data, like those from the test set.

---

class: middle

# Виконання прогнозу

.center.width-100[![](figures/lec2/testing-model.png)]

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

Кожне індекс $i$ в цьому масиві відповідає ймовірності того, що наше тестове зображення *test_digits[0]* належить до класу $i$.

---

class: middle
# Виконання прогнозу

Перше тестове зображення має найбільшу ймовірність (*0.99999106*, майже 1) для індекса масива *7*, тому відповідно до цього прогнозу моделі це має
бути *7*:

.center.width-70[![](figures/lec2/test-example.png)]

Перевіримо на відповідність тестовій мітці:

.center.width-60[![](figures/lec2/test-example2.png)]

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
On average, how good is our model at classifying such never-before-seen digits? Let’s check by computing average accuracy over the entire test set.

---

class: middle

# Оцінка моделі на нових даних

.center.width-100[![](figures/lec2/evaluate-model.png)]

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
The test-set accuracy turns out to be 97.8% &mdash; that’s quite a bit lower than the training-set accuracy (98.9%). This gap between training accuracy and test accuracy is an example of overfitting: the fact that machine learning models tend to perform worse on new data than on their training data. 

This concludes our first example—you just saw how you can build and train a neural network to classify handwritten digits in less than 15 lines of Python code.

---



class: blue-slide, middle, center
count: false

.larger-xx[Згорткові мережі]

???
We’re about to dive into the theory of what convnets are and why they have been so successful at computer vision tasks. But first, let’s take a practical look at a simple convnet example that classifies MNIST digits, a task we performed above using a densely connected network (our test accuracy then was 97.8%). Even though the convnet will be basic, its accuracy will blow our densely connected model out of the water.

---


class: middle

# Створюємо згорткову мережу

.center.width-100[![](figures/lec2/small-convnet.png)]

.smaller-xx[Важливо, що convnet приймає на вхід тензори форми *(image_height, image_width, image_channels)*, не включаючи розмірність пакету. У цьому випадку ми налаштуємо convnet для обробки вхідних даних розміром $(28, 28, 1)$, що відповідає формату зображень MNIST.]

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
The following listing shows what a basic convnet looks like. It’s a stack of *Conv2D* and *MaxPooling2D* layers. You’ll see in a minute exactly what they do. We’ll build the model using the Functional API.

Let’s display the architecture of our convnet.

---

class: middle

# Summary

.center.width-90[![](figures/lec2/model-summary.png)]


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
You can see that the output of every *Conv2D* and *MaxPooling2D* layer is a rank-3 tensor of shape $(height, width, channels)$. The width and height dimensions tend to shrink as you go deeper in the model. The number of channels is controlled by the first argument passed to the Conv2D layers $(32, 64, or 128)$.

After the last Conv2D layer, we end up with an output of shape (3, 3, 128) &mdash; a 3 × 3 feature map of 128 channels. The next step is to feed this output into a densely connected classifier like those you’re already familiar with: a stack of *Dense* layers. These classifiers process vectors, which are 1D, whereas the current output is a rank-3 tensor. To bridge the gap, we flatten the 3D outputs to 1D with a *Flatten* layer before adding the *Dense* layers.

Finally, we do 10-way classification, so our last layer has 10 outputs and a softmax activation. Now, let’s train the convnet on the MNIST digits. We’ll reuse a lot of the code from the MNIST example in *Fully connected NNs* section. Because we’re doing 10-way classification with a softmax output, we’ll use the categorical crossentropy loss, and because our labels are integers, we’ll use the sparse version, *sparse_categorical_crossentropy*.

---

class: middle

# Підготовка даних, компіляція та навчання моделі

.center.width-90[![](figures/lec2/training-convnet.png)]


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
Let’s evaluate the model on the test data.

---


class: middle

# Оцінка згорткової моделі на нових даних

.center.width-90[![](figures/lec2/eval-convnet.png)]


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
Whereas the densely connected model from *Fully connected NNs* section had a test accuracy of 97.8%, the basic convnet has a test accuracy of 99.1%: we decreased the error rate by about 60% (relative). Not bad!

But why does this simple convnet work so well, compared to a densely connected model? To answer this, let’s dive into what the *Conv2D* and *MaxPooling2D* layers do.

---

class: blue-slide, middle, center
count: false

.larger-xx[Будівельні блоки]

---


class: black-slide, middle

# Тензор (tensor)

масив чисел, розташованих у сітці зі змінною кількістю осей

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/t0.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/t02.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/t1.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/t12.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/t2.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/t22.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/t3.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/t32.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/anim-v2.gif)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/imc.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/imRGB.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/t32.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/imsec.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: black-slide, middle

.center.width-100[![](figures/lec2/imsec2.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---



class: black-slide, middle

# Згортка (convolution)

$$\boxed{(f*g)(x) = \int f(z)g(x - z) \, dz}$$


---

class: middle

.center.width-100[![](figures/lec2/Comparison_convolution_correlation.png)]

.footnote[Джерело: [Вікіпедія](https://en.wikipedia.org/wiki/Convolution)]

---


class: black-slide, middle

.center.width-100[![](figures/lec2/conv-cor.png)]

.footnote[Джерело: Kosta Derpanis. [Convolutional Networks](pdf/ConvNets.pdf)]

---

class: middle

# The convolution operation

.center.width-50[![](figures/lec2/im.png)]


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
The fundamental difference between a densely connected layer and a convolution layer is this: Dense layers learn global patterns in their input feature space (for example, for a MNIST digit, patterns involving all pixels), whereas convolution layers learn local patterns &mdash; in the case of images, patterns found in small 2D windows of the inputs (see figure on this slide). In the previous example, these windows were all 3 × 3.

Images can be broken into local patterns such as edges, textures, and so on.

---

class: middle

This key characteristic (convolution layers learn local patterns) gives convnets two interesting properties:

- *The patterns they learn are translation-invariant.* After learning a certain pattern in the lower-right corner of a picture, a convnet can recognize it anywhere: for example, in the upper-left corner. A densely connected model would have to learn the pattern anew if it appeared at a new location. This makes convnets data-efficient when processing images (because the visual world is fundamentally translation-invariant): they need fewer training samples to learn representations that have generalization power.

- *They can learn spatial hierarchies of patterns.* A first convolution layer will learn small local patterns such as edges, a second convolution layer will learn larger patterns made of the features of the first layers, and so on (see figure on next slide). This allows convnets to efficiently learn increasingly complex and abstract visual concepts, because t*he visual world is fundamentally spatially hierarchical*.


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
- *The patterns they learn are translation-invariant.* After learning a certain pattern in the lower-right corner of a picture, a convnet can recognize it anywhere: for example, in the upper-left corner. A densely connected model would have to learn the pattern anew if it appeared at a new location. This makes convnets data-efficient when processing images (because the visual world is fundamentally translation-invariant): they need fewer training samples to learn representations that have generalization power.

- *They can learn spatial hierarchies of patterns.* A first convolution layer will learn small local patterns such as edges, a second convolution layer will learn larger patterns made of the features of the first layers, and so on (see figure on next slide). This allows convnets to efficiently learn increasingly complex and abstract visual concepts, because t*he visual world is fundamentally spatially hierarchical*.

---

class: middle

# The convolution operation

.center.width-80[![](figures/lec2/cat.png)]


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
The visual world forms a spatial hierarchy of visual modules: elementary lines or textures combine into simple objects such as eyes or ears, which combine into high-level concepts such as “cat.”


Convolutions operate over rank-3 tensors called *feature maps*, with two spatial axes (*height* and *width*) as well as a depth axis (also called the *channels* axis). For an RGB image, the dimension of the depth axis is 3, because the image has three color channels: red, green, and blue. For a black-and-white picture, like the MNIST digits, the depth is 1 (levels of gray). The convolution operation extracts patches from its input feature map and applies the same transformation to all of these patches, producing an output feature map. This output feature map is still a rank-3 tensor: it has a width and a height. Its depth can be arbitrary, because the output depth is a parameter of the layer, and the different channels in that depth axis no longer stand for specific colors
as in RGB input; rather, they stand for *filters*. Filters encode specific aspects of the input data: at a high level, a single filter could encode the concept “presence of a face in the input,” for instance.

---


class: middle

## The concept of a response map: a 2D map of the presence of a pattern at different locations in an input

.center.width-80[![](figures/lec2/responce-map.png)]

.smaller-xx[In the MNIST example, the first convolution layer takes a feature map of size $(28, 28, 1)$ and outputs a feature map of size $(26, 26, 32)$: it computes 32 filters over its input. Each of these 32 output channels contains a 26 × 26 grid of values, which is a response map of the filter over the input, indicating the response of that filter pattern at different locations in the input (see figure above).]


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
In the MNIST example, the first convolution layer takes a feature map of size $(28, 28, 1)$ and outputs a feature map of size $(26, 26, 32)$: it computes 32 filters over its input. Each of these 32 output channels contains a 26 × 26 grid of values, which is a response map of the filter over the input, indicating the response of that filter pattern at different locations in the input (see figure above).

That is what the term feature map means: every dimension in the depth axis is a feature (or filter), and the rank-2 tensor $output[:, :, n]$ is the 2D spatial map of the response of this filter over the input.

---

class: middle, center

# Demo

.larger-x[[How convolution works in a convolutional layer?](https://ml4a.github.io/demos/convolution/)]

---


class: middle

Convolutions are defined by two key parameters:

- *Size of the patches extracted from the inputs* &mdash; These are typically 3×3 or   5×5. In the example, they were 3 × 3, which is a common choice.
- *Depth of the output feature map* &mdash; This is the number of filters computed by the convolution. The example started with a depth of 32 and ended with a depth of 64.

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

---


class: middle

In Keras Conv2D layers, these parameters (*Size of the patches extracted from the inputs*, *Depth of the output feature map*) are the first arguments passed to the layer:

- **Conv2D(output_depth, (window_height, window_width))**

.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
A convolution works by sliding these windows of size 3×3 or 5×5 over the 3D input feature map, stopping at every possible location, and extracting the 3D patch of surrounding features **(shape (window_height, window_width, input_depth)).** Each such 3D patch is then transformed into a 1D vector of shape **(output_depth,)**, which is done via a tensor product with a learned weight matrix, called the convolution kernel &mdash; the same kernel is reused across every patch. All of these vectors (one per patch) are then spatially reassembled into a 3D output map of shape **(height, width, output_depth)** . Every spatial location in the output feature map corresponds to the same location in the input feature map (for example, the lower-right corner of the output contains information about the lower-right corner of the input).

---


class: middle

# How convolution works

.smaller-xx[For instance, with 3×3 windows, the vector $output[i, j, :]$ comes from the 3D patch $input[i-1:i+1,
j-1:j+1, :]$. The full process is detailed in figure below.]

.center.width-55[![](figures/lec2/how_convolution_works.png)]


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
Note that the output width and height may differ from the input width and height for two reasons:

- *Border effects*, which can be countered by padding the input feature map
- The use of *strides*

Let’s take a deeper look at these notions.

---


class: middle

# Understanging border effects 

## Valid locations of 3×3 patches in a 5×5 input feature map


.center.width-90[![](figures/lec2/3x3_patches_in_5x5_input.png)]


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
Consider a 5×5 feature map (25 tiles total). There are only 9 tiles around which you can center a 3×3 window, forming a 3×3 grid (see figure on this slide). Hence, the output feature map will be 3×3. It shrinks a little: by exactly two tiles alongside each dimension, in this case. You can see this border effect in action in the earlier example: you start with 28×28 inputs, which become 26×26 after the first convolution layer.

---

class: middle

# Understanging padding

## Padding a 5×5 input in order to be able to extract 25 3×3 patches


.center.width-90[![](figures/lec2/padding_of_5x5_input.png)]


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
If you want to get an output feature map with the same spatial dimensions as the input, you can use *padding*. Padding consists of adding an appropriate number of rows and columns on each side of the input feature map so as to make it possible to fit center convolution windows around every input tile. For a 3×3 window, you add one column on the right, one column on the left, one row at the top, and one row at the bottom. For a 5×5 window, you add two rows.

In Conv2D layers, padding is configurable via the padding argument, which takes two values: *"valid"* , which means no padding (only valid window locations will be used), and *"same"*, which means “pad in such a way as to have an output with the same width and height as the input.” The padding argument defaults to *"valid"*.

---


class: middle

# Understanging convolution strides 

## 3×3 convolution patches with 2×2 strides


.center.width-90[![](figures/lec2/strides.png)]


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
The other factor that can influence output size is the notion of *strides*. Our description of convolution so far has assumed that the center tiles of the convolution windows are all contiguous. But the distance between two successive windows is a parameter of the convolution, called its *stride*, which defaults to 1. It’s possible to have strided convolutions: convolutions with a stride higher than 1. In figure, you can see the patches extracted by a 3×3 convolution with stride 2 over a 5×5 input (without padding).

Using stride 2 means the width and height of the feature map are downsampled by a factor of 2 (in addition to any changes induced by border effects). Strided convolutions are rarely used in classification models, but they come in handy for some types of models.

In classification models, instead of strides, we tend to use the max-pooling operation to downsample feature maps, which you saw in action in our first convnet example. Let’s look at it in more depth.

---


class: middle

# Understanging max-pooling operation

In the convnet example, you may have noticed that the size of the feature maps is halved after every MaxPooling2D layer. For instance, before the first *MaxPooling2D* layers, the feature map is 26×26, but the max-pooling operation halves it to 13×13. That’s the role of max pooling: to aggressively downsample feature maps, much like strided convolutions.

Max pooling consists of extracting windows from the input feature maps and outputting the max value of each channel. It’s conceptually similar to convolution,
except that instead of transforming local patches via a learned linear transformation (the convolution kernel), they’re transformed via a hardcoded *max* tensor
operation.




.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
In the convnet example, you may have noticed that the size of the feature maps is halved after every MaxPooling2D layer. For instance, before the first *MaxPooling2D* layers, the feature map is 26×26, but the max-pooling operation halves it to 13×13. That’s the role of max pooling: to aggressively downsample feature maps, much like strided convolutions.

Max pooling consists of extracting windows from the input feature maps and outputting the max value of each channel. It’s conceptually similar to convolution,
except that instead of transforming local patches via a learned linear transformation (the convolution kernel), they’re transformed via a hardcoded max tensor
operation.

A big difference from convolution is that *max pooling* is usually done with 2×2 windows and stride 2, in order to downsample the feature maps by a factor of 2. On the other hand, convolution is typically done with 3×3 windows and no stride (stride 1).

---



class: middle

## Understanging max-pooling operation 




.center.width-80[![](figures/lec2/conv-without-maxpolling.png)]


.footnote[Джерело: François Chollet. Deep Learning with Python, 2021.]

???
Why downsample feature maps this way? Why not remove the max-pooling layers and keep fairly large feature maps all the way up? Let’s look at this option. Our model would then look like the following listing.

What’s wrong with this setup? Two things:

- It isn’t conducive to learning a spatial hierarchy of features. The 3×3 windows in the third layer will only contain information coming from 7×7 windows in
the initial input. The high-level patterns learned by the convnet will still be very small with regard to the initial input, which may not be enough to learn to classify digits (try recognizing a digit by only looking at it through windows that are 7×7 pixels!). We need the features from the last convolution layer to contain information about the totality of the input.

- The final feature map has 22×22×128 = 61,952 total coefficients per sample. This is huge. When you flatten it to stick a Dense layer of size 10 on top, that
layer would have over half a million parameters. This is far too large for such a small model and would result in intense overfitting.

In short, the reason to use downsampling is to reduce the number of feature-map coefficients to process, as well as to induce spatial-filter hierarchies by making successive convolution layers look at increasingly large windows (in terms of the fraction of the original input they cover).


Note that max pooling isn’t the only way you can achieve such downsampling. As you already know, you can also use strides in the prior convolution layer. And you can use average pooling instead of max pooling, where each local input patch is transformed by taking the average value of each channel over the patch, rather than the max. But max pooling tends to work better than these alternative solutions. The reason is that features tend to encode the spatial presence of some pattern or concept over the different tiles of the feature map (hence the term feature map), and it’s more informative to look at the maximal presence of different features than at their average presence.

At this point, you should understand the basics of convnets &mdash; feature maps, convolution, and max pooling &mdash;  and you should know how to build a small convnet to solve a toy problem such as MNIST digits classification. 

---


class: end-slide, center
count: false

.larger-xx[Кінець]

---

count: false

# Література

- LeCun, Y., Bengio, Y., & Hinton, G. (2015). Deep learning. nature, 521(7553), 436-444.
