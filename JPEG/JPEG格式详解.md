JPEG中的位元组（字节）采用正序排列

JPEG文件的存储格式有很多种，但最常用的是JFIF格式，即JPEG File Interchange Format。

JPEG格式的主要不足之处也正是它的最大优点。也就是说，有损压缩算法将JPEG只局限於显示格式，而且每次保存JPEG格式的图像时都会丢失一些数据。因此，通常只在创作的最后阶段以JPEG格式保存一次图像即可。

在JFIF文件格式中，图像样本的存放顺序是从左到右和从上到下。

## 文件结构

JFIF文件格式直接使用JPEG标准爲应用程式定义的许多标记，因此JFIF格式成了事实上JPEG文件交换格式标准。JPEG的每个标记都是由2个位元组组成，其前一个位元组是固定值0xFF。每个标记之前还可以添加数目不限的0xFF填充位元组(fill byte)。

由于 JPEG 中以 0XFF 来做为特殊标记符，因此，如果某个像素的取值为 0XFF，那么实际在保存的时候，是以 0XFF00 来保存的，从而避免其跟特殊标记符 0XFF 之间产生混淆。所以，在读取文件信息的时候，如果遇 0XFF00，就必须去除后面的 00；即，将 0XFF00 当做0XFF；

下面是其中的8个标记：

> SOI 0xD8　 图像开始 </br>
> 　APP0 0xE0　 JFIF应用</br>
> 数据块</br>
> 　APPn 0xE1 - 0xEF　 其他的应用数据块(n, 1～15)</br>
> 　DQT 0xDB　 量化表</br>
> 　SOF0 0xC0　 帧开始</br>
> 　DHT 0xC4　 霍夫曼(Huffman)表</br>
> 　SOS 0xDA　 </br>
> 扫描线开始</br>
> 　EOI 0xD9　 图像结束</br>

 JPEG定义的标记：

> Symbol </br>
> (符号)    Code Assignment</br>
> (标记代码)    Deforbiddenion</br>
> (说明)</br>
> Start Of Frame markers, non-hierarchical Huffman coding</br>
> SOF0    0xFFC0    Baseline DCT</br>
> SOF1    0xFFC1    Extended sequential DCT</br>
> SOF2    0xFFC2    Progressive DCT</br>
> SOF3    0xFFC3    Spatial (sequential) lossless </br>
> Start Of Frame markers, hierarchical Huffman coding</br>
> SOF5    0xFFC5    Differential sequential DCT</br>
> SOF6    0xFFC6    Differential progressive DCT</br>
> SOF7    0xFFC7    Differential spatial lossless</br>
> Start Of Frame markers, non-hierarchical arithmetic coding</br>
> JPG    0xFFC8    Reserved for JPEG extensions</br>
> SOF9    0xFFC9    Extended sequential DCT</br>
> SOF10    0xFFCA    Progressive DCT</br>
> SOF11    0xFFCB    Spatial (sequential) Lossless</br>
> Start Of Frame markers, hierarchical arithmetic coding</br>
> SOF13    0xFFCD    Differential sequential DCT</br>
> SOF14    0xFFCE    Differential progressive DCT</br>
> SOF15    0xFFCF    Differential spatial Lossless</br>
> Huffman table specification</br>
> DHT    0xFFC4    Define Huffman table(s)</br>
> arithmetic coding conditioning specification</br>
> DAC    0xFFCC    Define arithmetic conditioning table</br>
> Restart interval termination</br>
> RSTm    0xFFD0～0xFFD7    Restart with modulo 8 counter m</br>
> Other marker</br>
> SOI    0xFFD8    Start of image</br>
> EOI    0xFFD9    End of image</br>
> SOS    0xFFDA    Start of scan</br>
> DQT    0xFFDB    Define quantization table(s)</br>
> DNL    0xFFDC    Define number of lines</br>
> DRI    0xFFDD    Define restart interval</br>
> DHP    0xFFDE    Define hierarchical progression</br>
> EXP    0xFFDF    Expand reference image(s) </br>
> APPn    0xFFE0～0xFFEF    Reserved for application use</br>
> JPGn    0xFFF0～0xFFFD    Reserved for JPEG extension</br>
> COM    0xFFFE    Comment</br>
> Reserved markers</br>
> TEM    0xFF01    For temporary use in arithmetic coding</br>
> RES    0xFF02～0xFFBF    Reserved</br>

