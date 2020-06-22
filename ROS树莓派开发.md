# ROS树莓派开发

## 1. 开始前的准备

### 1.1 为RaspberryPi 4B安装系统

#### 1.1.1 烧录Raspberry Buster系统到Micro SD卡

先去官网下载树莓派官方系统，有如下三个版本：

![img](https://img-blog.csdnimg.cn/20190816190921987.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9uaWVzb24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

Lite版本是最小化安装，没有桌面环境；Desktop版本则带有桌面； Desktop and recommended software版本还带有推荐软件，但比较大。选择第一个版本，点击Download下载压缩包并解压，得到扩展名为.img的系统镜像文件。

下载SD Card Formatter和Win32DiskImager这两个软件，将内存卡装入读卡器连接电脑。

![image-20191204153643872](C:\Users\hp\AppData\Roaming\Typora\typora-user-images\image-20191204153643872.png)

首先用SD Card Formatter对内存卡进行格式化，按照默认设置就行。格式化后打开Win32DiskImager进行系统刷入。

![image-20191204153830145](C:\Users\hp\AppData\Roaming\Typora\typora-user-images\image-20191204153830145.png)

选择之前下载的.img文件进行刷入即可。

#### 1.1.2 树莓派基本配置

将内存卡插入树莓派卡槽后，将树莓派连接显示器并接上电源。上电后进入开机界面，地区选择china，勾选US keyboard选项。连接iSMU的wifi后会提醒restart，点击重启。

重启后先安装一些工具。首先打开终端窗口，安装gedit和vim。命令分别是

```
sudo apt-get install gedit
```

```
sudo apt-get install vim
```

之后再装一个Linux多终端工具，命令为

```
sudo apt-get install terminator
```

然后更换树莓派的软件源和系统源。第一步，先备份源文件。

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

```
sudo cp /etc/apt/sources.list.d/raspi.list /etc/apt/sources.list.d/raspi.list.bak
```

第二步，编辑系统的源文件。

```
sudo nano /etc/apt/sources.list
```

第三步，将初始的源用#注释掉，添加如下两行清华的源。

```
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
```

```
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
```

第四步，按ctrl+x退出，然后选择Y确认更改。

![img](https://img-blog.csdnimg.cn/20190819201237418.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9uaWVzb24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

第五步，最后执行

```
sudo apt-get update
```

```
sudo apt-get upgrade
```

更新完之后，安装远程桌面

```
sudo apt-get install xrdp
```

最后，安装中文字体

```
sudo apt-get install ttf-wqy-zenhei ttf-wqy-microhei
```

然后执行

```
sudo reboot
```

至此，树莓派系统的基本配置完成。

### 1.2 安装ROS系统

#### 1.2.1 编译器切换

首先安装gcc5和g++5防止编译报错。先查看默认的gcc和g++

打开终端输入

```
gcc -v
g++ -v
```

系统默认为gcc 8的版本。此时安装gcc 5和g++ 5

```
sudo apt-get install gcc-5 g++-5
```

安装好之后进行版本切换，将系统的gcc与g++切换至5，对gcc的切换执行如下操作

```
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 60
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 20
```

接着输入

```
sudo update-alternatives --config gcc
```

会出现三个选项，输入对应的编号，按回车即可。

然后再测试gcc版本

```
gcc -v
```

如果版本改变则切换成功。

#### 1.2.2 准备工作

第一步，添加ROS仓库

```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

```
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```

然后执行

```
sudo apt-get update
sudo apt-get upgrade
```

进行更新

第二步，安装Bootstrap依赖项

```
sudo apt-get install -y python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential cmake
```

然后初始化rosdep

```
sudo rosdep init
rosdep update
```

如果这里报错，是因为rosdep版本错误导致的，执行

```
sudo apt-get install rosdep
```

即可。

#### 1.2.3 编译并安装libboost1.58

下载并解压代码

```
wget -O boost_1_58_0.tar.bz2 http://sourceforge.net/projects/boost/files/boost/1.58.0/boost_1_58_0.tar.bz2/download
```

```
tar --bzip2 -xvf boost_1_58_0.tar.bz2
```

进入到解压目录

```
cd boost_1_58_0
```

进行编译

```
./bootstrap.sh --with-libraries=all --with-toolset=gcc
```

```
./b2 toolset=gcc -j2
```

进行安装

```
sudo ./bjam install 
```

添加路径

```
sudo ldconfig
```

至此，boost库安装完成。

#### 1.2.4 安装ROS

第一步，创建一个catkin工作空间

```
mkdir -p ~/ros_catkin_ws
cd ~/ros_catkin_ws
```

接下来，下载ros核心包源码并构建

```
rosinstall_generator ros_comm --rosdistro kinetic --deps --wet-only --tar > kinetic-ros_comm-wet.rosinstall
```

```
wstool init src kinetic-ros_comm-wet.rosinstall
```

使用rosdep安装所需依赖

```
rosdep install -y --from-paths src --ignore-src --rosdistro kinetic -r --os=debian:buster
```

编译工作空间

```
sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/kinetic -j2
```

添加路径到环境变量

```
source /opt/ros/kinetic/setup.bash
```

测试安装，打开终端运行

```
roscore
```

若无报错，则成功安装

## 2. ROS项目编写

