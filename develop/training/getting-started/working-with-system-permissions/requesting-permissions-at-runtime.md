#在运行时请求权限

从 Android 6.0（API 级别 23）开始，用户开始在应用运行时向其授予权限，而不是在应用安装时授予。此方法可以简化应用安装过程，因为用户在安装或更新应用时不需要授予权限。它还让用户可以对应用的功能进行更多控制；例如，用户可以选择为相机应用提供相机访问权限，而不提供设备位置的访问权限。用户可以随时进入应用的“Settings”屏幕调用权限。

系统权限分为两类：正常权限和危险权限：  

* 正常权限不会直接给用户隐私权带来风险。如果你的应用在其清单中列出了正常权限，系统将自动授予该权限。  
* 危险权限会授予应用访问用户机密数据的权限。如果你的应用在其清单中列出了正常权限，系统将自动授予该权限。如果你列出了危险权限，则用户必须明确批准你的应用使用这些权限。
  
如需了解详细信息，请参阅[正常权限和危险权限](https://developer.android.google.cn/guide/topics/security/permissions.html#normal-dangerous)。

在所有版本的 Android 中，你的应用都需要在其应用清单中同时声明它需要的正常权限和危险权限，如[声明权限](https://developer.android.google.cn/training/permissions/declaring.html)中所述。不过，该声明的影响因系统版本和应用的目标 SDK 级别的不同而有所差异：

* 如果设备运行的是 Android 5.1 或更低版本，或者应用的目标 SDK 为 22 或更低：如果你在清单中列出了危险权限，则用户必须在安装应用时授予此权限；如果他们不授予此权限，系统根本不会安装应用。
* 如果设备运行的是 Android 6.0 或更高版本，或者应用的目标 SDK 为 23 或更高：应用必须在清单中列出权限，并且它必须在运行时请求其需要的每项危险权限。用户可以授予或拒绝每项权限，且即使用户拒绝权限请求，应用仍可以继续运行有限的功能。

>**注：**从 Android 6.0（API 级别 23）开始，用户可以随时从任意应用调用权限，即使应用面向较低的 API 级别也可以调用。无论你的应用面向哪个 API 级别，你都应对应用进行测试，以验证它在缺少需要的权限时行为是否正常。

##检查权限
如果你的应用需要危险权限，则每次执行需要这一权限的操作时你都必须检查自己是否具有该权限。用户始终可以自由调用此权限，因此，即使应用昨天使用了相机，它不能假设自己今天仍具有该权限。

要检查你是否具有某项权限，请调用 [ContextCompat.checkSelfPermission()](https://developer.android.google.cn/reference/android/support/v4/content/ContextCompat.html#checkSelfPermission) 方法。例如，以下代码段显示了如何检查 Activity 是否具有在日历中进行写入的权限：

	// Assume thisActivity is the current activity
	int permissionCheck = ContextCompat.checkSelfPermission(thisActivity,
	        Manifest.permission.WRITE_CALENDAR);

如果应用具有此权限，方法将返回 [PackageManager.PERMISSION_GRANTED](https://developer.android.google.cn/reference/android/content/pm/PackageManager.html#PERMISSION_GRANTED)，并且应用可以继续操作。如果应用不具有此权限，方法将返回 [PERMISSION_DENIED](https://developer.android.google.cn/reference/android/content/pm/PackageManager.html#PERMISSION_DENIED)，且应用必须明确向用户要求权限。

##请求权限
如果你的应用需要应用清单中列出的危险权限，那么，它必须要求用户授予该权限。Android 为你提供了多种权限请求方式。调用这些方法将显示一个标准的 Android 对话框，不过，你不能对它们进行自定义。

####解释应用为什么需要权限

在某些情况下，你可能需要帮助用户了解你的应用为什么需要某项权限。例如，如果用户启动一个摄影应用，用户对应用要求使用相机的权限可能不会感到吃惊，但用户可能无法理解为什么此应用想要访问用户的位置或联系人。在请求权限之前，不妨为用户提供一个解释。请记住，你不需要通过解释来说服用户；如果你提供太多解释，用户可能发现应用令人失望并将其移除。

你可以采用的一个方法是仅在用户已拒绝某项权限请求时提供解释。如果用户继续尝试使用需要某项权限的功能，但继续拒绝权限请求，则可能表明用户不理解应用为什么需要此权限才能提供相关功能。对于这种情况，比较好的做法是显示解释。

为了帮助查找用户可能需要解释的情形，Android 提供了一个实用程序方法，即 [shouldShowRequestPermissionRationale()](https://developer.android.google.cn/reference/android/support/v4/app/ActivityCompat.html#shouldShowRequestPermissionRationale)。如果应用之前请求过此权限但用户拒绝了请求，此方法将返回 true。

>**注：**如果用户在过去拒绝了权限请求，并在权限请求系统对话框中选择了 Don't ask again 选项，此方法将返回 false。如果设备规范禁止应用具有该权限，此方法也会返回 false。

##请求你需要的权限

如果应用尚无所需的权限，则应用必须调用一个 [requestPermissions()](https://developer.android.google.cn/reference/android/support/v4/app/ActivityCompat.html#requestPermissions) 方法，以请求适当的权限。应用将传递其所需的权限，以及你指定用于识别此权限请求的整型请求代码。此方法异步运行：它会立即返回，并且在用户响应对话框之后，系统会使用结果调用应用的回调方法，将应用传递的相同请求代码传递到 requestPermissions()。

以下代码可以检查应用是否具备读取用户联系人的权限，并根据需要请求该权限：

	// Here, thisActivity is the current activity
	if (ContextCompat.checkSelfPermission(thisActivity,
	                Manifest.permission.READ_CONTACTS)
	        != PackageManager.PERMISSION_GRANTED) {
	
	    // Should we show an explanation?
	    if (ActivityCompat.shouldShowRequestPermissionRationale(thisActivity,
	            Manifest.permission.READ_CONTACTS)) {
	
	        // Show an expanation to the user *asynchronously* -- don't block
	        // this thread waiting for the user's response! After the user
	        // sees the explanation, try again to request the permission.
	
	    } else {
	
	        // No explanation needed, we can request the permission.
	
	        ActivityCompat.requestPermissions(thisActivity,
	                new String[]{Manifest.permission.READ_CONTACTS},
	                MY_PERMISSIONS_REQUEST_READ_CONTACTS);
	
	        // MY_PERMISSIONS_REQUEST_READ_CONTACTS is an
	        // app-defined int constant. The callback method gets the
	        // result of the request.
	    }
	}

>**注：**当你的应用调用 requestPermissions() 时，系统将向用户显示一个标准对话框。你的应用无法配置或更改此对话框。如果你需要为用户提供任何信息或解释，你应在调用 requestPermissions() 之前进行，如[解释应用为什么需要权限](https://developer.android.google.cn/training/permissions/requesting.html#explain)中所述。

##处理权限请求响应

当应用请求权限时，系统将向用户显示一个对话框。当用户响应时，系统将调用应用的 [onRequestPermissionsResult()](https://developer.android.google.cn/reference/android/support/v4/app/ActivityCompat.OnRequestPermissionsResultCallback.html#onRequestPermissionsResult) 方法，向其传递用户响应。你的应用必须替换该方法，以了解是否已获得相应权限。回调会将你传递的相同请求代码传递给 requestPermissions()。例如，如果应用请求 [READ_CONTACTS](https://developer.android.google.cn/reference/android/Manifest.permission.html#READ_CONTACTS) 访问权限，则它可能采用以下回调方法：

	@Override
	public void onRequestPermissionsResult(int requestCode,
	        String permissions[], int[] grantResults) {
	    switch (requestCode) {
	        case MY_PERMISSIONS_REQUEST_READ_CONTACTS: {
	            // If request is cancelled, the result arrays are empty.
	            if (grantResults.length > 0
	                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
	
	                // permission was granted, yay! Do the
	                // contacts-related task you need to do.
	
	            } else {
	
	                // permission denied, boo! Disable the
	                // functionality that depends on this permission.
	            }
	            return;
	        }
	
	        // other 'case' lines to check for other
	        // permissions this app might request
	    }
	}

系统显示的对话框说明了你的应用需要访问的[权限组](https://developer.android.google.cn/guide/topics/security/permissions.html#perm-groups)；它不会列出具体权限。例如，如果你请求 [READ_CONTACTS](https://developer.android.google.cn/reference/android/Manifest.permission.html#READ_CONTACTS) 权限，系统对话框只显示你的应用需要访问设备的联系人。用户只需要为每个权限组授予一次权限。如果你的应用请求该组中的任何其他权限（已在你的应用清单中列出），系统将自动授予应用这些权限。当你请求此权限时，系统会调用你的 [onRequestPermissionsResult()](https://developer.android.google.cn/reference/android/support/v4/app/ActivityCompat.OnRequestPermissionsResultCallback.html#onRequestPermissionsResult) 回调方法，并传递 [PERMISSION_GRANTED](https://developer.android.google.cn/reference/android/content/pm/PackageManager.html#PERMISSION_GRANTED)，如果用户已通过系统对话框明确同意你的权限请求，系统将采用相同方式操作。

>**注：**你的应用仍需要明确请求其需要的每项权限，即使用户已向应用授予该权限组中的其他权限。此外，权限分组在将来的 Android 版本中可能会发生变化。你的代码不应依赖特定权限属于或不属于相同组这种假设。

例如，假设你在应用清单中列出了 [READ_CONTACTS](https://developer.android.google.cn/reference/android/Manifest.permission.html#READ_CONTACTS) 和 [WRITE_CONTACTS](https://developer.android.google.cn/reference/android/Manifest.permission.html#WRITE_CONTACTS)。如果你请求 READ_CONTACTS 且用户授予了此权限，那么，当你请求 WRITE_CONTACTS 时，系统将立即授予你该权限，不会与用户交互。

如果用户拒绝了某项权限请求，你的应用应采取适当的操作。例如，你的应用可能显示一个对话框，解释它为什么无法执行用户已经请求但需要该权限的操作。

当系统要求用户授予权限时，用户可以选择指示系统不再要求提供该权限。这种情况下，无论应用在什么时候使用 [requestPermissions()](https://developer.android.google.cn/reference/android/support/v4/app/ActivityCompat.html#requestPermissions) 再次要求该权限，系统都会立即拒绝此请求。系统会调用你的 [onRequestPermissionsResult()](https://developer.android.google.cn/reference/android/support/v4/app/ActivityCompat.OnRequestPermissionsResultCallback.html#onRequestPermissionsResult) 回调方法，并传递 [PERMISSION_DENIED](https://developer.android.google.cn/reference/android/content/pm/PackageManager.html#PERMISSION_DENIED)，如果用户再次明确拒绝了你的请求，系统将采用相同方式操作。这意味着当你调用 [requestPermissions()](https://developer.android.google.cn/reference/android/support/v4/app/ActivityCompat.html#requestPermissions) 时，你不能假设已经发生与用户的任何直接交互。

>**备注:**  
>翻译 ：[@jarylan](https://github.com/jarylan)  
>原始文档:[https://developer.android.com/training/permissions/requesting.html](https://developer.android.com/training/permissions/requesting.html)
