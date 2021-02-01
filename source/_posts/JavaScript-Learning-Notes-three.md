---
title: JavaScript 学习笔记(三)
date: 2021-02-01 17:56:05
tags:
  - JavaScript
  - 学习笔记
categories:
  - JavaScript
---

# Date

在JavaScript中，`Date`对象用来表示日期和时间。

要获取系统当前时间，用：

```javascript
const date = new Date();//  注意要new 否则调用不了方法
console.log(date.toString()); // Mon Feb 01 2021 15:12:07 GMT+0800 (香港标准时间)
console.log(date.getFullYear()); // 2021, 年份
console.log(date.getMonth()); // 1,月份，注意月份范围是0~11，1表示二月
console.log(date.getDate()); // 1号
console.log(date.getDay()); // 1,星期一
console.log(date.getHours()); // 15,时
console.log(date.getMinutes()); // 12,分
console.log(date.getSeconds()); // 7,秒
console.log(date.getMilliseconds()); // 283,毫秒
console.log(date.getTime()); //1612163527283 时间戳
```

如果要创建一个指定日期和时间的`Date`对象，可以用：

```javascript
const next = new Date(2021,1,1,1,1,1);
console.log(next.toString()); // Mon Feb 01 2021 01:01:01 GMT+0800 (香港标准时间)
```

更有意思的是，还能够用来自动更正时间：

```javascript
// 传入一个错误的时间 会自动校正
const errorDate = new Date(2020,1,55,1,1,1);
console.log(errorDate.toString()); // Thu Mar 26 2020 01:01:01 GMT+0800 (香港标准时间)
```

