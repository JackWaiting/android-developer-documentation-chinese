# 

Android 中最重要的功能之一就是可以利用一个想要执行的意图，使用户可以从一个应用跳转至另一个应用。例如，如果有一个公司的地址并且想显示在地图上，你不需要创建一个 activity 来展示这个地图。只需要使用 [Intent](https://developer.android.google.cn/reference/android/content/Intent.html) 发起一个请求来展示这个地址。Android 系统会启动能够展示地图的程序来展示此地址。    

正如第一节课所讲：[构建第一个应用](https://developer.android.google.cn/training/basics/firstapp/index.html)时，我们必须使用 intents 去实现在不同的 activity 之间进切换。通常定义一个显式的 intent，它定义了需要启动的组件类名。然而，当想要有一个单独的应用去执行某个操作，如“查看地图”，则必须使用隐式 intent 了。    

本课程主要讲解如何针对特定的操作创建一个隐式 intent，以及如何使用隐式intent 去启动在另一个应用中执行操作的 activity。
