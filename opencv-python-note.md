# OpenCV-Python 指南

# 开始

### 目标
> 学习如何读取图像, 显示图像, 保存图像
> 学习函数cv2.imread(), cv2.imshow(), cv2.imwrite()
> 学习使用Matplotlib显示图像

### 使用OpenCV
#### 读取图像
使用函数cv2.imread()读取图像, 图像应该是在工作目录下或者全路径下

第二个参数是一个flag, 指定那种方式读取图像
cv2.IMREAD_COLOR: 读取彩色图像, 图像的透明部分将被忽略, 默认flag  (值为1)
cv2.IMREAD_GRAYSCALE: 使用灰度级模式读取图像 (值为0)
cv2.IMREAD_UNCHANGED: 读取全部图像, 包括alpha通道 (值为-1)

```python
import numpy as np
import cv2
# 用灰度模式加载以一张彩色图像
img = cv2.imread('mess5.jpg',0)
```
#### 显示图像
使用函数cv2.imshow()在一个窗口中显示图像.

第一个参数是窗口标题字符串
第二个参数为图像对象

```python
cv2.imshow('image', img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
cv2.waitKey() 参数为毫秒, 是指在这个时间内等待按键事件, 当有按键来时, 程序将继续执行; 若为0, 则无限等待按键事件
cv2.destroyAllWindows() 关闭所有窗口
cv2.destroyWindow() 关闭指定窗口

使用cv2.namedWindow() 可以改变窗口模式, 如可以调整大小, 默认为cv2.WINDOW_NORMAL.

```python
cv2.namedWindow('image', cv2.WINDOW_NORMAL)
cv2.imshow('image', img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

#### 保存图像
使用函数cv2.imwrite()保存图像
```python
cv2.imwrite('messigray.png',img)
```


## OpenCV 的GUI功能

### 鼠标作为画刷
#### 目标
- 学习处理在openCV中处理鼠标事件
- 学习使用cv2.setMouseCallback()函数

#### 简单例子
首先创建一个函数用来通过鼠标事件来回调, 使用这个函数绘制圆圈,
使用cv2.setMouseCallbace()函数来讲特定的鼠标事件与该函数进行绑定, 当鼠标事件触发时调用该函数

```python
import cv2
import numpy as np
# 鼠标事件的回调函数
def draw_circle(event, x, y, flags, param):
    if event == cv2.EVENT_LBUTTONDBLCLK:
        cv2.circle(img, (x, y), 100, (255,0,0), -1)

img = np.zeros((512, 512, 3), np.uint8)
cv2.namedWindow('image')
cv2.setMouseCallback('image', draw_circle)
while(1):
    cv2.imshow('image', img)
    if cv2.waitKey(20) & 0xFF == 27:
        break;
cv2.destroyAllWindows()
```


#### 更复杂的例子

根据鼠标移动, 画出一个矩形框, 或者一个线条
```python
import cv2
import numpy as np

drawing = False
mode = True
ix, iy = -1,-1

def draw_circle2(event, x, y, flags, param):
    global ix, iy, drawing, mode

    if event == cv2.EVENT_LBUTTONDOWN:
        drawing = True
        ix, iy = x, y
    elif event == cv2.EVENT_MOUSEMOVE:
        if drawing == True:
            if mode == True:
                cv2.rectangle(img, (ix,iy), (x,y), (0,255,0), -1)
            else:
                cv2.circle(img, (x,y), 5, (0,0,255), -1)
    elif event == cv2.EVENT_LBUTTONUP:
        drawing = False
        if mode == True:
            cv2.rectangle(img, (ix,iy),(x,y),(0,255,0), -1)
        else:
            cv2.circle(img, (x,y), 5, (0,0,255), -1)

img = np.zeros((512, 512, 3), np.uint8)
cv2.namedWindow('image')
cv2.setMouseCallback('image', draw_circle2)
while(1):
    cv2.imshow('image', img)
    k = cv2.waitKey(1) & 0xFF
    if k == ord('m'):
        mode = not mode

    elif k == 27:
        break
cv2.destroyAllWindows()
```

### 使用Trackbar 作为颜色调色板

#### 目标
- 学习将trackbar 绑定到windows
- 学习函数CV2.getTrackbarPos(), cv2.createTrackbar()

#### Code Demo
```python
import cv2
import numpy as np

def nothing(x):
    pass

#Create a black image, a Windows

img = np.zeros((300,512,3), np.uint8)
cv2.namedWindow('image')

#create trackbars for color change
cv2.createTrackbar('R', 'image', 0, 255, nothing)
cv2.createTrackbar('G', 'image', 0, 255, nothing)
cv2.createTrackbar('B', 'image', 0, 255, nothing)

#create switch for ON/OFF functionality
switch = '0 : OFF \n : ON'
cv2.createTrackbar(switch, 'image', 0, 1, nothing)

while(1):
    cv2.imshow('image', img)
    k = cv2.waitKey(1) & 0xFF
    if k == 27:
        break

    # get current positions of four trackbars

    r = cv2.getTrackbarPos('R', 'image')
    g = cv2.getTrackbarPos('G', 'image')
    b = cv2.getTrackbarPos('B', 'image')
    s = cv2.getTrackbarPos(switch, 'image')

    if s == 0:
        img[:] = 0
    else:
        img[:] = [b,g,r]

cv2.destroyAllWindows()

```

### 图像的基本操作

#### 目标
- 读写图像的像素值
- 访问图像的属性值
- 设定图像的区域(ROI)
- 分割和融合图像

#### 读写图像的像素值
```python
import cv2
import numpy as np

img = cv2.imread('messi5.jpg')
px = img[100,100]
blue = img[100,100,0]

img[100,100] =  [255,255,255]

# accessing 10,10 RED value 对应的颜色值为(b, g, r) (0, 1, 2)
red = img.item(10,10,2)

# setting 10,10 red value
img.itemset((10,10,2), 100)

```

### 图像上的算术操作

- 使用cv2.add(), cv2.addWeighted()函数来计算图像加法, 融合等

np 库中算法和cv2库中的算法不一样, np中的加法是截断, cv2中是取余
```python
x = np.uint8([250])
y = np.uint8([10])
z = cv2.add(x,y) # 250 + 10 = 260 => 255
print(z) # [[255]]

u = x + y
print(u) # [4]
```

所以对于两幅图像来说cv2库的add函数提供了更好的结果, 对于图像最好使用cv2库

#### 图像融合

使用带权重的addWeighted()函数来进行两幅图像的融合

g(x) = (1-α)f(x) + (α)g(x)

```python
img1 = cv2.imread('ml.png')
img2 = cv2.imread('opencv_log.png')

dst = cv2.addWeighted(img1, 0.7, img2, 0.3, 0)

cv2.imshow('dst', dst)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
### 图像处理
#### 改变颜色空间
- 颜色空间转换RGB<->GRAY, BGR <-> HSV 等
- 抽取视频颜色对象
- 学习使用CV2.cvtColor(), cv2.inRange()

##### 改变颜色空间
OpenCV 支持超过150 中颜色空间, 常用的两种主要是BGR <-> GRAY, BGR <-> HSV
使用函数cv2.cvtColor(intput_image, flag) 转换
flag 确定是要转换到哪种类型的颜色空间

BGR <-> GRAY  flag为 cv2.COLOR_BGR2GRAY
BGR <-> HSV flag 为 cv2.COLOR_BGR2HSV

列举所有的支持转换的颜色空间
```python
import cv2
flags = [i for i in dir(cv2) if i.startswith('COLOR_')]
print(flags)
```
##### 对象追踪(Object Tracking)
对于视频来追踪相应对象
将BRG颜色空间转成HSV颜色空间,
HSV更容易表示一个颜色.

步骤:
- 获取视频的一帧
- 转换BGR到HSV颜色空间
- 设置HSV图像中阈值来获取蓝色范围
- 抽取蓝色对象做后续处理
```python
import cv2
import numpy as np

cap = cv2.VideoCapture(0)

while(1):
  # Take each frame
  _, frame = cap.read()
  # Convert BGR to HSV
  hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
  # define range of blue color in HSV
  lower_blue = np.array([110, 50,50])
  upper_blue = np.array([130, 255,255])

  # Threshold the HSV image to get only blue colors
  mask = cv2.inRange(hsv, lower_blue, upper_blue)
  # Bitwise_AND mask and original image
  res = cv2.bitwise_and(frame, frame, mask= mask)
  cv2.imshow('frame', frame)
  cv2.imshow('mask', mask)
  cv2.imshow('res', res)
  k = cv2.waitKey(5)&0xFF
  if k == 27:
    break
  cv2.destroyAllWindows()
```


#### 图像的几何变换

#### 图像的阈值转换
- 学习函数cv2.threshold(), cv2.adaptiveThreshold

##### cv2.threshold()
假如像素值大于threshold 阈值, 像素将设定一个值(白色white),其他小于等于的设置另外一个值(黑色black).
第一个参数为输入图像, 应该是灰阶图像, 第二个参数值为阈值
第三个参数为maxVal, 第四个参数指定阈值应用的不同风格
 - cv2.THRESH_BINARY
 - cv2.THRESH_BINARY_INV
 - cv2.THRESH_TRUNC
 - cv2.THRESH_TOZERO
 - cv2.THRESH_TOZERO_INV
```python
import cv2
import numpy as np
from matplotlib import pyplot as plt

img = cv2.imread('gradient.png',0)
ret, thresh1 = cv2.threshold(img, 127, 255, cv2.THRESH_BINARY)
ret, thresh2 = cv2.threshold(img, 127, 255, cv2.THRESH_BINARY_INV)
ret, thresh3 = cv2.threshold(img, 127, 255, cv2.THRESH_TRUNC)
ret, thresh4 = cv2.threshold(img, 127, 255, cv2.THRESH_TOZERO)
ret, thresh5 = cv2.threshold(img, 127, 255, cv2.THRESH_TOZERO_INV)

titles = ['Original Image','BINARY','BINARY_INV','TRUNC','TOZERO','TOZERO_INV']
images = [img, thresh1, thresh2, thresh3, thresh4, thresh5]

for i in xrange(6):
    plt.subplot(2,3,i+1),plt.imshow(images[i],'gray')
    plt.title(titles[i])
    plt.xticks([]),plt.yticks([])

plt.show()

```

 ![](https://opencv-python-tutroals.readthedocs.io/en/latest/_images/threshold.jpg)

##### cv2.adaptiveThreshold
适应性阈值, 在小范围内计算阈值, 而不是采用全局阈值, 对于亮度不均匀的图像效果更好

输入参数
- cv2.ADAPTIVE_THRESH_MEAN_C : 阈值取周边区域的平均值
- cv2.ADAPTIVE_THRESH_GAUSSIAN_C : 阈值取周边区域进行加权的高斯计算出的结果

**Block Size** 临近区域大小

**C** 常量, 邻近区域加权计算后减去的常量
```python
import cv2
import numpy as np

from matplotlib import pyplot as plt

img = cv2.imread('img/qipan.png',0)
# img = cv2.medianBlur(img, 5)
ret, th1 = cv2. threshold(img, 127, 255, cv2.THRESH_BINARY)
th2 = cv2.adaptiveThreshold(img, 255, cv2.ADAPTIVE_THRESH_MEAN_C, cv2.THRESH_BINARY, 11, 2)
th3 = cv2.adaptiveThreshold(img, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY, 11, 2)

titles = ['Original Image', 'Globale Thresholding', 'Adaptive Mean Thresholding', 'Adaptive Gaussian Thresholding']
images = [img, th1, th2,th3]

for i in range(0,4):
    plt.subplot(2,2,i+1), plt.imshow(images[i], 'gray')
    plt.title(titles[i])
    plt.xticks([]), plt.yticks([])
plt.show()
```
结果:
![](https://opencv-python-tutroals.readthedocs.io/en/latest/_images/ada_threshold.jpg)

##### Otsu's 双值化

![](https://opencv-python-tutroals.readthedocs.io/en/latest/_images/otsu.jpg)

Otsu's Binarization 工作原理 是查找图像的直方图中的两个峰值, 阈值取这两个峰值之间的区域, 这样可以尽可能的将图像中大部分的数据图像分开.

It actually finds a value of t which lies in between two peaks such that variances to both classes are minimum.

##### 图像的几何变换
- 学习不同的图像几何变换: 平移, 旋转, 仿射变换(affine 仿射转换指的是平行线在转换后还保持平行)
- 学习函数 cv2.getPerspectiveTransform()

OpenCV 提供了两个变换函数: cv2.warpAffine, cv2.warpPerspective
cv2.warpAffine 使用一个2*3的转移矩阵
cv2.warpPerspective 使用一个3*3的转移矩阵

###### 缩放
OpenCV 使用cv2.resize()函数来实现缩放
可以制定缩放图像大小, 或者制定缩放因子
差值方式:
cv2.INTER_AREA 缩小
cv2.INTER_CUBIC cv2.INTER_LINEAR 放大

```python
import cv2
import numpy as np

img = cv2.imread('messi5.jpg')
res = cv2.resize(img, None, fx=2, fy=2, interpolation = cv2.INTER_CUBIC)
# OR
height, width = img.shape[:2]
res = cv2.resize(img, (2*width, 2*height), interpolation = cv2.INTER_CUBIC)
```
###### 移动
转移矩阵M
![](https://opencv-python-tutroals.readthedocs.io/en/latest/_images/math/22fe551f03b8e94f1a7a75731a660f0163030540.png)

使用函数cv2.warpAffine()
```python
import cv2
import numpy as np

img = cv2.imread('messi5.jpg',0)
rows, cols = img.shape

M = np.float32([[1,0,100],[0,1,50]])
dst = cv2.warpAffine(img, M, (cols, rows))

cv2.imshow('img', dst)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
###### 旋转
旋转角度设为θ, 则图像的旋转矩阵为 (原点为0,0)
![](https://opencv-python-tutroals.readthedocs.io/en/latest/_images/math/f3a6bed945808a1f3a9df71b260f68f8e653af95.png)

OpenCV提供了可以使用给点(x,y)原点坐标的旋转, 则图像的旋转矩阵为
![](https://opencv-python-tutroals.readthedocs.io/en/latest/_images/math/91ff2b9b1db0760f4764631010749e594cdf5f5f.png)

where
![](https://opencv-python-tutroals.readthedocs.io/en/latest/_images/math/383c254fc602c57a059a8296357f90fdf421aee7.png)

OpenCV 提供函数cv2.getRotationMatrix2D()来生成矩阵
```python
img = cv2.imread('messi5.jpg',0)
rows, cols = img.shape

M = cv2.getRotationMatrix2D((cols/2, rows/2), 90, 1)
dst = cv2.warpAffine(img, M, (cols, rows))
```
###### Affine Transformation(仿射变换)
仿射变换, 是指所有的在输入的平行线, 在输出时仍然保持平行
```python
img = cv2.imread('drawing.png')
rows, cols, ch = img.shape

pts1 = np.float32([[50,50],[200,50],[50,200]])
pts2 = np.float32([[10,100],[200,50], [100,250]])

M = cv2.getAffineTransform(pts1, pts2)
dst = cv2.warpAffine(img, M, (cols, rows))
plt.subpolt(121), plt.imshow(img), plt.title('Input')
plt.subplot(122), plt.imshow(dst), plt.title('Output')
plt.show()
```
![](https://opencv-python-tutroals.readthedocs.io/en/latest/_images/affine.jpg)

###### Perspective Transformation (透视变换)
需要一个3*3的矩阵,直线在变化后仍然为直线,
需要4个点来确定该矩阵, 通过函数cv2.getPerspectiveTransform()来获取
通过cv2.warpPerspective()函数应用图像变换
```python
img = cv2.imread('sudokusmaill.png')
rows,cols,ch = img.shape

pts1 = np.float32([[56,65],[368,52],[28,387],[389,390]])
pts2 = np.float32([[0,0],[300,0],[0,300],[300,300]])

M = cv2.getPerspectiveTransform(pts1, pts2)

dst = cv2.warpPerspective(img, M, (300,300))
plt.subplot(121), plt.imshow(img), plt.title('Input')
plt.subplot(122), plt.imshow(dst), plt.title('Output')
plt.show()
```
![](https://opencv-python-tutroals.readthedocs.io/en/latest/_images/perspective.jpg)

#### 图像平滑(Smoothing Images)
目标:
- 使用低通滤波器平滑图像
- 使用自定义的过滤器处理图像
##### 2D Convolution(Image Filtering)
图像可以使用低通滤波器(LPF)和高通滤波器(HPF)
低通滤波器可以移除噪点或者平滑图像
高通滤波器可以用来在图像上寻找边界

OpenCV 提供cv2.filter2D() 卷积内核来处理图像
一个5*5平均过滤器算子
![](https://opencv-python-tutroals.readthedocs.io/en/latest/_images/math/220e403e44b16ea8e05d350c4ce69e9aedff5bd1.png)

```python
import cv2
import numpy as np

img = cv2.imread('opencv_logo.png')

kernel = np.ones((5,5), np.float32)/25
dst = cv2.filter2D(img, -1, kernel)

plt.subplot(121),plt.imshow(img),plt.title('Original')
plt.xticks([]), plt.yticks([])
plt.subplot(122),plt.imshow(dst),plt.title('Averaging')
plt.xticks([]), plt.yticks([])
plt.show()
```
![](https://opencv-python-tutroals.readthedocs.io/en/latest/_images/filter.jpg)




#### 形态变换(Morphological Transformations)

#### 图像渐变(Image Gradients)

#### Canny 边缘检测

#### 图像金字塔 (Image Pyramids)

#### 轮廓提取(Contours in OpenCV)

#### 直方图(Hisograms in OpenCV)

#### 图像变换(Image Transforms in OpenCV)

#### 模板匹配(Template Matching)

#### 霍夫线性变换(Hough Line Transform)

#### 霍夫圆形变换(Hough Circle Transform)

#### 使用分水岭算法分割图像(Image Segmentation with Watershed Algorithm)

#### 使用GrabCut算法进行前景抽取(Interactive Foreground Extraction using GrabCut Algorithm)
