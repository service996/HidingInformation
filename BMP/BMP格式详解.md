## 简介

​		BMP（Bitmap）是是 Windows 操作系统中的标准图像文件格式，可以分成两类：设备相关位图（DDB）和设备无关位图（DIB），使用非常广。它采用位映射存储格式，除了图像深度可选以外，不采用其他任何压缩，因此，BMP 文件所占用的空间很大。BMP 文件的图像深度可选 lbit、4bit、8bit 及 24bit。BMP 文件存储数据时，图像的扫描方式是按从左到右、从下到上的顺序。由于 BMP 文件格式是Windows 环境中交换与图有关的数据的一种标准，因此在Windows环境中运行的图形图像软件都支持 BMP 图像格式。

## 格式详解

### 文件结构

​	典型位图文件可看成由4个部分组成：

1. **位图文件头**：包含BMP图像文件类型、显示内容等信息；

2. **位图信息头**：包含BMP图像宽、高、压缩方法，以及定义颜色等信息；

3. **调色板**：可选，有些位图需要调色板，有些位图，比如真彩色图（24 位的 BMP）就不需要调色板；

4. **位图数据**：内容根据 BMP 位图使用的位数不同而不同，在 24 位图中直接使用 RGB，而其他的小于 24 位的使用调色板中颜色索引值。

   以下文件数据都是对于该文件夹下的BMP图像的分析数据

#### 位图文件头

采用小端（逆序）存储方式，即（低地址村存放低位数据，高地址存放高位数据），与之相反也有大端（正序）存储的方式

