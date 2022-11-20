![Kotlin 循环控制](http://m.qpic.cn/psc?/V10aaGcc22Xzag/ruAMsa53pVQWN7FLK88i5q7AiNtMBDWgGLrDm5DI.YwG8Gw3qf4ni2tAXuZh0wUJTDsLZ0vid3bzh1FvqOtss2Wm0fvpimefMs6RJ4b5Yi0!/b&bo=zAfcAwAAAAADBzY!&rf=viewer_4)  

## 一、概述

Kotlin 中的循环控制包括三种，分别是：  

1. for 循环；
2. while 循环；
3. do...while 循环；


## 二、详述  

### 1. for 循环  

Kotlin 中没有类似 Java 中的 for 循环，Kotlin 中的 for 循环和 Java 中的 for-each 很像，都是直接遍历数组、集合中的元素值。  

```java
// 这是 Java 中的 for 循环，Kotlin 中没有这样的 for 循环
int[] intArray = new int[]{1, 2, 3};
for (int i = 0; i < intArray.length; i++) {
    System.out.println(i);
}
```

```java
// Java 中的 for-each
int[] intArray = new int[]{1, 2, 3};
for (int i : intArray) {
    System.out.println(i);
}
```

```kotlin
// Kotlin 中的 for 循环
var intArray: IntArray = intArrayOf(1, 2, 3)
for (i in intArray) {
    println(i)
}
```

Kotlin 中的 for 循环是不是和 Java 中的 for-each 很像？  

顺便说一句，如果你对 Kotlin 中的数组不是很了解，可以看这里 👉 [Kotlin 基本数据类型分类](https://github.com/smart24/Study-Notes/blob/main/05_Android/01_Kotlin/01_Kotlin%20%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E5%88%86%E7%B1%BB/Kotlin%20%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E5%88%86%E7%B1%BB.md)。  

#### 1.1 正常遍历  

```
// 正常遍历（遍历的是数组中的元素值，即：元素值 1，2，3）
var intArray: IntArray = intArrayOf(1, 2, 3)
for (i in intArray) {
    println(i)
}

// 执行结果
1
2
3
```

#### 1.2 按元素索引值遍历  

除了直接遍历数组、集合中的元素值，Kotlin 中的 for 循环还支持按元素索引值遍历：  

```kotlin
// 按元素索引值遍历
var intArray: IntArray = intArrayOf(1, 2, 3)
for (i in intArray.indices) {
    println("[indices] the element at index $i's value is ${intArray[i]}")
}

// 执行结果
[indices] the element at index 0's value is 1
[indices] the element at index 1's value is 2
[indices] the element at index 2's value is 3
```

#### 1.3 用库函数「withIndex()」同时获取元素索引值和元素值  

```kotlin
// 用库函数「withIndex()」同时获取元素索引值和元素值
                                    👇
for ((index, value) in intArray.withIndex()) {
    println("[withIndex()] the element at index $index's value is $value")
}

// 执行结果
[withIndex()] the element at index 0's value is 1
[withIndex()] the element at index 1's value is 2
[withIndex()] the element at index 2's value is 3
```

#### 1.4 遍历指定区间范围内的元素  

由于 Kotlin 中没有 Java 中的 for 循环，所以，它提供了「区间」的概念，用于遍历指定区间内的元素。  

区间的意思是：两个值之间的间隔。这两个值通常是数字（还可以是其他类型，比如字符型），一个是起点，一个是终点。  

```kotlin
var scope = 1..4
```

需要注意的是：此种遍历方式，起点的元素值必须小于等于终点元素值，即 startPoint <= i <= endPoint，我且称之为「正序遍历」。  

```kotlin
// 遍历指定区间范围内的元素
var startPoint: Int = 4
var endPoint: Int = 6
for (i in startPoint..endPoint) {
    println("[..] the element at index ${i - startPoint}'s value is $i")
}

// 执行结果
[..] the element at index 0's value is 4
[..] the element at index 1's value is 5
[..] the element at index 2's value is 6
```

除了上面的遍历区间内所有元素之外，Kotlin 还支持设置步长，也就是按指定步长遍历指定区间范围内的元素：  

```kotlin
// 按指定步长遍历指定区间范围内的元素
var startPoint = 2
var endPoint = 6
var stepValue: Int = 2
for (i in startPoint..endPoint step stepValue) {
    println("[.. step] the element's value is $i")
}

// 执行结果
[.. step] the element's value is 2
[.. step] the element's value is 4
[.. step] the element's value is 6
```

上面的「遍历指定区间范围内的元素」在遍历元素的时候，会遍历起点和终点。如果遍历的时候，不想遍历终点，可以使用 util 关键字：  

```kotlin
// 使用 util 关键字过滤掉终点
var startPoint = 4
var endPoint = 6
for (i in startPoint until endPoint) {
    println("[util] the element's value is $i")
}

// 执行结果
[util] the element's value is 4
[util] the element's value is 5
```

#### 1.5 「逆序遍历」指定区间范围内的元素  

除了上面提到的「正序遍历」，for 循环还支持「逆序遍历」，「逆序遍历」需使用 downTo 关键字。  

类似于「正序遍历」的规则，「逆序遍历」的起点元素值必须大于等于终点元素值，即 startPoint >= i >= endPoint。  

```kotlin
// 「逆序遍历」指定区间范围内的元素
var startPoint = 6
var endPoint = 4
for (i in startPoint downTo endPoint) {
    println("[downTo] the element's value is $i")
}

// 执行结果
[downTo] the element's value is 6
[downTo] the element's value is 5
[downTo] the element's value is 4
```

「逆序遍历」也支持设置步长：  

```kotlin
// 「逆序遍历」也支持设置步长
var startPoint = 6
var endPoint = 2
var stepValue = 2
for (i in startPoint downTo endPoint step stepValue) {
    println("[downTo step] the element's value is $i")
}

// 执行结果
[downTo step] the element's value is 6
[downTo step] the element's value is 4
[downTo step] the element's value is 2
```

#### 1.6 循环体只有一行代码时，可以省略大括号  

当 for 循环的循环体只有一行代码时，可以省略大括号：  

```kotlin
// 循环体只有一行代码时，可以省略大括号
for (i in intArray) println("[single line] the element value is $i")

// 执行结果
[single line] the element value is 1
[single line] the element value is 2
[single line] the element value is 3
```


### 2. while 循环  

Kotlin 中的 while 循环与 Java 中的 while 循环没有什么区别，所以，这里就不再赘述了：  

```kotlin
// Kotlin while 循环
var number: Int = 5
while (number > 0) {
    println("[while] number current value is $number")
    number--
}

// 执行结果
[while] number current value is 5
[while] number current value is 4
[while] number current value is 3
[while] number current value is 2
[while] number current value is 1
```


### 3. do...while 循环  

Kotlin 中的 do...while 循环与 Java 中的 do...while 循环没有什么区别，所以，这里也再不赘述了：  

```kotlin
// Kotlin do...while 循环
var number: Int = 0
do {
    println("[do...while] number current value is $number")
    number++
} while (number < 5)

// 执行结果
[do...while] number current value is 0
[do...while] number current value is 1
[do...while] number current value is 2
[do...while] number current value is 3
[do...while] number current value is 4
```

最后，需要注意的是：Kotlin 中的循环控制同样也支持 continue、break、return，其用法和 Java 中的 continue、break、return 一样：  

```kotlin
// continue、break、return
for (i in 0..9) {
    if (i == 3) continue
    else if (i > 6) break
    println("[continue、break、return] current value is $i")
}
```
