# Traffic Sign Classification using Deep Learning
***
## Project: Build a Traffic Sign Recognition Program
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
***
## Overview

In this project I experimented building deep architecture for the task of correctly classifying road traffic signs typically found on German roads. To achieve this I considered two feed forward convolutational neural network architectures implemented in python using google's Tensorflow framework via the python API. I use the the German Traffic Sign Recognition Benchmark (GTSRB) data set to train and test the deep learning models. Once satisfactory performance have been achieved on the GTSRB data set, we then apply the models to real traffic signs on German roads acquired though google maps. The classification performance of the models on these images are then evaluated.

This work was done as part of the Udacity Self Driving Car Nano-Degree program

***
## The German Traffic Sign Recognition Benchmark Data-set

Historical development of road infrastructure has been done with the human visual system being the key method of information transfer between the environment and vehicle by means of a human perception and then control. Any autonomous driving system capable of safely operating in existing road environments must then as a bare minimum be capable of human level visual recognition of the environment it operates in.  Visual classification is a key element in more complex visual recognition tasks (such as object detection and semantic segmentation) and also a key task in the visual perception of an autonomous vehicle for correct classification of road signs.


 The German Traffic Sign Recognition Benchmark (GTSRB) data set is a multi-class, single-image classification challenge made up of images of images of traffic signs taken from German roads, with an associated class label for each image in the dataset. The images are taken from video data from a camera on-board a moving car, the images of the signs are taken from the video data removing all temporal information.

The size of the data-set is summarised as follows


* Each image is 32 x 32 pixels made up of 3 colour channels formatted in RGB. Each pixel is saved using an 8 bit unsigned integer giving a total possible 256 possible values per pixel.

*  Each image belongs to one of 43 unique classes/labels grouped by the design/meaning of the sign.

*  The training set is made up of 34799 images with their associated labels.

*  The validation set is made up of 4410 images with their associated labels.

*  The test set is made up of 12630 images with their associated labels

The data set can be downloaded [here](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/5898cd6f_traffic-signs-data/traffic-signs-data.zip).


### Visualisation of Data-set

On initial thought, correctly classifying traffic signs could be considered to be a fairly simple task due to the design of traffic signs being unique and showing extremely little variably in apprentice within each class. In addition to this, the designs are typically simple and are placed in positions which should be clear to drivers to identify. However there are many aspects of the GTSRB data set that is challenging from the point of view of accurate classification, these include

* Variations in viewpoint
* Variation in brightness of lighting condition
* Motion blur (due to the video capture)
* Damages to signs

