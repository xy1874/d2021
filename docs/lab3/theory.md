# 实验原理

&emsp;&emsp;循环神经网络（Recurrent Neural Network, RNN）是一类专门用于处理序列型数据（如文本、语音等）的神经网络。传统的RNN因为梯度弥散等问题对长序列的预测结果不甚理想，各方学者对此展开了研究和优化。其中，最具代表性的就是长短期记忆（Long Short-Term Memory, LSTM）神经网络。接下来将从传统RNN到LSTM，介绍循环神经网络的基本原理和网络结构。

## 1. RNN简介

### 1.1 RNN原理简介

&emsp;&emsp;假设有一个多层感知机（Multi-Layer Perceptron, MLP），现在将一个时间上连续的输入序列依次添加到MLP的每一个隐藏层，得到图1-1(a)所示的网络。此时，如果令所有隐藏层的权值和参数都相同，如图1-1(b)所示，则可以将所有隐藏层在时间上“折叠”起来，形成循环层，于是得到如图1-1\(c)所示的循环神经网络。

<center><img src="../assets/1-1a.png" width = 250> <img src="../assets/1-1b.png" width = 250> <img src="../assets/1-1c.png" width = 130></center>
<center>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;(a) MLP &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; (b) 隐藏层的参数都相同 &emsp;&emsp; \(c) 形成循环层</center>
<center>图1-1 从MLP到RNN</center>

&emsp;&emsp;图1-1\(c)中的循环层也被称作是RNN的循环神经元或Cell。一般地，可用图1-2所示的结构来简化表示RNN的网络结构。

<center><img src="../assets/1-2.jpg" width = 180></center>
<center>图1-2 RNN网络结构</center>

&emsp;&emsp;图1-2所示的RNN神经网络，其Cell所产生的输出随着输入的变化而变化。若输入随时间变化而变化，则Cell的输出也随着时间发生相应的变化。假设Cell在t时刻接收的输入是$\pmb x_t$，产生的输出是$\pmb h_t$，则可得式(1-1)所示的简单模型：

$$ \pmb h_t = f(\pmb h_{t-1}, \pmb x_t) \tag {1-1} $$

&emsp;&emsp;其中，$\pmb h_{t-1}$是Cell在上一个时刻的输出。

&emsp;&emsp;进一步地，假设输入$\pmb h_{t-1}$的权值是$\pmb W_{hh}$、输入$\pmb x_t$的权值是$\pmb W_{xh}$，偏置量是$\pmb b$，激活函数记为activation()，则可将式(1-1)改写为式(1-2)：

$$ \pmb h_t = activation(\pmb W_{hh}∙\pmb h_{t-1} + \pmb W_{xh} ∙ \pmb x_t + \pmb b) \tag {1-2} $$

&emsp;&emsp;输入结束后，RNN网络也随之计算得出最终的输出结果。假设Cell的输出$\pmb h_t$的权值是$\pmb W_{hy}$，则RNN的输出$\pmb y_t$如式(1-3)所示。

$$ \pmb y_t = \pmb W_{hy} ∙ \pmb h_t \tag {1-3} $$

&emsp;&emsp;总结RNN的计算步骤如下：

&emsp;&emsp;Step1: 将待预测数据$\pmb x_t$输入到RNN网络当中；

&emsp;&emsp;Step2: 基于式(1-2)，利用输入$\pmb x_t$和Cell在前一时刻的输出$\pmb h_{t-1}$，计算$\pmb h_t$；

&emsp;&emsp;Step3: 若数据输入完毕，基于式(1-3)计算网络输出；否则，t=t+1，转Step1。

### 1.2 RNN的反向传播

&emsp;&emsp;为了方便理解RNN的通过时间的反向传播（Back Propagation Through Time, BPTT），首先将RNN展开，如图1-3所示。

<center><img src="../assets/1-3.jpg" width = 550></center>
<center>图1-3 按时间对RNN进行展开</center>

&emsp;&emsp;将RNN展开后，图1-3右侧的每一个圆圈都代表了左侧的RNN Cell在不同时间节点的状态。

&emsp;&emsp;前向传播时，RNN按照输入序列的内部顺序，在每个时间节点处读取并处理序列中的一个数据。反向传播时，需要反过来对各个时间节点上的网络权值进行更新，故而将RNN的反向传播称为通过时间的反向传播。

