---
title: Red and Gray Images Improvement - Pillow / Python
layout: post
icon : fa-code 
---

This is a reiteration of a [previous post]({{ site.baseurl }}{% post_url 2018-07-28-red-enhance %}) in which we saw how to enhance a color channel of an image using OpenCV and python.  
In this post we will use a threshold value for a channel(red in this case) and use that ro retain color or to convert to grey scale.  
Firstly in this experiment we will use the [Pillow](https://python-pillow.org/) library.  
Now we would do our imports.  

```python
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt
```

Now we need to load our image into pillow and then convert it into a numpy array for further processing

```python
# Image path
img_path = 'image.jpeg'
# Load the image via Pillow
img_pil = Image.open('imgage.jpeg')
# Convert image to numpy array
img_arr = np.array(img_pil)
```

Now we can choose and set a threshold value for the red channel such that if the red value is less than the threshold in a pixel it will be converted into gray. After that we use that value to find indices of the pixels that are lower than the threshold. Finally we calculate the gray values using \\( gray = \frac{red + green + blue}{3} \\) and convert the required pixels to grayscale.
We do however, need a 3 channel value as the final image is still in RGB space.

```python
# Set a red value threshold
    red_thresh = 70
    # Check red values greather than threshold
    indices = np.where(img_arr[:, :, 0] < red_thresh)
    # Convert to gray where values are less than threshold
    grey_val = (
        img_arr[indices[0], indices[1], 0] +
        img_arr[indices[0], indices[1], 1] +
        img_arr[indices[0], indices[1], 2]
    ) / 3
    img_arr[indices[0], indices[1]] = np.swapaxes(np.array((grey_val, grey_val, grey_val)), 0, 1)
```

Finally we can convert it back into a Pillow image and save or view it.

```python
# Save the image
pil_img = Image.fromarray(img_arr.astype(np.uint8))
pil_img.save('red_grey.jpeg')
```

## Results  

### Original Image  

<a href="#" class="image centered"><img src="{{ 'assets/images/red-enhanced/rose.jpeg' | relative_url }}" alt="Rose" /></a>

### Enhanced image  

<a href="#" class="image centered"><img src="{{ 'assets/images/red-enhanced/red_grey_pil.jpeg' | relative_url }}" alt="Red Gray Rose" /></a>

__NOTE__ We can further remove artifacts in the image by tweaking the threshold value.
