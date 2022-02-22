---
title: DICOM相关信息
date: 2022-02-20 13:54:52
index_img: /img/cover/DICOM.png
cover: /img/cover/DICOM.png
tags:
- Tags
- DICOM
categories:
- 医疗相关
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---

### 参考文献

* [DICOM的常用Tag分类和说明](https://blog.csdn.net/inter_peng/article/details/46513847)
* [DICOM PS3.4 2022a - Service Class Specifications](https://dicom.nema.org/medical/dicom/current/output/html/part04.html#sect_C.4.1)
* [[Dcmlib] HOWTO Compute the Z spacing in DICOM](https://www.creatis.insa-lyon.fr/pipermail/dcmlib/2005-September/002141.html)
* [闲话DICOM](https://www.cnblogs.com/okaimee/archive/2013/01/10/2854783.html)
* [[医疗]国外开源的PACS服务器](https://www.cnblogs.com/kesalin/archive/2012/02/24/medical_pacs.html)

### 名词解释

* 基于*DICOM3.0*标准的医学图像中，每一张图像中都携带着许多的信息，这些信息主要可以分为**Patient, Study, Series**和**Image**四类。
* 每一个*DICOM Tag*都是由两个十六进制数的组合来确定的，分别为*Group*和*Element*。如*(0010,0010)*这个*Tag*表示的是*Patient’s Name*，它存储着这张*DICOM*图像的患者姓名。
* 处理DICOM的第三方库
  * 基于*C++*的*DCMTK*
  * 基于*Java*的*dcm4che*
* 普通DICOM 文件包含的四级属性，病人，检查，序列，影像。每一级别需要具有能够唯一标识这个等级属性的键值，类似关系数据库中的主键。
  * 病人对应的为Patient ID 可以为空,只是局部唯一
  * 检查 Study Instance UID
  * 序列 Series Instance UID
    * Modality, Bodypartexaminated, Patient Position, View Position, Series number, Series timez这些属性对医生来说, 图像质量,和挂片顺序(协议)非常重要，这些都是挂片条件！
  * 影像 SOP Instance UID
    * Image Number, image time(content time)

#### **HIS**

* Hospital Information System，医院信息系统，在国际学术界已公认为新兴的医学信息学(Medical Informatics)的重要分支。美国该领域的著名教授Morris.Collen于1988年曾著文为医院信息系统下了如下定义：利用电子计算机和通讯设备，为医院所属各部门提供病人诊疗信息和行政管理信息的收集、存储、处理、提取和数据交换的能力，并满足所有授权用户的功能需求。

#### **RIS**

* Radioiogy information system，放射信息管理系统。RIS是优化医院放射科工作流程管理的软件系统,一个典型的流程包括登记预约、就诊、产生影像、出片、报告、审核、发片等环节。RIS系统内含PACS系统，配合医学分类和检索、放射物资管理、影像设备管理和科室信息报表等外围模块，实现了患者在整个流程中的质量控制和实地跟踪，差错统计，为医患纠纷的举证倒置提供依据，从而使得放射科室的管理进入到清晰的数字化管理阶段。

#### **PACS**

* Picture archiving and communication systems，即医学影像存档与通讯系统。是近年来随着数字成像技术、计算机技术和网络技术的进步而迅速发展起来的、旨在全面解决医学图像的获取、显示、存贮、传送和管理的综合系统。 PACS在医院影像科室中迅速普及开来。如同计算机与互联网日益深入地影响我们的日常生活，PACS也在改变着影像科室的运作方式，一种高效率、无胶片化影像系统正在悄然兴起。在这些变化中，PACS的主要作用有：联接不同的影像设备（CT、MR、XRAY、超声、核医学等）；存储与管理图像；图像的调用与后处理。不同的PACS在组织与结构上可以有很大的差别，但都必须能完成这三种类型的功能。 对于PACS的实施，各个部门根据各自所处地区和经济状况的不同而可能有各自的实施方式和实施范围。不管是大型、中型或小型PACS，其建立不外乎由医学图像获取、大容量数据存储及数据库管理、图像显示和处理以及用于传输影像的网络等多个部分组成，保证PACS成为全开放式系统的重要的网络标准和协议DICOM3.0。

#### **LIS**

* Laborary information system，实验信息系统，是实验室自动化、现在化、正规化管理的必然要求，要求能够提供的功能有： 检验单录入（病人信息、结果数据），质量控制（室内质控、室间质控），检验数据工具（数据合并、修改、历史数据的查询），不同用户的授权，微生物药敏的专门软件。

#### **CIS**

* Clinical Information System，临床信息系统，其目标是支持医院医护人员的临床活动，收集和处理病人的临床医疗信息，丰富和积累临床医学知识，并提供临床咨询、辅助诊疗、辅助临床决策，提高医护人员的工作效率，为病人提供更多、更快、更好的服务。象医嘱处理系统、病人床边系统、医生工作站系统、实验室系统、药物咨询系统等就属于CIS范围。

#### **MRI**

* Magnetic Resonance Imaging，磁共振成像，是一种生物自旋成像技术，利用原子核自旋运动的特点，使用磁场人体层面的空间位置，利用无线电波进行序列照射，激发原子核产生共振。当停止无线电波照射，原子核自动恢复到平衡状态，把吸收的能量放出来。这个能量信号可用探测器检测，输入计算机进行编码，再用计算机创建图形。

#### **DICOM**

* Digital Imaging and Communications in Medicine，医学数字成像和通信标准，DICOM标准详细定义了影像及其相关信息的组成格式和交换方法，利用这个标准，人们可以在影像设备上建立一个接口来完成影像数据的输入/输出工作。

#### **WADO**

* Web Access to DICOM Objects，DICOM对象的Web接入标准，WADO的主要目地就是要共通化 URL 的格式及方法，使得不同厂商的DICOM 服务器和电子病历系统的组合均能兼容，并在电子病历系统上也能显示DICOM 影像。WADO 规格定义了客户端，如电子病历等系统，如何从 Web Enabled DICOM 服务器取得影像数据的URL格式及方法，以及相关的技术要求。

#### **VTK**

* Visualization toolkit，是一个开放资源的免费软件系统，主要用于三维计算机图形学、图像处理和可视化。Vtk是在面向对象原理的基础上设计和实现的，它的内核是用C++构建的，包含有大约250,000行代码，650多个类，还包含有几个转换界面，因此也可以自由的通过Java，Tcl/Tk和Python各种语言使用vtk。 Vtk几乎可以在任何一个基于Unix的平台上操作，以及Windows 95/98/NT/2000/XP。

#### **ITK**

* Insight Segmentation and Registration Toolkit 是一种开源的、跨平台的影像分析扩展软件工具。ITK的开发过程中采用了先进的多模态数据分割配准算法。它提供一些主流算法，如区域生长、阈值分割、基于分水岭的分割、Fast Marching算法、Level Set等多种分割算法，并将这些用于医学图像处理的算法和程序的开发过程屏蔽起来。ITK没有实现可视化的功能，在VTK中可以实现可视化，所以医学影像系统中，在用ITK进行分割的基础上，结合VTK对图像进行可视化处理。

#### **MIP**

* Maximum Intensity Projection，最大密度投影，有时又称为“**最大亮度投影**”，是在可视化平面之上投射三维空间数据的一种计算机可视化方法；其中，沿着从视点到投影平面的平行光线，各个体素密度值的所呈现的亮度将以某种方式加以衰减，并且最终在投影平面上呈现的是亮度最大的体素。

#### **MinIP**

* Minimum Intensity Projection，最大密度投影

#### **MPR**

* Multiplanar rerormation，多层平面重建

#### **MMPR**

* 双平面重建

#### **CPR**

* Curve multiplanar reformation，曲面重建技术

#### **ROI**

* Region on interest，感兴趣区

#### **MSCT**

* Multi—slice Spiral CT，多排螺旋CT

#### **CT**

* electronic computer X-ray tomography technique，CT是一种功能齐全的病情探测仪器，它是电子计算机[X射线](http://baike.baidu.com/view/45735.htm)断层扫描技术简称。

#### **DR**

* Digital Radography，也叫数字摄影，是采用平板探测器对X线产生的图像信号进行扫描和直接读出，成像原理是现将X线信号转变为可见光，通过光电二极管组成的藻膜层（TI）进行聚集，由专门的读出电路直接读出，送至计算机系统进行处理。
* DR与CR比较优点主要有以下几点方面：

  1. DR分辨率高图像清晰度优于CR。
  2. DR信噪比高于CR。
  2. 拍片速度比CR快。
  4. DR的X线转换效率更高故曝光计量较CR低。

#### **CR**

* Computed Radiography，也称为间接数字化成像，主要原理是利用存储荧光体成像，这种荧光体采用磷光体结晶构成的成像板 (Imaging Plate) 即IP吸收X线信息，IP板感光形成潜影，在经过扫描转化成数字化信号，进入计算机系统进行图像处理。

* CR的优点：
  1. CR的曝光剂量与常规X摄影相比，曝光剂量要比常规片小。
  2. 摄影条件要求比胶片低，几乎很少有废片。
  3. 采用CR时X线装备不用经过大的改变，其拍片过程与原有的X线摄影没有什么变化。
  4. 图像的后处理功能优越，可提高影像诊断的准确性及病诊范围。


#### **MRI**

* Magnetic Resonance Imaging，磁共振共像

#### **EHR**

* Electronic Health Record，电子健康记录，是个人官方的健康记录，这些记录可以在多个设备和机构中共享。

#### **SCR**

* 序列创建功能

#### **AIP**

* 平均投影

#### **VE**

* 虚拟内镜可视化

### 数据类型-值表现(VR)

| VR                                                     | 含义                                                         | 允许字符                                                     | 数据长度                      |
| ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------- |
| ***CS*** *- Code String*代码字符串                     | 开头结尾可以有没有意义的空格的字符串，比如*“CD123_4”*        | 大写字母，*0-9*，空格以及下划线字符                          | 最多 *16* 个字符              |
| **SH** *- Short String*短字符串                        | 短字符串，比如*:*电话号码，*ID*等                            |                                                              | 最多 *16* 个字符              |
| ***LO*** *- Long String* 长字符串                      | 一个字符串，可能在开头、结尾填有空 格。比如*“Introduction to DICOM”* |                                                              | 最多 *64* 个字符              |
| ***ST*** *- Short Text*短文本                          | 可能包含一个或多个段落的字符串                               |                                                              | 最多 *1024* 个字符            |
| ***LT*** *- Long Text*短文本                           | 可能包含一个或多个锻炼的字符串，与*LO*相同，但可以更长       |                                                              | 最多 *10240* 个字符           |
| ***UT*** *- Unlimited Text*无限制文本                  | 包含一个或多个段落的字符串，与 *LT* 类似                     |                                                              | 最多*(2*的*32*次方*–2)*个字符 |
| ***AE*** *- Application Entity*应用实体                | 标识一个设备的名称的字符串，开头和 结尾可以有无意义的字符。比如 *“MyPC01”* |                                                              | 最多 *16* 个字符              |
| ***PN*** *- Person Name*病人姓名                       | 有插入符号*(^)*作为姓名分隔符的病人姓名。比如*“SMITH^JOHN” “Morrison- Jones^Susan^^^Ph.D*， *Chief Executive Officer”* |                                                              | 最多 *64* 个字符              |
| ***UI*** *- Unique Identifier (UID)*唯一标识符         | 一个用作唯一标识各类项目的包含 *UID* 的字符串。比如*“1.2.840.10008.1.1”* | *0-9* 和半角句号(.)                                          | 最多*64* 个字符               |
| ***DA*** *- Date*日期                                  | 格式为 *YYYYMMDD* 的字符串；*YYYY* 代表年；*MM* 代表月；*DD* 代表日。比 如*“20050822”*表示 *2005* 年 *8* 月 *22* 日 | *0-9*                                                        | *8*个字符                     |
| ***TM*** *- Time*时间                                  | 格式为 *HHMMSS* 的字符串。*FRAC*； *HH* 表示小时*(*范围*“00”-“23”)*； *MM* 表示分钟*(*范围*“00”-“59”)*； 而 *FRAC* 包含秒的小数部分，即百万分 之一秒。比如*“183200.00”* 表示下午 *6:32* | *0-9* 和半角句号(.)                                          | 最多 *16* 个字符              |
| ***DT*** *- Date Time*日期时间                         | 格式为 YYYYMMDDHHMMSS. FFFFFF，串联的日期时间字符串。字符串的各部分从左至右是:年 YYYY；月 MM；日 DD；小时 HH；分钟 MM；秒 SS；秒的小数 FFFFFF。比如 20050812183000.00”表示 2005 年 8 月 12 日下午 18 点 30 分 00 秒 | *0-9*，加号，减号和半角句号                                  | 最多 *26* 个字符              |
| ***AS*** *- Age String*年龄字符串                      | 符合以下格式的字符串*:nnnD*， *nnnW*， *nnnM*， *nnnY*；其中 *nnn* 对于 *D* 来说表示天数，对于*W*来说表示周数，对于*M* 来说表示月数，对于 *Y* 来说表示岁数。 比如*“018M”*表示他的年龄是 *18* 个月 | *0–9*， *D*， *W*，*M*， *Y*                                 | *4* 个字符                    |
| ***S*** *- Integer String*整型字符串                   | 表示一个整型数字的字符串。比如*“-1234567”*                   | *0-9*，加号(+)，减号(-)                                      | 最多 *12* 个字符              |
| ***DS*** *- Decimal String* 小数字符串                 | 表示定点小数和浮点小数。 比如*“12345.67”*，*“-5.0e3”*        | *0-9*，加号(+)，减号(-)， 最多 *16* 个字符 *E*，*e* 和半角句号(.) | 最多 *16* 个字符              |
| ***SS*** *- Signed Short*有符号短型                    | 符号型二进制整数，长度 *16* 比特                             |                                                              | *2* 个字符                    |
| ***US*** *- Unsigned Short* 无符号短型                 | 无符号二进制整数，长度 *16* 比特                             |                                                              | *2* 个字符                    |
| ***SL*** *- Signed Long*有符号长型                     | 有符号二进制整数                                             |                                                              | *4* 个字符                    |
| ***UL*** *- Unsigned Long* 无符号长型                  | 无符号二进制整数，长度 *32* 比特                             |                                                              | *4* 个字符                    |
| ***AT*** *- Attribute Tag*属性标签                     | *16* 比特无符号整数的有序对，数据元素的标签                  |                                                              | *4* 个字符                    |
| ***FL*** *- Floating Single* 单精度浮点                | 单精度二进制浮点数字                                         |                                                              | *4* 个字符                    |
| ***FD*** *- Floating Point Double*双精度二进制浮点数字 | 双精度二进制浮点数字                                         |                                                              | *8* 个字符                    |
| ***OB*** *- Other Byte String*其他字节字符串           | 字节的字符串（*“*其他*”*表示没有在*VR*中定义的内容）         |                                                              |                               |
| ***OW*** *- Other Word String*其他单词字符串           | *16* 比特*(2* 字节*)*单词字符串                              |                                                              |                               |
| ***OF*** *- Other Float String*其他浮点字符串          | *32* 比特*(4* 个字节*)*浮点单词字符串                        |                                                              |                               |
| ***SQ*** *- Sequence Items*条目序列                    | 条目的序列                                                   |                                                              |                               |
| ***UN**– Unknown*未知                                  | 字节的字符串，其中内容的编码方式是未知的                     |                                                              |                               |

### 编码方式

* 传输语义(Transfer Syntax)
  * implictlittleendian
  * explicitlittleedian
  * explicitbigendian
  * JPEG Loseless
  * JPEG Lossy
  * RLE
  * JPEG2K
  * MPEG2
  * MPEG4

### DICOM Tag分类

#### *Patient Tag*(病人)

| *Group* | *Element* | *Tag Description*      | 中文解释     | *VR* |
| ------- | --------- | ---------------------- | ------------ | ---- |
| *0010*  | *0010*    | *Patient’s Name*       | 患者姓名     | *PN* |
| *0010*  | *0020*    | *Patient ID*           | 患者*ID*     | *LO* |
| *0010*  | *0030*    | *Patient’s Birth Date* | 患者出生日期 | *DA* |
| *0010*  | *0032*    | *Patient’s Birth Time* | 患者出生时间 | *TM* |
| *0010*  | *0040*    | *Patient’s Sex*        | 患者性别     | *CS* |
| *0010*  | *1030*    | *Patient’s Weight*     | 患者体重     | *DS* |
| *0010*  | *21C0*    | *Pregnancy Status*     | 怀孕状态     | *US* |

#### *Study Tag*(检验)

| *Group* | *Element* | *Tag Description*                                            | 中文解释                                        | *VR* |
| ------- | --------- | ------------------------------------------------------------ | ----------------------------------------------- | ---- |
| *0008*  | *0050*    | ***Accession Number**: A RIS generated number that identifies the order for the Study.* | 检查号：*RIS*的生成序号,用以标识做检查的次序.   | *SH* |
| *0020*  | *0010*    | ***Study ID***                                               | 检查ID.                                         | *SH* |
| *0020*  | *000D*    | ***Study Instance UID**:Unique identifier for the Study.*    | 检查实例号：唯一标记不同检查的号码              | *UI* |
| *0008*  | *0020*    | ***Study Date***：*Date the Study started.*                  | 检查日期：检查开始的日期                        | *DA* |
| *0008*  | *0030*    | ***Study Time***：*Time the Study started.*                  | 检查时间：检查开始的时间                        | *TM* |
| *0008*  | *0061*    | ***Modalities in Study***                                    | 一个检查中含有的不同检查类型                    | *CS* |
| *0008*  | *0015*    | ***Body Part Examined***                                     | 检查的部位                                      | *CS* |
| *0008*  | *1030*    | ***Study Description***                                      | 检查的描述                                      | *LO* |
| *0010*  | *1010*    | ***Patient’s Age***                                          | 做检查时刻的患者年龄*,*而不是此刻患者的真实年龄 | *AS* |

#### *Series Tag*(系列)

| *Group*  | *Element* | *Tag Description*                                            | 中文解释                                                     | *VR* |
| -------- | --------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| **0020** | **0011**  | **Series Number**: A number that identifies this Series.     | 序列号：识别不同检查的号码                                   | IS   |
| **0020** | **000E**  | **Series Instance UID**:Unique identifier for the Series.    | 序列实例号：唯一标记不同序列的号码                           | UI   |
| **0008** | **0060**  | **Modality**                                                 | 检查模态(MRI/CT/CR/DR)                                       | CS   |
| **0008** | **103E**  | **Series Description**                                       | 检查描述和说明                                               | LO   |
| **0008** | **0021**  | **Series Date**                                              | 检查日期                                                     | DA   |
| **0008** | **0031**  | **Series Time**                                              | 检查时间                                                     | TM   |
| **0020** | **0032**  | **Image Position (Patient)**：The x, y and z coordinates of the upper left hand corner of the image, in mm. | 图像位置：图像的左上角在空间坐标系中的x,y,z坐标,单位是毫米. 如果在检查中,则指该序列中第一张影像左上角的坐标. | DS   |
| **0020** | **0037**  | **Image Orientation (Patient)**:The direction cosines of the first row and the first column with respect to the patient. | 图像方位：第一行和第一列的方向余弦相对于病人                 | DS   |
| **0018** | **0050**  | **Slice Thickness**:Nominal slice thickness, in mm.          | 层厚.                                                        | DS   |
| **0018** | **0088**  | **Spacing Between Slices**                                   | 层与层之间的间距,单位为mm                                    | DS   |
| **0020** | **1041**  | **Slice Location**：Relative position of exposure expressed in mm. | 实际的相对位置，单位为mm                                     | DS   |
| **0018** | **0023**  | **MR Acquisition**                                           | 数据编码表的标识。枚举值：<br/>2D = 频率x相位<br/>3D = 频率x相位x相位 | CS   |
| **0018** | **0015**  | **Body Part Examined**                                       | 身体部位                                                     | CS   |

#### *Image Tag*(图像)

| *Group* | *Element* | *Tag Description*                                            | 中文解释                                                     | *VR* |
| ------- | --------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| *0008*  | *0008*    | ***Image Type**: Image identification characteristics.*      | 图像类型: 图象标识字特性                                     | *CS* |
| *0008*  | *0018*    | ***SOP Instance UID***                                       | *SOP*实例*UID*                                               |      |
| *0008*  | *0023*    | ***Content Date***：*The date the image pixel data creation started.* | 影像拍摄的日期                                               | *DA* |
| *0008*  | *0033*    | ***Content Time***                                           | 影像拍摄的时间                                               | *TM* |
| *0020*  | *0013*    | ***Image/Instance Number**: A number that identifies this image.* | 图像码：辨识图像的号码                                       | *IS* |
| *0028*  | *0002*    | ***Samples Per Pixel**: Number of samples (planes) in this image.* | 图像上的采样率                                               | *US* |
| *0028*  | *0004*    | ***Photometric Interpretation**: Specifies the intended interpretation of the pixel data.* | 光度计的解释*,*对于*CT*图像，用两个枚举值:*MONOCHROME1*，*MONOCHROME2.*用来判断图像是否是彩色的，*MONOCHROME1/2*是灰度图，*RGB*则是真彩色图，还有其他 | *CS* |
| *0028*  | *0010*    | ***Rows**: Number of rows in the image.*                     | 图像的总行数，行分辨率                                       | *US* |
| *0028*  | *0011*    | ***Columns**: Number of columns in the image.*               | 图像的总列数，列分辨率                                       | *US* |
| *0028*  | *0030*    | ***Pixel Spacing**: Physical distance in the patient between the center of each pixel.* | 像素间距*.*像素中心之间的物理间距                            | *DS* |
| *0028*  | *0100*    | ***Bits Allocated**: Number of bits allocated for each pixel sample. Each sample shall have the same number of bits allocated.* | 存储的位数：有*12*到*16*列举值*.存储的位数：有*12*到*16*列举值* | *US* |
| *0028*  | *0102*    | **Pixel Representation**:<br/>Data representation of the pixel samples. Each sample shall have the same pixel representation.<br/>Enum: 0000H=unsigned integer,0001H=2’s complement. | 像素数据的表现类型*:这是一个枚举值，分别为十六进制数*0000*和*0001.*0000H =* 无符号整数，*0001H = 2*的补码 | *US* |
| *0028*  | *1050*    | ***Window Center***                                          | 窗位                                                         | *DS* |
| *0028*  | *1051*    | ***Window Width***                                           | 窗宽                                                         | *DS* |
| *0028*  | *1052*    | **Rescale Intercept**:<br/>The value b in relationship between stored values (SV) and the output units.<br/>Output units = m*SV + b.<br/>Required if Modality LUT Sequence (0028, 0030) is not present. | 截距*:*如果表明不同模态的*LUT*颜色对应表不存在时*,*则使用方程.*Units = m\*SV + b,*计算真实的像素值到呈现像素值。其中这个值为表达式中的*b*。 | *DS* |
| *0028*  | *1053*    | ***Rescale Slope**: m in the equation specified by Rescale Intercept (0028,1052).Required if Rescale Intercept is present.* | 斜率*.*这个值为表达式中的*m*。                               | *DS* |
| *0028*  | *1054*    | **Rescale Type**:<br/><br/>Specifies the output units of Rescale Slope (0028,1053) and Rescale Intercept (0028,1052).<br/>Enum: US=Unspecified Requried if Photometric Interpretation is MONOCHROME2, and Bits Stored is greater than 1.<br/>This specifies an identity Modality LUT transformation. | 输出值的单位*.*这是一个枚举值                                | *LO* |

### DICOM判断3D图像的方向的TAG

* ImagePositionPatient (0020,0032)

  > specifies the x, y, and z coordinates of the upper left hand corner of the image. In other words, this tag specifies the coordinates of the the first voxel transmitted.

  * 图像位置：指示了图像左上角的第一个像素的空间坐标（x,y,z)。 也就是DICOM文件传输的第一个像素的坐标.

* ImageOrientationPatient (0020,0037)

  > specifies the direction cosines of the first row and the first column with respect to the patient. The direction of the axes are defined by the patients orientation to ensure LPS system ( x-axis increasing to the left hand side of the patient, y-axis increasing to the posterior side of the patient and z-axis increasing toward the head of the patient )

  * 图像方向：指示了图像第一行和第一列相对于病人的余弦方向。 坐标轴的方向是根据病人的方向来确定的（X轴指向病人的左手边，y轴指向病人的后面，Z轴指向病人的头部。

* PatientPosition( 0018,5100)

  > Patient position descriptor relative to the equipment. Required for CT and MR images. 
  >
  > Possible values: HFP= head first-prone, 
  >
  > ​							 HFS=head first-supine,
  >
  > ​							 HFDR= head first-decibitus right, 
  >
  > ​							 HFDL = head first-decubiturs left,
  >
  > ​							 FFP = feet first-prone, 
  >
  > ​							 FFS, FFDR, FFDL.
  
  * 病人的位置：  是描述病人相对于CT或者MR等成像设备的位置。 HFP：头部在前，俯卧； HFS：头在前，仰卧

### 相关框架

* [**FusionViewer**](http://fusionviewer.sourceforge.net/)
* [**Dcm4chee**](https://github.com/dcm4che/dcm4che)

### 国外开源的PACS服务器

* 名称：**Dcm4che**
  评级：★★★★★
  开源许可：GPL LGPL MPL
  功能： 影像处理，影像归档，影像管理，影像传输，Worklist支持
  标准：DICOM，HL7，IHE，MPPS，WADO
  语言：英语
  客户端： 桌面，基于web
  平台：跨平台
  编程语言：Java
  数据库：MySQL，Postgre SQL，Firebird
  官方网站：[http://www.dcm4che.org/](http://www.dcm4che.org/confluence/)
* 名称：**DCMTK-DICOM Toolkit**
  评级：★★★★★
  开源许可：BSD
  功能： 影像处理，影像归档，影像管理，影像传输
  标准：DICOM
  语言：英语
  客户端： 桌面
  平台：跨平台
  编程语言：C/C++
  官方网站：http://dicom.offis.de/
* 名称：**CDMEDIC PACS WEB**
  开源许可：GPL
  功能： 影像处理，影像归档，影像管理，影像传输
  标准：DICOM
  语言：英语，西班牙语
  客户端： 基于web
  平台：Unix，Mac OS
  官方网站：http://cdmedicpacsweb.sourceforge.net/
* 名称：**Conquest DICOM software**
  开源许可：Public Domain
  功能： 影像处理，影像归档，影像管理，影像传输，数据转换，Worklist支持
  标准：DICOM，HL7
  语言：英语
  客户端： 基于web
  平台：Unix，Windows，Mac OS
  官方网站：http://www.xs4all.nl/~ingenium/dicom.html
* 名称：**ClearCanvas**
  评级：★★★★★
  开源许可：BSD
  功能： 影像处理，影像归档，影像管理，影像传输，影像浏览
  标准：DICOM
  开发语言：C#
  客户端： 桌面
  平台：Windows
  官方网站：http://www.clearcanvas.ca/
* 名称：**mdcm**
  评级：★★★★
  开源许可：LGPL
  功能： 影像处理，影像归档，影像管理，影像传输，影像浏览
  标准：DICOM
  开发语言：C#
  客户端： 桌面
  平台：Windows
  官方网站：https://github.com/rcd/mdcm
* 名称：**OpenSourcePACS**
  开源许可：LGPL
  功能： 排队系统，影像处理，影像归档，影像管理，影像传输，影像浏览
  标准：DICOM
  语言：英语
  客户端： 桌面
  平台：跨平台
  官方网站：http://www.mii.ucla.edu/opensourcepacs/
* 名称：**OSPACS（Open Source Picture Archiving and Communication System）**
  开源许可：Cranfield Open-Source License
  功能： 影像处理，影像归档，影像管理，影像传输，影像浏览
  标准：DICOM
  语言：英语
  客户端： 桌面，基于web
  平台：Windows
  官方网站：http://www.ospacs.org/
* 名称：**Xebra**
  开源许可：GPL
  功能： 影像管理，影像传输，影像浏览，医疗顾问
  标准：DICOM，IHE，IHE-XDS-I
  语言：英语
  客户端： 桌面
  平台：跨平台
  编程语言：Java
  数据库：MySQL，Postgre SQL，Firebird
  官方网站：http://www.hxti.com/technology/xebra.html
