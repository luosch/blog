title: UIColor HexString
date: 2015-11-18 12:10:45
tags: [技术]
---
在开发 iOS app 的时候，较为蛋疼的一点是`UIColor`在设置颜色时用的 rgb 三个分量，而且用的是百分比，非常繁琐

```objective-c
// 白色
[UIColor colorWithRed:255.0/255.0 green:255.0/255.0 blue:255.0/255.0 alpha:1.0];
// 红色
[UIColor colorWithRed:255.0/255.0 green:0.0/255.0 blue:0.0/255.0 alpha:1.0];
```

##宏
通过一个处理 Hex 值的宏可以生成相应的 UIColor，这种做法耦合度较高，需要每个 view controller 里面都要设置这个宏

```objective-c
// RGB color macro
#define UIColorFromRGB(rgbValue) [UIColor \
colorWithRed:((float)((rgbValue & 0xFF0000) >> 16))/255.0 \
green:((float)((rgbValue & 0xFF00) >> 8))/255.0 \
blue:((float)(rgbValue & 0xFF))/255.0 alpha:1.0]
 
// RGB color macro with alpha
#define UIColorFromRGBWithAlpha(rgbValue,a) [UIColor \
colorWithRed:((float)((rgbValue & 0xFF0000) >> 16))/255.0 \
green:((float)((rgbValue & 0xFF00) >> 8))/255.0 \
blue:((float)(rgbValue & 0xFF))/255.0 alpha:a]
```

####例子
```objective-c
// 白色
self.view.backgroundColor = UIColorFromRGB(0xffffff)
// 红色
self.view.backgroundColor = UIColorFromRGB(0xff0000)
// 红色, 透明度为 50%
self.view.backgroundColor = UIColorFromRGBWithAlpha(0xff0000, 0.5)
```

##UIColor+HexString
一个耦合度低的做法是: 拓展 UIColor 类，加上`colorWithHexString`方法, 下面是一种`UIColor+HexString`的实现

```objective-c
// UIColor+HexString.m
+ (CGFloat) colorComponentFrom: (NSString *) string start: (NSUInteger) start length: (NSUInteger) length {
    NSString *substring = [string substringWithRange: NSMakeRange(start, length)];
    NSString *fullHex = length == 2 ? substring : [NSString stringWithFormat: @"%@%@", substring, substring];
    unsigned hexComponent;
    [[NSScanner scannerWithString: fullHex] scanHexInt: &hexComponent];
    return hexComponent / 255.0;
}

+ (UIColor *) colorWithHexString: (NSString *) hexString {
    NSString *colorString = [[hexString stringByReplacingOccurrencesOfString: @"#" withString: @""] uppercaseString];
    CGFloat alpha, red, blue, green;
    switch ([colorString length]) {
        case 3: // #RGB
            alpha = 1.0f;
            red   = [self colorComponentFrom: colorString start: 0 length: 1];
            green = [self colorComponentFrom: colorString start: 1 length: 1];
            blue  = [self colorComponentFrom: colorString start: 2 length: 1];
            break;
        case 4: // #ARGB
            alpha = [self colorComponentFrom: colorString start: 0 length: 1];
            red   = [self colorComponentFrom: colorString start: 1 length: 1];
            green = [self colorComponentFrom: colorString start: 2 length: 1];
            blue  = [self colorComponentFrom: colorString start: 3 length: 1];          
            break;
        case 6: // #RRGGBB
            alpha = 1.0f;
            red   = [self colorComponentFrom: colorString start: 0 length: 2];
            green = [self colorComponentFrom: colorString start: 2 length: 2];
            blue  = [self colorComponentFrom: colorString start: 4 length: 2];                      
            break;
        case 8: // #AARRGGBB
            alpha = [self colorComponentFrom: colorString start: 0 length: 2];
            red   = [self colorComponentFrom: colorString start: 2 length: 2];
            green = [self colorComponentFrom: colorString start: 4 length: 2];
            blue  = [self colorComponentFrom: colorString start: 6 length: 2];                      
            break;
        default:
            return nil;
    }
    return [UIColor colorWithRed: red green: green blue: blue alpha: alpha];
}
```

####例子

```objective-c
// 白色
[self.view setBackgroundColor:[UIColor colorWithHexString:@"#ffffff"]];
```


##参考
[UIColor macro with hex values](http://cocoamatic.blogspot.com/2010/07/uicolor-macro-with-hex-values.html)

[UIColor+HexString](https://github.com/kevinrenskers/UIColor-HexString)
