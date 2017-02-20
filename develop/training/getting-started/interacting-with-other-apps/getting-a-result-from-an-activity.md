#从 Activity 获取结果

启动另外一个 Activity 的方式，并不是单向的。你也可以启动另外一个 Activity ，并且接收一个返回结果。可以通过调用 [startActivityForResult()](https://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int))（而不是调用[startActivity()](https://developer.android.com/reference/android/app/Activity.html#startActivity(android.content.Intent))）。

例如，你的应用可以启动一个相机应用，并且接收一个诸如拍摄的照片的结果。你也可以启动一个通讯录应用来让用户选择一个联系人，然后返回一个联系人细节信息的结果。

当然，能够响应的 Activity 必须被设计成能够返回结果的。这样，它会将结果以 Intent 对象来返回。你的 Activity 在 [onActivityResult()](https://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent)) 回调方法中接收结果。

> **备注：**你可以在调用 [startActivityForResult()](https://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)) ，可以使用显式或引式 Intent。当启动一个你自己的 Activity 来接收一个结果的时候，你可以使用显式 Intent 来确保你可以接收到期望的结果。

##启动 Activity

当你启动一个 Activity 来获取一个结果的时候，对于 [Intent](https://developer.android.com/reference/android/content/Intent.html) 对象来说，没有什么特殊的，但是你需要传一个额外的整型参数给 [startActivityForResult()](https://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)) 方法。

这个整型参数是用来标记你请求的“请求码”。当你接收到一个 [Intent](https://developer.android.com/reference/android/content/Intent.html) 结果的时候，回调会提供相同的请求码，以便你的应用能够正确的区分结果和决定如何处理结果。

例如，下面是一个启动一个让用户选择一个联系人的 Activity 的例子：

```Java
static final int PICK_CONTACT_REQUEST = 1;  // The request code
...
private void pickContact() {
    Intent pickContactIntent = new Intent(Intent.ACTION_PICK, Uri.parse("content://contacts"));
    pickContactIntent.setType(Phone.CONTENT_TYPE); // Show user only contacts w/ phone numbers
    startActivityForResult(pickContactIntent, PICK_CONTACT_REQUEST);
}
```

##接收结果

当用户在随后的 Activity 中完成操作并且返回之后，系统会调用你的 Activity 的 [onActivityResult()](https://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent)) 方法。这个方法包含三个参数：  
* 你之前传给 [startActivityForResult()](https://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)) 方法的请求码。
* 通过第二个 Activity 指定的结果码。结果码要么是 [RESULT_OK](https://developer.android.com/reference/android/app/Activity.html#RESULT_OK)，表示操作成功。或者是 [RESULT_CANCELED](https://developer.android.com/reference/android/app/Activity.html#RESULT_CANCELED)，表示用户回退或者由于某些原因操作失败了。
* 携带结果数据的[Intent](https://developer.android.com/reference/android/content/Intent.html) 。

例如，下面是一个你可以处理“选择联系人” Intent 的结果：  
```Java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // Check which request we're responding to
    if (requestCode == PICK_CONTACT_REQUEST) {
        // Make sure the request was successful
        if (resultCode == RESULT_OK) {
            // The user picked a contact.
            // The Intent's data Uri identifies which contact was selected.

            // Do something with the contact here (bigger example below)
        }
    }
}
```

在这个例子中，由 Android 联系人应用返回的包含结果的 [Intent](https://developer.android.com/reference/android/content/Intent.html) 提供了可以标记用户选择的联系人的内容 [Uri](https://developer.android.com/reference/android/net/Uri.html)。

为了能够成功处理结果，你必须知道 [Intent](https://developer.android.com/reference/android/content/Intent.html) 结果的格式是什么。做到这一点，比较简单，只要 Acitivty 返回的结果是你自己 Activity 的。Android 平台的应用提供了他们自己的 API 能够让你处理指定的结果数据。例如，联系人应用总会返回一个带有能够识别所选联系人的内容 URI 的结果，相机应用会返回包含一个 [Bitmap](https://developer.android.com/reference/android/graphics/Bitmap.html) 对象的数据（参考类[拍摄照片](https://developer.android.com/training/camera/index.html)）。

###追加：读取联系人数据

上面的代码展示了如何从联系人应用中返回的结果中读取数据，而不必深入细节，因为这个要求更多关于 [Content Provider](https://developer.android.com/guide/topics/providers/content-providers.html) 的进一步的讨论。然而，如果你好奇的话，这里有进一步的代码教你如何从结果数据中获取到所选联系人的手机号码：  

```Java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // Check which request it is that we're responding to
    if (requestCode == PICK_CONTACT_REQUEST) {
        // Make sure the request was successful
        if (resultCode == RESULT_OK) {
            // Get the URI that points to the selected contact
            Uri contactUri = data.getData();
            // We only need the NUMBER column, because there will be only one row in the result
            String[] projection = {Phone.NUMBER};

            // Perform the query on the contact to get the NUMBER column
            // We don't need a selection or sort order (there's only one result for the given URI)
            // CAUTION: The query() method should be called from a separate thread to avoid blocking
            // your app's UI thread. (For simplicity of the sample, this code doesn't do that.)
            // Consider using CursorLoader to perform the query.
            Cursor cursor = getContentResolver()
                    .query(contactUri, projection, null, null, null);
            cursor.moveToFirst();

            // Retrieve the phone number from the NUMBER column
            int column = cursor.getColumnIndex(Phone.NUMBER);
            String number = cursor.getString(column);

            // Do something with the phone number...
        }
    }
}
```


>备注：在 Android 2.3 （API level 9）之前，查询联系人数据要求你的应用声明 [READ_CONTACTS](https://developer.android.com/reference/android/Manifest.permission.html#READ_CONTACTS) 的权限（参见[安全和权限](https://developer.android.com/guide/topics/security/security.html)）。然而从 Android 2.3 开始，联系人应用允许你的应用有一个读取联系人结果数据的临时权限。这个临时的权限之恩给你让你请求指定的联系人，所以当你没有声明 [READ_CONTACTS](https://developer.android.com/reference/android/Manifest.permission.html#READ_CONTACTS) 权限的时候，你就不能通过 Intent 中的 [Uri](https://developer.android.com/reference/android/net/Uri.html) 来查询超过一个联系人的数据。















