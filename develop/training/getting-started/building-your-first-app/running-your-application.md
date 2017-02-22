# 运行你的 App

在[上一课](https://developer.android.com/training/basics/firstapp/creating-project.html)，我们创建了一个“Hello World”的项目。现在运行App在真机或模拟器上，如果你还没有一个真机，请直接看 [在模拟器上运行](#2)。

##[在真机上运行](#1)##
---
环境准备，将你的设备调整为如下状态：   
1，通过USB数据线连接手机到你的电脑上。如果你的电脑系统是Windows，你可能需要为你的电脑安装一个合适的USB驱动。驱动安装帮助，请参见 [OEM USB Drivers](https://developer.android.com/studio/run/oem-usb.html) 文档。   

2，能够使用 **USB调试** 你的手机，打开 **设置 > 开发者选项**

>**提示**：在Android4.2以及更高的版本上， **开发者选项** 默认是隐藏的。打开 **设置 > 关于手机** ，连续点击 **版本号** 七次。返回到设置界面去找 **开发者选项**。   


在Android Studio上运行App的方式如下：   

1. 在Android Studio上选中项目，然后点击  **Run** 在工具栏上。  

2. 在 **Select Deployment Target** 的对话框中，选择设备并点击 **OK**。   

Android开始安装App在设备上，并会启动它。


##[在模拟器上运行](#2)##
---
如果你之前没有运行App在模拟器上，你需要先创建一个 [Android模拟器](https://developer.android.com/tools/devices/index.html) （AVD）。配置一个类型如Android手机、平板、穿戴、TV在你的Android模拟器上。   

创建Android模拟器的方式如下：   

1，打开Android模拟器管理窗口，**工具 》 Android 》 Android模拟器管理**，或者点击工具栏上的快捷图标。  
 
2，在 **你的虚拟设备** 页面中，点击 **创建虚拟设备** 。   

3，在 **选择硬件** 页面中，选择一款手机，如Nexus 6，接着点击 **Next**。   

4，在 **系统镜像** 页面中，为模拟器选择一个想要的系统，接着点击 **Next**。   
如果你选中的那个系统镜像没有安装，请点击“下载”链接进行安装。   

5，检查配置设置（对于第一个AVD，保持原样的设置），点击 **Finish**。   
关于使用Android虚拟机的更多信息，请参见 [创建和管理虚拟设备](https://developer.android.com/studio/run/managing-avds.html) 。   

在Android Studio上运行App的方式如下：   

1，在 **Android Studio** 上选中项目，然后点击 **Run** 在工具栏上。   

2，在**Select Deployment Target**的对话框中，选择设备并点击 **OK**。

<p>让这个模拟器启动你需要等待一会。之后你需要解锁屏幕。当你解锁后，“我的第一个App”就展现在模拟器上了。
<p>这些是告诉你如何创建并运行App在模拟器上！加油，继续 [下一课](https://developer.android.com/training/basics/firstapp/building-ui.html)。


>翻译：[@iOnesmile](https://github.com/iOnesmile)    
原始文档：<https://developer.android.com/training/basics/firstapp/running-app.html>