## 文件组成部分

一般由下面9个部分组成：

### 图像开始SOI（Start of Image）标记

00-01h	2字节	0xFFD8

</br>

### APP0标记（Marker）

02-03h	2字节	0xFFE0

①APP0长度(length)（①~⑨九个字段的总长度）

04-05h	2字节内容不定	0010h

②标识符(identifier)

06-0ah	5字节	0x4A46494600即"JFIF0"

③ 版本号(version)

0b-0ch	2字节	0x0102 JFIF的版本号目前基本上都是1.2

④ X和Y的密度单位(units=0：无单位；units=1：点数/英寸；units=2：点数/厘米)

0dh	1字节只有0，1，2三个值可选，其分别代表的意义如上	01h

⑤ X方向像素密度(X density)

0e-0fh	2字节取值范围未知	0048h

⑥ Y方向像素密度(Y density)

10-11h	2字节取值范围未知	0048h

⑦ 缩略图水平像素数目(thumbnail horizontal pixels)

12h	1字节取值范围未知	00

⑧ 缩略图垂直像素数目(thumbnail vertical pixels)

13h	1字节取值范围未知	00

⑨ 缩略图RGB位图(thumbnail RGB bitmap)

无	长度可能是3的倍数内容不定

本段（APP0）可以包含图像的一个微缩版本，存为24位的RGB像素。如果没有微缩图像（这种情况更常见），则⑦“缩略图水平像素数目”和⑧“缩略图垂直像素数目”的值均为0

</br>

### APPn标记（Markers）

① APPn长度(length)（①②两个字段的总长度）

② 详细信息(application specific information)

对每个APP：

若为APPN（N=1～F（以16进制表示，N任选其中一个））

* 标记： mh	2字节	0xFFEN
* 长度：(m+2)h	2字节内容不定（设为n（10进制））（本字段与下一字段的总长度）
* 详细信息： (m+4)h	n－2字节（即长度减2）内容不定

</br>

14-15h	FFEE

16-17h	000E（14）

18-23h	41 64 6F 62 65 00 64 00 00 00 00 01

</br>

### 一个或者多个量化表DQT

**DQT(Difine Quantization Table)**

24-25h	2字节	0xFFDB

① 量化表长度(quantization table length)（①~②两个字段的总长度）

26-27h	2字节内容不定（①~②两个字段的总长度）	0084（132）

② 量化表(quantization table)

A. P/T(高四位：精度，低四位：表ID)

B. 表项

对每个量化表：

* P/T(高四位：精度，低四位：表ID) mh 1字节精度, 0 表示 8 bit, 1表示 16 bit；ID取值范围为0～3, 否则错误
* 表项 (m+1)h (64×(精度+1))字节内容长，故略



</br>



### 帧图像开始SOF0(Start of Frame)

aa-abh	2字节	0xFF**C0**

① 帧开始长度(start of frame length) （①~⑥六个字段的总长度）

ac-adh	2字节内容不定（①~⑥六个字段的总长度）	0011

② 精度(precision)，每个颜色分量每个像素的位数(bits per pixel per color component)

aeh	1字节	每个样本位数, 通常是 8 (大多数软件不支持 12 和 16)

③ 图像高度(image height) af-b0h	2字节内容不定（如果不支持 DNL 就必须 >0）	0317

④ 图像宽度(image width) b1-b2h	2字节内容不定（如果不支持 DNL 就必须 >0）	03E8

⑤ 颜色分量数(number of color components)

b3h	1字节	内容不定（灰度图是 1, YCbCr/YIQ 彩色图是 3, CMYK 彩色图是 4，我们这里讨论的JFIF使用的是YCbCr，故这里颜色分量数为3）	03

⑥ 对每个颜色分量(for each component)

A. ID

B. 垂直方向的样本因子(vertical sample factor)

C. 水平方向的样本因子(horizontal sample factor) B、C共占用1字节，B占用低4位，C占用高4位）

