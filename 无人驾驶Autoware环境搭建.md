 有幸入选Udacity在贵阳的"无人驾驶工程师真车实训营"的12人名单，听说要使用日本名古屋大学基于ROS的无人驾驶开源软件--[Autoware](https://github.com/CPFL/Autoware)，七叔之前还没玩儿过，趁出发前的这段时间混个脸熟。

> 安装过程中某些步骤需要科学上网^_^

## OS
为什么要从操作系统开始？Github上看了个大概，官方推荐使用ubuntu系统，而且多次提到16.04版本，用Windows的旁友要么忍痛换ubuntu吧，毕竟用虚拟机挺难扛住这样的配置需求吧（Generic x86）：
```
Intel Core i7 (preferred), Core i5, Atom
16GB to 32GB of main memory
More than 30GB of SSD
NVIDIA GTX GeForce GPU (980M or higher performance)
```
基本就是把万元台式机的资源跑满了。

七叔之前玩深度学习的时候就搭了一套ubuntu，这里方便旁友们换阵营，放一个极简攻略：
* 准备一个容量≥4G的U盘
* 到[ubuntu官网](https://www.ubuntu.com/download/alternative-downloads)下载16.04版本的iso
* 到[教程](https://tutorials.ubuntu.com)页面查找如何制作安装U盘（推荐Windows使用[rufus](https://rufus.ie/en_IE.html)，MacOS使用[Ether](https://www.balena.io/etcher/)，Ubuntu使用自带的`Startup Disk Creator`
* 用制作好的U盘重启安装Ubuntu16.04吧（可参考[官方教程](https://tutorials.ubuntu.com/tutorial/tutorial-install-ubuntu-desktop?_ga=2.158281850.1592811729.1543221559-464904517.1539350579#0)）

OS安装完成后，需要先安装GPU驱动和CUDA，因为Autoware的安装对它们有依赖。

> 新装系统后建议执行`sudo apt-get install linux-headers-$(uname -r)`，以便预先安装好必要的开发包。


## GPU Driver
首先，确认你有一块可以安装CUDA的显卡`lspci | grep -i nvidia`，不然后面的事都不用做啦。然后根据自己的显卡型号无脑下载官方最新版[驱动](https://www.nvidia.cn/Download/index.aspx?lang=cn)，但别着急安装。

安装GPU驱动的关键是要**关闭自带的nouveau驱动**：
  * 创建配置文件`/etc/modprobe.d/blacklist-nouveau.conf`，填入如下内容：
```
blacklist nouveau
options nouveau modeset=0
```
  * 执行`sudo update-initramfs -u`

**重启**系统后在**登录界面即切换**至命令行环境安装驱动：
* 按`Ctrl + Alt + F1`切换到tty1的虚拟控制台命令行界面
* 执行`sudo service lightdm stop`关闭图形化环境。假设下载的是390.87版本，执行`sudo sh NVIDIA-Linux-x86_64-390.87.run`。

根据安装提示一顿点按之后，如果没有明显的报错信息，GPU驱动安装就差不多完成了。

启动图形化界面`sudo service lightdm start`，在ubuntu的terminal中执行`nvidia-smi`命令查看GPU状态，下面是在七叔的1080ti台式机上执行的返回信息，可作为参考：
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 390.87 Driver Version: 390.87                                    |
|-------------------------------+----------------------+----------------------+
| GPU Name Persistence-M        | Bus-Id Disp.A        | Volatile Uncorr. ECC |
| Fan Temp Perf Pwr:Usage/Cap   | Memory-Usage         | GPU-Util Compute M.  |
|===============================+======================+======================|
| 0 GeForce GTX 108... Off      | 00000000:01:00.0 On  | N/A                  |
| 0% 29C P5 32W / 300W          | 339MiB / 11177MiB    | 0% Default           |
+-------------------------------+----------------------+----------------------+
+-----------------------------------------------------------------------------+
| Processes: GPU Memory                                                       |
| GPU PID Type Process name Usage                                             |
|=============================================================================|
| 0 9829 G /usr/lib/xorg/Xorg 182MiB                                          |
| 0 10409 G compiz 151MiB                                                     |
| 0 10946 G /usr/lib/firefox/firefox 2MiB                                     |
+-----------------------------------------------------------------------------+
```


## CUDA
Autoware的安装主要依赖CUDA，但是为了后续方便使用Tensorflow，本节~~无违和~~插入cuDNN的安装过程，而又由于TF的版本环境依赖问题，CUDA和cuDNN的版本也需要相应考虑。当前TF官网稳定版本为1.12，它支持的CUDA版本为9.0，然后这个CUDA版本支持的cuDNN版本为7.4.1，总结一下就是如下的关系（谨作参考）：
```
Tensorflow v1.12 <-- CUDA v9.0 --> cuDNN v7.4.1
```
按照上述梳理的版本关系，下载好如下两个安装文件：
* cuda_9.0.176_384.81_linux.run  #CUDA安装文件内含显卡驱动，安装时跳过
* libcudnn7_7.4.1.5-1+cuda9.0_amd64.deb  #运行库
* libcudnn7-dev_7.4.1.5-1+cuda9.0_amd64.deb  #开发库
* libcudnn7-doc_7.4.1.5-1+cuda9.0_amd64.deb  #测试样例和用户手册

首先在**命令行界面安装CUDA**。对！和安装显卡驱动时一样……，所以建议在命令行界面中一次把驱动和CUDA都装好，省得来回切换界面。

执行CUDA的安装文件：
```
sudo sh cuda_9.0.176_384.81_linux.run
```
按照安装过程中的提示选择y/n，以及填写一些路径啥的有的没的。以下是七叔的操作参考：
```
Do you accept the previously read EULA？
accept/decline/quit: accept

Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 384.81?
(y)es/(n)o/(q)uit: n

Install the CUDA 9.0 Toolkit?
(y)es/(n)o/(q)uit: y

Enter Toolkit Location
 [ default is /usr/local/cuda-9.0 ]: 回车

Do you want to install a symbolic link at /usr/local/cuda?
(y)es/(n)o/(q)uit: y

Install the CUDA 9.0 Samples?
(y)es/(n)o/(q)uit: y

Enter CUDA Samples Location
 [ default is /home/... ]: 回车
```

安装完成后，可以在`NVIDIA_CUDA-9.0_Samples`目录下测试安装情况，例如：
```
cd 1_utilities/deviceQuery
make
./deviceQuery
```
执行后有正常返回显卡的信息即说明安装成功。

CUDA安装完成后建议配置环境变量。在`/etc/profile`文件中添加以下两行：
```
export PATH=/usr/local/cuda-9.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
系统重启后可执行`nvcc -V`查看CUDA的版本号来确认路径配置是否正确。

> 以下cuDNN的安装部分非必须，不需要的旁友可以跳过

安装cuDNN只需执行三个文件：
```
sudo dpkg -i libcudnn7_7.4.1.5-1+cuda9.0_amd64.deb
sudo dpkg -i libcudnn7-dev_7.4.1.5-1+cuda9.0_amd64.deb
sudo dpkg -i libcudnn7-doc_7.4.1.5-1+cuda9.0_amd64.deb
```
可以通过跑一下测试样例提供的mnist来检查cuDNN的安装情况：
```
cp -r /usr/src/cudnn_samples_v7/ $HOME
cd ~/cudnn_samples_v7/mnistCUDNN
make clean && make
./mnistCUDNN
```
如果看到`Test passed!`的返回信息即说明安装成功。


> Tensorflow相关的安装步骤可参考[官方指导](https://www.tensorflow.org/install/?hl=zh-cn)，这里不再赘述了

## Docker

Autoware官方墙裂推荐使用Docker环境安装（To install Autoware, we strongly recommend using our Docker environments -- 这是原话）。

> 本节基于Docker的CE版本

1. 安装Docker源

```
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88  #验证GPG
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"  #安装源
```
2. 安装Docker引擎

```
sudo apt-get update
sudo apt-get install docker-ce
sudo docker run hello-world #验证安装
```

3. 安装NVIDIA的Docker插件

```
wget https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker_1.0.1-1_amd64.deb
sudo dpkg -i nvidia-docker_1.0.1-1_amd64.deb
systemctl list-units --type=service | grep -i nvidia-docker ##验证nvidia-docker服务
sudo nvidia-docker run --rm nvidia/cuda nvidia-smi  ##验证Docker正常使用GPU资源
```

4. 在Docker上启动Autoware

```
git clone https://github.com/CPFL/Autoware.git
cd Autoware/docker/generic
sudo sh build.sh kinetic
sudo sh run.sh kinetic
```
启动之后的截图？
启动了！好鸡动！

## Demo

执行Demo需要[3D地图](https://www.autoware.ai/sample/sample_moriyama_data.tar.gz)和[ROSBAG数据](https://www.autoware.ai/sample/sample_moriyama_150324.tar.gz)，下载完成后解压：
```
tar zxfv sample_moriyama_data.tar.gz
tar zxfv sample_moriyama_150324.tar.gz
```
启动Autoware
```
cd .../Autoware/ros/
./run
```
可按照以下步骤玩耍Demo：
* 使用Autoware的模拟器管理界面，加载ROSBAG样例数据
* 点击`Play`，然后马上点击`Pause`
* 启动`RViz`
* 使用`Quick Start`管理页面，逐个加载预置脚本，这些脚本存放在`.../Autoware/docs/quick_start/`目录下

来看看Demo的效果吧


## 结语
Autoware初玩就能感觉到强大（也着实吃硬件），有着日本人在机器人系统开发方面的强大基因。这让七叔有冲动找时间试一下咱们本土的Apollo，届时搞个浅度对比，嘿嘿。

本文全过程没有专业指导，纯属自娱自乐，很可能有操作或处理得不好的地方，旁友们多包涵。待七叔在实训营中学到Autoware的专业使用技术，会及时更新本文的不妥之处。

> 第一次成稿于2018-11-27

需要深入了解Autoware的旁友们可以进入[官方网站](https://autoware.ai)慢慢研究

---
<br>
but in **THE END**, it dosen't even matter.
