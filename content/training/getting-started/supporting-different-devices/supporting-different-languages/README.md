# 

##支持不同的语言
在你的应用中把UI字符串存放到外部文件中并通过代码获取，一直是一个好的办法。Android 在每个 Android 项目中都提供一个资源目录，从而简化了这一过程。  

如果你是使用 Android SDK Tools创建的工程（请阅读[创建 Android 项目](https://android.developerdocumentation.cn/content/training/getting-started/building-your-first-app/index.html))，则会在工程的根目录中创建一个 `res/` 目录。此目录中存放着各类资源的子目录，还包含一些默认文件如 `res/values/strings.xml`，用于存放你的字符串值。

##创建不同的语言区域目录和字符串文件

为了支持多国语言，需要在`res/`中创建一个额外的 `valuesres/` 目录，并在目录名称末尾加上连字符和ISO 语言代码。例如，`values-es/` 目录包含的简单资源用于语言代码为“es”的语言区域。Android 应用在运行时会根据设备的语言区域设置加载相应的资源。如需了解详细信息，请参阅[提供备用资源](https://developer.android.com/guide/topics/resources/providing-resources.html#AlternativeResources)。


一旦你决定了为哪些语言提供支持，便可创建资源子目录和字符串资源文件。 例如：

    MyProject/    
          res/    
             values/        
                 　strings.xml    
            　values-es/    
                 　strings.xml     
              values-fr/     
       　　　　　　　 strings.xml
    
    
将各个语言区域的字符串值添加到相应文件中。
 
在运行时，Android 系统会根据当前为用户设备设置的语言区域使用相应的字符串资源集。

例如，以下是一些面向不同语言的不同字符串资源文件。

英语（默认语言区域)，`/values/strings.xml`：    

```xml
<?xml version="1.0" encoding="utf-8"?>    
<resources>    
    <string name="title">My Application</string>
    <string name="hello_world">Hello World!</string>
</resources>
```
西班牙语，`/values-es/strings.xml`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="title">Mi Aplicación</string>
    <string name="hello_world">Hola Mundo!</string>
</resources>
```
法语，`/values-fr/strings.xml`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="title">Mon Application</string>
    <string name="hello_world">Bonjour le monde !</string>
</resources>
```
>**Note**: You can use the locale qualifier (or any configuration qualifer) on any resource type, such as if you want to provide localized versions of your bitmap drawable. For more information, see [Localization](https://developer.android.com/guide/topics/resources/localization.html).

##使用字符串资源
你可以在源代码和其他XML文件中通过`<string>`元素的`name`属性来引用自己的字符串资源。

在源代码中可以通过`R.string.<string_name>`语法来引用一个字符串资源，很多方法都可以通过这种方式来接受字符串.    

例如:    

```java     
// Get a string resource from your app's Resources
String hello = getResources().getString(R.string.hello_world);

// Or supply a string resource to a method that requires a string
TextView textView = new TextView(this);
textView.setText(R.string.hello_world);
```
在其他XML文件中，只要 XML 属性接受字符串值，你就可以使用@string/<string_name>语法来引用字符串资源。    
例如:  

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/hello_world" />
```

