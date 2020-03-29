---
title: JavaScript正则表达式（含ES6扩展语法）
date: 2020-03-29 23:09:26
categories:
- JavaScript
tags:
- JavaScript
---


### 创建

+ 字面值直接创建

```
const regExp0 = /wh/
```

+ 新建一个正则表达式对象

```
const regExp1 = new RegExp("wh")	//用字符串创建正则表达式对象
const regExp2 = new RegExp(/wh/)	//用字面值创建正则表达式对象

reg1==reg2							//false
reg1.toString() === reg2.toLocaleString()	//true
```

toString()与toLocaleString()方法都会返回正则表达式字面值

```
new RegExp(new RegExp(/wh/g),"i")		// /wh/i
```

**【ES6】**RegExp构造函数第一个参数为RegExp对象时，可以使用第二个参数传入修饰符，覆盖原有的修饰符

### 修饰符

+ `i`：执行不区分大小写的匹配

```
/Wh/i.toString()==/wH/i.toString()  //false，同一个匹配规则不同的表达式
```

+ `g`：执行全局匹配，即找到所有匹配为止，与exec()方法连用时，每执行一次，都从当前剩余子串开始，一次返回一个匹配串

```
const str = "where when what"
const reg = /wh.*/g

reg.exec(str)		//index:0
reg.exec(str)		//index:6		对上个匹配后的剩余子串进行匹配
reg.exec(str)		//index:11
reg.exec(str)		//null			剩余子串为空
```

+ `m`：多行匹配模式，必须与^或$搭配使用，否则m修饰符无效。与^搭配使用时，匹配每一行的开头至换行符的子串；与$搭配使用时，匹配最后一行行尾至倒数第一个换行符的子串

```
const str1 = "Can u cAn a caN"
const str2 = "Can u \ncAn a caN"
const str3 = "Can u \rcAn \na caN"

const reg1 = /^C.*/img
const reg2 = /.*n$/img
str1.match(reg1)		//["Can u cAn a caN"]，m修饰符无效，在整个串中进行匹配
str2.match(reg1)		//["Can u ", "cAn a caN"]，从每一行行首开始进行匹配
str3.match(reg2)		//["a caN"],从末行行尾匹配至倒数第一个换行符
```

+ **【ES6】**`u`：Unicode模式，用来正确处理大于\uFFFF的Unicode字符，即可以正确处理4个字节的UTF-16编码

```
const utf16Str = '\uD824\uDFB7'
/^\uD824/.test(utf16Str)		//true，未能将utf16Str视为4个字节
/^\uD824/u.test(utf16Str)		//false

/𠮷{2}/.test('𠮷𠮷')		//false
/𠮷{2}/u.test('𠮷𠮷')		//true

/^\S$/.test(𠮷)				//false
/^\S$/u.test(𠮷)			//true

/[a-z]/i.test('\u212A')		//false，标准大写字母K的unicode码为\u004B
/[a-z]/iu.test('\u212A')	//true
```

* **【ES6】**`y`：粘连（sticky）模式，在全局匹配时确保匹配从剩余子串的第一个位置开始

```
const str = 'abc-abcabc'
const reg1 = /abc/g
const reg2 = /abc/y

reg1.exec(str)		//index:0
reg1.exec(str)		//index:4	剩余子串任意位置开始

reg2.exec(str)		//index:0
reg2.exec(str)		//null  	剩余子串第一位开始，此处为'-'
```

* **【ES6】**`s`：dotAll模式，使得`.`可以匹配行终止符

  行终止符有：

  * \u000A  即换行符`\n`
  * \u000D  即回车符`\r`
  * \u2028  行分隔符
  * \u2029  段分隔符

```
/foo.bar/.test('foo\nbar')		//false
/foo.bar/s/test('foo\nbar')		//true
```

### 直接量字符

+ `\o`：NULL字符（\u0000）
+ `\t`：制表符（\u0009）
+ `\n`：换行符（\u000A）
+ `\v`：垂直制表符（\u000B）
+ `\f`：换页符（\u000C）
+ `\r`：回车符（\u000D）
+ `\xnn`：由十六进制数nn指定的拉丁字符，如\x0A等价于\n
+ `\uxxxx`：由十六进制数制定的Unicode字符，如\u0009等价于\t
+ `\cX`：控制字符^X，如\cJ等价于换行符\n
+ `[\b]`：退格直接量，即Backspace

### 预设字符类

* `[]`：方括号内的任意字符

```
const str = '1245azc'
const reg = /[2-5a-c]/g
str.match(reg)				//["2", "4", "5", "a", "c"]
```

* `[^]`：非方括号内的任意字符

```
const str = '1245azc'
const reg = /[^2-5a-c]/g
str.match(reg)		//["1", "z"]
```

* `.`：除行终止符之外的任意字符
* `\w`：任何ASCII数字和大小写字母，等价于`[a-zA-Z0-9]`

```
const str1 = 'A\u212A-47'
const str2 = 'A\u004B-47'
const reg = /\w/g
str1.match(reg)		//["A", "4", "7"]
str2.match(reg)		//["A", "K", "4", "7"]
```

* `\W`：任何非ASCII数字和大小写字母，等价于`[^a-zA-Z0-9]`

* `\s`：任何Unicode空白符

```
const str = 'a\r1\n+\u2028-\u2029'
const reg = /\s/g
str.match(reg)				//["", "↵", "", ""]
```

+ `\S`：任何非空白的Unicode字符

```
const str = 'a\r1\n+\u2028-\u2029'
const reg = /\S/g
str.match(reg)				//["a", "1", "+", "-"]
```

