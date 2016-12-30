# 

## 保存键值对

如果你需要保存较小的键值对集合，那么你可以使用 [SharedPreferences](https://developer.android.google.cn/reference/android/content/SharedPreferences.html) 的 API。 每个 SharedPreferences 对象会对应一个键值对文件，并且提供了简单的方法去读写它们。每个 SharedPreferences 被 Framework 管理，能够设置为私有的或公开的。   

这一课将讲解用 SharedPreferences 的 API 如何存储和恢复简单类型的值。

> 注：SharedPreferences API 仅仅是用来读写键值对数据，请不要与 [Preference](https://developer.android.google.cn/reference/android/preference/Preference.html) API 混淆，它是用来构建应用设置界面的（尽管它会使用 SharedPreferences 来储存设置信息）。关于使用 Preference API 的信息，请参见 [Settings](https://developer.android.google.cn/guide/topics/ui/settings.html) 手册。   


### 获取 SharedPreferences 的实例

你可以创建一个新的 SharedPreferences 文件，或打开一个已经存在的文件，有两个方法可用：   

- [getSharedPreferences()](https://developer.android.google.cn/reference/android/content/Context.html#getSharedPreferences(java.lang.String,int)) —— 通过 SharedPreferences 的文件标识名来使用它，第一个参数表示它的文件名。你能在应用的任何 [Context](https://developer.android.google.cn/reference/android/content/Context.html) 中获取到这个方法。

- [getPreferences()](https://developer.android.google.cn/reference/android/app/Activity.html#getPreferences(int)) —— 使用它在 Activity 中，如果你仅需要使用这个 [Activity](https://developer.android.google.cn/reference/android/app/Activity.html) 的 SharedPreferences 文件。因为它默认会返回属于这个 Activity 的 SharedPreferences 文件，你不需要传入它的名字。   


例如：下面的代码是执行在 [Fragment](https://developer.android.google.cn/reference/android/app/Fragment.html) 中，它打开 SharedPreferences 文件的标识名是 string 资源文件里 *R.string.preference_file_key* 的值，并且是设置私有模式，所以这个文件仅仅只能在当前应用中调用。   

	Context context = getActivity();
	SharedPreferences sharedPref = context.getSharedPreferences(
	        getString(R.string.preference_file_key), Context.MODE_PRIVATE);

在命名 SharedPreferences 文件时，你应该使用在当前应用中的唯一的标识名，如：　　　

*"com.example.myapp.PREFERENCE_FILE_KEY"*　　　

另一种选择，你仅仅需要使用 Activity 中的 getPreferences() 方法来获取 SharedPreferences 文件：

	SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);

> 注意：如果你创建的 SharedPreferences 文件是 [MODE_WORLD_READABLE](https://developer.android.google.cn/reference/android/content/Context.html#MODE_WORLD_READABLE) 或 [MODE_WORLD_WRITEABLE](https://developer.android.google.cn/reference/android/content/Context.html#MODE_WORLD_WRITEABLE) 模式，那么任何一个其它的应用只要知道你的标识名都能访问数据。   


### 写 SharedPreferences 信息

在写 SharedPreferences 文件时，需要通过 [edit()](https://developer.android.google.cn/reference/android/content/SharedPreferences.html#edit()) 方法创建 [SharedPreferences.Editor](https://developer.android.google.cn/reference/android/content/SharedPreferences.Editor.html) 对象。   

通过 [putInt()](https://developer.android.google.cn/reference/android/content/SharedPreferences.Editor.html#putInt(java.lang.String,int)) 或 [putString()](https://developer.android.google.cn/reference/android/content/SharedPreferences.Editor.html#putString(java.lang.String,java.lang.String)) 等方法去写键值对数据，当调用 [commit()](https://developer.android.google.cn/reference/android/content/SharedPreferences.Editor.html#commit()) 方法时保存更改，如下：   

	SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
	SharedPreferences.Editor editor = sharedPref.edit();
	editor.putInt(getString(R.string.saved_high_score), newHighScore);
	editor.commit();


### 读 SharedPreferences 信息

从 SharedPreferences 文件中读取数据时，调用如 [getInt()](https://developer.android.google.cn/reference/android/content/SharedPreferences.html#getInt(java.lang.String,int)) 或 [getString()](https://developer.android.google.cn/reference/android/content/SharedPreferences.html#getString(java.lang.String,java.lang.String)) 等方法，传一个你需要值的键，再传一个默认的值作为该键不存在时结果，如：   

	SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
	int defaultValue = getResources().getInteger(R.string.saved_high_score_default);
	long highScore = sharedPref.getInt(getString(R.string.saved_high_score), defaultValue);




>翻译：[@iOnesmile](https://github.com/iOnesmile)    
原始文档：<https://developer.android.google.cn/training/basics/data-storage/shared-preferences.html#WriteSharedPreference>
