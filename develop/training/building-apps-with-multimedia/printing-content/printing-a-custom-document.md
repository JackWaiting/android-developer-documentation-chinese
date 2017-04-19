# 打印自定义文档

对于某些应用程序，如绘图应用程序，页面布局应用程序和其他专注于图形输出的应用程序，创建漂亮的打印页面是一个关键功能。 在这种情况下，打印图像或 HTML 文档是不够的。 这些类型应用程序的打印输出需要精确控制进入页面的所有内容，包括字体，文本流，分页符，页眉，页脚和图形元素。

为你的应用创建完全定制的打印输出需要比以前讨论的方法更多的编程投入。 您必须构建与打印框架通信的组件，适应打印机设置，绘制页面元素并管理多页打印。

本课程将向您展示如何连接打印管理器，创建打印适配器并构建打印内容。

## 连接到打印管理器 

当你的应用程序直接管理打印过程时，从用户收到打印请求之后的第一步是连接到 Android 打印框架并获取 PrintManager 类的实例。 该类允许你初始化打印作业并开始打印生命周期。 以下代码示例显示如何获取打印管理器并开始打印过程。

	private void doPrint() {
    	// Get a PrintManager instance
    	PrintManager printManager = (PrintManager) getActivity()
            .getSystemService(Context.PRINT_SERVICE);

    	// Set job name, which will be displayed in the print queue
    	String jobName = getActivity().getString(R.string.app_name) + " Document";

    	// Start a print job, passing in a PrintDocumentAdapter implementation
   		// to handle the generation of a print document
    	printManager.print(jobName, new MyPrintDocumentAdapter(getActivity()),
            null); //
	}

以上示例代码演示如何命名打印作业并设置 PrintDocumentAdapter 类的实例，该实例处理打印生命周期的步骤。下一节将讨论打印适配器类的实现。

> 注意：print（）方法中的最后一个参数会使用一个PrintAttributes对象。您可以使用此参数为打印框架提供提示，并根据先前的打印周期预设选项，从而改善用户体验。您也可以使用此参数来设置更适合于正在打印的内容的选项，例如在打印方向的照片时将方向设置为横向。

## 创建打印适配器

打印适配器与Android打印框架进行交互，并处理打印过程的步骤。此过程需要用户在创建要打印的文档之前选择打印机和打印选项。当用户选择具有不同输出功能，不同页面大小或不同页面方向的打印机时，这些选择可以影响最终输出。当进行这些选择时，打印框架要求您的适配器布局并生成打印文档，以准备最终输出。一旦用户点击打印按钮，框架将使用最终的打印文档并将其传递给打印提供者进行输出。在打印过程中，用户可以选择取消打印动作，因此您的打印适配器还必须收听取消请求并作出反应。

PrintDocumentAdapter抽象类旨在处理打印生命周期，它具有四个主要的回调方法。您必须在打印适配器中实现这些方法，才能与打印框架正常交互：

- onStart（） 在打印过程开始时调用一次。如果您的应用程序可以执行任何一次性的准备任务，例如获取要打印的数据的快照，请在此处执行。不需要在适配器中实现此方法。

- onLayout（） - 每次用户更改影响输出的打印设置（例如不同的页面大小或页面方向）时，调用它们，使您的应用程序有机会计算要打印的页面的布局。至少，此方法必须返回打印文档中预期的页数。

- onWrite（） - 调用将打印页面复制到要打印的文件中。在每次onLayout（）调用之后，此方法可能会被调用一次或多次。

- onFinish（） - 在打印过程结束时调用一次。如果您的应用程序有任何一次执行任务，请在此处执行。不需要在适配器中实现此方法。


以下部分介绍如何实现布局和写入方法，这对打印适配器的功能至关重要。

> 注意：这些适配器方法在应用程序的主线程上调用。如果您期望在执行中执行这些方法花费大量时间，请执行它们在单独的线程中执行。例如，您可以将布局或打印文档编写工作封装在单独的AsyncTask对象中。