&emsp;&emsp;假设$\pmb y_t$是RNN网络在t时刻输出的预测值，而$\pmb y ̂_t$是实际真实值，则可用交叉熵来计算误差，如式(1-4)所示。

$$ \pmb E_t (\pmb y ̂_t, \pmb y_t ) = -\pmb y ̂_t log⁡(\pmb y_t) \tag {1-4} $$

&emsp;&emsp;将所有时间节点的误差进行累加，即可得到总体误差，如式(1-5)所示。

$$ \pmb E(\pmb y ̂, \pmb y) = - \sum_{t} \pmb y ̂_t log⁡(\pmb y_t) \tag {1-5} $$

&emsp;&emsp;基于式(1-4)和式(1-5)，可将RNN的BPTT过程简述为：

&emsp;&emsp;Step1: 基于式(1-5)所示的交叉熵公式计算误差；

&emsp;&emsp;Step2: 将RNN按照时间节点完全展开；

&emsp;&emsp;Step3: 计算展开后每个时间节点上Cell的网络权值梯度；

&emsp;&emsp;Step4: 使用Step3计算得到的梯度更新Cell的权值。

### 1.3 梯度消失与梯度爆炸问题

&emsp;&emsp;RNN的原理使得其预测结果依赖于RNN Cell在所有时间节点上的状态。时间节点数越多，网络权值的更新越难。现在，考虑一个具有三个时间节点的RNN Cell，其网络权值梯度的计算需要使用链式求导，如式(1-6)所示。

$$ \frac {∂\pmb E}{∂\pmb W} = \frac {∂\pmb E}{∂\pmb y_3} ∙ \frac {∂\pmb y_3}{∂\pmb h_3} ∙ \frac {∂\pmb h_3}{∂\pmb y_2} ∙ \frac {∂\pmb y_2}{∂\pmb h_1} \tag {1-6} $$

&emsp;&emsp;如果式(1-6)中存在某个接近0的乘积项，则计算出的所有梯度都会接近于0，而其他时间节点的梯度更是以指数速率降低为0，于是便出现了梯度消失问题，如图1-4所示。显然，梯度消失问题不利于RNN网络对较长输入序列的学习。

<center><img src="../assets/1-4.png" width = 650></center>
<center>图1-4 梯度消失示意图</center>

&emsp;&emsp;类似地，梯度爆炸问题是梯度值因为时间节点增多而容易变得非常大的问题。一般地，可以通过设定阈值的方法解决梯度爆炸问题，因此对RNN的训练而言，如何解决梯度消失问题显得更为关键。

