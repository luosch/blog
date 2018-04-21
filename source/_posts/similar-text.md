title: 两个字符串的匹配度
date: 2015-12-14 12:49:51
tags: [技术]
---
在某个项目中遇到判断字符串相似程度的需求，google 了一下，原来`python`标准库里面没有这样的函数，反而是 PHP 里面有一个`similar_text()`函数，~~果然`PHP` 是世界上最好的编程语言~~

### 函数原型
`similar_text` 的原型是这样的

```php
int similar_text(string str1, string str2 [, float percent])
```

他返回的是两个字符串中能匹配的字符数量，如果有第三个参数，就修改第三个参数的值为 [0, 100] 区间的一个浮点数，表示匹配程度


<!-- more -->

### 例子

```php
$number = similar_text('aaaa', 'aaaa', $percent);
var_dump($number);  // int(4)
var_dump($percent); // int(100)


$number = similar_text('aaaa', 'aaaabbbb', $percent);
var_dump($number);  // int(4)
var_dump($percent); // int(67)

$number = similar_text('abcdef', 'aabcdefg', $percent);
var_dump($number);  // int(6)
var_dump($percent); // int(86)
```

### 原理分析
`similar_text` 代码可以在 [github](https://github.com/php/php-src/blob/master/ext/standard/string.c) 找到，搜索`PHP_FUNCTION(similar_text)` 就可以了

研究了一下发现，主要分为三步：

1. 找出两个字符串中相同部分最长的一段
2. 再用同样的方法在剩下的两段中分别找出相同部分最长的一段，以此类推，直到没有任何相同部分
3. 相似度 = 所有相同部分的长度之和 * 200.0 / 两个字符串的长度之和

下面是实现相关的代码，中文注释是我加上去进行说明的

```php
// 找出两个字符串中相同部分最长的一段
static void php_similar_str(const char *txt1, size_t len1, const char *txt2, size_t len2, size_t *pos1, size_t *pos2, size_t *max)
{
	char *p, *q;
	char *end1 = (char *) txt1 + len1;
	char *end2 = (char *) txt2 + len2;
	size_t l;

	*max = 0;
	// 以第一个字符串为基准开始遍历
	for (p = (char *) txt1; p < end1; p++) {
		// 遍历第二个字符串
		for (q = (char *) txt2; q < end2; q++) {
			// 发现有字符相同，继续循环找，l为相同部分的长度
			for (l = 0; (p + l < end1) && (q + l < end2) && (p[l] == q[l]); l++);
			// 记录最长的l，和他在两个字符串的位置
			if (l > *max) {
				*max = l;
				*pos1 = p - txt1;
				*pos2 = q - txt2;
			}
		}
	}
}

// 计算两个字符串的相同部分的总长度
static size_t php_similar_char(const char *txt1, size_t len1, const char *txt2, size_t len2)
{
	size_t sum;
	size_t pos1 = 0, pos2 = 0, max;
	
	// 找出两个字符串相同部分最长的一段
	php_similar_str(txt1, len1, txt2, len2, &pos1, &pos2, &max);
	
	// 这里是对sum的初始赋值，也是对max值的判断
	// 如果 max == 0，表示两个字符串没有任何相同的字符，则不会进行递归
	if ((sum = max)) {
		// 对前半段递归，相同段长度累加
		if (pos1 && pos2) {
			sum += php_similar_char(txt1, pos1,
									txt2, pos2);
		}
		// 对后半段递归，相同段长度累加
		if ((pos1 + max < len1) && (pos2 + max < len2)) {
			sum += php_similar_char(txt1 + pos1 + max, len1 - pos1 - max,
									txt2 + pos2 + max, len2 - pos2 - max);
		}
	}

	return sum;
}

/* {{{ proto int similar_text(string str1, string str2 [, float percent])
   Calculates the similarity between two strings */
// PHP函数定义
PHP_FUNCTION(similar_text)
{
	zend_string *t1, *t2;
	zval *percent = NULL;
	int ac = ZEND_NUM_ARGS();
	size_t sim;
	
	// 检查参数合法性
	if (zend_parse_parameters(ZEND_NUM_ARGS(), "SS|z/", &t1, &t2, &percent) == FAILURE) {
		return;
	}
	
	if (ac > 2) {
		convert_to_double_ex(percent);
	}
	
	// 如果两个字符串长度都为0，返回0
	if (ZSTR_LEN(t1) + ZSTR_LEN(t2) == 0) {
		if (ac > 2) {
			Z_DVAL_P(percent) = 0;
		}

		RETURN_LONG(0);
	}
	
	// 调用上面的函数，计算两个字符串的相似度
	sim = php_similar_char(ZSTR_VAL(t1), ZSTR_LEN(t1), ZSTR_VAL(t2), ZSTR_LEN(t2));
	
	// 如果有第三个参数，则改变计算方式
	if (ac > 2) {
		Z_DVAL_P(percent) = sim * 200.0 / (ZSTR_LEN(t1) + ZSTR_LEN(t2));
	}

	RETURN_LONG(sim);
}
```

### 进行移植
因为前文提到的项目主要是用 python 写的，所以我将 similar_text 这个函数移植到了 python，具体代码如下

```python
# -*- coding: utf-8 -*-

def similar_str(str1, str2):
    """
    return the len of longest string both in str1 and str2
    and the positions in str1 and str2
    """
    max_len = tmp = pos1 = pos2 = 0
    len1, len2 = len(str1), len(str2)

    for p in range(len1):
        for q in range(len2):
            tmp = 0
            while p + tmp < len1 and q + tmp < len2 \
                    and str1[p + tmp] == str2[q + tmp]:
                tmp += 1

            if tmp > max_len:
                max_len, pos1, pos2 = tmp, p, q

    return max_len, pos1, pos2


def similar_char(str1, str2):
    """
    return the total length of longest string both in str1 and str2
    """
    max_len, pos1, pos2 = similar_str(str1, str2)
    total = max_len

    if max_len != 0:
        if pos1 and pos2:
            total += similar_char(str1[:pos1], str2[:pos2])

        if pos1 + max_len < len(str1) and pos2 + max_len < len(str2):
            total += similar_char(str1[pos1 + max_len:], str2[pos2 + max_len:]);

    return total


def similar_text(str1, str2):
    """
    return a int value in [0, 100], which stands for match level
    """
    if not (isinstance(str1, str) or isinstance(str1, unicode)):
        raise TypeError("must be str or unicode")
    elif not (isinstance(str2, str) or isinstance(str2, unicode)):
        raise TypeError("must be str or unicode")
    elif len(str1) == 0 and len(str2) == 0:
        return 0.0
    else:
        return int(similar_char(str1, str2) * 200.0 / (len(str1) + len(str2)))
```

也可以在 [github](https://github.com/luosch/similar_text) 上查看

### 参考
[PHP source code](https://github.com/php/php-src)