## 计算打印文档信息

在PrintDocumentAdapter类的实现中，您的应用程序必须能够指定正在创建的文档的类型，并计算打印作业的总页数（给定有关打印页面大小的信息）。 在适配器中实现onLayout（）方法进行这些计算，并提供有关PrintDocumentInfo类中打印作业的预期输出的信息，包括页数和内容类型。 以下代码示例显示了PrintDocumentAdapter的onLayout（）方法的基本实现：

	@Override
	public void onLayout(PrintAttributes oldAttributes,
	                     PrintAttributes newAttributes,
	                     CancellationSignal cancellationSignal,
	                     LayoutResultCallback callback,
	                     Bundle metadata) {
	    // Create a new PdfDocument with the requested page attributes
	    mPdfDocument = new PrintedPdfDocument(getActivity(), newAttributes);
	
	    // Respond to cancellation request
	    if (cancellationSignal.isCancelled() ) {
	        callback.onLayoutCancelled();
	        return;
	    }
	
	    // Compute the expected number of printed pages
	    int pages = computePageCount(newAttributes);
	
	    if (pages > 0) {
	        // Return print information to print framework
	        PrintDocumentInfo info = new PrintDocumentInfo
	                .Builder("print_output.pdf")
	                .setContentType(PrintDocumentInfo.CONTENT_TYPE_DOCUMENT)
	                .setPageCount(pages);
	                .build();
	        // Content layout reflow is complete
	        callback.onLayoutFinished(info, true);
	    } else {
	        // Otherwise report an error to the print framework
	        callback.onLayoutFailed("Page count calculation failed.");
	    }
	}


在不能完成布局计算的情况下，onLayout（）方法的执行可以有三个结果：完成，取消或失败。 您必须通过调用PrintDocumentAdapter.LayoutResultCallback对象的相应方法来指示其中一个结果。

> 注意：onLayoutFinished（）方法的布尔参数指示布局内容自上次请求以来是否实际上已更改。 正确设置此参数允许打印框架避免不必要地调用onWrite（）方法，基本上缓存先前写入的打印文档并提高性能。

onLayout（）的主要工作是计算给定打印机属性的输出页数。 如何计算这个数字很大程度上取决于您的应用程序如何布局打印页面。 以下代码示例显示了一个实现，其中页数由打印方向决定：

	private int computePageCount(PrintAttributes printAttributes) {
	    int itemsPerPage = 4; // default item count for portrait mode
	
	    MediaSize pageSize = printAttributes.getMediaSize();
	    if (!pageSize.isPortrait()) {
	        // Six items per page in landscape orientation
	        itemsPerPage = 6;
	    }
	
	    // Determine number of print items
	    int printItemCount = getPrintItemCount();
	
	    return (int) Math.ceil(printItemCount / itemsPerPage);
	}


## 写一个打印文档文件

当将打印输出写入文件的时候，Android打印框架调用应用程序的PrintDocumentAdapter类的onWrite（）方法。该方法的参数指定要写入的页面和要使用的输出文件。您的这种方法的实现必须将每个请求的内容页面呈现到多页PDF文档文件。当此过程完成时，您调用回调对象的onWriteFinished（）方法。

> 注意：Android打印框架可以每次调用onLayout（）一次或多次调用onWrite（）方法。因此，当打印内容布局未更改时，将onLayoutFinished（）方法的布尔参数设置为false是很重要的，以避免不必要的重新打印打印文档。

> 注意：onLayoutFinished（）方法的布尔参数指示布局内容自上次请求以来是否实际上已更改。正确设置此参数允许打印框架避免不必要地调用onLayout（）方法，基本上缓存以前写入的打印文档并提高性能。