&emsp;&emsp;为了解决梯度消失问题，Sepp Hochreiter和Ju ̈rgen Schmidhuber在1997年提出了[LSTM](https://www.mitpressjournals.org/doi/abs/10.1162/neco.1997.9.8.1735)，为人工智能领域作出了教科书级的贡献。

## 2. LSTM原理简述

&emsp;&emsp;在图1-2所示的RNN网络结构中，Cell只能存储当前时刻的输出$\pmb h_t$，因而对短期的输入比较敏感。为了增加RNN对长期输入（或长序列输入）的预测效果，现在给Cell增加一个变量$\pmb C_t$，用于存储Cell的当前状态，并由此形成LSTM Cell，如图1-5所示。其中，$\pmb C_t$被称为单元状态（Cell State）。

<center><img src="../assets/1-5.png" width = 200></center>
<center>图1-5 RNN Cell的改进</center>

&emsp;&emsp;同样地，可将图1-5所示的LSTM Cell按时间展开，得到如图1-6所示的结构。

<center><img src="../assets/1-6.png" width = 450></center>
<center>图1-6 展开的LSTM Cell</center>

&emsp;&emsp;观察图1-6中时刻t所对应的LSTM Cell，可知在时刻t，Cell接受三个输入，分别是当前时刻的外部输入$\pmb x_t$、前一时刻的输出$\pmb h_{t-1}$与单元状态$\pmb C_{t-1}$；产生两个输出，分别是当前时刻的对外输出$\pmb h_t$与单元状态$\pmb C_t$。

&emsp;&emsp;传统的RNN Cell只能存储当前时刻的输出$\pmb h_t$。LSTM Cell在RNN Cell的基础上增加了遗忘门、输入门、输出门等子结构，如图1-7所示。
 
<center><img src="../assets/1-7.png" width = 550></center>
<center>(a) 传统RNN Cell的结构 &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; (b) LSTM Cell的结构 &emsp;&emsp;&emsp;&emsp;&emsp;</center>
<center>图1-7 RNN结构的演化</center>

&emsp;&emsp;接下来，逐一介绍LSTM Cell的遗忘门、输入门、输出门以及状态产生逻辑和输出逻辑，如图1-8所示。
 
<center><img src="../assets/1-8.png" width = 400></center>
<center>图1-8 LSTM Cell结构组成</center>

### &clubs; 遗忘门（Forget Gate）

&emsp;&emsp;遗忘门的输入是$\pmb h_{t-1}$与$\pmb x_t$拼接后得到的向量$[\pmb h_{t-1}, \pmb x_t]$，输出$\pmb f_t$被称为遗忘门的输出控制向量，用于控制Cell对上一个单元状态$\pmb C_{t-1}$的遗忘程度。

&emsp;&emsp;根据图1-8中的遗忘门结构，可得到遗忘门输出向量$\pmb f_t$的计算公式如式(1-7)所示。

$$ \pmb f_t = sigmoid(\pmb W_f∙[\pmb h_{t-1}, \pmb x_t] + \pmb b_f) \tag {1-7} $$

&emsp;&emsp;其中，$\pmb W_f$和$\pmb b_f$分别是遗忘门的权值矩阵和偏置向量。

### &clubs; 输入门（Input Gate）

&emsp;&emsp;输入门的输入是$\pmb h_{t-1}$与$\pmb x_t$拼接后得到的向量$[\pmb h_{t-1}, \pmb x_t]$，输出$\pmb i_t$被称为输入门的输出控制向量，用于控制Cell的新状态的产生。

&emsp;&emsp;根据图1-8中的输入门结构，可得到输入门输出向量$\pmb i_t$的计算公式如式(1-8)所示。

$$ \pmb i_t = sigmoid(\pmb W_i∙[\pmb h_{t-1}, \pmb x_t] + \pmb b_i) \tag {1-8} $$

&emsp;&emsp;其中，$\pmb W_i$和$\pmb b_i$分别是输入门的权值矩阵和偏置向量。

&emsp;&emsp;此外，输入门根据向量$[\pmb h_{t-1}, \pmb x_t]$产生候选状态向量$\pmb C ̃_t$，用于给Cell产生新的单元状态。

&emsp;&emsp;根据图1-8中的输入门结构，可得到输入门产生的候选状态向量$\pmb C ̃_t$的计算公式如式(1-9)所示。

$$ \pmb C ̃_t = tanh(\pmb W_C ∙ [\pmb h_{t-1}, \pmb x_t] + \pmb b_c) \tag {1-9} $$

&emsp;&emsp;其中，$\pmb W_C$和$\pmb b_c$分别是候选状态向量的权值矩阵和偏置向量。

### &clubs; 输出门（Output Gate）

&emsp;&emsp;输出门的输入是$\pmb h_{t-1}$与$\pmb x_t$拼接后得到的向量$[\pmb h_{t-1}, \pmb x_t]$，输出$\pmb o_t$被称为输出门的输出控制向量，用于控制Cell输出向量$\pmb h_t$的产生。

&emsp;&emsp;根据图1-8中的输出门结构，可得到输出门输出向量$\pmb o_t$的计算公式如式(1-10)所示。

$$ \pmb o_t = sigmoid(\pmb W_o ∙ [\pmb h_{t-1}, \pmb x_t] + \pmb b_o) \tag {1-10} $$

&emsp;&emsp;其中，$\pmb W_o$和$\pmb b_o$分别是输入门的权值矩阵和偏置向量。

### &clubs; 状态产生逻辑

&emsp;&emsp;状态产生逻辑根据Cell的上一个单元状态$\pmb C_{t-1}$、遗忘门产生的控制向量$\pmb f_t$、输入门产生的控制向量$\pmb i_t$以及候选状态向量$\pmb C ̃_t$，产生新的单元状态$\pmb C_t$。

&emsp;&emsp;根据图1-8中的状态产生逻辑，可得到新的单元状态$\pmb C_t$的计算公式如式(1-11)所示。

$$ \pmb C_t = \pmb C_{t-1} * \pmb f_t + \pmb i_t * \pmb C ̃_t \tag {1-11} $$

&emsp;&emsp;其中，$*$表示向量的 *Hadamard* 积。

!!! info "补充说明 :mega:"
    向量 *Hadamard* 积的定义：  
    &emsp;&emsp;设有维度相同的向量$\pmb a = \{a_1,a_2,⋯,a_n\}$和向量$\pmb b = \{b_1,b_2,⋯,b_n\}$，则有：  
    <center>$\pmb a * \pmb b = \{a_1 b_1, a_2 b_2, ⋯, a_n b_n\}$</center>

### &clubs; 输出逻辑

&emsp;&emsp;输出逻辑根据Cell的新状态$\pmb C_t$和输出门的控制向量$\pmb o_t$，产生LSTM Cell的对外输出向量$\pmb h_t$。

&emsp;&emsp;根据图1-8所示的输出逻辑，可得到Cell的输出向量$\pmb h_t$的计算公式如式(1-12)所示。

$$ \pmb h_t = \pmb o_t * tanh⁡(\pmb C_t) \tag {1-12} $$

&emsp;&emsp;观察式(1-7)至式(1-12)，可以发现向量$\pmb f_t$、$\pmb i_t$、$\pmb C ̃_t$、$\pmb o_t$、$\pmb C_t$和$\pmb h_t$都具有相同的维度，从而权值矩阵$\pmb W_f$、$\pmb W_i$、$\pmb W_C$和$\pmb W_o$维度相同，偏置向量$\pmb b_f$、$\pmb b_i$、$\pmb b_c$和$\pmb b_o$也维度相同。

&emsp;&emsp;为方便描述，将n维向量$\pmb a$的维度记为dim($\pmb a$)，即dim($\pmb a$) = n。

&emsp;&emsp;一般地，将$\pmb h_t$的维度称为hidden_size或num_units，表示LSTM Cell所等效的隐藏层节点数目，即dim($\pmb h_t$) = hidden_size或dim($\pmb h_t$) = num_units。由此可将各个权值矩阵的维度表示为hidden_size×(dim($\pmb x_t$) + hidden_size)，而各偏置向量的维度就是hidden_size。

## 3. 基于LSTM的手写数字识别

### 3.1 MNIST手写数字数据集

&emsp;&emsp;MNIST数据集是28×28像素的灰度手写数字图片，其中数字的范围从0到9，具体如表1-1所示（参考自[http://yann.lecun.com/exdb/mnist/](http://yann.lecun.com/exdb/mnist/)）。

<center>表1-1 MNIST数据集描述</center>
<center>

| 文  件 | 内  容 |
| :--: | :---: |
| train-images-idx3-ubyte.gz | 训练集图片（训练集55000张，验证集5000张） |
| train-labels-idx1-ubyte.gz | 训练集图片所对应的数字标签 |
| t10k-images-idx3-ubyte.gz | 测试集图片（10000张） |
| t10k-labels-idx1-ubyte.gz | 测试集图片所对应的数字标签 |

</center>

&emsp;&emsp;每一张图片都有对应的标签，也就是图片所对应的数字。例如，图1-9所示的图片显示的是数字1，即其标签为1。

<center><img src="../assets/1-9.png" width = 400></center>
<center>图1-9 MNIST数据集图片范例</center>

&emsp;&emsp;MNIST数据集包含训练集和测试集2部分。其中，训练集共60000张手写数字图片（55000张作为训练集，剩余5000张作为验证集）；而测试集则包含10000张手写数字图片。训练集是一个维度为[60000, 784]的张量——第1个维度表示训练集图片的数量，第2个维度表示每张图片的像素点个数。此张量里的每一个元素，都表示某张图片里的某个像素的强度值，值介于0和1之间。

&emsp;&emsp;每一张训练集图片都有一个与之对应的0到9之间的数字标签，用来表示训练集图片所显示的数字。此处使用独热向量（One-hot Vector）来记录训练集图片的标签。独热向量由若干个独热编码（One-hot Code）组成。每一个独热编码的位宽均为10（因为0-9共有10个数字），且独热编码中只有一位是1，其余位均是0。例如，图1-9所示的图片，其标签为1，且对应的独热编码是[0,1,0,0,0,0,0,0,0,0]。由此，可将训练集所对应的标签集看作是一个[60000, 10]的二进制矩阵，如图1-10所示。

<center><img src="../assets/1-10.png" width = 350></center>
<center>图1-10 MNIST数据集与标签集</center>

### 3.2 LSTM识别手写数字

&emsp;&emsp;MNIST数据集的每一张手写数字图片都是一个28×28的二维矩阵。为此，可采用包含输入层、LSTM Cell和输出层的RNN网络，且输入层包含28个神经元，隐藏层的LSTM Cell包含128个神经元（num_units为128，即dim($\pmb h_t$) = 128），而输出层则包含10个神经元，代表手写数字图片所具有的10个分类结果。该RNN的网络结构如图1-11所示。

<center><img src="../assets/1-11.png" width = 600></center>
<center>图1-11 本实验的RNN网络结构</center>

&emsp;&emsp;接下来，讨论如何使用图1-11所示的RNN网络对手写数字图片进行预测。

&emsp;&emsp;不妨将输入的手写数字图片的每一行看作是一个行向量，则每一张手写数字图片都可以表示成一个包含28个行向量的列向量，如图1-12所示。

<center><img src="../assets/1-12.png" width = 500></center>
<center>图1-12 MNIST手写数字图片示意图</center>

&emsp;&emsp;由此，可以得到一个长度为28的输入序列。利用RNN进行前向推导时，每次向网络中输入一个行向量$\pmb x_i(0≤i<28)$，此时RNN网络会产生相应的输出$\pmb h_i$；当行向量$\pmb x_{27}$被输入RNN进行前向推导时，RNN产生的输出$\pmb h_{27}$就是LSTM Cell的最终输出向量，该向量包含了所输入图片的预测分类信息，如图1-13所示。

<center><img src="../assets/1-13.png" width = 400></center>
<center>图1-13 RNN识别手写数字图片的示意图</center>

&emsp;&emsp;接下来，使用全连接层作为的线性分类器对LSTM Cell输出的$\pmb h_{27}$向量进行分类，其计算公式如式(1-13)所示。

$$ \pmb y_{27} = \pmb W_y ∙ \pmb h_{27} + \pmb b_y \tag {1-13} $$

&emsp;&emsp;最后，找到分类向量$\pmb y_{27}$的最大分量所对应的下标，即为输入图片的预测值。

## 4. RNN加速器架构

&emsp;&emsp;本实验设计的RNN加速器架构如图1-14所示。

<center><img src="../assets/1-14.png" width = 500></center>
<center>图1-14 RNN加速器架构图</center>

&emsp;&emsp;在图1-14中，RNN加速器主要由PS端的ARM CPU、PL端的RNN加速IP核、PL端的DMA控制器和DDR控制器组成。PS端的ARM CPU用于对加速系统进行必要的初始化，比如在运行时加载Overlay、配置DMA控制器等；此外还负责对RNN网络推导所需的数据进行预处理，比如对MNIST数据集进行解析等。DDR控制器用于解析PS和PL对板上DDR存储器的访问请求，并向DDR存储芯片发出相应的读写信号。DMA控制器用于实现PS与PL之间的快速数据传输，以提高加速器的整体性能。RNN硬件IP核用于实现RNN网络前向推导的硬件加速，是整个硬件加速系统的核心运算部件。

&emsp;&emsp;增设了DMA控制器后，ARM CPU可通过DMA方式与RNN IP核进行快速的数据交互。在总线频率为150MHz、总线数据位宽为32bit的前提下，PS与PL之间的实测通信带宽可以达到550MB/s以上（理论最大带宽为600MB/s）。

&emsp;&emsp;需要注意的是，单次DMA传输的数据批量越大，能够获得的带宽越大，数据传输速率越高。当数据批量达到阈值后，带宽不再增大，进入饱和状态。