第二种创建一个指定日期和时间的方法是解析一个符合[ISO 8601](http://www.w3.org/TR/NOTE-datetime)格式的字符串：

```javascript
const isoDate = Date.parse('2021-02-01T22:49:22.875+08:00');
console.log(isoDate.toString()); // 1612190962875
```

但返回的不是`Date`对象，而是一个时间戳。不过时间就可以很容易地转换为一个`Date`：

```javascript
const timeStampDate = new Date(1612190962875);
console.log(timeStampDate.toString()); // Mon Feb 01 2021 22:49:22 GMT+0800 (香港标准时间)
```

## 时区

`Date`对象表示的时间总是按浏览器所在时区显示的，不过我们既可以显示本地时间，也可以显示调整后的UTC时间：

```javascript
const localDate = new Date();
// 获取本地时间
console.log(localDate.toLocaleString()); // 2021/2/1下午5:26:06
// 获取 UTC 时间 相差8小时
console.log(localDate.toUTCString()); // Mon, 01 Feb 2021 09:26:06 GMT
console.log(localDate.toLocaleDateString()); // 2021/2/1
console.log(localDate.toLocaleTimeString()); // 下午5:26:06
```

## 时间的计算

### 比较两个时间的大小

```javascript
const date1 = new Date('2021-02-01 17:23:13');
const date2 = new Date('2021-02-01 18:07:54');
const result = date1.getTime() - date2.getTime();
if (result > 0) {
  console.log('date1 > date2');
} else if (result < 0) {
  console.log('date1 < date2');
} else {
  console.log('date1 == date2');
}

// date1 < date2
```

### 加上一段时间

```javascript
const date = new Date();
console.log(date.toLocaleString()); // 2021/2/1下午5:31:06
// 加上一个小时
date.setTime(date.getTime() + 60 * 60 * 1000);
console.log(date.toLocaleString()); // 2021/2/1下午6:31:06
```

# Math

JavaScript Math 对象允许您对数字执行数学任务.

```javascript
const pi = Math.PI; // 3.141592653589793
// 四舍五入为最接近的整数
Math.round(6.8); // 7
// Math.pow(x, y) 的返回值是 x 的 y 次幂
Math.pow(8,2); // 64
// Math.sqrt(x) 返回 x 的平方根
Math.sqrt(64); // 8
// 绝对值
Math.abs(-4.7); // 4.7
// Math.ceil(x) 的返回值是 x 上舍入最接近的整数
Math.ceil(6.4); // 7
// Math.floor(x) 的返回值是 x 下舍入最接近的整数
Math.floor(2.7); // 2
// Math.sin(x) 返回角 x（以弧度计）的正弦（介于 -1 与 1 之间的值）
Math.sin(Math.PI / 2);
// Math.min() 和 Math.max() 可用于查找参数列表中的最低或最高值
Math.min(0, 450, 35, 10, -8, -300, -78);  // 返回 -300
// Math.random() 返回介于 0（包括） 与 1（不包括） 之间的随机数
Math.random();     // 返回随机数
```

# Proxy

Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。

ES6 原生提供 Proxy 构造函数，用来生成 Proxy 实例。

```javascript
var proxy = new Proxy(target, handler);
```

Proxy 对象的所有用法，都是上面这种形式，不同的只是`handler`参数的写法。其中，`new Proxy()`表示生成一个`Proxy`实例，`target`参数表示所要拦截的目标对象，`handler`参数也是一个对象，用来定制拦截行为。

下面是另一个拦截读取属性行为的例子。

```javascript
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

proxy.time // 35
proxy.name // 35
proxy.title // 35
```

上面代码中，作为构造函数，`Proxy`接受两个参数。第一个参数是所要代理的目标对象（上例是一个空对象），即如果没有`Proxy`的介入，操作原来要访问的就是这个对象；第二个参数是一个配置对象，对于每一个被代理的操作，需要提供一个对应的处理函数，该函数将拦截对应的操作。比如，上面代码中，配置对象有一个`get`方法，用来拦截对目标对象属性的访问请求。`get`方法的两个参数分别是目标对象和所要访问的属性。可以看到，由于拦截函数总是返回`35`，所以访问任何属性都得到`35`。

注意，要使得`Proxy`起作用，必须针对`Proxy`实例（上例是`proxy`对象）进行操作，而不是针对目标对象（上例是空对象）进行操作。

# 正则表达式

JS 的正则表达式跟 Java 的正则表达式类似。

用`\d`可以匹配一个数字，`\w`可以匹配一个字母或数字, `.`可以匹配任意一个字符。

要匹配变长的字符，在正则表达式中，用`*`表示任意个字符（包括0个），用`+`表示至少一个字符，用`?`表示0个或1个字符，用`{n}`表示n个字符，用`{n,m}`表示n-m个字符。

`\s` 可以匹配一个空格（包括Tab等空白符）。

要做更精确地匹配，可以用`[]`表示范围，比如：

- `[0-9a-zA-Z\_]`可以匹配一个数字、字母或者下划线；
- `[0-9a-zA-Z\_]+`可以匹配至少由一个数字、字母或者下划线组成的字符串，比如`'a100'`，`'0_Z'`，`'js2015'`等等；
- `[a-zA-Z\_\$][0-9a-zA-Z\_\$]*`可以匹配由字母或下划线、$开头，后接任意个由一个数字、字母或者下划线、$组成的字符串，也就是JavaScript允许的变量名；
- `[a-zA-Z\_\$][0-9a-zA-Z\_\$]{0, 19}`更精确地限制了变量的长度是1-20个字符（前面1个字符+后面最多19个字符）。

`A|B`可以匹配A或B，所以`(J|j)ava(S|s)cript`可以匹配`'JavaScript'`、`'Javascript'`、`'javaScript'`或者`'javascript'`。

`^`表示行的开头，`^\d`表示必须以数字开头。

`$`表示行的结束，`\d$`表示必须以数字结束。

你可能注意到了，`js`也可以匹配`'jsp'`，但是加上`^js$`就变成了整行匹配，就只能匹配`'js'`了。整行匹配就是要全部匹配，不能多也不能少。

```javascript
const js = /^js$/;
console.log(js.test("jss")); // false
```

## 切分字符串

用正则表达式切分字符串比用固定的字符更灵活，请看正常的切分代码：

```javascript
'a b   c'.split(' '); // ['a', 'b', '', '', 'c']
```

嗯，无法识别连续的空格，用正则表达式试试：

```javascript
'a b   c'.split(/\s+/); // ['a', 'b', 'c']
```

无论多少个空格都可以正常分割。加入`,`试试：

```javascript
'a,b, c  d'.split(/[\s\,]+/); // ['a', 'b', 'c', 'd']
```

再加入`;`试试：

```javascript
'a,b;; c  d'.split(/[\s\,\;]+/); // ['a', 'b', 'c', 'd']
```

如果用户输入了一组标签，下次记得用正则表达式来把不规范的输入转化成正确的数组。

## 分组

除了简单地判断是否匹配之外，正则表达式还有提取子串的强大功能。用`()`表示的就是要提取的分组（Group）。比如：

`^(\d{3})-(\d{3,8})$`分别定义了两个组，可以直接从匹配的字符串中提取出区号和本地号码：

```javascript
var re = /^(\d{3})-(\d{3,8})$/;
re.exec('010-12345'); // ['010-12345', '010', '12345']
re.exec('010 12345'); // null
```

如果正则表达式中定义了组，就可以在`RegExp`对象上用`exec()`方法提取出子串来。

`exec()`方法在匹配成功后，会返回一个`Array`，第一个元素是正则表达式匹配到的整个字符串，后面的字符串表示匹配成功的子串。

`exec()`方法在匹配失败时返回`null`。

## 贪婪匹配

正则匹配默认是贪婪匹配，也就是匹配尽可能多的字符。举例如下，匹配出数字后面的`0`：

```javascript
var re = /^(\d+)(0*)$/;
re.exec('102300'); // ['102300', '102300', '']
```

由于`\d+`采用贪婪匹配，直接把后面的`0`全部匹配了，结果`0*`只能匹配空字符串了。

必须让`\d+`采用非贪婪匹配（也就是尽可能少匹配），才能把后面的`0`匹配出来，加个`?`就可以让`\d+`采用非贪婪匹配：

```javascript
var re = /^(\d+?)(0*)$/;
re.exec('102300'); // ['102300', '1023', '00']
```

## 全局搜索

JavaScript的正则表达式还有几个特殊的标志，最常用的是`g`，表示全局匹配：

```javascript
var r1 = /test/g;
// 等价于:
var r2 = new RegExp('test', 'g');
```

全局匹配可以多次执行`exec()`方法来搜索一个匹配的字符串。当我们指定`g`标志后，每次运行`exec()`，正则表达式本身会更新`lastIndex`属性，表示上次匹配到的最后索引：

```javascript
var s = 'JavaScript, VBScript, JScript and ECMAScript';
var re=/[a-zA-Z]+Script/g;

// 使用全局匹配:
re.exec(s); // ['JavaScript']
re.lastIndex; // 10

re.exec(s); // ['VBScript']
re.lastIndex; // 20

re.exec(s); // ['JScript']
re.lastIndex; // 29

re.exec(s); // ['ECMAScript']
re.lastIndex; // 44

re.exec(s); // null，直到结束仍没有匹配到
```

全局匹配类似搜索，因此不能使用`/^...$/`，那样只会最多匹配一次。

正则表达式还可以指定`i`标志，表示忽略大小写，`m`标志，表示执行多行匹配。

## 语法

```shell
/正则表达式主体/修饰符(可选)
```

正则表达式主体即正则表达式。

修饰符的话，则有三种，如下：

| 修饰符 | 描述                                                     |
| :----- | :------------------------------------------------------- |
| i      | 执行对大小写不敏感的匹配。                               |
| g      | 执行全局匹配（查找所有匹配而非在找到第一个匹配后停止）。 |
| m      | 执行多行匹配。                                           |

对于字符串而言，正则表达式通常有两个方法：search() 和 replace(),显然前者是检索后者是替换。

现在我们来尝试一下：

```javascript
var str = "Hello World";
//  i 搜索不区分大小写 返回第一次出现的位置
// 例如：下面检索 第一次出现world的地方
var i = str.search(/WORLD/i);
// g 是区分大小写的,并且是全局搜索
var g = str.search(/L/g);

// 对于 m 测试发现,用处真的不是很大,g 就可以了
var str2 = "1\n 2\n 3\n";
var g2 = str2.search(/2/g);
var m2 = str2.search(/2/m);

// 替换字符串
var str3 = "AABB";

// 仅将第一次出现的 A的替换成 1
var i3 = str3.replace(/a/i,"1");
// 既全局匹配又忽略大小写,将所有的A均替换为1,修饰符可以直接拼接,是&的关系
var gi3 = str3.replace(/a/gi,"1");
 i => 6
 g => -1  上面没有大写的L 所以返回 -1
 g2 => 3
 m2 => 3
 str3 => AABB 替换后，原来的字符串是不变的，返回的string才变了
 i3 => 1ABB
 gi3 => 11BB
```

以上是字符串的方法。

对于正则表达式，JavaScript 里有一个预定义了属性和方法的正则表达式对象 RegExp 。

该对象有两个方法：test() 和 exec().

现在尝试一下：

```javascript
var rep = /a/i;
// 另种写法,new 一个RegExp 对象,也可以省略,用上面的写法也行
// var rep = new RegExp(/a/i);
// 该方法会返回一个boolean 类型,如果有符合正则表达式返回true,反之返回false
var test = rep.test("abcd");
// 该方法会返回一个数组
var exec = rep.exec("abcd");
test => true
exec => [ 'a', index: 0, input: 'abcd' ]
```

# JSON

JSON是JavaScript Object Notation的缩写，它是一种数据交换格式。

JSON实际上是JavaScript的一个子集。在JSON中，一共就这么几种数据类型：

- number：和JavaScript的`number`完全一致；
- boolean：就是JavaScript的`true`或`false`；
- string：就是JavaScript的`string`；
- null：就是JavaScript的`null`；
- array：就是JavaScript的`Array`表示方式——`[]`；
- object：就是JavaScript的`{ ... }`表示方式。

以及上面的任意组合。

并且，JSON还定死了字符集必须是UTF-8，表示多语言就没有问题了。为了统一解析，JSON的字符串规定必须用双引号`""`，Object的键也必须用双引号`""`。

由于JSON非常简单，很快就风靡Web世界，并且成为ECMA标准。几乎所有编程语言都有解析JSON的库，而在JavaScript中，可以直接使用JSON，因为JavaScript内置了JSON的解析。

## 序列化

```javascript
const person = {
    name:"Tom",
    age:"22",
    phone:12345678910
};
const personString = JSON.stringify(person); 
// "name":"Tom","age":"22","phone":12345678910}
```

要输出得好看一些，可以加上参数，按缩进输出：

```javascript
JSON.stringify(person,null,"  ");
{
  "name": "Tom",
  "age": "22",
  "phone": 12345678910
}
```

第二个参数用于控制如何筛选对象的键值，如果我们只想输出指定的属性，可以传入`Array`：

```javascript
JSON.stringify(person,["name","age"],"  ");
{
  "name": "Tom",
  "age": "22"
}
```

