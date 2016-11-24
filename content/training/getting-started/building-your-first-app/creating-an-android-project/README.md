# 

创建一个android应用

一个android的应用包含所有的源码和文件.这个Android SDK工具将会生成一些默认的目录和文件，因此你将会轻松的去新建一个android工程。

这篇课程将会告诉你如何使用Eclipse(安装ADT插件)或者SDK工具去创建一个新的工程。

注:你应该已经安装了Android SDK，如果你使用Eclipse你应该安装了ADT插件。如果你没安装这些，看Installing the Andrid SDK并完成这些安装
然后回到这里。

使用Eclipse创建一个新项目
1.在Eclipse中，选择File>New>Project.在弹出对话框中应该有一个带有android标签的文件夹.(如果你没有看见这个Android的文件夹啊，那么你是
没有安装ADT插件-去阅读Installing the ADT Plugin)

2.打开Android文件夹，选择Android Project 并且点击Next。

3.确定工程的名称(例如"MyFirstApp")并且点击Next.

4.选择构建目标.这是编译的app的目标平台版本.

 我们推荐你尽可能选择最新的版本.你任然可以编辑你的app去支持旧的版本，但是设置的目标版本为最新的版本会你轻松的优化你的应用在最新的设备设备上，
和最好的用户体验。

如果你没有看见任何编译目标列表，你需要使用Android SDK Manager tool安装某些，阅读 step 4 in the installing guide.

点击Next.
5 指定其他app的细节，例如:
	Application Name: 这个应用的名字展现给用户，输入"My First App".
	Package Name: 为你的app命名空间(包名的规则遵循Java编程语言).你的包名必须是唯一
