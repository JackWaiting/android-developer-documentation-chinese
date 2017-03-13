# 

## 建立分享文件

当向其它应用安全的提供一个文件时，需要在应用中为提供的文件配置安全的方式，通过内容 URI。Android [FileProvider](https://developer.android.google.cn/reference/android/support/v4/content/FileProvider.html) 组件会为文件生成内容 URI，依赖于你在 XML 中提供的配置。本节将说明如何在应用实现 [FileProvider](https://developer.android.google.cn/reference/android/support/v4/content/FileProvider.html) ，以及如何指定你要提供给另外一个应用的文件。   

> 注：[FileProvider](https://developer.android.google.cn/reference/android/support/v4/content/FileProvider.html) 是在 V4（[v4 Support Library](https://developer.android.google.cn/topic/libraries/support-library/features.html#v4)）包中，关于导入这个库到应用中，请参见 [Support Library Setup](https://developer.android.google.cn/tools/support-library/setup.html)。 

## 详解 FileProvider

在清单文件中需要配置一个入口来定义 [FileProvider](https://developer.android.google.cn/reference/android/support/v4/content/FileProvider.html)。这个入口指向一个生成的内容 URIs，同样的这个 XML 文件名指向一个你分享的目录。

下面的代码片段表述了如何在清单文件中添加 [<provider>](https://developer.android.google.cn/guide/topics/manifest/provider-element.html) 元素，来指向 FileProvider 对象，如下：

    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.example.myapp">
        <application
            ...>
            <provider
                android:name="android.support.v4.content.FileProvider"
                android:authorities="com.example.myapp.fileprovider"
                android:grantUriPermissions="true"
                android:exported="false">
                <meta-data
                    android:name="android.support.FILE_PROVIDER_PATHS"
                    android:resource="@xml/filepaths" />
            </provider>
            ...
        </application>
    </manifest>


上例中，[android:authorities](https://developer.android.google.cn/guide/topics/manifest/provider-element.html#auth) 属性指向一个 URI，这个 URI 是 FileProvider 根据内容生成的。本例中，认证码是：*com.example.myapp.fileprovider*。为自己的应用指向的认证码是 [ android:package](https://developer.android.google.cn/guide/topics/manifest/manifest-element.html#package) 加 "fileprovider" 的后缀。学习更多的认证值，见主题 [Content URIs](https://developer.android.google.cn/guide/topics/providers/content-provider-basics.html#ContentURIs) 和 android:authorities 属性的文档。   

在 XML 文件中的 [<provider>](https://developer.android.google.cn/guide/topics/manifest/provider-element.html) 节点下，[<meta-data>](https://developer.android.google.cn/guide/topics/manifest/meta-data-element.html) 子元素是指向你想分享的目录。*android:resource* 的值是文件的路径和名称，排除了 *.xml* 的后缀。下一节将介绍描述文件的内容。   

## 指定可分享的目录

一旦你添加 FileProvider 到清单文件中，就需要去指定包括你想分享的文件目录。创建 *filepaths.xml* 文件来指定目录，在项目的 *res/xml/* 子目录下。在这个文件中，为每一个目录添加一个 XML 标签。下面的片段是 *res/xml/filepaths.xml* 例子的内容。这个片段也展示了如何如分享子目录，在内存卡区域的 *file/* 目录中。

    <paths>
        <files-path path="images/" name="myimages" />
    </paths>
    
在本例中，*<files-path>* 标签分享的目录是在内存卡的 files/ 目录下。path 属性表示分享 images/ 的子目录来自 files/ 目录下。name 属性告诉 FileProvider 添加路径 files/images/ 到内容 URIs 中。

*<paths>* 元素可以拥有多个子元素，每个元素指向不同分享的目录。在添加的 *<files-path>* 元素中，使用 *<external-path>* 标签分享内存卡目录，使用 *<cache-path>* 标签分享内部缓存目录。学习更多分享目录的子标签，参见 FileProvider 相关文档。

> 注：XML 文件是分享文件仅有的方式，不能直接通过程序分享目录。

现在学习了如何使用 FileProvider 的完整流程，生成的内容 URIs 对应的是应用的内存卡或 files/ 的子目录下的文件。当为文件生成一个内容 URI 时，它包含 *<provider>* 元素 *(com.example.myapp.fileprovider)*，路径 *images/* 和 文件名。

例如，当按照本节去定义一个 FileProvider 时，你请求 *default_image.jpg* 文件的 URI，FileProvider 会返回如下路径：

    content://com.example.myapp.fileprovider/myimages/default_image.jpg
    
    


>翻译：[@iOnesmile](https://github.com/iOnesmile)    
原始文档：<https://developer.android.google.cn/training/secure-file-sharing/setup-sharing.html#DefineProvider>
