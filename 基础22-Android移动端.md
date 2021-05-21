# Android移动端

### adb工具

##### 工具说明

adb工具即Android Debug Bridge（安卓调试桥） tools。它就是一个命令行窗口，运行在5037端口，用于通过电脑端与模拟器或者真实设备交互。

![1f03436b04d1492967d1a0a763e5eceeacbc7ef2](image/1f03436b04d1492967d1a0a763e5eceeacbc7ef2.jpg)

![acfda02f47704618da807d8fb08602214e5776f2](image/acfda02f47704618da807d8fb08602214e5776f2.jpg)

使用场景：比如说你的手机不能开机了，手机又不能装sd卡，这种情况下你在其他模式连接手机，通过adb命令把rom推送到手机内存（手机内置存储），然后卡刷就可以了，如果没有adb命令的话根本无法操作手机导入rom，因为手机进不了系统更何况不能装sd卡。

##### Android权限设置

连接手机使用adb工具就要开放手机上的一些权限，操作流程如下（仅限小米手机）：

  1.将手机通过USB数据线连接到电脑

  2.开启手机开发者模式（设置——我的设备——全部参数——MIUI版本（点击5次））

  3.开启USB调试（设置——系统和设备——更多设置——开发者选项——USB调试打开）

  4.开启安全设置（设置——系统和设备——更多设置——开发者选项——USB调试（安全设置）打开）

##### 下载安装

官网下载地址：https://adbshell.com/downloads

![QQ截图20210516125124](image/QQ截图20210516125124.png)

下载好以后，解压到指定路径下面：

![QQ截图20210516151434](image/QQ截图20210516151434.png)

将解压路径配置到system-path环境变量里面：

![QQ截图20210516151648](image/QQ截图20210516151648.png)

在命令行里面就可以使用adb驱动了：

![image-20210516151738898](C:/Users/ChenZhuo/AppData/Roaming/Typora/typora-user-images/image-20210516151738898.png)

##### adb常用命令

```
adb start-server：启动adb服务

adb remount：adb挂载

adb shell：查看设备是否连接成功

adb devices：显示连接设备

adb kill-server：关闭adb服务

adb shell screencap -p /mnt/sdcard/screencap.png：屏幕截图为screencap.pn文件，存放在手机中的/mnt/sdcard路径下

adb pull /mnt/sdcard/screencap.png {} >> {}/jump.out".format(tempfile.gettempdir() + "/screenshot.png", tempfile.gettempdir())：将手机中的screencap.png文件下载到电脑的临时文件夹下，文件名称仍然为screenshot.png（tempfile需要导入）

adb shell input swipe 0 0 0 0 100：在屏幕坐标(0,0)处按压100ms

adb shell input swipe 100 200 500 600 900：900ms内从屏幕坐标(100,200)滑动到(500,600)坐标处
```

##### 可能报错

使用adb命令过程中可能会遇到如下错误：

使用 `adb devices` 连接设备成功界面，如下图：

![QQ截图20210516155932](image/QQ截图20210516155932.png)

如果出现 `adb devices unauthorized` 则是连接失败：

![874326-20180110153533738-689451892](image/874326-20180110153533738-689451892.png)

这个问题主要是调试授权没有成功（未授权状态）；一般出现这个问题时，打开你的手机，就会看到连接后弹出授权提示，需要你点击同意的密钥授权连接；没有的话，关掉usb连接，重新连接，开发者模式，usb调试打开，同意授权提示，最后重启adb服务。

### Android移动端操作

既然adb是一个命令行窗口的调试工具，那么我们就可以通过Python程序来执行adb命令，进而操作Android移动端的设备。

##### 查看连接设备

```python
import os
print(os.popen('adb devices').read())

'''
输出：
List of devices attached
21a767fa	device
'''
```

##### 查看手机文件







```
import os
import tempfile
import time
from PIL import Image

# BACKGROUND_POS = (40, 500)

SCREENSHOT_PATH = tempfile.gettempdir() + "/screenshot.png"


# 利用adb工具截图并保存
def screenshot():
    os.system("adb shell screencap -p /mnt/sdcard/screencap.png")
    os.system("adb pull /mnt/sdcard/screencap.png {} >> {}/jump.out".format(SCREENSHOT_PATH, tempfile.gettempdir()))


# 裁剪图片有效区域
left = 260
top = 1000
right = 1080
bottom = 1620
def crop():
    im = Image.open(SCREENSHOT_PATH)
    # screenshot.crop((left, top, right, bottom))
    im = im.crop((left, top, right, bottom))
    im.save('image.png')
    return im


# 遍历图片像素点
def get_gap(image):
    for i in range(0, image.size[0] - 1):
        for j in range(0, image.size[1]-1):
            if not is_pixel_equal(image, i, j):
                left = i
                return left

# 判断两个像素是否相同
def is_pixel_equal(image, x, y):
    """
    :param image1: 图⽚1
    :param image2: 图⽚2
    :param x: 位置x
    :param y: 位置y
    :return: 像素是否相同
    """
    # 取两个像素点
    pixel1 = image.load()[x, y]
    pixel2 = image.load()[x, y+1]
    threshold = 80
    # ⽐较像素点的RGB值,本⽂设置的差值在80以内,可以根据情况调整
    if abs(pixel1[0] - pixel2[0]) < threshold and abs(pixel1[1] - pixel2[1]) < threshold and abs(
            pixel1[2] - pixel2[2]) < threshold:
        return True
    else:
        return False

# 计算时间
def distance2time(distance):
    if distance <=100:
        DISTANCE_TO_TIME_RATIO = 2.50
        return int(distance * DISTANCE_TO_TIME_RATIO)
    if distance <=200:
        DISTANCE_TO_TIME_RATIO = 1.70
        return int(distance * DISTANCE_TO_TIME_RATIO)
    if distance <=300:
        DISTANCE_TO_TIME_RATIO = 1.35
        return int(distance * DISTANCE_TO_TIME_RATIO)
    if distance <=400:
        DISTANCE_TO_TIME_RATIO = 1.28
        return int(distance * DISTANCE_TO_TIME_RATIO)
    if distance <=500:
        DISTANCE_TO_TIME_RATIO = 1.25
        return int(distance * DISTANCE_TO_TIME_RATIO)
    if distance <=600:
        DISTANCE_TO_TIME_RATIO = 1.15
        return int(distance * DISTANCE_TO_TIME_RATIO)
    if distance <=700:
        DISTANCE_TO_TIME_RATIO = 1.10
        return int(distance * DISTANCE_TO_TIME_RATIO)

# 按压
def jump(touch_time):
        os.system("adb shell input swipe 0 0 0 0 {}".format(touch_time))

flag = True
while flag:
    screenshot()
    image = crop()
    distance = get_gap(image=image)
    print(f'距离{distance}')
    touch_time = distance2time(distance)
    print(f'按压时间{str(touch_time)}')
    if touch_time == 0:
        flag = False
    else:
        jump(touch_time)
        time.sleep(2)