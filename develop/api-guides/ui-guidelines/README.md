# 图标设计指导原则

>已经有新版针对应用设计师的指南！  
>可在 **[Android 设计](https://developer.android.com/design/index.html)**中查看最新的文档，包括更多关于[图标](https://developer.android.com/design/style/iconography.html)的指导原则。

在整个用户界面中保持统一的外观和风格，对于你的产品来说是增值的。精简的图形风格也会让用户觉得你的用户界面对于用户来说，看起来更专业。

此文档提供帮助你创建针对你应用用户界面不同部分的图标，这些图标适配采用 Android 2.x 框架的一般样式。遵循这些设计指南将帮助你创一个优雅并且统一的用户体验。

以下文档是针对 Android 应用中使用的所有的常见类型的图标的细节指南探讨：

[启动器图标](https://developer.android.com/guide/practices/ui_guidelines/icon_design_launcher.html)

启动器图标是在启动器窗口和设备主屏幕上代表你应用的图片。

[菜单图标](https://developer.android.com/guide/practices/ui_guidelines/icon_design_menu.html)

菜单图标是当用户点击菜单按钮的时候，放置在可选菜单中的图片元素。

[操作栏图标](https://developer.android.com/guide/practices/ui_guidelines/icon_design_action_bar.html)

操作栏图标是在[操作栏](https://developer.android.com/guide/topics/ui/actionbar.html)中代表操作条目的图片元素。

[状态栏图标](https://developer.android.com/guide/practices/ui_guidelines/icon_design_status_bar.html)

状态栏图标是在状态栏中代表你应用的通知。

[标签图标](https://developer.android.com/guide/practices/ui_guidelines/icon_design_tab.html)

标签图标是在多标签界面中的代表单个标签的图片元素。

[对话框图标](https://developer.android.com/guide/practices/ui_guidelines/icon_design_dialog.html)

对话框图标是在弹出的对话框中显示的，用来提醒用户进行交互的图标。

[列表图标](https://developer.android.com/guide/practices/ui_guidelines/icon_design_list.html)

列表图标是在 [ListView](https://developer.android.com/reference/android/widget/ListView.html) 中代表列表条目的。例如设置应用。为了更加快速的创建你的图标，你可以下载 Android 图标模板包。

##Android 图标模板包的使用

Android 图标模板包，是一个设计、结构和层样式的模板集合，能够让你更加容易的创建符合文档中指南的要求的图标。我们建议你开始图标设计之前，先下载模板包存档。

图片模板是以 Adobe Photoshop 文件格式（.psd）提供的，这样格式的文件保留了我们创建 Android 平台的标准图标时候的图层和设计处理。你可以用任何兼容的图片编辑程序载入模板文件，尽管你对图层和设计处理的能力，可能会因为你使用的程序不同而不同。

你可以从以下链接获取最新的图标模板包：

[下载最新的 Android 4.0 图标模板包»](https://developer.android.com/shareables/icon_templates-v4.0.zip)

对于较早版本的图标模板包，请参见本页面右上角的下载部分。

##提供特定密度的图标集合

Android 被设计成可以在各种不同屏幕尺寸和分辨率设备上运行的系统。当你设计应用图标的时候，切记你的应用将会被安装到这些设备上。如[多屏幕支持](https://developer.android.com/guide/practices/screens_support.html)文档中介绍的一样，Android 平台让你通过这种方式直截了当的提供图标，能够在任何设备上正确的显示，而不用考虑设备的屏幕尺寸和分辨率。

一般来说，推荐的针对每一个广义的屏幕密度创建一套单独的图标。然后将他们存储在你应用的指定密度资源目录。当你的应用运行时，Android 平台会检查设备屏幕特性并且载入正确的指定密度的图标资源。更多关于如何在你的应用中存储指定密度资源，请参见[屏幕尺寸和密度的资源目录限定符](https://developer.android.com/guide/practices/screens_support.html#qualifiers)。

如何创建和管理针对多密度的图标集合的小提示，请参见[设计师的小提示](https://developer.android.com/guide/practices/ui_guidelines/icon_design.html#design-tips)。

当你开发你应用的图标或者其他图片资源的时候，你会发现以下小提示比较有用。小提示假设你使用 Adobe Photoshop 或者类似的光栅和矢量图片编辑程序。

###针对图标资源使用常用的命名约定

在同一个目录中的文件命名可以尝试将相关资源分在一组，这样可以通过字母表排序归类到一起。尤其是当针对同一种图标类型使用相同前缀。例如：

|资源类型|前缀|举例
|---|---|---
|图标|`ic_`|`ic_star.png`
|启动器图标|`ic_launcher `|`ic_launcher_calendar.png`
|菜单图标和操作栏图标|`ic_menu `|`ic_menu_archive.png`
|状态栏图标|`ic_stat_notify`|`ic_stat_notify_msg.png`
|标签图标|`ic_tab`|`ic_tab_recent.png`
|对话框图标|`ic_dialog`|`ic_dialog_info.png`

请注意：并不是强制要求你使用任何类型共享前缀，这样做，仅仅是为了让你更加方便。

###为多密度创建一个组织文件的工作空间

多屏幕密度的支持，意味着你需要创建同一个图标的多个版本。为了保证多份文件的拷贝的安全性和更容易找到，我们建议在你的工作空间中创建一个组织针对每个分辨率的资源文件目录结构。例如：

```
art/...
    ldpi/...
        _pre_production/...
            working_file.psd
        finished_asset.png
    mdpi/...
        _pre_production/...
            working_file.psd
        finished_asset.png
    hdpi/...
        _pre_production/...
            working_file.psd
        finished_asset.png
    xhdpi/...
        _pre_production/...
            working_file.psd
        finished_asset.png
```

这种结构和在你应用资源中存储的最终处理好的资源结构保持一致的。因为你工作空间的结构和应用的类似，你可以快速决定哪个资源可以复制到应用资源目录。根据不同密度区分的资源也能帮你检测不同密度文件名的差异，因为不同密度的相关资源必须采用相同的文件名称。

为了对比，以下给出的是一个典型应用的资源目录结构：

```
res/...
    drawable-ldpi/...
        finished_asset.png
    drawable-mdpi/...
        finished_asset.png
    drawable-hdpi/...
        finished_asset.png
    drawable-xhdpi/...
        finished_asset.png
```

###尽可能的使用矢量图

许多诸如 Adobe Photoshop 的图片编辑程序允许你使用矢量图、栅格图层和效果的组合。如果可以的话，尽量使用矢量图，这样就可以在需要的时候，资源可以无损放大，而不会导致细节损失和脆边。

###从大画板开始

因为你需要针对不同的屏幕密度创建资源，是在一个能包含多个目标图标尺寸的大画板上的图标设计的方式，是一个不错的选择。例如，启动器图标是 96，72，48，或 36 pixels 宽，取决于屏幕密度。如果你刚开始是在 864x864 的画板上开始设计启动器图标的话，就会比较容易和清晰的调整图标到最终目标尺寸的资源。

###缩放的时候，如果有必要就重绘位图图层

如果你是从一个位图图层放大一张图片，而不是从一个矢量图层放大一张图片，这些图层需要手动重绘，以便在更高密度中清晰显示。例如，如果一个 60x60 的圆以 mdpi 密度的位图绘制的话，那么对于一个  90x90 的圆，应当以 hdpi 密度绘制。

###保存图片资源的时候，移除非必须的元数据

尽管 Android SDK 工具会在应用资源打包成应用二进制文件的时候，自动压缩 PNG 图片，但是移除 PNG 资源的非必须的头和元数据是一个好的习惯。诸如 [OptiPNG](http://optipng.sourceforge.net/) 或 [Pngcrush](http://pmt.sourceforge.net/pngcrush/) 的工具可以确保元数据被移除，优化图片资源文件的大小。

###确保不同密度相关资源使用相同的文件名

每个密度的对应的图片资源文件必须使用相同的文件名，但是需要分别存储到指定密度的资源目录中。这种方式可以让系统根据设备屏幕特性来查找和载入正确的资源。正因为如此，需要确保在每个目录下的资源保持一致，文件名不要使用特定密度前缀。

更多关于特定密度资源和系统如何使用这些资源来适配不同设备，请参见[多屏幕支持](https://developer.android.com/guide/practices/screens_support.html)。





















