---
title: 使用ESP8266配合0.96寸OLED显示AIDA64读取到的电脑状态信息
date: 2020-04-17
categories: tutorial
description: 使用ESP8266配合0.96寸OLED显示器显示电脑状态信息，如CPU、GPU的温度、风扇转速、占用率等。
tags: [Arduino,ESP8266,AIDA64,OLED]
---
>本文的代码基于[《ESP8266电脑主机状态监控数显模块制作》](https://www.mydigit.cn/forum.php?mod=viewthread&tid=126778)，本文只是对其教程进行更为详细的转述，感谢这位老哥的分享。

## 前言：
前不久入手了一个工控主板用作软路由，CPU使用是集成于主板的AMD A8-6410，TDP为15W，4核心，PassMark的多核得分1741，单核得分764。以下为这款处理器的详细参数如下表所示：

| AMD A8-6410 APU |  |
|:--------:|:--------:|
| **Other names** | AMD A8-6410 APU with AMD Radeon R5 Graphics |
| **Class** | Laptop |
| **Clockspeed** | 2.0 GHz |
| **Turbo Speed** | 2.4 GHz |
| **No of Cores** | 4 |
| **Typical TDP** | 15 W |
| **Average CPU Mark** | 1741 |
| **Single Thread Rating** | 764 |
| **CPU First Seen on Charts** | Q2 2014 |
| **Cross-Platform Rating** | 3603 |
| **Samples:**  | 1375* |
| *[**Margin for error**](https://www.cpubenchmark.net/graph_notes.html#samples)  | Low |
| **Overall Rank** | 1556 |

上表数据来源：[PassMark - AMD A8-6410 APU](https://www.cpubenchmark.net/cpu.php?cpu=AMD%20A8-6410%20APU)

现在板子安装的系统是Windows 10 企业版LTSC，版本号为1809，软路由系统OpenWrt安装于Win 10自带的Hyper-V虚拟机内，至于为什么不直接将OpenWrt安装于物理机中，恩山论坛的这位老哥[**墨色之月**](https://www.right.com.cn/forum/space-uid-306311.html)的帖子里有说明：[《这是怎么做到的，A8-6410能做到待机功耗7W》](https://www.right.com.cn/forum/thread-2959996-1-1.html)，这里不再赘述。平时不接显示器使用，于是需要对系统的运行状态进行监控，在网上找到了一篇帖子：[《ESP8266电脑主机状态监控数显模块制作》](https://www.mydigit.cn/forum.php?mod=viewthread&tid=126778)，本文只是对其教程进行更为详细的转述。

## 硬件要求

 1. ESP8266开发板；
 2. 0.96寸4针OLED液晶显示屏；
 3. 母对母杜邦线或者其它能用的导线至少4根；
 4. Micro USB数据线；
 5. 2.4G WiFi路由器（需要知道SSID和密码）；
 6. ESP8266开发板与0.96寸OLED屏的接线：
 ![31.png](https://i.loli.net/2020/04/17/lkzAfhvHKW4bLUx.png)

## 软件要求

1. Arduino IDE，下载链接：[点此进入下载页面](https://www.arduino.cc/en/Main/Software?setlang=cn)（可能需要特殊网络环境）。
2. AIDA64，文件下载链接附于文末。

以本文所附版本为例：先解压，再打开aida64.exe-文件-设置-LCD，选择“RemoteSensor”，更改“TCP/IP端口”，比如我改为888，然后勾选“启用RemoteSensor LCD支持”和“Maxmize on double-click”。如图所示：
![24.png](https://i.loli.net/2020/04/17/dXwTSPfoR9FnK41.png)
再选择“LCD项目”，导入文件配置文件`8266OLED.rslcd`。操作过程及导入后的效果如图所示：
![25.png](https://i.loli.net/2020/04/17/5ZRyrHTUxDCdfaL.png)
**上述设置完之后一定要记得保存！！！** 
![28.png](https://i.loli.net/2020/04/17/M3xHckJC16pdKej.png)
在浏览器中打开链接：`http://你电脑的IP:设置的端口`，就可以看到如下画面：
![26.png](https://i.loli.net/2020/04/17/qxjdUHXPysBrAFl.png)
上图这些都是电脑的状态信息，本文中ESP8266的功能就是通过连接WiFi读取这些内容并显示在OLED屏幕上。

## 依赖安装

1. 在Arduino IDE（版本号1.8.12）中安装对8266开发板的支持。打开Arduino IDE，在文件-首选项的“附加开发板管理器网址”中输入：`http://arduino.esp8266.com/stable/package_esp8266com_index.json`
![11.png](https://i.loli.net/2020/04/17/tWvzpneNaXo48bU.png)
然后，在工具-开发板-开发板管理器中搜索“`esp8266`”，点击安装（可能需要特殊网络环境）。
![12.png](https://i.loli.net/2020/04/17/rLDIhemUsOGdc6F.png)
2. 安装需要用的库文件，在项目-加载库-管理库中搜索并安装以下库文件（可能需要特殊网络环境）：
- ArduinoJson
![15.png](https://i.loli.net/2020/04/17/rRYNZnQymPdVS1W.png)
- WiFiManager
![16.png](https://i.loli.net/2020/04/17/kVldPYK5ZQrbsJz.png)
- U8g2
![17.png](https://i.loli.net/2020/04/17/IDV1bkaKNzPtQvE.png)

## 代码运行

1. 将ESP8266开发板连接电脑，打开设备管理器，查看端口号（我的电脑里是COM3，每台电脑可能不一样，没关系）：
![21.png](https://i.loli.net/2020/04/17/x2gaELpYnA9ulwh.png)
若不出现端口号，则需要安装开发板驱动，我使用的是驱动精灵免安装版进行驱动安装，可在文末链接中下载。
2. 在工具-开发板中选中NodeMCU 1.0(ESP-12E Module)，如图所示：
![22.png](https://i.loli.net/2020/04/17/A54FiScyNnUmX1I.png)
在Arduino IDE的工具-端口中，选中上一步得到的端口号，如图所示：
![23.png](https://i.loli.net/2020/04/17/itJD2BawqVLNEAH.png)
3. 使用Arduino IDE打开代码，代码下载链接附于文末。
4. 修改代码：
- 更改WiFi名称、WiFi密码以及需要访问的域名和端口号，如图所示：
![19.png](https://i.loli.net/2020/04/17/xAzQJYH2F1lgNrL.png)
- 更改需要显示的内容，是显示CPU信息还是GPU信息（暂时只能选其一）：
![18.png](https://i.loli.net/2020/04/17/j9Mayh5lWGmS6H1.png)
6. 开始编译运行并上传：
- 打开项目-上传，即会对代码进行编译并上传至开发板。
![27.png](https://i.loli.net/2020/04/17/7a3V1w8qIsDFfrW.png)
 -  Enjoy！！！
 
## 效果

1. AMD A8-6410 CPU状态信息：
![29.jpg](https://i.loli.net/2020/04/17/fTXB4HYMrkEmQgW.jpg)
2. GPU状态信息：
![30.jpg](https://i.loli.net/2020/04/17/gTxDSdQbnHosfqc.jpg)

## 所需文件
所有文件：[网盘下载](https://xrp001.lanzous.com/b00zesdlc)、[GitHub库下载](https://github.com/xrp001/AIDA64_ESP8266_Reader)（任选其一）

## 扩展

1. 可以自行更改图标；
2. 增加CPU状态信息和GPU状态信息的轮换
3. 。。。
