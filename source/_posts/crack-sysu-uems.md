title: 记一次破解验证码的过程
date: 2015-09-13 00:43:03
tags: [技术]
---
教务系统现在使用了验证码，原来的抢课脚本不能用了，于是最近写了一个新的[抢课脚本](https://github.com/luosch/crack-sysu-uems)

本文是关于如何对验证码进行识别，具体抢课的功能代码请点击上面的链接查看

###环境
``` python
python3      # 编程环境 
tesseract    # OCR 库
pytesseract  # tesseract 的 python 接口调用
```

###验证码特点
![验证码](/resource/15.jpg)

- 宽 68px
- 高 21px
- 有边框
- 有噪点
- 字体颜色不一

###思路
1. 直接用 pytesseract 进行识别的话效果非常差
2. 对于噪点和颜色不一，采用像素二值化的方法，即将灰度值高的点变为黑色，灰度值低的点变为白色

###具体代码
``` python
im = Image.open('code.jpg')
im = im.convert('L')

# 灰度值高的点变为黑色
im = im.point(lambda x:255 if x > 128 else x)

# 灰度值低的点变为白色
im = im.point(lambda x:0 if x < 255 else 255)
    
# 去除边框
box = (2, 2, im.size[0] - 2, im.size[1] - 2)
im = im.crop(box)
	
# -psm 7 是 tesseract 的文字识别模式
# J 容易识别成 ], 故在此进行替换
verify_code = pytesseract.image_to_string(im, lang="eng", config="-psm 7").replace(']', 'J')
```

### 效果
![验证码](/resource/15.jpg)
![验证码](/resource/16.jpg)

###链接

[tesseract](https://github.com/tesseract-ocr/tesseract)

[pytesseract](https://github.com/madmaze/pytesseract)
