# 实验步骤

## 1. 编写脉动阵列IP核

&emsp;&emsp;首先，利用实验包`systolic_hls`目录下的源文件，自行建立脉动阵列的Vivado HLS工程。

&emsp;&emsp;在理解脉动阵列结构及其工作机制的前提下，根据代码注释提示，完成`systolic_array.cpp`。

!!! attention "请注意 :gun:"
    &emsp;&emsp;实验包的`systolic_array.cpp`包含有 **2个版本** 的脉动阵列，**一个是`Size-Limited version`，另一个则是`Size-Free version`**。前者最大只能支持2个$180 \times 180$矩阵的相乘，后者则通过矩阵分块实现了任意大小矩阵的相乘。

    &emsp;&emsp;实验时，应当尽可能完成`Size-Free version`的脉动阵列。如果只完成了`Size-Limited version`，则仅能得到基础的一小部分分数。

!!! info "友情提醒 :wink:"
    &emsp;&emsp;在`systolic_array.cpp`中，函数`gemm_kernel`仅实现了通用矩阵乘法。**为了支持卷积操作，需要在输出结果（即`copy_result`函数）时，为乘法结果加上偏置（bias）。**

    &emsp;&emsp;请同学们在理解脉动阵列实现卷积的原理的基础上，结合[实验2-实验原理-2.3 Tiny YOLOv2算法和网络模型](https://hitsz-cslab.gitee.io/dla/lab2/theory/#23-tiny-yolov2)中的三维卷积伪代码，自行实现`copy_result`函数。

&emsp;&emsp;代码编写完成后，参考[实验1-实验步骤-1.利用HLS生成IP核](https://hitsz-cslab.gitee.io/dla/lab1/step/#1-hlsip)，对脉动阵列进行CSim仿真。确认代码功能正确后，进行综合、导出RTL，最终得到脉动阵列IP核。

## 2. 构建Overlay

&emsp;&emsp;参考[实验1-实验步骤-2.创建Block Design](https://hitsz-cslab.gitee.io/dla/lab1/step/#2-block-design)，利用自己生成的脉动阵列IP核，以及实验包的池化IP核，自行搭建Block Design、生成比特流，并导出Overlay。

&emsp;&emsp;将导出的Overlay重命名为`mnist_systolic`，并拷贝到实验包的`systolic_app`目录中。

&emsp;&emsp;将实验包的`systolic_app`目录拷贝到PYNQ-Z2的`~\jupyter_notebooks`目录下。

## 3. GEMM测试

&emsp;&emsp;将实验包中的`systolic_app.zip`上传到Jupyter并解压。

&emsp;&emsp;点击进入`systolic_app`文件夹，运行`systolic_gemm.ipynb`以测试脉动阵列IP核的性能，记录下此时获得的加速比。

!!! warning "请注意 :loudspeaker:"
    &emsp;&emsp;如果仅实现了`Size-Limited version`的脉动阵列，则测试时需要修改测试矩阵的行数和列数，使其不超过180。

## 4. HLS Directive优化

&emsp;&emsp;回到脉动阵列的Vivado HLS工程，参照[实验2-实验原理-4.HLS优化](https://hitsz-cslab.gitee.io/dla/lab2/theory/#4-hls)，使用HLS Directive对脉动阵列进行优化。

&emsp;&emsp;优化后继续进行综合、导出RTL，得到新的脉动阵列IP核。然后将IP核更新到所构建的Block Design中，重新生成比特流，然后导出新的Overlay。

&emsp;&emsp;用新的Overlay再次运行GEMM测试，记录下次数获得的加速比，并与优化前进行对比。

## 5. 运行卷积测试

&emsp;&emsp;优化完成后，补全`systolic_conv.ipynb`中关于im2col操作的部分，然后运行卷积测试。

## 6. 运行CNN

&emsp;&emsp;通过了GEMM测试和卷积测试后，补全`mnist_cnn.ipynb`中的相关代码，并运行测试。

&emsp;&emsp;将`systolic_app.zip`的`data`目录中的1.jpg拷贝出来，使用“画图”软件打开，如图2-1所示。

<center><img src="../assets/2-1.png" width = 400></center>
<center>图2-1 打开并绘制数字</center>

&emsp;&emsp;使用橡皮擦工具擦除原来的数字，并使用刷子绘制任意数字，保存1.jpg文件。

&emsp;&emsp;将新的1.jpg上传到`systilic_app/data`目录，并再次运行推导部分的代码。此时，程序将对刚刚绘制的数字进行识别，如图2-2所示。

<center><img src="../assets/2-2.png" width = 280></center>
<center>图2-2 加速比与识别结果</center>
