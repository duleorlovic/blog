---
layout: post
---

# Installing in python

https://docs.opencv.org/master/d6/d00/tutorial_py_root.html
```
sudo apt install python-opencv ipython
ipython
import cv2 as cv
print cv.__version__
4.1.0
```

https://docs.opencv.org/4.2.0/dc/d2e/tutorial_py_image_display.html
Gui
```
file_name = 'test/fixtures/files/computer_text.png'
img = cv.imread(file_name, cv.IMREAD_GRAYSCALE)
cv.rectangle(img,(384,0),(510,128),(0,255,0),3)
cv.imshow('window_name', img)
cv.waitKey(0)
cv.destroyAllWindows()
```

Operations https://docs.opencv.org/4.2.0/d3/df2/tutorial_py_basic_ops.html
```
print img.shape, img.size, img.dtype
rows,cols,channels = img2.shape

# split rgb channels
b,g,r = cv.split(img)
img = cv.merge((b,g,r))

# convert to grayscale
gray = cv.cvtColor(image, cv2.COLOR_BGR2GRAY)

# OpenCV add uses saturation (max 255) and Numpy uses moduo
x = np.uint8([250])
y = np.uint8([10])
print( cv.add(x,y) ) # 250+10 = 260 => 255
print( x+y )          # 250+10 = 260 % 256 = 4

# blend, merge two images
dst = cv.addWeighted(img1,0.7,img2,0.3,0)
```

Image processing https://docs.opencv.org/4.2.0/d2/d96/tutorial_py_table_of_contents_imgproc.html

# Python syntax

```
# debugger
import pdb; pdb.set_trace()
```

* `array[1:10, 50:60]` select area in Numpy 2D array
* `dict(name="Dule",sport="Kayak")` dicionary is the same as defining as hash
  `{ "name": "Dule", "sport": "Kayak"}`
* `a[st==1]` selecting only...

# Ruby wrapper

https://github.com/D-Alex/ropencv
```
sudo apt-get install ruby rubygems cmake g++ libopencv-dev
gem install ropencv
```

```
require 'ropencv'
include OpenCV

img = cv::imread(file_name)
```


# Interesting

* bar code scanner https://www.pyimagesearch.com/2014/11/24/detecting-barcodes-images-python-opencv
* https://github.com/jasonmayes/Real-Time-Person-Removal
