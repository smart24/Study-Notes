![Kotlin 基本数据类型分类](http://m.qpic.cn/psc?/V10aaGcc22Xzag/ruAMsa53pVQWN7FLK88i5vGqi3ttnnpbnzf2kfsq92Bk8m*SF1SDzW32OSwjhtq4Kj2.suDHhTwDRwZQx*TpSRlNA*YV6yl9idcArxp.uZQ!/b&bo=pAfeAwAAAAADB1w!&rf=viewer_4)  

## 一、概述  

在 Kotlin 中有两种条件控制，分别是：  

1. if；
2. when；

接下来将对二者分别进行描述。  


## 二、详述  

### 1. if 条件控制  

Kotlin 中的 if 条件控制既可以被当作语句来处理，也可以被当作表达式来处理。  

当 Kotlin 中的 if 条件控制被当作语句来处理时，它的作用就和 Java 中的 if 条件控制语句作用一样；当 Kotlin 中的 if 条件控制被当作表达式来处理时，可以把表达式的结果赋值给一个变量。  

#### 1.1 if 条件控制作为语句  

```kotlin
// if 条件控制作为语句
var number: Int = 24
if (number > 18) {
    println("$number is bigger than 18")
} else if (number == 18) {
    println("$number is equals to 18")
} else {
    println("$number is smaller than 18")
}

// 执行结果
24 is bigger than 18
```

#### 1.2 if 条件控制作为表达式  

```kotlin
// if 条件控制作为表达式
var cupHeight: Int = 18
var bottleHeight: Int = 32
var smallerHeight = if (cupHeight < bottleHeight) cupHeight else bottleHeight
println("smallerHeight's value is $smallerHeight")

// 执行结果
smallerHeight's value is 18
```

由于 if 条件控制作为表达式之后，其作用和 Java 中的三目运算符作用相同，所以，Kotlin 中没有三目运算符：  

```kotlin
// 在 Kotlin 中，如果你这样写，编译器会报错
var anotherSmallerHeight = cupHeight < bottleHeight ? cupHeight : bottleHeight
```

#### 1.3 if 条件控制配合 in 运算符用来做区间检测  

```kotlin
var circleRadius: Int = 24
                 👇
if (circleRadius in 18..24) {
    println("$circleRadius between 18 and 24")
}

// 执行结果
24 between 18 and 24
```


### 2. when 条件控制  

Kotlin 中的 When 条件控制类似于 Java 中的 Switch 条件控制语句——都是将它的参数和所有的分支条件顺序比较，直到某个分支满足条件。只不过这里的 When 条件控制既可以被当作语句来处理，也可以被当作表达式来处理。  

当 Kotlin 中的 When 条件控制被当作语句来处理时，它的作用就和 Java 中的 Switch 条件控制语句作用一样；当 Kotlin 中的 When 条件控制被当作表达式来处理时，可以把表达式的结果赋值给一个变量。  

#### 2.1 when 条件控制作为语句  

```kotlin
// when 条件控制作为语句
var number: Int = 5
when (number) {
    0 -> println("number is 0")
    1 -> println("number is 1")
    else -> println("number is not 0 or 1")
}

// 执行结果
number is not 0 or 1
```

如果多个分支需要用相同的处理逻辑，则可以把多个分支条件放在一起，用逗号分隔：  

```kotlin
// 如果多个分支需要用相同的处理逻辑，则可以把多个分支条件放在一起，用逗号分隔：
var number: Int = 5
when (number) {
    0, 1 -> println("number is 0 or 1")
    else -> println("number is not 0 or 1")
}

// 执行结果
number is not 0 or 1
```

#### 2.2 when 条件控制作为表达式  

```kotlin
// when 条件控制作为表达式
var anotherNumber = when (number) {
    0, 1 -> 1000
    else -> 1000_000
}
println("another number's value is $anotherNumber")

// 执行结果
another number's value is 1000000
```

#### 2.3 when 条件控制配合 in 运算符用来做区间检测  

```kotlin
// when 条件控制配合 in 运算符用来做区间检测
var anotherNumber: Int = 1000_000
var intArray: IntArray = intArrayOf(0, 1000, 1000_000)
when (anotherNumber) {
    in 0..1000 -> println("$anotherNumber between 0 and 1000")
    in intArray -> println("$anotherNumber is in the array intArray")
    else -> println("none of the above")
}

// 执行结果
1000000 is in the array intArray
```

#### 2.4 when 条件控制配合 is 运算符用来做类型判断  

```kotlin
// when 条件控制配合 is 运算符用来做类型判断
var any: Any = 123
when (any) {
    is Byte -> println("any's type is Byte")
    is Int -> println("any's type is Int")
    is String -> println("any's type is Int")
    else -> println("none of the above")
}

// 执行结果
any's type is Int
```

#### 2.5 when 条件控制替换 if 条件控制  

```kotlin
// when 条件控制替换 if 条件控制
var number: Int = 5
when {
    number < 0 -> println("number is smaller than 0")
    number == 0 -> println("number is equals to 0")
    else -> println("number is bigger than 0")
}

// 执行结果
number is bigger than 0
```

```kotlin
// 另一个例子，"android" in stringList 等价于 stringList.contains("android")
var stringList: List<String> = listOf("how", "are", "you", "?")
when {
    "android" in stringList -> println("「android」 is in the string list")
    "how" in stringList -> println("「how」is in the string list")
    "are" in stringList -> println("「are」is in the string list")
    "you" in stringList -> println("「you」is in the string list")
    "?" in stringList -> println("「?」is in the string list")
    else -> println("none of the above")
}

// 执行结果
「how」is in the string list
```
