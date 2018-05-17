## 前言

类型转换在各个语言中都存在，而在 JavaScript 中由于缺乏对其的了解而不慎在使用中经常造成bug被人诟病。为了避免某些场景下的意外，甚至推崇直接使用 Strict Equality( === )来代替 ==。这确实能避免很多bug，但更是一种对语言不理解的逃避(个人观点)。

## 引入

先抛出在 You Don't Know JavaScript (中) 看到的一个例子
```javascript
  [] == [] // false
  [] == ![] // true
  {} == !{} // false
  {} == {} // false
```
是不是很奇怪？本文将从书中看到的知识与规范相结合，来详细说明一下JavaScript在类型转换时候发生的故事。

## 类型转换

很多人喜欢说显示类型转换与隐式类型转换，但个人感觉只是说法上的不同，实质都在发生了类型转换而已，故不想去区分他们了（感觉一万个人有一万种说法）

> 仅在6大基本类型 null undefined number boolean string object 作讨论 symbol未考虑

- ### 举个栗子
  ```javascript
  var a = String(1)
  var b = Number('1')
  var c = 1 + ''
  var d = +'1'
  ```
  a,b直接调用了原生函数，发生了类型转换。c,d使用了+运算符的一些规则，发生了类型转换。这些是很简单的也是我们常用的。

  其实真正起作用的，是语言内部对规范中抽象操作的实现，接下来我们所说的 ```ToString, ToNumber, ToBoolean``` 等都是抽象操作，而不是JS里对应的内置函数

- ### ToString - 规范9.8

  按照以下规则转化被传递的参数
  Argument Type | Result
  --- | ---
  Undefined | "undefined"
  Null | "null"
  Boolean | true -> "true" <br> false - > "false"  
  Number | NaN -> "NaN" <br> +0 -0 -> "0" <br> -1 -> "-1" <br> infinity -> "Infinity" <br> 较大的数科学计数法 (详见规范9.8.1)
  String | 不转换 直接返回
  Object | 1. 调用ToPrimitive抽象操作, hint 为 String 将返回值作为 value <br> 2. 返回toString(value)

  ```javascript
  String(undefined) // "undefined"
  String(null) // "null"
  String(true) // "true"
  ```
  > ToPrimitive 抽象操作下面会提及

- ### ToNumber - 规范9.3

  按照以下规则转换被传递参数
  Argument Type | Result
  --- | ---
  Undefined | NaN
  Null | +0
  Boolean | true -> 1 <br> false -> +0
  Number | 直接返回
  String | 如果不是一个字符串型数字，则返回NaN(具体规则见规范9.3.1)
  Object | 1. 调用ToPrimitive抽象操作, hint 为 Number 将返回值作为 value <br> 2. 返回toString(value)

- ### ToBoolean - 规范9.2

  按照以下规则转换被传递参数
  Argument Type | Result
  --- | ---
  Undefined | false
  Null | false
  Boolean | 直接返回
  Number | +0 -0 NaN -> false <br> 其他为true
  String | 空字符串(length为0) -> false <br> 其他为true
  Object | true

- ### ToPrimitive - 规范9.1

  顾名思义，该抽象操作定义了该如何将值转为基础类型(非对象)，接受2个参数，第一个必填的要转换的值，第二个为可选的hint，暗示被转换的类型。

  按照以下规则转换被传递参数
  Argument Type | Result
  --- | ---
  Undefined | 直接返回
  Null | 直接返回
  Boolean | 直接返回
  Number | 直接返回
  String | 直接返回
  Object | 返回一个对象的默认值。一个对象的默认值是通过调用该对象的内部方法[[DefaultValue]]来获取的，同时传递可选参数hint。

