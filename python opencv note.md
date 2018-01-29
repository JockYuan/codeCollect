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