以下示例演示了使用PrintedPdfDocument类创建PDF文件的此过程的基本机制：

	@Override
	public void onWrite(final PageRange[] pageRanges,
	                    final ParcelFileDescriptor destination,
	                    final CancellationSignal cancellationSignal,
	                    final WriteResultCallback callback) {
	    // Iterate over each page of the document,
	    // check if it's in the output range.
	    for (int i = 0; i < totalPages; i++) {
	        // Check to see if this page is in the output range.
	        if (containsPage(pageRanges, i)) {
	            // If so, add it to writtenPagesArray. writtenPagesArray.size()
	            // is used to compute the next output page index.
	            writtenPagesArray.append(writtenPagesArray.size(), i);
	            PdfDocument.Page page = mPdfDocument.startPage(i);
	
	            // check for cancellation
	            if (cancellationSignal.isCancelled()) {
	                callback.onWriteCancelled();
	                mPdfDocument.close();
	                mPdfDocument = null;
	                return;
	            }
	
	            // Draw page content for printing
	            drawPage(page);
	
	            // Rendering is complete, so page can be finalized.
	            mPdfDocument.finishPage(page);
	        }
	    }
	
	    // Write PDF document to file
	    try {
	        mPdfDocument.writeTo(new FileOutputStream(
	                destination.getFileDescriptor()));
	    } catch (IOException e) {
	        callback.onWriteFailed(e.toString());
	        return;
	    } finally {
	        mPdfDocument.close();
	        mPdfDocument = null;
	    }
	    PageRange[] writtenPages = computeWrittenPages();
	    // Signal the print framework the document is complete
	    callback.onWriteFinished(writtenPages);
	
	    ...
	}


该示例将PDF页面内容的呈现委托给drawPage（）方法，这将在下一节中讨论。

与布局一样，在无法写入内容的情况下，onWrite（）方法的执行可以有三个结果：完成，取消或失败。您必须通过调用PrintDocumentAdapter.WriteResultCallback对象的相应方法来指示其中一个结果。

> 注意：渲染用于打印的文档可能是资源密集型的操作。为了避免阻塞应用程序的主用户界面线程，您应该考虑在单独的线程上执行页面呈现和写入操作，例如在AsyncTask中执行。有关处理执行线程（如异步任务）的更多信息，请参阅进程和线程。

## 绘制PDF页面内容
当您的应用程序打印时，您的应用程序必须生成PDF文档并将其传递到Android打印框架进行打印。您可以使用任何PDF生成库来实现此目的。本课程将介绍如何使用PrintedPdfDocument类从您的内容生成PDF页面。

PrintedPdfDocument类使用Canvas对象在PDF页面上绘制元素，类似于活动布局上的绘图。您可以使用Canvas绘制方法在打印页面上绘制元素。以下示例代码演示了如何使用以下方法在PDF文档页面上绘制一些简单元素：

	private void drawPage(PdfDocument.Page page) {
	    Canvas canvas = page.getCanvas();
	
	    // units are in points (1/72 of an inch)
	    int titleBaseLine = 72;
	    int leftMargin = 54;
	
	    Paint paint = new Paint();
	    paint.setColor(Color.BLACK);
	    paint.setTextSize(36);
	    canvas.drawText("Test Title", leftMargin, titleBaseLine, paint);
	
	    paint.setTextSize(11);
	    canvas.drawText("Test paragraph", leftMargin, titleBaseLine + 25, paint);
	
	    paint.setColor(Color.BLUE);
	    canvas.drawRect(100, 100, 172, 172, paint);
	}

当使用Canvas绘制PDF页面时，元素以点为单位指定，为1/72英寸。 请确保使用此度量单位来指定页面上元素的大小。 对于绘制元素的定位，坐标系从页面左上角的0,0开始。

> 提示：虽然Canvas对象允许您将打印元素放置在PDF文档的边缘，但是许多打印机无法打印到实体纸张的边缘。 当您使用此类构建打印文档时，请确保您解决页面的不可打印边缘。