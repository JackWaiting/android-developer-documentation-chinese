#Fragments 之间的交互
为了重用 Fragment UI 组件，你应该建立一个完全独立的，模块化的组件，定义自己的布局和行为。一次定义可重用的 Fragments，你可以使其和 Activity 关联，并且在应用逻辑连接他们从而实现整体的复合界面。

通常你想让一个 Fragment 和另一个去交互，比如根据用户事件改变内容。所有 Fragment 和 Fragment 之间的交互是通过关联的 Activity 。两个 Fragments 不应该交互的。

##定义一个接口

允许 Fragment 和关联的 Activity 交互，你可以定义一个接口在 Fragment 类并且在 Activity 中实现这个接口。这个 Fragment 在生命周期 onAttch() 方法获得这个接口的实现，然后可以调用接口方法与Activity交互。

这里是一个 Framengt 和 Activity 交互的例子：
```java
public class HeadlinesFragment extends ListFragment {
    OnHeadlineSelectedListener mCallback;

    // Container Activity must implement this interface
    public interface OnHeadlineSelectedListener {
        public void onArticleSelected(int position);
    }

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);

        // This makes sure that the container activity has implemented
        // the callback interface. If not, it throws an exception
        try {
            mCallback = (OnHeadlineSelectedListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString()
                    + " must implement OnHeadlineSelectedListener");
        }
    }

    ...
}
```

现在这个 Fragment 可以使用 `OnHeadlineSelectedListener` 接口的实例 `mCallback` 去调用 `onArticleSelected()` 方法(或者接口中其他方法)传递消息。
例如，当用户单击列表项时，fragment 会调用下面的方法。这个 Fragment 使用回调接口去传递事件给父 Activity。
```java
@Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Send the event to the host activity
        mCallback.onArticleSelected(position);
    }
```
##实现这个接口

为了接收来自 Frament 的回调事件，activity 必须去实现这个在 Fragment 类中定义的接口。
例如，下面这个 Activity 实现了上面例子的接口：
```java
public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...

    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article
    }
}
```

##传递消息给一个 Fragment
这个主Activity可以通过 [findFragmentById](https://developer.android.google.cn/reference/android/support/v4/app/FragmentManager.html#findFragmentById(int)) 获得  [Fragment](https://developer.android.google.cn/reference/android/support/v4/app/Fragment.html) 实例去传递消息给这个 fragment。然后直接调用 fragment 中的公开方法。

例如，假设上面 activity 可能包含其他 fragment 用于去显示上面回调方法返回的数据在指定选项中。在这个情况，这个 activity 可以通过回调方法接受的消息传递给其他要显示选项的 fragment：
```java
public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...

    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article

        ArticleFragment articleFrag = (ArticleFragment)
                getSupportFragmentManager().findFragmentById(R.id.article_fragment);

        if (articleFrag != null) {
            // If article frag is available, we're in two-pane layout...

            // Call a method in the ArticleFragment to update its content
            articleFrag.updateArticleView(position);
        } else {
            // Otherwise, we're in the one-pane layout and must swap frags...

            // Create fragment and give it an argument for the selected article
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
        }
    }
}
```

>翻译：[@edtj](https://github.com/edtj)    
原始文档：<https://developer.android.google.cn/training/basics/fragments/communicating.html>
