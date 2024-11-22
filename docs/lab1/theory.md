# 实验原理

## 1. 概述

### 1.1 背景简介

&emsp;&emsp;近年来，随着人工智能、大数据等技术的不断发展，应用需求也逐渐变得多种多样。在技术发展趋势和各种应用需求的驱动下，人们对移动端设备或嵌入式设备的智能化期望越来越高。在云计算的背景下，移动端设备和嵌入式设备通常合称为端设备，与“云”的概念相对应。端设备数量庞大，产生的数据极多。因此，若想单纯依靠传统云计算来实现端设备的智能化，则云与端之间的网络带宽必定成为系统的性能瓶颈。为了解决这个问题，需要 **将云端的一部分计算任务下放到端设备，以减轻云端和网络带宽的压力**。然而端设备大多采用嵌入式处理器 —— 嵌入式处理器受到功耗、体积、散热等多方客观因素的限制，其性能远不如桌面平台。为了提高端设备的性能，同时保证其满足原有的功耗、体积、散热等需求，可以利用FPGA、ASIC等 **低功耗**、**高能效** 的器件，为相应的应用场景定制该领域所专用的加速器。

### 1.2 神经网络简介

&emsp;&emsp;神经网络一般包含 **输入层**、**中间层** 和 **输出层**。输入层获取神经网络的外部输入数据，其结构取决于输入数据的维度。中间层又称隐藏层，负责对输入数据进行特征提取和分析，从而产生相应的特征图。输出层接收中间层的特征图作为输入，并输出神经网络的最终计算结果，其结构根据应用场景而有所不同。

&emsp;&emsp;使用神经网络解决实际问题时，首先要对应用场景进行需求分析，确定需要使用哪一种类型的神经网络。然后，结合实际应用的特点，搭建神经网络的初步结构。接下来，需要通过具备一定规模的数据集对所搭建的网络进行训练，得到网络中各个层各个节点的参数。必要时需要根据训练后的测试结果对神经网络的结构进行合理的调整以完成神经网络的迭代和优化。当神经网络的性能达到预期后，可将其部署到实际应用场景中，进行前向推导。**神经网络的前向推导指的是网络接收外部数据，并在经过网络内部的运算后，产生特定结果的过程。**

### 1.3 加速器设计方法概述

&emsp;&emsp;神经网络硬件加速器的设计通常遵循如图1-1所示的基本流程。

<center><img src="../assets/1-1.png" width = 400></center>
<center>图1-1 AI加速器设计流程</center>

&emsp;&emsp;一般地，硬件加速器的设计可采用完全硬件化和部分硬件化2种设计方法。

&emsp;&emsp;**完全硬件化** 的设计方法需要根据网络结构，使用HDL（Hardware Description Language）或HLS（High-Level Synthesis）实现神经网络的每一个层。该方法的优点是硬件化程度高，可实现网络层次的流水线，并且能够获得很高的加速效果，但显然存在开发难度大、开发周期长、硬件资源消耗较多以及通用性差的缺点。

