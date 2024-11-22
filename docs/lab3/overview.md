# 实验目的

&emsp;&emsp;1. 加深对循环神经网络RNN基本原理的理解；

&emsp;&emsp;2. 了解LSTM识别手写数字的基本原理；

&emsp;&emsp;3. 掌握使用HLS设计实现LSTM硬件加速器的基本流程和方法。

&emsp;&emsp;

# 实验内容

&emsp;&emsp;本实验要求设计实现LSTM的硬件加速器，具体包括：

&emsp;&emsp;1. 使用HLS编写LSTM Cell、Sigmoid函数、Tanh函数、向量加法等函数，并综合、打包生成  
&emsp;&emsp;RNN加速IP核；

&emsp;&emsp;2. 在Vivado中构建Block Design电路图，生成比特流并导出Overlay；

&emsp;&emsp;3. 上板测试，观察并对比分析RNN软件推导和硬件推导的差别。