+ `\d`：任何ASCII数字，等价于`[0-9]`
+ `\D`：任何非ASCII数字，等价于`[^0-9]`

### 重复数量

+ `{m,n}`：m<=匹配前一项数量<=n

```
'a'.match(/a{2,4}/g)		//null
'aaaaa'.match(/a{2,4}/g)	//["aaaa"]
```

+ `{m,}`：匹配前一项数量>=m
+ `{n}`：匹配前一项n次
+ `？`：匹配前一项0次或1次，即{0,1}
+ `+`：匹配前一项1次或多次，即{1,}
+ `*`：匹配前一项0次或多次，即{0,}

```
'aaaa'.match(/a*/g)			//["aaaa", ""]  因为*会匹配0次'a'
```

### 选择、分组、引用

+ `|`：选择，匹配该符号左边的表达式或右边的表达式，匹配顺序从左到右，左边匹配到就忽略右边匹配项
+ `()`：分组，将若干匹配项组合为一个匹配项，以对整体进行`?` `*` `+` `|`操作，且记忆分组内容供引用操作使用
+ `(?:)`：分组，将若干匹配项组合为一个匹配项，以对整体进行`?` `*` `+` `|`操作，不记忆分组内容
+ \n：引用，n代表分组的序号

```
/['"][^']*['"]/	
/(['"])[^']*\1/
```

两者等价，`\1`就是`(['"])`对这个分组的引用

在`String.prototype.replace()`中，可以使用`$分组序号`的方式作为第二个参数

```
str.replace(/(['"])[^']*\1/g,"$1")		//将单双引号及内容替换为每一组单双引号的起始'"
```

### 锚字符

* `^`：匹配字符串的开头
* `$`：匹配字符串的结尾
* `\b`：匹配一个单词的边界

```
const str = "this that sthiss"
str.match(/this/g)				//["this","this"]
str.match(/\bthis\b/g)			//["this"]
```

+ `\B`：匹配非单词边界的位置

```
const str = "athat thatb cthatd"
const regA = /\Bthat/g
const regB = /that\B/g
const regCD = /\Bthat\B/g
str.match(regA)					//["that", "that"]	athat,cthatd
str.match(regB)					//["that", "that"]	thatb,cthatd
str.match(regCD)				//["that"]		cthatd
```

### 断言

+ `/x(?=y)/`先行断言

```
'10% 20'.match(/\d+(?=%)/g)	//["10"]
```

+ `/x(?!y)/`先行否定断言

```
'10% 20'.match(/\d+(?!%)/g)		//["1", "20"]
```

+ `/(?<=y)x/`后行断言

```
'$50 ￥100'.match(/(?<=\$)\d+/g)	//["50"]
```

+ `/(?<!y)x/`后行否定断言

```
'$50 ￥100'.match(/(?<!\$)\d+/g)	//["0", "100"]
```

### RegExp实例属性

| 属性名            | 属性类型 | 含义                                               |
| ----------------- | -------- | -------------------------------------------------- |
| global            | 布尔值   | 是否设置了g修饰符                                  |
| ignoreCase        | 布尔值   | 是否设置了i修饰符                                  |
| mutiline          | 布尔值   | 是否设置了m修饰符                                  |
| lastIndex         | 整数     | 一个计步器，表示开始搜索下一个匹配项的字符位置     |
| source            | 字符串   | 正则表达式的字符串表示，即不含`\\`与`修饰符`的部分 |
| **【ES6】**sticky | 布尔值   | 是否设置了y修饰符                                  |
| **【ES6】**flags  | 字符串   | 正则表达式的修饰符                                 |

### RegExp实例方法

```
const str = "where when what"
const regExp = /wh/
```

- **RegExp.prototype.exec(string)** 返回首个匹配正则表达式的子串的首字符下标

```
regExp.exec(str)		//[ 'wh', index: 0, input: 'where when what', groups: undefined ]
```

+ **RegExp.prototype.test(string)** 返回字符串是否匹配正则表达式的布尔值

```
regExp.test(str)		//true
```

### 字符串实例方法

```
const str = "This string contains numbers 123 Capitalized LETTERS and +_^% Symbols"
```

+ **String.prototype.match(RegExp)** 返回匹配正则表达式的子串列表(g全局模式下)

```
str.match(/Th.s/g)		//["This"]
```

+ **String.prototype.replace(RegExp,String)**返回用String替代匹配正则表达式的子串

```
const reg1 = /[^0-9A-Za-z\s]/g
str.replace(reg1,"")		
//'This string contains numbers 123 Capitalized LETTERS and  Symbols'
```

+ **String.prototype.search(RegExp)**返回首个匹配正则表达式的子串首字符下标

```
const reg2 = /\d/g
console.log(str.search(reg2));		//29	即数字1的位置
```

+ **String.prototype.split(RegExp)**返回以匹配正则表达式的字符串为间隔的数组

```
const reg3 = /\s/g
console.log(str.split(reg3));
//["This", "string", "contains", "numbers", "123", "Capitalized", "LETTERS", "and", "+_^%", "Symbols"]
```





> 【1】David Flanagan著. JavaScript权威指南（第6版）. 淘宝前端团队译.  机械工业出版社，2012.3
>
> 【2】Nicholas C.Zakas著. 《JavaScript高级程序设计（第3版）》李松峰，曹力译. 人民邮电出版社，2012.3
>
> 【3】阮一峰著. 《ES6标准入门（第3版）》. 电子工业出版社，2017.9

