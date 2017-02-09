# A Rough Guide to OpenCV 3.* in Python 3.*

OpenCV is an incredibly powerful tool for implementing computer vision applications in Python. However, it can
be fairly inaccessible in the begeinning. Here, we try to collect all the information we gathered in an attempt
to provide a guide on how to do certain things. This is by no means complete but, hopefully, a useful starting point.

## Getting Started with Images
Import OpenCV and numpy

```
import cv2
import numpy
```

Read an image:
```
img = cv2.imread('path', flag)
```
reads the image from the path provided. Here, ```flag = cv2.IMREAD_COLOR, cv2.IMREAD_GRAYSCALE, cv2.IMREAD_UNCHANGED``` determines whether the image is read as color, grayscale or as such, i.e. including the [alpha channel](http://www.howtogeek.com/howto/42393/rgb-cmyk-alpha-what-are-image-channels-and-what-do-they-mean/). Alternatively, the flag can be specified as an integer, i.e. ```flag = 1,0,-1```.

An image can be displayed in a named window:
```
cv2.imshow('name of window', img)
```
Not suprisingly, the first argument is the name of the window and the second one the image to be displayed. In order to admire the image on screen you need to tell OpenCV to wait for a key to be pressed:
```
k = cv2.waitKey(delay=0) & 0xff
if k == 27:
    cv2.destroyAllWindows()
```
```delay``` is the lag in milliseconds, 0 means wait forever, ```& 0xff``` is needed on 64-Bit machines. if the esc-key is pressed (```k==27```) the opened window is destroyed. To be precise, ```cv2.destroyAllWindows()``` closes all open windows. A single named window can be closed with ```cv2.destroyWindow('name of window')```.

An image can be saved to disk with
```
cv2.imwrite('path',img)
```

Accessing the properties of an image:```img.shape``` returns a tuple of numbers of rows, columns and channels, ```img.size``` returns the total number of pixels, and ```img.dtype``` gives the image datatype.

An image can be resized with 
```
img2 = cv2.resize(img, None, fx=0.5, fy=0.5, interpolation = cv2.INTER_AREA)
```
Here, ```fx``` and ```fy``` denote the factors by which the image is rescaled in the x and y direction, i.e. in this example the image is shrunk by half. ```interpolation``` denotes the interpolation method used: ```cv2.INTER_AREA``` is recommended for shrinking, while ```cv2.INTER_CUBIC``` and ```cv2.INTER_LINEAR``` can be used for zooming, the latter being significantly faster.

Individual pixel values can be accessed and modified by row and column coordinates, i.e. ```img[y,x]``` is either a triplet, e.g. of blue, green and red values for a BGR image, or the intensity for a gray scale image. As a faster alternative, numpy array methods should be used:
```
img.item(y,x,2)  # access the red value of pixel x,y for a BGR image
img.itemset((y,x,2),100) # set the red value of pixel x,y to 100
```
Note that the row and column indices, ```y``` and ```x``` start in the upper left corner of the image, making it a somewhat unintuitive choice of coordinate system.

A region of interest in an image can be selected by standard index slicing methods:
```
img[y_min:y_max, x_min:x_max]
```

Different channels of an image can be accessed:
```
blue, green, red = cv2.split(img)
img = cv2.merge((blue, green, red))
```
Again, numpy array methods are significantly more efficient, e.g.,
```
green = img[:,:,1]
```

Add a border to an image
```
img2 = cv2.copyMakeBorder(img, top, bottom, left, right, borderType)
```
Here, ```img``` ist the original image, ```top```, ```bottom```, ```left```, and ```right``` are the widths of the border in the corresponding direction, and ```borderType``` denotes the kind of border:
* ```cv2.BORDER_CONSTANT``` constant colored border. Requires additional parameter giving the color, e.g. ```value=[255,0,0]``` for a blue border
* ```cv2.BORDER_REFLECT``` reflection of image border, e.g. left border will be columns 0 to left-1 of image in reversed order
* ```cv2.BORDER_REFLECT_101``` or ```cv2.BORDER_DEFAULT``` reflection of image border, e.g. left border will be columns 1 to left of image in reversed order
* ```cv2.BORDER_REPLICATE``` repeat outermost element, e.g left border is column 0 repeated left times
* ```cv2.BORDER_WRAP``` periodic boundary conditions, e.g. right border will be columns 0 to left-1 of image

We can convert between different [colorspaces](http://docs.opencv.org/3.2.0/de/d25/imgproc_color_conversions.html), e.g. from BGR to gray scale, with 
```
cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
```
The whole variety of available conversions can be found [here](http://docs.opencv.org/3.2.0/d7/d1b/group__imgproc__misc.html#ga4e0972be5de079fed4e3a10e24ef5ef0)

This can also be used to convert a single color, e.g. the HSV value of green in BGR is given by:
```
green = numpy.uint8([[[0,255,0 ]]])
hsv_green = cv2.cvtColor(green,cv2.COLOR_BGR2HSV)
```

Affine transformations, i.e. translation, scaling, rotation, shear mapping, are implemented by passing a 2x3 matrix to ```cv2.warpAffine```. The first two columns of the matrix encode scaling, rotation and shear and the last column the translation.
Example of a translation by 100 pixels in x and y:
```
rows, columns = img.shape[0:2]

# transformation matrix:
U = numpy.float32([[1,0,100],[0,1,100]])

img2 = cv2.warpAffine(img, U, (columns,rows))
```
The first argument of ```cv2.warpAffine``` is the image to be transformed, the second the transformation matrix and the third the size of the output image. A more complex scaled rotation ca be characterized by a rotation angle, the point around which to rotate and a scale. The function ```cv2.getRotationMatrix2D``` returns the transformation matrix needed for the affine transformation:
```
U = cv2.getRotationMatrix2D((x,y),theta,scale)
```
Here, ```(x,y)``` is the coordinate of the center of rotation, ```theta``` the angle, and ```scale``` the scale.

Alternatively, an affine transformation can be defined by explicitly defining the mapping for a set of three points:
```
orig_points = numpy.float32([[0,0],[100,100],[50,200]])
trans_points = numpy.float32([[100,100],[100,100],[100,250]])

U = cv2.getAffineTransform(orig_points, trans_points)

img7 = cv2.warpAffine(img2, U, (columns,rows))
```
Here, ```cv2.getAffineTransform``` returns the corresponding transformation matrix.

Similarly, a perspective transformation ca be defined by explicitly specifying the mapping for a set of four points. ```cv2.getPerspectiveTransform``` then yields the corresponding 3x3 transformation matrix:
```

```
 



Add, subtract, blend images images
Bitwise operations
PCA, SVD, ...


`````` `````` `````` `````` `````` `````` ``````