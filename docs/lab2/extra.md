# 附加题（<font color=green>**+2**</font>分）

&emsp;&emsp;在必做题中，我们在Jupyter Notebook中绘制并观察了网络权值和输入特征图的散点图，分别如图3-1(a)、图3-1(b)所示。  

<center><img src="../assets/3-1a.png" width = 320> <img src="../assets/3-1b.png" width = 314></center>
<center>&emsp;&emsp;(a) 网络权值的散点图 &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; (b) 输入特征图的散点图</center>
<center>图3-1 权值和特征图的散点图</center>

&emsp;&emsp;观察图3-1可知，网络权值基本关于原点对称，因此必做题中的INT8线性对称量化方法已经可以将网络权值比较均匀地分布到[-127, 127]的量化区间，因而在理论上具有较好的量化效果。相比之下，特征图的散点图几乎没有对称性，而且散点图的中心轴与原点具有一定的偏移，这些特点使得特征图不适合使用线性对称的浮点数量化方法。

&emsp;&emsp;基于以上分析，本题要求如下：

&emsp;&emsp;（1）使用INT8线性非对称量化方法对输入特征图进行量化、使用INT8线性对称量化方法对权  
&emsp;&emsp;值进行量化；

&emsp;&emsp;（2）推导出线性非对称量化与反量化的公式、推导出线性对称量化后的权值W与线性非对称  
&emsp;&emsp;量化后的特征feature_in的乘积的反量化公式，并将推导过程和推导出的公式写入实验报告；

&emsp;&emsp;（3）基于所推导的量化与反量化公式，修改卷积IP核的HLS代码（含仿真代码），并在C仿真  
&emsp;&emsp;无误后，生成新的卷积IP核；

&emsp;&emsp;（4）将新的卷积IP核更新到Block Design，生成比特流并导出Overlay；

&emsp;&emsp;（5）创建Tiny_YOLOv2_Quant.ipynb副本，并重命名为Tiny_YOLOv2_Quant_EXTRA.ipynb，  
&emsp;&emsp;然后基于所推导的公式，修改第5层和第9层的量化代码；

&emsp;&emsp;（6）运行修改后的Tiny YOLOv2算法，保存输出图片output_quant.jpg，并将其重命名为  
&emsp;&emsp;output_quant_EXTRA.jpg；

&emsp;&emsp;（7）在报告中对比分析无量化、纯线性对称量化和加入了线性非对称量化之后的运行时间和  
&emsp;&emsp;预测准确度差异；

&emsp;&emsp;（8）提交作业时，需将附加题的源程序文件和运行结果（卷积IP核的HLS源文件和头文件、  
&emsp;&emsp;仿真源码文件、Tiny_YOLOv2_Quant.ipynb、output_quant_EXTRA.jpg）也一并提交。

!!! hint "参考结果 :peach:"
    &emsp;&emsp;本题的参考结果如表3-1所示。

    <center>表3-1 非对称量化参考结果</center>

    <center>

    | 量化策略 | 数据类型 | 狗 | 自行车 | 小汽车 |
    | :-: | :-: | :-: | :-: | :-: |
    | 无 | FP32 | 0.810 | 0.508 | 0.796 |
    | 线性对称 | INT8 | 0.745 | 0.422 | 0.731 |
    | 线性非对称 | IN8 | 0.830 | 0.566 | 0.827 |

    </center>
