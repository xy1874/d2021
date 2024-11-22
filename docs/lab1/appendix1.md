# 附录1：PYNQ-Z2上手指南

## 1. 烧录SD卡镜像

&emsp;&emsp;PYNQ官网提供了PYNQ-Z2开发板的[SD卡镜像文件](http://www.pynq.io/board.html)（.img）。该镜像文件中含有已安装好的嵌入式Linux系统。烧录时，打开Win32 Disk Imager工具，选择待烧录镜像文件的路径和SD卡所在的盘符，如图3-1所示。

<center><img src="../assets/3-1.png" width = 410></center>
<center>图3-1 烧写镜像</center>

&emsp;&emsp;点击“写”按钮，耐心等待烧录完成。烧录完成后，SD卡存在2个分区。第一个分区是大小约为105MB的FAT32分区，用于存放PYNQ上电启动时所需的BOOT.BIN文件和Linux镜像image.ub文件，如图3-2所示。

<center><img src="../assets/3-2.png" width = 90></center>
<center>图3-2 FAT32分区</center>

&emsp;&emsp;其中，BOOT.BIN文件中包含了FSBL和U-Boot，分别用于上电初始化板卡、引导Linux启动。

&emsp;&emsp;第二个分区是EXT4分区，存放嵌入式Linux的文件系统，如图3-3所示。

<center><img src="../assets/3-3.png" width = 550></center>
<center>图3-3 EXT4分区</center>

!!! Notes
    注意：**PYNQ-Z2实验板上的SD卡已烧录了v2.5的镜像，可直接使用**。同学可根据实际需要自行烧录其他版本的镜像。

## 2. 连接PYNQ与PC

&emsp;&emsp;使用micro USB线将PC和PYNQ的串口连接起来，然后使用以太网线将PYNQ和PC连接到同一个局域网下，如图3-4所示。

<center><img src="../assets/3-4.png" width = 500></center>
<center>图3-4 连接PYNQ和PC</center>

&emsp;&emsp;连接网线时，只要令PC与PYNQ处于同一局域网即可。因此，既可以像图3-4所示直接使用网线连接PYNQ与PC，也可以将PYNQ和PC通过交换机、路由器LAN口连接。

## 3. 启动PYNQ平台

&emsp;&emsp;将PYNQ与PC连接完成后，接上PYNQ电源。需要注意的是，PYNQ既可以使用USB供电，也可以使用12V电源适配器供电。为了保证PYNQ可以正常运作，**建议同学们统一使用12V电源适配器供电**。

&emsp;&emsp;**启动PYNQ前，检查“PYNQ-Z2”右侧的启动方式跳线帽是否插在“SD”标签处（即最左侧）**，如图3-5所示。

<center><img src="../assets/3-5.png" width = 400></center>
<center>图3-5 启动方式跳线帽</center>

&emsp;&emsp;电源连接完成后，打开图3-6所示的电源开关。

<center><img src="../assets/3-6.png" width = 300></center>
<center>图3-6 PYNQ电源开关</center>

&emsp;&emsp;在PC的设备管理器中查看PYNQ开发板所使用的串口号，如图3-7所示。

<center><img src="../assets/3-7.png" width = 400></center>
<center>图3-7 查看PYNQ所使用的COM口</center>

&emsp;&emsp;打开串口调试助手SecureCRT，新建串口连接，如图3-8所示。

<center><img src="../assets/3-8.png" width = 300></center>
<center>图3-8 在SecureCRT中新建串口连接</center>

&emsp;&emsp;连接图3-7所示的串口，并按图3-9所示进行串口参数设置。参数设置完毕后，点击“连接”按钮。

<center><img src="../assets/3-9.png" width = 300></center>
<center>图3-9 串口参数设置</center>

&emsp;&emsp;连接串口后，即可在串口助手SecureCRT中查看嵌入式Linux的启动信息，如图3-10所示。Linux启动完毕后，即是Linux的命令行终端。

<center><img src="../assets/3-10.png" width = 500></center>
<center>图3-10 查看Linux启动信息及命令行终端</center>

## 4. 访问Jupyter

&emsp;&emsp;将PC的IP设置成和PYNQ同一网段，即设置成192.168.2.XX，如图3-11所示。注意XX不能是99，因为PYNQ默认的IP是192.168.2.99。

<center><img src="../assets/3-11.png" width = 400></center>
<center>图3-11 设置PC的IP地址</center>

&emsp;&emsp;在PC的Web浏览器中访问192.168.2.99:9090，输入密码xilinx，即可登录Jupyter Notebook，如图3-12所示。

<center><img src="../assets/3-12.png" width = 500></center>
<center>图3-12 登录Jupyter Notebook</center>

## 5. 编写测试代码

&emsp;&emsp;在Jupyter中新建一个Python3的Notebook，如图3-13所示。

<center><img src="../assets/3-13.png"></center>
<center>图3-13 新建Notebook</center>

&emsp;&emsp;新建完毕后可见如图3-14所示的页面。

<center><img src="../assets/3-14.png"></center>
<center>图3-14 新建Notebook成功后的页面</center>

&emsp;&emsp;编写程序测试IP核之前，需要先导入Overlay。假设现有一Overlay名为base（包含base.tcl、base.bit两个文件），则需要创建一个base对象以导入比特流base.bit，如图3-15所示。

<center><img src="../assets/3-15.png" width = 600></center>
<center>图3-15 导入BaseOverlay包、创建对象并加载比特流</center>

&emsp;&emsp;点击上图所示的cell，并点击菜单栏的Run按钮以运行代码，如图3-16所示。

<center><img src="../assets/3-16.png" width = 500></center>
<center>图3-16 运行cell中的代码</center>

&emsp;&emsp;上述代码在运行完毕前，cell左侧的“<font color=blue>In  [1]</font>”将显示为“<font color=blue>In  [*]</font>”。该cell中的代码使用base.bit的比特流文件烧写PL端的FPGA，大约需执行10s左右，同学们需耐心等待。

&emsp;&emsp;导入Overlay后，还需编写一个简单的驱动函数以实现测试程序与IP核的数据交互。
