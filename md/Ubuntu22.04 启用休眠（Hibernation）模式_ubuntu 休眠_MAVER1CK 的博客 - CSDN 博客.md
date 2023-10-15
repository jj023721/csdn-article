> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_35395195/article/details/125650465?spm=1001.2014.3001.5506)

#### 文章目录

*   [1. 找出交换文件所在分区的 UUID](#1_UUID_2)
*   [2. 找出交换文件的偏移量](#2__9)
*   [3. 编辑 /etc/default/grub](#3__etcdefaultgrub_16)
*   [4. 更新 GRUB 配置](#4_GRUB_31)
*   [5. 编辑 /etc/initramfs-tools/conf.d/resume](#5__etcinitramfstoolsconfdresume_38)
*   [6. 重新生成 initramfs](#6__initramfs_54)
*   [7. 重启电脑，然后使用以下命令测试](#7__60)
*   *   [注意](#_65)
*   [8. 为休眠模式添加图标](#8__82)

使用了 swap 文件，大小应大于内存，具体设置方法参见

[Ubuntu 调整 swap 大小](https://blog.csdn.net/qq_35395195/article/details/125616388)

1. 找出交换文件所在分区的 UUID
-------------------

```
findmnt -no UUID -T /swapfile
```

![](https://img-blog.csdnimg.cn/74ff4e4c4f324573a13df526aa0fab17.png#pic_center)  
在此示例中为 a742e8c6-fd89-46d3-8e75-e518121f08cc

2. 找出交换文件的偏移量
-------------

```
sudo filefrag -v /swapfile
```

![](https://img-blog.csdnimg.cn/ae83d2ee37d9414693a69e48e463680b.png#pic_center)  
复制 physical_offset: 中第一行第一列（0:）的值，不含后面的两个 “.”，在此示例中是 27680768 。

3. 编辑 /etc/default/grub
-----------------------

```
sudo vim /etc/default/grub
```

![](https://img-blog.csdnimg.cn/7a43c016e14040eda8124aa9fec40f57.png#pic_center)  
在 GRUB_CMDLINE_LINUX_DEFAULT 这一行，向 splash 后添加以下内容

```
resume=UUID=第一步中获得的UUID resume_offset=第二步中获得的偏移量
```

例如在此示例中为

```
resume=UUID=a742e8c6-fd89-46d3-8e75-e518121f08cc resume_offset=27680768
```

![](https://img-blog.csdnimg.cn/82e9f3643ee8456288b1ece9faddae12.png#pic_center)

4. 更新 GRUB 配置
-------------

```
sudo update-grub
```

![](https://img-blog.csdnimg.cn/5b6ceacddb4e4c638236e906941eaad9.png#pic_center)

5. 编辑 /etc/initramfs-tools/conf.d/resume
----------------------------------------

```
sudo vim /etc/initramfs-tools/conf.d/resume
```

![](https://img-blog.csdnimg.cn/aebc251a430949ea8c959d01743f2d7c.png#pic_center)

如果文件存在且有一行以 RESUME 开头，则编辑该行，没有则添加内容，内容如下所示

```
RESUME=UUID=第一步中获得的UUID resume_offset=第二步中获得的偏移量
```

在本示例中为

```
RESUME=UUID=a742e8c6-fd89-46d3-8e75-e518121f08cc resume_offset=27680768
```

![](https://img-blog.csdnimg.cn/f71b333fad384d31a36d6734a710fe27.png#pic_center)

6. 重新生成 initramfs
-----------------

```
sudo update-initramfs -c -k all
```

![](https://img-blog.csdnimg.cn/b628ba23057946af91fff86a3b8ced2f.png#pic_center)

7. 重启电脑，然后使用以下命令测试
------------------

```
sudo systemctl hibernate
```

### 注意

*   若安装了 NVIDIA 驱动且安装了 CUDA  
    ![](https://img-blog.csdnimg.cn/f561f4ce26c448e59617985b2eae7639.png#pic_center)
    
    则需删除 /etc/systemd/system 下的三个文件
    
    ```
    nvidia-hibernate.service
    nvidia-resume.service
    nvidia-suspend.service
    ```
    
    然后执行
    
    ```
    systemctl daemon-reload
    ```
    
    最后才能成功使用挂起或休眠
    
*   和 Windows 不同的是，若休眠时电脑外接了设备，例如显示器，或连接有外设，下次启动时如果更换了外接设备（显示器或外设等），则有可能启动失败，**这时需要按住 alt + Sysrq 键，然后依次在键盘上按下 r e i s u b（即 busier 倒过来） ，系统就会安全重启**。因此**建议休眠时先拔掉所有外接设备再休眠**，启动时是否外接设备则不影响
    

8. 为休眠模式添加图标
------------

```
sudo vim /usr/share/applications/hibernation-mode.desktop
```

写入以下内容

```
[Desktop Entry]
Type=Application
Name=Hibernation Mode
GenericName=Hibernation Mode
Comment=Enter Hibernation Mode
NoDisplay=false
Icon=drive-multidisk
Exec=systemctl hibernate
Terminal=true
Categories=System;Utility;Settings;
```

即可在应用程序菜单看到图标，或在菜单内搜索 Hibernation Mode ，点击即可进入休眠模式