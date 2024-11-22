# 附录2：Jupyter Notebook简介

&emsp;&emsp;Jupyter Notebook（一下简称Jupyter）是一个Web应用程序，便于创建和共享文学化程序文档，支持40多种编程语言和数学方程、可视化和Markdown等功能。用途包括数学清理和转换、数值模拟、统计建模、机器学习等等。

&emsp;&emsp;Jupyter支持IPython。IPython是一个Python的交互式Shell，支持变量自动补全、自动缩进、Bash Shell命令等，内置了许多有用的功能和函数。

&emsp;&emsp;Jupyter与IPython终端共享同一个内核。内核进程可以同时连接多个前端。此时，不同的前端访问的是同一个变量。这种设计可满足2种需求：相同内核不同前端，用以支持快速开发新的前端；相同前端不同内核，用以支持新的开发语言。

## 1. 基本界面

&emsp;&emsp;Jupyter的基本界面如图4-1所示。

<center><img src="../assets/4-1.png"></center>
<center>图4-1 Jupyter基本界面</center>

&emsp;&emsp;点击菜单栏中的Running标签，可查看正在运行的Notebook，也可以Shutdown某个运行中的Notebook，如图4-2所示。

<center><img src="../assets/4-2.png"></center>
<center>图4-2 Running界面</center>

## 2. 编辑界面

&emsp;&emsp;新建Notebook后，进入编辑界面。编辑界面由名称、菜单栏、工具条和cell组成，如图4-3所示。

<center><img src="../assets/4-3.png" width = 600></center>
<center>图4-3 Jupyter编辑界面</center>

&emsp;&emsp;菜单栏与一般的IDE类似，有“File”、“Edit”、“View”等功能，此处不作详细介绍。感兴趣的同学可以自行点击菜单栏，查看其具体功能。

&emsp;&emsp;菜单栏下方的是工具条。工具条的功能已包含在菜单栏中，但使用工具条可以更快速便捷地操作。工具条如图4-4所示。

<center><img src="../assets/4-4.png"></center>
<center>图4-4 工具条简介</center>

&emsp;&emsp;工具条下方的是单元格区域，也叫cell区域。cell有编辑模式（Edit Mode）和命令模式（Command Mode）2种模式。在不同的模式下可以进行不同的操作。

&emsp;&emsp;编辑模式下，右上角出现铅笔图标，cell左侧框线呈绿色，如图4-5所示。

<center><img src="../assets/4-5.png"></center>
<center>图4-5 编辑模式</center>

&emsp;&emsp;按ESC键或运行cell中的代码，将进入命令模式。命令模式下，铅笔图标消失，cell左侧框线呈蓝色，如图4-6所示。

<center><img src="../assets/4-6.png"></center>
<center>图4-6 命令模式</center>

&emsp;&emsp;此时，按Enter键或鼠标点击cell中的编辑框，可切换回编辑模式。

## 3. 常用快捷键

&emsp;&emsp;Jupyter常用的快捷键如表1-1所示。更多快捷键可点击Jupyter菜单栏中的的Help->Keyboard Shortcuts查看。

<center>表4-1 Jupyter常用快捷键</center>
<center>

| 快捷键 | 作用 |
| :-: | :----------: |
| Ctrl+Enter | 运行当前cell |
| Shift+Enter | 运行当前cell，选中下一个cell |
| Alt+Enter | 运行当前cell，并在其下方插入新的cell |

</center>

## 4. IPython功能

&emsp;&emsp;在cell中可执行Linux的Shell命令，如图4-7所示。

<center><img src="../assets/4-7.png" width = 500></center>
<center>图4-7 在Jupyter中执行pwd和ls命令</center>

&emsp;&emsp;IPython还有许多其他功能，如magic，需要时可通过在cell中键入?查看。
