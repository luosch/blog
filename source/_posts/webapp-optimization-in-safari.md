title: webapp 在 safari 上的优化
date: 2015-08-28 00:00:49
tags: [技术]
---
一般的网站在移动设备上的效果都不佳，最近在做一个 app 的分享页面的时候就遇到过一些问题

微信内的浏览器是封装了 webview, 然后绑定了点击事件实现长按出现二维码界面，以及通过劫持页面跳转来封杀正常的应用分发(apk 和跳转 app store)

连跳转 app store这一正常操作都被禁用了，只能跳转到腾讯自家的应用宝~~为了KPI也是拼~~

iOS 上大部分应用的内部浏览器封装自 webview，所以我们在调页面的时候要遵循苹果官方文档中关于 webapp 在 safari 的优化说明

<!-- more -->

### 禁止页面缩放
为了让我们的 webapp 更像一个 native 应用，我们要禁用浏览器的页面缩放功能，在苹果的官方文档中，标准做法是通过 viewport meta 来控制的

	<meta content="width=device-width; initial-scale=1.0; maximum-scale=1.0; user-scalable=0" name="viewport">
	
在 `<head>` 中添加如上代码即可禁止页面缩放

### 文本大小
有时候，我们会遇到 webapp 的文本太小的问题，这时我们可以通过修改 `-webkit-text-size-adjust` 属性来解决这个问题

	html {-webkit-text-size-adjust:none} // disable
	html {-webkit-text-size-adjust:auto} // default
	html {-webkit-text-size-adjust:200%} // zoom in
	html {-webkit-text-size-adjust: 50%} // zoom out


### 输入框
iOS 的键盘有 `autocorrect` 和 `autocapitalize` 的补全和改正功能，但是我们有时并不需要，比如在输入账号和密码的时候，这时我们可以在 `<input>` 标签设置对应属性来关闭这两个功能

	<input type="text" name="account" autocorrect="none" autocapitalize="none">

### 跳转到本地应用
![](https://developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/Art/smartbanner_2x.png)

	<meta name="apple-itunes-app" content="app-id=myAppStoreID, affiliate-data=myAffiliateData, app-argument=myURL">
### 伪装成 native app
	/* 设置保存到桌面的图标 */
	<link rel="apple-touch-icon" href="touch-icon-iphone.png">
	<link rel="apple-touch-icon" sizes="76x76" href="touch-icon-ipad.png">
	<link rel="apple-touch-icon" sizes="120x120" href="touch-icon-iphone-retina.png">
	<link rel="apple-touch-icon" sizes="152x152" href="touch-icon-ipad-retina.png">
	
	/* 设置全屏操作 */
	<meta name="apple-mobile-web-app-capable" content="yes">
	
	/* 设置顶栏颜色 */
	<meta name="apple-mobile-web-app-status-bar-style" content="black">

### 取消电话邮箱的识别
	<meta name="format-detection" content="telephone=no email=no">

### 参考资料
[apple-developer-document](https://developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/Introduction/Introduction.html#//apple_ref/doc/uid/TP40002051-CH1-SW1 "webapp for safari")
