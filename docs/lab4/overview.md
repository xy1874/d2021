# 实验目的

&emsp;&emsp;1. 了解脉动阵列的基本原理，熟悉脉动阵列的基本结构；

&emsp;&emsp;2. 掌握脉动阵列的HLS实现方法，掌握如何利用脉动阵列加速矩阵乘法和卷积运算；

&emsp;&emsp;3. 进一步熟悉使用HLS搭建硬件加速系统的方法和流程。

&emsp;&emsp;

# 实验内容

&emsp;&emsp;本实验要求利用HLS实现脉动阵列IP核，并使用该IP核加速矩阵乘法、卷积运算和CNN的前向推导，具体包括：

&emsp;&emsp;1. 使用HLS编写脉动阵列，并对编写的代码进行CSim、综合并打包成IP核；

&emsp;&emsp;2. 利用脉动阵列IP核搭建Block Design，进行综合、实现、生成比特流并导出Overlay；

&emsp;&emsp;3. 编写Jupyter程序对矩阵乘法、卷积运算和CNN进行测试；

&emsp;&emsp;4. 使用HLS Directives对脉动阵列进行优化。
