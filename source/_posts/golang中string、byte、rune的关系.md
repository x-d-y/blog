---
title: golang中string、byte、rune的关系
date: 2019-08-07 18:06:36
tags: golang
---
&#160;&#160;&#160;&#160;&#160;&#160;在go语言中，有关于字符操作的基本类型有三种，分别是string、byte和rune,在其他的语言中，大部分语言都没有byte和rune类型。并且他们之间也是经常相互转换，虽然大量的go语言书记都对go语言这三种类型作了介绍，但仍然觉得没有一个清晰的认识，本篇博客就通过一些go程序例子来探究他们之间的关系。
<!--more-->
## string
&#160;&#160;&#160;&#160;&#160;&#160;在《go语言圣经》一书中对于字符串的描述是这样的：一个字符串是一个不可改变的字节序列。字符串可以包含任意的数据,包括byte值0,但是通常是用来包含人类可读的文本。通过以上描述我们能知道字符串是字节序列，用于包含人类可读的文本。通过go语言程序来实现以下如下：
```
package main

import (
	"fmt"

	"reflect"
)

func main() {
	a := "welcome"

	fmt.Println(reflect.TypeOf(a))  //string

	fmt.Println([]rune(a))          //welcome
}

```
通过打印结果能够看出，string确实打印出了"welcome"字样，这个使我们人类所能读懂的文本。但如何来理解字节序列？还需要看byte这个变量的内容。

## byte
&#160;&#160;&#160;&#160;&#160;&#160;byte的直译是字节、位单元，其实这个意思用在golang中也是准确的，即一个字节的意思。在上一篇博客中《unicode和utf8关系》中已经做了介绍，ASCII字符全部都是采用一个字节表示。
```
package main

import (
	"fmt"

	"reflect"
)

func main() {
	a := "welcome"

	fmt.Println([]byte(a))    //[119 101 108 99 111 109 101]

}

```
由以上可以看到，“welcome”里个字母都属于ASCII字符集，对应的都是六个字节，通过查询ASCII字码表可知“w”是119，“e”是101，其余的四个字母也都一一对应。所以通过以上的代码实验可以得出，byte的作用就是将string转换为字节数。那对于非ASCII字符集的字符如何表示呢？
<div align=center>
<img src="https://github.com/x-d-y/blog/blob/master/source/_posts/golang%E4%B8%ADstring%E3%80%81byte%E3%80%81rune%E7%9A%84%E5%85%B3%E7%B3%BB/screenshot.png?raw=true" width=600>
</div>
```
func main() {
	a := "中文"

	fmt.Println([]byte(a))  //[228 184 173 230 150 135]
}
```
一般来说中文都是采用三个字节进行表示，这里的两个汉子很可能就是每三个字节对应一个汉字。具体是否是猜想的这样，还需要看看对应的Unicode码点值。

## rune
&#160;&#160;&#160;&#160;&#160;&#160;在go语言中，rune也是字符的一种表达方式，先来看看，rune表达的字符的内容。
```
func main() {
	a := "中文"
	b := "welcome"
	fmt.Println(b, []byte(b), []rune(b))    //welcome [119 101 108 99 111 109 101] [119 101 108 99 111 109 101]

	fmt.Println(a, []byte(a), []rune(b))    //中文 [228 184 173 230 150 135] [20013 25991]
}
```

rune()操作之后的值个数正好等于字符的个数，也就是说一个字符对应一个rune，不难发现在ASCII编码的字符中，byte的值和rune的值一一对应，数值也完全相同。因为Unicode编码是完全兼容ASCII的，所以很有可能rune对应的值就是Unicode值。其中“中”字符所对应的Unicode码为\u4e2d，一种十六进制的表达方式，将\u4e2d转换为10进制后发现值为20013，说明rune就是一个字符对应一个Unicode码。至于byte为三个字节228 184 173，展开成3个二进制8位码，值为：11100100 10111000 10101101 .看到这个三个值有没有觉得很像utf-8的编码？三个字节对应111XXXXX 10XXXXXX 10XXXXXX。将所有的X提出来组成新的二进制码100111000101101，它对应的10进制为20013.

### 总结
&#160;&#160;&#160;&#160;&#160;&#160;通过以上的代码实验探究可以得出以下结论：

- string所代表的字符为我们正常人能够读懂的最基础的字符含义如"welcome"
- byte所代表的含义为对应的字符转成utf-8编码之后的字节信息，包含头部110,10等的信息
- rune所代表的信息为Unicode码，显示的时候转换成十进制进行的显示