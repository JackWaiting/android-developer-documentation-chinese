# 创建灵活的 UI

当你的应用设计成支持较大范围的屏幕尺寸的时候，你可以通过不同布局配置的 Fragment 来优化基于可用的屏幕空间的用户体验。

例如，在一个手持设备上，针对一个单窗口 UI ，一次显示一个 Fragment ，可能是正确的。相反,你可以在一个拥有更宽屏幕尺寸的平板电脑上，并排的设置多个 Fragment，这样可以展示更多信息给用户。

![](fragments-screen-mock.png)

图 1.在相同 Activity 不同屏幕尺寸上，通过不同的配置来显示。在大屏幕上，两个 Fragment 并排展示，但是在手持设备上，一次只展示一个 Fragment，当用户到另外一个 Fragment 的时候，当前 Fragment 会被替换。

[FragmentManager](https://developer.android.com/reference/android/support/v4/app/FragmentManager.html) 类提供了往运行中的 Activity 中添加、移除、替换 Fragment 的方法，这些能够让你创建一个动态的体验。

## 在运行的 Activity 中添加 Fragment

除了[之前课程](https://developer.android.com/training/basics/fragments/creating.html)中提到的可以在 Activity 的布局文件中通过 `<fragment>` 元素来定义 Fragment 在 Activity 运行时添加一个 Fragment。如果你想要在 Activity 的生命周期中修改 Fragment，那么这个是必要的。

执行诸如添加和移除 Fragment 的事务，你应当使用 [FragmentManager](https://developer.android.com/reference/android/support/v4/app/FragmentManager.html) 来创建一个 [FragmentTransaction](https://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html) ，它提供了添加、移除、替换和执行其它 Fragment  事务的 API。

如果你的 Activity 允许移除和替换 Fragment，你应当在 Activity 的 [onCreate()](https://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 方法中，添加 Fragment 的初始化。

在处理 Fragment 的时候，尤其是在运行时添加 Fragment，有一个重要的规则，就是你的 Activity 的布局必须包含一个你能够插入一个 Fragment 的 [View](https://developer.android.com/reference/android/view/View.html) 容器。

以下布局是[之前课程](https://developer.android.com/training/basics/fragments/creating.html)中的一次添加一个 Fragment 的备选。为了替换 Fragment，Activity 中的布局包含了一个空的 [FrameLayout](https://developer.android.com/reference/android/widget/FrameLayout.html) ，作为 Fragment 的容器。

请注意，文件名和之前课程中讲的布局文件相同，但是这个布局目录没有 `large` 标识，所以这个布局是在设备屏幕尺寸小于 `large` 的时候使用，因为这种屏幕不能同时容下两个 Fragment。

`res/layout/news_articles.xml:`

``` xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

在你的 Activity 里面，通过使用支持库 API 调用 [getSupportFragmentManager()](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html#getSupportFragmentManager()) 方法来获得一个 [FragmentManager](https://developer.android.com/reference/android/support/v4/app/FragmentManager.html) 对象。然后调用 [beginTransaction()](https://developer.android.com/reference/android/support/v4/app/FragmentManager.html#beginTransaction()) 方法来创建一个 [FragmentTransaction](https://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html) 对象，最后通过调用 [add()](https://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#add(android.support.v4.app.Fragment, java.lang.String)) 方法来添加一个 Fragment。

你可以通过相同的 [FragmentTransaction](https://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html) 对象在 Activity 中执行多个 Fragment 事务。当你准备好修改的时候，你需要调用 [commit()](https://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#commit()) 方法。

For example, here's how to add a fragment to the previous layout:

例如，下面有一个例子展示如何在初始布局中添加一个 Fragment：

``` Java
import android.os.Bundle;
import android.support.v4.app.FragmentActivity;

public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_articles);

        // Check that the activity is using the layout version with
        // the fragment_container FrameLayout
        if (findViewById(R.id.fragment_container) != null) {

            // However, if we're being restored from a previous state,
            // then we don't need to do anything and should return or else
            // we could end up with overlapping fragments.
            if (savedInstanceState != null) {
                return;
            }

            // Create a new Fragment to be placed in the activity layout
            HeadlinesFragment firstFragment = new HeadlinesFragment();

            // In case this activity was started with special instructions from an
            // Intent, pass the Intent's extras to the fragment as arguments
            firstFragment.setArguments(getIntent().getExtras());

            // Add the fragment to the 'fragment_container' FrameLayout
            getSupportFragmentManager().beginTransaction()
                    .add(R.id.fragment_container, firstFragment).commit();
        }
    }
}
```
因为这个 Fragment 是在运行时添加到 FrameLayout 容器中的，而不是通过 Activity 的布局中的 `<fragment>` 元素添来定义的，所以这个 Activity 可以通过替换当前 Fragment 来追加新的 Fragment。

## Fragment 的替换

Fragment 的替换流程和添加流程差不多，只是不是调用 [add()](https://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#add(android.support.v4.app.Fragment, java.lang.String)) 方法，而是 [replace()](https://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#replace(int, android.support.v4.app.Fragment)) 方法。

Keep in mind that when you perform fragment transactions, such as replace or remove one, it's often appropriate to allow the user to navigate backward and "undo" the change. To allow the user to navigate backward through the fragment transactions, you must call addToBackStack() before you commit the FragmentTransaction.

请记住，当你执行 Fragment 事务的时候，例如替换和移除一个 Fragment，通常采取的正确方法是允许用户向后或者撤销操作。为了允许用户向后操作，你应当在你提交 [FragmentTransaction](https://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html) 之前，调用 [addToBackStack()](https://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#addToBackStack(java.lang.String)) 方法。

> **备注**：当你移除或者替换一个 Fragment 的，并且是在回退栈中添加了事务的话，这个 Fragment 是被移除和停止了（但是并没有被销毁掉）。当用户回退到这个 Fragment 的时候，它有重启了。如果你没有将这个事务添加到回退栈中，这个 Fragment 在被移除或者替换的时候，被销毁了。

以下是 Fragment 替换的例子：

``` Java
// Create fragment and give it an argument specifying the article it should show
ArticleFragment newFragment = new ArticleFragment();
Bundle args = new Bundle();
args.putInt(ArticleFragment.ARG_POSITION, position);
newFragment.setArguments(args);

FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();

// Replace whatever is in the fragment_container view with this fragment,
// and add the transaction to the back stack so the user can navigate back
transaction.replace(R.id.fragment_container, newFragment);
transaction.addToBackStack(null);

// Commit the transaction
transaction.commit();
```

[addToBackStack()](https://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#addToBackStack(java.lang.String)) 方法提供了一个可选的 String 参数来指定事务的唯一名字。这个名字除非你通过 [FragmentManager.BackStackEntry](https://developer.android.com/reference/android/support/v4/app/FragmentManager.BackStackEntry.html) 的 API 来执行更高级的 Fragment 的操作，否则你是不需要的。


备注：  
翻译：[@ifeegoo](https://github.com/ifeegoo)  
原始文档：https://developer.android.com/training/basics/fragments/fragment-ui.html


































