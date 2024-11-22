# 实验原理

## 1. 脉动阵列概述

### 1.1 背景简介

&emsp;&emsp;脉动阵列最初提出是为了解决VLSI（Very Large Scale Integration，超大规模集成电路）片上通信存在的性能瓶颈问题，属于面向特定领域而专门设计的特殊架构。脉动阵列的设计者H. T. Kung提出了[3点设计专用系统时需要考虑的因素](http://www.eecs.harvard.edu/~htk/publication/1982-kung-why-systolic-architecture.pdf)：

- （1）简单性和规律性

&emsp;&emsp;Kung认为在设计专用系统时，首先应当考虑的是系统的成本或代价。换言之，在满足应用需求的前提下，应当尽可能将系统的成本控制到最低。一般而言，系统的成本包括设计成本和器件成本，而其中又以设计成本为主。我们在数字逻辑设计课程中讲授过系统的模块化设计。试想如果一个系统可以被划分成若干个功能几乎完全相同的子模块，那么这个系统就可以使用一个简单的、有规律的硬件架构来实现。得益于简单性和规律性，系统将具有较低的设计成本——对于使用HDL开发电路而言，重复的模块只需要不断实例化同一个module即可。

!!! note "记笔记 :triangular_ruler:"
    &emsp;&emsp;除了降低设计成本外，简单性和规律性还有利于提高系统的可扩展性。

- （2）并行和通信

&emsp;&emsp;由Amdahl定律的延伸可知，不管是通用系统还是专用系统，只要系统中具有可并行处理的任务，那么只要使用流水线、多处理器等并行技术来加速这些任务，系统的总体性能总能够得到相应程度的提升。另外，当系统中存在大量同时工作的子模块时，子模块之间的协作和通信效率将对系统的性能造成不可忽视的影响。为了避免通信成为性能瓶颈，Kung提出的解决思路是使得子模块的通信和控制尽可能简单和规律。

- （3）平衡计算与I/O

&emsp;&emsp;数据需要通过特定的物理接口输入到系统进行处理；处理所得结果也需要通过相应的物理接口进行输出。当I/O速率跟不上处理单元的数据运算速率时，I/O便成为了系统的性能瓶颈。一般而言，消除I/O瓶颈的方法是优化I/O通道所采用的通信协议，或改用支持更高带宽的I/O接口。这两种方法行之有效的前提条件是I/O接口所支持的最大物理带宽不小于处理单元的吞吐率。针对这个问题，Kung的方案是增加处理单元的数量，让数据在系统中多停留一段时间，从而在I/O带宽维持不变的前提下增加系统在单位时间内的数据处理能力，如图1-1所示。

<center><img src="../assets/1-1.png" width = 400></center>
<center>图1-1 平衡计算与I/O的基本原理</center>

&emsp;&emsp;假设在图1-1中，数据在存储器和PE之间传输需要耗费`100ns`，PE的运算延迟忽略不计。在架构1中，每`200ns`才能完成一次数据运算，因而其运算能力是5MOPS（Million Operations Per Second）。在架构2中，数据传输时延不变，但每次传输可以完成6次数据运算，因而其运算能力相比架构1提高到30MOPS。

!!! hint "小提示 :bulb:"
    &emsp;&emsp;图1-2所示的平衡计算与I/O的方法同时也体现了系统设计的简单性和规律性。

&emsp;&emsp;需要注意的是，虽然理论上架构2的PE数量越多，其运算能力越强，但不论如何，数据传输时延都需要200ns。这意味着这种类型的架构只适合那些对传输带宽要求较低的计算密集型应用。

### 1.2 拓扑结构

&emsp;&emsp;脉动阵列的定义有很多版本，但一般指的是一种按照特定规律连接起来的流水式同构多处理器架构。脉动阵列由若干个完全相同的PE（Processing Element，处理单元）构成，并且一般每一个PE都仅与其相邻的PE进行连接。如图1-2所示。

<center><img src="../assets/1-2.png" width = 650></center>
<center>图1-2 脉动阵列的几种拓扑结构</center>

&emsp;&emsp;在图1-2中，不管采用哪一种连接方式，数据都按照一定的规律有节奏地在阵列中传播。当数据传播到一定次数之后，脉动阵列完成计算并产生所需的结果。

&emsp;&emsp;脉动阵列的结构具有良好的简单性和规律性，其基本特点是各个PE完全相同，并且PE只与其相邻的PE进行数据通信。与数据广播所需的总线连接方式相比，脉动阵列采取的连接方式具有更短的物理线路，更低的扇出，不仅有利于降低PE间的通信延迟，还降低了电路布局布线的难度，从而提高了电路的工作频率。但是，脉动阵列的结构同时也限制了它只适合用于带宽要求较低且运算具有较大规律性的应用场合，比如矩阵乘法、卷积、排序、多项式计算等。

### 1.3 工作原理

&emsp;&emsp;脉动阵列本质是结构简单的流水线，其工作过程可看作是数据在时钟的驱动下像脉搏一般在阵列中向前跳动，如图1-3所示。

<center><img src="../assets/1-3.png" width = 450></center>
<center>图1-3 脉动阵列工作原理</center>

## 2. GEMM的脉动阵列实现

&emsp;&emsp;GEMM（GEneral Matrix Multiply，通用矩阵乘法）中存在大量的MAC（Multiply and ACcumulate, 乘累加）运算，具有运算量较大、运算种类单一的特点，因而非常适合使用脉动阵列来实现GEMM。

&emsp;&emsp;设有矩阵$\pmb A_{3 \times 3}$、$\pmb B_{3 \times 4}$：

$$
\pmb A = \left[
    \begin{matrix}
        a_{00} & a_{01} & a_{02} \\
        a_{10} & a_{11} & a_{12} \\
        a_{20} & a_{21} & a_{22}
    \end{matrix}
    \right],{\quad}
\pmb B = \left[
    \begin{matrix}
        b_{00} & b_{01} & b_{02} & b_{03} \\
        b_{10} & b_{11} & b_{12} & b_{13} \\
        b_{20} & b_{21} & b_{22} & b_{23}
    \end{matrix}
    \right].
$$

&emsp;&emsp;使用脉动阵列计算矩阵$\pmb C = \pmb A \cdot \pmb B$时，需要将$\pmb A$、$\pmb B$的数据分多次输入阵列，如图1-4所示。

<center><img src="../assets/1-4.png" width = 540></center>
<center>图1-4 脉动阵列计算矩阵乘法的初始状态</center>

&emsp;&emsp;第1拍时，左侧和上方缓存内的矩阵数据开始流入脉动阵列，并开始计算$c_{00}$，如图1-5所示。

<center><img src="../assets/1-5.png" width = 580></center>
<center>图1-5 脉动阵列计算矩阵乘法的第1个时钟</center>

&emsp;&emsp;同样的，第2拍时，缓存内的数据继续流入脉动阵列，并开始计算$c_{01}$和$c_{10}$、继续计算$c_{00}$，如图1-6所示。

<center><img src="../assets/1-6.png" width = 620></center>
<center>图1-6 脉动阵列计算矩阵乘法的第2个时钟</center>

&emsp;&emsp;第3拍时，开始计算$c_{02}$、$c_{11}$和$c_{20}$，继续计算$c_{01}$和$c_{10}$，而此时$c_{00}$计算完毕，如图1-7所示。

<center><img src="../assets/1-7.png"></center>
<center>图1-7 第3个时钟时$c_{00}$计算完毕</center>

&emsp;&emsp;依此类推，第4拍时$c_{01}$和$c_{10}$计算完成；第5拍时$c_{02}$、$c_{11}$和$c_{20}$计算完成，等等。直到第8拍时，矩阵$\pmb C$的最后一个元素$c_{23}$计算完成。

## 3. 卷积的脉动阵列实现

&emsp;&emsp;卷积与矩阵乘法类似，也含有大量的MAC运算，同样具有运算量较大、运算种类单一的特点，因而也适合使用脉动阵列来实现。

&emsp;&emsp;一维卷积可以通过递推公式$y_{n+1} = y_n + w_{n+1} \cdot x_{n+1}$很容易地得出相应的一维脉动阵列架构的实现。因此，此处仅讨论如何使用脉动阵列实现三维卷积。

### 3.1 im2col操作

&emsp;&emsp;为了方便处理，在使用脉动阵列实现三维卷积时，需要先对卷积的输入特征图进行im2col处理。卷积时，卷积核相当于一个滑动窗口，不停地在输入特征图上从左到右、从上到下滑动。所谓im2col处理，就是将当前滑动窗口下的特征图数据进行展开和重排，如图1-8所示。

<center><img src="../assets/1-8.png" width = 600></center>
<center>图1-8 im2col原理</center>

&emsp;&emsp;类似地，卷积核也需要进行相应的展开操作。需要注意的是，输入特征图滑动窗口内的数据被展开成列向量，而卷积核则被展开成行向量。

&emsp;&emsp;设有3通道的输入特征图$(\pmb R, \pmb G, \pmb B)$，以及2个卷积核$\pmb f_0$和$\pmb f_1$，分别对它们进行im2col操作之后，得到相应的展开后的矩阵数据，如图1-9所示。

<center><img src="../assets/1-9.png"></center>
<font size = 2>注：右下角的矩阵$\pmb Y$对应于卷积运算的结果（输出特征图的尺寸是2$\times$2、2通道）。</font>
<center>图1-9 im2col处理前后的特征图和卷积核</center>

### 3.2 方法1：固定卷积核

&emsp;&emsp;展开完成后，一种方法是将卷积核的权值数据存储在脉动阵列的PE当中，然后令特征图数据从左侧输入脉动阵列并向右传播、令卷积中间结果从上往下传播。此时，脉动阵列的初始状态如图1-10所示。

<center><img src="../assets/1-10.png"></center>
<center>图1-10 脉动阵列计算三维卷积的初始状态</center>

&emsp;&emsp;第1拍时，特征图数据从左侧的缓存流入脉动阵列，开始计算$y_{00}$，如图1-11所示。

<center><img src="../assets/1-11.png"></center>
<center>图1-11 脉动阵列计算三维卷积的第1个时钟</center>

&emsp;&emsp;第2拍时，开始计算$y_{01}$和$y'_{00}$，而$y_{00}$则向下传播，如图1-12所示。

<center><img src="../assets/1-12.png"></center>
<center>图1-12 脉动阵列计算三维卷积的第2个时钟</center>

&emsp;&emsp;依此类推，一直到第12拍时，$y_{00}$计算完成并从脉动阵列下方流出，如图1-13所示。

<center><img src="../assets/1-13.png"></center>
<center>图1-13 脉动阵列计算三维卷积的第12个时钟</center>

&emsp;&emsp;接下来，从第13拍到第15拍，每一拍都流出2个矩阵$\pmb Y$的元素。最后一个元素将在第16拍流出，此时计算结束。

!!! warning "注意 :gun:"
    &emsp;&emsp;矩阵$\pmb Y$的尺寸和形状与输出特征图的形状不同。因此，在真正输出结果之前，需要对矩阵$\pmb Y$的元素进行重新排布，从而将其还原成输出特征图应有的尺寸和形状。

### 3.2 方法2：矩阵乘法

&emsp;&emsp;显然，要想用脉动阵列实现三维卷积，除了上述方法之外，还可以在im2col操作之后，直接将其当成普通的矩阵乘法处理。

!!! question "动动脑筋 :dizzy:"
    &emsp;&emsp;请对比上述两种方法的优缺点。


## 3. 基于CNN识别MNIST手写数字

### 3.1 CNN的搭建、训练与测试

&emsp;&emsp;本实验使用一个四层的CNN网络来实现手写数字0-9的识别。相关代码在虚拟机`~/tensorflow/MNIST`的目录下，训练代码为`mnist_int16_test.py`和`input_data.py`。

&emsp;&emsp;在终端中输入如下命令，即可开始网络训练：

``` Bash
$> cd /home/cs/tensorflow/MNIST
$> python mnist_int16_test.py
```

&emsp;&emsp;本实验所使用的CNN，其网络结构如图1-14所示。

<center>

<table><tbody>
    <tr><th>Convolutional Layer1 + ReLU + MAX Pooling</th></tr>
    <tr><th>Convolutional Layer2 + ReLU + MAX Pooling</th></tr>
    <tr><th>Fully Connected Layer1 + ReLU + Dropout</th></tr>
    <tr><th>Fully Connected Layer2 To Prediction</th></tr>
</table> 

</center>
<center>图1-14 本实验的CNN网络结构</center>

&emsp;&emsp;接下来，逐一介绍CNN权重初始化以及各网络层的实现方法。

#### &clubs; 权重初始化

&emsp;&emsp;建立模型时，需要对网络权值和偏置进行初始化。初始化权值时，应加入少量噪声来打破对称性和避免零梯度。由于我们使用了ReLU神经元，因此比较好的做法是用一个较小的正数来初始化网络偏置，以避免神经元节点输出恒为0的问题（Dead Neurons）。

&emsp;&emsp;为了避免建立模型时的反复初始化，定义两个初始化函数，如图1-15所示。

<center><img src="../assets/1-15.png" width = 450></center>
<center>图1-15 初始化函数</center>

#### &clubs; 卷积和池化

&emsp;&emsp;TensorFlow在卷积和池化上有很强的灵活性。卷积使用1步长（Stride）、0边距（Padding）的模板，保证输出的维度和输入相同。池化则使用传统的2×2模板做最大池化。为了代码更简洁，这部分被定义成函数，如图1-16所示。

<center><img src="../assets/1-16.png" width = 600></center>
<center>图1-16 卷积和池化的配置函数</center>

#### &clubs; 卷积层搭建

&emsp;&emsp;第一层由一个卷积接一个最大池化完成。卷积在每个3×3的Patch中算出16个特征。卷积的权重张量形状是[3, 3, 1, 16]，前两个维度是Patch的大小，接着分别是输入和输出的通道数。每个输出通道都有一个与之对应的偏置量。接下来将卷积核与输入的x_image进行卷积，并通过ReLU激活函数，再做最大池化处理，如图1-17所示。

<center><img src="../assets/1-17.png" width = 550></center>
<center>图1-17 第1层卷积实现</center>

&emsp;&emsp;第二层构建一个更深的网络，每个3×3的Patch会得到32个特征。卷积核大小3×3×16，数量为32个，构造过程类似上一层，如图1-18所示。

<center><img src="../assets/1-18.png" width = 550></center>
<center>图1-18 第2层卷积实现</center>

#### &clubs; 全连接层搭建

&emsp;&emsp;原始图片尺寸是28×28，经过两次2×2的池化后，长宽尺寸降低到7×7。现在加入一个128个神经元的全连接层，将上一层输出的结果Vector化，变成一个向量，将其与权重W_fc1相乘，加上偏置b_fc1，对其使用ReLU，如图1-19所示。

<center><img src="../assets/1-19.png" width = 550></center>
<center>图1-19 全连接层实现</center>

&emsp;&emsp;为了减少过拟合，我们在输出层之前加入dropout。我们用一个placeholder来代表一个神经元的输出在dropout中保持不变的概率。这样我们可以在训练过程中启用dropout，在测试过程中关闭dropout，如图1-20所示。

<center><img src="../assets/1-20.png" width = 450></center>
<center>图1-20 添加dropout</center>

&emsp;&emsp;TensorFlow的tf.nn.dropout操作除了可以屏蔽神经元的输出外，还会自动处理神经元输出值的scale。所以用dropout的时候可以不用考虑scale。

#### &clubs; 输出层搭建

&emsp;&emsp;最后一层使用全连接层，并通过Softmax回归来输出预测结果。

&emsp;&emsp;Softmax模型可以用来给不同的对象分配概率，如图1-21所示。对于输入的x_i加权求和，再分别加上一个偏置量，最后再输入到Softmax函数中。

<center><img src="../assets/1-21.png" width = 400></center>
<center>图1-21 Softmax模型示意图</center>

&emsp;&emsp;Softmax函数的计算公式为：

$$ evidence_i = \sum_{j} W_{i,j} x_j + b_i \tag {1-1} $$

$$ y=softmax(evdience) \tag {1-2} $$

$$ softmax(x)=normalize(exp⁡(x)) \tag {1-3} $$

$$ {softmax(x)}_i = \frac {exp⁡(x_i)} {\sum_{j} exp⁡(x_j)} \tag {1-4} $$

&emsp;&emsp;Softmax层的实现如图1-22所示。

<center><img src="../assets/1-22.png" width = 600></center>
<center>图1-22 Softmax层实现</center>

### 3.2 训练和评估模型

&emsp;&emsp;训练模型前，首先定义损失函数（Loss Function），并在训练过程中尽量最小化这个指标。这里使用的损失函数是"交叉熵"（Cross-Entropy）。交叉熵产生于信息论里面的信息压缩编码技术，但是它后来演变成为从博弈论到机器学习等其他领域里的重要技术手段。它的定义如下：

$$ H_{y'} = - \sum_{i} {y'}_i log(y_i) \tag {1-5} $$

&emsp;&emsp;计算交叉熵后，就可以使用梯度下降来优化参数。由于前面已经部署好网络结构，所以TensorFlow可以使用反向传播算法计算梯度，自动地优化参数，直到交叉熵最小。TensorFlow提供了多种优化器，这里选择更加复杂的Adam优化器来做梯度最速下降，学习率0.0001，如图1-23所示。

<center><img src="../assets/1-23.png" width = 650></center>
<center>图1-23 利用Adam优化器实现梯度最速下降</center>

&emsp;&emsp;每次训练随机选择50个样本，加快训练速度，每轮训练结束后，计算预测准确度，如图1-24所示。

<center><img src="../assets/1-24.png" width = 650></center>
<center>图1-24 随机选取50个样本加速训练</center>

&emsp;&emsp;在CNN网络训练过程中，通过TensorFlow中的可视化工具Tensorboard，可以跟踪网络的整个训练过程中的信息，比如每次循环过程中的参数变化、损失变化等。输入如下命令，即可打开Tensorboard工具：

``` Bash
$> tensorboard –logdir=logs
```

&emsp;&emsp;运行上述命令后，可在浏览器中访问`http://cs-virtual-machine:6006/`。此时，点击Graph选项卡，可查看网络结构等信息，如图1-25所示。

<center><img src="../assets/1-25.png" width = 350></center>
<center>图1-25 查看网络结构</center>

### 3.3 保存并测试网络

&emsp;&emsp;网络训练完成后，需要将网络权值和偏置保存起来。首先基于numpy包实现网络参数的保存函数，如图1-26所示。

``` Python
def Record_Tensor(tensor, name):
    print ("Recording tensor " + name + " ...")
    f = open('./record/' + name + '.dat', 'w')
    array=tensor.eval()
    #print("The range: [" + str(np.min(array)) + ":" + str(np.max(array)) + "]")
    if(np.size(np.shape(array)) == 1):
        Record_Array1D(array, name, f)
    else:
        if (np.size(np.shape(array)) == 2):
            Record_Array2D(array, name, f)
        else:
            if (np.size(np.shape(array)) == 3):
                Record_Array3D(array, name, f)
            else:
                Record_Array4D(array, name, f)
    f.close()
```

<center>图1-26 定义用于保存网络参数的`Record_Tensor`函数</center>

&emsp;&emsp;训练完毕后，使用验证集测试CNN的预测准确度，并通过控制台打印结果。然后调用`Record_Tensor`将各层的参数保存成.dat格式的文件，如图1-27所示。

<center><img src="../assets/1-27.png" width = 400></center>
<center>图1-27 测试网络预测准确度</center>

&emsp;&emsp;由图1-27可知，总迭代次数为3000次，最终的预测准确度是95.13%。

### 3.4 网络的前向推导

&emsp;&emsp;本实验在Ubuntu虚拟机上进行CNN网络训练，并在PYNQ-Z2的异构平台上进行推导测试。网络推导的源文件是实验包的`systolic_app/mnist_cnn.ipynb`。

#### &spades; 网络参数的提取

&emsp;&emsp;训练时，CNN的网络参数以`.dat`格式保存。推导时，首先需要使用虚拟机中的`~/tensorflow/dat2bin`工具，将其转换为`.bin`格式。

&emsp;&emsp;`mnist_cnn.ipynb`中定义了`readbinfile`函数以读取`.bin`格式的网络权值和偏置文件，如图1-28所示。

``` Python
def readbinfile(filename,size):
    f = open(filename, "rb")
    z=[]
    for j in range(size):
        data = f.read(4)
        data_float = struct.unpack("f", data)[0]
        z.append(data_float)
    f.close()
    z = np.array(z)
    return z
```

<center>图1-28 读取.bin文件</center>

&emsp;&emsp;调用`readbinfile`函数读取CNN的网络权值和偏置，如图1-29所示。

``` Python
# Read weights and bias from pre-tranined file
print("Conv1:\tloading weight... ", end = "")

w_conv1 = readbinfile("./data/W_conv1.bin", KERNEL_W1*KERNEL_W1*IN_CH1*OUT_CH1)
w_conv1 = w_conv1.reshape((KERNEL_W1, KERNEL_W1, IN_CH1, OUT_CH1))
for r in range(KERNEL_W1):
    for c in range(KERNEL_W1):
        for ch_i in range(IN_CH1):
            for ch_o in range(OUT_CH1):
                W_conv1[ch_o][ch_i][r][c] = w_conv1[r][c][ch_i][ch_o]

print("done")
print("\tloading bias... ", end = "")
                
B_conv1 = readbinfile("./data/b_conv1.bin",OUT_CH1)
for i in range(OUT_CH1):
    b_conv1[i] = B_conv1[i]

print("done")

......
```

<center>图1-29 读取网络的权值和偏置参数</center>

#### &spades; IP核的驱动/调用

&emsp;&emsp;读取网络参数后，需要根据CNN的网络结构，在主程序中调用脉动阵列IP核与池化IP核，以完成CNN的前向推导过程。

&emsp;&emsp;`mnist_cnn.ipynb`还定义了脉动阵列IP核的驱动函数。该函数将IP核所需的各个参数通过特定的接口，经由AXI总线写入IP核当中，并通过特定接口读取IP核输出的数据，如图1-30所示。

``` Python
# 脉动阵列驱动函数
def RunSystolic(array, din_a, din_b, bias, out):
    array.write(0x10, din_a.shape[0])
    array.write(0x18, din_a.shape[1])
    array.write(0x20, din_b.shape[1])
    array.write(0x28, din_a.physical_address)
    array.write(0x30, din_b.physical_address)
    array.write(0x38, bias.physical_address)
    array.write(0x40, out.physical_address)
    array.write(0, (array.read(0) & 0x80) | 0x01)
    tp = array.read(0)
    while not ((tp >> 1) & 0x1):
        tp = array.read(0)
```

<center>图1-30 脉动阵列IP核的驱动函数</center>

&emsp;&emsp;池化IP核的驱动函数与`RunSystolic`函数类似，此处不再赘述。

!!! info "补充说明 :mega:"
    &emsp;&emsp;脉动阵列IP核默认仅支持GEMM运算，所以如果想使用该IP核计算卷积，还需要另外实现卷积函数`hwConv`。这个函数负责完成特征图和卷积核的im2col操作，然后调用脉动阵列IP核，最后调整运算结果的尺寸和形状。