Many of these artefacts can be seen by viewing the images from the dataset below

 ![enter image description here]( https://github.com/joshwadd/Deep-traffic-sign-classification/blob/master/ClassExamples2.png?raw=true)
 
The random sample above is typical of the dataset as a whole. Variation in the brightness of each sample is typically the most notable visual degradation and should be given attention in the preprocessing stage.


### Exploration of the GTSRB Data-set

A histogram plot of the number of each samples per class shows a large imbalance in distribution across the classes. Some classes contain as many as almost 10 times the number of samples as others. This imbalance in distribution should be addressed to prevent any bias arising in training the model on this dataset. 



![enter image description here](https://github.com/joshwadd/Deep-traffic-sign-classification/blob/master/trainingdistribution.png?raw=true)


***
## Data Augmentation

Observing the GTSRB data sets two things became very apparent

* The dataset has a large imbalance in the number of sample occurrences across classes. That is to say some classes contain a lot more image data samples then others.

* The size of the data set is insufficient for training a high capacity deep network, and over-fitting to this small training set is likely to occur.

To address both of these issues, I used data augmentation techniques to create new image samples artificially, enlarging the data set using class preserving image transformations. The images were generated in such way that the resulting augmented training distribution was balanced across classes.

The data augmentation pipeline is made up of three components that will be used in series applying randomly generated parameters to generate the transformations. The transforms used are chosen so that once applied the underlying class content is maintained. For example, mirror flipping a left turn sign would make it a right turn and should not be used. The following three components make up the pipeline

### 1. Rotation

I first apply a rotation of a random angle between -15 and +15 degrees to the input image
```python
from skimage.transform import rotate

def rotate_image(image, max_angle =15):
    rotate_out = rotate(image, np.random.uniform(-max_angle, max_angle), mode='edge')
    return rotate_out
```


![enter image description here](https://github.com/joshwadd/Deep-traffic-sign-classification/blob/master/Rotationexample.png?raw=true)


### 2. Translation

Next I apply a random translation in the height and width up-to a maximum translation of 5 pixels.

```python
import cv2

def translate_image(image, max_trans = 5, height=32, width=32):
    translate_x = max_trans*np.random.uniform() - max_trans/2
    translate_y = max_trans*np.random.uniform() - max_trans/2
    translation_mat = np.float32([[1,0,translate_x],[0,1,translate_y]])
    trans = cv2.warpAffine(image, translation_mat, (height,width))
    return trans
```
![](https://github.com/joshwadd/Deep-traffic-sign-classification/blob/master/Translationexample.png?raw=true)

### 3. Projection Transform

Finally I apply a projection (homography) transform with randomly selected co-ordinates

```python
from skimage.transform import ProjectiveTransfor

def projection_transform(image, max_warp=0.8, height=32, width=32):
    #Warp Location
    d = height * 0.3 * np.random.uniform(0,max_warp)
    
    #Warp co-ordinates
    tl_top = np.random.uniform(-d, d)     # Top left corner, top margin
    tl_left = np.random.uniform(-d, d)    # Top left corner, left margin
    bl_bottom = np.random.uniform(-d, d)  # Bottom left corner, bottom margin
    bl_left = np.random.uniform(-d, d)    # Bottom left corner, left margin
    tr_top = np.random.uniform(-d, d)     # Top right corner, top margin
    tr_right = np.random.uniform(-d, d)   # Top right corner, right margin
    br_bottom = np.random.uniform(-d, d)  # Bottom right corner, bottom margin
    br_right = np.random.uniform(-d, d)   # Bottom right corner, right margin
        
    ##Apply Projection
    transform = ProjectiveTransform()
    transform.estimate(np.array((
                (tl_left, tl_top),
                (bl_left, height - bl_bottom),
                (height - br_right, height - br_bottom),
                (height - tr_right, tr_top)
            )), np.array((
                (0, 0),
                (0, height),
                (height, height),
                (height, 0)
            )))
    output_image = warp(image, transform, output_shape=(height, width), order = 1, mode = 'edge')
    return output_image
```
![](https://github.com/joshwadd/Deep-traffic-sign-classification/blob/master/RotationProjection.png?raw=true)

### Full Augmentation Pipeline

Using these three transforms gives the full class preserving data augmentation pipeline. Examples of typical images produced using this pipeline are shown below.

![](https://github.com/joshwadd/Deep-traffic-sign-classification/blob/master/dataAug1.png?raw=true)
![](https://github.com/joshwadd/Deep-traffic-sign-classification/blob/master/dataAug2.png?raw=true)


***

## Deep Learning Architectures

For this

### AlexNet Style

| Layer         		|     Description	        					| Input |Output| 
|:---------------------:|:---------------------------------------------:| :----:|:-----:|
| Convolution 5x5     	| 1x1 stride, valid padding, RELU activation 	|**32x32x1**|28x28x48|
| Max pooling			| 2x2 stride, 2x2 window						|28x28x48|14x14x48|
| Convolution 5x5 	    | 1x1 stride, valid padding, RELU activation 	|14x14x48|10x10x96|
| Max pooling			| 2x2 stride, 2x2 window	   					|10x10x96|5x5x96|
| Convolution 3x3 		| 1x1 stride, valid padding, RELU activation    |5x5x96|3x3x172|
| Max pooling			| 1x1 stride, 2x2 window        				|3x3x172|2x2x172|
| Flatten				| 3 dimensions -> 1 dimension					|2x2x172| 688|
| Fully Connected | connect every neuron from layer above			|688|84|
| Fully Connected | output = number of traffic signs in data set	|84|**43**|
