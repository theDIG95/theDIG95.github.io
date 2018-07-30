---
title: Red and Gray Images - OpenCV / Python
layout: post
icon : fa-code 
---

A lot of the art in this digital age is created using complex machines and softwares. From simple tasks as changing the contrast of images to separating an object from an image is done via image processing and computer vision algorithms. The [OpenCV library](http://opencv.org/) provides an excellent implementation of almost all the modern algorithms. Also, it contains bindings for python scripts.  
[Find the Gist for this example here](https://gist.github.com/theDIG95/64b2a2c214a517bd42051205ba22fb52)  
__NOTE__ This is not exactly ready to win any artistic praise but it demonstrates the basic procedure to create more complex images.

## Creating Red and Gray Images

Many times we see images with an bright red colors with the background in grayscale. In this post we will also create a rudimentary form of such an image.
Firstly, we import the OpenCV library and open the target image.

```python
import numpy as np
import cv2

img = cv2.imread("image.jpg")
```

The image is stored as a numpy array of the shape (Width, Height, Color Channels). Most commonly images have three color channels i.e. Red, Blue, Green.

__Note__ OpenCV stores colors in Blue, Green, Red order, i.e. Blue is the 0 index channel and Red is the 2 index channel.

## Grayscaling an image

Although OpenCV provides a built-in flag to either read the image as grayscale

```python
gray_img = cv2.imread("image.jpg", cv2.IMREAD_GRAYSCALE)
```

Or to convert a loaded image to grayscale via a method.

```python
gray_image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
```

However here we will manually convert the image to grayscale. This can be done from a simple arithmetic operation.

$$ gray = \frac{red + green + blue}{3} $$

We implement it as

```python
gray = img.sum(axis=2) / 3
```

## Selection a region to retain color  

OpenCV allows for an interactive window to select a desired region from an image.

```python
region = cv2.selectROI("Select Region to Enhance", img)
```

This opens an interactive window where a four sided region can be selected. The returned `region` is a of type `tuple` which gives `(x-top-corner, y-top-corner, width, height)`.  
Using this region we can separate the region's red color channel.

```python
red_region = img[region[1]:region[1]+region[3], region[0]:region[0]+region[2], 2]
```

The last index i.e. 2 is for the red channel.

## Applying gamma correction  

Now to make the red color region stand out further we apply gamma correction.  
Gamma correction is done as

$$ corrected = image^{gamma} $$

Where \\( gamma < 1 \\) causes a darker image by extending the darker colors and \\(gamma > 1 \\) causes brighter regions by expanding the bright colors.

## Creating the final image  

We start be creating an empty array and filling in all the channels as gray first. Then we fill in the region at the red channel by the gamma corrected version of the red region.

```python
# Create placeholder image
red_enhanced_img = np.zeros(self._img.shape)
# Fill placeholder image
red_enhanced_img[:, :, 0] = grey ** gamma_grey
red_enhanced_img[:, :, 1] = grey ** gamma_grey
red_enhanced_img[:, :, 2] = grey ** gamma_grey
# Fill in the region
red_enhanced_img[region[1]:region[1]+region[3], region[0]:region[0]+region[2], 2] = reg_grey_avg ** gamma_red
```

Then we can change the type to `numpy.int32` as after the gamma correction the image becomes of the type `float` which is not rendered as an image.

```python
red_enhanced_img = red_enhanced_img.astype(numpy.int32)
```

Then we can either save the image to file or display it.

```python
# Save to file
cv2.imwrite("red_grey.jpeg", enhanced_img)
# Display
cv2.imshow("Red Gray Image", enhanced_img)
cv2.waitKey(0)
cv2.DestroyAllWindows()
```

## Results  

### Original Image  

<a href="#" class="image centered"><img src="{{ 'assets/images/red-enhanced/rose.jpeg' | relative_url }}" alt="Rose" /></a>

### Enhanced image  

<a href="#" class="image centered"><img src="{{ 'assets/images/red-enhanced/red_grey.jpeg' | relative_url }}" alt="Red Gray Rose" /></a>
