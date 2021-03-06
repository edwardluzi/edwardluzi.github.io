---
layout: post
title:  "OSI Model Brief on Physical Layer and Data Link Layer"
date:   2017-03-23 14:23:00 +0800
categories: network
---
Docker和Kubernetes都使用了网络虚拟化技术来实现容器、Pod之间的通信。网络虚拟化是将硬件和软件网络资源和网络功能组合到一个基于软件的管理实体（虚拟网络）中的过程。为了更好的理解这一虚拟化技术，本文及接下来的文章将分别阐述现实的网络技术及设备、虚拟化的网络技术及设备、以及Docker和Kubernetes采用的虚拟化技术。

电信网络（Telecommunications Network）
-------------------------------------
电信网络是终端节点和链路的集合，终端节点和链路被连接在一起以便终端节点之间进行通信。 节点使用电路交换、消息交换或分组交换，通过正确的链路和节点传递信号到达正确的目的地终端。

网络中的每个终端通常都有唯一的地址，因此消息或连接可以路由到正确的终端。网络中的地址集合称为地址空间。

常见的电信网络有`电话网络`和`计算机网络`等。

### 电话网络（Telephone Network）

电话网络是用于两方或多方之间的电话呼叫的电信网络，有多种不同类型：
* 固定线路网络，电话线必须直接连接到一个电话交换机，这被称为公共交换电话网络（Public Switched Telephone Network/PSTN）。
* 移动无线网络，电话是可移动的并且可以在覆盖区域内的任何地方移动。
* 私有网络/集团电话，一组封闭的主要用于内部通话的电话，并使用网关联通外界。这通常在公司和呼叫中心内部使用，被称为专用交换机（PBX）。
* 综合业务数字网（Integrated Services Digital Network/ISDN）

公共电话运营商（PTO）拥有并建立前两种网络，并由国家授权向公众提供服务。虚拟网络运营商（VNO）从PTO批发租赁能力，直接向公众销售电话服务。

### 计算机网络（Computer network）

计算机网络是允许节点共享资源的电信网络。在计算机网络中，地理位置不同的具有独立功能的多台计算机及其外部设备使用有线介质或无线介质建立节点之间的连接，使用数据链路彼此交换数据，在网络操作系统、网络管理软件及网络通信协议的管理和协调下，实现资源共享和信息传递。

开放系统互连参考模型（OSI Model）
--------------------------------
20世纪70年代末期，`国际标准化组织（ISO）`和`国际电报电话咨询委员会（CCITT）`分别制定了一个类似的网络模型的文件。1983年，这两个文件合并形成一个标准，称为开放系统互连参考模型，也称为OSI参考模型，或仅仅是OSI模型。该标准于1984年由ISO和CCITT（现称国际电信联盟电信标准化部门或ITU-T）发布，分别称为标准ISO7498和标准X.200。

OSI有两个主要组件，一个抽象的网络模型，称为基本参考模型或七层模型，以及一组特定协议。

OSI模型是一种概念模型，用以描述和标准化电信和计算系统的通信功能，并且不考虑其潜在的内部结构和技术。其目标是利用标准协议达到不同通信系统的互操作。该模型将通信系统分为多个抽象层，模型的原始版本定义了七层，由ISO / IEC 7498-1标识维护。

1. 物理层（Physical Layer），通过物理介质发送和接收原始比特流
2. 数据链路层（Data Link Layer），通过物理层连接的两个节点之间`可靠`传输数据帧
3. 网络层（Network Layer），构建和管理多节点网络，包括寻址，路由和流量控制等
4. 传输层（Transport Layer），在网络的各个节点之间可靠地传输数据包，包括分段，确认和复用（segmentation，acknowledgement and multiplexing）
5. 会话层（Session Layer），管理通信会话，即两个节点之间的连续的多次往返传输的信息交换 
6. 表示层（Presentation Layer），网络服务与应用之间的数据翻译; 包括字符编码，数据压缩和加密/解密等
7. 应用层（Application Layer），高级API，包括资源共享，远程文件访问等

在OSI模型中，每一层N对等实体间通过该层的协议交换协议数据单元（Protocol Data Units， PDU）。每个PDU包含称为服务数据单元（SDU）的有效载荷，以及协议相关的控制信息。

两个OSI兼容的通信设备的数据处理是这样完成的：
1． 数据在发送设备的最上层（层N）被组成协议数据单元（PDU）
2． PDU被传递到层N-1，这一层它被称为服务数据单元（SDU）
3． 在层N-1处，SDU与报头/报尾相连接，产生层N-1的PDU，然后将其传递到层N-2
4． 此过程持续进行，直到到达计算机网络最低层，数据从该最低层传送到接收设备
5． 在接收设备处，数据从最低层向最高层传送，每一层都从自己的PDU中剥离报头和报尾形成SDU，然后把SDU传递给上一次，直到最顶层


本文将着重介绍物理层和数据链路层的相关技术及设备。

物理层
------

物理层是OSI模型中最低的一层，该层定义了通过物理链路连接的网络节点传输原始比特（并非逻辑数据）的方法。比特流可能会被分组成编码或符号，并被转换成通过硬件传输介质发送的物理信号。物理层定义了数据连接的电气和物理规格，以及设备和物理传输介质（例如，铜缆或光纤光缆，射频）之间的关系。这包括针对有线设备的引脚，电压，线路阻抗，电缆规格，信号时序等特征以及针对无线设备的频率（2.4GHz或5GHz）等。简单的说，物理层确保原始的数据可在各种物理介质上传输。

物理层由网络的网络硬件传输技术组成，它是网络中高功能的逻辑数据结构的基础层。 由于具有广泛变化特性的大量可用硬件技术，这可能是OSI架构中最复杂的层。

