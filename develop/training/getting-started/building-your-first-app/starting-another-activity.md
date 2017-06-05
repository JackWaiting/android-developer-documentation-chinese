# 启动另一个 Activity

完成上一课的学习后，您已构建了一个显示一个 Activity（单一屏幕）并带有一个文本字段和一个按钮的应用。 在本课中，您将为 MainActivity 添加一些代码，这些代码可在用户点击“Send”按钮时启动一个新的 Activity。

##响应 Send 按钮
1、在文件 res > layout > activity_main.xml 中，将 android:onClick 属性添加到
Button 元素，如下所示：

		<Button
      		android:layout_width="wrap_content"
      		android:layout_height="wrap_content"
      		android:text="@string/button_send"
      		android:onClick="sendMessage" />

    
每次用户点击按钮时，此属性均会提示系统调用 Activity 中的 sendMessage() 方法。

2、在文件 java > com.example.myfirstapp > MainActivity.java 中，添加 sendMessage() 方法存根，如下所示：
	
		public class MainActivity extends AppCompatActivity {
    		@Override
    		protected void onCreate(Bundle savedInstanceState) {
        		super.onCreate(savedInstanceState);
        		setContentView(R.layout.activity_main);
   			}

		    /** Called when the user clicks the Send button */
		    public void sendMessage(View view) {
		        // Do something in response to button
		    }
		}

要让系统将此方法与为 android:onClick 指定的方法名称匹配，签名必须与所示内容完全相同。具体而言，该方法必须：



- 是公共方法

- 具有空返回值

- 以 View 作为唯一参数（这将是之前点击的 View）

接下来，您需要填写此方法以读取文本字段的内容，并将该文本传递给另一个 Activity。

##构建一个 Intent
Intent 是指在相互独立的组件（如两个 Activity）之间提供运行时绑定功能的对象。Intent 表示一个应用“执行某项操作的 Intent”。 您可以将 Intent 用于各种任务，但在本课程中，Intent 用于启动另一个 Activity。

在 MainActivity.java 中，将如下所示代码添加到 sendMessage()：

		public class MainActivity extends AppCompatActivity {
		    public final static String EXTRA_MESSAGE = "com.example.myfirstapp.MESSAGE";
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_main);
		    }
		
		    /** Called when the user clicks the Send button */
		    public void sendMessage(View view) {
		        Intent intent = new Intent(this, DisplayMessageActivity.class);
		        EditText editText = (EditText) findViewById(R.id.edit_message);
		        String message = editText.getText().toString();
		        intent.putExtra(EXTRA_MESSAGE, message);
		        startActivity(intent);
		    }
		}
Android Studio 将显示**无法解析符号**错误，这是因为此代码引用了未导入的类。 通过按 Alt + Enter（在 Mac 上，则按 Option + Return），您可使用 Android Studio 的“导入类”功能来解决其中一些问题。您的导入应按如下所示结束：

		import android.content.Intent;
		import android.support.v7.app.AppCompatActivity;
		import android.os.Bundle;
		import android.view.View;
		import android.widget.EditText;
仍出现 **DisplayMessageActivity** 错误，但没关系，您将在下一部分中修复该错误。

sendMessage() 中会执行许多操作，因此我们来解释下具体会执行哪些操作。

Intent 构造函数采用两个参数：

- Context 是第一个参数（之所以使用 this ，是因为 Activity 类是 Context 的子类）

- 应用组件的 Class，系统应将 Intent（在本例中，为应启动的 Activity）传递至该类。

	注：引用 DisplayMessageActivity 会在 Android Studio 中引发错误，因为该类尚不存在。可以暂时忽略该错误，因为您很快要创建该类。

putExtra() 方法将 EditText 的值添加到 Intent。Intent 能够以名为 extra 的键值对形式携带数据类型。您的键是一个公共常量 EXTRA_MESSAGE，因为下一个 Activity 将使用该键来检索文本值。为 Intent extra 定义键时最好使用应用的软件包名称作为前缀。这可以确保在您的应用与其他应用交互过程中这些键始终保持唯一。

startActivity() 方法将启动 Intent 指定的 DisplayMessageActivity 实例。现在，您需要创建类。

##创建第二个 Activity

1. 在 Project 窗口中，右键点击 app 文件夹并选择 New > Activity > Empty Activity。

2. 在 Configure Activity 窗口中，为 Activity Name 输入 “DisplayMessageActivity”，然后点击 Finish

Android Studio 自动执行三项操作：

- 使用必需的 onCreate() 方法的实现创建类 DisplayMessageActivity.java。

- 创建对应的布局文件 activity_display_message.xml

- 在 AndroidManifest.xml 中添加必需的 <activity> 元素。

如果运行应用并在第一个 Activity 上点击“Send”按钮，则将启动第二个 Activity，但它为空。这是因为第二个 Activity 使用模板提供的默认空布局。

###显示消息

现在，您将修改第二个 Activity，以显示第一个 Activity 传递的消息。


1. 在 DisplayMessageActivity.java 中，向 onCreate() 方法添加下列代码：

		@Override
		protected void onCreate(Bundle savedInstanceState) {
		   super.onCreate(savedInstanceState);
		   setContentView(R.layout.activity_display_message);
		
		   Intent intent = getIntent();
		   String message = intent.getStringExtra(MainActivity.EXTRA_MESSAGE);
		   TextView textView = new TextView(this);
		   textView.setTextSize(40);
		   textView.setText(message);
		
		   ViewGroup layout = (ViewGroup) findViewById(R.id.activity_display_message);
		   layout.addView(textView);
		}

按 Alt + Enter（在 Mac 上，则按 Option + Return）导入缺少的类。 您的导入应按如下所示方式结束：

		import android.content.Intent;
		import android.support.v7.app.AppCompatActivity;
		import android.os.Bundle;
		import android.view.ViewGroup;
		import android.widget.TextView;

此处会执行大量操作，因此我们接着解释：

1. 调用 getIntent() 采集启动 Activity 的 intent。无论用户如何导航到目的地，每个 Activity 都由一个 Intent 调用。 调用 getStringExtra() 将检索第一个 Activity 中的数据。


1. 您可以编程方式创建 TextView 并设置其大小和消息。

1. 您可将 TextView 添加到 R.id.activity_display_message 标识的布局。您可将布局投射到 ViewGroup，因为它是所有布局的超类且包含 addView() 方法。

**注：早期版本的 Android Studio 生成的 XML 布局可能不包括 android:id 属性。如果布局没有 android:id 属性，则调用 findViewById() 将失败。如果出现这种情况，请打开
activity_display_message.xml 并将属性 android:id="@+id/activity_display_message" 添加到布局元素。**

您现在便可运行该应用。当应用打开后，请在文本字段键入消息，然后点击 Send。第二个 Activity 将替换屏幕中的第一个 Activity，并显示您在第一个 Activity 中输入的消息。

就这么简单，您的第一个 Android 应用已成功诞生！ 


备注：

翻译：JackWaiting[
[https://github.com/JackWaiting](https://github.com/JackWaiting "JackWaiting")
]
原始文：[https://developer.android.com/training/basics/firstapp/starting-activity.html#RespondToButton](https://developer.android.com/training/basics/firstapp/starting-activity.html#RespondToButton "https://developer.android.com/training/basics/firstapp/starting-activity.html#RespondToButton")

aaaaaaaa