- ### [[DefaultValue]] (hint) - 规范8.12.8

  - 当传递的hint为 String 时候，
    - 如果该对象的toString方法可用则调用toStrin
    g
      - 如果toString返回了一个原始值(除了object的基础类型)val，则返回val
    - 如果该对象的valueOf方法可用则调用valueOf方法
      - 如果valueOf返回了一个原始值(除了object的基础类型)val，则返回val

    - 抛出TypeError的异常

  - 当传递的hint为 Number 时候，
    - 如果该对象的valueOf方法可用则调用valueOf方法
      - 如果valueOf返回了一个原始值(除了object的基础类型)val，则返回val
    - 如果该对象的toString方法可用则调用toStrin
    g
      - 如果toString返回了一个原始值(除了object的基础类型)val，则返回val

    - 抛出TypeError的异常
  - hint的默认值为Number，除了Date object
  - 举个栗子
  ```javascript
  var a = {}
  a.toString = function () {return 1}
  a.valueOf = function () {return 2}
  String(a) // "1"
  Number(a) // 2
  a + '' // "2"   ???????
  +a // 2
  a.toString = null
  String(a) // "2"
  a.valueOf = null
  String(a) // Uncaught TypeError: balabala
  ```

  似乎我们发现了一个很不合规范的返回值，为什么 ```a + ''```不应该返回"1"吗
  
  - 问题的答案其实很简单 + 操作符会对两遍的值进行 toPrimitive 操作。由于没有传递 hint 参数，那么就会先调用a.valueOf 得到2后因为+右边是字符串，所以再对2进行ToString抽象操作后与""的字符串拼接。

## 不要畏惧使用 ==

基础概念已经了解了，那么在 == 中到底发生了什么样的类型转换，而导致了经常产生出乎意料的bug，导致了它臭名昭著。

- ### 抽象相等 - 规范11.9.3

  x == y 判断规则如下:

  1. 如果xy类型相同 (与严格相等判断一致，不赘述了，详见规范)
  1. 如果 x 为 null y 为 undefined, 返回true
  1. 如果 x 为 undefined y 为 null, 返回true
  1. 如果 x 类型为 Number, y 类型为 String, 返回 x == ToNumber(y)
  1. 如果 x 类型为 String, y 类型为 Number, 返回ToNumber(x) == y
  1. 如果 x 类型为 Boolean, 返回 ToNumber(x) == y
  1. 如果 y 类型为 Boolean, 返回 x == ToNumber(y)
  1. 如果 x 类型为 String 或 Number, y 类型为 Object, 返回 x == ToPrimitive(y)
  1. 如果 x 类型为 Object, y 类型为 String 或 Number, 返回 ToPrimitive(x) == y
  1. return false

- ### 再看引入

```javascript
  [] == [] // false
  // 1. 两遍类型都为 Object，比较引用地址，不同返回false 搞定
  [] == ![] // true
  // 1. ![]强制类型转换 变为 [] == false
  // 2. 根据规范第7条，返回 [] == ToNumber(false), 即 [] == 0
  // 3. 根据规范第9条，返回ToPromitive([]) == 0，数组的valueOf为本身，不是原始值，则返回toString()即 "" == 0
  // 4. 根据规范第5条，返回ToNumber("") == 0, 即 0 == 0
  // 5. 根据规范第1条，返回 true

  // 下面的不赘述了，分析类似上面
  {} == !{} // false
  {} == {} // false
```

我们不难看出以下几点
- 其实在x y类型相同的时候，== 与 === 没有任何区别。
- 除了undefined与null, 大多数值都会转换为相同类型后进行对比，也就是说 === 是 == 某些情况下必经的步骤

> 引用 << 你不知道的JS(中) >> 中的2句话
> - 如果两遍的值中有 true 或者 false , 千万不要使用 == (会被转为数字0,1来进行判断，会出现一些意外的情况)
> - 如果两遍的值中有[]、""或者0，尽量不要使用 == 

## 抽象比较

先来看看这个例子
```javascript
var a = { b: 42 }
var b = { b: 43 }
a < b // false
a == b // false
a > b // false

a <= b // true
a >= b // true
```
是不是感觉到世界又崩塌了？？？

让我们来仔细分析一下

```javascript
var a = { b: 42 }
var b = { b: 43 }
a < b // false 
// 1. 两遍调用ToPrimitive, 返回[object Object] 两遍一致 返回 false
a == b // false
// 两遍不同的引用，返回false
a > b // false
// 同 a < b

a <= b // true
// 按规范其实是处理成 !(a > b) 所以为true
a >= b // true
```

所以在不相等比较的时候，我们最后还是进行手动的类型转换较为安全

## 总结

深入了解类型转换的规则，我们就可以很容易取其精华去其糟粕，写出更安全也更简洁可读的代码。

## 参考

- 你不知道的JS(中)
- [ecma-262 5.1规范](http://www.ecma-international.org/ecma-262/5.1/index.html)