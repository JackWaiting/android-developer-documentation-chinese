# 

#创建 Fragment
你可以把 fragment 当做 activity 的一个组成模块，它有自己的生命周期并接收输入事件，当 activity 运行时可以进行添加或者删除(类似一个"子 activity"，可以在不同的activity 中重复使用)。这个课程主要说明如何继承 [Support Library](https://developer.android.com/topic/libraries/support-library/index.html) 中的 [Fragment](https://developer.android.com/reference/android/support/v4/app/Fragment.html) ，从而使 APP 可以兼容到系统为Android 1.6的低版本。    

在开始学习这个课程之前，必须在 Android 项目中先引用 Support Library。如果你之前未使用过Support Library,可以参考 [Support Library Setup](https://developer.android.com/topic/libraries/support-library/setup.html) 文档使用 v4 库。当然你也可以使用包含 [app bar](https://developer.android.com/training/appbar/index.html)的 **v7 appcompat** 库。该库兼容了 Android 2.1 (API level 7)，也包含了 [Fragment](https://developer.android.com/reference/android/support/v4/app/Fragment.html) API。

##创建 Fragment 类
创建一个 fragment时，需要继承 [Fragment](https://developer.android.com/reference/android/support/v4/app/Fragment.html) 类，然后结合 APP 的逻辑去重写关键的生命周期方法，[Activity](https://developer.android.com/reference/android/app/Activity.html) 也是类似这种处理方式。    
其中一个区别是：当创建一个 Fragment时，必须重写 [onCreateView()](https://developer.android.com/reference/android/support/v4/appp/Fragment.html#onCreateView(android.view.LayoutInflater, android.view.ViewGroup, android.os.Bundle)) 回调去定义布局，实际上这是唯一一个为了使 fragment 运行起来的回调。例如下面这个自定义布局的示例 fragment：

```java
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.ViewGroup;

public class ArticleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.article_view, container, false);
    }
}
```

和 Activity 一样，当 Fragment 从 Activity 添加或者移除、或 Activity 生命周期发生变化时，Fragment 通过生命周期回调函数管理其状态。例如，当 Activity 的 [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 被调用时，它内部所有 Fragment 的 onPause() 方法也会被触发。

更多关于 Fragment 的声明周期和回调方法，详见 [Fragments](https://developer.android.com/guide/components/fragments.html) 开发指南。

##使用 XML 在 Activity 中添加 Fragment    
fragments 是可重用、可模块化的 UI 组件，每一个 fragment 实例都必须关联一个父 [FragmentActivity](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html)，你可以在 Activity 的 XML 布局文件中逐个定义 Fragment 来实现这种关联。
>**注**: FragmentActivity 是 Support Library 提供的一种特殊 Activity，用于处理 API 11 版本以下的 Fragment。如果 APP 中的最低版本大于等于 11，则可以使用普通的 [Activity](https://developer.android.com/reference/android/app/Activity.html)。
 
 这里有一个布局文件的例子，即当手机屏幕属于“large”时（用目录名称中的 large 字符来区分），在一个 activity 中添加二个 fragment。    
 
res/layout-large/news_articles.xml
    
```xml
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="horizontal"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent">
 
    <fragment android:name="com.example.android.fragments.HeadlinesFragment"
              android:id="@+id/headlines_fragment"
              android:layout_weight="1"
              android:layout_width="0dp"
              android:layout_height="match_parent" />

    <fragment android:name="com.example.android.fragments.ArticleFragment"
              android:id="@+id/article_fragment"
              android:layout_weight="2"
              android:layout_width="0dp"
              android:layout_height="match_parent" />

    </LinearLayout>
```

>**提示**： 更多关于不同屏幕尺寸创建不同布局的信息，请阅读 [兼容不同屏幕尺寸](https://developer.android.com/training/multiscreen/screensizes.html)。    

然后将这个布局文件用到 Activity 中。

```java
import android.os.Bundle;
import android.support.v4.app.FragmentActivity;

public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_articles);
    }
}
```

如果使用 [v7 appcompat library](https://developer.android.com/tools/support-library/features.html#v7-appcompat)，应该使 activity 继承 [AppCompatActivity](https://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html)，它是 FragmentActivity 的子类。更多信息请参阅 [Adding the App Bar](https://developer.android.com/training/appbar/index.html)。

>**注**:当通过定义 XML 布局文件在 activity 中添加一个 fragment 时，你不能动态的移除 fragment。如果想在用户交互时对 fragment 进行切入和切出，必须在 activity 第一次启动时就把 fragment 添加进来，这些将在下节课讲解。
    
    
>翻译：[@misparking](https://github.com/misparking)    
原始文档：<https://developer.android.com/training/basics/fragments/creating.html>
