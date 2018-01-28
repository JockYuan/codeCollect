# OpenCV-Python 指南

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
