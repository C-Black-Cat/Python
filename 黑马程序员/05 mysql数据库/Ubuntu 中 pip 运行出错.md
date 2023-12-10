# ubuntu python3 pip报错SyntaxError: invalid syntax

### 经历

 python3执行python脚本import cv2报错，发现原因是指定了python2.7目录下的cv2，后来为了在安装python3版本的[opencv](https://so.csdn.net/so/search?q=opencv&spm=1001.2101.3001.7020)，安装了pip3，结果安装opencv报错，然后升级了pip3，然后pip的所有指令都不能用了。一直报错。

![image-20221012112858558](D:\Typora\my_file\图片\image-20221012112858558.png)

### 原因

 **查了很多，大多数老外的说法是，高版本的pip不再支持python3.5**

### 解决

1. 卸载pip
    `sudo apt remove --purge python3-pip`
2. 移除pip的相关文件
    `sudo apt autoremove`
3. 下载pip的安装文件
    `curl -O https://bootstrap.pypa.io/pip/3.5/get-pip.py`
4. 执行安装文件
    `/usr/bin/python3 /home/qin/下载/rosbag2video-master/get-pip.py`
5. pip降级
    `sudo -E python3 -m pip install --upgrade "pip < 19.2"`

大功告成，希望对大家有所帮助