![位图文件头](https://chenshuoke-pictures.oss-cn-beijing.aliyuncs.com/HidingInformation/BMP/BMP_FileHeader.png)

00-01：424Dh = 'BM'，表示这是如今Windows支持的位图格式

02-05：0001F116hB = 127254B = 124.27KB（16进制单位字节，转化为10进制单位字节）

06-09：00000000，这是两个保留段，必须设置为0

0A-0D：00000036h = 54，即从头文件到位图数据需偏移54字节

#### 位图信息头

![位图信息头](https://chenshuoke-pictures.oss-cn-beijing.aliyuncs.com/HidingInformation/BMP/BMP_InfoHeader.png)

0E-11：00000028h = 40，文件信息头的大小，一般为40字节

12-15：0000012Bh = 299，图像宽为299像素

16-19：000000D4h = 212，图像高为212像素

1A-1B：0001h = 1，为目标设备说明颜色平面数，该值总为1

1C-1D：0010h = 16，每像素所含有的比特数（位数），其值为1、4、8、16、24或32

> 1	-	单色位图（实际上可有两种颜色，缺省情况下是黑色和白色。你可以自己定义这两种颜色）
>
> 4	-	16色位图
>
> 8	-	256色位图
>
> 16	-	16bit高彩色位图
>
> 24	-	24bit真彩色位图
>
> 32	-	32bit增强型真彩色位图

1E-21：00000000h，说明图像数据的压缩类型。

> 0 - 不压缩 （使用BI_RGB表示）
>
> 1 - RLE 8-使用8位RLE压缩方式（用BI_RLE8表示）
>
> 2 - RLE 4-使用4位RLE压缩方式（用BI_RLE4表示）
>
> 3 - Bitfields-位域存放方式（用BI_BITFIELDS表示）

22-25：0001F0E0hB = 127200B = 124.21KB，说明图像的大小，单位字节（使用BI_RGB时，可设置为0）该数必须是4的倍数

26-29：00000000h，说明水平分辨率，用像素/米表示，有符号整数（缺省）

2A-2D：00000000h，说明竖直分辨率，用像素/米表示，有符号整数（缺省）

2E-31：00000000h，说明位图实际使用的彩色表中的颜色索引数（设为0的话，则说明使用所有调色板项）

32-35：00000000h，说明对图像显示有重要影响的颜色索引的数目（设为0的话，表示都重要）

#### 调色板

16位，无压缩（BI_RGB）；无调色板

关于这一点可以参考 [bitBitCount](#bitBitCount)

#### 位图数据

36h开始至结束

### 构件详解

#### 位置头文件

位图文件头包含有关于文件类型、文件大小、存放位置等信息，在Windows 3.0以上版本的位图文件中用BITMAPFILEHEADER结构来定义：

```c++
typedef struct tagBITMAPFILEHEADER {
	/* bmfh */
	UINT bfType;
	DWORD bfSize;
	UINT bfReserved1;
	UINT bfReserved2;
	DWORD bfOffBits;
} BITMAPFILEHEADER;
```

其中：

bfType

说明文件的类型.（该值必需是**0x4D42**，也就是字符'BM'。我们不需要判断OS/2的位图标识，这么做看来似乎已经没有什么意义了，而且如果要支持OS/2的位图，程序将变得很繁琐。所以，在此只建议你检察'BM'标识）

**注意：查ascii表B 0x42,M0x4d,bfType为两个字节，B为low字节，M为high字节所以bfType=0x4D42，而不是0x424D，但注意**

bfOffBits

说明从文件头开始到实际的图象数据之间的字节的偏移量。这个参数是非常有用的，因为位图信息头和调色板的长度会根据不同情况而变化，所以你可以用这个偏移值迅速的从文件中读取到位数据。

#### 位图信息

位图信息用BITMAPINFO结构来定义，它由位图信息头（bitmap-information header）和彩色表（color table）组成，前者用BITMAPINFOHEADER结构定义，后者用RGBQUAD结构定义。BITMAPINFO结构具有如下形式：

```c++
vtypedef struct tagBITMAPINFO {
	/* bmi */
	BITMAPINFOHEADER bmiHeader;
	RGBQUAD bmiColors[1];
} BITMAPINFO;
```

其中：

bmiHeader

说明BITMAPINFOHEADER结构，其中包含了有关位图的尺寸及位格式等信息

bmiColors

说明彩色表RGBQUAD结构的阵列，其中包含索引图像的真实RGB值。

BITMAPINFOHEADER结构包含有位图文件的大小、压缩类型和颜色格式，其结构定义为：

```c++
typedef struct tagBITMAPINFOHEADER {
	/* bmih */
	DWORD biSize;
	LONG biWidth;
	LONG biHeight;
	WORD biPlanes;
	WORD biBitCount;
	DWORD biCompression;
	DWORD biSizeImage;
	LONG biXPelsPerMeter;
	LONG biYPelsPerMeter;
	DWORD biClrUsed;
	DWORD biClrImportant;
} BITMAPINFOHEADER;
```

其中：

biSize

说明BITMAPINFOHEADER结构所需要的字数。注：这个值并不一定是BITMAPINFOHEADER结构的尺寸，它也可能是sizeof(BITMAPV4HEADER）的值，或是sizeof(BITMAPV5HEADER）的值。这要根据该位图文件的格式版本来决定，不过，就现在的情况来看，绝大多数的BMP图像都是BITMAPINFOHEADER结构的（可能是后两者太新的缘故吧:-）。

biWidth

说明图象的宽度，以象素为单位

biHeight

说明图象的高度，以象素为单位。注：这个值除了用于描述图像的高度之外，它还有另一个用处，就是指明该图像是倒向的位图，还是正向的位图。如果该值是一个正数，说明图像是倒向的，如果该值是一个负数，则说明图像是正向的。大多数的BMP文件都是倒向的位图，也就是时，高度值是一个正数。（注：当高度值是一个负数时（正向图像），图像将不能被压缩（也就是说biCompression成员将不能是BI_RLE8或BI_RLE4）。

biPlanes

为目标设备说明位面数，其值将总是被设为1

biBitCount

说明比特数/象素，其值为1、4、8、16、24、或32

biCompression

说明图象数据压缩的类型。其值可以是下述值之一：

BI_RGB：没有压缩；

BI_RLE8：每个象素8比特的RLE压缩编码，压缩格式由2字节组成（重复象素计数和颜色索引）；

BI_RLE4：每个象素4比特的RLE压缩编码，压缩格式由2字节组成

BI_BITFIELDS：每个象素的比特由指定的掩码决定。

biSizeImage

说明图象的大小，以字节为单位。当用BI_RGB格式时，可设置为0

biXPelsPerMeter

说明水平分辨率，用象素/米表示

biYPelsPerMeter

说明垂直分辨率，用象素/米表示

biClrUsed

说明位图实际使用的彩色表中的颜色索引数（设为0的话，则说明使用所有调色板项）

biClrImportant

说明对图象显示有重要影响的颜色索引的数目，如果是0，表示都重要。

现就BITMAPINFOHEADER结构作如下说明：

#### 彩色表定位

应用程序可使用存储在biSize成员中的信息来查找在BITMAPINFO结构中的彩色表，如下所示：

pColor = ((LPSTR) pBitmapInfo + (WORD) (pBitmapInfo->bmiHeader.biSize))

#### biBitCount

biBitCount=1 表示位图最多有两种颜色，缺省情况下是黑色和白色，你也可以自己定义这两种颜色。图像信息头装调色板中将有两个调色板项，称为索引0和索引1。图象数据阵列中的每一位表示一个象素。如果一个位是0，显示时就使用索引0的RGB值，如果位是1，则使用索引1的RGB值。

biBitCount=4 表示位图最多有16种颜色。每个象素用4位表示，并用这4位作为彩色表的表项来查找该象素的颜色。例如，如果位图中的第一个字节为0x1F，它表示有两个象素，第一象素的颜色就在彩色表的第2表项中查找，而第二个象素的颜色就在彩色表的第16表项中查找。此时，调色板中缺省情况下会有16个RGB项。对应于索引0到索引15。

biBitCount=8 表[位图最多有256种颜色。每个象素用8位表示，并用这8位作为彩色表的表项来查找该象素的颜色。例如，如果位图中的第一个字节为0x1F，这个象素的颜色就在彩色表的第32表项中查找。此时，缺省情况下，调色板中会有256个RGB项，对应于索引0到索引255。

biBitCount=16 表示位图最多有65536种颜色。每个色素用16位（2个字节）表示。这种格式叫作高彩色，或叫增强型16位色，或64K色。它的情况比较复杂，当biCompression成员的值是BI_RGB时，它没有调色板。16位中，最低的5位表示蓝色分量，中间的5位表示绿色分量，高的5位表示红色分量，一共占用了15位，最高的一位保留，设为0。这种格式也被称作555 16位位图。如果biCompression成员的值是BI_BITFIELDS，那么情况就复杂了，首先是原来调色板的位置被三个DWORD变量占据，称为红、绿、蓝掩码。分别用于描述红、绿、蓝分量在16位中所占的位置。在Windows 95（或98）中，系统可接受两种格式的位域：555和565，在555格式下，红、绿、蓝的掩码分别是：0x7C00、0x03E0、0x001F，而在565格式下，它们则分别为：0xF800、0x07E0、0x001F。你在读取一个像素之后，可以分别用掩码“与”上像素值，从而提取出想要的颜色分量（当然还要再经过适当的左右移操作）。在NT系统中，则没有格式限制，只不过要求掩码之间不能有重叠。（注：这种格式的图像使用起来是比较麻烦的，不过因为它的显示效果接近于真彩，而图像数据又比真彩图像小的多，所以，它更多的被用于游戏软件）。

biBitCount=24 表示位图最多有1670万种颜色。这种位图没有调色板（bmiColors成员尺寸为0），在位数组中，每3个字节代表一个象素，分别对应于颜色R、G、B。

biBitCount=32 表示位图最多有4294967296(2的32次方）种颜色。这种位图的结构与16位位图结构非常类似，当biCompression成员的值是BI_RGB时，它也没有调色板，32位中有24位用于存放RGB值，顺序是：最高位—保留，红8位、绿8位、蓝8位。这种格式也被成为888 32位图。如果  biCompression成员的值是BI_BITFIELDS时，原来调色板的位置将被三个DWORD变量占据，成为红、绿、蓝掩码，分别用于描述红、绿、蓝分量在32位中所占的位置。在Windows 95/Windows  98中，系统只接受888格式，也就是说三个掩码的值将只能是：0xFF0000、0xFF00、0xFF。而在NT系统中，你只要注意使掩码之间不产生重叠就行。（注：这种图像格式比较规整，因为它是DWORD对齐的，所以在内存中进行图像处理时可进行汇编级的代码优化（简单））。