### 常见的物理层协议
* `Bluetooth（蓝牙）`，是一种无线技术标准，可实现固定设备、移动设备和楼宇个人域网之间的短距离数据交换（使用2.4—2.485GHz的ISM波段的UHF无线电波）。蓝牙技术最初由电信巨头爱立信公司于1994年创制，当时是作为RS232数据线的替代方案。蓝牙可连接多个设备，克服了数据同步的难题。
* `DSL（Digital Subscriber Line，数字用户线路）`， 是通过铜线或者本地电话网提供数字连接的一种技术。DSL技术在传递公用电话网络的用户环路上支持对称和非对称传输模式，解决了经常发生在网络服务供应商和最终用户间的`最后一公里的传输瓶颈问题。DSL包括ADSL（非对称数字用户线）、RADSL（速率自适应数字用户线路）、HDSL（高速用户数字线）和VDSL（超高速用户数字线）等等。
* `EIA RS-232`，美国电子工业联盟（EIA）制定的串行数据通信的接口标准，原始编号全称是EIA-RS-232，被广泛用于计算机串行接口外设连接。EIA-RS-232规定了连接电缆和机械、电气特性、信号功能及发送过程。其他常用电气标准还有EIA-RS-422-A、EIA-RS-423A、EIA-RS-485。 
* `Ethernet（以太网）` 物理层，是计算机网络标准的以太网系列的物理层组件，其定义了设备与网络之间或网络设备之间的物理连接的电气或光学属性。以太网物理层已经演变了相当长的时间，其涵盖了相当多的物理介质接口（从同轴电缆到双绞线和光纤）和速度范围（从1 Mbit / s到100 Gbit / s）。 通常，网络协议栈软件可以类似地工作在所有物理层上。以太网是目前应用最普遍的局域网技术。
* `GSM （Global System for Mobile Communications，全球移动通讯系统），Um Air Interface` 物理层， GSM是由欧洲电信标准协会（ETSI）开发的标准，用于描述手机使用的第二代（2G）数字蜂窝网络的协议，首先在芬兰部署。Um空中接口是GSM移动电话标准的空中接口。 它是移动台（MS）和基站收发台（BTS）之间的接口。
* `G.hn/G.9960` 物理层， G.hn是一个家庭网络规范，数据速率高达1 Gbit / s，以电话线、同轴电缆和电源线为有线传输媒质，能够最大程度地利用已布设的各种线缆，在网络覆盖及终端接入层面上为物联网的普及提供了现实的实体支撑。
* `IEEE 1394`，IEEE 1394是用于高速通信和同步实时数据传输的串行总线的接口标准。 它是在20世纪80年代后期和20世纪90年代初由苹果公司开发的，它被称为火线（FireWire）。 火线一词为苹果电脑登记之商标，因此其他制造商在运用这项科技时，会采用不同的名称，比如i.LINK（索尼）和Lynx（德州仪器）。
* `ISDN（Integrated Services Digital Network，综合业务数字网）`，是一个数字电话网络国际标准，是一种典型的电路交换网络系统（circuit-switching network）。它通过普通的铜缆以更高的速率和质量传输语音和数据。ISDN是欧洲普及的电话网络形式。GSM移动电话标准也可以基于ISDN传输数据。
* `OTN（Optical Transport Network）`，ITU-T将光纤传输网络定义为能够提供承载客户信号的光信道的传输，复用，交换，管理，监控和生存能力功能的通过光纤链路连接的一组光网络元件。OTN是以波分复用技术为基础、在光层组织网络的传送网，是下一代的骨干传送网。
* `SONET（Synchronous Optical Networking，同步光纤网络）和SDH（Synchronous Digital Hierarchy，同步数字体系）`，SONET是使用光纤进行数字化信息通信的一个标准。为了传送大量的电话和数据业务就开发了它用以替换准同步数字体系（PDH）系统，它由Telcodia的GR-253-CORE定义。更先进的SDH标准建立在SONET发展的经验上。今天SONET和SDH两种技术体制都被广泛的应用；SONET应用在美国和加拿大，SDH应用在世界其他国家。
* `Telephone network modems（调制解调器） V.92`，调制解调器是一种网络硬件设备，发送端对数字信息编码然后调制成一个或多个载波信号进行传输，最后接收端经过解调载波信号然后解码得到原始的数字信息，其目的是产生一个可以容易传输和解码以再现原始数字信息的信号。调制解调器可以用于从发光二极管发射模拟信号到无线电的任何手段，一种常用的的调制解调器是将计算机的数字信息调制成电信号，然后通过`电话线`进行传输，并由接收端的另一调制解调器进行解调以恢复原始数字信息。
* `USB（Universal Serial Bus，通用串行总线）` 物理层，是连接计算机系统与外部设备的一种串口总线标准，也是一种输入输出接口的技术规范，被广泛地应用于个人电脑和移动设备等信息通讯产品，并扩展至摄影器材、数字电视（机顶盒）、游戏机等其它相关领域。
* `Varieties of 802.11， WI-FI`物理层，IEEE 802.11是现今无线局域网通用的标准，它是由国际电机电子工程学会（IEEE）所定义的无线网络通信的标准。Wi-Fi，是Wi-Fi联盟制造商的商标做为产品的品牌认证，是一个创建于IEEE 802.11标准的无线局域网技术。

### 非常见的物理层协议
* `1-Wire`，是由Dallas半导体公司设计的器件通信总线系统，可通过一条线路提供低速数据，信号和功率。
* `CAN（Controller Area Network，控制器局域网络）`，是ISO国际标准化的串行通信协议。在汽车产业中，出于对安全性、舒适性、方便性、低公害、低成本的要求，各种各样的电子控制系统被开发了出来。由于这些系统之间通信所用的数据类型及对可靠性的要求不尽相同，由多条总线构成的情况很多，线束的数量也随之增加。为适应`减少线束的数量`、`通过多个LAN，进行大量数据的高速通信`的需要，1986年德国电气商博世公司开发出面向汽车的CAN 通信协议。此后，CAN 通过ISO11898 及ISO11519 进行了标准化，在欧洲已是汽车网络的标准协议。
* `Etherloop`，是一种结合了以太网和DSL功能的DSL技术。它允许在标准电话线上组合语音和数据传输。 在正确的条件下，它允许在最多6.4公里（21,000英尺）的范围内以高达每秒6兆比特的速度传输。
* `I²C（Inter-Integrated Circuit）， I²S（Inter-IC Sound）`，I²C ，是由飞利浦半导体（现在的恩智浦半导体）发明的多主多从机，分组交换，单端，串行的计算机总线，它通常用于将低速外围IC连接主板、嵌入式系统或手机等设备。I²S是IC间传输数字音频数据的一种接口标准，采用序列的方式传输2组（左右声道）数据。I²S常被使用在发送CD的PCM音频数据到CD播放器的DAC中。由于I²S将数据信号和时钟频率信号分开发送，它的抖动（jitter）有损十分地小。 
* `IrDA` 物理层， IrDA提供了一套完整的无线红外通信协议规范，使用IrDA的主要原因是使用点对点原理在最后一米的距离上进行无线数据传输，因此，它广泛应用于诸如移动电话，膝上型计算机，照相机，打印机和医疗设备等便携式设备中。
* `SPI（Serial Peripheral Interface Bus，串行外设接口）`，是一种用于短程通信的同步串行通信接口规范，主要应用于单片机系统中。类似I²C。 这种接口首先被Motorola（摩托罗拉）公司开发，然后发展成了一种行业规范。典型应用包含SD卡和液晶显示器。 SPI设备之间使用全双工模式通信，包含一个主机和一个或多个从机。主机产生待读或待写的帧数据，多个从机通过一个片选线路 决定哪个来响应主机的请求。 有时SPI接口被称作四线程接口，SPI准确来讲称为同步串行接口，但是与同步串行接口协议（SSI）不同，SSI是一个四线程 同步通信协议，但是使用差分信号输入同时仅提供一个单工通信信道。
* `SMB（System Management Bus，系统管理总线）`，是一种两条讯号所组成源自于I²C的一种总线，其设计应用于轻量级的通讯。最常于主板的电源开关指令的通讯中发现其存在（例如笔记型电脑中，重复充电的子系统），其他的元件，例如温度、风扇或电压的感测器的通讯中也可以看到其踪影。
* `T-carrier links（T载波），和 E-carrier links（E载波）`，T载波（T1、T3），是贝尔实验室于1960年代所研发，为了在数位传输线上传送语音讯号所发展的多工传送方式，其资料传送速度分别是1.544Mbps，44.736Mbps，Fractional T1是指将T1的频宽加以分割，提供给频宽需求低于T1速度的使用者。E载波（E1、E3等）类似于北美T载波，是提供复用、多信道、点对点通信链路的通信系统和标准。通信公司使用它来链接远程电话交换局并普遍用于用户到通信公司的接入链路。
* `TransferJet` 物理层，  一种近距离无线传输技术标准，由索尼公司研发，于2008年发表。TransferJet技术，类似于近场通讯技术，能让两个贴近的电子装置，以点对点方式，高速交换资料。它的传输率可以达到375Mbps，主要运作于560MHz频带中，采用4.48GHz频道。它与其他近场通讯技术的主要不同，在于它采取电感磁场原理，而不是无线电频率的技术来实做，这让它不会受到其他无线技术的干扰或退化现象。
* `X10`，用于家庭自动化的电子设备之间的通信协议。 它主要使用电力线路来进行信令和控制，其中信号涉及表示数字信息的短暂的射频突发。

数据链路层
----------
数据链路层是OSI模型的第二层，提供节点到节点之间的数据传输 - 两个直接连接的节点之间的链路。它检测并可能纠正物理层中可能发生的错误，它还定义了建立和终止两个物理连接设备之间的连接的协议，以及流量控制协议。

在OSI网络架构的语义中，数据链路层协议响应来自网络层的服务请求，并通过向物理层发出服务请求来执行其功能。

### 帧

帧是计算机网络和电信网络中的数字数据传输单元。一个帧通常包括帧同步特征，该特征由一系列比特或符号组成，这些比特或符号向接收者指示其接收的比特或符号流中的有效载荷数据的开始和结束。如果一个接收者在帧传输的过程中连接到系统，则它会忽略数据，直到检测到一个新的帧同步序列。

在OSI模型中，帧是数据链路层的协议数据单元（PDU）。帧是通过物理层传输数据之前最后一层封装的结果。帧是数据链路层协议中的传输单元，由一个数据链路层报头，后跟一个数据包组成。帧与帧的发送需要间隔一段时间，这个间隔时间称作帧间距（Interframe Gap）。

帧的报头包含源和目标地址，用以指示哪个设备发起帧，哪个设备预期接收并处理它。 与网络层的分层和可路由地址相反，第2层地址是平坦的，这意味着地址的任何部分都不能用于标识地址所属的逻辑或物理组。

因此，数据链路提供了跨物理链路的数据传输。该数据传输可以是可靠的，也可以是不可靠的，许多数据链路协议都没有接收或接受成功确认，一些数据链路协议甚至可能没有任何形式的校验和来检查传输错误。 在这些情况下，高级协议必须提供流量控制，错误检查，确认和重传等机制。

### 子层

IEEE 802是IEEE标准中关于局域网和城域网的一系列标准。更确切的说，IEEE 802标准仅限定在传输可变大小数据包的网络。其中最广泛使用的有以太网、令牌环、无线局域网等。

在诸如IEEE 802局域网的一些网络中，由于可能存在介质争用，它还可以细分成介质访问控制（MAC）子层和逻辑链路控制（LLC）子层，介质访问控制（MAC）子层专职处理介质访问的争用与冲突问题。同时这意味着IEEE 802.2 LLC协议可以与所有IEEE 802 MAC层一起使用，例如以太网，令牌环，IEEE 802.11等，以及一些非802 MAC层，例如FDDI。

在ITU-T G.hn标准中，提供了使用现有家庭布线（电力线，电话线和同轴电缆）创建高速（高达1吉比特）局域网的方式，数据链路层分为三个子层（应用协议融合，逻辑链路控制和介质访问控制）。

### 介质访问控制（MAC）子层

MAC子层提供定址和信道访问控制机制，使得不同设备或网络上的节点可以在多点网络上通信，而不会互相冲突。MAC子层在多点网络中模拟全双工逻辑通信信道，该信道可以提供单播，多播或广播通信服务。

#### 定址机制

IEEE 802网络和FDDI网络中使用的本地网络地址称为介质访问控制（MAC）地址；它们基于早期以太网实现中使用的定址方案。 MAC地址是唯一的，由网络设备制造商在生产时写在硬件内部。MAC地址一般采用6字节（48比特）来表示，其中前24位是由生产网卡的厂商向IEEE申请的厂商地址，后24位由厂商自行分配，这样的分配使得世界上任意一个拥有48位MAC地址的网卡都有唯一的标识。这使得帧可以在通过中继器，集线器，网桥和交换机的组合而不是由网络层路由器的某些组合来互连主机的网络链路上传送。

以太网和Wi-Fi网络都是IEEE 802局域网并且使用IEEE 802 48位MAC地址。

在全双工点对点通信中不需要MAC层，但由于兼容性原因，地址字段仍然包含在某些点对点协议中。

#### 信道访问控制机制

由MAC层提供的信道访问控制机制也称为多路访问协议，因为MAC提供配合特定通道访问（Channel Access Method）需要的协议及控制机制。因此连接在同一传输介质的几个设备可以共享其介质。共享物理介质的例子总线网、环状网、HUB网络、无线网络及半双工的链接。如果使用基于分组模式争用的信道访问方法，则多址协议可以检测或避免数据分组冲突，如果使用基于电路交换或信道化的信道访问方法，则多址协议可以预留资源以建立逻辑信道。信道访问控制机制依赖于物理层复用方案。

最广泛使用的多路访问协议是以太网中使用的基于争用的CSMA/CD协议。该机制仅在网络冲突域内使用，例如以太网总线网络或基于集线器的星形拓扑网络。以太网可以分为几个冲突域，由网桥和交换机互连。

交换式全双工网络，比如当今的交换式以太网，不需要多路访问协议，但出于兼容性原因，这些网络设备通常支持多路访问协议。

#### 常见有线网络多路访问协议:

- CSMA/CD (Ethernet and IEEE 802.3)
- Token Bus (IEEE 802.4)
- Token Ring (IEEE 802.5)
- Token Passing (FDDI)

#### 常见无线网络多路访问协议:

- CSMA/CA (IEEE 802.11/WiFi WLANs)
- Slotted ALOHA
- Dynamic TDMA
- Reservation ALOHA (R-ALOHA)
- Mobile Slotted Aloha (MS-ALOHA)
- CDMA
- OFDMA
- OFDM

#### 蜂窝网络中的MAC

GSM，UMTS或LTE等蜂窝网络也使用MAC层，蜂窝网络中的MAC协议被设计为最大限度地利用昂贵的许可频谱。蜂窝网络的空中接口对应于OSI模型的物理层和数据链路层，数据链路层还进一步分成多个协议层。 在UMTS和LTE中，数据链路层被分为分组数据汇聚协议（Packet Data Convergence Protocol，PDCP），无线链路控制（Radio Link Control，RLC）协议和MAC协议。 基站具有对空中接口的绝对控制，并调度下行接入以及所有设备的上行接入。 3GPP在TS 25.321和TS 36.321中分别为UMTS和LTE指定MAC协议。

### 逻辑链路控制（LLC）子层

LLC子层是局域网中数据链路层的上层部分，IEEE 802.2中定义了逻辑链路控制协议。

LLC子层提供多路复用机制，使得多个网络协议（IP，IPX，Decnet和Appletalk）可以在多点网络内共存并通过相同的网络介质传输。 它还可以提供流量控制和自动重复请求（ARQ）错误管理机制。与MAC层不同，LLC和物理介质全无关系。

LLC使用依赖于特定传输介质（以太网，令牌环，FDDI，802.11）的介质访问控制（MAC）服务。对于除以太网之外的所有IEEE 802网络，使用LLC是强制性的。它也用于不属于IEEE 802系列的光纤分布式数据接口（FDDI）。

### 常见的数据链路层协议
* `ARCNET（Attached Resource Computer Network）`，是局域网的通信协议。ARCNET是第一个广泛使用的微型计算机网络系统，在20世纪80年代因办公自动化而流行，后来被应用于嵌入式系统。

  ARCNET是1977年由DataPoint公司开发的一种安装广泛的局域网（LAN）技术，它采用令牌总线（Token-Bus）方案来管理LAN上工作站和其他设备之间的共享线路，其中，LAN服务器总是在一条总线上连续循环的发送一个空信息帧。

  当有设备要发送报文时，它就在空帧中插入一个`令牌`以及相应的报文。当目标设备或LAN服务器接收到该报文后，就将`令牌`重新设置为0，以便该帧可被其他设备重复使用。这种方案是十分有效的，特别是在网络负荷大的时候，它为网络中的各个设备提供平等使用网络资源的机会。

  ARCNET可采用同轴电缆或光缆线。ARCNET为4项主要的LAN技术之一，其他三项为Ethernet，Token Ring和FDDI。

* ` ATM（Asynchronous transfer mode，异步传输模式）`，根据ATM论坛，`由ANSI和ITU（前CCITT）定义的用于运送完整范围的用户流量（包括语音，数据和视频信号）的一个电信概念`。ATM的开发是为了满足上世纪80年代末制定的宽带综合业务数字网络（B-ISDN）的需求，旨在统一电信和计算机网络。 它被设计用于处理传统的高吞吐量数据流量（例如文件传输）的网络以及诸如语音和视频的实时低延迟内容。

  ATM适用于局域网和广域网。
  
  ATM的参考模型近似映射到ISO-OSI参考模型的三个最低层：网络层、数据链路层和物理层。ATM是公共交换电话网（PSTN）和综合业务数字网（ISDN）在SONET/SDH骨干网上使用的核心协议。

  ATM提供类似于电路交换和分组交换网络的功能：ATM使用异步时分复用技术将数据编码为称为信元（Cell）的小型固定长度的分组（ISO-OSI帧）。每个信元长53字节。其中报头占了5字节，这与因特网协议或以太网的使用可变大小的分组和帧的的方法不同。

  ATM使用面向连接的模型，在实际数据交换开始之前必须在两个端点之间建立虚拟电路。这些虚拟电路可以是`永久的`，即通常由服务提供商预先配置的专用连接，或`可切换的`，即，使用信令在每个呼叫的基础上设置一个虚拟电路，并且在呼叫终止时被断开。

  
* `Ethernet（以太网）`是一种局域网技术。IEEE组织的IEEE 802.3标准制定了以太网的技术标准，它规定了包括物理层的连线、电子信号和介质访问层协议的内容。自此之后以太网被改进以支持更高的比特率和更长的链路距离。随着时间的推移，以太网已经在很大程度上取代了诸如令牌环，FDDI和ARCNET等有线局域网技术。

  最初的以太网是采用同轴电缆来连接各个设备的。计算机通过一个叫做附加单元接口（Attachment Unit Interface，AUI）的收发器连接到电缆上。一条简单网路线对于一个小型网络来说很可靠，而对于大型网络来说，某处线路的故障或某个连接器的故障，都会造成以太网某个或多个网段的不稳定。
 
  因为所有的通信信号都在共用线路上传输，即使信息只是想发给其中的一个目的地，却会使用广播的形式，发送给线路上的所有计算机。在正常情况下，网络接口卡会滤掉不是发送给自己的信息，接收到目标地址是自己的信息时才会向CPU发出中断请求。共享电缆也意味着共享带宽，所以在某些情况下以太网的速度可能会非常慢，比如电源故障之后，当所有的网络终端都重新启动时。
 
  带冲突检测的载波侦听多路访问（CSMA/CD）技术规定了多台计算机共享一个通道的方法。这项技术最早出现在1960年代由夏威夷大学开发的ALOHAnet，它使用无线电波为载体。这个方法要比令牌环网或者主控制网简单。
 
  大多数现代以太网用以太网交换机（Switch）代替集线器（Hub）。尽管布线方式和集线器相同，但交换式以太网比共享介质以太网有很多明显的优势，例如更大的带宽和更好的异常结果隔离设备。交换网络典型的使用星型拓扑，虽然设备在半双工模式下运作时仍是共享介质的多节点网，但10BASE-T和以后的标准皆为全双工以太网，不再是共享介质系统。

  以太网可以采用多种连接介质，包括同轴缆、双绞线和光纤等。其中双绞线多用于从主机到集线器或交换机的连接，而光纤则主要用于交换机间的级联和交换机到路由器间的点到点链路上。同轴缆作为早期的主要连接介质已经逐渐趋于淘汰。

  新的万兆以太网标准（IEEE 802.3ae）包含7种不同类型，分别适用于局域网、城域网和广域网。
  
* `FDDI（Fiber Distributed Data Interface、 光纤分布式数据接口）`，是一个局域网数据传输标准。它使用光纤作为其标准的底层物理介质，数据传输速率可达100Mbit/s，最远传送距离200公里。采用五类双绞线作为传输介质的FDDI，称为CDDI。

  虽然FDDI的逻辑拓扑是一个基于环的令牌网络，但它没有使用IEEE 802.5令牌环协议作为其基础；相反，其协议来自IEEE 802.4令牌总线定时令牌协议。除了覆盖大的地理区域外，FDDI局域网可以支持数千个用户。FDDI提供双连接站（DAS），反向旋转令牌环形拓扑和单一连接站（SAS），令牌总线通过环形拓扑。

  由光纤构成的FDDI，其基本结构为逆向双环。一个环为主环，另一个环为备用环。一个顺时针传送信息，另一个逆时针。当主环上的设备失效或光缆发生故障时，通过从主环向备用环的切换可继续维持FDDI的正常工作。 主环提供高达100 Mbit/s的容量。当网络不需要辅助环做备份时，辅助环也可以承载数据，扩展容量为200Mbit/s。FDDI具有比标准以太网系列更大的帧大小，其大小为4352字节，标准以太网仅支持最大帧大小为1500字节，因此在某些情况下 FDDI允许更好的有效数据速率。

* `Frame Relay`，帧中继是一种标准化的广域网技术，它使用分组交换方法来指定数字电信信道的物理和数据链路层。最初设计用于跨综合业务数字网（ISDN）基础设施的传输。
  
  帧中继是一种先进的包交换技术，是一种快速分组通信方式。它采用虚电路技术，能充分利用网络资源。帧中继为多区域间，全国范围内以及国际间实现通信提供了一个灵活高效的广域网解决方案。

  帧中继的设计者旨在为局域网（LAN）之间和广域网（WAN）中端点之间的间歇性业务提供成本效益数据传输的电信服务。 帧中继将数据放在称为`帧`的可变大小单元中，并把任何必要的纠错措施（如数据重传）留给端点处理。 这加快了整体数据传输速度。 对于大多数服务，网络提供永久的虚拟电路（PVC），这意味着客户可以看到连续的专用连接，而无需支付全时租用线路的费用，同时服务提供商计算每帧到达目的地而行进的路线，可以根据使用情况收费。
  
  帧中继的技术基础是旧的X.25分组交换技术，X.25用于在模拟语音线路上传输数据。
但是帧中继的设计思想与x.25有明显差别：X。25实际上是为不稳定连接的运行而开发的，X。25强调数据传输的高可靠性；而帧中继是因为光纤技术发展后，差错控制显得不太必要，帧中继则着重于数据的快速传输，最大程度地提高网络吞吐量。当帧中继网络检测到帧中的错误时，它会简单地丢弃该帧。端点负责检测和重传丢帧、然而，数字网络相对于模拟网络的误差发生率非常小。
  
  帧中继通常通过租用T-1线路用于将局域网（LAN）与主要骨干网以及公共广域网（WAN）以及专用网络环境连接。在传输期间需要专门的连接。帧中继不能为语音或视频传输提供理想的路径，这两者都需要稳定的传输。然而，在某些情况下，语音和视频传输确实使用帧中继。
  
  帧中继正逐渐被`ATM`、`IP`等协议（包括IP虚拟专用网）替代。

* `HDLC（High-Level Data Link Control，高级数据链路控制）`，是一组用于在网络结点间传送数据的协议，是由国际标准化组织（ISO）颁布的一种高可靠性、高效率的数据链路控制规程，其特点是各项数据和控制信息都以比特为单位，采用`帧`的格式传输。

  作为面向比特的数据链路控制协议的典型，HDLC具有如下特点：协议不依赖于任何一种字符编码集；数据报文可透明传输，用于实现透明传输的`0比特插入法`易于硬件实现；全双工通信，不必等待确认便可连续发送数据，有较高的数据链路传输效率；所有帧均采用CRC校验，对信息帧进行编号，可防止漏收或重份，传输可靠性高；传输控制功能与处理功能分离，具有较大灵活性和较完善的控制功能。由于以上特点，使得网络设计普遍使用HDLC作为数据链路管制协议。

  国际电信联盟已把HDLC规程引入到X.25协议栈。现在HDLC作为同步点对点协议（PPP）的基础已经被用于很多服务中来接入广域网。

* `IEEE 802.11和WI-FI`，是用于在900MHz和2.4GHz、3.6GHz、5GHz和60GHz频带中实现无线局域网（WLAN）计算机通信的一组介质访问控制（MAC）和物理层规范。

  802．11协议家族由一系列使用相同基本协议的半双工空中调制技术组成。 802。11-1997是家族中第一个无线网络标准，物理层定义了工作在2。4GHz的ISM频段上的两种扩频作调制方式和一种红外线传输的方式，总数据传输速率设计为2Mbit/s。两个设备可以自行构建临时网络，也可以在基站（Base Station/BS）或者接入点（Access Point/AP）的协调下通信。为了在不同的通讯环境下获取良好的通讯质量，采用CSMA/CA（Carrier Sense Multiple Access/Collision Avoidance）硬件沟通方式。

  802．11b是第一个被广泛接受的无线网络标准，数据传输速率高达11Mbit/s， 其次是802.11a，802.11g，802.11n和802.11ac。 家族中的其他标准（c-f，h，j）是用于扩展现有标准的现有范围的服务修订，其中还可能包括对先前规范的更正。

  Wi-Fi是Wi-Fi联盟制造商的商标做为产品的品牌认证，是一个创建于IEEE 802.11标准的无线局域网技术。并不是每样匹配IEEE 802.11的产品都申请Wi-Fi联盟的认证，相对地缺少Wi-Fi认证的产品并不一定意味着不兼容Wi-Fi设备。


* `OpenFlow`，一种网络通信协议，属于OSI模型的数据链路层，能够控制网络交换机或路由器的转送平面（forwarding plane），借此改变网络数据包所经过的网络路径。

  OpenFlow将原来完全由交换机/路由器控制的报文转发过程转化为由OpenFlow交换机（OpenFlow Switch）和控制服务器（Controller）来共同完成，在OpenFlow交换机上实现数据转发，而在控制器上实现数据的转发控制，从而实现了数据转发和路由控制的分离。控制器可以通过事先规定好的接口操作来控制OpenFlow交换机中的流表，从而达到控制数据转发的目的。更为重要的是，在内部网络和外网的连接处应用OpenFlow交换机可以通过更改数据流的路径以及拒绝某些数据流来增强企业内网的安全性。

  OpenFlow协议的发明人认为OpenFlow是软件定义网络（SDN）的推动者。在SDN中，交换设备的数据转发层和控制层是分离的，因此网络协议和交换策略的升级只需要改动控制层。基于OpenFlow实现SDN，则在网络中实现了软硬件的分离以及底层硬件的虚拟化，从而为网络的发展提供了一个良好的发展平台。

  OpenFlow协议标准由开放网络基金会（ONF，成立于2011年）负责管理。

* `PPP（Point-to-Point Protocol，点对点协议）`，工作在OSI参考模型的数据链路层。它通常用在两节点间创建直接的连接，并可以提供连接认证、传输加密（使用ECP，RFC 1968）以及压缩。

  PPP被用在许多类型的物理网络中，包括串口线、电话线、中继线（Trunk Line）、移动电话、特殊无线电链路以及光纤链路（如SONET）。

  PPP还用在互联网接入连接上。互联网服务提供商（ISP）使用PPP为用户提供到Internet的拨号接入，这是因为IP报文无法在没有数据链路协议的情况下通过调制解调器线路自行传输。

  PPP的两个派生物PPPoE（ Point-to-Point Protocol over Ethernet）和PPPoA（Point-to-Point Protocol over ATM）被ISP广泛用来与用户创建数字用户线路（DSL）Internet服务连接。

  PPP被广泛用作连接同步和异步电路的数据链路层协议，替换了陈旧的串行线路IP协议（SLIP）以及电话公司的拥有的标准（如 X。25协议族中的LAPB）。PPP被设计用来与许多网络层协议协同工作，包括网际协议（IP）、TRILL、Novell的互联网分组交换协议（IPX）、NBF以及AppleTalk。

  
* `Token Ring（令牌环）`，是定义在IEEE 802.5标准中的一种局域网接入方式，是一个OSI模型中的数据链路层协议。

  令牌环网络的基本原理是利用令牌（代表发信号的许可）来避免网络中的冲突，与使用冲突检测算法CSMA/CD的以太网相比，令牌环网的数据传送率更高，此外，令牌环网还可以设置发送的优先度。一个4M的令牌环网络和一个10M的以太网数据传送率相当，一个16M的令牌环网络的数据传送率接近一个100M的以太网。

  令牌环网络的缺点是网络不可复用，从而导致网络利用率低下。当网络中一个结点拿到令牌使用网络后，不管此结点使用多少带宽，其它结点必须等待其使用完网络并放弃令牌后才有机会申请令牌并使用网络。此外，网络中还需要专门结点维护令牌。

  令牌环也暗示了除了使用令牌外，这还是一个环形的网络拓扑。

### 非常见的数据链路层协议
* `1-Wire`，详见`非常见的物理层协议`。

* `CAN（Controller Area Network，控制器局域网络）`，是一种车辆总线标准，旨在允许微控制器和设备在没有主机的应用中相互通信。 它是一种基于消息的协议，最初用于汽车中的多路电线，但也被用于许多其他情况。

  1993年，国际标准化组织（ISO）发布了CAN标准ISO 11898，该标准后来被重组为两部分：涵盖数据链路层的ISO 11898-1和覆盖高速CAN物理层的ISO 11898-2以及涵盖了用于低速，容错CAN物理层的 ISO 11898-3。

* `CDP（Cisco Discovery Protocol）`，是Cisco Systems开发的专有数据链路层协议。它能够运行在大部分的思科设备上面。通过运行CDP 协议，思科设备能够在与它们直连的设备之间分享有关操作系统软件版本，以及IP地址，硬件平台等相关信息。

  思科设备默认每60秒向01-00-0C-CC-CC-CC这个组播地址发送一次通告，如果在180秒内未获得先前邻居设备的CDP通告，它将清除原来收到的CDP信息。因为它不依赖任何的三层协议，透过CDP协议，可以解决一些三层错误配置的故障，比如错误的三层地址等等。

* `Econet`，是Acorn Computers的低成本局域网系统，供学校和小企业使用。Econet是一个五线总线网络。 一对线用于时钟，一对线用于数据，一根线用作公共接地。 信令使用RS-422 5伏差分标准，每个时钟周期传输一个比特位。非屏蔽电缆用于短距离，屏蔽电缆用于较长的网络。

  Acorn Computers Ltd是1978年在英国剑桥成立的一家电脑公司，有时被称为英国的苹果公司。2010年，该公司被ZDNet大卫•迈耶（David Meyer）列为十大死亡的IT巨人中的第九名。

* `I²C`，详见`非常见的物理层协议`。

* `LattisNet`是在20世纪80年代由Synoptics Communications建立和销售的计算机网络硬件和软件产品系列，比如1000，2500和3000系列的LattisHub网络中心。 LattisNet是第一个利用非屏蔽双绞线和星型拓扑结构实施的高达10兆/秒的局域网。
  
* `IEEE 802.1aq`，该标准规定的最短路径桥接（SPB）是一种计算机网络技术，旨在简化网络的创建和配置，同时实现多路径路由。它是旧的生成树协议（IEEE 802.1D，IEEE 802.1w，IEEE 802.1s）的替代品。

  该标准目前在全球的企业、运营商以及云计算服务中使用，这是一个即插即用的标准，其设计目的是为了避免网络配置中的一些人为错误。各个节点间的连接进行了优化、更有逻辑、易于维护。

* `LLDP（ Link Layer Discovery Protocol、链路层发现协议）`，是供应商无关的链路层协议，网络设备可以通过在本地网络中发送LLPDU（Link Layer Discovery Protocol Data Unit）来通告其他设备自身的状态。是一种能够使网络中的设备互相发现并通告状态、交互信息的协议。

  2005年5月，此协议已经被认可为IEEE802.1AB-2005标准。它代替了供应商私有的协议，如Cisco公司的思科发现协议（Cisco Discovery Protocol）、Extreme Networks的EDP协议（Extreme Discovery Protocol）、Enterasys Networks的CDP协议（Cabletron Discovery Protocol）以及Nortel Networks的NDP协议（Nortel Discovery Protocol）等。

* `NDP（Nortel Discovery Protocol）`、是用于发现Nortel网络设备和Avaya和Ciena的某些产品的数据链路层网络协议。 

* `STP（Spanning Tree Protocol，扩展树协议）`，是一个基于OSI模型的数据链路层通信协议，用作确保一个无环路的局域网环境。

  STP让一个网络被设计成包含备用（重复的）连接以当一条运作中的线路失效时，自动提供备用路径，并排除引起桥接器环路、及手动引导、关闭该些备用连接的需要。因此，通过使用STP，可以达到四个效果：1. 防止环路；2. 防止MAC地址震荡；3. 防止重复帧的出现；4. 防止广播风暴的出现。 

  STP的基本原理是，通过在交换机之间传递一种特殊的协议报文，网桥协议数据单元（Bridge Protocol Data Unit，简称BPDU），来确定网络的拓扑结构。BPDU有两种，配置BPDU（Configuration BPDU）和TCN BPDU。前者是用于计算无环的生成树的，后者则是用于在二层网络拓扑发生变化时产生用来缩短MAC表项的刷新时间的（由默认的300s缩短为15s）。

  

常见网络设备
------------

### NIC（Network Interface Controller，网络接口控制器）

网络接口控制器，也叫网络适配器（Network Adapter）、网卡（Network Interface Card）或局域网适配器（LAN Adapter），是一块被设计用来允许计算机在计算机网络上进行通讯的计算机硬件。

NIC使用一个特定的物理层和数据链路层标准，例如令牌环、以太网、光纤通道或Wi-Fi来实现通讯所需要的电路系统。这为一个完整的网络协议栈提供了基础，使得在同一局域网中的小型计算机之间进行通信，并通过诸如IP协议等可路由协议进行大规模的网络通信。

NIC是`物理层`和`数据链路层`设备，因为它提供对网络介质的物理访问，特别是对于IEEE 802规范所定义的局域网和类似网络，NIC通过MAC地址来提供定址功能。

### Repeater（中继器）

中继器是工作在`物理层`上的连接设备。适用于完全相同的两类网络的互连，主要功能是通过对数据信号的重新发送或者转发，来扩大网络传输的距离。中继器是对信号进行再生和还原的网络设备。

### Hub（集线器）

集线器（Hub）是指将多条以太网双绞线或光纤集合连接在同一段物理介质下的设备。集线器是运作在OSI模型中的`物理层`。它可以视作多端口的中继器，若它侦测到碰撞，它会提交阻塞信号。

由于集线器会把收到的任何数字信号，经过再生或放大，再从集线器的所有端口提交，这会造成信号之间碰撞的机会很大，而且信号也可能被窃听，并且这代表所有连到集线器的设备，都是属于同一个碰撞网域以及广播网域，因此大部分集线器已被交换机替换。

### Modem（调制解调器）

调制解调器是一个将数字信号调制(modulation)到模拟信号上进行传输，并解调(demodulation)收到的模拟信号以得到数字信号的网络设备。它的目标是产生能够方便传输的模拟信号并且能够通过解码还原原来的数字信号。根据不同的应用场合，调制解调器可以使用不同的手段来传送模拟信号，比如使用光纤，射频无线电或电话线等。

使用普通电话线音频波段进行数据通信的电话调制解调器是人们最常接触到的调制解调器。

### Fiber Media Converter（光纤介质转换器） 

光纤介质转换器是一种简单的网络设备，可以连接两种不同类型的介质，如双绞线和光纤。
  
### Network Bridge（桥接器）

桥接器，又称网桥，一种网络设备，负责网络桥接（network bridging）之用。桥接器将网络的多个网段在数据链路层连接起来（即桥接）。

网桥在功能上与集线器等其他用于连接网段的设备类似，不过网桥工作在`数据链路层`，而集线器工作在物理层。

- 网桥能够识别数据链路层中的数据帧，并将这些数据帧临时存储于内存，再重新生成信号作为一个全新的数据帧转发给相连的另一个网段（network segment）。由于能够对数据帧拆包、暂存、重新打包（称为存储转发机制 store-and-forward），网桥能够连接不同技术参数传输速率的数据链路。

- 数据帧中有一个位叫做FCS，用来通过CRC方式校验数据帧中的位。网桥可以检查FCS，将那些损坏的数据帧丢弃。

- 网桥在向其他网段转发数据帧时会做冲突检测控制。

- 网桥还能通过地址自学机制和过滤功能控制网络流量，具有OSI模型数据链路层网络交换机功能。

- 网桥仅仅在不同网络之间有数据传输的时候才将数据转发到其他网络，不是像集线器那样对所有数据都进行广播。对于以太网，`桥接`这一术语正式的含义是指匹配IEEE 802.1D标准的设备，即`网络切换`。网桥可以分区网段，不似集线器仍是在为同一碰撞域，因此网桥可隔离碰撞。

### Network Switch（网络交换机）

网络交换机，是一个扩大网络的器材，能为子网络中提供更多的连接端口，以便连接更多的计算机。

按照OSI模型，交换机又可以分为第二层交换机、第三层交换机、第四层交换机等，一直到第七层交换机。

二层交换机工作于OSI模型的第二层，即`数据链路层`。交换机内部的CPU会在每个端口成功连接时，通过将MAC地址和端口对应，形成一张MAC表。在今后的通讯中，发往该MAC地址的数据包将仅送往其对应的端口，而不是所有的端口。因此，交换机可用于划分数据链路层广播，即冲突域；但它不能划分网络层广播，即广播域。

三层交换机则可以处理第三层`网络层`协议，用于连接不同网段，通过对缺省网关的查询学习来创建两个网段之间的直接连接。三层交换机具有一定的`路由`功能，但只能用于同一类型的局域网子网之间的互连。这样，三层交换机可以像二层交换机那样通过MAC地址标识数据包，也可以像传统路由器那样在两个局域网子网之间进行功能较弱的路由转发，它的路由转发不是通过软件来维护的路由表，而是通过专用的ASIC芯片处理这些转发；

四层交换机可以处理第四层`传输层`协议，可以将会话与一个具体的IP地址绑定，以实现虚拟IP。

七层交换器是更加智能的交换器，可以充分利用频宽资源来过滤，识别和处理应用层数据转换的交换设备。

### Router（路由器）

路由器是一种电讯网络设备，提供路由与转送两种重要机制，可以决定数据包从来源端到目的端所经过的路由路径（host到host之间的传输路径），这个过程称为路由；将路由器输入端的数据包移送至适当的路由器输出端（在路由器内部进行），这称为转送。路由工作在OSI模型的`第三层，即网络层`，例如网际协议（IP）。

基于地理位置的网络分类
--------

### NFC（Near Field Communication，近场通信）

NFC、又称近距离无线通信，是一种短距离的高频无线通信技术，允许电子设备之间进行非接触式点对点数据传输，在十厘米（3.9英寸）内交换数据。

这个技术由非接触式射频识别（RFID）演变而来，由飞利浦半导体（现恩智浦半导体）、诺基亚和索尼共同研制开发，其基础是RFID及互连技术。近场通信是一种短距高频的无线电技术，在13.56MHz频率运行于20厘米距离内。其传输速度有106 Kbit/秒、212 Kbit/秒或者424 Kbit/秒三种。目前近场通信已通过成为ISO/IEC IS 18092国际标准、EMCA-340标准与ETSI TS 102 190标准。NFC采用主动和被动两种读取模式。

### PAN（Personal Area Network，个人局域网）

指个人范围（随身携带或数米之内）的计算设备（如计算机、电话、PDA、数码相机等）组成的通信网络。个人网即可用于这些设备之间互相交换数据，也可以用于连接到高层网络或互联网。

个人网可以是有线的形式，例如USB或者Firewire（IEEE1394）总线；也可以是无线的形式，例如红外（IrDA）或蓝牙。采用无线技术的个人局域网，称为无线个人局域网（wireless personal area network，缩写为WPAN）

### LAN（Local Area Network，局域网）

局域网是一种计算机网络，可连接住宅，学校，实验室，大学校园或办公大楼等有限区域内的计算机，并将其网络设备和互联网进行本地管理。相比之下，广域网（WAN）不仅覆盖较大的地理距离，而且还通常涉及租用电信电路或互联网链路。 

IEEE推动了局域网技术的标准化，由此产生了IEEE 802系列标准。这使得在建设局域网时可以选用不同厂家的设备，并能保证其兼容性。这一系列标准覆盖了双绞线、同轴电缆、光纤和无线等多种传输介质和组网方式，并包括网络测试和管理的内容。随着新技术的不断出现，这一系列标准仍在不断的更新变化之中。

以太网和Wi-Fi是用于局域网的两种最常见的传输技术。有线网络历史技术包括ARCNET，令牌环和FDDI等。当前无线网络技术还包括蓝牙、ZigBee和Thread等。

### CAN（Campus Network，校园网/园区网）

校园网/园区网，是一个在有限的地理区域相互连接的局域网组成的。网络设备（交换器、 路由器）和传输介质（光纤、5类电缆等）几乎由园区承租人或所有人（比如企业、大学、政府等）完全拥有。

### Backbone（骨干网）

骨干网是用来连接多个区域或地区的高速网络。每个骨干网中至少有一个和其他骨干网进行互联互通的连接点。不同的网络供应商都拥有自己的骨干网，用以连接其位于不同区域的网络。

骨干网可以将同一建筑物，校园环境中的不同建筑物或广泛地区的不同网络结合在一起。 通常，骨干网的容量大于与其相连的网络。

骨干网通常是用于描述大型网络结构，主要是要看清楚网络拓扑结构，而非具体使用的传输方式或协议。骨干网一般都是广域网：作用范围几十到几千公里。

### MAN（Metropolitan Area Network，城域网）

城域网是将一个城市范围内的用户与计算机资源相互连接的计算机网络，是介于LAN和WAN之间能传输语音与数据的公用网络。

城域网的典型应用即为宽带城域网，就是在城市范围内，以IP和ATM电信技术为基础，以光纤作为传输媒介，集数据、语音、视频服务于一体的高带宽、多功能、多业务接入的的多媒体通信网络。

### WAN（Wide Area Network，广域网）

教科书对WAN的定义是跨越地区、国家甚至世界的计算机网络。然而，在计算机网络协议和概念的应用方面，最好将WAN视为用于长距离以及不同的LAN、MAN和其他本地化的计算机网络架构之间传输数据的计算机网络技术。这是因为在常见的以太网或Wi-Fi上运行的LAN技术通常被设计为物理近端网络，不能传输数十，数百甚至数千公里。

广域网通常跨接很大的物理范围，所覆盖的范围从几十公里到几千公里，它能连接多个地区、城市和国家，或横跨几个洲并能提供远距离通信，形成国际性的远程网络。

很多技术都可用于广域网链路，比如电话线，无线电波和光纤。

常见的广域网包括公用电话交换网（ P S T N）、分组交换网（X .2 5）、数字数据网（D D N）、帧中继（ F R）、交换式多兆位数据服务（ S M D S）和异步传输模式（AT M）。

### Internet（互联网）

互联网是网络与网络之间所串连成的庞大网络，这些网络以一组标准的网络`TCP/IP协议族`相连，链接全世界几十亿个设备，形成逻辑上的单一巨大国际网络。这是一个网络的网络，它是由从地方到全球范围内几百万个私人的、学术界的、企业的和政府的网络所构成，通过电子，无线和光纤网络技术等等一系列广泛的技术联系在一起。这种将计算机网络互相联接在一起的方法可称作`网络互联`，在这基础上发展出覆盖全世界的全球性互联网络称互联网，即是互相连接一起的网络。

互联网带有范围广泛的信息资源和服务，例如相互关系的超文本文件，还有万维网的应用，支持电子邮件的基础设施，点对点网络，文件共享，以及IP电话服务。

互联网的主要前身为阿帕网（ARPANET）。1974年美国国防部国防高等研究计划署（ARPA）的罗伯特•卡恩和斯坦福大学的文顿•瑟夫开发了TCP/IP协议，定义了在电脑网络之间传送信息的方法。1983年1月1日，ARPA网将其网络核心协议由网络控制程序改变为TCP/IP协议。ARPA网使用的技术（如TCP/IP协议）成为了以后互联网的核心。它采纳的征求修正意见书过程，一直是发展互联网协议与标准所使用的机制，至今仍然发挥着作用。


结语
----
本文简要介绍了OSI模型的物理层和数据链路层的相关协议以及常见的网络设备，在接下来的文章中将会简要介绍网络的虚拟化技术，敬请期待。

Happy developing！

--------

参考资料
--------
1. [OSI Model](https://en.wikipedia.org/wiki/OSI_model)