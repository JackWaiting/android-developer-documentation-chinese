#构建简单的用户界面
***

在本课中，您将学习如何创建一个 XML 格式的布局，其中包含一个文本字段和一个按钮。 在下一课中，在按下此按钮时，您的应用会将文本字段的内容发送给另一个 Activity 作为响应。

Android 应用的图形界面使用 [View](https://developer.android.google.cn/reference/android/view/View.html) 对象和 [ViewGroup](https://developer.android.google.cn/reference/android/view/ViewGroup.html) 对象层次结构而构建。View 对象通常为[按钮](https://developer.android.google.cn/guide/topics/ui/controls/button.html)或[文本字段](https://developer.android.google.cn/training/keyboard-input/style.html)之类的 UI 小部件。而 ViewGroup 对象则为不可见的视图容器，它们定义子视图的布局，比如是网格布局还是垂直列表布局。

Android 提供对应于 View 和 ViewGroup 子类的 XML 词汇，以便您使用 UI 元素层次结构以 XML 格式定义 UI。

布局是 ViewGroup 的子类。在本练习中，您将学习创建 [LinearLayout](https://developer.android.google.cn/reference/android/widget/LinearLayout.html)。

![image](viewgroup.png)

##创建线性布局
1.在 Android Studio 的 Project 窗口中，打开 app > res > layout > activity_main.xml。
此 XML 文件定义 Activity 的布局。它包含默认的“Hello World”文本视图。

2.在[布局编辑器](https://developer.android.google.cn/studio/write/layout-editor.html)中，当您打开一个布局文件时，首先显示的是设计编辑器。 在本课中，您将直接使用 XML，因此请点击窗口底部的 Text 标签以切换到文本编辑器。

3.删除所有内容并插入以下 XML：
		
		<?xml version="1.0" encoding="utf-8"?>
		<LinearLayout
		    xmlns:android="http://schemas.android.com/apk/res/android"
		    xmlns:tools="http://schemas.android.com/tools"
		    android:layout_width="match_parent"
		    android:layout_height="match_parent"
		    android:orientation="horizontal">
		</LinearLayout>

LinearLayout 是一个视图组（ViewGroup 的子类），它会按照 [android:orientation](https://developer.android.google.cn/reference/android/widget/LinearLayout.html#attr_android:orientation) 属性的指定，将子视图设置为垂直或水平方向布局。LinearLayout 的每个子视图都会按照它们各自在 XML 中的出现顺序显示在屏幕上。

其他两个属性 [android:layout_width](https://developer.android.google.cn/reference/android/view/View.html#attr_android:layout_width) 和 [android:layout_height](https://developer.android.google.cn/reference/android/view/View.html#attr_android:layout_height) 则是所有视图的必备属性，用于指定它们的尺寸。

LinearLayout 是布局中的根视图，因此应将宽度和高度设置为 "match_parent"，从而填满可供应用使用的整个屏幕区域。 该值表示视图应扩大其宽度或高度，以匹配父视图的宽度或高度。

如需了解有关布局属性的详细信息，请参阅[布局](https://developer.android.google.cn/guide/topics/ui/declaring-layout.html)指南。

##添加文本字段


在 activity_main.xml 文件的 [<LinearLayout](https://developer.android.google.cn/reference/android/widget/LinearLayout.html) 元素内，添加以下 [<EditText](https://developer.android.google.cn/reference/android/widget/EditText.html) 元素：

	<LinearLayout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:orientation="horizontal">
	    <EditText android:id="@+id/edit_message"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:hint="@string/edit_message" />
	</LinearLayout>

无需担心出现 @string/edit_message 错误；您很快就会修复该错误。

下面说明了您添加的 <EditText 中的属性：

[android:id](https://developer.android.google.cn/reference/android/view/View.html#attr_android:id)

这会为视图赋予唯一的标识符，您可以使用该标识符从应用代码中引用对象，例如读取和操作对象（下一课中将有介绍）。

从 XML 引用任何资源对象时，都需要使用 @ 符号。 请在该符号后依次输入资源类型（本例中为 id）、斜杠和资源名称 (edit_message)。

只有在第一次定义资源 ID 时，才需要在资源类型之前加一个加号 (+)。 当您编译应用时，SDK 工具会使用 ID 名称在项目的 R.java 文件中新建一个引用 [EditText](https://developer.android.google.cn/reference/android/widget/EditText.html) 元素的资源 ID。一旦以此方式声明资源 ID，其他对该 ID 的引用皆无需使用加号。 只有在指定新资源 ID 时才必须使用加号，对于字符串或布局等具体资源则不必如此。 如需了解有关资源对象的详细信息，请参阅边栏。

>资源对象
>
资源对象是一个唯一的整型名称，它与应用资源（如位图、布局文件或字符串）相关联。
>		
每个资源在您项目的 R.java 文件中都定义有相应的资源对象。您可以使用 R 类中的对象名称来引用您的资源，例如当您需要为 android:hint 属性指定字符串值时，就可以这样做。您也可以使用 android:id 属性创建任意与视图相关联的资源 ID，从而可以从其他代码中引用该视图。
>		
SDK 工具会在您每次编译应用时生成 R.java 文件。切勿手动修改该文件。
>		
如需了解详细信息，请阅读[提供资源](https://developer.android.google.cn/guide/topics/resources/providing-resources.html)指南。

[android:layout_width](https://developer.android.google.cn/reference/android/view/View.html#attr_android:layout_width) 和 [android:layout_height](https://developer.android.google.cn/reference/android/view/View.html#attr_android:layout_height)

"wrap_content" 值并不规定宽度和高度的具体大小，而是指定根据需要缩放视图，使其适合视图的内容。 如果您要改用 "match_parent"，则 EditText 元素将填满屏幕，因为它会匹配父 LinearLayout 的大小。如需了解详细信息，请参阅布局指南。

[android:hint](https://developer.android.google.cn/reference/android/widget/TextView.html#attr_android:hint)

这是文本字段为空时显示的默认字符串。"@string/edit_message" 并非使用硬编码字符串作为其值，而是引用另一个文件中定义的一个字符串资源。 由于它引用的是一个具体资源（而不仅仅是标识符），因此不需要加号。 不过，由于您尚未定义字符串资源，所以最初会出现编译错误。 在下一节中，您可以通过定义字符串来修正该错误。
	
	注：此字符串资源与元素 ID edit_message 同名。不过，对资源的引用始终都按资源类型（如 id 或 string）确定其作用域，因此使用相同的名称不会引起冲突。

##添加字符串资源

默认情况下，您的 Android 项目在 res > values > strings.xml 位置包含一个字符串资源文件。您需要在此处添加两个新字符串。

1.从 Project 窗口中，打开 res > values > strings.xml。

2.添加两个字符串，使您的文件如下所示：
		
	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	    <string name="app_name">My First App</string>
	    <string name="edit_message">Enter a message</string>
	    <string name="button_send">Send</string>
	</resources>

对于用户界面中的文本，务必将每个字符串都指定为资源。 字符串资源允许您在单一位置管理所有 UI 文本，从而简化文本的查找和更新。 此外，将字符串外部化还可让您为每个字符串资源提供替代定义，从而将您的应用本地化为不同的语言。

如需了解有关使用字符串资源将应用本地化为其他语言的详细信息，请参阅[支持不同设备](https://developer.android.google.cn/training/basics/supporting-devices/index.html)课程。

##添加按钮

返回到 activity_main.xml 文件并在 <EditText 后添加一个按钮。您的文件应如下所示：

	<LinearLayout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:orientation="horizontal"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
	        <EditText android:id="@+id/edit_message"
	          android:layout_width="wrap_content"
	          android:layout_height="wrap_content"
	          android:hint="@string/edit_message" />
	        <Button
	          android:layout_width="wrap_content"
	          android:layout_height="wrap_content"
	          android:text="@string/button_send" />
	</LinearLayout>

>注：此按钮不需要 android:id 属性，因为不需要从 Activity 代码中引用它。

如图 2 所示，该布局当前设计为根据内容大小调整 [EditText](https://developer.android.google.cn/reference/android/widget/EditText.html) 和 [Button](https://developer.android.google.cn/reference/android/widget/Button.html) 小部件的尺寸。

![](edittext_wrap.png)

图 2. EditText 小组件和 Button 小部件的宽度均设置为 "wrap_content"。

这对按钮很适合，但不太适合文本字段，因为用户键入的内容可能更长。 最好让文本字段填满未使用的屏幕宽度。 您可以在 LinearLayout 内通过 weight 属性来实现此目的，该属性可使用 android:layout_weight 属性来指定。

Weight 值是一个数字，用于指定每个视图与其他同级视图在剩余空间中的占比。 这有点像饮料配方中各种成分的比例： “2 份苏打、1 份糖浆”是指饮料中三分之二是苏打。例如，如果您将一个视图的 weight 值指定为 2，将另一个视图的 weight 值指定为 1，总和是 3，那么第一个视图将填满剩余空间的 2/3，而第二个视图则填满其余部分。 如果您添加了第三个视图，将其 weight 值指定为 1，那么现在第一个视图（weight 值为 2）将获得 1/2 的剩余空间，其余两个视图则各占 1/4。

所有视图的默认 weight 值都为 0，所以如果您仅将一个视图的 weight 值指定为大于 0，那么等到其他所有视图都获得所需空间后，该视图便会填满所有剩余空间。

##使输入框填满屏幕宽度

在 activity_main.xml 中，修改 <EditText ，使这些属性如下所示：

	<EditText android:id="@+id/edit_message"
	    android:layout_weight="1"
	    android:layout_width="0dp"
	    android:layout_height="wrap_content"
	    android:hint="@string/edit_message" />

将宽度设置为零 (0dp) 可提高布局性能，这是因为如果将宽度设置为 "wrap_content"，则会要求系统计算宽度，而该计算最终毫无意义，因为 weight 值还需要计算另一个宽度，才能填满剩余空间。

![](edittext_gravity.png)

图 3. EditText 小部件获得了布局的所有 weight，因此它填满了 LinearLayout 中的剩余空间。

完整的 activity_main.xml 布局文件现在看上去应该像下面这样：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	   xmlns:tools="http://schemas.android.com/tools"
	   android:orientation="horizontal"
	   android:layout_width="match_parent"
	   android:layout_height="match_parent">
	    <EditText android:id="@+id/edit_message"
	        android:layout_weight="1"
	        android:layout_width="0dp"
	        android:layout_height="wrap_content"
	        android:hint="@string/edit_message" />
	    <Button
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:text="@string/button_send" />
	</LinearLayout>

##运行您的应用

要查看应用目前在设备或模拟器上的外观，请点击工具栏中的 Run 。

要添加诸如响应按钮和启动另一个 Activity 等应用行为，请继续学习下一课。

>**备注**  
翻译:  [@jarylan](https://github.com/jarylan)  
原始文档:  [https://developer.android.com/training/basics/firstapp/building-ui.html](https://developer.android.com/training/basics/firstapp/building-ui.html)
