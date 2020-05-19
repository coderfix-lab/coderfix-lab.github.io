---
layout: post
title:  "【RaspberryPi】使用0.91寸显示屏SSD1306展示想要的内容"
date:   2020-05-16 23:33:00 +0800
categories: RaspberryPi
---
本文为本人原创，首发地址 https://coderfix.blog.csdn.net/article/details/104415208



##  设备
- 树莓派3B+
- 0.91英寸显示屏SSD1306

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220180653621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)

## 接线

这里给出一个树莓派的针脚图

![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy82NDQ5MjAzLWQ3ZTNiNmU2YWNhNTFhY2MucG5n?x-oss-process=image/format,png)
一般接入设备，需要两部分

- 电源，正极(3.3v 5v)
- 数据，输入输出

屏幕 GND 接树莓派 GND
屏幕 VCC 接树莓派 3V3
屏幕 SDA 接树莓派 SDA
屏幕 SCL 接树莓派 SCL

如图
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020022104243284.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)

## 远程连接树莓派
我个人推荐远程连接树莓派设备，毕竟它不会一直外接显示器鼠标键盘。

下面的前提是你已经为设备连接上了路由器，安装树莓派系统可以参考这一篇 https://blog.csdn.net/diandianxiyu_geek/article/details/78949393 

推荐 IP Scanner查找设备

ssh连接设备

```
xiaoyu@localhost ~ % ssh pi@192.168.0.118
ssh: connect to host 192.168.0.118 port 22: Operation timed out
xiaoyu@localhost ~ % ssh pi@192.168.0.118
pi@192.168.0.118's password: 
Linux xiaoyupi 4.19.66-v7+ #1253 SMP Thu Aug 15 11:49:46 BST 2019 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Feb 19 22:17:30 2020 from 192.168.0.115

```

当然也可以使用vnc连接设备

##  开启I2C
```
sudo apt-get install -y python-smbus
sudo apt-get install -y i2c-tools
sudo raspi-config
```
选择第5项
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200221041741578.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)
打开I2c
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200221041829969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)

然后重启设备

## 检查设备是否连接成功

`sudo i2cdetect -y 1`

```
pi@xiaoyupi:~ $ sudo i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- 3c -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- -- 
```
这样表示设备的位置是3c,表示连接成功。

## 安装对应库
```
sudo python -m pip install --upgrade pip setuptools wheel

sudo apt-get install python-pil python3-pil

sudo pip install Adafruit-SSD1306
```

拉取官方事例

```
git clone https://github.com/adafruit/Adafruit_Python_SSD1306.git
```

## 展示内容
examples文件夹内是事例，我们可以根据这个去修改对应内容，`stat3.py`是我自己复制出来的。
```
pi@xiaoyupi:~/Adafruit_Python_SSD1306/examples $ ls
animate.py  font21449.rar         happycat_oled_64.ppm  shapes.py  stats.py
buttons.py  happycat_oled_32.ppm  image.py              stat3.py
pi@xiaoyupi:~/Adafruit_Python_SSD1306/examples $ python animate.py 
Press Ctrl-C to quit.
```

这样我们就完成了显示屏的接入。

## 总结
-  树莓派的价值在于连接各种硬件展示读取数据
- python的价值在于大量的外部库，而不是它的语法本身
- 本来我还买了光敏和温度传感器，但是发现买的不对，只能返回高低电平，囧
- 本系列后续还打算做，温度湿度传感器-对接阿里云物联网平台
