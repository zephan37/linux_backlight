# linux_backlight
## 解决笔记本下Linux的亮度问题
个人电脑安装Linux时出现使用Fn无法调节亮度的问题，具体如下：

1./sys/class/backlight下有两个与亮度相关文件夹，对于我来说是intel_backlight与panasonic

2.使用xbacklight命令可以调节亮度

3.使用Fn调节亮度时，/sys/class/backlight下的intel_backlight的brightness未被改变，但亮度调节主要与此相关。

解决方法：
1.建立/etc/udev/rules.d/99-writeintelbacklight.rules这个文件，里面加上
```
ACTION=="change", SUBSYSTEM=="backlight", RUN+="/usr/sbin/writeintelbacklight.sh"
```
2.建立/usr/sbin/writeintelbacklight.sh文件，里面加上如下:
```
#!/bin/bash
intelmaxbrightness=`cat /sys/class/backlight/intel_backlight/max_brightness`
acpimaxbrightness=`cat /sys/class/backlight/panasonic/max_brightness`
scale=`expr $intelmaxbrightness / $acpimaxbrightness`
acpibrightness=`cat /sys/class/backlight/panasonic/brightness`
newintelbrightness=`expr $acpibrightness \* $scale`
curintelbrightness=`cat /sys/class/backlight/intel_backlight/actual_brightness`
if [ "$newintelbrightness" -ne "$curintelbrightness" ]
then
        echo $newintelbrightness > /sys/class/backlight/intel_backlight/brightness
fi
exit 0
```
3.赋予可执行权限
```
sudo chmod +x /usr/sbin/writeintelbacklight.sh
```
