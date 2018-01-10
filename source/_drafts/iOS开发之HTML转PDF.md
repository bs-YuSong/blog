---
title: iOS开发之HTML转PDF
tags: iOS
categories: iOS
---

前言
===

最近在做一个HTML转PDF的功能，由于之前没有做过相关功能，就顺手去Google了一下，发现搜出来的文章全都千篇一律，都像是stackoverflow上一个很久之前的答案的扩展版，讲的非常粗略，没有解决我的疑惑，所以就自己研究了一下官方文档，最终成果就是这篇文章。
由于最近比较忙，所以我也只是简单的学习了一下，并未深处研究，有时间的话，我会再写一遍详解。

<!-- more -->

<h5 style="color: red">Print 和 Image and PDF</h5>

Print是UIKit框架中的一个功能模块，其中包括了一些和打印相关的类和方法。我这里主要讲的是UIPrintPageRenderer和UIViewPrintFormatter。

UIPrintPageRenderer是一个负责绘制Page的类，当我们想要渲染一个Page时，就需要重写UIPrintPageRenderer类，然后来自定义我们想要渲染的页面的样式。
UIPrintPageRenderer主要提供的重写方法有：
	```swift
		drawPage(at:in:)
		drawHeaderForPage(at:in:)
		drawContentForPage(at:in:)
		drawPrintFormatter(_:forPageAt:)
		drawFooterForPage(at:in:)
	```
关于这些方法的具体使用网上很多，我不细讲，我主要讲的是两个需要重新的属性：
	```swift
	let paper = CGRect(x: 0, y: 0, width: 595.2, height: 841.8)
	override var paperRect: CGRect {
		return paper
	}
	override var printableRect: CGRect {
		return paper
	}
	```
paperRect表示可以绘制的paper的大小(其实在输出PDF时，PDF的一页的大小是另外一个属性控制的)，printableRect表示实际绘制时的区域(其实是和别的属性共通控制的)。

如果不重写这两个属性的话，绘制出来的PDF会是空白的，所以一定要重写，我这里建议将paperRect的大小设置为PDF的一页的大小，至于PDF一页的大小要如何设置，我下边会讲到。

简单说完UIPrintPageRenderer，接下来讲讲UIViewPrintFormatter，看名字就知道，这是一个格式类，管理导出PDF的格式。

这个UIViewPrintFormatter并不需要我们来手动创建，可以通过UIWebView(or WKWebView)、UITextView、 MKMapView调用viewPrintFormatter()来获取，自带内容，使用起来很方便。

UIViewPrintFormatter自带很多和格式有关的属性和方法，有兴趣的可以去看看它的文档，我这里只讲几个下边会用到的。
	```swift
	perPageContentInsets /* 用来控制PDF内容的边距的，和UIPrintPageRenderer的printableRect共通作用*/
	pageCount //内容的页数，UIViewPrintFormatter根据layout自动计算得出，很有用
	```

上文我说过，实际绘制时的区域是由printableRect控制，实际绘制区域是由printableRect和perPageContentInsets共通控制的，当perPageContentInsets和printableRect的表示类似或相同时，perPageContentInsets会被忽略。

Images and PDF同样是UIKit中的一个模块，主要通过bitmap和pdf来创建和管理图片，我们经常用到的UIGraphicsBeginImageContext方法就是属于这个模块的。这个方法是用来创建一个Image的context，通过context可以获取Image，同理，想要获取PDF，也需要一些类似的方法：
	```swift
	UIGraphicsBeginPDFContextToData(_:_:_:) /* 通过Data创建一个context */
	UIGraphicsBeginPDFPage()	/* 创建一个Page页 */
	UIGraphicsEndPDFContext() /* 关闭PDF context */
	```

<h5 style="color: red">纸张大小的转换</h5>

关于point和纸张大小的转换我也不知道具体公式，我只知道A4纸的大小是595.2 \* 841.8，我算了一下，比例大概就是：纸张比例 \* 2.843。有了这应该也可以凑活用了。

<h5 style="color: red">具体流程</h5>

重头戏来啦。前边说了这么多，为了就是接下来的代码展示。

首先是重写UIPrintPageRenderer：
	```swift
	class CustomPrintPageRenderer: UIPrintPageRenderer {
    let paper = CGRect(x: 0, y: 0, width: 595.2, height: 841.8)
    override var paperRect: CGRect {
        return paper
    }
    override var printableRect: CGRect {
        return paper
    }
	}
	```
接着是HTML转为PDF：
	```swift
	func drawPDFUsingPrintPageRenderer(_ printPageRenderer:UIPrintPageRenderer, page: Int) -> NSData! {
    let data = NSMutableData()
    UIGraphicsBeginPDFContextToData(data, CGRect.zero, nil)
    for i in 0..<page {
        UIGraphicsBeginPDFPage()
        printPageRenderer.drawPage(at: i, in: UIGraphicsGetPDFContextBounds())
    }
    UIGraphicsEndPDFContext()
    return data
	}
	func exportHTMLContentToPDF() {
    let printPageRenderer = CustomPrintPageRenderer()
    let printFormatter = webView.viewPrintFormatter()
    printPageRenderer.addPrintFormatter(printFormatter, startingAtPageAt: 0)
    if let pdfData = drawPDFUsingPrintPageRenderer(printPageRenderer, page: printFormatter.pageCount) {
        pdfData.write(toFile: "test.pdf", atomically: true)
    }
	}
	```

最后需要注意的一点是，我前边提到了paperRect控制的paper大小并不是PDF一页的大小。实际控制PDF一页大小的是UIGraphicsBeginPDFPage())。通过UIGraphicsBeginPDFPage()方法创建的page的大小为默认值： 612 \* 792,如果想要修改PDF一页的大小，需要使用UIGraphicsBeginPDFPageWithInfo(_:_:)来创建。
















