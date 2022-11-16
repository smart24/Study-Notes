![Kotlin 基本数据类型分类](http://m.qpic.cn/psc?/V10aaGcc22Xzag/ruAMsa53pVQWN7FLK88i5pJ7beSaYpWMrIPtxgy0X5RpBZ0eN7ccxfSldsWNLV0kXun9WIdYQKZKPLwTLxGkacBfNX*9YR5x9NQctBSfyco!/b&bo=VAY4BAAAAAADB0w!&rf=viewer_4)

## 一、概述  

Kotlin 的基本数据类型可分为五类，分别是：  
1. 数值型
    - Byte
    - Short
    - Int
    - Long
    - Float
    - Double
2. 字符型
3. 布尔型
4. 字符串
5. 数组


## 二、各数据类型详解  

### 1. 数值型  

#### 1.1 定义数值型变量  

数值型变量的定义很简单，基本语法如下：  

```
var variableName digitType = digitValue

或

val variableName digitType = digitValue

///////////////////////////////////////////
//                DIVIDER                //
///////////////////////////////////////////

示例：

var byteValue: Byte = 123

var shortValue: Short = 123

var intValue: Int = 123

// Long 数值类型末尾加不加 L 均可（如果要加 L 的话，这里只能加大写的 L，小写的 l 会报错）
var longValue: Long = 123L
var anotherLongValue: Long = 123

// Float 数值类型末尾要加 f 或 F，否则会报错
var floatValue: Float = 123f
var anotherFloatValue: Float = 123F

// Double 数值类型末尾要有小数点，否则会报错
var doubleValue: Double = 123.0
```

众所周知，数字的表示方式除了十进制（上面用到的就是十进制），还有二进制、八进制、十六进制，那 Kotlin 支不支持这些呢？答案是肯定的。Kotlin 除了不支持八进制以外，二进制、十进制和十六进制都是支持的。那 Kotlin 如何定义二进制、十六进制数据呢？  

```
// 二进制数据以 0b 或 0B 开头
var binaryValue: Int = 0b111
var anotherBinaryValue: Int = 0B111

// 默认即为十进制数据，所以，十进制数据什么也不加
var decimalValue: Int = 123

// 十六进制数据 0x 或 0X 开头
var hexadecimalValue: Int = 0x123
var anotherHexadecimalValue: Int = 0X123
```

另外，有时为了使数值数据更易读，还可以以下面定义数值数据变量值：  

```
// 等价于 val tenThousand = 10000
val tenThousand = 10_000

// 等价于 val oneMillion = 1000000
val oneMillion = 1000_000

// 等价于 val oneBillion = 10000000000
val oneBillion = 10_000_000_000

// 等价于 val cardNumber = 1234567890123456L
val cardNumber = 1234_5678_9012_3456L

// 等价于 val earthRadius = 6371393
val earthRadius = 6.371393e6
val anotherEarthRadius = 6.371393E6
```

通过上面的内容，不难得知：Kotlin 中是没有基础数据类型的，只有封装的数值类型。也就是说，你每定义一个数值型变量，Kotlin 都会将其封装成一个对象（这样做，可以保证不会出现空指针）。  


#### 1.2 比较大小  

虽然 Kotlin 中的数值类型都是对象，但这丝毫不影响它们的使用方式——数值型变量定义完之后，直接使用变量进行大小比较即可：  

```
// 相同数值类型
            👇
var width: Int = 720
             👇
var height: Int = 1280
if (width > height) {
    println("width $width is bigger than height $height")
} else if (width < height) {
    println("width $width is smaller than height $height")
} else {
    println("width $width is equals to height $height")
}

// 不同数值类型
                     👇
var buildingHeight: Int = 123
                     👇
var bookThickness: Float = 0.01f
if (buildingHeight > bookThickness) {
    println("buildingHeight $buildingHeight is bigger than bookThickness $bookThickness")
} else if (buildingHeight < bookThickness) {
    println("buildingHeight $buildingHeight is smaller than bookThickness $bookThickness")
} else {
    println("buildingHeight $buildingHeight is equals to bookThickness $bookThickness")
}

///////////////////////////////////////////
//                DIVIDER                //
///////////////////////////////////////////

// 执行结果
width 720 is smaller than height 1280
buildingHeight 123 is bigger than bookThickness 0.01
```

既然可以像上面这样比较数值大小，那数值相等性比较的时候，肯定用的是两个等号「 == 」。如果数值相等性比较的时候用的是两个等号「 == 」，那判断对象地址相等性的时候用什么呢？  

在 Kotlin 中，用两个等号「 == 」来判断数值相等性，用三个等号「 === 」来判断对象地址相等性：  

```
var length: Int = 128
var stringLength: Int? = length
var anotherStringLength: Int? = length
// 比较数值相等性
println("variable length's value is equals to stringLength's value ${length == stringLength}")
// 比较对象地址相等性
println("variable length's address is equals to anotherStringLength's address ${stringLength === anotherStringLength}")

// 执行结果
variable length's value is equals to stringLength's value true
variable length's address is equals to anotherStringLength's address false
```


#### 1.3 类型转换  

由于所有的数值类型都是对象，所以，数据范围较小的数值类型并不是数据范围大的数值类型的子类型，也就说在 Kotlin 里没有 Java 里面的基本数值类型的隐式类型转换——在 Java 基本数值类型中，数据范围小数值类型的可以隐式转换为数据范围大的数值类型：  

```
// Java
// 此时将数据范围较小的数值类型 byteValue 赋值给数据范围较大的数值类型 intValue，无需强制转换，转换会自动完成，
// 把 intValue 赋值给 doubleValue 也一样
// 数据范围较小的数值类型
byte byteValue = 123;

// 数据范围较大的数值类型
int intValue = byteValue;

// 数据范围较大的数值类型
double doubleValue = intValue;
```

在 Kotlin 中，这意味着在不进行显式转换的情况下，我们是不能把 Byte 数值类型的值赋给一个 Int 数值类型的变量的：  

```
var screenHeight: Int = 1920
// 报错：Type mismatch.
var treeHeight: Double = screenHeight
```

处理办法也很简单，只需进行强制类型转换即可：

```
var screenHeight: Int = 1920
                                         👇
var treeHeight: Double = screenHeight.toDouble()
```

这六种数值型数据都支持如下强制类型转换操作：  


| 序号 | 方法 | 含义 |
| --- | --- | --- |
| 01 | toByte(): Byte | 转 Byte |
| 02 | toShort(): Short | 转 Short |
| 03 | toInt(): Int | 转 Int |
| 04 | toLong(): Long | 转 Long |
| 05 | toFloat(): Float | 转 Float |
| 06 | toDouble(): Double | 转 Double |
| 07 | toChar(): Char | 转 Char |

```
var longValue: Long = 123L
var anotherByteValue: Byte = longValue.toByte()
var anotherShortValue: Short = longValue.toShort()
var anotherIntValue: Int = longValue.toInt()
var anotherLongValue: Long = longValue.toLong()
var anotherFloatValue: Float = longValue.toFloat()
var anotherDoubleValue: Double = longValue.toDouble()
var anotherCharValue: Char = longValue.toChar()
```


#### 1.4 位操作符  

对于 Int 和 Long 数值类型，还有一系列的位操作符可以使用，分别是：

```
shl(bits)   // 左移（SHIFT LEFT），相当于 Java 中的 <<
shr(bits)   // 右移（SHIFT RIGHT），相当于 Java 中的 >>
ushr(bits)  // 无符号右移（UNSIGNED SHIFT RIGHT），相当于 Java 中的 >>>
and(bits)   // 与（AND），相当于 Java 中的 &
or(bits)    // 或（OR），相当于 Java 中的 |
xor(bits)   // 异或（XOR），相当于 Java 中的 ^
inv()       // 取反（INVERT），相当于 Java 中 ～

///////////////////////////////////////////
//                DIVIDER                //
///////////////////////////////////////////

示例：

var one: Int = 1
println("after shift left one bit the number $one's value is ${one.shl(1)}")
println("after shift right one bit the number $one's value is ${one.shr(1)}")
println("after unsigned shift right one bit the number $one's value is ${one.ushr(1)}")
println("after \"and\" with 0 the number $one's value is ${one.and(0)}")
println("after \"or\" with 0 the number $one's value is ${one.or(0)}")
println("after \"xor\" with 0 the number $one's value is ${one.xor(0)}")
println("after \"inv\" the number $one's value is ${one.inv()}")

// 执行结果
after shift left one bit the number 1's value is 2
after shift right one bit the number 1's value is 0
after unsigned shift right one bit the number 1's value is 0
after "and" with 0 the number 1's value is 0
after "or" with 0 the number 1's value is 1
after "xor" with 0 the number 1's value is 1
after "inv" the number 1's value is -2
```


#### 2. 字符型  

#### 2.1 定义字符型变量  

在 Kotlin 中，字符型变量用关键字 Char 表示。不过，Kotlin 中的字符型变量和 Java 中的字符型变量不太一样，具体表现为以下两点：  

- Char 不能直接和数字操作；
- Char 必须用单引号「'」包起来；

```
// 在 Kotlin 中，这样定义 Char 会报错
//    var charValue: Char = 1
// 在 Kotlin 中，只有这样定义 Char 才可以
var charValue: Char = '1'
```

#### 2.2 转义字符  

同 Java 一样，Kotlin 也支持转义字符，Kotlin 支持的转义字符有：  

| 序号 | 字符 | 含义 |
| --- | --- | --- |
| 1 | \t | 水平制表符 |
| 2 | \b | 退格（删除「\b」前面的一个字符） |
| 3 | \n | 换行（Line Feed），将光标移动到下一行起始位置 |
| 4 | \r | 回车（Carriage Return），将光标移动到本行起始位置，「\r」之前内容会被清除 |
| 5 | \\' | 打印单引号 |
| 6 | \\" | 打印双引号 |
| 7 | \\ | 打印单斜杠 |
| 8 | \\\\ | 打印双斜杠 |
| 9 | \\$ | 打印 $ |

```
println("origin: abcd")
print('\t')
print("origin: abcd")

// 执行结果
origin: abcd
	origin: abcd
```


### 3. 布尔型  

在 Kotlin 中，布尔型变量用关键字 Boolean 表示，它有两个值，分别是：true 和 false。

```
var booleanValue: Boolean = false
```


### 4. 字符串  

在 Kotlin 中，字符串变量也用 String 关键字表示。  

Kotlin 中的 String 跟 Java 中的 String 不同之处在于：  

1. Kotlin 中的 String 支持双引号「"xxx"」和三引号「"""xxx"""」，其中三引号支持多行字符串；
2. Kotlin 中的 String 可以通过方括号「[]」语法可以很方便的获取字符串中的某个字符；
3. Kotlin 中的 String 可以通过 for 循环遍历；
4. Kotlin 中的 String 支持模版表达式
     - 即一些小段代码，会求值并把结果合并到字符串中。模板表达式以「$」开头，由一个简单的名字或者用花括号扩起来的任意表达式构成；

#### 4.1 定义字符串  

```
// Kotlin 中的 String 支持双引号「"xxx"」和三引号「"""xxx"""」，其中三引号支持多行字符串
// 单行字符串
var singleLine: String = "How Are You?"
// 多行字符串
var multiLines: String = """
    The longest journey begins with the first step.
    千里之行，始于足下
""".trimIndent()
println(singleLine)
println(multiLines)

// 执行结果
How Are You?
The longest journey begins with the first step.
千里之行，始于足下
```

#### 4.2. 通过方括号「[]」语法获取字符串中的某个字符  

```
// Kotlin 中的 String 可以通过方括号「[]」语法可以很方便的获取字符串中的某个字符
var number: String = "0123456789"
println(number[5])

// 执行结果
5
```

#### 4.3 通过 for 循环遍历字符串中的字符  

```
// Kotlin 中的 String 可以通过 for 循环遍历
var number: String = "0123456789"
for (e in number) {
    print("$e, ")
}

// 执行结果
0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 
```

#### 4.4 模版表达式  

```
// Kotlin 中的 String 支持模版表达式
var name: String = "smart 24"
// 一个简单的名字构成的模版表达式
var selfIntroduction: String = "Hello, everyone! My name is $name"
println(selfIntroduction)

// 用花括号扩起来的任意表达式
var calculateNameLength: String = "\"$name\".length is ${name.length}"
println(calculateNameLength)

// 执行结果
Hello, everyone! My name is smart 24
"smart 24".length is 8
```

Kotlin 中的模版表达式的功能类似于 Java 中的 String.format() 函数的功能，上面的示例代码等价于 Java 中的：  

```
String name = "smart 24";
String selfIntroduction = String.format("Hello, everyone! My name is %s", name);
System.out.println(selfIntroduction);
String calculateNameLength = String.format("\"%s\".length is %d", name, name.length());
System.out.println(calculateNameLength);

// 执行结果
Hello, everyone! My name is smart 24
"smart 24".length is 8
```


### 5. 数组  

在 Kotlin 中，数组用 Array 关键字表示。  

#### 5.1 定义数组  

数组的创建方式有两种，分别是：  

1. 使用函数 arrayOf()；
2. 使用工厂函数；

**1. 使用函数 arrayOf() 创建数组**  

```
var array: Array<Int> = arrayOf(1, 2, 3)
```

**2. 使用工厂函数创建数组**  

```
var anotherArray: Array<Int> = Array(3) { i -> (i + 1) }
```

#### 5.2 获取数组长度及访问数组中的元素  

与 Java 数组不同的是：Kotlin 中的数组，没有了 length 属性，取而代之的是 size 属性。另外，Kotlin 中的数组还多了 set 和 get 方法。另另外，由于「[]」重载了 get 和 set 方法，所以我们可以通过下标很方便地设置或获取数组对应索引位置的值。  

```
// 获取数组尺寸
println("Array array's size is ${array.size}")

// 设置指定索引位置的值
println(array[0])
array.set(0, 999) // 等价于 array[0] = 999
println(array[0])

// 获取指定索引位置的值
println(array.get(0)) // 等价于 println(array.get(0))

// 执行结果
Array array's size is 3
1
999
999
```

#### 5.3 除了 Array 之外，还有哪些数组？  

除了上面提到的 Array 类之外，Kotlin 还提供了 ByteArray、ShortArray、IntArray、LongArray、FloatArray、DoubleArray、BooleanArray、CharArray，这些数组的用法和 Array 一样，只不过省去了装箱操作，因此效率更高：  

```
var byteArray: ByteArray = byteArrayOf(1, 2, 3)
var shortArray: ShortArray = shortArrayOf(1, 2, 3)
var intArray: IntArray = intArrayOf(1, 2, 3)
var longArray: LongArray = longArrayOf(1, 2, 3)
var floatArray: FloatArray = floatArrayOf(1f, 2f, 3f)
var doubleArray: DoubleArray = doubleArrayOf(1.0, 2.0, 3.0)
var booleanArray: BooleanArray = booleanArrayOf(true, true, true)
var charArray: CharArray = charArrayOf('1', '2', '3')
```