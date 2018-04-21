title: 中大课程助手
date: 2014-12-24 11:19:43
tags: [技术, 生活]
---
最近有点无聊，于是想研究一下微信公众号，发现免费的应用就只有订阅号了

于是把微信订阅号和上学期做的刷课脚本结合一下，中大课程助手（公众号）就完成了

###开发环境
1. 杂牌电脑一台（win 8）
2. python，使用 flask 框架做后台处理
3. 笔和纸

###微信公众号申请
去[微信公众平台](http://mp.weixin.qq.com)提交订阅号的申请（还要交身份证照片，怕怕的）

###原理
发张公众号原理图
![](/resource/12.jpg)
我们的后台都是处理微信服务器转发的请求

###认证
开发者提交信息后，微信服务器将发送GET请求到填写的URL上，GET请求携带四个参数：

|  参数         | 描述                       |
| ---------------- |:---------------------------------:|
| signature  | 微信加密签名，signature结合了开发者填写的token参数和请求中的timestamp参数、nonce参数。  |
| timestamp | 时间戳 |
| nonce  | 随机数          |
| echostr  | 随机字符串  |

开发者通过检验signature对请求进行校验（下面有校验方式）。若确认此次GET请求来自微信服务器，请原样返回echostr参数内容，则接入生效，成为开发者成功，否则接入失败。
``` text
加密/校验流程如下：
1. 将token、timestamp、nonce三个参数进行字典序排序
2. 将三个参数字符串拼接成一个字符串进行sha1加密
3. 开发者获得加密后的字符串可与signature对比，标识该请求来源于微信
```
我写的进行校验的代码
``` python
import hashlib
# 微信公众号验证
def checkSignature(token, timestamp, nonce, signature):
    tmpStr = reduce(lambda x, y: x + y, sorted([token, timestamp, nonce]))
    return hashlib.sha1(tmpStr).hexdigest() == signature

@app.route('/', methods=['GET', 'POST'])
def weixin():
    # 验证信息
    if request.method == 'GET':

        signature = request.args.get('signature', '')
        timestamp = request.args.get('timestamp', '')
        nonce = request.args.get('nonce', '')
        echostr = request.args.get('echostr', '')
        if (signature == '' or timestamp == '' or nonce == '' or echostr == ''):
            return "error"
        else:
            if checkSignature(token, timestamp, nonce, signature):
                return echostr;
            else:
                return "error"
```

详细的认证方法可以去[开发者文档](http://mp.weixin.qq.com/wiki/home/index.html)了解

###读取课程信息
一开始想去旧的教务系统爬信息，发现抓取回来的 html 是空的，需要 js 进行渲染，遂放弃

转战新的[教务系统](http://uems.sysu.edu.cn/elect)，里面的“我的选课结果”就是本学期的课程
原理就是用你的账号密码登录，选课网站会返回一个 sid 凭证，在 url 和 response 里面都有，有了这个 sid 就可以到处访问了

爬虫代码

``` python
# get the sid
loginUrl = "http://uems.sysu.edu.cn/elect/login"
def login(stuNum, stuPasswd):
    try:
        # set cookie
        cookie = cookielib.CookieJar()
        cookieProc = urllib2.HTTPCookieProcessor(cookie)
        opener = urllib2.build_opener(cookieProc)
        urllib2.install_opener(opener)

        # login
        user = urllib.urlencode({'username': stuNum, 'password': stuPasswd})
        req = urllib2.Request(loginUrl, user)
        res = urllib2.urlopen(req)
        sid = res.geturl().replace('http://uems.sysu.edu.cn/elect/s/types?sid=', '')
        return sid

    except urllib2.HTTPError, e:
        return 0

# get the course list
courseUrl = 'http://uems.sysu.edu.cn/elect/s/courseAll?xnd=%s&xq=%s&sid=%s'
def getCourseList(sid, number):
    req = urllib2.Request(courseUrl % ("2014-2015", "2", sid))
    res = urllib2.urlopen(req)
    content = res.read()
    cut = content.find("toolbarTuitionMessage")
    data = listName()
    data.feed(content[cut:].replace("<td class='c'></td>", "<td class='c'>1</td>"))

    if not os.path.isfile("cache/%s.html" % number):
        print >> open("cache/%s.html" % number, "w"), content[cut:].replace("<td class='c'></td>", "<td class='c'>1</td>")

    courseList = []
    for index, item in enumerate(data.tdList):
        courseIndex = index / 14
        if index % 14 == 3:
            courseList.append({'name': item.decode("utf-8").encode("gbk")})
        if index % 14 == 7:
            courseList[courseIndex]['time'] = item.decode("utf-8").encode("gbk")
        if index % 14 == 9:
            courseList[courseIndex]['teacher'] = item.decode("utf-8").encode("gbk")
    return courseList
```
###调试接口
微信的在线调试工具是针对对已经上线的公众号，我写了个简单的调试脚本
``` python
import urllib2, time, parse

url = "http://localhost:8000"

if __name__ == "__main__":
    while(1):
        data = "<xml>"
        data += "<ToUserName><![CDATA[sysucc]]></ToUserName>" # % raw_input("ToUserName: ")
        data += "<FromUserName><![CDATA[%s]]></FromUserName>" % raw_input("FromUserName: ")
        data += "<CreateTime>%s</CreateTime>" % str(int(time.time()))
        data += "<MsgType><![CDATA[text]]></MsgType>"
        data += "<Content><![CDATA[%s]]></Content>" % raw_input("Content: ")
        data += "<MsgId>1234567890123456</MsgId>"
        data += "</xml>"

        req = urllib2.Request(url, data, {"Content-type" : "text/xml"})
        res = urllib2.urlopen(req)

        print res.read().decode("utf-8").encode("gbk")
```
###优化
一开始很傻很天真的使用了 mysql 存储网站信息，结果发现太慢了，于是全部存在文本里面
代码都放在[github](https://github.com/luosch/sysucourse)上面

###欢迎关注
![](/resource/14.jpg)
