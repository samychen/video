# a_Save

[Android硬件加速原理与实现简介](https://tech.meituan.com/hardware-accelerate.html)

## CPU与GPU结构对比

CPU（Central Processing Unit，中央处理器）是计算机设备核心器件，用于执行程序代码，软件开发者对此都很熟悉；GPU（Graphics Processing Unit，图形处理器）主要用于处理图形运算，通常所说“显卡”的核心部件就是GPU。

下面是CPU和GPU的结构对比图。其中：

- 黄色的Control为控制器，用于协调控制整个CPU的运行，包括取出指令、控制其他模块的运行等；
- 绿色的ALU（Arithmetic Logic Unit）是算术逻辑单元，用于进行数学、逻辑运算；
- 橙色的Cache和DRAM分别为缓存和RAM，用于存储信息。

![img](https://tech.meituan.com/img/hardware-accelerate/cpu-gpu.png)



- 从结构图可以看出，CPU的控制器较为复杂，而ALU数量较少。因此CPU擅长各种复杂的逻辑运算，但不擅长数学尤其是浮点运算。
  - 以8086为例，一百多条汇编指令大部分都是逻辑指令，数学计算相关的主要是16位加减乘除和移位运算。一次整型和逻辑运算一般需要1~3个机器周期，而浮点运算要转换成整数计算，一次运算可能消耗上百个机器周期。
  - 更简单的CPU甚至只有加法指令，减法用补码加法实现，乘法用累加实现，除法用减法循环实现。
  - 现代CPU一般都带有硬件浮点运算器（FPU），但主要适用于数据量不大的情况。
- CPU是串行结构。以计算100个数字为例，对于CPU的一个核，每次只能计算两个数的和，结果逐步累加。
- 和CPU不同的是，GPU就是为实现大量数学运算设计的。从结构图中可以看到，GPU的控制器比较简单，但包含了大量ALU。GPU中的ALU使用了并行设计，且具有较多浮点运算单元。
- **硬件加速的主要原理，就是通过底层软件代码，将CPU不擅长的图形计算转换成GPU专用指令，由GPU完成**。

> 扩展：很多计算机中的GPU有自己独立的显存；没有独立显存则使用共享内存的形式，从内存中划分一块区域作为显存。显存可以保存GPU指令等信息。

## Android中的硬件加速

在Android中，大多数应用的界面都是利用常规的View来构建的（除了游戏、视频、图像等应用可能直接使用OpenGL ES）

```js
var VSHADER_SOURCE = `
        attribute vec2 a_Position;
        varying vec2 uv;

        void main(){
            gl_Position = vec4(a_Position,0.0,1.0);
            uv = vec2(0.5, 0.5) * (a_Position + vec2(1.0, 1.0));
        }
`;
var FSHADER_SOURCE = `
        precision mediump float;
        precision highp float;
        varying vec2 uv;

        uniform float progress, ratio;

        uniform sampler2D u_Sampler0, u_Sampler1;
            vec4 getFromColor(vec2 uv){
            return texture2D(u_Sampler0, uv);
        }
        vec4 getToColor(vec2 uv){
        	return texture2D(u_Sampler1, uv);
        }
        vec4 transition (vec2 uv) {
            return mix(
            getFromColor(uv),
            getToColor(uv),
            progress
            );
        }

        void main(){
        	gl_FragColor=transition(uv);
        }
`;
```







rasterize 光栅化，点阵化
