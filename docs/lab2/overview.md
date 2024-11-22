# 实验目的

&emsp;&emsp;1. 了解利用YOLO算法进行目标检测的基本原理；

&emsp;&emsp;2. 了解浮点数量化的意义和原理，掌握浮点数的量化方法；

&emsp;&emsp;3. 掌握使用HLS Directives优化IP核性能的方法。

&emsp;&emsp;

# 实验内容

&emsp;&emsp;本实验要求实现网络卷积参数的量化与IP核的并行性优化，具体包括：

&emsp;&emsp;1. 运行量化前的Tiny YOLOv2，记录识别效果和运行时间；

&emsp;&emsp;2. 将卷积层和全连接层的网络参数进行量化，在量化后再次运行Tiny YOLOv2算法，并对比和  
&emsp;&emsp;分析量化前后网络参数的大小变化以及网络预测的准确度差异；

&emsp;&emsp;3. 使用Xilinx HLS Directives对卷积IP核与池化IP核进行并行性优化，并对比和分析优化前后的  
&emsp;&emsp;神经网络前向推导的性能差别。
