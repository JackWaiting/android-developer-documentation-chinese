# 打印HTML文档
在 Android 中打印内容超越一个简单的照片需要组合打印文本和图形的文档。Android 框架提供一种用 HTML 方式使用最少的代码去打印文档。

在Android 4.4(API level 19), [WebVie]() 类已更新开启打印 HTML 内容。这个类允许你去加载一个本地的 HTML 资源或者从网络下载的页面。创建一个打印任务,并把它传递给 Android 的打印服务。

本课程将向您展示如何快速构建包含文本和图形的HTML文档，并使用 WebView 进行打印。

## 加载HTML文档

使用 WebView 打印 HTML文档包含加载 HTML 资源或使用 HTML 文档的字符串。本节介绍如何构建 HTML 字符串并将其加载到 WebView 进行打印。

通常使用这个视图对象作为一个活动布局的一部分。然而,如果您的应用程序不使用 WebView,您可以创建类的实例专门为印刷的目的。创建自定义打印视图的主要步骤是:

1. 创建一个在加载 HTML 资源之后开始打印任务的 [WebViewClient]()
2. 将 HTML 资源加载 到 ebView 对象中

以下代码示例演示了如何创建一个简单 的 WebViewClient 并加载一个即时创建的 HTML 文档：

```
private WebView mWebView;

private void doWebViewPrint() {
    // Create a WebView object specifically for printing
    WebView webView = new WebView(getActivity());
    webView.setWebViewClient(new WebViewClient() {

            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                return false;
            }

            @Override
            public void onPageFinished(WebView view, String url) {
                Log.i(TAG, "page finished loading " + url);
                createWebPrintJob(view);
                mWebView = null;
            }
    });

    // Generate an HTML document on the fly:
    String htmlDocument = "<html><body><h1>Test Content</h1><p>Testing, " +
            "testing, testing...</p></body></html>";
    webView.loadDataWithBaseURL(null, htmlDocument, "text/HTML", "UTF-8", null);

    // Keep a reference to WebView object until you pass the PrintDocumentAdapter
    // to the PrintManager
    mWebView = webView;
}

```

>**NOTE:** 确保您在生成打印作业的调用发生在你在上一节中创建的 WebViewClient 的 [onPageFinished（）]() 方法中。 如果你不等待页面加载完成，打印输出可能不完整或空白，否则可能会完全失败。

>**NOTE:** 上面的示例代码包含 WebView 对象的实例，因此在创建打印作业之前不会垃圾回收。 确保你在自己的实现中执行相同操作，否则打印过程可能会失败。


如果要在页面中包含图形，请将图形文件放在项目的 assets /目录中，并在 [loadDataWithBaseURL（）]() 方法的第一个参数中指定一个基本 URL，如下面的代码示例所示：

```
webView.loadDataWithBaseURL("file:///android_asset/images/", htmlBody,
        "text/HTML", "UTF-8", null);
```

你还可以通过用 [loadUrl（）]() 替换loadDataWithBaseURL（）方法来加载要打印的网页，如下所示。


```
// Print an existing web page (remember to request INTERNET permission!):
webView.loadUrl("http://developer.android.com/about/index.html");
```

当使用 WebView 创建打印文档时，你应该注意以下限制：

* 你不能向文档添加页眉或页脚，包括页码。

* HTML 文档的打印选项不包括打印页面范围的功能，例如：不支持打印10页 HTML 文档中的第2页到第4页。

* WebView 的一个实例一次只能处理一个打印作业。不支持包含CSS打印属性（如风景属性）的 HTML 文档。

* 你不能在 HTML 文档中使用 JavaScript 来触发打印。

>**NOTE** 布局中包含的 WebView 对象的内容也可以在加载文档时打印。


## 创建打印作业
在创建 WebView 并加载 HTML 内容之后，您的应用程序几乎完成了其部分打印过程。 接下来的步骤是访问 [PrintManager]()，创建打印适配器，最后创建打印作业。


```
private void createWebPrintJob(WebView webView) {

    // Get a PrintManager instance
    PrintManager printManager = (PrintManager) getActivity()
            .getSystemService(Context.PRINT_SERVICE);

    // Get a print adapter instance
    PrintDocumentAdapter printAdapter = webView.createPrintDocumentAdapter();

    // Create a print job with name and adapter instance
    String jobName = getString(R.string.app_name) + " Document";
    PrintJob printJob = printManager.print(jobName, printAdapter,
            new PrintAttributes.Builder().build());

    // Save the job object for later status checking
    mPrintJobs.add(printJob);
}
```


此示例保存 [PrintJob]() 对象的实例以供应用程序使用，这不是必需的。您的应用程序可能会使用此对象来跟踪正在处理的打印作业的进度。当您想要在应用程序中监视打印作业的状态以完成，失败或用户取消时，此方法很有用。不需要创建应用内通知，因为打印框架会自动为打印作业创建系统通知。