D. 量化表号(quantization table#)

JFIF格式使用的是YCbCr所以有3个分量（这里特别要注意的是颜色分量的ID号是有含义的，1代表Y，2代表Cb，3代表Cr，4代表I，5代表Q）：

1) ID

b4h	1字节	0x01

（高四位）水平（低四位）垂直样本因子

b5h	共1字节	0x11

量化表号

b6h	1字节	内容不定（本分量使用的量化表的ID号）	00

2) ID

b7h	1字节	0x02

（高四位）水平（低四位）垂直样本因子

b8h	共1字节	0x11

量化表号

b9h	1字节	内容不定（本分量使用的量化表的ID号）	01

3) ID

bah	1字节	0x03

（高四位）水平（低四位）垂直样本因子

bbh	共1字节	0x11

量化表号

bch	1字节	内容不定（本分量使用的量化表的ID号）	01

</br>

### 一个或者多个霍夫曼表DHT

**DHT (Difine Huffman Table)**

c3-c4h	2字节	0xFF**C4**

① 霍夫曼表的长度(Huffman table length) （①~②两个字段的总长度）

c5-c6h	2字节	内容不定（①~②两个字段的总长度）	01A2（418）

② 对每个霍夫曼表(一般情况下，霍夫曼表不止一个，但是绝对不多于4个)

A. 表号

B. 类型：AC或者DC(其中0：DC表，1：AC表)；A、B共占用1字节，A占用低4位，B占用高4位）

C. 长16个字节的编码，其代码代数和为接下来的编码的长度

D. 内容编码

对每个霍夫曼表：

* （高四位）类型和（低四位）表号： mh 共1字节内容不定（有四个可能：0x00表示第0个DC表，0x01表示第1个DC表，0x10表示第0个AC表，0x11表示第1个AC表）

* 长16个字节的编码： (m+1)h 16字节内容不定（设这16个字节上数据之和为n）

* 内容编码： (m+17)h n字节内容长，故略)

</br>

c7h	00

c8-d7h	00 00 07 01 01 01 01 01 00 00 00 00 00 00 00 00

</br>

### 定义重新开始间隔DRI

**DRI (Define Restart Interval)**

（在没有DRI标记，或间隔为零时，就不存在重新开始间隔和重开始标记）

bd-beh	2字节	0xFF**DD**

① 长度	bf-c0h	2字节	0x0004（①~②两个字段的总长度）

② MCU 块的单元中的重新开始间隔

c1-c2h	2字节	内容不定（设为n，则意思是说，每n个MCU块就有一个RSTn标记。第一个标记是RST0，然后是RST1等，RST7后再从RST0重复）	007D

</br>

### 扫描开始SOS

**SOS(Start of Scan)**

267-268h	2字节	0xFF**DA**

① 扫描开始长度(start of scan length)

269-26ah	2字节	内容不定（①~③再加上④的A\B\C的总长度）	000C（12）

② 颜色分量数(number of color components)

26bh	1字节应该和⑸⑤的值相同（灰度图是1, YCbCr/YIQ 彩色图是3, CMYK 彩色图是4）	03

③ 每个颜色分量

A. ID

B. 交流系数表号(AC table #)

C. 直流系数表号(DC table #)

（B、C共占用1字节，B：占用低4位，C：占用高4位）

由②得到这里的颜色分量数为3（这里的颜色分量的ID号的含义和⑸⑥的一样，1代表Y，2代表Cb，3代表Cr，4代表I，5代表Q）：

1） ID

26ch	1字节	0x01 （高四位）直流（低四位）交流数表号

26dh	共1字节	0x00

2） ID

26eh	1字节	0x02 （高四位）直流（低四位）交流数表号

26fh	共1字节	0x11

3） ID

270h	1字节	0x03 （高四位）直流（低四位）交流数表号

271h	共1字节	0x11

④ 压缩图像数据(compressed image data)

A. 谱选择开始 272h	1字节	0x00

B. 谱选择结束 273h	1字节	0x3F

C. 两个4位字段，高位和低位的谱选择 274h 1字节在基本JPEG中总为0x00

D. 数据 275h 长度不定内容长，故略

</br>

### 图像结束EOI

EOI (End of Image)

0h	2字节	0xFF**D9**