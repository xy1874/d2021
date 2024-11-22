# 附加题

## 题目1：HLS优化（<font color=green>**+1**</font>分）

&emsp;&emsp;使用HLS Directives对RNN IP核进行优化，可使用循环展开、数组划分、流水线等优化策略，具体要求如下：

&emsp;&emsp;（1）使用HLS指导语句优化RNN IP核性能，对比分析优化前后的综合报告，写入实验报告；

&emsp;&emsp;（2）至少使用循环展开、数组划分、流水线、数据流、内联等至少2种优化策略；

&emsp;&emsp;（3）生成Overlay并上板测试，对比分析优化前后的性能，计算平均加速比，写入实验报告；

&emsp;&emsp;（4）优化后的RNN前向推导性能要明显比必做题高；

&emsp;&emsp;（5）提交作业时，需把本题的Overlay（mnist_lstm.tcl和mnist_lstm.bit）、源程序文件和运  
&emsp;&emsp;行结果（RNN IP核的源文件、优化前后的资源使用截图、优化前后的运行时间截图）也一并  
&emsp;&emsp;提交。

## 题目2：DMA传输优化（<font color=green>**+2.5**</font>分）

&emsp;&emsp;在必做题所设计的RNN加速器中，每次DMA仅传输一张图片到IP核，或仅从IP核读取一个维度为10的FP32向量，如图3-1所示。前者的单次DMA传输数据批量为28*28*8bit = 784B，而后者则仅仅只有40B。

<center><img src="../assets/3-1a.png" width = 650></center>
<center>(a) 通过DMA与IP核进行数据通信的Python实现</center>
<center><img src="../assets/3-1b.png" width = 300></center>
<center>(b) 对(a)中代码的形象化描述</center>
<center>图3-1 必做题的DMA通信方案</center>

&emsp;&emsp;已知在DMA的实际带宽达到饱和之前，单次DMA传输的数据批量越大，PS和PL之间进行数据通信能够获得的带宽就越大，加速器整体性能就越高。因此，如果可以将待预测的所有手写数字图片合并成一个大批量数据，一次性传输到RNN IP核进行前向推导，并一次性返回所有图片的预测结果，那么DMA的带宽将得到更充分的利用，如图3-2所示。

<center><img src="../assets/3-2.png" width = 460></center>
<center>图3-2 增大单次DMA传输的数据批量以提高数据传输效率</center>

&emsp;&emsp;基于以上分析，本题目要求：

&emsp;&emsp;（1）改写RNN硬件IP核的代码，使其支持多图片连续预测；

&emsp;&emsp;（2）生成比特流，导出Overlay并上板测试；

&emsp;&emsp;（3）修改mnist_lstm.ipynb，增加send_buf数组以存放多张待测试的图片；增加recv_buf数组  
&emsp;&emsp;以存放IP核返回的多张图片的预测结果；

&emsp;&emsp;（4）计算每张测试图片的平均前向推导时间，并计算DMA传输优化后，RNN加速器相对软件  
&emsp;&emsp;推导的加速比；

&emsp;&emsp;（5）对比分析DMA传输优化前后的加速器推导性能差别，并将DMA传输优化前后的运行结果  
&emsp;&emsp;和运行时间截图，在课程报告中进行对比分析；

&emsp;&emsp;（6）提交作业时，需把本题的Overlay（mnist_lstm.tcl和mnist_lstm.bit）、源程序文件和运  
&emsp;&emsp;行结果（RNN IP核的源文件、优化前后的运行结果和运行时间截图、mnist_lstm.ipynb）也一  
&emsp;&emsp;并提交。

!!! hint "参考结果 :peach:"
    &emsp;&emsp;优化效果参考：在图3-2中，当$n = 8$时，加速比从1.4提高到10。

## 题目3：RNN量化（<font color=green>**+3.5**</font>分）

&emsp;&emsp;在必做题中，我们利用PyTorch训练RNN网络，并将训练好的网络参数导出后保存在HLS工程的weight.h头文件中，如图3-3所示。

<center><img src="../assets/3-3.png"></center>
<center>图3-3 导出的RNN网络参数</center>

&emsp;&emsp;由图3-1可知，RNN的网络参数是以FP32格式存储的，因而最终生成的RNN加速器消耗了PYNQ-Z2的大量Block RAM存储资源，如图3-4所示。

<center><img src="../assets/3-4a.png" width = 350></center>
<center>(a) HLS综合报告中的资源消耗估计表</center>
<center><img src="../assets/3-4b.png" width = 500></center>
<center>(b) Vivado实现后的资源使用图表</center>
<center>图3-4 必做题RNN加速器的资源使用情况</center>

&emsp;&emsp;其中，HLS综合报告和Vivado综合后的资源使用情况图表都是基于代码分析得出的估计值，而Vivado实现后的资源使用图表则更接近实际值。

&emsp;&emsp;基于以上分析，本题目要求：

&emsp;&emsp;（1）绘制RNN网络参数的分布图，观察其对称性；

&emsp;&emsp;（2）根据RNN网络参数的分布情况，选择合适的量化策略，对RNN的网络参数进行量化；

&emsp;&emsp;（3）修改RNN IP核的HLS代码，使得该IP核支持参数量化后的定点运算；

&emsp;&emsp;（4）添加量化运算后的反量化代码；

&emsp;&emsp;（5）为修改后的代码添加TestBench，以测试量化网络的正确性；

&emsp;&emsp;（6）生成Overlay并上板测试，观察和对比量化前后的存储资源使用量、预测准确率和前向推  
&emsp;&emsp;导性能差别，并将量化前后的资源使用量和运行结果截图，在课程报告中进行对比分析；

&emsp;&emsp;（7）提交作业时，需把本题的Overlay（mnist_lstm.tcl和mnist_lstm.bit）、源程序文件和运  
&emsp;&emsp;行结果（RNN IP核的源文件、HLS仿真源文件、量化前后的资源使用截图、量化前后的运行  
&emsp;&emsp;时间截图）也一并提交。
