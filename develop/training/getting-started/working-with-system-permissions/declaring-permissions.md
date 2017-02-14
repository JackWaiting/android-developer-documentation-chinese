# 

## 声明权限

所有的 Android 应用都运行在一个访问受限制的沙盒中。如果你的需要使用应用之外的资源或信息，那么应用需要请求对应的权限。声明所需的权限在 [AppManifest](https://developer.android.google.cn/guide/topics/manifest/manifest-intro.html) 文件的权限列表中。   

根据权限的敏感程度，系统会自动授予一部分权限，另外一部分则需要请求用户来授予。例如：应用需要打开设备闪光灯的权限，这个权限会被系统自动授予。但是如果你需要读取用户联系人，系统会询问用户是否授予该权限。根据不同系统版本，Android5.0 及一下用户授予权限在安装时，Android6.0 及以上在应用运行时由用户授予权限。   

## 确定你需要的权限   

在开发应用中，你应该注意功能所需要请求的权限。通常，应用需要请求权限在使用非自身创建的信息或资源时，或在执行影响设备或其它应用行为的操作时。例如，如果需要连接互联网，使用设备相机，或打开和关闭 Wifi，都需要请求适当的权限。系统权限列表，参见[《普通与危险权限》](https://developer.android.google.cn/guide/topics/permissions/requesting.html#normal-dangerous)。   

应用只需要为直接执行的动作授予权限。当请求因外一个应用执行任务或提供信息时不需要权限。例如，如果应用需要去读取用户通讯录，则需要 [READ_CONTACTS](https://developer.android.google.cn/reference/android/Manifest.permission.html#READ_CONTACTS) 权限。但如果通过 Intent 去请求通讯录应用的信息，你的应用不需要权限，但是通讯录应用需要有权限。更多信息，参见[《细思Intent》](https://developer.android.google.cn/training/permissions/usage-notes.html#perms-vs-intents)。    

## 添加权限到清单文件   

声明应用所需的权限，放 [<uses-permission>](https://developer.android.google.cn/guide/topics/manifest/uses-permission-element.html) 标签到[应用清单](https://developer.android.google.cn/guide/topics/manifest/manifest-intro.html)中，作为顶级 [<manifest>](https://developer.android.google.cn/guide/topics/manifest/manifest-element.html) 标签的子项。例如，应用需要发送短信应该放入如下代码在清单文件中：   

	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	        package="com.example.snazzyapp">
	
	    <uses-permission android:name="android.permission.SEND_SMS"/>
	    
	
	    <application ...>
	        ...
	    </application>
	
	</manifest> 

你声明的权限有多敏感决定了之后的系统行为。如果你的权限不涉及到用户隐私，系统会自动授予。如果权限访问到用户的敏感信息，系统会提示用户是否授予该权限。更多关于不同权限种类的信息，参见[《普通与危险权限》](https://developer.android.google.cn/guide/topics/permissions/requesting.html#normal-dangerous)。




>翻译：[@iOnesmile](https://github.com/iOnesmile)    
原始文档：<https://developer.android.google.cn/training/permissions/declaring.html>
