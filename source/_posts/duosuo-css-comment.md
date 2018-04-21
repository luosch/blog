title: 多说评论框样式
date: 2014-11-26 23:43:36
tags: [技术]
---
![](/resource/5.png)


本博客使用[多说](http://duoshuo.com/ "多说评论系统")评论系统，发现默认主题太丑了，于是动手改了下

<!-- more -->

### 评论框扁平化
``` css
#ds-reset .ds-comment-body {
	background: #F7F7F7;
	padding: 10px 15px 10px 47px;
	box-shadow: #B8B9B9 0 1px 3px;
	border: white 1px solid;
}
```
### 构造评论框左边凹陷效果
``` css
#ds-reset .ds-post .ds-avatar {
	top: 15px;
	left: -20px;
	padding: 5px;
	width: 36px;
	height: 36px;
	-webkit-box-shadow: -1px 0 1px rgba(0,0,0,.15) inset;
	-moz-box-shadow: -1px 0 1px rgba(0,0,0,.15) inset;
	-ms-box-shadow: -1px 0 1px rgba(0,0,0,.15) inset;
	-o-box-shadow: -1px 0 1px rgba(0,0,0,.15) inset;
	box-shadow: -1px 0 1px rgba(0,0,0,.15) inset;
	-webkit-border-radius: 46px;
	-moz-border-radius: 46px;
	-ms-border-radius: 46px;
	-o-border-radius: 46px;
	border-radius: 46px;
	background: #eaeaea;
}
```
### 圆形头像
``` css
#ds-reset .ds-avatar img {
	width: 32px !important;
	height: 32px !important;
	-webkit-border-radius: 32px !important;
	-moz-border-radius: 32px !important;
	-ms-border-radius: 32px !important;
	-o-border-radius: 32px !important;
	border-radius: 32px !important;
	box-shadow: 0 1px 3px rgba(0, 0, 0, 0.22);
	-webkit-transition:.4s all ease-in-out;
	-moz-transition:.4s all ease-in-out;
	-ms-transition:.4s all ease-in-out;
	-o-transition:.4s all ease-in-out;
	transition:.4s all ease-in-out;
}
```
### 头像旋转效果
``` css
.ds-post-self:hover .ds-avatar img {
	-webkit-transform: rotate(360deg);
	-moz-transform: rotate(360deg);
	-o-transform: rotate(360deg);
	-ms-transform: rotate(360deg);
	transform: rotate(360deg);
}
```

最后上传到多说管理后台即可以看到效果啦~
