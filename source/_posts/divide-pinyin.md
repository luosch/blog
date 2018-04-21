title: 拼音分词
date: 2015-05-20 20:01:31
tags: [技术]
---
``` text
最近有个同学问我一个问题：给一串拼音，怎样得到所有可能的合法拼音组合，即拼音分词
输入：luosicheng
得到：lu,o,si,cheng 和 luo,si,cheng
```


###解决方案：
把拼音保存起来（只有338个，代价小），然后对那串拼音做一个深搜即可

``` python
# copyright lsc 2015-05-20
# coding=utf-8

pinyin = []
stack = []

# dfs
def deal(src):
    if len(src) == 0:
        string = ""
        for item in stack:
            string = string + item + ","

        string = string[:len(string)-1]
        print(string)
        return

    for obj in pinyin[ord(src[0]) - ord('a')]:
        length = len(obj)
        if src[0:length] == obj:
            stack.append(obj)
            deal(src[length:])
            stack.pop()


if __name__ == "__main__":
    # initialize the pinyin data
    with open("data.txt", "r", encoding="utf-8") as f:
        for line in f:
            item = line.strip("\n").split(" ")
            pinyin.append(item)

    string = input("please input a string\n")
    deal(string.lower())
```

拼音表
``` text
a ai an ang ao
ba bai ban bang bao bei ben beng bi bian biao bie bin bing bo bu
ca cai can cang cao ce cei cen ceng cha chai chan chang chao che chen cheng chi chong chou chu chua chuai chuan chuang chui chun chuo ci cong cou cu cuan cui cun cuo
da dai dan dang dao de den dei deng di dia dian diao die ding diu dong dou du duan dui dun duo
e ei en eng er
fa fan fang fei fen feng fo fou fu
ga gai gan gang gao ge gei gen geng gong gou gu gua guai guan guang gui gun guo
ha hai han hang hao he hei hen heng hong hou hu hua huai huan huang hui hun huo
i!
ji jia jian jiang jiao jie jin jing jiong jiu ju juan jue jun
ka kai kan kang kao ke ken keng kong kou ku kua kuai kuan kuang kui kun kuo
la lai lan lang lao le lei leng li lia lian liang liao lie lin ling liu long lou lu lv luan lve lun luo
m ma mai man mang mao me mei men meng mi mian miao mie min ming miu mo mou mu
na nai nan nang nao ne nei nen neng ni nian niang niao nie nin ning niu nong nou nu nv nuan nve nuo nun
o ou
pa pai pan pang pao pei pen peng pi pian piao pie pin ping po pou pu
qi qia qian qiang qiao qie qin qing qiong qiu qu quan que qun
ran rang rao re ren reng ri rong rou ru ruan rui run ruo
sa sai san sang sao se sen seng sha shai shan shang shao she shei shen sheng shi shou shu shua shuai shuan shuang shui shun shuo si song sou su suan sui sun suo
ta tai tan tang tao te teng ti tian tiao tie ting tong tou tu tuan tui tun tuo
u!
v!
wa wai wan wang wei wen weng wo wu
xi xia xian xiang xiao xie xin xing xiong xiu xu xuan xue xun
ya yan yang yao ye yi yin ying yo yong you yu yuan yue yun
za zai zan zang zao ze zei zen zeng zha zhai zhan zhang zhao zhe zhei zhen zheng zhi zhong zhou zhu zhua zhuai zhuan zhuang zhui zhun zhuo zi zong zou zu zuan zui zun zuo
```