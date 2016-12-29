#创建Android项目

这篇课程将会告诉你如何使用Android Studio创建新的Android项目和介绍项目中的一些文件。

## 1.在Android Studio中，创建新的项目
* 如果你未打开任何项目，请在Weclome to Android Studio的窗口中，点击Start a new Android Studio project
* 如果你已打开项目，请选择File > New Project

## 2.在New Project输入以下内容
* Application Name: "My First App"
* Company Domain："example.com"
<br>Android Studio 会为你提供默认的软件包名称和项目位置， 但您也可根据需求去编辑这些内容

## 3.点击Next

## 4.在Target Android Devices 屏幕中，保留默认并点击 Next。
   Minimum Required SDK 是指您的应用所能支持的最低 Android 版本，由API级别表示。为了能支持尽可能多的设备，您应该设置成可让你的应用提供核心功能可用的最低版本。 如果应用的任何功能只能在更新版本的 Android上运行，并且不属于核心功能集的关键功能，则可仅当在支持该功能的版本上运行时启用该功能（请参阅[支持不同平台版本](https://android.developerdocumentation.cn/develop/training/getting-started/supporting-different-devices/supporting-different-platform-versions/index.html)）。
   
## 5.在Add an Activity to Mobile屏幕中，选择Empty Activity，然后点击 Next。

## 6.在Customize the Activity 屏幕中，保留默认值并点击Finish。

经过一些处理后,Android Studio将打开并显示包含默认文件的"Hello World"应用。在以下课程中，你将向其中一些文件添加功能。

下面让我们花一点时间回顾一下最重要的文件：首先，请确保已打开 Project 窗口（选择 `View > Tool Windows > Project`），并从顶部的下拉列表中选择 Android 视图。随后，您可以看到下列文件：

`app > java > com.example.myfirstapp > MainActivity.java`
<br>完成新项目向导后，该文件将显示在 Android Studio 中。 它包含您之前创建的 Activity 的类定义。当您构建并运行应用时，Activity 会启动，并加载显示“Hello world!”的布局文件。

`app > res > layout > activity_main.xml`
<br>此 XML 文件定义您的 Activity 的布局。它包含带有文本“Hello world!”的 TextView 元素。

`app > manifests > AndroidManifest.xml`
<br>清单文件描述应用的基本特性并定义其每个组件。 随着课程学习的深入和为应用添加组件的增多，您会反复用到这个文件。

`Gradle Scripts > build.gradle`
<br>Android Studio 使用 Gradle 来编译和构建您的应用。您的项目的每个模块都有相应的 build.gradle 文件，整个项目也有相应的 build.gradle 文件。 通常您只关心模块（在本例中，为 app 或称应用模块）的 build.gradle 文件。如需了解有关此文件的详细信息，请参阅使用 Gradle 构建您的项目。

要运行该应用，请继续学习[下一课](https://android.developerdocumentation.cn/develop/training/getting-started/building-your-first-app/running-your-application/index.html)。
>翻译：[@edtj](https://github.com/edtj)    
原始文档：<https://developer.android.google.cn/training/basics/firstapp/creating-project.html>