&emsp;&emsp;**部分硬件化** 的设计方法首先从神经网络的运算类型出发进行考虑，通过分析和必要的测试，得出神经网络内部各类运算的占比，然后有针对性地选择占比较大的运算进行加速；对于占比较小的运算，则仍然使用软件实现。部分硬件化设计方法的理论依据是[Amdahl定律](https://www.inlighting.org/archives/amdahls-law-and-its-proof/)。该方法具有开发难度较低、开发周期较短、硬件资源消耗较少的优点，同时还能够使加速器具有一定的通用性和灵活性，并且在某些情形下能够获得与完全硬件化方法相近的性能。

&emsp;&emsp;本课程实验对上述2种设计方法均有所涉及。**在实际应用场合中，同学们需要根据具体需求，自行选择合适的设计方法**。


## 2. PYNQ简述

&emsp;&emsp;本课程实验将采用PYNQ-Z2实验平台。[PYNQ](http://www.pynq.io/)是Xilinx公司基于ZYNQ平台推出的开源项目，旨在降低开发者（特别是软件开发者）使用异构平台的门槛。所谓异构，是指同一系统中存在功能相似而架构不同的组成部分，如现今流行的大小核处理器即可视为异构平台。开发者使用PYNQ，可以较为方便地实现 **硬件加速算法**、**并行计算**、**高帧率视频处理**、**实时信号处理**、**低时延控制** 等高性能应用。

### 2.1 ZYNQ简介

&emsp;&emsp;ZYNQ是Xilinx推出的全可编程片上系统（All-Programmable System-On-Chip，APSoC）。所谓片上系统/SoC指的是在一块芯片上集成一个相对完整的系统，通常包含 **时钟**、**处理器**、**总线**、**存储**、**I/O** 等等。

&emsp;&emsp;ZYNQ都是异构SoC，通常至少包含ARM CPU和FPGA资源，高端ZYNQ甚至还包含GPU，如Mali-400系列的嵌入式GPU。所谓异构SoC，是指SoC内部包含多个处理核，且这些处理核的架构互不相同。ZYNQ的典型架构可简化为如图1-2所示。

<center><img src="../assets/1-2.png" width = 350></center>
<center>图1-2 ZYNQ架构简图</center>

&emsp;&emsp;由图1-2可知，ZYNQ主要包含 **PS**（Processing System, 处理系统）和 **PL**（Programmable Logic, 可编程逻辑）两大部分。PS通常使用ARM Cortex系列的嵌入式CPU，一般采用同构多核CPU，高端芯片则可能采用异构大小核处理器。PL则一般指FPGA的可编程逻辑资源。此外，PS和PL通过 **AXI总线** 进行数据和控制信号的交互。

### 2.2 PYNQ架构浅析

&emsp;&emsp;PYNQ官网给出了如图1-3所示的PYNQ软硬件架构图。

<center><img src="../assets/1-3.png"></center>
<center>图1-3 PYNQ架构图</center>

&emsp;&emsp;与一般的计算机系统类似，PYNQ的系统架构可粗分为3个层次：硬件层、系统软件层和应用层。硬件层即是ZYNQ，主要包括PS和PL两部分。系统软件层包括FSBL（First Stage BootLoader）、U-Boot、嵌入式Linux、设备驱动程序等等。应用层则包括Python开发环境、Jupyter Notebook等应用软件。

!!! note "划重点 :point_down:"
    &emsp;&emsp;本课程实验的Xilinx PYNQ-Z2开发板，其PS采用的是双核的ARM Cortex-A9处理器，主芯片型号为<font color = orange>**xc7z020clg400-1**</font>。

&emsp;&emsp;可将图1-3所示的架构简化成如图1-4所示的架构。

<center><img src="../assets/1-4.png" width = 260></center>
<center>图1-4 PYNQ架构简图</center>

&emsp;&emsp;应用层想要读写硬件数据时，需要调用特定的API。API将应用层的读写请求和参数传递给驱动层。驱动层解析应用层的读写请求，并通过特定的接口读写硬件IP核的数据。驱动层的读写程序最终被编译器和汇编器转换成了指令序列在CPU中执行。CPU通过指令的译码和执行，向特定地址的接口发出数据读/写请求，此时，应用层的读写请求已转换成硬件层面的读写信号。读写信号通过AXI总线传递到FPGA上的IP核。IP核根据总线读/写逻辑，读取硬件的数据或将收到的数据写入到特定的存储器中。

&emsp;&emsp;更多PYNQ资料，请查看官方文档：[https://pynq.readthedocs.io/en/v2.5/index.html](https://pynq.readthedocs.io/en/v2.5/index.html)。


## 3. 高层次综合（HLS）

&emsp;&emsp;传统FPGA开发使用HDL（Hardware Description Language）进行，如Verilog、VHDL。HDL可将代码编译成片上设计，这种设计方法在上世纪80年代开始得到广泛使用。随着硬件电路复杂度的指数性增长，硬件工程师们不断寻求更便捷高效的硬件开发语言和工具。由此，HLS（High-Level Synthesis）应运而生。

&emsp;&emsp;使用HLS设计电路时，更多地需要考虑电路的整体架构而非某个单独的部件。最早大多数HLS工具是基于Verilog的，用户需要使用Verilog描述电路，工具也通过Verilog产生RTL。现今很多HLS工具开始支持使用C/C++等高级语言来完成硬件电路设计。

&emsp;&emsp;HLS可自动完成以下曾经需要手动完成的工作：

<font color = dimgray>

- 自动分析并利用一个算法中潜在的并行性

- 自动在需要的路径上插入寄存器，并自动选择最理想的时钟

- 自动产生控制数据在一个路径上输入/输出方向的逻辑

- 自动完成设计中各子模块之间的接口

- 自动映射数据到存储单元以平衡资源使用量和带宽

- 自动将程序中的计算部分对应到逻辑单位，在实现等效计算时自动选取最有效的实现方式

</font>

### 3.1 HLS与电路的对应关系

&emsp;&emsp;一般地，软件程序中的函数最终会综合成为相应的电路模块实体，而程序中的控制流和数据流则由HLS工具中的调度和绑定程序（Scheduling and Binding Processes）映射到硬件电路当中。具体的软件成分与硬件组成之间的对应关系如表1-1所示。

<center>表1-1 软硬件映射关系（以C语言为例）</center>
<center>

| 软件成分 | 对应的硬件组成 |
| :-: | :-: |
| 函数 | 模块 |
| 函数的参数 | 模块的输入/输出端口 |
| 操作符 | 功能单元 |
| 变量 | 线网（wire）或寄存器（reg） |
| 数组 | 存储器 |
| 控制流 | 控制逻辑 |

</center>

!!! example "举个栗子 :chestnut:"
    &emsp;&emsp;程序中的函数通常会被综合成单独的电路模块，如图1-5所示。

    <center><img src="../assets/1-5.png" width = 400></center>
    <center>图1-5 函数将被综合成module</center>

    &emsp;&emsp;需要注意的是，在图1-5中函数B被调用了2次。如果在运行时，这2处调用不会同时执行，那么HLS工具将为函数B生成一个 **运行时可重用** 的电路模块实例；否则，将生成2个电路模块实例。

    &emsp;&emsp;类似地，函数的参数将被综合成模块的输入/输出端口，如图1-6所示。

    <center><img src="../assets/1-6.png" width = 520></center>
    <center>图1-6 函数参数将被综合成module的I/O Ports</center>

    &emsp;&emsp;由图1-6可知，除了参数本身，HLS工具还将额外生成必要的辅助信号。

### 3.2 HLS的设计规范

&emsp;&emsp;虽然HLS可将高级语言描述的程序转换成硬件电路，但HLS并没有强大到可以处理任何代码。许多在软件编程中常用的概念在硬件中很难实现，所以有时需要将HLS与HDL结合，从而使得设计更加灵活。

&emsp;&emsp;HLS工具通常需要用户提供附加信息（通过suggestion或#pragma）来帮助完善程序，因此我们说HLS工具会同时"限制"又"加强"了一门语言。例如，HLS一般无法进行动态内存分配，且大部分HLS工具对标准库的支持也非常有限；此外，使用HLS编程时，应当避免使用系统调用和递归语句，以尽量降低程序的复杂程度。除去这些设计限制，HLS的处理范围非常广（包括DMA，数据流，Scratchpad Memory等），优化效率也较高。

&emsp;&emsp;一般地，<font color = lightseagreen>**使用HLS设计开发硬件电路时，应遵循的规范如下**</font>：

- 不使用动态内存分配，如malloc()、free()、new和delete等

- 不使用系统调用，如abort()、exit()、printf()等（可在测试代码中使用系统调用，但在需要综合的代码中，系统调用将被自动忽略或删除）

- 不使用递归语句

- 减少使用指针对指针的操作

- 减少使用标准库函数（HLS支持math.h中的常用函数，但仍存在不兼容）

- 减少使用C++中的函数指针和虚函数

### 3.3 HLS设计IP核的流程

&emsp;&emsp;使用HLS设计硬件IP核的流程如图1-50所示。

<center><img src="../assets/1-50.png" width = 500></center>
<center>图1-50 HLS设计流程</center>

&emsp;&emsp;由图1-50可知，HLS设计硬件IP核的步骤为：

&emsp;&emsp;1）使用C/C++、System C等高级语言实现目标算法，并编写Test Bench以验证和调试代码；

&emsp;&emsp;2）通过C Synthesis工具，将算法综合成VHDL或Verilog描述的硬件电路。此时，可进行C和  
&emsp;&emsp;RTL的联合仿真调试；

&emsp;&emsp;3）将通过功能验证的RTL打包成硬件IP核；

&emsp;&emsp;4）在Vivado中导入IP核、构建Block Design、生成比特流并进行下板验证。

&emsp;&emsp;加州大学圣地亚哥分校（UCSD）的CSE 237C课程里有很多基于HLS开发的项目，比如cordic、DFT、FFT、FIR、histogram、huffman、matrixm、sort、spmv、sum和vs等经典的算法。感兴趣的同学可以自行[从Github下载源码](https://github.com/xupsh/pp4fpgas-cn-hls)学习。

&emsp;&emsp;更多HLS资料，请参考[《Parallel Programming for FPGAs》](https://xupsh.gitbook.io/pp4fpgas-cn/)。
