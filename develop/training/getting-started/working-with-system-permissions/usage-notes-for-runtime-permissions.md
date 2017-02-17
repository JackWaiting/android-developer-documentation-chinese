
#权限使用说明

应用程序很容易通过权限请求来影响用户。如果用户发现应用程序使用起来令人沮丧，或者用户担心应用程序可能正在处理用户的信息，他们可能会避免使用应用程序或完全卸载它。以下最佳做法可以帮助您避免这种糟糕的用户体验。

##考虑使用意图

在许多情况下，您可以在应用程序执行任务的两种方式之间进行选择。您可以让您的应用程序请求执行操作本身的权限。或者，您可以让应用程序使用意图让另一个应用程序执行任务。

例如，假设您的应用程序需要能够使用设备相机拍摄照片。您的应用程式可以要求相机权限，让您的应用程式直接存取[相机](https://developer.android.com/reference/android/Manifest.permission.html#CAMERA)。然后，您的应用程序将使用相机APIs控制相机并拍摄照片。这种方法使您的应用程序完全控制摄影过程，并允许您将相机用户界面合并到您的应用程序。

但是，如果您不需要此类完全控制，则可以使用[ACTION_IMAGE_CAPTURE](https://developer.android.com/reference/android/provider/MediaStore.html#ACTION_IMAGE_CAPTURE)意图请求图像。当您发送意图时，系统提示用户选择相机应用程序（如果没有默认的相机应用程序）。用户使用所选的相机应用程序拍摄照片，该应用程序将该图片返回到应用程序的[onActivityResult（）](https://developer.android.com/reference/android/app/Activity.html#onActivityResult)方法。

同样，如果您需要拨打电话，访问用户的联系人等，您可以通过创建相应的意图来完成此操作，也可以直接请求权限并访问相应的对象。每种方法都有优点和缺点。

如果您使用权限：

- 当您执行操作时，您的应用程序可完全控制用户体验。然而，这种广泛的控制增加了你的任务的复杂性，因为你需要设计一个合适的UI。

- 系统会提示用户在运行时或安装时授予一次权限（具体取决于用户的Android版本）。之后，您的应用程序可以执行操作，而不需要用户的额外交互。但是，如果用户未授予权限（或稍后撤消该权限），您的应用程序将无法完成任何操作。

如果你使用意图：

- 您不必为操作设计UI。处理意图的应用程序提供了UI。但是，这意味着您无法控制用户体验。用户可以与您从未见过的应用程序进行交互。

- 如果用户没有操作的默认应用程序，则系统提示用户选择应用程序。如果用户未指定默认处理程序，则他们可能必须在每次执行操作时都通过一个额外的对话框。

##只需要您需要的权限

每次你请求一个权限，你强制用户做出决定。您应该尽量减少发出这些请求的次数。如果用户运行的是Android 6.0（API级别23）或更高版本，则每次用户尝试一些需要权限的新应用程序功能时，应用程序必须通过权限请求中断用户的工作。如果用户运行的是较早版本的Android，则用户必须在安装应用程序时授予每个应用程序的权限;如果列表太长或似乎不合适，用户可能决定不安装您的应用程序。出于这些原因，您应该最小化您的应用需要的权限的数量。

通常，您的应用程序可以避免通过使用意图请求权限。如果某个功能不是应用程序功能的核心部分，您应该考虑将工作交给另一个应用程序，如考虑使用[意图中所述](https://developer.android.com/training/permissions/usage-notes.html#perms-vs-intents)。

##不要影响用户

如果用户运行的是Android 6.0（API级别23）或更高版本，则用户必须在运行应用程序时授予其权限。如果您同时面对许多权限请求，您可能会对用户造成压力，导致他们退出您的应用。相反，您应该在需要时请求权限。

在某些情况下，一个或多个权限可能对您的应用程序是绝对必要的。应用程式启动时请求所有权限可能有意义。例如，如果您制作了一个摄影应用程序，该应用程序将需要访问设备的相机。当用户第一次启动应用程序时，他们不会惊讶地被要求使用相机的权限。但如果同一个应用程序也有一个功能来与用户的联系人共享照片，你可能不应该在第一次启动时请求[READ_CONTACTS](https://developer.android.com/reference/android/Manifest.permission.html#READ_CONTACTS)权限。相反，请等到用户尝试使用“共享”功能，然后请求权限。

如果您的应用提供教程，在教程序列结束时请求应用程序的基本权限可能是有意义的。

##解释为什么你需要权限

当您调用[requestPermissions（）](https://developer.android.com/reference/android/support/v4/app/ActivityCompat.html#requestPermissions)时系统显示的权限对话框说明您的应用程序需要什么权限，但不说为什么。在一些情况下，用户可能发现困惑。在调用[requestPermissions（）](https://developer.android.com/reference/android/support/v4/app/ActivityCompat.html#requestPermissions)之前，向用户解释为什么您的应用程序需要权限是一个好主意。

例如，摄影应用可能需要使用位置服务，因此可以对照片进行地理标记。典型的用户可能不知道照片可以包含位置信息，并且会困惑他们的摄影应用想要知道位置的原因。所以在这种情况下，应用程序在调用[requestPermissions（）](https://developer.android.com/reference/android/support/v4/app/ActivityCompat.html#requestPermissions)之前告诉用户此功能是一个好主意。

通知用户的一种方法是将这些请求合并到应用教程中。教程可以依次显示每个应用程序的功能，因为它可以解释需要什么权限。例如，摄影应用程序的教程可以演示其“与您的联系人共享照片”功能，然后告诉用户他们需要授予应用程序的权限才能查看用户的联系人。然后，应用程序可以调用[requestPermissions（）](https://developer.android.com/reference/android/support/v4/app/ActivityCompat.html#requestPermissions)来请求用户访问该访问。当然，并不是每个用户都会遵循教程，因此您仍然需要在应用程序正常操作期间检查并请求权限。

##测试两个权限模型

从Android 6.0（API级别23）开始，用户在运行时授予和撤销应用权限，而不是在安装应用时执行此操作。因此，您必须在更广泛的条件下测试您的应用程式。在Android 6.0之前，你可以合理地假设，如果你的应用程序是在运行，它具有它在应用程序清单中声明的所有权限。在新的权限模式下，您不能再进行此假设。

以下提示将帮助您确定运行API级别23或更高版本的设备的与权限相关的代码问题：


- 确定您应用的当前权限和相关代码路径。

- 测试跨受权限保护的服务和数据的用户流。

- 使用授予或撤销权限的各种组合进行测试。例如，相机应用程序可能会在其清单中列出[CAMERA](https://developer.android.com/reference/android/Manifest.permission.html#CAMERA)，[READ_CONTACTS](https://developer.android.com/reference/android/Manifest.permission.html#READ_CONTACTS)和[ACCESS_FINE_LOCATION](https://developer.android.com/reference/android/Manifest.permission.html#ACCESS_FINE_LOCATION)。您应该测试应用程序的每个权限打开和关闭，以确保应用程序可以正常处理所有权限配置。请注意，从Android 6.0开始，用户可以为任何应用打开或关闭权限，即使是目标为API级别22或更低的应用。

- 使用[adb](https://developer.android.com/studio/command-line/adb.html)工具从命令行管理权限：

	
	按组列出权限和状态：
	
		$ adb shell pm list permissions -d -g
	
	授予或撤销一个或多个权限：
	
		$ adb shell pm [grant | revoke] <permission-name> ...
	

- 分析您的应用程序使用权限的服务。
 

