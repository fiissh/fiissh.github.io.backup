---
title: Flutter 开发（00）：Dart 语言语法预览
permalink: dart-language-language-tour
comments: true
date: 2019-02-21 15:14:40
updated: 2019-02-21 15:14:40
tags:
  - Dart 语法预览
categories:
  - Flutter
---

本文主要针对 `Dart` 语言的语法规则以及常用概念进行介绍，主要目标是快速了解 `Dart` 的主要特性。

本文中介绍的 `Dart` 语言特性是基于 `Dart 2.1.0`。由于肥肥本身从事于 `Android` 开发多年，使用 `Java` 作为主要开发语言，所以在整理、撰写本文时会选择性的忽略一些语言特性的介绍。

本文是基于官网文档 [A Tour of the Dart Language](https://www.dartlang.org/guides/language/language-tour)以及 [Dart 翻译小组](http://dart.goodev.org)翻译的 [Dart 语法概览](http://dart.goodev.org/guides/language/language-tour) 文档。

感谢先驱们对 `Dart` 社区做出的贡献，更感谢 [Dart 翻译小组](http://dart.goodev.org)所付出的辛苦和贡献。

在 `Dart` 中一些重要的概念如下：

* 所有的东西都是对象，所有的对象都是类的实例。即使数字、方法、`null` 也都是对象。所有的对象都继承自 `Object` 类
* 指定静态类型表明你的意图，并使检查类型检查成为可能
* `Dart` 在运行前解析所有的代码，可以使用些小技巧，例如：通过使用类型或编译时常量，来捕捉错误或使代码运行的更快
* `Dart` 支持顶级的函数，也支持类或对象的静态和实例方法。也可以在函数内部嵌套函数或本地函数
* `Dart` 支持顶级的变量，也支持类或对象的静态变量和实例变量(也被称作字段或属性)
* `Dart` 没有 `public`、`protected`、`private` 等关键字，如果一个标识符以 `_` 开头则表示私有
* 标识符以小写字母或下划线 `_` 开头，后面跟着字符和数字的任意组合
* `Dart` 中，明确区分表达式和语句
* `Dart tools` 会报告两种类型的问题：警告(`warnings`)和错误(`errors`)。警告仅标志着你的代码可能不会工作，但并不会阻止程序执行；错误可能是编译时错误，也可能是运行时错误。编译时错误会阻止程序执行；运行时错误会在程序执行时抛出异常
* `Dart` 有两种运行时模式：生产模式和检查模式。推荐在开发和 `debug` 时使用检查模式，生产环境中生产模式。生产模式是 `Dart` 程序默认的运行时模式

<!--more-->
## 关键字

`Dart` 语言中有三种类型的关键字：

* 内置关键字：内置关键字的主要用作是方便 `JavaScript` 的代码向 `Dart` 移植，见下表<font color=red>红色字体</font>所示关键字
* 异步相关的关键字：`Dart 1.0` 之后为了支持异步特性而增加的关键字，见下表<font color=green>绿色字体</font>所示关键字
* 保留关键字：上述两种关键字之外的其他关键字为保留关键字，见下表<font color=blue>蓝色色字体</font>所示关键字



| 关键字 | 关键字 | 关键字 | 关键字 | 关键字 |
| :-: | :-: | :-: | :-: | :-: |
| <font color=red>abstract</font> | <font color=blue>continue</font> | <font color=blue>false</font> | <font color=blue>new</font> | <font color=blue>this</font> |
| <font color=red>as</font> | <font color=blue>default</font> | <font color=blue>final</font> | <font color=blue>null</font> | <font color=blue>throw</font> |
| <font color=blue>assert</font> | <font color=red>deferred</font> | <font color=blue>finally</font> | <font color=red>operator</font> | <font color=blue>true</font> |
| <font color=green>async</font> | <font color=blue>do</font> | <font color=blue>for</font> | <font color=red>part</font> | <font color=blue>try</font> |
| <font color=green>async*</font> | <font color=red>dynamic</font> | <font color=red>get</font> | <font color=blue>rethrow</font> | <font color=red>typedef</font> |
| <font color=green>await</font> | <font color=blue>else</font> | <font color=blue>if</font> | <font color=blue>return</font> | <font color=blue>var</font> |
| <font color=blue>break</font> | <font color=blue>enum</font> | <font color=red>implements</font> | <font color=red>set</font> | <font color=blue>void</font> |
| <font color=blue>case</font> | <font color=red>export</font> | <font color=red>import</font> | <font color=red>static</font> | <font color=blue>while</font> |
| <font color=blue>catch</font> | <font color=red>external</font> | <font color=blue>in</font> | <font color=blue>super</font> | <font color=blue>with</font> |
| <font color=blue>class</font> | <font color=blue>extends</font> | <font color=blue>is</font> | <font color=blue>switch</font> | <font color=green>yield</font> |
| <font color=blue>const</font> | <font color=red>factory1</font> | <font color=red>library</font> | <font color=green>sync*</font> | <font color=green>yield*</font> |

## 变量

可以使用 `var` 声明一个变量，编译器会自动推导其变量类型：

```dart
var name = 'fiissh.zhao';
```

上述代码表示，一个名字为 `name` 的变量引用了一个内容为 `fiissh.zhao` 的 `String` 对象。

相应的，我们可以使用具体的类型来声明一个变量：

```dart
String name = 'fiissh.zhao';
```

如果一个变量声明之后，我们将不会对变量产生修改，那么可以使用 `final` 或 `const` 关键字修饰变量：

* 一个使用 `final` 修饰的变量只能被赋值一次
* 一个使用 `const` 修饰的变量指的是编译时常量

> 默认的，`Dart` 中没有初始化的变量将获得默认值 `null`。这与 `Java` 中的 `Object` 类型的对象的默认值是一样的。需要注意的是，与 `Java` 相比，`Dart` 中所有的对象（包括数字、方法以及 `null`）都继承自 `Object` 类。
> `Dart` 中 `final` 的用法与 `Java` 中的 `final` 的用法相似，都是表示一个不可变的变量。`Dart` 中 `const` 的用法则与 `Java` 中的 `static final` 的用法相似，用于表示一个常量。关于 `final` 和 `const` 的使用此处不做过多介绍。

## 内置类型

`Dart` 支持如下内置类型：

* 数字类型
* 字符串类型
* 布尔类型
* 数组类型
* 哈希类型
* `runes`
* `symbols`

### 数字类型

`Dart` 支持如下两种类型的数字：

* `int`：整型数值，其取值范围为 $ -2^{53} $ 到 $ 2^{53} $ 之间
* `double`：64位双精度浮点数，符合 `IEEE 754` 标准

> `int` 和 `double` 都是 `num` 的子类。`num` 类定义了基本的操作。如果 `num` 中定义的操作不能满足需求，则可以考虑使用 `dart:math` 库。
> `String` 类型的数据与 `num` 类型的相互转换可以通过 `num` 的 `parse` 方法和 `toString` 方法：
> ```Dart
> // String -> int
> var one = int.parse('1');
>
> // String -> double
> var onePointOne = double.parse('1.1');
>
> // int -> String
> String oneAsString = 1.toString();

> // double -> String
> String piAsString = 3.14159.toStringAsFixed(2);
> ```

与 `Java` 一样，`int` 类型的数据的可以支持位移操作：

```dart
assert((3 << 1) == 6);  // 0011 << 1 == 0110
assert((3 >> 1) == 1);  // 0011 >> 1 == 0001
assert((3 | 4)  == 7);  // 0011 | 0100 == 0111
```

> `assert` 表示断言。断言只在检查模式下有效，而在生产模式则无效。

使用 `const` 修饰的数值类型，其数学运算的结果也是 `const` 类型的：

```dart
const msPerSecond = 1000;
const secondsUntilRetry = 5;
const msUntilRetry = secondsUntilRetry * msPerSecond;
```

### 字符串类型

`Dart` 中字符串使用 `UTF-16` 编码，可以使用单引号（`''`）或者双引号（`""`）来初始化：

```dart
var s1 = 'Single quotes work well for string literals.';
var s2 = "Double quotes work just as well.";
var s3 = 'It\'s easy to escape the string delimiter.';
var s4 = "It's even easier to use the other delimiter.";
```

如果需要在字符串中使用表达式，可以使用 `${expression}`。如果表达式的结果是一个非字符串对象，那么可以使用 `toString` 来获取字符串：

```dart
var s = 'string interpolation';
assert('Dart has $s, which is very handy.' ==
       'Dart has string interpolation, ' +
       'which is very handy.');
assert('That deserves all caps. ' +
       '${s.toUpperCase()} is very handy!' ==
       'That deserves all caps. ' +
       'STRING INTERPOLATION is very handy!');
```

> 需要注意的是，`==` 操作符判断两个对象的内容是否一样。如果两个字符串包含一样的字符编码序列， 则他们是相等的。使用 `+` 操作符可以把多个字符串拼接为一个字符串。而且，如果字符串中使用的表达式仅仅是一个标识符，那么可以省略 `{}`

使用三个单引号（`'''`）或者双引号（`"""`）可以创建一个多行的字符串对象：

```dart
var s1 = '''
You can create
multi-line strings like this one.
''';

var s2 = """This is also a
multi-line string.""";
```

通过提供一个 `r` 前缀可以创建一个`原始 raw` 字符串：

```dart
var s = r"In a raw string, even \n isn't special.";
```

> 关于`原始 raw` 字符串，将会在 `Runes` 相关的章节中介绍。

与 `num` 类型相似的，如果使用 `const` 修饰 `Strings` 类型，其运算结果也是 `const` 类型：

```dart
const aConstNum = 0;
const aConstBool = true;
const aConstString = 'a constant string';

const validConstString = '$aConstNum $aConstBool $aConstString';
```

### 布尔类型

`Dart` 使用 `bool` 类型的子类型 `true` 和 `false` 表示布尔类型的数据，而且他们都是 `const` 类型的。

在 `Dart` 中，只有 `true` 对象才被真正认定为 `true`，其他的所有的对象都是 `false`。

### 数组类型

`Dart` 中使用的数组对象是 `List`：

```dart
var list = [1 , 2, 3];
```

与 `Java` 中的数组一样，`Dart` 中的 `List` 下标索引也是从 0 开始的，而且索引数据的方式也与数组类似：

```dart
var i = list[0];

assert(i == 1);
```

如果在 `List` 的初始化时使用 `const` 修饰，那么可以创建一个不变的 `List` 对象：

```dart
var list = const [1 , 2, 3];
```

### 哈希类型

`Dart` 中使用的键值类型是 `Map`：

* 键和值可以是任何类型的对象
* 每个键只能出现一次，而值则可以出现多次

可以使用如下两种方式声明一个 `Map` 对象：

```dart
var gifts = {
// Keys      Values
  'first' : 'partridge',
  'second': 'turtledoves',
  'fifth' : 'golden rings'
};

var gifts = new Map();
gifts['first'] = 'partridge';
gifts['second'] = 'turtledoves';
gifts['fifth'] = 'golden rings';

```

相应的，可以通过如下方式读取 `Map` 中的对象：

```dart
var object = gifts['first'];
```

如果在 `Map` 的初始化时使用 `const` 修饰，那么可以创建一个不变的 `Map` 对象：

```Dart
const constantMap = const {
  2: 'helium',
  10: 'neon',
  18: 'argon',
};
```

### Runes

在 `Dart` 中，`runes` 代表字符串的 `UTF-32` 编码。

> `Unicode` 为每一个字符、标点符号、表情符号等都定义了 一个唯一的数值。 由于 `Dart` 字符串是 `UTF-16` 字符编码，，所以如果在字符串中表达 `32-bit Unicode`（即 `UTF-32` 编码） 值就需要 新的语法了。

通常使用 `\uXXXX` 的方式来表示 `Unicode` 编码（`XXXX` 表示4个16进制的数）。 例如，心形符号 (♥) 是 `\u2665`。对于那么不是4个数值的情况， 把编码值放到大括号中即可。 例如，笑脸 `emoji` (😆) 是 `\u{1f600}`。


`String` 类有一些属性可以提取 `runes` 信息。 `codeUnitAt` 和 `codeUnit` 属性返回 `UTF-16` 编码。

下列实例演示了 `runes`、`UTF-16` 编码 和 `UTF-32` 编码之间的关系：

```dart
main() {
  var clapping = '\u{1f44f}';
  print(clapping);
  print(clapping.codeUnits);
  print(clapping.runes.toList());

  Runes input = new Runes(
      '\u2665  \u{1f605}  \u{1f60e}  \u{1f47b}  \u{1f596}  \u{1f44d}');
  print(new String.fromCharCodes(input));
}
```

运行之后输出：

```CONSOLE
👏
[55357, 56399]
[128079]
♥  😅  😎  👻  🖖  👍
```

### Symbols

在 `Dart` 中，一个 `Symbol` 对象表示程序中声明的操作符和标识符。在标识符前面增加一个 `#` 用于表示 `Symbol` 对象：

```dart
#radix
#bar
```

在 `Dart`中，`Symbol` 是使用较少的特性。在混淆之后的代码中，由于使用 `Symbol` 的名字不会改变，这使得我们可以方便的使用名字来引用标识符。

## 方法

在 `Dart` 中，方法是 `Function` 对象，这也就意味着我们可以给方法赋值，甚至可以把方法作为参数进行传递。

方法的定义方式如下：

```dart
bool isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}
```

当然，我们也可以忽略方法返回值和参数列表的类型定义：

```dart
isNoble(atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}
```

对于只有一个表达式的方法，你可以选择 使用缩写语法来定义：

```dart
bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;
```

这个 `=>` 语法是 `{ return expr; }` 形式的缩写。

方法可以有两种类型的参数：必需的和可选的。 必需的参数在参数列表前面，后面是可选参数。

### 可选参数

可选参数可以是命名参数或者基于位置的参数，但是这两种参数不能同时当做可选参数。

#### 可选命名参数

调用方法的时候，我们可以使用 `param_name:value` 的形式来指定命名参数：

```dart
enableFlags(bold: true, hidden: false);
```

在定义方法的时候，可以使用 `{param_1,param_2,param_3,...}` 的形式来指定命名参数：

```dart
enableFlags({bool bold, bool hidden}) {
  // ...
}
```

#### 可选位置参数

把一些方法的参数放到 `[]` 中就可以变成可选位置参数：

```dart
String say(String from, String msg, [String device]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  return result;
}
```

下面是不使用可选位置参数调用上述方法的示例：

```dart
assert(say('Bob', 'Howdy') == 'Bob says Howdy');
```

下面是使用可选位置参数调用上述方法的示例：

```dart
assert(say('Bob', 'Howdy', 'smoke signal') == 'Bob says Howdy with a smoke signal');
```

#### 默认参数值

在定义方法的时候，可以使用 `=` 来定义可选参数的默认值。默认值只能是 `const` 类型（编译时常量）。如果没有提供默认值，那么参数默认为 `null`。

下面是为可选命名参数设置默认值的示例：

```dart
void enableFlags({bool bold = false, bool hidden = false}) {

}
```

下面是为可选位置参数设置默认值的示例：

```dart
String say(String from, String msg,
    [String device = 'carrier pigeon', String mood]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  if (mood != null) {
    result = '$result (in a $mood mood)';
  }
  return result;
}
```

下面示例是当方法中使用了 `List` 或 `Map` 时设置默认值的方式：

```dart
void doStuff(
    {List<int> list = const [1, 2, 3],
    Map<String, String> gifts = const {
      'first': 'paper',
      'second': 'cotton',
      'third': 'leather'
    }}) {
  print('list:  $list');
  print('gifts: $gifts');
}
```

### 入口函数

每一个 `Dart` 应用程序都有一个顶级的 `main()` 方法的入口才能运行。`main()` 方法的返回值为 `void` 并且有一个可选的 `List<String>` 参数：

```dart
void main() {
  for (var i = 0; i < 4; i++) {
    print('hello $i');
  }
}
```

### 匿名方法

在 `Dart` 中。大部分方法是有名字的，例如 `main()` 方法。但是我们也可以创建一个匿名方法（也称为 `lambda` 或者 `closure` 闭包）：

```dart
([[Type] param1[, …]]) {
  codeBlock;
};
```

与非匿名方法类似的，我们可以在括号之间定义一些参数，参数之间使用逗号分隔，而大括号之间则是方法体。

下面的代码定义了一个参数为 `i`（该参数没有指定类型）的匿名方法。`list` 中的每个元素都会调用这个函数来 打印出来，同时来计算了每个元素在 `list` 中的索引位置：

```dart
var list = ['apples', 'oranges', 'grapes', 'bananas', 'plums'];
list.forEach((i) {
  print(list.indexOf(i).toString() + ': ' + i);
});
```

### 静态作用域

由于 `Dart` 是静态语言，所以变量的作用域在写代码的时候就已经确定了。通常情况下，大括号中定义的变量只能在大括号中访问，其作用域与 `Java` 类似。

下面是作用域的一个示例：

```dart
var topLevel = true;

main() {
  var insideMain = true;

  myFunction() {
    var insideFunction = true;

    nestedFunction() {
      var insideNestedFunction = true;

      assert(topLevel);
      assert(insideMain);
      assert(insideFunction);
      assert(insideNestedFunction);
    }
  }
}
```

> 需要注意的是，`nestedFunction()` 可以访问所有的变量，包含顶级变量。

### 闭包语法

一个闭包是一个方法对象，不管该对象在何处被调用，该对象都可以访问其作用域内的变量。方法可以封闭定义在其作用域内的变量：

```dart
Function makeAdder(num addBy) {
  return (num i) => addBy + i;
}

main() {
  // Create a function that adds 2.
  var add2 = makeAdder(2);

  // Create a function that adds 4.
  var add4 = makeAdder(4);

  assert(add2(3) == 5);
  assert(add4(3) == 7);
}
```

在上述示例中，`makeAdder()` 捕获到了变量 `addBy`，不管我们在何处执行 `makeAdder()` 方法所返回的方法（`(num i) => addBy + i`），都可以使用 `addBy` 参数。

### 返回值

在 `Dart` 中，所有的方法都有返回值。如果没有指定返回值，则默认把 `return null;` 作为方法最后一行的语句执行。

## 操作符

下表展示了 `Dart` 中定义的操作符：

| 描述 | 操作符 |
| --- | --- |
| unary postfix  | `expr++`、`expr--`，`()`、`[]`、`.`、`?.` |
| unary prefix | `-expr`、`!expr`、`~expr`、`++expr`、`--expr` |
| multiplicative | `*`、`/`、`%`、`~/` |
| additive | `+`、`-` |
| shift | `<<`、`>>` |
| bitwise AND | `&` |
| bitwise XOR | `^` |
| bitwise OR | `|` |
| relational and type test | `>=`、`>`、`<=`、`<`、`as`、`is`、`is!` |
| equality | `==`、`!=` |
| logical AND | `&&` |
| logical OR | `||` |
| if null | `??` |
| conditional | `expr1 ? expr2 : expr3` |
| cascade | `..` |
| assignment | `=`、`*=`、`/=`、`~/=`、`%=`、`+=`、`-=`、`<<=`、`>>=`、`&=`、`^=`、`|=`、`??=` |

上述表格所列的操作符都是按照优先级顺序从左到右，从上到下的方式来列出的，上面和左边的操作符优先级要高于下面和右边的。

例如 `%` 操作符优先级高于 `==`，而 `==` 高于 `&&`。所以下面的代码结果是一样的：

```dart
// 1: Parens improve readability.
if ((n % i == 0) && (d % i == 0))

// 2: Harder to read, but equivalent.
if (n % i == 0 && d % i == 0)
```

### 算术操作符

`Dart` 支持常见的算术操作符：


| 操作符 | 解释 |
| --- | --- |
| `+` | 加号 |
| `-` | 减号 |
| `-expr` | 负号 |
| `*` | 乘号 |
| `/` | 除号 |
| `~/` | 除号（返回值为0） |
| `%` | 取模 |
| `++` | 自增操作 |
| `--` | 自减操作 |

> 需要注意的是，`++` 和 `--` 操作符在变量的前后位置不一样，其运算过程不一样。例如 `++var` 和 `var++` 运算，其表达式都等同于 `var = var +1`。但是 `++var` 运算时先执行 `var + 1` 然后再执行运算，而 `var++` 则正好相反，是先执行运算操作，然后再执行 `var + 1` 操作。这与 `Java` 中的自增自减运算是一致的。

### 等式与不等式操作符

`Dart` 支持常见的等式与不等式操作符：


| 操作符 | 解释 |
| --- | --- |
| `==` | 相等 |
| `!=` | 不相等 |
| `>` | 大于 |
| `>=` | 大于等于 |
| `<` | 小于 |
| `<=` | 小于等于 |


如果需要比较两个对象是否为同样的内容，使用 `==` 操作符。下面是 == 操作符工作原理解释：

* 如果 `x` 或 `y` 是 `null`，如果两个都是 `null` 则返回 `true`，如果 只有一个是 `null` 返回 `false`
* 返回如下函数的返回值 `x.==(y)`

> 在某些情况下，如果我们需要判断两个对象是否是同一个对象，则使用 `identical()` 方法。

### 类型判断操作符

`Dart` 支持如下类型判断操作符：

| 操作符 | 解释 |
| --- | --- |
| `as` | 类型转换 |
| `is` | 如果对象是指定的类型返回 `true` |
| `is!` | 如果对象是指定的类型返回 `false` |

只有当 `obj` 实现了 `T` 的接口， `obj is T` 才会返回 `true`。例如 `obj is Object` 总是返回 `true`。

使用 `as` 操作符把对象转换为特定的类型。一般情况下，你可以把它当做用 `is` 判定类型然后调用所判定对象的方法的缩写形式。例如下面的示例：

```dart
if (emp is Person) { // Type check
  emp.firstName = 'Bob';
}
```

使用 `as` 操作符可以简化上面的代码：

```dart
(emp as Person).firstName = 'Bob';
```

> 需要注意的是，上述两个代码片段是有区别的，如果 `emp` 为 `null` 或者不是 `Person` 类型，第一个代码片段的 `if` 判断就不会成立，`emp.firstName = 'Bob';` 则不会被执行。而第二个代码片段则会抛出一个异常。

### 赋值操作符

`Dart` 支持使用 `=` 和 `??=` 进行赋值操作。`??=` 的作用是为值为 `null` 的对象赋值：

```dart
a = value;   // 给 a 变量赋值
b ??= value; // 如果 b 是 null，则赋值给b，如果不是 null，则 b 的值保持不变
```

`Dart` 中还支持如下复合赋值操作符：


| 操作符 | 解释 |
| --- | --- |
| `–=` | `a -= b` 等同于 `a = a-b` |
| `/=` | `a /= b` 等同于 `a = a/b` |
| `%=` | `a %= b` 等同于 `a = a%b` |
| `>>=` | `a >>= b` 等同于 `a = a>>b` |
| `^=` | `a %^= b` 等同于 `a = a^b` |
| `+=` | `a += b` 等同于 `a = a+b` |
| `*=` | `a *= b` 等同于 `a = a*b` |
| `~/=` | `a ~/= b` 等同于 `a = a~/b` |
| `<<=` | `a <<= b` 等同于 `a = a<<b` |
| `&=` | `a &= b` 等同于 `a = a&b` |
| `|=` | `a |= b` 等同于 `a = a|=b` |

> 复合赋值操作符是作用于左侧操作数的。表达式 `a op= b` 等同于 `a op= b`。即 `a += b` 等同于 `a = a+b`

### 逻辑操作符

`Dart` 中支持如下逻辑操作符：

| 操作符 | 解释 |
| --- | --- |
| `!expr` | 对表达式结果取反（`true` 变为 `false` ，`false` 变为 `true`） |
| `||` | 逻辑 `OR` （逻辑或） |
| `&&` | 逻辑 `AND` （逻辑与） |

### 位移操作符

`Dart` 中支持如下位移操作符：

| 操作符 | 解释 |
| --- | --- |
| `&` | 按位 `AND` （按位与） |
| `|` | 按位 `OR` （按位或） |
| `^` | 按位 `!OR` 按位异或 |
| `~expr` | 按位非 |
| `<<` | 左移 |
| `>>` | 右移 |

### 条件表达式

`Dart` 中支持 `if-else` 条件表达式。另外，`Dart` 还提供额外的两种条件表达式写法用于代替 `if-else` 表达式：

* `condition ? expr1 : expr2`：如果 `condition` 是 `true`，执行 `expr1` (并返回执行的结果)；否则执行 `expr2` 并返回其结果
* `expr1 ?? expr2`：如果 `expr1` 是 `non-null`，返回其值；否则执行 `expr2` 并返回其结果

### 级联操作符

在 `Dart` 中，级联操作符（`..`）允许我们在同一个对象上连续调用多个函数以及访问成员变量。使用级联操作符可以避免创建临时变量：

```dart
querySelector('#button') // Get an object.
  ..text = 'Confirm'   // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));
```

第一个方法 `querySelector()` 返回了一个 `selector` 对象。后面的级联操作符都是调用这个对象的成员，并忽略每个操作 所返回的值。

下面的代码在功能上等同于上述代码：

```dart
var button = querySelector('#button');
button.text = 'Confirm';
button.classes.add('important');
button.onClick.listen((e) => window.alert('Confirmed!'));
```

下面的示例展示了级联调用的嵌套使用情况：

```dart
final addressBook = (new AddressBookBuilder()
      ..name = 'jenny'
      ..email = 'jenny@example.com'
      ..phone = (new PhoneNumberBuilder()
            ..number = '415-555-0100'
            ..label = 'home')
          .build())
    .build();
```

### 其他操作符

`Dart` 中还有一些其他的操作符：

| 操作符 | 名称 | 解释 |
| --- | --- | --- |
| `()` | 使用方法 | 代表调用一个方法 |
| `[]` | 访问 `List` | 访问 `List` 中特定位置的元素 |
| `.` | 访问元素 | `foo.bar` 代表访问 `foo` 中的元素 `bar` |
| `?.` | 条件成员访问 | 与 `.` 类似，但是左边的操作对象不能为 `null` |

## 流程控制

`Dart` 中支持如下几种流程控制语法：

* `if-else` 条件判断
* `for` 循环
* `while` 和 `do-while` 循环
* `break` 和 `continue`
* `switch` 条件判断
* `assert` 断言

### if-else 条件判断

`Dart` 中支持 `if` 语句以及可选的 `else` 语句：

```dart
if (isRaining()) {
  you.bringRainCoat();
} else if (isSnowing()) {
  you.wearJacket();
} else {
  car.putTopDown();
}
```

### for 循环

`Dart` 中支持如下 `for` 循环语句：

```dart
var message = new StringBuffer("Dart is fun");
for (var i = 0; i < 5; i++) {
  message.write('!');
}
```

如果要遍历的对象实现了 `Iterable` 接口，则可以使用 `forEach()` 方法。如果没必要当前遍历的索引，则使用 `forEach()` 方法是个非常好的选择：

```dart
candidates.forEach((candidate) => candidate.interview());
```

`List` 和 `Set` 等实现了 `Iterable` 接口的类还支持 `for-in` 形式的遍历：

```dart
var collection = [0, 1, 2];
for (var x in collection) {
  print(x);
}
```

### while 和 do-while 循环

`Dart` 中，`while` 循环在执行循环之前先判断条件是否满足：

```dart
while (!isDone()) {
  doSomething();
}
```

而 `do-while` 是先执行循环再判断条件是否满足：

```dart
do {
  printLine();
} while (!atEndOfPage());
```

### break 和 continue

`Dart` 中可以实现 `break` 来中断当前的循环：

```dart
while (true) {
  if (shutDownRequested()) break;
  processIncomingRequests();
}
```

而 `continue` 则是中断当前循环并进入下一次循环：

```dart
for (int i = 0; i < candidates.length; i++) {
  var candidate = candidates[i];
  if (candidate.yearsExperience < 5) {
    continue;
  }
  candidate.interview();
}
```

> 上述代码在实现了 `Iterable` 接口的对象上可以采用如下写法：
>
> ```dart
> candidates.where((c) => c.yearsExperience >= 5)
>           .forEach((c) => c.interview());
> ```

### switch 条件判断

在 `Dart` 中可以使用`switch` 语句比较 `integer`、`string`、或者编译时常量。比较的对象必须都是同一个类的实例，`class` 必须没有覆写 `==` 操作符：

```dart
var command = 'OPEN';
switch (command) {
  case 'CLOSED':
    executeClosed();
    break;
  case 'PENDING':
    executePending();
    break;
  case 'APPROVED':
    executeApproved();
    break;
  case 'DENIED':
    executeDenied();
    break;
  case 'OPEN':
    executeOpen();
    break;
  default:
    executeUnknown();
}
```

> 每个非空的 `case` 语句都必须有一个 `break` 语句。另外还可以通过 `continue`、`throw` 或者 `return` 来结束非空 `case` 语句。当没有 `case` 语句匹配的时候，可以使用 `default` 语句来匹配这种默认情况。

如果你需要实现这种继续到下一个 `case` 语句中继续执行，则可以 使用 `continue` 语句跳转到对应的标签（`label`）处继续执行：

```dart
var command = 'CLOSED';
switch (command) {
  case 'CLOSED':
    executeClosed();
    continue nowClosed;
    // Continues executing at the nowClosed label.

nowClosed:
  case 'NOW_CLOSED':
    // Runs for both CLOSED and NOW_CLOSED.
    executeNowClosed();
    break;
}
```

> 需要注意的是，每个 `case` 语句都可以有局部变量，局部变量只有在这个语句内可见。

### assert

在 `Dart` 中，如果条件表达式结果不满足需要，则可以使用 `assert` 语句俩打断代码的执行：

```dart
// Make sure the variable has a non-null value.
assert(text != null);

// Make sure the value is less than 100.
assert(number < 100);

// Make sure this is an https URL.
assert(urlString.startsWith('https'));
```

> 需要注意的是， 断言只在检查模式下运行有效，如果在生产模式运行，则断言不会执行。

`assert` 方法的参数可以为任何返回布尔值的表达式或者方法。如果返回的值为 `true`， 断言执行通过，执行结束。如果返回值为 `false`， 断言执行失败，会抛出一个 `AssertionError` 异常。

## 异常

在 `Dart` 代码中可以通过抛出异常和捕获异常以达到处理某些错误的目的。`Dart` 中提供了 `Exception` 和 `Error` 类型用于表示异常。我们可以抛出已有的异常类型，也可以自定义异常类型。与 `Java` 不同的是，`Dart` 中可以抛出任何 `non-null` 对象为异常。

### throw

在 `Dart` 中使用关键字 `throw` 抛出一个 `FormatException` 异常：

```dart
throw new FormatException('Expected at least 1 section');
```

或者抛出一个 `String` 类型的异常：

```dart
throw 'Out of llamas!';
```

### catch 或 on

在 `Dart` 中可以使用 `catch` 或者 `on` 处理异常。`on` 用于指定异常类型，`catch` 用于捕获异常对象：

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // A specific exception
  buyMoreLlamas();
} on Exception catch (e) {
  // Anything else that is an exception
  print('Unknown exception: $e');
} catch (e) {
  // No specified type, handles all
  print('Something really unknown: $e');
}
```

`catch` 后面可以有一个或者两个参数，第一个参数表示抛出的异常对象，第二个参数表示堆栈信息（`StackTrace`）对象。

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // A specific exception
  buyMoreLlamas();
} on Exception catch (e) {
  // Anything else that is an exception
  print('Unknown exception: $e');
} catch (e, s) {
  // No specified type, handles all
  print('Something really unknown: $e');
  print('Stack trace:\n $s');
}
```

我们可以使用 `rethrow` 关键字把异常重新抛出：

```dart
final foo = '';

void misbehave() {
  try {
    foo = "You can't change a final variable's value.";
  } catch (e) {
    print('misbehave() partially handled ${e.runtimeType}.');
    rethrow; // Allow callers to see the exception.
  }
}

void main() {
  try {
    misbehave();
  } catch (e) {
    print('main() finished handling ${e.runtimeType}.');
  }
}
```

### finally

为了确保在抛出异常时（或者没有抛出异常），某些逻辑能够正常执行，可以使用 `finally` 关键字来确保代码的正常执行：

```dart
try {
  breedMoreLlamas();
} catch(e) {
  print('Error: $e');  // Handle the exception first.
} finally {
  cleanLlamaStalls();  // Then clean up.
}
```

如果省略了 `catch`，则会在 `finally` 执行完毕之后将异常抛出：

```dart
try {
  breedMoreLlamas();
} finally {
  // Always clean up, even if an exception is thrown.
  cleanLlamaStalls();
}
```

## 类

与 `Java` 类似的，`Dart` 中的类也都是派生于 `Object` 超类，并且是单一继承结构的。

在 `Dart` 中通过使用 `new` 关键字和构造函数来创建一个新的对象（构造函数名字可以为 `ClassName` 或者 `ClassName.identifier`）：

```dart
var jsonData = JSON.decode('{"x":1, "y":2}');

// Create a Point using Point().
var p1 = new Point(2, 2);

// Create a Point using Point.fromJson().
var p2 = new Point.fromJson(jsonData);
```

对象的成员包括方法和数据 (方法和实例变量)。当你调用一个函数的时候，你是在一个对象上调用该函数需要访问对象的方法和数据。

使用点(`.`)来引用对象的变量或者方法：

```dart
var p = new Point(2, 2);

// Set the value of the instance variable y.
p.y = 3;

// Get the value of y.
assert(p.y == 3);

// Invoke distanceTo() on p.
num distance = p.distanceTo(new Point(4, 4));
```

或者使用 `?.` 来替代 `.` 以避免当左边对象为 `null` 时抛出异常：

```dart
// If p is non-null, set its y value to 4.
p?.y = 4;
```

有些类提供了常量构造函数。使用常量构造函数可以创建编译时常量，要使用常量构造函数只需要用 `const` 替代 `new` 即可：

```dart
var p = const ImmutablePoint(2, 2);
```

> 在 `Dart` 中，两个一样的编译时常量其实是同一个对象。也就是说 `var a = const ImmutablePoint(1, 1);` 和 `var b = const ImmutablePoint(1, 1);` 中 `a` 和 `b` 实际上是同一个对象（即 `assert(identical(a, b));` 为 `true`）。

可以使用 `Object` 的 `runtimeType` 属性来判断实例的类型，该属性返回一个 `Type` 对象：

```dart
print('The type of a is ${p.runtimeType}');
```

### 变量实例

在 `Dart` 中，所有类的实例变量或默认初始化为 `null`：

```dart
class Point {
  num x; // Declare instance variable x, initially null.
  num y; // Declare y, initially null.
  num z = 0; // Declare z, initially 0.
}
```

每个实例变量会自动生成一个隐式的 `getter` 方法，对于非 `final` 变量还会自动生成一个隐式的 `setter` 方法。下属代码是可以正常运行的：

```dart
main() {
  var point = new Point();
  point.x = 4;          // Use the setter method for x.
  assert(point.x == 4); // Use the getter method for x.
  assert(point.y == null); // Values default to null.
}
```

### 构造函数

定义一个和类名字一样的方法就定义了一个构造函数：

```dart
class Point {
  num x;
  num y;

  Point(num x, num y) {
    // There's a better way to do this, stay tuned.
    this.x = x;
    this.y = y;
  }
}
```

由于把构造函数参数赋值给实例变量的场景太常见了，`Dart` 提供了一个语法糖来简化这个操作：

```dart
class Point {
  num x;
  num y;

  // Syntactic sugar for setting x and y
  // before the constructor body runs.
  Point(this.x, this.y);
}
```

#### 默认构造函数

如果我们没有定义构造函数，则会有个默认构造函数。默认构造函数没有参数，并且会调用超类的没有参数的构造函数。

需要注意的是，在 `Dart` 中子类不会继承父类的构造函数。子类如果没有定义构造函数，那么就只有一个默认构造函数。如果需要调用父类的构造函数，则需要我们在子类中手动调用。在构造函数参数后使用冒号 (`:`) 可以调用 超类构造函数：

```dart
class Person {
  String firstName;

  Person.fromJson(Map data) {
    print('in Person');
  }
}

class Employee extends Person {
  // Person does not have a default constructor;
  // you must call super.fromJson(data).
  Employee.fromJson(Map data) : super.fromJson(data) {
    print('in Employee');
  }
}

main() {
  var emp = new Employee.fromJson({});

  // Prints:
  // in Person
  // in Employee
  if (emp is Person) {
    // Type check
    emp.firstName = 'Bob';
  }
  (emp as Person).firstName = 'Bob';
}
```

#### 命名构造函数

使用命名构造函数可以为一个类实现多个构造函数：

```dart
class Point {
  num x;
  num y;

  Point(this.x, this.y);

  // Named constructor
  Point.fromJson(Map json) {
    x = json['x'];
    y = json['y'];
  }
}
```

#### 初始化列表

在构造函数体执行之前除了可以调用超类构造函数之外，还可以初始化实例参数。使用逗号（`,`）分隔初始化表达式：

```dart
class Point {
  num x;
  num y;

  Point(this.x, this.y);

  // Initializer list sets instance variables before
  // the constructor body runs.
  Point.fromJson(Map jsonMap)
      : x = jsonMap['x'],
        y = jsonMap['y'] {
    print('In Point.fromJson(): ($x, $y)');
  }
}
```

初始化列表非常适合用来设置 `final` 变量的值。下面示例代码中初始化列表设置了三个 `final` 变量的值：

```dart
import 'dart:math';

class Point {
  final num x;
  final num y;
  final num distanceFromOrigin;

  Point(x, y)
      : x = x,
        y = y,
        distanceFromOrigin = sqrt(x * x + y * y);
}

main() {
  var p = new Point(2, 3);
  print(p.distanceFromOrigin);
}
```

#### 重定向构造函数

如果需要在一个构造函数中调用类中的其他构造函数，那么可以使用冒号（`:`）调用其他的构造函数：

```dart
class Point {
  num x;
  num y;

  // The main constructor for this class.
  Point(this.x, this.y);

  // Delegates to the main constructor.
  Point.alongXAxis(num x) : this(x, 0);
}
```

#### 常量构造函数

如果我们的类是一个状态不变的对象，我们可以把这些对象定义为编译时常量。通过对构造函数使用 `const` 关键字并把所有的变量声明为 `final` 来实现：

```dart
class ImmutablePoint {
  final num x;
  final num y;
  const ImmutablePoint(this.x, this.y);
  static final ImmutablePoint origin =
      const ImmutablePoint(0, 0);
}
```

#### 工厂方法构造函数

如果一个构造函数并不总是返回一个新的对象，则使用 `factory` 来定义这个构造函数。例如，一个工厂构造函数可能从缓存中获取一个实例并返回，或者返回一个子类型的实例：

```dart
class Logger {
  final String name;
  bool mute = false;

  // _cache is library-private, thanks to the _ in front
  // of its name.
  static final Map<String, Logger> _cache =
      <String, Logger>{};

  factory Logger(String name) {
    if (_cache.containsKey(name)) {
      return _cache[name];
    } else {
      final logger = new Logger._internal(name);
      _cache[name] = logger;
      return logger;
    }
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) {
      print(msg);
    }
  }
}
```

最后，通过 `new` 关键字来调用工厂构造函数：

```dart
var logger = new Logger('UI');
logger.log('Button clicked');
```

### 方法

#### 实例方法

对象的实例方法可以访问 `this`：

```dart
import 'dart:math';

class Point {
  num x;
  num y;
  Point(this.x, this.y);

  num distanceTo(Point other) {
    var dx = x - other.x;
    var dy = y - other.y;
    return sqrt(dx * dx + dy * dy);
  }
}
```

#### getter 和 setter 方法

`getter` 和 `setter` 方法是用来设置和访问对象属性的特殊函数。每个实例变量都有一个隐式的 `getter` 方法，每一个非 `final` 变量都有一个隐式的 `setter` 方法。我们可以通过使用 `get` 和 `set` 关键字来定义 `getter` 和 `setter` 方法：

```dart
class Rectangle {
  num left;
  num top;
  num width;
  num height;

  Rectangle(this.left, this.top, this.width, this.height);

  // Define two calculated properties: right and bottom.
  num get right             => left + width;
      set right(num value)  => left = value - width;
  num get bottom            => top + height;
      set bottom(num value) => top = value - height;
}

main() {
  var rect = new Rectangle(3, 4, 20, 15);
  assert(rect.left == 3);
  rect.right = 12;
  assert(rect.left == -8);
}
```

#### 抽象方法

在 `Dart` 中，实例函数、`getter` 和 `setter` 方法可以是抽象函数。抽象函数是只定义函数接口但是没有具体实现的函数：

```dart
abstract class Doer {
  // ...Define instance variables and methods...

  void doSomething(); // Define an abstract method.
}

class EffectiveDoer extends Doer {
  void doSomething() {
    // ...Provide an implementation, so the method is not abstract here...
  }
}
```

#### 可复写的操作符

下表中的操作符可以进行复写：

| --- | --- | --- | --- |
| `<` | `+` | `[]` | `|` |
| `<`| `/` | `^`| `[]=` |
| `<=` | `~/` | `&` | `~` |
| `>=` | `*`| `<<` | `==` |
| `-` | `%` | `>>` |  |

下面是覆写了 `+` 和 `-` 操作符的示例：

```Dart
class Vector {
  final int x;
  final int y;
  const Vector(this.x, this.y);

  /// Overrides + (a + b).
  Vector operator +(Vector v) {
    return new Vector(x + v.x, y + v.y);
  }

  /// Overrides - (a - b).
  Vector operator -(Vector v) {
    return new Vector(x - v.x, y - v.y);
  }
}

main() {
  final v = new Vector(2, 3);
  final w = new Vector(2, 2);

  // v == (2, 3)
  assert(v.x == 2 && v.y == 3);

  // v + w == (4, 5)
  assert((v + w).x == 4 && (v + w).y == 5);

  // v - w == (0, 1)
  assert((v - w).x == 0 && (v - w).y == 1);
}
```

### 抽象类

在 `Dart` 中，`abstract` 关键字用于修饰一个抽象类。抽象类通常用来定义接口以及部分实现，如果我们希望我们的抽象类是可实例化的，那么需要定义一个工厂构造函数：

```dart
abstract class AbstractContainer {
  // ...Define constructors, fields, methods...

  void updateChildren(); // Abstract method.
}
```

与 `Java` 中抽象类一样，在 `Dart` 中抽象类是不能实例化的。如果我们在一个普通类中定义了一个抽象函数，则是可以实例化的：

```dart
class SpecializedContainer extends AbstractContainer {
  // ...Define more constructors, fields, methods...

  void updateChildren() {
    // ...Implement updateChildren()...
  }

  // Abstract method causes a warning but
  // doesn't prevent instantiation.
  void doSomething();
}
```

### 实现接口

一个类可以通过 `implements` 关键字来实现一个或者多个接口， 并实现每个接口：

```dart
class Person {
  // In the interface, but visible only in this library.
  final _name;

  // Not in the interface, since this is a constructor.
  Person(this._name);

  // In the interface.
  String greet(who) => 'Hello, $who. I am $_name.';
}

// An implementation of the Person interface.
class Imposter implements Person {
  // We have to define this, but we don't use it.
  final _name = "";

  String greet(who) => 'Hi $who. Do you know who I am?';
}

greetBob(Person person) => person.greet('bob');

main() {
  print(greetBob(new Person('kathy')));
  print(greetBob(new Imposter()));
}
```

### 扩展类

通过使用 `extends` 实现从父类派生一个子类的目的：

```dart
class Television {
  void turnOn() {
    _illuminateDisplay();
    _activateIrSensor();
  }
  // ...
}

class SmartTelevision extends Television {
  void turnOn() {
    super.turnOn();
    _bootNetworkInterface();
    _initializeMemory();
    _upgradeApps();
  }
  // ...
}
```

> `super` 关键字用于访问父类对象

子类可以覆写实例方法，`getter` 和 `setter` 方法。下面是覆写 `Object` 类的 `noSuchMethod()` 方法的例子，如果调用了对象上不存在的方法，则就会触发 `noSuchMethod()` 方法：

```dart
class A {
  @override
  void noSuchMethod(Invocation mirror) {
    // ...
  }
}
```

> `@override` 注解表明该方法是复写的父类方法。

### 枚举类

枚举类是一种特殊类型的类，主要用来表示一个数量固定的常量集。我们可以使用 `enum` 来定义一个枚举类：

```dart
enum Color {
  red,
  green,
  blue
}
```

枚举类中的每个值都有一个 `index` 的 `getter` 方法，该方法返回该值在枚举类型定义中的位置（从 0 开始。枚举的 `values` 常量则可以返回所有的枚举值。

### 扩展类的功能

`mixin` 是一种在多类继承中重用一个类代码的方法。使用 `with` 关键字后面为一个或者多个 `mixin` 名字来使用 `mixin`：

```dart
class Musician extends Performer with Musical {
  // ...
}

class Maestro extends Person
    with Musical, Aggressive, Demented {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}
```

定义一个类继承 `Object`，该类没有构造函数，不能调用 `super` ，则该类就是一个 `mixin`：

```dart
abstract class Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
```

### 静态变量和静态方法

在 `Dart` 中，可以使用 `static` 关键字来实现类级别的变量和方法，它们分别被称为静态变量和静态方法。

#### 静态变量

静态变量在第一次使用的时候才被初始化。静态变量对于类级别的状态是非常有用的：

```dart
class Color {
  static const red = const Color('red'); // A constant static variable.
  final String name;      // An instance variable.
  const Color(this.name); // A constant constructor.
}

main() {
  assert(Color.red.name == 'red');
}
```

#### 静态方法

静态方法可以当做编译时常量使用。例如，我们可以把静态方法当做常量构造函数的参数来使用。静态方法的示例如下：

```dart
import 'dart:math';

class Point {
  num x;
  num y;
  Point(this.x, this.y);

  static num distanceBetween(Point a, Point b) {
    var dx = a.x - b.x;
    var dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
  }
}

main() {
  var a = new Point(2, 2);
  var b = new Point(4, 4);
  var distance = Point.distanceBetween(a, b);
  assert(distance < 2.9 && distance > 2.8);
}
```

## 泛型

在 `Dart` 中，通常使用一个字母来代表泛型参数，例如 `E`、`T`、`S`、`K` 和 `V` 等。


在 `Dart` 中，类型是可选的，我们可以选择不使用泛型。有些情况下我们可能需要使用类型来表明意图，不管是使用泛型还是具体类型。例如，如果我们希望一个 `List` 只包含字符串对象，那么我们可以使用泛型的方式定义 `List<String>`:

```dart
var names = new List<String>();
names.addAll(['Seth', 'Kathy', 'Lars']);
// ...
names.add(42); // Fails in checked mode (succeeds in production mode).
```


另外一个使用泛型的原因是减少重复的代码。泛型可以在多种类型之间定义同一个实现，同时还可以继续使用检查模式和静态分析工具提供的代码分析功能。例如，我们创建一个保存缓存对象的接口：

```dart
abstract class ObjectCache {
  Object getByKey(String key);
  setByKey(String key, Object value);
}
```

随着需求的变化，我们不得不重新定义一个新的缓存类来缓存 `String` 对象：

```dart
abstract class StringCache {
  String getByKey(String key);
  setByKey(String key, String value);
}
```

使用泛型则可以避免上述重复的代码，我们只需要定一个泛型接口：

```dart
abstract class Cache<T> {
  T getByKey(String key);
  setByKey(String key, T value);
}
```

在上面的代码中，`T` 是一个备用类型。这是一个类型占位符，在我们调用该接口的时候需要指定具体类型。

### 在构造函数中使用泛型

在调用构造函数的时候， 在类名字后面使用尖括号(`< >`)来指定 泛型类型：

```dart
var names = new List<String>();
names.addAll(['Seth', 'Kathy', 'Lars']);
var nameSet = new Set<String>.from(names);
```

> `Dart` 中的泛型类型是固化的，在运行时有也可以判断具体的类型。这与 `Java` 是不同的。`Java` 中的泛型信息是编译时的，泛型信息在运行时是不存在的。在 `Java` 中我们可以判断一个对象是否为 `List`， 但是我们无法判断一个对象是否为 `List<String>`。

### 限制泛型类型

在声明泛型时，我们可以通过使用 `extends` 关键字来限制泛型的类型：

```dart
class Foo<T extends SomeBaseClass> {...}

class Extender extends SomeBaseClass {...}

void main() {
  // It's OK to use SomeBaseClass or any of its subclasses inside <>.
  var someBaseClassFoo = new Foo<SomeBaseClass>();
  var extenderFoo = new Foo<Extender>();

  // It's also OK to use no <> at all.
  var foo = new Foo();

  // Specifying any non-SomeBaseClass type results in a warning and, in
  // checked mode, a runtime error.
  // var objectFoo = new Foo<Object>();
}
```

### 泛型方法

在方法中使用泛型的示例如下：

```dart
T first<T>(List<T> ts) {
  // ...Do some initial work or error checking, then...
  T tmp ?= ts[0];
  // ...Do some additional checking or processing...
  return tmp;
}
```

`first (<T>)` 泛型方法可以在如下地方使用参数 `T` ：

* 方法的返回值类型 (`T`)
* 参数的类型 (`List<T>`)
* 局部变量的类型 (`T tmp`)

## 库和可见性

在 `Dart` 中，使用 `import` 和 `library` 指令可以帮助我们创建模块化的可分享的代码。库不仅仅提供访问接口，还是一个私有单元：以下划线 (`_`) 开头的标识符只有在库内部可见。

> 事实上，每个 `Dart app` 都是一个库， 即使没有使用 `library` 命令也是一个库。

库可以使用 `Dart package` 工具部署。参考 [Pub Package 和 Asset Manager](https://www.dartlang.org/tools/pub) 来获取关于 `pub`（`Dart` 的包管理工具） 的更多信息。

### 使用库

在 `Dart` 中，使用 `import` 来指定一个库：

```dart
import 'dart:html';
```

`import` 必须参数为库的 `URI`。 对于内置的库，`URI` 使用特殊的 `dart:scheme`。对于其他的库，我们可以使用文件系统路径或者 `package:scheme`（`package:scheme` 指定的库通过包管理器来提供，例如 `pub` 工具）：

```dart
import 'dart:io';
import 'package:mylib/mylib.dart';
import 'package:utils/utils.dart';
```

### 指定库前缀

如果我们导入的两个库有冲突的标识符，我们可以使用库的前缀来区分。例如，如果 `library1` 和 `library2` 都有一个名字为 `Element` 的类，我们可以这样使用：

```dart
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;
// ...
Element element1 = new Element();           // Uses Element from lib1.
lib2.Element element2 = new lib2.Element(); // Uses Element from lib2.
```

### 导入库的一部分

如果我们只使用库的一部分功能，则可以选择导入需要的内容：

```dart
// Import only foo.
import 'package:lib1/lib1.dart' show foo;

// Import all names EXCEPT foo.
import 'package:lib2/lib2.dart' hide foo;
```

### 延迟加载库

下面是一些使用延迟加载库的场景：

* 减少 `APP` 的启动时间
* 执行 `A/B` 测试，例如尝试各种算法的不同实现
* 加载很少使用的功能，例如可选的屏幕和对话框


要延迟加载一个库，需要先使用 `deferred as` 来导入：

```dart
import 'package:deferred/hello.dart' deferred as hello;
```

当需要使用的时候，使用库标识符调用 `loadLibrary()` 方法来加载库：

```dart
greet() async {
  await hello.loadLibrary();
  hello.printGreeting();
}
```

> 在上述代码中，使用 `await` 关键字暂停代码执行一直到库加载完成。

在一个库上你可以多次调用 `loadLibrary()` 函数。 但是该库只是载入一次。使用延迟加载库的时候，需要注意如下问题：

* 延迟加载库的常量在导入的时候是不可用的。 只有当库加载完毕的时候，库中常量才可以使用
* 在导入文件的时候无法使用延迟库中的类型。 如果你需要使用类型，则考虑把接口类型移动到另外一个库中， 让两个库都分别导入这个接口库
* `Dart` 隐式的把 `loadLibrary()` 方法导入到使用 `deferred as` 的命名空间中。`loadLibrary()` 方法返回一个 `Future` 对象

## 异步支持

`Dart` 有一些语言特性来支持异步编程。最常见的特性是 `async` 方法和 `await` 表达式。

`Dart` 库中有很多返回 `Future` 或者 `Stream` 对象的方法。这些方法是异步的：这些函数在设置完基本的操作后就返回了，而无需等待操作执行完成。例如读取一个文件，在打开文件后就返回了。

有两种方式可以使用 `Future` 对象中的数据：

* 使用 `async` 和 `await`
* 使用 `Future API`

同样的，从 `Stream` 中获取数据也有两种方式：

* 使用 `async` 和一个异步 `for` 循环 (`await for`)
* 使用 `Stream API`

使用 `async` 和 `await` 的代码是异步的：

```dart
await lookUpVersion()
```


要使用 `await`，其方法必须带有 `async` 关键字：

```dart
checkVersion() async {
  var version = await lookUpVersion();
  if (version == expectedVersion) {
    // Do something.
  } else {
    // Do something else.
  }
}
```

可以使用 `try`、`catch` 和 `finally` 来处理使用 `await` 的异常：

```dart
try {
  server = await HttpServer.bind(InternetAddress.LOOPBACK_IP_V4, 4044);
} catch (e) {
  // React to inability to bind to the port...
}
```

### 声明异步方法

一个异步方法是方法体被标记为 `async` 的方法。虽然异步方法的执行可能需要一些时间，但是异步方法在调用之后就立刻返回了————在方法体还没执行之前就返回了：

```dart
checkVersion() async {
  // ...
}

lookUpVersion() async => /* ... */;
```

在一个方法上添加 `async` 关键字，则这个方法返回值为 `Future`。例如，下面是一个返回字符串的同步方法：

```dart
String lookUpVersionSync() => '1.0.0';
```

如果使用 `async` 关键字，则该方法返回一个 `Future`，并且认为该函数是一个耗时的操作：

```dart
Future<String> lookUpVersion() async => '1.0.0';
```

注意，方法体并不需要使用 `Future API`。 `Dart` 会自动在需要的时候创建 `Future` 对象。

### 使用 await 表达式

在`Dart` 中，`await` 表达式具有如下的形式：

```dart
await expression
```

在一个异步方法内可以使用多次 `await` 表达式。例如，下面的示例使用了三次 `await` 表达式来执行相关的功能：

```dart
var entrypoint = await findEntrypoint();
var exitCode = await runExecutable(entrypoint, args);
await flushThenExit(exitCode);
```

在 `await expression` 中， `expression` 的返回值通常是一个 `Future`；如果返回的值不是 `Future`，则 `Dart` 会自动把该值放到 `Future` 中返回。

`Future` 对象代表返回一个对象的 `promise`。 `await expression` 执行的结果为这个返回的对象。`await expression` 会阻塞住，直到需要的对象返回为止。

### 异步 for 循环

在 `Dart` 中，异步 `for` 循环具有如下的形式：

```dart
await for (variable declaration in expression) {
  // Executes each time the stream emits a value.
}
```

上面 `expression` 返回的值必须是 `Stream` 类型的。 执行流程如下：

1. 等待直到 `Stream` 返回一个数据
2. 使用 `Stream` 返回的参数执行 `for` 循环代码
3. 重复执行 1 和 2 直到 `Stream` 数据返回完毕

使用 `break` 或者 `return` 语句可以停止接收 `Stream` 的数据，这样就跳出了 `for` 循环并且从 `Stream` 上取消注册了。

在 `main()` 方法中使用异步 for 循环的示例如下：

```dart
main() async {
  ...
  await for (var request in requestServer) {
    handleRequest(request);
  }
  ...
}
```

> 如果爱 `main()` 方法中使用了 `await`，那么 `main()` 方法也必须使用 `async`。

异步编程的更多信息，请参考 [dart:async](https://www.dartlang.org/guides/libraries/library-tour#dartasync---asynchronous-programming)。

## Callable classes

如果 `Dart` 类实现了 `call()` 方法则可以把类当做方法来调用：

```dart
class WannabeFunction {
  call(String a, String b, String c) => '$a $b $c!';
}

main() {
  var wf = new WannabeFunction();
  var out = wf("Hi","there,","gang");
  print('$out');
}
```

关于更多 `Callable classes` 的信息，请参考 [Emulating Functions in Dart](https://www.dartlang.org/articles/language/emulating-functions)。

### 协程

所有的 `Dart` 代码在协程中运行而不是线程。每个协程都有自己的堆内存，并且确保每个协程的状态都不能被其他协程访问。

### Typedefs

在 `Dart` 语言中，方法也是对象。使用 `typedef` 或者 `function-type alias` 的方式为方法类型命名，然后可以使用命名的方法。当把方法类型赋值给一个变量的时候，`typedef` 保留类型信息。

下面的代码没有使用 `typedef`：

```dart
class SortedCollection {
  Function compare;

  SortedCollection(int f(Object a, Object b)) {
    compare = f;
  }
}

 // Initial, broken implementation.
 int sort(Object a, Object b) => 0;

main() {
  SortedCollection coll = new SortedCollection(sort);

  // 我们只知道 compare 是一个 Function 类型，
  // 但是不知道具体是何种 Function 类型？
  assert(coll.compare is Function);
}
```

当把 `f` 赋值给 `compare` 的时候，类型信息会丢失（`f(Object a, Object b)` 类型变为 `int`）。

如果我们 `typedef`，则可以保留这些信息：

```dart
typedef int Compare(Object a, Object b);

class SortedCollection {
  Compare compare;

  SortedCollection(this.compare);
}

 // Initial, broken implementation.
 int sort(Object a, Object b) => 0;

main() {
  SortedCollection coll = new SortedCollection(sort);
  assert(coll.compare is Function);
  assert(coll.compare is Compare);
}
```

由于 `typedef` 只是别名，`Dart` 还提供了一种判断任意 `function` 的类型的方法：

```dart
typedef int Compare(int a, int b);

int sort(int a, int b) => a - b;

main() {
  assert(sort is Compare); // True!
}
```

### Metadata

我们可以通过使用 `Metadata`（元数据）为代码添加额外的信息。元数据注解是以 `@` 开头，后面紧跟一个编译时常量或者调用一个常量构造函数实现的：

```dart
class Television {
  /// _Deprecated: Use [turnOn] instead._
  @deprecated
  void activate() {
    turnOn();
  }

  /// Turns the TV's power on.
  void turnOn() {
    print('on!');
  }
}
```

我们可以定义自己的元数据注解。下面的示例定义了一个带有两个参数的 `@todo` 注解：

```dart
library todo;

class todo {
  final String who;
  final String what;

  const todo(this.who, this.what);
}
```

使用 `@todo` 注解的示例：

```dart
import 'todo.dart';

@todo('seth', 'make this do something')
void doSomething() {
  print('do something');
}
```

元数据可以在 `library`、`class`、`typedef`、`type parameter`、`constructor`、`factory`、`function`、`field`、`parameter` 和 `variable` 声明之前使用，也可以在 `import` 或者 `export` 指令之前使用。使用反射可以在运行时获取元数据信息。


> 有三个注解所有的 `Dart` 代码都可以使用： `@deprecated`、 `@override` 和 `@proxy`。` @override` 用于表示该方法是复写的父类方法，`@proxy` 用于表示忽略警告，`@deprecated` 用以表示该方法或者字段时废弃的。

## 注释

在 `Dart` 中支持单行、多行以及文档注释。

单行注释以 `//` 开始。 `//` 后面的一行内容为 `Dart` 代码注释。

多行注释以 `/*` 开始， `*/` 结尾。 多行注释可以嵌套。

文档注释可以使用 `///` 开始， 也可以使用 `/**` 开始 并以 `*/` 结束。
