# Eye In The Sky

[Satellite Image Classification](http://inter-iit.tech/events/the-eye-in-the-sky.html), InterIIT Techmeet 2018, IIT Bombay.

## About

This repository contains the implementation of two algorithms namely [U-Net: Convolutional Networks for Biomedical
Image Segmentation](https://arxiv.org/pdf/1505.04597.pdf) and [Pyramid Scene Parsing Network](https://arxiv.org/pdf/1612.01105.pdf) modified for the problem of satellite image classification.

## Files

- [`main_unet.py`](main_unet.py) : Python code for training the algorithm with U-Net architecture including the encoding of the ground truths.
- [`unet.py`](unet.py) : Contains our implementation of U-Net layers.
- [`test_unet.py`](test_unet.py) : Code for Testing, calculating accuracies, calculating confusion matrices for training and validation and saving predictions by the U-Net model on training, validation and testing images.
- [`Inter-IIT-CSRE`](Inter-IIT-CSRE) : Contains all the training, validation ad testing data.
- [`Comparison_Test.pdf`](Comparison_Test.pdf) : Side by side comparison of the test data with the U-Net model predictions on the data.
- [`train_predictions`](train_predictions) : U-Net Model predictions on training and validation images.
- [`plots`](plots) : Accuracy and loss plots for training and validation for U-Net architecture.
- [`Test_images`](Test_images), [`Test_outputs`](Test_outputs) : Contains test images and their predictions b the U-Net model.
- [`class_masks`](class_masks), [`compare_pred_to_gt`](compare_pred_to_gt), [`images_for_doc`](images_for_doc) : Contains several images for documentation.
- [`PSPNet`](PSPNet) : Contains training files for implementation of PSPNet algorithm to satellite image classification.

## Usage

Clone the repository, change your present working directory to the cloned directory.
Create folders with names `train_predictions` and `test_outputs` to save model predicted outputs on training and testing images (Not required now as the repo already contains these folders)

```
$ git clone https://github.com/manideep2510/eye-in-the-sky.git
$ cd eye-in-the-sky
$ mkdir train_predictions
$ mkdir test_outputs
```

For training the U-Net model and saving weights, run the below command

```
$ python3 main_unet.py
```

To test the U-Net model, calculating accuracies, calculating confusion matrices for training and validation and saving predictions by the model on training, validation and testing images.

```
$ python3 test_unet.py
```

## Now, let's discuss!
Let's now discuss 

**1. What this project is about,** 

**2. Architectures we have used and experimented with and** 

**3. Some novel training strategies we have used in the project**

### Introduction

[Remote sensing](https://www.usgs.gov/faqs/what-remote-sensing-and-what-it-used) is the science of obtaining information about objects or areas from a distance, typically from aircraft or satellites.

We realized the problem of satellite image classification as a [semantic segmentation](https://people.eecs.berkeley.edu/~jonlong/long_shelhamer_fcn.pdf) problem and built semantic segmentation algorithms in deep learning to tackle this problem.

### Algorithms Implemented

1. UNet - GT with RGB channels
2. PSPNet - GT with RGB channels
3. UNet with One Hot Encoded GT
4. PSPNet with One Hot Encoded GT

**[U-Net: Convolutional Networks for Biomedical Image Segmentation](https://arxiv.org/pdf/1505.04597.pdf)**

<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/images_for_doc/unet.png" width="640"\>
</p>

**[Pyramid Scene Parsing Network - PSPNet](https://arxiv.org/pdf/1612.01105.pdf)**

<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/images_for_doc/pspnet.png" height="300" width="700"\>
</p>

## Mapping RGB Values in the ground truth to a one-hot encode vector to generate n channel encoded ground truth for training

The ground truths provided are 3 channel RGB images. In the current dataset, there are only 9 unique RGB values in the ground truths as there are 9 classes that are to be classified. These 9 different RGB values are one-hot encoded to generate a 9 channel encoded ground truth with each channel representing a particular class.

Below is the encoding scheme

<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/images_for_doc/table_onehot.png" width="640"\>
</p>

Realisation of each channel in the encoded ground truth as a class

<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/images_for_doc/channel_classes.png" width="900"\>
</p>

So instead of training on the RGB values of the ground truth we have converted them into the one-hot values of different classes. This approach yielded us a validation accuracy of 85% and training accuracy of 92% compared to 71% validation accuracy and 65% training accuracy when we were using the RGB ground truth values for training.

This might be due to decrease in variance and mean of the ground truth of training data as it acts as an effective normalization technique. **The better performance of this training technique is also because the model is giving an output with 9 feature maps each map indicating a class, i.e, this training technique is acting as if the model is trained on each of the 9 classes separately for some extent(but here definitely the prediction on one channel which corresponds to a particular class depends on others)**.

### Problems with PSPNet

Our results on PSPNet for Satellite image classification:

Training Accuracy - 49%
Validation Accuracy - 60%

*Reasons:*

- Huge number of parameters (46M trainable params)
- Contains a Resnet which demands more data to learn the features properly.
- Under-fitting - We currently have very less data even after the strided cropping (15k images), so the PSPNet could not learn to segment (classify) the satellite images

### Final Architecture chosen

*U-Net:*

- Has less number of parameters (31M)
- Lesser number of layers than in PSPNet.
- Best suitable for cases where the data is comparatively less, especially in the current case, training on 14 images

*Modified U-Net:*

- Batch Normalization before every downsampling and upsampling layers to decrease the variance and mean of the feature maps.
- Used deconvolution layers instead of conv layers in the upsampling part of the UNet, but the results weren't good


### Data Processing during training

**The Strided Cropping:**

To have sufficient training data from the given high definition images cropping is required to train the classifier which has about 31M parameters of our U-Net implementation. The crop size of 64x64 we find under-representation of the individual classes and the geometry and continuity of the objects is lost, decreasing the field of view of the convolutions.

Using a cropping window of 128x128 pixels with a stride of 32 resultant of 15887 training 414 validation images.

**Image Dimensions:**

Before cropping, the dimensions of training images are converted into multiples of stride for convenience during strided cropping.

For the cases where the no. of crops is not the multiple of the image dimensions we initially tried zero padding , we realised that adding padding will add unwanted artefacts in the form of black pixels in training and test images leading to training on false data and image boundary.

Alternatively we have correctly changed the image dimensions by adding extra pixels in the right most side and bottom of the image. So we padded the difference from the left most part of the image to it’s right deficit end and similarly for the top and bottom of the image.

## Results

### Model Predictions Vs Ground truths on some training and Validation images

**Training Example 1: Image '2.tif' from training data**

<p float="left">
  <img src="images_for_doc/2.png" width="275" />
  <img src="images_for_doc/gts/2.png" width="275" /> 
  <img src="images_for_doc/pred2.jpg" width="275" />
</p>
<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/images_for_doc/label.png" width="750"\>
</p>

**Training Example 2: Image '4.tif' from training data**

<p float="left">
  <img src="images_for_doc/4.png" width="275" />
  <img src="images_for_doc/gts/4.png" width="275" /> 
  <img src="images_for_doc/pred4.jpg" width="275" />
</p>
<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/images_for_doc/label.png" width="750"\>
</p>

**Validation Example: Image '14.tif' from dataset**

<p float="left">
  <img src="images_for_doc/14.png" width="275" />
  <img src="images_for_doc/gts/14.png" width="275" /> 
  <img src="images_for_doc/pred14.jpg" width="275" />
</p>
<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/images_for_doc/label.png" width="750"\>
</p>

### An interesting observation

Our model is able to predict some classes which a human annotator wasn't able to. The un-identifiable classes in the images are labeled as white pixels by the human annotator. Our model is able to predict some of these white pixels correctly as some class, but this caused a decrease in the overall accuracy as the white pixels are considered as a seperate class by the model.

Here the model is able to predict the white pixels as a building which is correct and can be clearly seen in the input image

<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/images_for_doc/unclass_pred.png" width="750"\>
</p>

**Chech out [`Comparison_Test.pdf`](Comparison_Test.pdf) for comparision between test images and their predicted outputs by the model**

<!---### Model Predictions on Testing images (Ground truths not available)--->

<!--- 
<p float="left">
  <img src="Test_images/1.png" width="200" />
  <img src="Test_outputs/1.jpg" width="200" /> 
  <img src="Test_images/2.png" width="200" />
  <img src="Test_outputs/2.jpg" width="200" />
</p>
--->

<!---
<p float="left">
  <img src="Test_images/3.png" width="200" />
  <img src="Test_outputs/3.jpg" width="200" />
  <img src="Test_images/4.png" height="220" width="200" />
  <img src="Test_outputs/4.jpg" height="220" width="200" /> 
</p>
--->

<!---
<p float="left">
  <img src="Test_images/5.png" width="200" />
  <img src="Test_outputs/5.jpg" width="200" />
  <img src="Test_images/6.png" height="250" width="200" />
  <img src="Test_outputs/6.jpg" height="250" width="200" />
</p> --->

<!---Solarized dark             |  Solarized Ocean
:-------------------------:|:-------------------------:
![agaksbmsnbc](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_images/1.png)  |  ![ajsvsdsdddhv](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_outputs/1.jpg)
![agaksbmsnbc](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_images/2.png)  |  ![ajsvsdsdddhv](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_outputs/2.jpg)
![agaksbmsnbc](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_images/3.png)  |  ![ajsvsdsdddhv](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_outputs/3.jpg)
![agaksbmsnbc](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_images/4.png)  |  ![ajsvsdsdddhv](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_outputs/4.jpg)
![agaksbmsnbc](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_images/5.png)  |  ![ajsvsdsdddhv](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_outputs/5.jpg)
![agaksbmsnbc](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_images/6.png)  |  ![ajsvsdsdddhv](https://github.com/manideep2510/eye-in-the-sky/blob/master/Test_outputs/6.jpg)--->

### Accuracy and Loss plots for training and validation

<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/plots.png" width="800"\>
</p>

### Confusion matrices for training

<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/images_for_doc/conf.png" width="800"\>
</p>

### Kappa Coefficient

Kappa Coefficients With and Without considering the unclassified pixels

<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/images_for_doc/kappa.png" width="600"\>
</p>

### Overall Accuracy

Overall Accuracy With and Without considering the unclassified pixels

<p align="center">
    <img src="https://github.com/manideep2510/eye-in-the-sky/blob/master/images_for_doc/overall.png" width="600"\>
</p>

## References

[1] [U-Net: Convolutional Networks for Biomedical Image Segmentation](https://arxiv.org/pdf/1505.04597.pdf), Olaf Ronneberger, Philipp Fischer, and Thomas Brox

[2] [Pyramid Scene Parsing Network](https://arxiv.org/pdf/1612.01105.pdf), Hengshuang Zhao, Jianping Shi, Xiaojuan Qi, Xiaogang Wang, Jiaya Jia

[3] [A 2017 Guide to Semantic Segmentation with Deep Learning](http://blog.qure.ai/notes/semantic-segmentation-deep-learning-review), Sasank Chilamkurthy
