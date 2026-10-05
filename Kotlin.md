# Kotlin

## 目录

**基础语法**

- [一、Kotlin 概述与运行机制](#一kotlin-概述与运行机制)
- [二、变量、常量与基本数据类型](#二变量常量与基本数据类型)
- [三、空安全（Null Safety）](#三空安全null-safety)
- [四、类型系统（Any、Unit、Nothing 与类型转换）](#四类型系统anyunitnothing-与类型转换)
- [五、运算符与流程控制（if、when、for、区间）](#五运算符与流程控制ifwhenfor区间)
- [六、字符串与字符串模板](#六字符串与字符串模板)
- [七、数组与集合基础（只读与可变）](#七数组与集合基础只读与可变)

**函数与函数式编程**

- [八、函数（默认参数、具名参数、可变参数）](#八函数默认参数具名参数可变参数)
- [九、Lambda 与高阶函数](#九lambda-与高阶函数)
- [十、内联函数（inline、noinline、crossinline、reified）](#十内联函数inlinenoinlinecrossinlinereified)
- [十一、扩展函数与扩展属性](#十一扩展函数与扩展属性)
- [十二、作用域函数（let、run、with、apply、also）](#十二作用域函数letrunwithapplyalso)
- [十三、集合操作 API 与序列（Sequence）](#十三集合操作-api-与序列sequence)

**面向对象**

- [十四、类与构造函数（主构造器、次构造器、init）](#十四类与构造函数主构造器次构造器init)
- [十五、属性、幕后字段与延迟初始化（lateinit、by lazy）](#十五属性幕后字段与延迟初始化lateinitby-lazy)
- [十六、继承、接口与抽象类](#十六继承接口与抽象类)
- [十七、数据类、密封类与枚举类](#十七数据类密封类与枚举类)
- [十八、对象声明、伴生对象与单例](#十八对象声明伴生对象与单例)
- [十九、可见性修饰符与嵌套类、内部类](#十九可见性修饰符与嵌套类内部类)
- [二十、委托（类委托与属性委托）](#二十委托类委托与属性委托)
- [二十一、泛型与型变（in、out、where）](#二十一泛型与型变inoutwhere)

**协程与并发**

- [二十二、协程基础（挂起函数、CoroutineScope、Dispatchers）](#二十二协程基础挂起函数coroutinescopedispatchers)
- [二十三、结构化并发（Job、取消与异常处理）](#二十三结构化并发job取消与异常处理)
- [二十四、Flow 与冷热流（StateFlow、SharedFlow）](#二十四flow-与冷热流stateflowsharedflow)
- [二十五、Channel 与并发安全（Mutex、原子类）](#二十五channel-与并发安全mutex原子类)

**工程实践与原理**

- [二十六、Kotlin 与 Java 互操作](#二十六kotlin-与-java-互操作)
- [二十七、反射、注解与 DSL 构建](#二十七反射注解与-dsl-构建)
- [二十八、编译原理与字节码剖析](#二十八编译原理与字节码剖析)
- [二十九、工程实践与生态（Gradle KTS、序列化、单元测试）](#二十九工程实践与生态gradle-kts序列化单元测试)

---

## 一、Kotlin 概述与运行机制

### 1. Kotlin 是什么

由 **JetBrains**（IntelliJ IDEA 的开发商）推出的静态类型编程语言，2016 年发布 1.0，目前主流版本为 **2.x**（编译器已切换到新一代 **K2**）。

一句话定位：**跑在 JVM 上、与 Java 100% 互操作、更简洁更安全的现代语言**。

| 时间 | 事件 |
| ---- | ---- |
| 2016 | Kotlin 1.0 发布 |
| 2017 | Google I/O 宣布 Android **官方支持** Kotlin |
| 2019 | Google 宣布 Android 开发 **Kotlin 优先（Kotlin-first）** |
| 2024 | Kotlin 2.0 发布，默认启用 K2 编译器，编译速度大幅提升 |

---

### 2. 核心特性（面试开场必问）

| 特性 | 说明 |
| ---- | ---- |
| ⭐**空安全** | 类型系统区分 `String` 与 `String?`，把 NPE 从运行时提前到**编译期** |
| ⭐**简洁** | 类型推断、数据类、字符串模板，同样功能代码量比 Java 少 30%~40% |
| ⭐**协程** | 语言级并发方案，用同步的写法做异步的事，替代回调地狱 |
| **完全互操作** | 可与 Java 相互调用、同一模块混合编译，老项目可渐进迁移 |
| **函数式支持** | 函数是一等公民，高阶函数、Lambda、不可变集合 |
| **扩展函数** | 不改源码、不继承，直接给已有类"加方法" |
| **多平台** | 一套代码编译到 JVM / Android / iOS / JS / Native |

> Kotlin 不是"Java 的语法糖"，它有自己的类型系统（可空类型、`Nothing`、型变声明）和运行时库 `kotlin-stdlib`。

---

### 3. 编译与运行机制 ⭐

Kotlin 是**多后端**语言，同一份源码可以编译到不同目标平台：

```text
                    ┌─ Kotlin/JVM    → .class 字节码 → JVM 运行（最常用：后端、Android）
   .kt 源码 ──────── ├─ Kotlin/Native → 机器码（LLVM）→ 直接运行（iOS、桌面）
   （K2 编译器）      ├─ Kotlin/JS     → JavaScript   → 浏览器 / Node.js
                    └─ Kotlin/Wasm   → WebAssembly
```

**最常用的 Kotlin/JVM 流程**：

```text
Hello.kt ──kotlinc──> HelloKt.class ──> JVM 执行（运行时需要 kotlin-stdlib.jar）
```

三个关键结论：

1. **编译产物就是普通的 `.class` 字节码**，JVM 根本不知道它是 Kotlin 写的，因此能和 Java 无缝混用。
2. **运行时必须带上 `kotlin-stdlib`**，因为 `let`、`Intrinsics` 空检查、集合扩展等都在标准库里（Gradle 引入 Kotlin 插件后会自动依赖）。
3. **大量工作在编译期完成**：类型推断、空检查插桩、`inline` 函数展开、`data class` 自动生成方法 —— 所以 Kotlin 的语法糖基本**没有运行时开销**。

---

### 4. 第一个程序与文件结构

```kotlin
package com.example.demo          // 包声明，可与目录结构不一致（但规范要求一致）

import kotlin.math.max            // 导入

fun main() {                      // ⭐ 顶层函数，不需要写在类里；参数 args 可省略
    val name = "Kotlin"           // 类型推断为 String，不用写类型
    println("Hello, $name!")      // 字符串模板，不用拼接
}
```

对比 Java 的 `public class Main { public static void main(String[] args) }`，差异点：

| 项 | Kotlin |
| ---- | ---- |
| 语句结尾分号 | **可省略**（规范：不写） |
| 函数位置 | 可直接写在文件顶层，无需依附类 |
| 文件名与类名 | **无需一致**，一个文件可放多个类和函数 |
| 类默认修饰符 | `public final`（默认公开且不可继承） |
| 打印 | `println()`（`System.out.println` 的标准库封装） |

> **顶层函数怎么运行的？** 编译器会为 `Hello.kt` 生成一个 `HelloKt` 类，把顶层函数编译成它的 `static` 方法。所以 Java 侧调用要写 `HelloKt.main()`。

---

### 5. Kotlin vs Java 关键差异速览

| 对比项 | Java | Kotlin |
| ---- | ---- | ---- |
| 变量声明 | `String s = "a";` | `val s = "a"`（只读）/ `var`（可变） |
| 空安全 | 无，运行时 NPE | ⭐类型级支持：`String` / `String?` |
| 基本类型 | `int` 与 `Integer` 区分 | 统一写 `Int`，编译期自动选择装箱与否 |
| 静态成员 | `static` | 无 `static`，用**顶层声明**或**伴生对象** |
| 受检异常 | 必须 `throws` / `catch` | **无受检异常**，不强制处理 |
| 三元表达式 | `a > b ? a : b` | 无三元，用 `if (a > b) a else b`（if 是表达式） |
| switch | `switch` | ⭐`when`，更强大且可返回值 |
| 数据模型 | 手写 getter/setter/equals | ⭐`data class` 一行搞定 |
| 异步 | 线程、线程池、回调 | ⭐协程 `suspend` |
| 工具方法 | `XxxUtils` 静态类 | ⭐扩展函数 |

---

### 6. 行业使用现状

| 领域 | 使用情况 |
| ---- | ---- |
| ⭐**Android** | 绝对主流。Google 官方首选，Jetpack、Compose 全部 Kotlin 优先，新项目基本不再用 Java |
| **服务端** | Spring Boot 官方支持 Kotlin；Ktor（JetBrains 自研，轻量协程框架）在国内外都有增长 |
| **构建脚本** | Gradle KTS（`build.gradle.kts`）逐步取代 Groovy，有类型提示和补全 |
| **跨平台（KMP）** | 共享业务逻辑到 iOS/Android/桌面，Compose Multiplatform 生态成型中 |
| **大数据/其他** | 少量，该领域仍以 Java/Scala 为主 |

> **求职提示**：国内招聘中 Kotlin 需求集中在 **Android 岗**（几乎必备），服务端岗多为"Java 为主、Kotlin 加分"。面试考察重点固定在三块：**空安全、协程、与 Java 的差异及互操作**。

---

### 7. 面试重点

**Q1：Kotlin 相比 Java 的优势是什么？**

四点足够：① **空安全**，编译期消灭大部分 NPE；② **简洁**，数据类、类型推断、扩展函数大幅减少样板代码；③ **协程**，语言级异步方案，比线程池 + 回调更轻量易读；④ **完全兼容 Java**，可与现有项目无缝混用、渐进迁移。

**Q2：Kotlin 运行需要虚拟机吗？跟 Java 什么关系？**

Kotlin/JVM 编译后就是标准 `.class` 字节码，运行在 JVM 上，因此能直接调用所有 Java 类库。但 Kotlin 不止于 JVM，还可以通过 Kotlin/Native 编译成机器码、Kotlin/JS 编译成 JavaScript。另外运行时需要额外依赖 `kotlin-stdlib`。

**Q3：Kotlin 的空安全在字节码层面是怎么实现的？**

编译期做两件事：一是**类型检查**，可空类型不允许直接解引用；二是**插桩**，在 public 函数入口插入 `Intrinsics.checkNotNullParameter()` 之类的校验，防止 Java 侧传入 `null` 绕过检查。所以 Kotlin 的空安全主要是编译期能力，运行时只留少量断言。

**Q4：Kotlin 为什么没有 static？**

Kotlin 用**顶层声明**（函数/属性直接写在文件里，编译成 `FileNameKt` 的静态成员）和**伴生对象 `companion object`** 来替代。这样静态成员也是真正的对象，可以实现接口、支持扩展函数，设计上更统一。需要 Java 侧以静态方式调用时，加 `@JvmStatic` 注解。

**Q5：Kotlin 有受检异常吗？**

没有。所有异常都是非受检的，不强制 `try-catch` 或 `throws`。好处是减少样板代码，代价是编译器不再提醒你处理异常，需要靠规范和 code review 保证。Kotlin 调 Java 的受检异常方法也不需要强制捕获。

---

## 二、变量、常量与基本数据类型

### 1. val 与 var ⭐

Kotlin 声明变量只有两个关键字，**类型写在名字后面**，且大多数时候可以省略：

```kotlin
var count: Int = 0        // 可变：能重新赋值
count = 10                // ✔

val name: String = "Tom"  // 只读：只能赋值一次
name = "Jerry"            // ✘ 编译错误：Val cannot be reassigned

val age = 20              // ⭐类型推断，等价于 val age: Int = 20
```

| | `var` | `val` |
| ---- | ---- | ---- |
| 含义 | variable，可变 | value，**只读** |
| 能否重新赋值 | 能 | 不能 |
| 对应 Java | 普通变量 | `final` 变量 |
| 使用建议 | 确实需要改才用 | ⭐**默认首选** |

> **关键理解：`val` 是"引用不可变"，不是"对象不可变"。**

```kotlin
val list = mutableListOf(1, 2)
list.add(3)          // ✔ 对象内容可以改
list = mutableListOf() // ✘ 引用不能改
```

---

### 2. 声明的三条规则

**① 变量使用前必须初始化**，但可以延后赋值：

```kotlin
val msg: String              // 只声明不赋值，必须显式写类型
if (flag) msg = "a" else msg = "b"
println(msg)                 // ✔ 编译器能确认所有分支都已赋值
```

**② 类型一旦确定不可改变**，`var` 也只能换值不能换类型：

```kotlin
var num = 10
num = "abc"        // ✘ 类型不匹配
```

**③ 类中的属性必须初始化**（否则用 `lateinit` 或 `by lazy`，见第十五章）：

```kotlin
class User {
    var name: String = ""    // ✔ 必须给初始值
    var age: Int = 0
}
```

---

### 3. const val：编译期常量

```kotlin
const val MAX_SIZE = 100              // ⭐编译期常量，直接内联到使用处
val currentTime = System.currentTimeMillis()   // 运行期才确定，不能加 const
```

| | `val` | `const val` |
| ---- | ---- | ---- |
| 赋值时机 | 运行期 | **编译期** |
| 声明位置 | 任意 | 只能在**顶层**、`object`、`companion object` 中 |
| 支持类型 | 任意 | 只能是基本类型和 `String` |
| 编译结果 | 字段 + getter | 常量值直接内联，无方法调用开销 |

```kotlin
// 企业写法：常量集中在 object 或顶层
object Constants {
    const val BASE_URL = "https://api.xxx.com"
    const val TIMEOUT = 30_000L
}
```

---

### 4. 基本数据类型

Kotlin **没有 Java 那种"基本类型 / 包装类"的区分**，一切皆对象，编译器自动决定用不用装箱：

| 分类 | 类型 | 位数 | 说明 |
| ---- | ---- | ---- | ---- |
| 整型 | `Byte` | 8 | -128 ~ 127 |
| | `Short` | 16 | |
| | ⭐`Int` | 32 | 整数字面量**默认类型**，约 ±21 亿 |
| | `Long` | 64 | 时间戳、雪花 ID 必用 |
| 浮点 | `Float` | 32 | 约 6~7 位有效数字 |
| | ⭐`Double` | 64 | 小数字面量**默认类型** |
| 字符 | `Char` | 16 | 单引号，**不是数字**，不能直接参与算术 |
| 布尔 | `Boolean` | — | 只有 `true` / `false` |

**字面量写法**：

```kotlin
val a = 100          // Int
val b = 100L         // Long，加 L
val c = 3.14         // Double
val d = 3.14F        // Float，加 F
val e = 0xFF         // 十六进制 255
val f = 0b1010       // 二进制 10（⚠ Kotlin 不支持八进制）
val g = 1_000_000    // ⭐下划线分隔，提升可读性
val h = 'A'          // Char
```

> **无符号类型**（`UInt`、`ULong` 等）用于处理协议、位运算等场景，业务开发几乎用不到，了解即可。

---

### 5. 类型转换：没有隐式转换 ⭐

Java 里 `int → long` 会自动扩宽，**Kotlin 一律不行**，必须显式调用 `toXxx()`：

```kotlin
val i: Int = 10
val l: Long = i          // ✘ 编译错误：Type mismatch
val l: Long = i.toLong()  // ✔

val d: Double = 3.99
val n: Int = d.toInt()    // 3 ⚠ 直接截断，不是四舍五入
val r = Math.round(3.99)  // 4，要四舍五入用 round
```

常用方法：`toByte()` `toShort()` `toInt()` `toLong()` `toFloat()` `toDouble()` `toChar()` `toString()`。

**两个例外**（编译期就能确定的字面量，可以直接写）：

```kotlin
val l: Long = 100        // ✔ 字面量在 Long 范围内，编译器直接按 Long 处理
val sum = 1L + 1         // ✔ 混合运算时较小类型自动提升
```

**两个仍然存在的坑**：

```kotlin
// ① 溢出不报错，静默出错
val x = Int.MAX_VALUE + 1        // -2147483648
val ms = 1000 * 60 * 60 * 24 * 30  // ✘ 全是 Int，中间就溢出了
val ms = 1000L * 60 * 60 * 24 * 30 // ✔ 第一个操作数写成 Long

// ② 浮点数精度丢失，金额必须用 BigDecimal
println(0.1 + 0.2)               // 0.30000000000000004
```

---

### 6. Char 与数字

`Char` 在 Kotlin 中是独立类型，**不能当整数用**：

```kotlin
val c: Char = 'A'
val n: Int = c           // ✘ 编译错误
val n: Int = c.code      // ✔ 65，取 Unicode 码点（1.5 起用 code，toInt() 已废弃）
val c2: Char = 66.toChar()   // 'B'

val next = 'a' + 1       // 'b'，Char + Int 结果是 Char
val diff = 'z' - 'a'     // 25，Char - Char 结果是 Int
val isDigit = c.isDigit()    // 标准库判断方法：isLetter / isDigit / isWhitespace
```

---

### 7. == 与 === ⭐（高频陷阱）

| 运算符 | 作用 | 等价于 Java |
| ---- | ---- | ---- |
| `==` | ⭐**比内容**，自动调用 `equals()`，且**空安全** | `a != null && a.equals(b)` |
| `===` | 比**引用地址** | `==` |

```kotlin
val s1 = "abc"
val s2 = "ab" + "c"
println(s1 == s2)     // true  ⭐比内容
println(s1 === s2)    // 比地址

// 装箱缓存陷阱：可空类型才会装箱
val a: Int? = 127
val b: Int? = 127
println(a == b)       // true
println(a === b)      // true   ← -128~127 走 Integer 缓存，同一对象

val c: Int? = 128
val d: Int? = 128
println(c == d)       // true
println(c === d)      // ⭐false  ← 超出缓存范围，装箱成两个对象
```

> 结论：**业务代码一律用 `==`**，`===` 只在判断"是不是同一个对象"时用。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐**能用 `val` 就不用 `var`** | 减少可变状态，天然线程友好，也是 code review 重点 |
| 类型能推断就不写 | `val name = "Tom"` 优于 `val name: String = "Tom"` |
| 公开 API 显式写返回类型 | 便于阅读，也避免改实现时意外改变对外类型 |
| 常量用 `const val` + 全大写下划线 | `const val MAX_RETRY = 3` |
| 金额一律 `BigDecimal` | 禁止用 `Double` 算钱 |
| 数字字面量用 `_` 分隔 | `30_000L` 比 `30000L` 好读 |
| 小驼峰命名，禁拼音、禁单字母 | `userName`、`orderId` |

---

### 9. 面试重点

**Q1：`val` 和 Java 的 `final` 完全一样吗？**

大体一致（编译后确实是 `final`），但有两点差异：① `val` 只保证**引用**不可变，指向的对象内容仍可修改；② `val` 属性可以自定义 getter，每次访问返回不同值，例如 `val random get() = Random.nextInt()`，这是 `final` 做不到的。

**Q2：`val` 和 `const val` 的区别？**

`val` 在**运行期**赋值，编译成私有字段 + getter 方法，可以放在任何位置；`const val` 是**编译期常量**，值会直接内联到使用处（无方法调用开销），但只能声明在顶层 / `object` / `companion object` 中，且类型必须是基本类型或 `String`。

**Q3：Kotlin 有基本数据类型吗？`Int` 会装箱吗？**

源码层面没有，`Int` 就是一个类。但编译器会优化：**能用基本类型时编译成 `int`**；只有在**可空（`Int?`）、作为泛型参数（`List<Int>`）、或需要当作对象**时才装箱成 `Integer`。所以 `Int` 不等于永远装箱。

**Q4：`==` 和 `===` 的区别？下面这段为什么是 false？**

```kotlin
val a: Int? = 128; val b: Int? = 128
println(a === b)   // false
```

`==` 比内容（调 `equals`，且自带空判断），`===` 比引用。`Int?` 会装箱成 `Integer`，而 JVM 的 `Integer` 缓存只覆盖 **-128 ~ 127**，128 超出缓存，两次装箱产生两个不同对象，所以 `===` 为 `false`（`==` 仍为 `true`）。

**Q5：Kotlin 为什么取消了隐式类型转换？**

为了避免 Java 中隐式扩宽带来的隐蔽问题（尤其是与装箱、`equals` 混用时的判等错误），Kotlin 要求所有数值转换显式调用 `toLong()` 之类的方法，把"这里发生了类型变化"摆在明面上。代价是代码略啰嗦，收益是少一类难查的 bug。

**Q6：变量声明时不给初始值可以吗？**

局部变量可以先声明后赋值，但编译器会做**确定性赋值检查**，使用前必须保证所有路径都已赋值；类的属性必须在声明或构造器中初始化，否则要用 `lateinit`（非空且延后初始化）或 `by lazy`（首次访问时初始化）。

---

## 三、空安全（Null Safety）

> Kotlin 最核心的特性，也是面试第一高频考点。一句话概括：**把 NPE 从运行时崩溃，变成编译期报错。**

### 1. 可空类型与非空类型 ⭐

类型后面加 `?` 才允许为 null，这是整套空安全的地基：

```kotlin
var name: String = "Tom"
name = null           // ✘ 编译错误：Null can not be a value of a non-null type String

var nick: String? = "Jerry"
nick = null           // ✔ 可空类型
println(nick.length)  // ✘ 编译错误：可空类型不能直接调用成员
```

| | `String` | `String?` |
| ---- | ---- | ---- |
| 能否为 null | 不能 | 能 |
| 直接调用成员 | ✔ | ✘ 必须先处理空 |
| 编译器保证 | 永不为 null | 使用前强制判空 |

> **设计思想**：把"可能为空"写进**类型**里。编译器强制你在用之前处理它，NPE 就没有藏身之处了。

---

### 2. 处理空值的五种方式 ⭐

**① `?.` 安全调用 —— 为空就返回 null，不执行**

```kotlin
val len: Int? = nick?.length            // nick 为 null 时，len 也是 null，不崩溃

user?.address?.city?.name               // ⭐链式调用，任意一环为空，整体为 null
```

**② `?:` Elvis 运算符 —— 为空时给默认值**

```kotlin
val len = nick?.length ?: 0             // 为空则取 0
val name = user?.name ?: "未知用户"

// ⭐右侧可以是 return / throw，常用于参数校验（提前返回）
fun save(user: User?) {
    val id = user?.id ?: return                       // 为空直接返回
    val name = user.name ?: throw IllegalArgumentException("name 不能为空")
}
```

**③ `?.let {}` —— 非空才执行一段逻辑**

```kotlin
nick?.let {
    println("长度是 ${it.length}")      // 只有 nick 非空时才会进来
}
```

**④ `as?` 安全类型转换 —— 转换失败返回 null，不抛异常**

```kotlin
val str = obj as String       // ✘ 类型不符会抛 ClassCastException
val str = obj as? String      // ✔ 类型不符返回 null
val str = obj as? String ?: ""  // ⭐配合 Elvis 兜底
```

**⑤ `!!` 非空断言 —— 我确定不为空，为空就崩给我看**

```kotlin
val len = nick!!.length       // ⚠ nick 为 null 时抛 NullPointerException
```

> ⭐ **`!!` 是空安全的后门，等于主动放弃编译器保护。** 企业代码中原则上禁用，code review 见到 `!!` 就要问一句"为什么"。真正需要"必须非空否则视为 bug"的场景，用 `requireNotNull()` / `checkNotNull()`，它们能带上明确的错误信息。

---

### 3. 智能转换（Smart Cast）⭐

判空之后，编译器自动把类型收窄为非空，后面直接用：

```kotlin
fun show(text: String?) {
    if (text != null) {
        println(text.length)     // ⭐这里 text 已被智能转换为 String，无需 ?.
    }
}

// 提前返回写法更常用（减少嵌套）
fun show(text: String?) {
    if (text == null) return
    println(text.length)         // 同样是智能转换
}
```

**⚠ 智能转换失效的场景（高频面试题）**：

```kotlin
class User {
    var name: String? = null
}

fun test(user: User) {
    if (user.name != null) {
        println(user.name.length)   // ✘ 编译错误！
    }
}
```

原因：`user.name` 是**可变属性（var）**，编译器无法保证"判空之后、使用之前"它没有被其他线程或其他代码改成 null。

**三种解法**：

```kotlin
val name = user.name              // ① 赋给局部 val，最推荐
if (name != null) println(name.length)

user.name?.let { println(it.length) }        // ② let
println(user.name?.length ?: 0)              // ③ 安全调用 + Elvis
```

> 规律：智能转换只对**局部变量、`val` 属性**等编译器能确定不会改变的对象生效。

---

### 4. 可空类型的常用标准库

```kotlin
val s: String? = null

s.isNullOrEmpty()          // ⭐true —— 注意：可空接收者，不需要 ?.
s.isNullOrBlank()          // true —— 空、null 或全是空白字符
s.orEmpty()                // "" —— 为空时返回空字符串

val list: List<String>? = null
list.orEmpty()             // 空集合，避免遍历前判空

listOfNotNull(1, null, 2)              // [1, 2] 构建时直接过滤 null
listOf(1, null, 2).filterNotNull()     // [1, 2] ⭐List<Int?> → List<Int>
listOf("1", "a").mapNotNull { it.toIntOrNull() }   // [1] 转换 + 过滤一步完成

requireNotNull(user) { "用户不能为空" }   // 为空抛 IllegalArgumentException
checkNotNull(config) { "配置未初始化" }   // 为空抛 IllegalStateException
```

**集合中的可空要分清两种写法**：

```kotlin
val a: List<String?>    // 集合非空，元素可能为空
val b: List<String>?    // 集合可能为空，元素非空
val c: List<String?>?   // 都可能为空
```

---

### 5. 平台类型：空安全的最大漏洞 ⭐

调用 Java 代码时，Kotlin **无法知道**返回值是否可空，于是标记为**平台类型 `String!`**（你不能自己写这个类型，只会在 IDE 提示中看到）：

```kotlin
// Java 代码
public class JavaUtil {
    public static String getName() { return null; }
}
```

```kotlin
val name = JavaUtil.getName()    // 推断为 String!（平台类型）
println(name.length)             // ⚠ 编译通过，运行时 NPE！
```

**平台类型 = 编译器放弃检查**，责任交还给开发者。三种应对：

```kotlin
val name: String? = JavaUtil.getName()   // ⭐① 显式声明为可空，最安全
val name: String = JavaUtil.getName()    // ② 声明为非空，Kotlin 会在赋值处插入检查，
                                          //   为 null 时立刻抛错（快速失败，好过传染下去）
```

③ 在 Java 侧加 `@Nullable` / `@NonNull` 注解（如 `org.jetbrains.annotations`、`androidx.annotation`），Kotlin 会识别并映射成 `String?` / `String`。

> **结论**：混合工程里，**Java 来的值一律当可空处理**。这也是面试常问的"Kotlin 还会不会 NPE"的标准答案之一。

---

### 6. Kotlin 还会出现 NPE 吗？

会，且只有这几种情况：

| 来源 | 说明 |
| ---- | ---- |
| ⭐`!!` 非空断言 | 主动放弃保护 |
| ⭐平台类型 | Java 返回 null 却当非空用 |
| `lateinit` 未初始化 | 抛 `UninitializedPropertyAccessException`（属 IllegalState） |
| 构造器中泄漏 `this` | 初始化未完成就被外部访问 |
| 显式 `throw NullPointerException()` | 手动抛 |

除此之外，纯 Kotlin 代码几乎不可能出现 NPE。

---

### 7. 实现原理（加分项）

空安全**主要是编译期能力**，运行时只留少量断言：

1. **类型检查**：可空类型不允许直接解引用，编译期报错。
2. **插桩校验**：编译器在 public 函数入口插入 `Intrinsics.checkNotNullParameter(param, "param")`，防止 Java 侧传 null 绕过检查。
3. **`?.` 的本质**：编译成 `if (x == null) null else x.foo()`，是语法糖，**没有额外性能开销**。
4. **字节码层面**：`String` 和 `String?` 都是 `java.lang.String`，可空信息通过 `@Nullable` / `@NotNull` 注解保留在元数据中。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐**默认用非空类型** | 只有业务上真的允许为空才加 `?`，不要图省事全写可空 |
| ⭐**禁用 `!!`** | 用 `?:` 兜底、`?.let` 或 `requireNotNull()` 替代 |
| Java 交互一律按可空处理 | 接收 Java 返回值时显式声明 `?` |
| 优先"提前返回"而非嵌套判空 | `val id = user?.id ?: return` 比多层 `if` 清晰 |
| 集合返回空集合而非 null | 用 `emptyList()`，调用方少一次判空 |
| 参数校验用 `require` / `check` | 错误信息明确，便于排查 |

---

### 9. 面试重点

**Q1：Kotlin 是怎么做到空安全的？**

把可空性写进**类型系统**：`String` 与 `String?` 是两个不同类型，非空类型不允许赋 null，可空类型不允许直接调用成员，必须经过 `?.`、`?:`、判空智能转换等方式处理。检查发生在**编译期**，运行时只在函数入口插入少量 `Intrinsics` 断言防止 Java 侧传空。

**Q2：`?.`、`?:`、`!!` 各自的作用？**

`?.` 安全调用，接收者为空时整个表达式返回 null；`?:` Elvis，左边为空时取右边的值，右边还可以是 `return`/`throw`；`!!` 非空断言，强制当作非空，为空时抛 NPE —— 相当于放弃空安全保护，生产代码应避免。

**Q3：Kotlin 里还会有 NPE 吗？**

会。主要来自 `!!`、**平台类型**（Java 返回 null 被当作非空）、`lateinit` 未初始化访问，以及手动抛出。纯 Kotlin 且不用 `!!` 的代码基本不会出现。

**Q4：什么是平台类型？为什么危险？**

Kotlin 调用 Java 时，因为无法确定可空性而产生的中间类型（IDE 显示为 `String!`），编译器对它**不做空检查**。危险在于：Java 返回 null，你却当非空用，编译期毫无提示，运行时直接 NPE。解决办法是接收时显式声明为可空类型，或在 Java 侧补 `@Nullable` / `@NonNull` 注解。

**Q5：为什么下面这段代码编译不通过？**

```kotlin
if (user.name != null) { println(user.name.length) }
```

`user.name` 是 `var` 属性，编译器无法保证判空后到使用前它不被修改（例如其他线程改成 null），因此**智能转换不生效**。解决：先赋给局部 `val`，或用 `user.name?.let { }`。智能转换只对局部变量和 `val` 属性等不可变对象生效。

**Q6：`?.let` 和 `if != null` 有什么区别？**

功能上等价，但 `?.let` 是表达式，可以直接返回值并参与链式调用，且对 `var` 属性同样有效（因为接收者只求值一次）；`if != null` 更直观，适合逻辑较长、需要 `else` 分支的场景。可读性优先，不要为了链式而堆叠多层 `let`。

---

## 四、类型系统（Any、Unit、Nothing 与类型转换）

### 1. 类型层次结构 ⭐

```text
            Any?          ← 顶层类型（Top），万物之父，可以装任何值包括 null
           /    \
        Any      String? / Int? ... （各种可空类型）
       /   \              |
  String   Int ...        |
       \   /              |
      Nothing            Nothing?   ← 底层类型（Bottom），是所有类型的子类型
```

| 类型 | 定位 | 一句话理解 |
| ---- | ---- | ---- |
| ⭐`Any` | 所有**非空**类型的父类 | 对应 Java 的 `Object` |
| ⭐`Any?` | 真正的顶层类型 | `Any` + null |
| ⭐`Unit` | 表示"无返回值" | 对应 Java 的 `void`，但它是真实对象 |
| ⭐`Nothing` | 表示"根本不会返回" | 没有任何实例，专门标记异常/死循环 |

---

### 2. Any：所有类的父类

```kotlin
val a: Any = 1
val b: Any = "abc"
val c: Any? = null      // 只有 Any? 能装 null
```

**Any 只有三个方法**：`equals()`、`hashCode()`、`toString()`。

Java `Object` 上的 `wait()` / `notify()` / `getClass()` **不在 Any 上**，因为它们与 JVM 强绑定，Kotlin 要保证跨平台：

```kotlin
val clazz = obj.javaClass          // ⭐取 Class 用 javaClass（Kotlin 反射用 ::class）
(obj as java.lang.Object).wait()   // 需要 wait/notify 必须显式转成 Object（几乎不该这么写）
```

> 编译后 `Any` 就是 `java.lang.Object`，所以互操作没有任何障碍。

---

### 3. Unit：Kotlin 版的 void

```kotlin
fun log(msg: String): Unit {       // 返回类型是 Unit
    println(msg)
}

fun log(msg: String) {             // ⭐Unit 可以省略，实际开发都这么写
    println(msg)
}
```

| 对比项 | Java `void` | Kotlin `Unit` |
| ---- | ---- | ---- |
| 本质 | 关键字，"没有类型" | **单例对象**，是真实的类型 |
| 能否作泛型参数 | ✘ 只能用 `Void` 变通 | ✔ 直接写 `List<Unit>`、`(Int) -> Unit` |
| 编译结果 | `void` | 返回位置编译成 `void` |

**为什么要设计成对象？** 为了让函数式编程统一：Lambda 类型 `(Int) -> Unit` 里的 `Unit` 必须是一个真实类型，否则"无返回值的函数"就无法作为参数传递。

```kotlin
val action: (String) -> Unit = { println(it) }    // ⭐高阶函数中天天见
```

---

### 4. Nothing：永不返回 ⭐

`Nothing` 是**所有类型的子类型**，且**没有任何实例**，用来告诉编译器"执行到这里就不会再往下走了"。

```kotlin
fun fail(msg: String): Nothing = throw IllegalStateException(msg)   // 只抛异常，永不返回

fun loop(): Nothing {           // 死循环也是 Nothing
    while (true) { }
}

TODO("待实现")                   // 标准库的 TODO() 返回 Nothing，抛 NotImplementedError
```

**它到底有什么用？** 让下面这些写法能通过类型检查：

```kotlin
// ① throw 是表达式，类型是 Nothing，而 Nothing 是 String 的子类型，所以整体推断为 String
val name: String = user.name ?: throw IllegalArgumentException("name 为空")

// ② 自定义 fail 同样可以放在 Elvis 右边
val id: Long = user.id ?: fail("id 为空")

// ③ 编译器知道 fail() 之后的代码不可达
fun check(x: Int) {
    if (x < 0) fail("非法")
    println(x)                  // 编译器确定这里 x >= 0
}
```

**`Nothing?` 只有一个值：null**

```kotlin
val x = null        // ⭐类型被推断为 Nothing?
```

---

### 5. 类型检查与转换 ⭐

| 运算符 | 作用 | 失败时 |
| ---- | ---- | ---- |
| `is` / `!is` | 类型判断，⭐**自动智能转换** | 返回 false |
| `as` | 强制转换（不安全） | 抛 `ClassCastException` |
| ⭐`as?` | 安全转换 | 返回 `null` |

```kotlin
fun handle(obj: Any) {
    if (obj is String) {
        println(obj.length)        // ⭐is 之后自动智能转换为 String，无需再写 as
    }

    val s1 = obj as String         // ⚠ 类型不符直接崩
    val s2 = obj as? String        // ✔ 类型不符得到 null
    val s3 = obj as? String ?: ""  // ⭐工程中最常用：安全转换 + 兜底
}
```

**配合 `when` 使用（最常见的写法）**：

```kotlin
fun parse(obj: Any): String = when (obj) {
    is String -> "字符串，长度 ${obj.length}"       // 每个分支内自动转换
    is Int    -> "整数，加一 ${obj + 1}"
    is List<*> -> "集合，大小 ${obj.size}"
    else      -> "未知类型"
}
```

**⚠ 泛型擦除的限制**：

```kotlin
if (list is List<String>)   // ✘ 编译错误：运行时泛型已擦除，无法判断
if (list is List<*>)        // ✔ 只能判断"是不是 List"，元素类型未知
```

> 想在运行时拿到泛型的真实类型，需要 `inline` + `reified`（见第十章）。

---

### 6. 类型别名 typealias

给已有类型起个短名字，**不产生新类型**，纯粹为了可读性：

```kotlin
typealias UserMap = HashMap<String, List<User>>
typealias OnClick = (View, Int) -> Unit        // ⭐给复杂函数类型起名，最实用的场景

fun setListener(listener: OnClick) { }
```

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐类型转换优先 `as?` | 配合 `?:` 兜底，避免 `ClassCastException` |
| ⭐用 `is` + 智能转换，不要先 `is` 再 `as` | 编译器已经帮你转好了 |
| 少用 `Any` 作参数类型 | 丢失类型信息，能用泛型就用泛型 |
| 返回类型 `Unit` 一律省略 | 保持简洁 |
| 抛异常的工具方法声明为 `Nothing` | 让编译器识别不可达分支，也能用在 Elvis 右侧 |
| 复杂函数类型用 `typealias` | 尤其是回调，提升可读性 |

---

### 8. 面试重点

**Q1：`Any` 和 Java 的 `Object` 是一回事吗？**

编译后确实是 `java.lang.Object`，但源码层面有三点差异：① `Any` 只声明了 `equals`/`hashCode`/`toString`，没有 `wait`/`notify`/`getClass`（这些与 JVM 绑定，Kotlin 要支持多平台）；② `Any` 不可为空，真正的顶层类型是 `Any?`；③ 取 Class 要用 `javaClass` 或 `::class`。

**Q2：`Unit` 和 `void` 的区别？**

`void` 是关键字，表示"没有类型"；`Unit` 是一个**单例对象**，是真实存在的类型，因此可以作为泛型参数出现在 `(Int) -> Unit` 这类函数类型里。编译到字节码时，返回位置的 `Unit` 会变成 `void`。

**Q3：`Nothing` 是什么？有什么用？**

表示"永远不会返回值"的类型，**没有实例**，且是**所有类型的子类型**。用途：① `throw` 表达式的类型就是 `Nothing`，因此 `val a: String = b ?: throw XxxException()` 能通过类型检查；② 标记只抛异常或死循环的函数，让编译器识别后续代码不可达；③ `TODO()` 的返回类型。

**Q4：`Nothing` 和 `Unit` 有什么区别？**

`Unit` 表示**函数正常执行完了，只是没有有意义的返回值**；`Nothing` 表示**函数根本没有正常返回**（抛异常或永不结束）。前者有唯一实例，后者没有任何实例。

**Q5：`as` 和 `as?` 的区别？什么时候用 `is`？**

`as` 转换失败抛 `ClassCastException`；`as?` 失败返回 `null`，通常配 `?:` 兜底，生产代码优先用它。如果转换后要分支处理，直接用 `is`——判断成功后编译器会**智能转换**，不需要再手动 `as`。

**Q6：`val x = null` 的类型是什么？**

`Nothing?`。因为 `null` 唯一可归属的最具体类型就是 `Nothing?`（它只有 null 一个值）。实际开发中应显式写类型，例如 `val x: String? = null`，否则这个变量后面基本没法用。

---

## 五、运算符与流程控制（if、when、for、区间）

### 1. 运算符总览

| 分类 | 运算符 | 说明 |
| ---- | ---- | ---- |
| 算术 | `+` `-` `*` `/` `%` | 整数相除仍然截断：`5 / 2 = 2` |
| 自增自减 | `++` `--` | 前缀后缀规则同 Java |
| 赋值 | `=` `+=` `-=` `*=` `/=` `%=` | ⭐**赋值是语句不是表达式**，不能写 `a = b = 1` |
| 比较 | `>` `<` `>=` `<=` | 底层调用 `compareTo()` |
| 相等 | ⭐`==` `!=` | 比**内容**（调 `equals`，且空安全） |
| 引用 | `===` `!==` | 比**地址** |
| 逻辑 | `&&` `\|\|` `!` | 短路 |
| 区间 | ⭐`..` `..<` `in` `!in` | 见下文 |
| 空安全 | `?.` `?:` `!!` | 见第三章 |
| 类型 | `is` `!is` `as` `as?` | 见第四章 |

**Kotlin 没有的三样**（高频考点）：

| 缺失 | 替代写法 |
| ---- | ---- |
| ⭐三元运算符 `? :` | `val max = if (a > b) a else b`（if 是表达式） |
| 位运算符 `& \| ^ << >>` | ⭐**中缀函数**：`and` `or` `xor` `shl` `shr` `ushr` `inv` |
| `switch` | ⭐`when`，更强大 |

```kotlin
val flag = 0b1100 and 0b1010     // 8，按位与
val move = 1 shl 4               // 16，左移
val rev  = 5.inv()               // -6，按位取反
```

---

### 2. if 是表达式 ⭐

```kotlin
// 用法一：当语句用，和 Java 一样
if (score >= 60) println("及格") else println("不及格")

// ⭐用法二：当表达式用，直接返回值（代替三元运算符）
val level = if (score >= 90) "优秀" else if (score >= 60) "及格" else "不及格"

// ⭐用法三：代码块形式，最后一行就是返回值
val result = if (score >= 60) {
    println("记录日志")
    "及格"                 // ← 这一行是返回值
} else {
    "不及格"
}
```

> **当表达式用时 `else` 必须写**，否则可能没有值可返回，编译报错。

---

### 3. when：Kotlin 的杀手锏 ⭐

**基本用法（对标 switch，但不需要 break）**：

```kotlin
when (day) {
    1 -> println("周一")
    2, 3, 4, 5 -> println("工作日")     // ⭐多个值用逗号
    6, 7 -> println("周末")
    else -> println("非法输入")
}
```

**作为表达式返回值（最常用）**：

```kotlin
val desc = when (day) {
    1 -> "周一"
    in 2..5 -> "工作日"                 // ⭐支持区间
    else -> "周末"
}
```

**四种进阶用法**：

```kotlin
// ① 类型判断 + 智能转换
val info = when (obj) {
    is String -> obj.length            // 分支内已转换为 String
    is Int -> obj + 1
    else -> 0
}

// ② ⭐无参 when —— 替代多层 if-else if，可读性极佳
val level = when {
    score >= 90 -> "优秀"
    score >= 60 -> "及格"
    else -> "不及格"
}

// ③ 在 when 中直接声明变量，作用域仅限 when 内部
when (val response = api.getUser()) {
    is Success -> show(response.data)
    is Error -> toast(response.msg)
}

// ④ 条件可以是任意表达式，不限于常量
when (x) {
    getValue() -> println("相等")
    in list -> println("在集合中")      // in 底层调用 contains()
}
```

**when vs switch**：

| 对比项 | Java `switch` | Kotlin `when` |
| ---- | ---- | ---- |
| 是否需要 break | 需要，忘写就穿透 | ⭐**不需要**，匹配即结束 |
| 支持类型 | 有限（整型、枚举、String） | ⭐**任意类型** |
| 分支条件 | 只能是常量 | 常量、区间、类型、任意表达式 |
| 能否返回值 | 传统写法不行 | ⭐可以 |

> ⭐ **作为表达式时必须穷尽所有分支**：一般靠 `else` 兜底；如果条件是**枚举或密封类**且写全了所有情况，可以省略 `else`（见第十七章）——这是密封类最大的价值。

---

### 4. 区间（Range）⭐

```kotlin
1..5              // [1,2,3,4,5]  闭区间
1..<5             // [1,2,3,4]    左闭右开（Kotlin 1.9+，等价于 until）
1 until 5         // [1,2,3,4]    老写法，仍大量存在
5 downTo 1        // [5,4,3,2,1]  递减
1..10 step 2      // [1,3,5,7,9]  步长
'a'..'z'          // 字符区间
```

**区间最常用的其实是判断，而不是遍历**：

```kotlin
if (age in 18..60) println("适龄")        // ⭐比 age >= 18 && age <= 60 清爽
if (score !in 0..100) println("非法分数")
if (c in 'a'..'z') println("小写字母")
```

---

### 5. for 循环 ⭐

Kotlin **没有** `for (int i = 0; i < n; i++)` 这种传统写法，只有 for-in：

```kotlin
// ① 遍历区间
for (i in 1..5) print(i)                  // 12345
for (i in 5 downTo 1) print(i)            // 54321
for (i in 0 until list.size) { }          // 等价于 Java 的下标循环

// ② ⭐遍历集合（最常用）
for (item in list) println(item)

// ③ 需要下标时
for (i in list.indices) println(list[i])              // indices = 0..size-1
for ((index, item) in list.withIndex()) {             // ⭐同时要下标和元素
    println("$index : $item")
}

// ④ 遍历 Map
for ((key, value) in map) println("$key = $value")    // ⭐解构，非常常用

// ⑤ 固定次数
repeat(3) { i -> println("第 ${i + 1} 次") }           // ⭐标准库函数，不是关键字
```

---

### 6. while 与 do-while

与 Java 完全一致：

```kotlin
while (x > 0) { x-- }

do { x-- } while (x > 0)      // 至少执行一次
```

---

### 7. break、continue 与标签 ⭐

```kotlin
for (i in 1..5) {
    if (i == 3) continue      // 跳过本次
    if (i == 5) break         // 跳出循环
}
```

**标签（label）：跳出多重循环**

```kotlin
outer@ for (i in 1..3) {          // ⭐定义标签：名字@
    for (j in 1..3) {
        if (j == 2) continue@outer   // 继续外层循环
        if (i == 3) break@outer      // ⭐直接跳出外层循环
    }
}
```

**⚠ Lambda 中不能直接 break / continue**：

```kotlin
list.forEach {
    if (it == 3) return@forEach    // ⭐等价于 continue，跳过本次
    // break 无法直接实现，需要改用 for 循环，或用 first { } / takeWhile { } 等函数式写法
}
```

> `return@forEach` 这类叫**标签返回**，只结束当前这次 Lambda 调用；直接写 `return` 会返回外层函数（非局部返回，仅内联函数支持），详见第九、十章。

---

### 8. 运算符重载与中缀函数（了解）

Kotlin 的运算符本质是**函数调用**，用 `operator` 关键字可以给自己的类重载：

| 运算符 | 对应函数 | 运算符 | 对应函数 |
| ---- | ---- | ---- | ---- |
| `a + b` | `a.plus(b)` | `a[i]` | `a.get(i)` |
| `a - b` | `a.minus(b)` | `a[i] = v` | `a.set(i, v)` |
| `a * b` | `a.times(b)` | `a in b` | `b.contains(a)` |
| `a == b` | `a.equals(b)` | `a > b` | `a.compareTo(b) > 0` |
| `a..b` | `a.rangeTo(b)` | `a()` | `a.invoke()` |

```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point) = Point(x + other.x, y + other.y)
}
val p = Point(1, 2) + Point(3, 4)     // Point(4, 6)

// 中缀函数：单参数成员/扩展函数加 infix，可省略点和括号
infix fun Int.pow(n: Int): Int = ...
val r = 2 pow 3
```

> 业务开发**不要滥用**运算符重载，只在语义天然成立时使用（如时间、金额、向量）。

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐多分支一律用 `when`，不用 `if-else if` 长链 | 尤其是无参 `when` |
| ⭐优先把 `if` / `when` 当表达式用 | 直接赋值/返回，减少中间变量 |
| 遍历集合用 for-in 或 `forEach` | 不要手动维护下标 |
| 需要下标用 `withIndex()` / `indices` | 避免 `list[i]` 越界风险 |
| 范围判断用 `in` | `age in 18..60` 优于两个比较 |
| 密封类/枚举的 `when` 不写 `else` | ⭐新增类型时编译器会报错提醒，防止漏改 |
| 不滥用运算符重载和 `infix` | 语义不自然反而降低可读性 |

---

### 10. 面试重点

**Q1：Kotlin 为什么没有三元运算符？**

因为 `if` 本身就是**表达式**，可以直接返回值：`val max = if (a > b) a else b`，功能覆盖了三元运算符，再保留一套语法属于冗余。同理 `when`、`try` 也都是表达式。

**Q2：`when` 相比 `switch` 强在哪？**

① 不需要 `break`，不存在分支穿透；② 支持任意类型，不限于整型/枚举/String；③ 分支条件可以是常量、区间（`in 1..5`）、类型（`is String`，还自带智能转换）、甚至任意表达式；④ 可以作为表达式返回值；⑤ 支持无参形式替代 `if-else if` 长链。

**Q3：`when` 什么时候必须写 `else`？**

当 `when` **作为表达式使用**（有返回值）时必须穷尽所有可能。若条件是**枚举或密封类**且列出了全部子类型，编译器能确认已穷尽，就可以省略 `else`。这也是推荐密封类不写 `else` 的原因——将来新增子类时编译期直接报错，避免遗漏。

**Q4：Kotlin 为什么没有传统的 `for (i = 0; i < n; i++)`？**

Kotlin 只保留 for-in，用区间（`1..n`、`until`、`downTo`、`step`）表达迭代范围。好处是意图更明确、不会写错边界条件，也避免了循环变量在循环外泄漏。需要下标时用 `indices` 或 `withIndex()`。

**Q5：`until` 和 `..` 的区别？`..<` 又是什么？**

`1..5` 是闭区间包含 5；`1 until 5` 左闭右开不含 5，常用于按下标遍历。`..<` 是 Kotlin 1.9 引入的运算符，与 `until` 等价但更直观，新代码推荐使用。

**Q6：为什么 `forEach` 里不能用 `break`？怎么跳出？**

`forEach` 是普通的**高阶函数**而非语言级循环结构，Lambda 内部没有循环上下文，因此不支持 `break`/`continue`。`return@forEach` 只能实现 `continue` 的效果。要跳出应改用 `for` 循环，或换成 `first { }`、`any { }`、`takeWhile { }` 等本身带短路语义的函数。

**Q7：Kotlin 的位运算为什么写成 `and`、`shl`？**

`&`、`|`、`<<` 这些符号在 Kotlin 里被留作它用或不予支持，位运算改由标准库的**中缀函数**提供（`and`/`or`/`xor`/`shl`/`shr`/`ushr`/`inv`），可读性更好，且只对 `Int`、`Long` 开放，避免误用。

---

## 六、字符串与字符串模板

### 1. 基础特性

Kotlin 的 `String` 编译后就是 `java.lang.String`，同样**不可变**（所有"修改"方法都是返回新字符串）：

```kotlin
val s = "Hello"
println(s.length)        // 5
println(s[0])            // 'H'  ⭐可以像数组一样用下标（Java 要 charAt）
println(s.first())       // 'H'
println(s.last())        // 'o'

for (c in s) print(c)    // ⭐直接遍历字符
```

---

### 2. 字符串模板 ⭐

Kotlin 里几乎不再用 `+` 拼接字符串：

```kotlin
val name = "Tom"
val age = 20

println("我叫 $name，今年 $age 岁")                  // ⭐简单变量：$变量名
println("明年 ${age + 1} 岁")                        // ⭐表达式：${...}
println("名字长度 ${name.length}")                    // 属性/方法调用必须用 ${}
println("等级 ${if (age >= 18) "成年" else "未成年"}")  // 里面还能嵌表达式

println("单价：\$100")                                // 输出 $ 要转义
```

| 写法 | 使用场景 |
| ---- | ---- |
| `$变量` | 只有一个简单变量时 |
| `${表达式}` | ⭐属性访问、方法调用、任何表达式 |
| `\$` | 输出 `$` 本身 |

> 编译后模板会转成 `StringBuilder` 拼接（或 JVM 9+ 的 `invokedynamic` 字符串拼接），**性能与手写拼接一致**，放心用。

---

### 3. 原始字符串（多行字符串）⭐

用三引号包裹，**不处理转义符**，所见即所得，写 JSON、SQL、正则时非常有用：

```kotlin
val json = """
    {
      "name": "Tom",
      "path": "C:\Users\Tom"     // ⭐反斜杠不用转义
    }
""".trimIndent()                  // ⭐去掉每行公共的前置缩进，必加

val sql = """
    |SELECT *
    |FROM user
    |WHERE age > 18
""".trimMargin()                  // 以 | 为边界裁剪（默认前缀就是 |）
```

> `trimIndent()` 和 `trimMargin()` 是最常忘写的两个方法，不写会把代码缩进也带进字符串里。

---

### 4. 常用 API

**判空（高频）**：

```kotlin
"".isEmpty()            // true   长度为 0
"  ".isEmpty()          // false
"  ".isBlank()          // ⭐true 只有空白字符也算"空"
str.isNotEmpty()        // 取反版本
str.isNotBlank()

val s: String? = null
s.isNullOrEmpty()       // ⭐true 可空接收者，不用先判 null
s.isNullOrBlank()       // true
s.orEmpty()             // "" 为空时返回空串
```

**截取与查找**：

```kotlin
"HelloWorld".substring(0, 5)        // "Hello"
"HelloWorld".take(5)                // ⭐"Hello" 取前 5 个，越界不报错
"HelloWorld".takeLast(5)            // "World"
"HelloWorld".drop(5)                // "World" 丢弃前 5 个
"HelloWorld".indexOf("World")       // 5，找不到返回 -1
"HelloWorld".contains("World")      // true，也可写 "World" in "HelloWorld"
"HelloWorld".startsWith("He")       // true
"a.txt".substringAfterLast(".")     // ⭐"txt" 取扩展名，比 indexOf 计算下标清爽
"k=v".substringBefore("=")          // "k"
```

**转换与处理**：

```kotlin
" abc ".trim()                      // "abc"（另有 trimStart / trimEnd）
"abc".uppercase()                   // "ABC" ⭐1.5 起替代 toUpperCase()
"ABC".lowercase()                   // "abc"
"abc".reversed()                    // "cba"
"ab".repeat(3)                      // "ababab"
"5".padStart(3, '0')                // "005" 补位
"a,b,c".split(",")                  // ⭐[a, b, c]，返回 List<String>
"a b\nc".lines()                    // 按行分割
"abcdef".chunked(2)                 // [ab, cd, ef] 按长度切块
"a-b".replace("-", "_")             // "a_b"
listOf(1, 2, 3).joinToString("-")   // ⭐"1-2-3" 集合拼字符串，最常用
listOf(1, 2, 3).joinToString(prefix = "[", postfix = "]") { "第${it}项" }
```

> ⚠ Kotlin 的 `split("...")` 参数是**字面量**不是正则（与 Java 相反），需要正则时用 `split(Regex("\\d+"))`。

---

### 5. 字符串比较 ⭐

```kotlin
val a = "abc"
val b = "ABC"

a == b                          // false ⭐比内容，等价于 Java 的 equals
a.equals(b, ignoreCase = true)  // ⭐true 忽略大小写
a === b                         // 比引用地址，业务代码基本不用
a.compareTo(b)                  // 字典序比较，返回 Int
```

> Kotlin 里 `==` 就是内容比较且**自带空判断**（`null == null` 为 true 不会崩），因此**不存在 Java 那种"必须用 equals、常量放前面防 NPE"的规矩**。

---

### 6. 类型转换 ⭐

```kotlin
"123".toInt()          // 123
"abc".toInt()          // ⚠ 抛 NumberFormatException
"abc".toIntOrNull()    // ⭐null，解析失败不抛异常
"abc".toIntOrNull() ?: 0   // ⭐生产标配：转换 + 兜底

"1.5".toDoubleOrNull()
"true".toBoolean()
123.toString()
"%.2f".format(3.14159)         // "3.14"
String.format("%05d", 42)      // "00042"
```

> **处理外部输入（接口参数、配置、用户输入）一律用 `toXxxOrNull()`**，这是 Kotlin 相比 Java 最实用的小改进之一。

---

### 7. 拼接性能与 StringBuilder

```kotlin
// ✘ 循环里用 + 拼接：每次都创建新对象
var s = ""
for (i in 1..1000) s += i

// ✔ 用 StringBuilder
val sb = StringBuilder()
for (i in 1..1000) sb.append(i)

// ⭐更 Kotlin 的写法：buildString
val result = buildString {
    for (i in 1..1000) append(i)
}

// ⭐集合拼接直接用 joinToString，不要手写循环
val str = list.joinToString(",")
```

---

### 8. 与 Java 的差异速览

| 对比项 | Java | Kotlin |
| ---- | ---- | ---- |
| 拼接 | `"a" + b + "c"` | ⭐字符串模板 `"a${b}c"` |
| 多行文本 | 手动 `\n` 拼接 | ⭐`"""..."""` + `trimIndent()` |
| 取字符 | `charAt(0)` | `s[0]` |
| 内容比较 | `equals()`，`==` 是比地址 | ⭐`==` 就是比内容 |
| 判空 | `s != null && !s.isEmpty()` | ⭐`s.isNullOrEmpty()` |
| 转数字失败 | 抛异常 | ⭐`toIntOrNull()` 返回 null |
| split 参数 | 正则 | 字面量（正则需显式 `Regex`） |
| 集合拼接 | 手写循环或 `String.join` | ⭐`joinToString()` |

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐拼接一律用字符串模板 | 禁止 `"a" + b + "c"` 这种可读性差的写法 |
| ⭐外部输入用 `toIntOrNull()` | 配合 `?:` 给默认值，避免异常 |
| 判空用 `isNullOrBlank()` | 用户输入场景要判空白而非仅判空 |
| 多行文本必加 `trimIndent()` | 否则缩进会进入字符串 |
| 集合转字符串用 `joinToString()` | 不要手写循环拼接 |
| 循环拼接用 `buildString` | 避免创建大量临时对象 |
| 常量字符串抽到 `const val` | 禁止魔法值散落各处 |

---

### 10. 面试重点

**Q1：Kotlin 字符串模板的原理是什么？有性能问题吗？**

模板是**编译期语法糖**，`"$a-$b"` 会被编译成 `StringBuilder` 拼接（JVM 9+ 目标则是 `invokedynamic` 的 `StringConcatFactory`），与手写拼接生成的字节码基本一致，**没有额外开销**。但在循环中反复拼接仍然会产生大量临时对象，应改用 `buildString` 或 `StringBuilder`。

**Q2：Kotlin 的 `==` 比较字符串安全吗？和 Java 有什么区别？**

安全。Kotlin 的 `==` 编译成 `Intrinsics.areEqual(a, b)`，内部先判 null 再调 `equals`，因此**比的是内容且不会 NPE**。Java 的 `==` 比的是引用地址，必须用 `equals`，还要注意常量池带来的误判。要比引用请用 `===`。

**Q3：`isEmpty()` 和 `isBlank()` 的区别？**

`isEmpty()` 只判断长度是否为 0；`isBlank()` 还会把只含空格、制表符、换行的字符串视为空。**处理用户输入用 `isBlank()`** 更贴合业务语义。对应的可空版本是 `isNullOrEmpty()` / `isNullOrBlank()`，它们定义在可空接收者上，不需要先判 null。

**Q4：`toInt()` 和 `toIntOrNull()` 怎么选？**

`toInt()` 解析失败抛 `NumberFormatException`，适用于"格式一定正确，错了就是 bug"的场景；`toIntOrNull()` 失败返回 `null`，配合 `?:` 给默认值，适用于接口参数、配置、用户输入等**不可信数据**——生产代码绝大多数情况该用后者。

**Q5：原始字符串为什么要加 `trimIndent()`？**

三引号字符串会**原样保留**每行的前导空格，而代码里的缩进也在其中。`trimIndent()` 会计算所有非空行的公共最小缩进并整体去掉，同时删除首尾空行。若想显式控制边界，用 `trimMargin()` 配合 `|` 前缀。

**Q6：Kotlin 的 `split` 和 Java 的有什么坑？**

Java 的 `String.split(String)` 参数按**正则**解释，所以 `split(".")` 会得到空数组；Kotlin 的 `split(String)` 参数是**字面量**，`split(".")` 就是按点号切分，符合直觉。Kotlin 中要用正则必须显式传 `Regex(...)`。另外 Kotlin 返回 `List<String>` 而非数组。

---

## 七、数组与集合基础（只读与可变）

> 本章只讲**结构和基础操作**；`map`、`filter`、`groupBy` 等函数式操作见第十三章。

### 1. 数组

```kotlin
val a = arrayOf(1, 2, 3)              // Array<Int>
val b = arrayOfNulls<String>(5)       // 长度 5，全是 null 的 Array<String?>
val c = Array(5) { it * 2 }           // ⭐[0,2,4,6,8] 用 Lambda 按下标初始化
val d = IntArray(5)                   // [0,0,0,0,0] 默认值填充

println(a[0])                          // 取值
a[0] = 10                              // 赋值（数组元素永远可变）
println(a.size)                        // ⭐size 是属性，不是 length
for (i in a.indices) { }               // 按下标遍历
```

**⭐ 基本类型数组：避免装箱**

| 写法 | 编译后 | 说明 |
| ---- | ---- | ---- |
| `Array<Int>` | `Integer[]` | ⚠ **每个元素都装箱** |
| ⭐`IntArray` | `int[]` | 无装箱，性能好 |

同类还有 `LongArray`、`DoubleArray`、`FloatArray`、`ByteArray`、`ShortArray`、`CharArray`、`BooleanArray`，构造函数为 `intArrayOf(1, 2, 3)`。

**⚠ 数组比较必须用 contentEquals**：

```kotlin
val x = intArrayOf(1, 2)
val y = intArrayOf(1, 2)
x == y                 // false ⭐比的是引用
x.contentEquals(y)     // ✔ true 比内容
println(x.contentToString())      // [1, 2]（直接 toString 打印的是地址）
```

**数组与集合互转**：

```kotlin
arrayOf(1, 2).toList()          // Array → List
listOf(1, 2).toTypedArray()     // List → Array<Int>
listOf(1, 2).toIntArray()       // List → IntArray
```

> **业务开发优先用集合**，数组只在性能敏感、与 Java API 交互、或用 `vararg` 时使用。

---

### 2. 集合体系：只读与可变 ⭐

Kotlin 集合最大的设计特点：**每种集合都有"只读接口"和"可变接口"两套**。

```text
        Iterable
           │
      Collection ────────────── MutableCollection
       /      \                    /        \
    List      Set            MutableList   MutableSet
                                  
        Map  ──────────────────  MutableMap    （Map 不属于 Collection）
```

| 只读接口 | 可变接口 | 说明 |
| ---- | ---- | ---- |
| `List` | `MutableList` | 有序、可重复 |
| `Set` | `MutableSet` | 无序、去重 |
| `Map` | `MutableMap` | 键值对，键唯一 |

**创建方式**：

```kotlin
// 只读
val list = listOf(1, 2, 3)
val set  = setOf(1, 2, 2)              // [1, 2] 自动去重
val map  = mapOf("a" to 1, "b" to 2)   // ⭐to 是中缀函数，生成 Pair
val empty = emptyList<Int>()
val clean = listOfNotNull(1, null, 2)  // [1, 2] 过滤 null

// 可变
val mList = mutableListOf(1, 2)
val mSet  = mutableSetOf(1, 2)
val mMap  = mutableMapOf("a" to 1)

// 指定具体实现（需要特定行为时）
val al = arrayListOf(1, 2)             // ArrayList
val ls = linkedSetOf(1, 2)             // LinkedHashSet，保持插入顺序
val hm = hashMapOf("a" to 1)           // HashMap，不保证顺序
```

> 默认实现：`mutableListOf` → `ArrayList`，`mutableSetOf` → `LinkedHashSet`，`mutableMapOf` → `LinkedHashMap`（**默认都保持插入顺序**，这点和 Java 直接 `new HashMap` 不同）。

---

### 3. ⚠ 只读 ≠ 不可变（核心考点）⭐

`List` 只是**没有提供修改方法的接口视图**，底层对象仍然是可变的：

```kotlin
val mutable = mutableListOf(1, 2)
val readOnly: List<Int> = mutable      // 只读视图

readOnly.add(3)      // ✘ 编译错误：List 没有 add
mutable.add(3)       // ✔ 通过原引用修改
println(readOnly)    // ⭐[1, 2, 3] 只读视图跟着变了！
```

**要真正隔离，必须复制一份**：

```kotlin
val snapshot = mutable.toList()        // ⭐复制，之后 mutable 怎么改都不影响它
```

同理，**Java 代码可以直接修改 Kotlin 传过去的只读 List**（编译后都是 `java.util.List`，只读性只存在于 Kotlin 编译期）。

---

### 4. List 常用操作

```kotlin
val list = listOf("a", "b", "c")

list[0]                     // "a"，越界抛 IndexOutOfBoundsException
list.getOrNull(10)          // ⭐null，安全取值
list.getOrElse(10) { "默认" } // 越界给默认值
list.first()                // "a"（空集合会抛异常）
list.firstOrNull()          // ⭐空集合返回 null
list.last()
list.size
list.isEmpty()
list.isNotEmpty()
"a" in list                 // ⭐true，等价于 contains
list.indexOf("b")           // 1
list.subList(0, 2)          // [a, b]

// 只读集合的"修改"：返回新集合
val more = list + "d"       // ⭐[a,b,c,d]，原集合不变
val less = list - "a"       // [b, c]

// 可变集合的真正修改
val m = mutableListOf(1, 2)
m.add(3); m.remove(1); m[0] = 9; m.clear()
m += 4                      // ⭐可变集合的 += 是原地添加
```

---

### 5. Map 常用操作 ⭐

```kotlin
val map = mapOf("a" to 1, "b" to 2)

map["a"]                       // ⭐1，返回 Int? —— 键不存在时为 null
map["x"] ?: 0                  // 兜底写法
map.getValue("a")              // 1，键不存在抛 NoSuchElementException
map.getOrElse("x") { 0 }       // 0
map.containsKey("a")           // 也可写 "a" in map
map.keys                       // 所有键
map.values                     // 所有值
map.size

// ⭐遍历（解构写法最常用）
for ((k, v) in map) println("$k = $v")
map.forEach { (k, v) -> println("$k = $v") }

// 可变 Map
val m = mutableMapOf<String, Int>()
m["a"] = 1                     // ⭐等价于 put
m.put("b", 2)
m.remove("a")
m.getOrPut("c") { 3 }          // ⭐没有则放入并返回，缓存场景常用
```

> **`map[key]` 的返回值是可空类型**，这是 Kotlin 相比 Java 的重要改进——强制你处理"键不存在"的情况。

---

### 6. 与 Java 集合的对应关系

| Kotlin | 编译后 / 实际类型 | 备注 |
| ---- | ---- | ---- |
| `List<T>` / `MutableList<T>` | `java.util.List` | ⭐只读性仅存在于 Kotlin 编译期 |
| `listOf()` | 不可变的内部实现 | 调用 add 会抛 `UnsupportedOperationException` |
| `mutableListOf()` | `ArrayList` | |
| `mutableSetOf()` | `LinkedHashSet` | 保持插入顺序 |
| `mutableMapOf()` | `LinkedHashMap` | 保持插入顺序 |
| `Array<T>` | `T[]` | 装箱 |
| `IntArray` | `int[]` | 不装箱 |

> Kotlin **没有提供线程安全集合**，并发场景仍然直接使用 `ConcurrentHashMap`、`CopyOnWriteArrayList` 等 JUC 类。

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐**默认用只读集合** | 需要修改才用 `mutableXxx`，减少意外变更 |
| ⭐对外暴露只读类型 | 内部 `MutableList`，对外属性声明为 `List`，防止外部乱改 |
| 返回空集合而非 null | 用 `emptyList()`，调用方省一次判空 |
| 取值用 `getOrNull` / `getOrElse` | 避免下标越界崩溃 |
| Map 取值必须处理 null | `map[k] ?: default` |
| 基本类型大数组用 `IntArray` | 避免装箱带来的内存和 GC 压力 |
| 数组比较用 `contentEquals` | `==` 比的是引用 |
| 并发场景用 JUC 集合 | Kotlin 标准集合非线程安全 |

---

### 8. 面试重点

**Q1：Kotlin 的 `List` 和 `MutableList` 有什么区别？只读集合是不可变的吗？**

`List` 是**只读接口**，只暴露读取方法；`MutableList` 额外提供 `add`/`remove` 等修改方法。⭐**只读不等于不可变**：只读集合只是一个视图，如果底层对象还被一个 `MutableList` 引用着，它的内容照样会变。要得到真正的快照必须 `toList()` 复制一份。此外 Java 侧拿到这个集合可以随意修改，因为编译后都是 `java.util.List`。

**Q2：`Array<Int>` 和 `IntArray` 有什么区别？**

`Array<Int>` 编译成 `Integer[]`，每个元素都是装箱对象；`IntArray` 编译成 `int[]`，无装箱。数据量大或性能敏感时必须用 `IntArray`，能显著减少内存占用和 GC 压力。

**Q3：为什么 `map["key"]` 返回的是可空类型？**

因为键可能不存在。Kotlin 用类型系统强制调用方处理这种情况，避免 Java 中 `map.get()` 返回 null 后直接使用导致的 NPE。确定键一定存在时可以用 `getValue()`（不存在则抛异常），或用 `getOrElse` / `getOrPut` 提供默认值。

**Q4：`listOf()` 创建的集合调用 `add` 会怎样？**

编译期就过不了——`List` 接口根本没有 `add` 方法。如果通过 Java 侧或强转成 `MutableList` 去调用，会抛 `UnsupportedOperationException`，因为 `listOf()` 返回的是内部的不可变实现。

**Q5：数组能用 `==` 比较吗？**

不能。数组的 `==` 比的是引用地址，比较内容要用 `contentEquals()`（多维数组用 `contentDeepEquals()`）。打印同理，要用 `contentToString()`，直接 `toString()` 打印的是地址。

**Q6：为什么推荐对外暴露只读集合？**

封装性。类内部用 `MutableList` 维护数据，对外属性声明成 `List` 类型，外部就无法直接增删元素，只能通过类提供的方法修改，从而保证状态变更可控——这也是 code review 常见的检查点。

---

## 八、函数（默认参数、具名参数、可变参数）

### 1. 基本语法

```kotlin
fun sum(a: Int, b: Int): Int {      // fun 函数名(参数名: 类型): 返回类型
    return a + b
}

fun log(msg: String) {              // ⭐返回 Unit 时省略返回类型
    println(msg)
}
```

**三条规则**：

| 规则 | 说明 |
| ---- | ---- |
| 参数**必须**显式写类型 | 函数参数不做类型推断 |
| ⭐参数是 `val`，**不能重新赋值** | Kotlin 参数不存在 `var`，避免副作用 |
| 有返回值时**必须**写返回类型 | 单表达式函数除外 |

```kotlin
fun test(a: Int) {
    a = 10        // ✘ 编译错误：Val cannot be reassigned
}
```

---

### 2. 单表达式函数 ⭐

函数体只有一个表达式时，用 `=` 直接写，返回类型可省略：

```kotlin
fun sum(a: Int, b: Int) = a + b                     // ⭐等价于 { return a + b }
fun isAdult(age: Int) = age >= 18
fun level(score: Int) = when {                       // 配合 when 表达式非常常用
    score >= 90 -> "优秀"
    score >= 60 -> "及格"
    else -> "不及格"
}
```

> **公开 API 建议显式写返回类型**：`fun sum(a: Int, b: Int): Int = a + b`，避免改实现时对外类型悄悄变化。

---

### 3. 默认参数 ⭐

给参数设默认值，调用时可省略——**Kotlin 靠它消灭了大量方法重载**：

```kotlin
fun createUser(
    name: String,
    age: Int = 18,                       // 默认值
    city: String = "北京",
    vip: Boolean = false
) { }

createUser("Tom")                        // 其余用默认值
createUser("Tom", 20)
createUser("Tom", 20, "上海", true)
```

对比 Java 需要写 4 个重载方法，Kotlin 一个函数搞定。

**两个细节**：

```kotlin
// ① 默认值可以引用前面的参数、调用函数
fun copy(src: String, dest: String = src.uppercase()) { }

// ② ⭐跳过中间参数必须用具名参数
createUser("Tom", vip = true)            // 跳过 age、city
```

**⚠ 与 Java 互操作**：默认参数编译后是**一个带 bitmask 的合成方法**，不是多个重载，因此 **Java 侧调用不到默认值**。需要给 Java 用时加注解：

```kotlin
@JvmOverloads                            // ⭐编译器会额外生成所有参数组合的重载方法
fun createUser(name: String, age: Int = 18) { }
```

---

### 4. 具名参数 ⭐

调用时写上参数名，**提升可读性 + 可打乱顺序**：

```kotlin
createUser(name = "Tom", city = "上海", age = 20)      // ⭐顺序随意

// 典型价值：消灭"魔法参数"
setMargin(0, 0, 10, 0)                                 // ✘ 完全看不懂
setMargin(left = 0, top = 0, right = 10, bottom = 0)   // ✔ 一目了然

showDialog(cancelable = false)                          // ✘ showDialog(false)
```

**规则**：

- 可以与位置参数混用，但**位置参数必须处在它本来的位置上**，写乱了编译报错。
- ⚠ **调用 Java 方法时不能用具名参数**（Java 字节码默认不保留参数名）。

---

### 5. 可变参数 vararg ⭐

```kotlin
fun sum(vararg nums: Int): Int {
    // ⭐函数内部 nums 的类型是 IntArray（引用类型则是 Array<out T>）
    return nums.sum()
}

sum()                 // 0
sum(1, 2, 3)          // 6

// ⭐展开运算符 *：把已有数组当作可变参数传入
val arr = intArrayOf(1, 2, 3)
sum(*arr)             // 6，不加 * 会报类型错误
```

**规则**：

- 一个函数**只能有一个** `vararg` 参数。
- 通常放在最后；若后面还有参数，那些参数**必须用具名方式传递**。

```kotlin
fun log(vararg msgs: String, level: Int = 1) { }
log("a", "b", level = 2)          // level 必须具名
```

---

### 6. 函数的三种位置 ⭐

```kotlin
// ① 顶层函数：直接写在文件里，不需要类（⭐Kotlin 特有）
fun formatMoney(value: Long): String = "￥$value"

// ② 成员函数：定义在类中
class UserService {
    fun getUser(id: Long): User? = null
}

// ③ ⭐局部函数：函数里再定义函数，可直接访问外层变量
fun process(list: List<String>) {
    val prefix = ">> "

    fun printItem(item: String) {       // 只在 process 内可见
        println(prefix + item)          // 直接用外层的 prefix，不用传参
    }

    list.forEach { printItem(it) }
}
```

> **局部函数的价值**：把只在当前函数内复用的逻辑就近封装，既不污染类的命名空间，又能共享外层变量。校验逻辑、递归辅助函数常用。

---

### 7. 返回多个值

```kotlin
// ① Pair / Triple（临时使用）
fun minMax(list: List<Int>): Pair<Int, Int> = Pair(list.min(), list.max())
val (min, max) = minMax(list)         // ⭐解构赋值

// ② ⭐业务场景优先用 data class，字段有名字，可读性远好于 first/second
data class PageResult(val list: List<User>, val total: Int)
fun query(): PageResult = ...
val (users, total) = query()
```

> `Pair` 的 `first`/`second` 没有语义，超过两个值或跨方法传递时**一律用 data class**。

---

### 8. 尾递归优化 tailrec（了解）

递归调用是函数最后一步时，加 `tailrec` 让编译器把递归**优化成循环**，避免栈溢出：

```kotlin
tailrec fun factorial(n: Int, acc: Long = 1): Long =
    if (n <= 1) acc else factorial(n - 1, acc * n)     // 递归必须是最后一个动作
```

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐用默认参数替代方法重载 | 一个函数覆盖多种调用方式，维护成本更低 |
| ⭐参数超过 3 个、或有 Boolean 参数时用具名调用 | 杜绝"魔法参数" |
| 公开函数显式声明返回类型 | 便于阅读、稳定 API |
| 供 Java 调用的函数加 `@JvmOverloads` | 否则 Java 侧用不了默认值 |
| 返回多值优先 data class | 不用 `Pair`/`Triple` 传递业务数据 |
| 函数保持单一职责，控制长度 | 复杂逻辑拆成局部函数或私有函数 |
| 布尔参数尽量拆成两个函数或用枚举 | `setVisible(true)` 不如 `show()` / `hide()` 清晰 |

---

### 10. 面试重点

**Q1：Kotlin 的默认参数是怎么实现的？为什么 Java 调不到？**

编译器会为带默认值的函数生成一个**合成方法 `xxx$default`**，通过一个 `int` 位掩码标记哪些参数被省略，在方法内部填充默认值——**不是生成多个重载**。Java 侧看到的只有全参方法和这个签名怪异的合成方法，因此用不了默认值。解决办法是加 `@JvmOverloads`，让编译器额外生成各参数组合的重载方法。

**Q2：默认参数和方法重载相比有什么优势？**

① 代码量少，一个函数替代 N 个重载；② 避免重载之间逻辑重复或不一致；③ 配合具名参数可以任意组合传参，而重载只能覆盖预设的几种组合；④ 新增参数时不用再写一个重载，调用方无需改动。

**Q3：具名参数有什么用？调用 Java 方法能用吗？**

作用是提升可读性（尤其是多个同类型参数或布尔参数）、允许打乱参数顺序、以及跳过中间的默认参数。**调用 Java 方法时不能用**，因为 Java 字节码默认不保留参数名（除非编译时加 `-parameters`），Kotlin 无法确定名称对应关系。

**Q4：`vararg` 在函数内部是什么类型？怎么把数组传进去？**

内部是数组：`vararg n: Int` 对应 `IntArray`，`vararg s: String` 对应 `Array<out String>`。传已有数组要用**展开运算符** `*`，如 `sum(*arr)`，否则会被当成一个数组元素而类型不匹配。

**Q5：Kotlin 的函数参数为什么不能修改？**

Kotlin 的参数一律是 `val`。这样可以避免"修改参数导致方法内外语义不一致"的隐蔽 bug，也让代码更容易推断。需要变化的值应声明局部变量：`var temp = param`。

**Q6：什么是局部函数？有什么使用场景？**

定义在函数内部的函数，作用域仅限外层函数，并且能直接访问外层的局部变量。适合把只在本函数内重复使用的逻辑（参数校验、递归辅助、格式化）就近抽取，既避免污染类的命名空间，也省去反复传参。

**Q7：`tailrec` 有什么用？**

标记尾递归函数，让编译器把递归改写成循环，从而避免深层递归导致的 `StackOverflowError`。前提是**递归调用必须是函数的最后一个操作**，否则编译器会给出警告并按普通递归处理。

---

## 九、Lambda 与高阶函数

> Kotlin 中**函数是一等公民**：可以赋值给变量、当参数传、当返回值返回。这是集合操作、协程、DSL 的共同基础。

### 1. 函数类型 ⭐

函数也有类型，写法是 `(参数类型) -> 返回类型`：

```kotlin
val sum: (Int, Int) -> Int = { a, b -> a + b }
val print: (String) -> Unit = { println(it) }
val supply: () -> String = { "hello" }
val callback: ((Int) -> Unit)? = null      // ⭐可空函数类型，注意括号

sum(1, 2)              // 3
sum.invoke(1, 2)       // 等价写法
callback?.invoke(1)    // 可空时这样调用
```

---

### 2. Lambda 语法 ⭐

```kotlin
val f = { a: Int, b: Int -> a + b }
//      { 参数列表      ->  函数体 }
```

| 规则 | 说明 |
| ---- | ---- |
| ⭐**最后一行是返回值** | 不写 `return`（写了是返回外层函数） |
| ⭐单参数可用 `it` | `list.forEach { println(it) }` |
| 参数类型可推断 | 上下文能确定时不用写 |
| `_` 忽略不用的参数 | `map.forEach { _, v -> println(v) }` |
| 支持解构 | `map.forEach { (k, v) -> }` |

```kotlin
list.filter { it > 0 }                       // it 指代单个参数
map.forEach { (key, value) -> println(key) } // 解构
list.forEachIndexed { _, item -> println(item) }  // 忽略下标
```

---

### 3. 高阶函数 ⭐

**参数或返回值是函数**的函数，就叫高阶函数。

```kotlin
// ① 函数作为参数
fun operate(a: Int, b: Int, op: (Int, Int) -> Int): Int = op(a, b)

operate(1, 2) { x, y -> x + y }        // 3

// ② 函数作为返回值
fun multiplier(factor: Int): (Int) -> Int = { it * factor }

val double = multiplier(2)
double(5)                               // 10
```

**实战价值：封装模板代码**

```kotlin
// 把"计时"这种重复逻辑抽成高阶函数
inline fun <T> measure(tag: String, block: () -> T): T {
    val start = System.currentTimeMillis()
    val result = block()
    println("$tag 耗时 ${System.currentTimeMillis() - start}ms")
    return result
}

val users = measure("查询用户") { userDao.queryAll() }   // ⭐调用处非常干净
```

> 事务、加锁、重试、异常兜底、埋点……这些"前后固定、中间变化"的逻辑，都适合用高阶函数封装。

---

### 4. 尾随 Lambda ⭐

**Lambda 是最后一个参数时，可以移到括号外面**——Kotlin 简洁写法的关键来源：

```kotlin
// 原始写法
list.filter({ it > 0 })

// ⭐移出括号
list.filter() { it > 0 }

// ⭐括号内没有其他参数时，括号也省掉
list.filter { it > 0 }

// 有其他参数时
list.joinToString(separator = "-") { it.name }
```

> 这条规则解释了为什么 Kotlin 代码看起来"像有很多新语法"——`repeat { }`、`run { }`、`launch { }` 其实都只是**普通函数 + 尾随 Lambda**。

---

### 5. 函数引用 `::` ⭐

已有函数可直接当作 Lambda 传入，不必再包一层：

```kotlin
fun isPositive(x: Int) = x > 0

list.filter { isPositive(it) }     // 一般写法
list.filter(::isPositive)          // ⭐函数引用，更简洁

list.map(String::length)           // 成员引用
list.map(::User)                   // ⭐构造函数引用：List<String> → List<User>
val f = user::getName              // 绑定到具体对象的引用
```

---

### 6. Lambda 中的 return ⭐（高频坑）

```kotlin
list.forEach {
    if (it == 3) return          // ⭐直接 return 会退出【外层函数】，不是跳过本次
    println(it)
}

list.forEach {
    if (it == 3) return@forEach  // ⭐标签返回，只结束本次 Lambda，相当于 continue
    println(it)
}
```

- **非局部返回**（直接 `return` 退出外层函数）**只有内联函数才允许**，`forEach` 恰好是 `inline` 的。
- 非内联的高阶函数里写 `return` 会**编译报错**，只能用标签返回。

**匿名函数**：另一种写法，`return` 行为更符合直觉——只返回自身：

```kotlin
list.forEach(fun(item: Int) {
    if (item == 3) return        // 只结束这一次调用
    println(item)
})
```

| | Lambda | 匿名函数 |
| ---- | ---- | ---- |
| 写法 | `{ x -> ... }` | `fun(x: Int) { ... }` |
| 返回值 | 最后一行 | 必须 `return` |
| `return` 含义 | 退出**外层函数**（仅 inline 可用） | 退出**自身** |
| 使用频率 | ⭐绝大多数场景 | 需要多处 return 时 |

---

### 7. SAM 转换 ⭐

调用 **Java 的单抽象方法接口**时，可以直接传 Lambda：

```kotlin
// Java: void setOnClickListener(OnClickListener l)
button.setOnClickListener { view -> doSomething() }     // ⭐自动 SAM 转换

// Java: new Thread(Runnable)
Thread { println("run") }.start()
```

**⚠ Kotlin 自己的接口默认不支持 SAM 转换**，必须声明为 `fun interface`（函数式接口）：

```kotlin
fun interface OnResult {              // ⭐fun 关键字，1.4+
    fun onSuccess(data: String)
}

fun request(callback: OnResult) { }
request { data -> println(data) }     // ✔ 可以传 Lambda 了
```

> 纯 Kotlin 代码中，回调优先直接用**函数类型**（`(String) -> Unit`），比定义接口更简单；`fun interface` 主要用于需要给 Java 调用或需要具名接口的场景。

---

### 8. 闭包 ⭐

Lambda 可以捕获外部变量，**而且能修改**（Java 要求必须是 final 或"事实 final"）：

```kotlin
fun counter(): () -> Int {
    var count = 0                 // ⭐被 Lambda 捕获
    return { ++count }            // 每次调用都能修改
}

val next = counter()
next()   // 1
next()   // 2
```

**原理**：Kotlin 编译器会把被捕获的 `var` 包装进一个 `Ref` 对象（`IntRef` / `ObjectRef`），Lambda 持有的是这个包装对象的引用，从而绕开了"局部变量在栈上、方法结束即销毁"的限制。

> ⚠ 闭包会**延长被捕获对象的生命周期**。Android 中把 Activity 的成员捕获进长生命周期的 Lambda（如静态回调、全局单例）是典型的内存泄漏来源。

---

### 9. 底层实现与性能

```kotlin
list.filter { it > 0 }       // 这个 Lambda 编译后是什么？
```

- Lambda 会被编译成实现了 `Function0` / `Function1` / … 接口的对象（Kotlin 2.x 默认走 JVM 的 `invokedynamic`，运行时才生成）。
- **不捕获任何变量**的 Lambda 会被优化成**单例**复用；**捕获了变量**的每次调用都要创建新对象。
- 因此高频调用的高阶函数存在**对象创建 + 虚方法调用**的开销 —— 这正是 `inline` 关键字存在的原因（见第十章）。

---

### 10. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐Lambda 控制在 3~5 行内 | 过长应抽成具名函数，否则可读性急剧下降 |
| ⭐嵌套 Lambda 显式命名参数 | 多层嵌套时 `it` 会指代混乱，写成 `{ user -> }` |
| 能用函数引用就用 `::` | `filter(::isValid)` 优于 `filter { isValid(it) }` |
| 回调优先用函数类型 | 纯 Kotlin 场景不必定义接口 |
| 自定义高阶函数加 `inline` | 避免运行时对象创建（注意别对大函数滥用） |
| 注意闭包导致的内存泄漏 | 长生命周期对象不要捕获短生命周期引用 |
| Lambda 内避免直接 `return` | 明确使用 `return@label` 表达意图 |

---

### 11. 面试重点

**Q1：什么是高阶函数？举个实际用途。**

参数或返回值为函数的函数。实际用途是把"前后固定、中间变化"的模板逻辑抽出来，例如计时、事务、加锁、重试、异常兜底：`measure("查询") { dao.query() }`。标准库的 `filter`、`map`、`forEach`、`let`、`run` 全都是高阶函数。

**Q2：Lambda 里的 `return` 会返回到哪里？**

会退出**外层函数**（非局部返回），而不是结束本次 Lambda——这是常见 bug 来源。只有**内联函数**的 Lambda 允许这么写（`forEach` 就是 inline 的），非内联的会编译报错。想只结束本次调用要用**标签返回** `return@forEach`。

**Q3：Lambda 和匿名函数有什么区别？**

① 语法不同；② Lambda 的返回值是最后一行表达式，匿名函数必须显式 `return`；③ **`return` 语义不同**：Lambda 的 `return` 退出外层函数，匿名函数的 `return` 只退出自身。日常用 Lambda，需要在多个分支提前返回时用匿名函数更清晰。

**Q4：什么是 SAM 转换？Kotlin 接口为什么不支持？**

单抽象方法接口可以用 Lambda 代替实例，即 SAM 转换。Kotlin 对 **Java 接口**自动支持；但 Kotlin 自己的接口默认不支持，因为 Kotlin 更推荐直接用**函数类型**表达回调，无需接口。确实需要时可用 `fun interface` 声明函数式接口。

**Q5：Kotlin 的闭包能修改捕获的变量，是怎么做到的？**

Java 要求捕获的局部变量必须是 final（值拷贝）；Kotlin 编译器会把被捕获的 `var` 包装成 `Ref` 对象（如 `IntRef`、`ObjectRef`）放到堆上，Lambda 持有该对象引用，因此读写的是同一份数据。代价是多一次对象分配，且会延长被捕获对象的生命周期，可能造成内存泄漏。

**Q6：Lambda 有性能开销吗？**

有。非内联 Lambda 会生成 `FunctionN` 接口的实现对象，捕获了外部变量时每次调用都要新建实例，还有一次虚方法调用；在循环或高频路径上会带来 GC 压力。解决办法是给高阶函数加 `inline`，编译期把函数体展开到调用处，彻底消除对象创建（详见第十章）。

**Q7：`it` 什么时候能用？多层嵌套怎么办？**

Lambda **只有一个参数**且类型可推断时，可以省略参数声明直接用 `it`。多层嵌套时内层的 `it` 会遮蔽外层，极易读错，规范要求显式命名参数，例如 `users.forEach { user -> user.orders.forEach { order -> } }`。

---

## 十、内联函数（inline、noinline、crossinline、reified）

### 1. 为什么需要 inline ⭐

高阶函数虽然好用，但每个 Lambda 都是一个对象，高频调用时有创建开销和虚方法调用开销。

`inline` 的作用：**编译期把函数体和 Lambda 体直接"复制"到调用处**，运行时压根不存在函数调用和 Lambda 对象。

```kotlin
inline fun measure(block: () -> Unit) {
    val start = System.currentTimeMillis()
    block()
    println("耗时 ${System.currentTimeMillis() - start}")
}

// 调用处
measure { doSomething() }
```

编译后调用处等价于：

```kotlin
val start = System.currentTimeMillis()
doSomething()                                   // ⭐Lambda 体被直接展开，零对象创建
println("耗时 ${System.currentTimeMillis() - start}")
```

> 标准库里 `let`、`run`、`apply`、`forEach`、`filter`、`map` 等几乎所有高阶函数都是 `inline` 的——**这就是 Kotlin 函数式写法没有性能负担的原因**。

---

### 2. inline 附带的两个能力 ⭐

**① 非局部返回**：Lambda 里可以直接 `return` 退出外层函数（因为代码本来就展开在外层函数里）：

```kotlin
fun find(list: List<Int>): Int? {
    list.forEach {
        if (it > 10) return it     // ⭐只有 inline 函数的 Lambda 才允许
    }
    return null
}
```

**② `reified` 具体化泛型**：见本章第 5 节。

---

### 3. noinline：不内联某个 Lambda

内联的 Lambda 不是对象，**因此不能被存储、传递给其他函数、或作为返回值**。这时用 `noinline` 单独排除：

```kotlin
inline fun request(
    onSuccess: () -> Unit,              // 内联
    noinline onError: (String) -> Unit  // ⭐不内联，因为要存起来
) {
    callbackList.add(onError)           // 没有 noinline 这里会编译报错
    onSuccess()
}
```

---

### 4. crossinline：内联但禁止非局部返回 ⭐

当 Lambda 不是被直接调用，而是**放到另一个执行上下文中调用**（线程、Runnable、回调）时，非局部返回没有意义（外层函数可能早就返回了），编译器要求用 `crossinline`：

```kotlin
inline fun runOnThread(crossinline block: () -> Unit) {
    Thread {
        block()          // ⭐在另一个 Lambda 内部调用，必须 crossinline
    }.start()
}

runOnThread {
    // return       ✘ 编译错误：不允许非局部返回
    return@runOnThread   // ✔ 只能用标签返回
}
```

**三个关键字对比**：

| 关键字 | 是否内联 | 能否非局部返回 | 使用场景 |
| ---- | :--: | :--: | ---- |
| ⭐`inline` | ✔ | ✔ | 默认，普通高阶函数 |
| `noinline` | ✘ | ✘ | Lambda 需要被存储或传递 |
| `crossinline` | ✔ | ✘ | Lambda 在其他上下文（线程/回调）中执行 |

---

### 5. reified：突破泛型擦除 ⭐

JVM 泛型在运行时被擦除，普通函数拿不到 `T` 的真实类型：

```kotlin
fun <T> parse(json: String): T {
    return Gson().fromJson(json, T::class.java)   // ✘ 编译错误：无法访问 T
}
```

`inline` + `reified` 让类型参数在展开时被**替换成真实类型**：

```kotlin
inline fun <reified T> parse(json: String): T {
    return Gson().fromJson(json, T::class.java)   // ✔
}

val user = parse<User>(json)      // 编译后 T 直接变成 User.class

// 其他用法
inline fun <reified T> Any.isType() = this is T           // ⭐可以做 is 判断
inline fun <reified T> List<*>.filterType() = filterIsInstance<T>()
```

**典型应用**（面试常问"你在项目中怎么用的"）：

```kotlin
// JSON 解析
inline fun <reified T> String.toBean(): T = Gson().fromJson(this, T::class.java)
val user: User = jsonStr.toBean()

// Android 页面跳转
inline fun <reified T : Activity> Context.start() {
    startActivity(Intent(this, T::class.java))
}
context.start<MainActivity>()
```

> ⚠ `reified` **只能用在 `inline` 函数**上，因为它依赖编译期展开时的类型替换。

---

### 6. 什么时候该用 inline

| 场景 | 是否加 inline |
| ---- | ---- |
| ⭐函数**带 Lambda 参数**且函数体较小 | ✔ 推荐 |
| ⭐需要 `reified` 或非局部返回 | ✔ 必须 |
| 函数体很大 | ✘ 字节码膨胀，调用点越多包越大 |
| 没有函数类型参数的普通函数 | ✘ 编译器会警告"收益微乎其微" |
| 递归函数 | ✘ 无法内联 |
| public inline 函数中访问 private 成员 | ✘ 编译报错（展开后调用方无权访问） |

---

### 7. 扩展：内联类 value class（了解）

同名不同物，但也是"消除对象开销"的思路：**编译期擦除包装，运行时就是底层类型**。

```kotlin
@JvmInline
value class UserId(val value: Long)          // ⭐只能有一个属性

fun findUser(id: UserId) { }

findUser(UserId(100L))     // 运行时其实就是传了个 long，没有对象创建
findUser(100L)             // ✘ 编译错误 —— 这就是它的价值：类型安全
```

> 用途：给 `Long`、`String` 这类"裸类型"加上语义和类型检查（订单号、用户 ID、金额），**零运行时开销**。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐自定义高阶函数默认加 `inline` | 小函数收益明显 |
| ⭐大函数体不要 inline | 会造成字节码膨胀，尤其 Android 影响包体和方法数 |
| Lambda 需存储时用 `noinline` | 回调列表、延迟执行场景 |
| Lambda 跨线程/跨上下文执行用 `crossinline` | 避免非局部返回引发的逻辑错误 |
| `reified` 用于封装 JSON 解析、类型判断、页面跳转 | 消除到处写 `T::class.java` 的样板代码 |
| ID/金额等裸类型考虑 `value class` | 类型安全且无额外开销 |

---

### 9. 面试重点

**Q1：`inline` 的作用和原理？**

编译期把**函数体和它的 Lambda 参数体**直接展开到调用处，消除函数调用和 Lambda 对象创建开销。代价是字节码体积增大，所以只适合体积小、带 Lambda 参数的函数。标准库的 `let`、`run`、`forEach` 等都是内联函数。

**Q2：所有函数都加 `inline` 会更快吗？**

不会。① 没有函数类型参数时几乎没有收益，编译器会直接警告；② 函数体大 + 调用点多会导致字节码膨胀，反而影响指令缓存和包体大小；③ 递归函数无法内联。只在"高阶函数 + 函数体小"时使用。

**Q3：`noinline` 和 `crossinline` 分别解决什么问题？**

内联的 Lambda 不是真实对象，无法被存储或传递，需要保存时用 `noinline` 让它退回普通对象；`crossinline` 保持内联，但**禁止非局部返回**，用于 Lambda 被放到线程、Runnable、回调等其他执行上下文中调用的场景——那时候外层函数可能已经返回，非局部返回会破坏控制流。

**Q4：什么是 `reified`？为什么必须配合 `inline`？**

`reified` 让泛型参数在运行时"具体化"，可以使用 `T::class`、`is T`、`as T`。它依赖 `inline` 在编译期把函数展开、并把 `T` 替换成调用处的真实类型来实现，因此**离开 `inline` 就无法工作**。典型用途是封装 JSON 解析、类型过滤、Android 的 `startActivity<T>()`。

**Q5：为什么 public inline 函数不能访问 private 成员？**

因为函数体会被展开到调用方那里，而调用方并没有访问这些 private 成员的权限，展开后就会产生非法访问。要么把成员改成 `internal`/`public`，要么用 `@PublishedApi internal` 标记。

**Q6：`inline` 函数里的 Lambda 为什么能直接 `return`？**

因为 Lambda 体被展开到了外层函数内部，此时的 `return` 在语义上就是外层函数自身的 `return`，即"非局部返回"。非内联的高阶函数没有展开，Lambda 是独立对象，无法影响外层控制流，因此直接 `return` 会编译报错。

**Q7：`value class`（内联类）有什么用？**

用来给基础类型加语义和类型安全。`value class UserId(val value: Long)` 在运行时会被擦除为 `long`，**没有对象创建开销**，但在编译期是独立类型，可以防止把订单号误传给用户 ID 这类 bug。限制是只能有一个属性、不能继承类。

---

## 十一、扩展函数与扩展属性

> 一句话：**不修改源码、不用继承，给任何已有类"加方法"**。Kotlin 用它彻底替代了 Java 满天飞的 `XxxUtils`。

### 1. 扩展函数 ⭐

```kotlin
// 语法：fun 接收者类型.函数名(参数): 返回值 { ... }
fun String.isPhone(): Boolean {
    return this.length == 11 && this.all { it.isDigit() }   // ⭐this 指代调用者
}

"13800138000".isPhone()      // true —— 用起来和 String 自带方法一模一样
```

对比 Java 的写法，差别在调用体验：

```java
StringUtils.isPhone(str);    // Java：工具类 + 静态方法，要先知道工具类叫什么
```

```kotlin
str.isPhone()                // ⭐Kotlin：点出来就有，IDE 自动提示
```

---

### 2. 扩展属性

```kotlin
val String.lastChar: Char
    get() = this[length - 1]           // ⭐必须自己写 getter

var StringBuilder.lastChar: Char
    get() = this[length - 1]
    set(value) { setCharAt(length - 1, value) }

"abc".lastChar        // 'c'
```

**⚠ 扩展属性没有幕后字段（backing field），不能有初始值**：

```kotlin
val String.count: Int = 0     // ✘ 编译错误：扩展属性不能有初始化器
```

因为扩展并没有真的往类里添加字段，只是一组静态方法的语法糖。

---

### 3. 三种常用扩展形式

```kotlin
// ① ⭐可空接收者扩展：内部要自己判空，调用时不需要 ?.
fun String?.orDefault(default: String = ""): String = if (this.isNullOrBlank()) default else this

val name: String? = null
name.orDefault("匿名")          // ⭐直接调用，不用 ?.

// ② 泛型扩展
fun <T> List<T>.secondOrNull(): T? = if (size >= 2) this[1] else null

// ③ 伴生对象扩展（给"静态方法"扩展）
fun User.Companion.fromJson(json: String): User = ...
User.fromJson(json)
```

---

### 4. 实现原理 ⭐（面试核心）

扩展函数**编译后就是静态方法，接收者作为第一个参数**：

```kotlin
// Kotlin
fun String.isPhone(): Boolean = length == 11
```

```java
// 等价的 Java（假设文件名 StringExt.kt）
public static boolean isPhone(String $this) { return $this.length() == 11; }
// Java 侧调用：StringExtKt.isPhone(str)
```

由这一条原理，直接推出三个必考结论：

**① ⭐静态解析：看声明类型，不看运行时类型（没有多态）**

```kotlin
open class Animal
class Dog : Animal()

fun Animal.name() = "动物"
fun Dog.name() = "狗"

val a: Animal = Dog()
println(a.name())        // ⭐"动物"！ 按变量的【声明类型】Animal 解析
```

> 成员函数是运行时动态分派（多态），扩展函数是**编译期静态绑定**。

**② ⭐成员函数优先级高于扩展函数**

```kotlin
class Foo {
    fun show() = "成员函数"
}
fun Foo.show() = "扩展函数"

Foo().show()      // ⭐"成员函数"，扩展函数永远不会被调用（IDE 会给出警告）
```

> 推论：给第三方库写扩展时，如果对方后续版本加了同名成员方法，**你的扩展会静默失效**。

**③ 扩展函数不能访问接收者的 `private` / `protected` 成员**

因为它本质是外部的静态方法，权限和普通调用方一致。

---

### 5. 作用域与导入

```kotlin
// StringExt.kt
package com.example.ext
fun String.isPhone() = ...
```

```kotlin
// 使用处：必须导入
import com.example.ext.isPhone
```

| 定义位置 | 可见范围 |
| ---- | ---- |
| ⭐顶层（推荐） | 导入后全项目可用 |
| 类内部 | 仅该类内部可用（有两个接收者，复杂度高，慎用） |
| 函数内部 | 仅当前函数可用 |

---

### 6. 实战用法 ⭐

标准库本身就是扩展函数的最大用户：`let`、`apply`、`filter`、`map`、`joinToString`、`isNullOrEmpty` **全部是扩展函数**。

业务中最常见的三类：

```kotlin
// ① 替代工具类
fun Long.toMoney(): String = "￥%.2f".format(this / 100.0)
fun String.md5(): String = ...
fun Date.format(pattern: String = "yyyy-MM-dd"): String = ...

order.amount.toMoney()          // ⭐调用链自然，无需记忆工具类名

// ② 给第三方/系统类补能力（Android 尤其常见）
fun Context.toast(msg: String) = Toast.makeText(this, msg, Toast.LENGTH_SHORT).show()
fun View.gone() { visibility = View.GONE }
fun View.visible() { visibility = View.VISIBLE }

// ③ 链式处理，让代码从"嵌套"变"顺序"
val result = rawJson.trim().toBean<User>().takeIf { it.isValid() }?.name.orEmpty()
```

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐按接收者类型分文件 | `StringExt.kt`、`ViewExt.kt`、`DateExt.kt`，别堆在一个 `Utils.kt` |
| ⭐只给"通用能力"写扩展 | 业务逻辑应放在业务类中，不要给 `String` 加 `toOrderStatus()` 这种 |
| 命名遵循标准库风格 | `isXxx` / `toXxx` / `orXxx`，见名知意 |
| 不要滥用可空接收者扩展 | 容易掩盖真正该处理的空值问题 |
| 慎重给第三方类型写扩展 | 对方新增同名成员方法后你的扩展会失效 |
| 扩展函数保持无副作用 | 不要在扩展里改全局状态 |
| 团队统一维护扩展库 | 避免同一功能出现五个不同名字的扩展 |

---

### 8. 面试重点

**Q1：扩展函数的原理是什么？**

编译后是**一个静态方法**，接收者对象作为它的第一个参数传入。所谓"给类加方法"只是编译期的语法糖，类本身没有被修改，字节码里也不存在新增成员。Java 侧调用形式为 `StringExtKt.isPhone(str)`。

**Q2：⭐扩展函数能被重写吗？下面输出什么？**

```kotlin
open class Animal; class Dog : Animal()
fun Animal.name() = "动物"
fun Dog.name() = "狗"
val a: Animal = Dog(); println(a.name())
```

输出 **"动物"**。扩展函数**没有多态**，是**静态解析**的——编译器只看变量的声明类型（`Animal`），不看运行时的实际类型（`Dog`）。这是因为它本质是静态方法，方法调用在编译期就已确定。

**Q3：成员函数和扩展函数同名，调用哪个？**

**永远调用成员函数**，扩展函数被"遮蔽"（IDE 会警告 shadowed）。因此给第三方库写扩展存在风险：对方升级后新增了同名成员方法，你的扩展会静默失效，行为可能改变。

**Q4：扩展函数能访问类的私有成员吗？**

不能。它是类外部的静态方法，可见性规则与普通外部调用一致，只能访问 `public` / `internal`（同模块）成员。这也意味着扩展**不能真正参与类的封装**，只适合基于公开 API 的能力增强。

**Q5：扩展属性为什么不能有初始值？**

因为扩展没有向类中真正添加字段，**不存在幕后字段（backing field）**，无处存储这个值。扩展属性只能通过自定义 `getter`（和 `setter`）由已有状态推导出来。

**Q6：扩展函数相比工具类的优势？**

① 调用方式自然，`str.isPhone()` 优于 `StringUtils.isPhone(str)`；② 有 IDE 自动补全，不需要记住工具类名；③ 支持链式调用，代码由嵌套变顺序；④ 可以直接给第三方类和系统类补能力，无需继承或包装。代价是它没有多态、受成员函数遮蔽，且过度使用会让全局命名空间变脏。

**Q7：可空接收者扩展是怎么回事？**

接收者类型声明为可空（`fun String?.orDefault()`），函数内部的 `this` 可能是 null，需要自己判空。好处是调用方不需要写 `?.`，例如标准库的 `isNullOrEmpty()`、`orEmpty()` 就是这样实现的。滥用会掩盖空值问题，仅适合"空值有明确默认语义"的场景。

---

## 十二、作用域函数（let、run、with、apply、also）

> 五个标准库函数，作用都是"**在对象的上下文里执行一段代码**"。它们全是 `inline` 的，没有性能开销。面试几乎必问，工作中天天用。

### 1. 一张表看懂区别 ⭐

| 函数 | 对象引用 | 返回值 | 是否扩展函数 | 典型用途 |
| ---- | :--: | ---- | :--: | ---- |
| ⭐`let` | `it` | **Lambda 结果** | ✔ | 空安全处理、转换 |
| `run` | `this` | **Lambda 结果** | ✔ | 配置对象并返回计算结果 |
| `with` | `this` | **Lambda 结果** | ✘（普通函数） | 对同一对象多次操作 |
| ⭐`apply` | `this` | **对象自身** | ✔ | 对象初始化配置 |
| `also` | `it` | **对象自身** | ✔ | 附加副作用（日志、校验） |

**记忆口诀**：

- **返回什么**：`apply` / `also` 返回**对象自身**（可继续链式调用），其余三个返回 **Lambda 最后一行**。
- **怎么引用**：`let` / `also` 用 **`it`**（可改名，更清晰），其余用 **`this`**（可省略，写起来更短）。

---

### 2. let：空安全与转换 ⭐

```kotlin
// ① ⭐最高频用法：配合 ?. 做非空处理
user?.let {
    println(it.name)          // 只有非空才执行
}

// ② 转换：返回值和接收者类型不同
val length: Int = name.let { it.length }

// ③ 给链式调用的中间结果起个名字，避免多次求值
list.first().let { first ->
    println("$first - ${first.length}")
}

// ④ 配合 Elvis 做兜底
val result = user?.let { it.name } ?: "未知"
```

---

### 3. apply：对象初始化 ⭐

```kotlin
// ⭐最高频用法：创建对象后连续设置属性
val user = User().apply {
    name = "Tom"          // this 可省略
    age = 20
    city = "北京"
}                          // 返回的还是 User

// Android 中的典型场景
val intent = Intent(this, MainActivity::class.java).apply {
    putExtra("id", 1)
    flags = Intent.FLAG_ACTIVITY_NEW_TASK
}

val paint = Paint().apply {
    color = Color.RED
    isAntiAlias = true
}
```

> `apply` 让"创建 + 配置"成为一个表达式，可以直接赋值给 `val`，替代了 Java 的 Builder 模式和一堆 setter 调用。

---

### 4. run：配置 + 返回结果

```kotlin
// ① 有接收者：像 apply，但返回的是计算结果
val desc = user.run {
    "$name, $age 岁"        // this 可省略
}

// ② ⭐无接收者形式：单纯限定作用域，让临时变量不外泄
val result = run {
    val a = compute1()
    val b = compute2()
    a + b                   // 返回值
}

// ③ 配合 ?. 做空安全 + 返回值
val len = user?.run { name.length } ?: 0
```

---

### 5. with：对同一对象多次操作

```kotlin
// ⚠ 不是扩展函数，对象作为第一个参数传入，因此不能配合 ?. 使用
with(user) {
    println(name)
    println(age)
    println(city)
}

val info = with(StringBuilder()) {
    append("a")
    append("b")
    toString()              // 返回值
}
```

> `with(obj)` 和 `obj.run {}` 功能几乎一样，区别只在写法；**对象可空时只能用 `?.run`**，这是二者最实际的取舍点。

---

### 6. also：链式中插入副作用

```kotlin
// ⭐典型：不打断链式调用，插入日志/校验/埋点
val users = repository.query()
    .also { println("查到 ${it.size} 条") }     // 不影响返回值
    .filter { it.age > 18 }
    .also { log("过滤后 ${it.size} 条") }

// 创建对象时同时注册/记录
val user = User("Tom").also {
    cache.put(it.id, it)
}
```

---

### 7. 如何选择 ⭐

```text
需要返回对象自身（链式调用）？
├─ 是 → 操作的是对象属性（配置）→ apply
│       只是附带做点别的（日志/校验）→ also
└─ 否（要返回计算结果）
        对象可能为空 → ?.let（推荐）/ ?.run
        引用对象多次、类型转换 → let（it 可命名，更清晰）
        对象非空、连续调用多个成员 → with / run
```

**经验法则**：日常 90% 的场景只需要 ⭐`let`（判空）和 ⭐`apply`（初始化）两个。

---

### 8. 同族函数：takeIf / takeUnless ⭐

```kotlin
// takeIf：满足条件返回自身，否则返回 null —— ⭐把 if 变成链式
val validName = name.takeIf { it.isNotBlank() } ?: "默认名"

val user = getUser().takeIf { it.age >= 18 }     // 不满足则为 null
user?.let { grantAccess(it) }

// takeUnless：条件取反
val nonEmpty = text.takeUnless { it.isEmpty() }
```

---

### 9. 常见陷阱 ⚠

**① 嵌套导致 `it` / `this` 指代混乱**

```kotlin
user?.let {
    it.orders.forEach {        // ✘ 内层 it 覆盖了外层，极易读错
        println(it)
    }
}

user?.let { u ->               // ✔ 显式命名
    u.orders.forEach { order -> println(order) }
}
```

**② `apply` 中 `this` 与外部同名变量冲突**

```kotlin
class Presenter {
    var name = "外部"
    fun test(user: User) {
        user.apply {
            name = "Tom"       // ⚠ 改的是 user.name，不是 Presenter.name
            this@Presenter.name = "改外部"   // ⭐要改外部必须用限定 this
        }
    }
}
```

**③ 滥用导致可读性变差**

```kotlin
// ✘ 为了链式而链式，读起来很累
val x = a.let { it.b }.also { log(it) }.run { c }.apply { d() }
```

> 作用域函数是为了**减少临时变量和判空样板**，不是为了把所有代码写成一行。逻辑超过 3~5 行就应该用普通变量和语句。

---

### 10. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐判空统一用 `?.let` | 团队内保持一致，不要 `let`/`run`/`apply` 随机混用 |
| ⭐对象初始化统一用 `apply` | 替代连续 setter |
| 嵌套时必须显式命名参数 | 禁止两层以上的裸 `it` |
| 日志/埋点用 `also` | 保证不改变链式调用的返回值 |
| 作用域函数内的代码控制在 5 行内 | 更长应抽取为函数 |
| 不要为了"看起来 Kotlin"而强行链式 | 可读性优先于简洁 |
| 可空对象不要用 `with` | `with` 不是扩展函数，不支持 `?.` |

---

### 11. 面试重点

**Q1：五个作用域函数的区别？**

从两个维度区分：**返回值**——`apply`/`also` 返回对象自身，`let`/`run`/`with` 返回 Lambda 的最后一行结果；**上下文引用**——`let`/`also` 用 `it`，`run`/`with`/`apply` 用 `this`。此外 `with` 不是扩展函数，对象作为参数传入，因此无法配合 `?.` 使用。

**Q2：`let` 和 `apply` 分别什么时候用？**

`let` 用于**空安全处理和类型转换**：`user?.let { ... }` 只在非空时执行，且返回值可以是别的类型。`apply` 用于**对象初始化配置**：创建对象后连续设置属性并返回对象本身，`User().apply { name = "Tom" }`。这两个覆盖了日常绝大多数场景。

**Q3：`run` 和 `with` 有什么区别？**

功能几乎相同（都用 `this`、都返回 Lambda 结果），区别在于 `run` 是**扩展函数**，可以写成 `obj?.run { }` 支持空安全；`with` 是普通函数，对象作为参数传入，不能配 `?.`。另外 `run` 还有无接收者的形式，用来限定临时变量的作用域。

**Q4：`also` 有什么用？**

在链式调用中插入**副作用**且不改变返回值，典型如打日志、埋点、缓存、校验。因为它返回对象自身，所以可以插在任意两步之间而不打断链条。

**Q5：作用域函数有性能开销吗？**

没有。它们都是 `inline` 函数，编译期会把函数体和 Lambda 展开到调用处，不产生额外对象和函数调用。

**Q6：`takeIf` 有什么用？**

把条件判断变成链式表达式：满足条件返回对象自身，否则返回 `null`，通常接 `?:` 兜底或 `?.let` 继续处理。例如 `name.takeIf { it.isNotBlank() } ?: "默认名"`，比写 `if-else` 更紧凑。`takeUnless` 是它的取反版本。

**Q7：作用域函数嵌套有什么问题？**

内层的 `it` 会遮蔽外层的 `it`，`this` 同理，导致代码难读甚至改错对象。规范要求嵌套时显式命名 Lambda 参数（`let { user -> }`），需要访问外层 `this` 时用限定写法 `this@ClassName`。更根本的做法是控制嵌套层级，超过两层就抽函数。

---

## 十三、集合操作 API 与序列（Sequence）

> Kotlin 标准库提供了上百个集合扩展函数，**基本告别手写 for 循环**。它们全是 `inline` 扩展函数，性能等同手写循环。

### 1. 转换 ⭐

```kotlin
val users = listOf(User("Tom", 20), User("Amy", 17))

users.map { it.name }                 // ⭐[Tom, Amy]  一对一转换，最常用
users.mapIndexed { i, u -> "$i:${u.name}" }
users.mapNotNull { it.email }         // ⭐转换 + 自动过滤 null
listOf(listOf(1, 2), listOf(3)).flatten()          // [1,2,3] 打平
users.flatMap { it.orders }           // ⭐转换 + 打平：所有用户的所有订单

// ⭐List 转 Map（业务中极高频）
users.associateBy { it.id }           // Map<Long, User>   以 id 为键
users.associateWith { it.age }        // Map<User, Int>    以对象为键
users.associate { it.id to it.name }  // Map<Long, String> 自定义键值

listOf(1, 2).zip(listOf("a", "b"))    // [(1,a), (2,b)] 两个集合配对
```

---

### 2. 过滤 ⭐

```kotlin
users.filter { it.age >= 18 }         // ⭐保留满足条件的
users.filterNot { it.age >= 18 }      // 保留不满足的
list.filterNotNull()                  // ⭐List<T?> → List<T>
list.filterIsInstance<String>()       // 按类型过滤

users.take(3)                         // 前 3 个（不足不报错）
users.drop(3)                         // 丢弃前 3 个
users.takeWhile { it.age > 0 }        // ⭐从头取，直到条件不满足就停
users.distinctBy { it.name }          // ⭐按指定字段去重
```

---

### 3. 查找与判断 ⭐

```kotlin
users.find { it.age > 18 }            // ⭐第一个匹配的，没有返回 null（= firstOrNull{}）
users.firstOrNull()                   // 第一个元素，空集合返回 null（first() 会抛异常）
users.lastOrNull()
users.indexOfFirst { it.age > 18 }    // 下标，找不到返回 -1

users.any { it.age > 18 }             // ⭐是否存在满足条件的
users.all { it.age > 18 }             // 是否全部满足
users.none { it.age > 18 }            // 是否全部不满足
users.count { it.age > 18 }           // 满足条件的数量
users.isNotEmpty()
```

> `any` / `all` / `find` 都是**短路**的，找到结果立即返回，不会遍历完整个集合。

---

### 4. 聚合 ⭐

```kotlin
list.sum()                            // 数字集合求和
users.sumOf { it.age }                // ⭐按字段求和（支持 Int/Long/Double/BigDecimal）
users.maxByOrNull { it.age }          // ⭐年龄最大的【那个对象】
users.maxOfOrNull { it.age }          // 最大的【年龄值】
users.minByOrNull { it.age }
list.average()
users.joinToString(", ") { it.name }  // ⭐拼接成字符串

// reduce vs fold ⭐
listOf(1, 2, 3).reduce { acc, i -> acc + i }        // 6，⚠ 空集合会抛异常
listOf(1, 2, 3).fold(100) { acc, i -> acc + i }     // 106，⭐有初始值，空集合安全
users.fold(mutableListOf<String>()) { acc, u -> acc.apply { add(u.name) } }  // 可换类型
```

| | `reduce` | `fold` |
| ---- | ---- | ---- |
| 初始值 | 用第一个元素 | ⭐显式提供 |
| 空集合 | ⚠ 抛异常 | 返回初始值 |
| 结果类型 | 必须与元素同类型 | ⭐可以是任意类型 |

> 优先用 `fold`：更安全、更灵活。

---

### 5. 分组与分块 ⭐

```kotlin
users.groupBy { it.city }             // ⭐Map<String, List<User>> 按城市分组，最常用
users.groupBy({ it.city }, { it.name })  // Map<String, List<String>> 指定值

users.partition { it.age >= 18 }      // ⭐Pair<成年人List, 未成年List> 一次拆两组
val (adults, minors) = users.partition { it.age >= 18 }

list.chunked(3)                       // ⭐[[1,2,3],[4,5,6]] 分批，批量入库/请求常用
list.windowed(3, step = 1)            // 滑动窗口
list.zipWithNext()                    // 相邻两两配对

users.groupingBy { it.city }.eachCount()   // ⭐Map<String, Int> 直接统计各组数量
```

---

### 6. 排序 ⭐

```kotlin
list.sorted()                         // 自然排序（元素需实现 Comparable）
list.sortedDescending()
users.sortedBy { it.age }             // ⭐按字段升序
users.sortedByDescending { it.age }   // 降序
users.reversed()                      // 反转

// ⭐多字段排序
users.sortedWith(compareBy({ it.city }, { it.age }))
users.sortedWith(compareByDescending<User> { it.age }.thenBy { it.name })
```

**⚠ 区分"返回新集合"和"原地修改"**：

| 只读风格（返回新集合） | 可变集合原地修改 |
| ---- | ---- |
| ⭐`sortedBy` | `sortBy` |
| `reversed` | `reverse` |
| `shuffled` | `shuffle` |
| `filter` / `map`（返回新集合） | `removeAll` / `retainAll` / `removeIf` |

> 记忆规律：**带 `ed`/`ing` 的是返回新集合，原形动词是原地修改**。

---

### 7. Sequence：惰性求值 ⭐

**问题**：链式操作的每一步都会生成一个**中间集合**。

```kotlin
list.filter { ... }      // 生成集合1
    .map { ... }         // 生成集合2
    .take(10)            // 生成集合3
```

10 万条数据、3 步操作 = 创建 3 个大集合，全部元素都被处理了一遍。

**Sequence（对标 Java Stream）**：元素**一个一个地流过整条链**，不产生中间集合，且天然短路。

```kotlin
list.asSequence()            // ⭐转成序列
    .filter { it.age > 18 }  // 中间操作：惰性，此刻什么都不做
    .map { it.name }         // 中间操作
    .take(10)
    .toList()                // ⭐末端操作：到这里才真正开始执行
```

**执行方式对比**：

```text
List：   [全部]filter → [全部]map → [全部]take     横向，逐步全量处理
Sequence：元素1 filter→map→take，元素2 filter→map→take …  纵向，逐个走完整条链
```

| 对比项 | 集合（List） | ⭐Sequence |
| ---- | ---- | ---- |
| 求值方式 | 立即（Eager） | 惰性（Lazy） |
| 中间集合 | 每步都创建 | 不创建 |
| 短路能力 | 弱 | ⭐强（`take`/`first` 拿够就停） |
| 小数据量（< 1000） | ⭐更快（无额外开销） | 略慢 |
| 大数据量 + 多步操作 | 慢、内存占用高 | ⭐明显更快 |

**其他构建方式**：

```kotlin
generateSequence(1) { it + 1 }.take(5).toList()      // [1,2,3,4,5] 无限序列
sequence {                                           // 用 yield 自定义
    yield(1)
    yieldAll(listOf(2, 3))
}
```

**选择建议**：

- 数据量小（几百条以内）或只有一两步操作 → **直接用集合**，Sequence 的迭代器开销反而更亏。
- 数据量大、链式步骤多、或有 `first`/`take`/`any` 这类短路操作 → **用 Sequence**。
- ⚠ Sequence **没有并行能力**（Java Stream 有 `parallelStream`），需要并行请用协程或 Java Stream。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐能用标准库就不手写循环 | `filter`/`map`/`groupBy` 表意清晰、不易出错 |
| ⭐链式不超过 3~5 步 | 更长要拆分并命名中间变量，否则难以调试 |
| 聚合优先 `fold` 而非 `reduce` | 避免空集合抛异常 |
| 取最值用 `maxByOrNull` | 空集合安全，`maxBy` 系列会抛异常 |
| 大数据量多步链式用 `asSequence()` | 小数据量不要用，反而更慢 |
| 批量处理用 `chunked` | 分批入库、分批请求接口 |
| 只读集合用 `sortedBy`，可变集合才用 `sortBy` | 分清返回新集合与原地修改 |
| 遍历中不要修改原集合 | 会抛 `ConcurrentModificationException` |

---

### 9. 面试重点

**Q1：`map` 和 `flatMap` 的区别？**

`map` 是一对一转换，`List<User>` → `List<String>`；`flatMap` 是"转换成集合再打平"，`List<User>` → 每个用户的订单列表 → 合并成一个 `List<Order>`。当转换结果本身是集合、且你想要一个扁平结果时用 `flatMap`。

**Q2：`reduce` 和 `fold` 的区别？**

`reduce` 用**第一个元素**作为初始值，空集合会抛异常，且累积结果必须与元素类型相同；`fold` 需要**显式传入初始值**，空集合安全返回初始值，累积结果可以是任意类型。生产代码优先 `fold`。

**Q3：Sequence 和普通集合操作的区别？什么时候用？**

集合操作是**立即求值**，每一步都生成中间集合；Sequence 是**惰性求值**，元素逐个流过整条链，中间操作不执行，直到遇到 `toList`、`sum`、`first` 等末端操作才触发，因此不产生中间集合并天然支持短路。**大数据量 + 多步链式 + 短路操作**时用 Sequence；小数据量下 Sequence 的迭代器开销反而更慢。

**Q4：Sequence 和 Java Stream 有什么区别？**

思路一致（惰性 + 中间/末端操作），差异在于：① Kotlin Sequence **不支持并行**，Java Stream 有 `parallelStream()`；② Sequence 是纯 Kotlin 实现，可用于所有目标平台（JS、Native），Stream 依赖 JVM 8+；③ Sequence 的 API 与集合 API 命名统一，学习成本更低。

**Q5：`associateBy` 和 `groupBy` 有什么区别？**

`groupBy` 返回 `Map<K, List<T>>`，同一个键下的元素**放进列表**；`associateBy` 返回 `Map<K, T>`，同一个键**后者覆盖前者**。按唯一 ID 建索引用 `associateBy`，按分类聚合用 `groupBy`。

**Q6：`sortBy` 和 `sortedBy` 有什么区别？**

`sortedBy` 是只读集合的扩展，返回**排序后的新集合**，原集合不变；`sortBy` 是 `MutableList` 的扩展，**原地排序**，无返回值。同类规律还有 `reversed`/`reverse`、`shuffled`/`shuffle`——带 `ed` 的返回新集合。

**Q7：这些集合函数有性能问题吗？**

单步操作没有：它们都是 `inline` 函数，Lambda 会被展开，等价于手写循环。问题在于**链式调用会产生中间集合**——每一步都是一次完整遍历加一次内存分配。数据量大时改用 `asSequence()`，或合并成一次遍历（如用 `mapNotNull` 替代 `map` + `filterNotNull`）。

---

## 十四、类与构造函数（主构造器、次构造器、init）

### 1. 类的定义与实例化

```kotlin
class User                       // ⭐类体为空时连大括号都不用

class Person {
    var name: String = ""
    fun sayHi() = println("hi")
}

val p = Person()                 // ⭐没有 new 关键字
```

**默认修饰符**（与 Java 的重要差异）：

| 项 | Java 默认 | Kotlin 默认 |
| ---- | ---- | ---- |
| 可见性 | 包私有 | ⭐`public` |
| 可继承性 | 可继承 | ⭐`final`（不可继承，要继承必须加 `open`） |
| 方法可重写性 | 可重写 | ⭐`final` |

---

### 2. 主构造器 ⭐

写在**类名后面**，是 Kotlin 最有特色的语法：

```kotlin
class User(val name: String, var age: Int)     // ⭐一行搞定：构造器 + 两个属性
```

这一行等价于 Java 里十几行的字段声明 + 构造方法 + getter/setter。

**⚠ 加不加 `val`/`var` 完全不同**：

```kotlin
class A(val name: String)     // ⭐name 是【属性】，可以 a.name 访问
class B(name: String)         // name 只是【构造参数】，出了构造过程就没了

A("Tom").name                 // ✔
B("Tom").name                 // ✘ 编译错误
```

**默认参数让构造器更灵活**：

```kotlin
class User(
    val name: String,
    val age: Int = 18,
    val city: String = "北京"
)

User("Tom")
User("Tom", city = "上海")     // ⭐具名参数跳过中间参数
```

**需要修饰符或注解时，`constructor` 关键字不能省**：

```kotlin
class User private constructor(val name: String)     // 私有构造器，配合工厂方法
```

---

### 3. init 初始化块 ⭐

主构造器**不能写代码**，需要执行逻辑就用 `init`：

```kotlin
class User(val name: String, val age: Int) {

    val isAdult: Boolean         // 属性初始化器可以用构造参数

    init {                       // ⭐可以有多个 init 块
        require(name.isNotBlank()) { "姓名不能为空" }    // 参数校验的常见位置
        isAdult = age >= 18
        println("User 初始化完成")
    }
}
```

> 主构造器的参数可以在 **`init` 块**和**属性初始化器**中使用，其他地方（如成员函数）用不了——除非它带了 `val`/`var` 成为属性。

---

### 4. 次构造器

用 `constructor` 定义在类体中，**必须直接或间接委托给主构造器**：

```kotlin
class User(val name: String, val age: Int) {

    constructor(name: String) : this(name, 18) {     // ⭐this(...) 委托主构造器
        println("次构造器执行")
    }

    constructor() : this("匿名") { }                  // 委托给上面的次构造器
}
```

> 若类**没有**主构造器，次构造器则直接（隐式）调用父类构造器。

**⭐ 企业实践：能用默认参数就不写次构造器**

```kotlin
// ✘ Java 思维
class User(val name: String, val age: Int) {
    constructor(name: String) : this(name, 18)
}

// ✔ Kotlin 思维
class User(val name: String, val age: Int = 18)
```

---

### 5. 初始化顺序 ⭐（高频面试题）

```kotlin
open class Base {
    init { println("2. 父类 init") }
}

class Child(val name: String) : Base() {       // 1. 主构造参数
    val a = println("3. 属性 a 初始化")
    init { println("4. 子类 init 块") }
    val b = println("5. 属性 b 初始化")

    constructor() : this("默认") {
        println("6. 次构造器体")
    }
}
```

执行 `Child()` 的输出顺序：

```text
2. 父类 init          ← ⭐父类永远先完成初始化
3. 属性 a 初始化       ← ⭐属性初始化器与 init 块【按声明顺序】交织执行
4. 子类 init 块
5. 属性 b 初始化
6. 次构造器体          ← ⭐次构造器体最后执行
```

**记住三条**：

1. 父类 → 子类。
2. 子类内部：**属性初始化器和 `init` 块按源码书写顺序依次执行**（不是先属性后 init）。
3. 次构造器体在最后。

**⚠ 由此引出的经典坑**：父类构造器中调用可被重写的方法，会执行到子类的实现，而此时子类属性还没初始化：

```kotlin
open class Base {
    init { print() }                       // ⚠ 危险
    open fun print() { }
}
class Child : Base() {
    val name = "Tom"
    override fun print() = println(name)   // ⭐输出 null！此时 name 还没赋值
}
```

> 规范：**构造过程中不要调用 open 方法**。

---

### 6. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐属性尽量在主构造器中声明 | 一行完成声明 + 赋值 + getter/setter |
| ⭐用默认参数替代次构造器 | 次构造器只在"参数含义完全不同"时才写 |
| 参数校验放在 `init` 中 | 用 `require` / `check`，错误信息明确 |
| 构造参数多时换行 + 尾逗号 | 提升可读性，Git diff 也更干净 |
| 构造器中不做耗时操作 | 网络、IO、复杂计算应放到方法里显式调用 |
| ⚠ 构造过程中不调用 `open` 方法 | 子类属性尚未初始化，会拿到 null |
| 需要控制创建过程时用私有构造器 + 工厂方法 | `private constructor` + `companion object` |

---

### 7. 面试重点

**Q1：主构造器和次构造器的区别？**

主构造器写在类名后面，一个类**最多一个**，不能包含代码（逻辑写在 `init` 块里），可以直接用 `val`/`var` 声明属性；次构造器用 `constructor` 写在类体中，可以有多个、可以有函数体，但**必须直接或间接委托主构造器**（`: this(...)`）。Kotlin 中大多数场景用主构造器 + 默认参数即可，很少需要次构造器。

**Q2：主构造器参数加不加 `val` 有什么区别？**

加了 `val`/`var` 就是**类的属性**，会生成字段和 getter（`var` 还有 setter），可以在任何成员中使用；不加就只是**构造参数**，作用域仅限于 `init` 块和属性初始化器，构造完成后即不可访问。

**Q3：⭐类的初始化顺序是怎样的？**

① 主构造器参数绑定；② **父类**初始化完成；③ 子类的**属性初始化器和 `init` 块，按源码书写顺序**依次执行；④ 次构造器函数体。注意第三步是交织执行的，不是"先所有属性再所有 init"。

**Q4：为什么在父类构造器里调用 open 方法很危险？**

因为父类构造执行时，**子类的属性尚未初始化**。此时调用被子类重写的方法，访问到的子类属性会是默认值（`null` 或 `0`），造成难以排查的 bug。规范做法是构造过程中只调用 `private` 或 `final` 方法。

**Q5：`init` 块可以有多个吗？和属性初始化器谁先执行？**

可以有多个。它们与属性初始化器**按书写顺序**混合执行——写在前面的先执行。因此 `init` 块中不能引用它下方声明的属性（编译器会报错或拿到未初始化值）。

**Q6：Kotlin 为什么不需要 `new`？**

`new` 在 Kotlin 中没有必要：构造器调用与函数调用形式统一（`User()`），这样构造器就能像普通函数一样被引用（`::User`）、当作高阶函数参数传递（`list.map(::User)`），语法也更简洁。

**Q7：Kotlin 的类为什么默认是 final？**

遵循《Effective Java》的"**要么为继承而设计并提供文档，要么禁止继承**"。默认 `final` 强制开发者显式用 `open` 声明继承意图，避免了大量非预期继承导致的脆弱基类问题。需要继承时加 `open` 即可（Spring/JPA 等场景可用 `all-open` 编译器插件自动放开）。

---

## 十五、属性、幕后字段与延迟初始化（lateinit、by lazy）

### 1. Kotlin 没有"字段"，只有属性 ⭐

```kotlin
class User {
    var name: String = ""     // ⭐这是【属性】，不是字段
}

user.name = "Tom"             // 看起来像直接访问字段
println(user.name)            // 实际调用的是 setName() / getName()
```

**属性 = 私有字段 + getter +（`var` 才有的）setter**，编译器自动生成，等价于 Java 的：

```java
private String name = "";
public String getName() { return name; }
public void setName(String v) { this.name = v; }
```

> 所以 Kotlin 里**不需要也不应该手写 getter/setter 方法**，直接用属性即可。

---

### 2. 自定义访问器与幕后字段 field ⭐

```kotlin
class User {
    var name: String = ""
        get() = field.trim()                    // ⭐field 就是幕后字段
        set(value) {
            field = value.uppercase()           // ⚠ 必须用 field 赋值
        }
}
```

**⚠ 在访问器里写 `this.name = value` 会无限递归**（又调用了 setter），必须用 `field`。

**幕后字段的生成规则（面试点）**：只有**使用了默认访问器**或**在访问器中引用了 `field`** 时，编译器才会真正生成字段。

```kotlin
// ⭐计算属性：没有幕后字段，每次访问都重新计算，不占内存
val fullName: String
    get() = "$firstName $lastName"

val isAdult: Boolean
    get() = age >= 18
```

---

### 3. 访问器的可见性控制 ⭐

```kotlin
class Counter {
    var count: Int = 0
        private set              // ⭐外部只能读，只有类内部能改

    fun increase() { count++ }
}

counter.count        // ✔ 可读
counter.count = 10   // ✘ 编译错误
```

> 这是**对外只读、对内可写**的标准写法，比 Java 写一个 private 字段 + 只提供 getter 简洁得多。

---

### 4. lateinit：延迟初始化的 var ⭐

用于"**构造时还拿不到值，但后面一定会赋值**"的非空属性：

```kotlin
class MainActivity : Activity() {
    private lateinit var binding: ActivityMainBinding    // ⭐不用写成可空类型

    override fun onCreate(savedInstanceState: Bundle?) {
        binding = ActivityMainBinding.inflate(layoutInflater)
        binding.textView.text = "hi"       // 后面直接用，不需要 ?. 或 !!
    }
}
```

**限制**（面试常问）：

| 限制 | 说明 |
| ---- | ---- |
| 只能用 `var` | `val` 必须在构造时确定 |
| ⭐不能是**可空类型** | 可空的话直接初始化为 null 即可，没必要延迟 |
| ⭐不能是**基本类型** | `Int`/`Long`/`Boolean` 等不支持（没有 null 状态可标记未初始化） |
| 不能有自定义访问器 | 需要幕后字段 |

**未初始化就访问会抛 `UninitializedPropertyAccessException`**，可以先检查：

```kotlin
if (::binding.isInitialized) { }      // ⭐判断是否已初始化
```

---

### 5. by lazy：延迟初始化的 val ⭐

**首次访问时才计算，之后缓存结果**：

```kotlin
class Repository {
    // ⭐重对象、耗时初始化的标准写法
    private val database by lazy {
        println("只会打印一次")
        Room.databaseBuilder(...).build()
    }
}
```

**三种线程安全模式**：

| 模式 | 说明 |
| ---- | ---- |
| ⭐`SYNCHRONIZED`（默认） | 加锁，保证只初始化一次，**线程安全** |
| `PUBLICATION` | 允许多线程同时计算，但只有第一个结果被采用 |
| `NONE` | ⭐无同步，**性能最好**，确定单线程（如 Android 主线程）时使用 |

```kotlin
private val adapter by lazy(LazyThreadSafetyMode.NONE) { MyAdapter() }
```

---

### 6. lateinit vs by lazy ⭐（必考对比）

| 对比项 | `lateinit` | `by lazy` |
| ---- | ---- | ---- |
| 修饰 | 只能 `var` | 只能 `val` |
| 初始化者 | ⭐**外部**手动赋值 | ⭐**自身**的 Lambda |
| 初始化时机 | 任意时刻，可多次赋值 | 首次访问时，只一次 |
| 基本类型 | ✘ 不支持 | ✔ 支持 |
| 可空类型 | ✘ 不支持 | ✔ 支持 |
| 线程安全 | ✘ 不保证 | ⭐默认保证 |
| 能否判断已初始化 | `::x.isInitialized` | 不需要 |
| 典型场景 | 依赖注入、`onCreate` 中赋值的 View/Binding | 单例、重对象、配置、缓存 |

> 一句话选择：**值由外部给 → `lateinit var`；值自己算 → `by lazy val`**。

---

### 7. 与 Java 互操作 ⭐

```kotlin
class User {
    var name: String = ""         // Java 侧：user.getName() / user.setName()

    @JvmField
    var age: Int = 0              // ⭐暴露为公开字段，Java 侧：user.age

    var isVip: Boolean = false    // ⚠ Java 侧是 isVip() / setVip()
}
```

**注解使用目标**（给属性的某个部分加注解）：

```kotlin
class User(
    @get:JsonProperty("user_name")     // ⭐加在 getter 上
    val name: String
)
```

> ⚠ `boolean` 属性命名要注意：`isVip` 生成的 getter 就是 `isVip()`；若写成 `vip`，则是 `getVip()`。序列化框架（Jackson/Gson）对此敏感，团队需统一约定。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐不要手写 `getXxx()` / `setXxx()` 方法 | 直接用属性 |
| ⭐对外只读用 `private set` | 保护内部状态 |
| 派生值用**计算属性**（`get()`） | 不占内存、不会与源数据不一致 |
| 重对象/耗时初始化用 `by lazy` | 提升启动性能 |
| 依赖注入、View 绑定用 `lateinit` | 避免满屏 `?.` 和 `!!` |
| 确定单线程时 `lazy` 用 `NONE` 模式 | 省掉同步开销 |
| ⚠ 不要用 `lateinit` 掩盖"本该可空"的状态 | 该可空就老实用 `?` |
| 访问器里必须用 `field` 赋值 | 用属性名会无限递归 |

---

### 9. 面试重点

**Q1：Kotlin 的属性和 Java 的字段有什么区别？**

Java 的字段就是一块存储；Kotlin 的属性是**字段 + 访问器**的整体抽象，`user.name` 这种写法实际调用的是 getter/setter。好处是可以在不改变调用方代码的前提下，随时给属性加上校验、日志或改成计算属性。

**Q2：什么是幕后字段（`field`）？什么时候会生成？**

`field` 是编译器为属性生成的、只能在访问器内部访问的存储。生成条件：**使用了默认访问器**，或**自定义访问器中引用了 `field`**。如果属性只有 `get() = 表达式`（计算属性），就不会生成字段，也不占内存。

**Q3：在 setter 里写 `this.name = value` 会怎样？**

会**无限递归**导致 `StackOverflowError`，因为这行代码又调用了 setter 自身。访问器内部必须用 `field = value` 直接操作幕后字段。

**Q4：⭐`lateinit` 和 `by lazy` 的区别？**

① `lateinit` 修饰 `var`，由**外部**在任意时刻赋值，可多次赋值；`by lazy` 修饰 `val`，由**自身的 Lambda** 在首次访问时计算一次。② `lateinit` 不支持基本类型和可空类型，`by lazy` 都支持。③ `by lazy` 默认线程安全，`lateinit` 不保证。④ 场景：`lateinit` 用于依赖注入、`onCreate` 中初始化的 View；`by lazy` 用于单例、重对象、配置缓存。

**Q5：`lateinit` 为什么不能用于基本类型？**

它靠"字段是否为 null"来判断是否已初始化，而 `Int`、`Boolean` 这类基本类型没有 null 状态，无法表达"未初始化"。这类场景应给默认值，或改用 `by lazy`、可空类型。

**Q6：`by lazy` 的线程安全是怎么保证的？**

默认模式 `SYNCHRONIZED` 采用双重检查加锁，确保多线程下初始化 Lambda 只执行一次。若明确在单线程使用（如 Android 主线程），可用 `LazyThreadSafetyMode.NONE` 去掉同步开销；`PUBLICATION` 则允许并发计算但只采用第一个结果。

**Q7：怎么实现"外部只读、内部可写"的属性？**

```kotlin
var count: Int = 0
    private set
```

给 setter 单独加 `private` 可见性。对集合属性还可以更进一步：内部保存 `MutableList`，对外暴露 `List` 类型的只读属性。

**Q8：`@JvmField` 有什么用？**

让 Kotlin 属性在字节码中直接暴露为**公开字段**，不生成 getter/setter。用于需要按字段访问的 Java 互操作场景（某些反射框架、序列化库），或对性能极敏感的地方。代价是失去了访问器带来的封装能力。

---

## 十六、继承、接口与抽象类

### 1. 继承：默认禁止 ⭐

Kotlin 的类和方法**默认是 `final`**，必须显式 `open` 才能被继承/重写：

```kotlin
open class Animal(val name: String) {        // ⭐必须加 open
    open fun sound() = "..."                 // ⭐方法也要 open
    fun sleep() = "zzz"                      // 不加 open 的方法不能被重写
}

class Dog(name: String) : Animal(name) {     // ⭐冒号继承，并调用父类构造器
    override fun sound() = "汪"              // ⭐override 是强制的
}
```

| 语法点 | 说明 |
| ---- | ---- |
| `: Animal(name)` | 继承并**调用父类构造器**（括号不能少） |
| `open` | 允许被继承 / 重写 |
| ⭐`override` | 重写时**必须**写（Java 的 `@Override` 只是注解，可省略） |

**没有主构造器时，在次构造器中用 `super`**：

```kotlin
class Cat : Animal {
    constructor(name: String) : super(name)
}
```

---

### 2. 重写的细节 ⭐

```kotlin
open class Base {
    open fun a() { }
    open val x: Int = 1
}

class Child : Base() {
    override fun a() { }              // ⭐override 的成员默认仍是 open 的
    final override val x: Int = 2     // ⭐加 final 才禁止子类继续重写
}
```

**属性重写规则**：

```kotlin
open class A {
    open val name: String = "A"
}

class B : A() {
    override var name: String = "B"   // ⭐val 可以被 var 重写（读能力升级为读写）
}                                     //   反过来 var 不能被 val 重写
```

**在主构造器中直接重写属性**（很常用）：

```kotlin
open class Shape { open val area: Double get() = 0.0 }
class Circle(override val area: Double) : Shape()
```

**调用父类实现**：

```kotlin
override fun sound(): String = super.sound() + "!"
```

---

### 3. 抽象类

```kotlin
abstract class BaseActivity {
    abstract val layoutId: Int              // ⭐抽象属性
    abstract fun initView()                 // 抽象方法，无需写 open

    fun onCreate() {                        // ⭐模板方法：固定流程
        setContentView(layoutId)
        initView()
        initData()
    }

    open fun initData() { }                 // 可选重写的钩子
}
```

- `abstract` 类**不能实例化**，默认就是 `open`。
- 抽象成员默认 `open`，不用再写。
- 抽象类**可以有构造器、状态（字段）和已实现的方法**。

---

### 4. 接口 ⭐

Kotlin 接口比 Java 更强：**可以有默认实现，也可以有属性**。

```kotlin
interface Clickable {
    val tag: String                       // ⭐抽象属性

    val desc: String                      // ⭐可以有默认 getter
        get() = "可点击的 $tag"

    fun click()                           // 抽象方法
    fun showTip() = println("提示")        // ⭐默认实现
}

class Button(override val tag: String) : Clickable {
    override fun click() = println("点击")
}
```

**⚠ 接口不能有状态**（没有幕后字段）：

```kotlin
interface A {
    var count: Int = 0     // ✘ 编译错误：接口属性不能有初始值
}
```

---

### 5. 多实现与冲突解决 ⭐

Kotlin 是**单继承 + 多实现**。两个接口有同名默认实现时，**必须显式覆盖**：

```kotlin
interface A { fun hello() = println("A") }
interface B { fun hello() = println("B") }

class C : A, B {
    override fun hello() {          // ⭐不重写会编译报错
        super<A>.hello()            // ⭐用尖括号指定调用哪个父类型的实现
        super<B>.hello()
    }
}
```

---

### 6. 抽象类 vs 接口 ⭐

| 对比项 | 抽象类 | 接口 |
| ---- | ---- | ---- |
| 数量限制 | ⭐单继承 | ⭐多实现 |
| 构造器 | ✔ 有 | ✘ 没有 |
| 状态（幕后字段） | ⭐✔ 可以有 | ⭐✘ 不能有 |
| 属性 | 可带初始值 | 只能抽象或提供 getter |
| 默认实现 | ✔ | ✔（Kotlin 支持） |
| 成员可见性 | 可 `private`/`protected` | 只能 `public`（默认） |
| 设计语义 | ⭐**是什么**（is-a），共享实现与状态 | ⭐**能做什么**（can-do），定义能力 |

**选择原则**：优先用接口（灵活、可多实现）；需要共享**状态**或**构造逻辑**时才用抽象类。

---

### 7. 与 Java 互操作的注意点 ⭐

Kotlin 接口的默认实现，编译后默认生成一个 `DefaultImpls` 静态内部类来承载方法体，**Java 类实现该接口时拿不到默认实现**（会要求你实现所有方法）。

解决办法：编译时开启 `-Xjvm-default=all`，让它生成 Java 8 的真正 `default` 方法。混合工程中定义供 Java 使用的接口时要留意这一点。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐**优先组合，而非继承** | 继承层级超过 2~3 层就应重新审视设计 |
| ⭐面向接口编程 | 依赖抽象而非具体实现，便于测试与替换 |
| 不打算被继承的类不要加 `open` | 保持默认 `final` 是 Kotlin 的良好实践 |
| 接口只定义能力，不塞业务逻辑 | 默认实现仅用于通用、无状态的逻辑 |
| 抽象类用于模板方法 | 固定流程放父类，可变步骤留抽象方法 |
| ⚠ 构造过程中不要调用 `open` 成员 | 子类属性尚未初始化（见第十四章） |
| Spring/JPA 实体类用 `all-open` 插件 | 避免手动给每个类加 `open` |

---

### 9. 面试重点

**Q1：Kotlin 的类为什么默认 final？和 Java 有什么区别？**

Java 的类默认可继承，容易出现"未按继承设计却被继承"的脆弱基类问题。Kotlin 反过来，默认 `final`，必须显式 `open` 才能继承或重写——继承意图必须被主动声明。此外 Kotlin 的 `override` 是**强制关键字**而非可选注解，避免了误重写和拼写错误。

**Q2：`override` 的方法还能被子类重写吗？**

能。`override` 的成员**默认仍然是 open** 的，会一直向下开放。要在某一层截断，需要显式写 `final override fun xxx()`。

**Q3：Kotlin 接口和 Java 8 接口有什么区别？**

相同点是都支持默认方法。不同点：Kotlin 接口**可以声明属性**（抽象属性或带自定义 getter 的属性），Java 只能有常量；Kotlin 接口的默认实现在字节码中默认走 `DefaultImpls` 静态类，Java 侧实现该接口时无法自动继承默认实现，需要开启 `-Xjvm-default=all` 才生成真正的 `default` 方法。

**Q4：接口能有属性吗？能有状态吗？**

可以声明属性，但**不能有状态**——接口属性没有幕后字段，因此不能赋初始值，只能是抽象的（由实现类提供）或提供一个由其他成员推导出的 `get()`。需要保存状态必须用抽象类。

**Q5：抽象类和接口怎么选？**

需要**共享状态或构造逻辑**、表达"是什么"的 is-a 关系时用抽象类；只需要定义"能做什么"的能力、且希望一个类具备多种能力时用接口。由于 Kotlin 只支持单继承，优先用接口保留扩展空间，抽象类留给模板方法这类确实需要共享实现的场景。

**Q6：多个接口有同名默认方法怎么办？**

编译器强制实现类重写该方法，在其中用 `super<接口名>.方法()` 明确指定调用哪一个，或者给出全新实现。不处理会直接编译报错，不存在 Java 中"菱形继承"的歧义隐患。

**Q7：`val` 属性能被 `var` 重写吗？**

可以。`val` 只要求可读，子类用 `var` 重写相当于**增加了写能力**，不违反里氏替换原则。反过来 `var` 不能被 `val` 重写，因为那会丢掉父类承诺的写能力。

---

## 十七、数据类、密封类与枚举类

> 三种"特殊的类"，是 Kotlin 与 Java 差异最大、面试最爱追问的部分。

## 数据类 data class

### 1. 一行代替几十行 ⭐

```kotlin
data class User(val id: Long, val name: String, var age: Int)
```

编译器**自动生成**五样东西：

| 生成内容 | 作用 |
| ---- | ---- |
| ⭐`equals()` / `hashCode()` | 按属性值比较，而非引用 |
| ⭐`toString()` | `User(id=1, name=Tom, age=20)`，日志友好 |
| ⭐`copy()` | 复制并修改部分属性 |
| ⭐`component1()`…`componentN()` | 支持解构声明 |

```kotlin
val a = User(1, "Tom", 20)
val b = User(1, "Tom", 20)
println(a == b)        // ⭐true（普通类是 false）
println(a)             // User(id=1, name=Tom, age=20)
```

**声明条件**：主构造器至少有一个参数，且**所有参数必须是 `val`/`var`**；数据类不能是 `abstract`、`open`、`sealed`、`inner`。

---

### 2. copy()：不可变对象的更新方式 ⭐

```kotlin
val user = User(1, "Tom", 20)
val older = user.copy(age = 21)      // ⭐只改 age，其余照搬
```

> ⚠ **`copy()` 是浅拷贝**：引用类型属性仍然共享同一个对象。需要深拷贝要自己实现或走序列化。

---

### 3. 解构声明 ⭐

```kotlin
val (id, name, age) = user           // ⭐依次调用 component1/2/3
for ((k, v) in map) { }              // Map.Entry 也有 componentN

val (_, name2) = user                // 用 _ 跳过不需要的
```

**⚠ 解构按【位置】而不是【名字】匹配**：调整主构造器参数顺序会让所有解构处静默出错，是重构时的隐蔽风险。

---

### 4. ⚠ 只有主构造器的属性算数（高频考点）

```kotlin
data class User(val id: Long) {
    var name: String = ""            // ⭐写在类体里！
}

val a = User(1).apply { name = "Tom" }
val b = User(1).apply { name = "Amy" }

println(a == b)        // ⭐true！ name 不参与 equals
println(a)             // User(id=1) —— toString 里也没有 name
a.copy()               // 也不会复制 name
```

> 结论：**希望参与相等性判断的字段，必须写进主构造器**。

---

### 5. 使用注意

| 注意点 | 说明 |
| ---- | ---- |
| ⭐属性尽量用 `val` | 用 `var` 时，对象放入 `HashSet`/`HashMap` 后再修改属性，`hashCode` 变化会导致**再也查不到它** |
| 与 Java `record` 的区别 | `record` 完全不可变且无 `copy`；data class 支持 `var`、`copy`、默认参数，更灵活 |
| JPA 实体慎用 data class | 需要无参构造器和可变性，一般用普通类 + `no-arg` 插件 |
| 大对象的 `equals` 有成本 | 属性很多时逐一比较，热点路径需留意 |

---

## 枚举类 enum

### 6. 基本用法

```kotlin
enum class Status(val code: Int, val desc: String) {     // ⭐可以带属性
    SUCCESS(200, "成功"),
    NOT_FOUND(404, "未找到"),
    ERROR(500, "错误");                                  // ⚠ 有其他成员时这个分号必须写

    fun isOk() = code == 200                             // ⭐可以有方法

    companion object {
        fun of(code: Int) = entries.find { it.code == code }
    }
}
```

**常用 API**：

```kotlin
Status.SUCCESS.name          // "SUCCESS"
Status.SUCCESS.ordinal       // 0，声明顺序
Status.valueOf("SUCCESS")    // 按名字取，找不到抛异常
Status.entries               // ⭐1.9+ 推荐，返回 List，替代会产生数组拷贝的 values()
Status.values()              // 老写法，每次调用都复制一份数组
```

**每个实例可以有自己的实现**：

```kotlin
enum class Op {
    PLUS { override fun apply(a: Int, b: Int) = a + b },
    MINUS { override fun apply(a: Int, b: Int) = a - b };

    abstract fun apply(a: Int, b: Int): Int
}
```

---

## 密封类 sealed class ⭐

### 7. 什么是密封类

**子类种类固定、且编译器全部知道**的类。子类必须与父类在**同一模块、同一包**内（Kotlin 1.5+ 放宽，不再要求同一文件）。

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()     // ⭐每个子类可以携带不同数据
    data class Error(val code: Int, val msg: String) : Result<Nothing>()
    data object Loading : Result<Nothing>()              // 无数据时用 object 单例
}
```

### 8. 核心价值：when 穷尽检查 ⭐

```kotlin
fun handle(result: Result<User>) = when (result) {       // ⭐不需要 else！
    is Result.Success -> show(result.data)               //   分支内自动智能转换
    is Result.Error -> toast(result.msg)
    Result.Loading -> showLoading()
}
```

**将来新增一个子类（比如 `Empty`）时，所有没有处理它的 `when` 都会编译报错**——这是密封类最大的价值：把"漏改分支"从线上 bug 变成编译错误。

> 所以规范要求：**密封类的 `when` 不要写 `else`**，否则就丧失了这个保护。

### 9. 典型应用场景

```kotlin
// ① 网络请求结果（最经典）
sealed interface UiState {                     // ⭐1.5+ 也支持 sealed interface
    data object Loading : UiState
    data class Success(val list: List<User>) : UiState
    data class Error(val msg: String) : UiState
}

// ② 页面/业务事件
sealed class Event {
    data class Click(val id: Int) : Event()
    data object Refresh : Event()
}
```

### 10. sealed vs enum vs abstract ⭐

| 对比项 | `enum` | ⭐`sealed` | `abstract` |
| ---- | ---- | ---- | ---- |
| 限制的是 | **实例**个数固定 | **子类**种类固定 | 无限制 |
| 每个分支能否携带不同数据 | ✘ 结构相同 | ⭐✔ 各自定义属性 | ✔ |
| 能否有多个实例 | ✘ 每个常量唯一 | ⭐✔ `Success` 可以有无数个 | ✔ |
| `when` 穷尽检查 | ✔ | ⭐✔ | ✘ |
| 典型用途 | 状态码、类型常量 | ⭐结果封装、UI 状态、事件 | 通用继承体系 |

> 一句话：**分支只是"标识"用枚举；分支要"带数据"用密封类**。

---

### 11. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐DTO / VO / 实体优先 data class | 自动 `equals`/`toString`/`copy`，日志和比较都省心 |
| ⭐data class 属性尽量 `val` | 避免可变对象作为 Map 键导致的诡异问题 |
| 参与相等性的字段必须放主构造器 | 类体中的属性不参与 `equals`/`copy`/`toString` |
| ⭐网络与 UI 状态用密封类 | `Loading`/`Success`/`Error` 三态，杜绝"忘记处理某状态" |
| ⭐密封类的 `when` 禁止写 `else` | 保留新增子类时的编译期提醒 |
| 枚举遍历用 `entries` | 1.9+ 起替代 `values()`，避免数组拷贝 |
| 枚举承载业务码时补 `of()` 工厂方法 | 统一 code → 枚举的转换入口 |
| 敏感字段注意 `toString` 泄漏 | 密码、token 等要重写 `toString` |

---

### 12. 面试重点

**Q1：⭐data class 自动生成了什么？有什么限制？**

生成 `equals()`、`hashCode()`、`toString()`、`copy()` 和 `componentN()`。限制：主构造器至少一个参数且参数必须是 `val`/`var`；不能是 `abstract`、`open`、`sealed`、`inner`。**最关键的一点是：这些方法只基于主构造器中的属性**，写在类体里的属性不参与。

**Q2：data class 的 `copy()` 是深拷贝吗？**

不是，是**浅拷贝**。只复制引用类型属性的引用，新旧对象仍然共享同一个子对象。需要深拷贝得自己实现，或借助序列化 / 手动递归 `copy`。

**Q3：data class 和 Java 的 `record` 有什么区别？**

`record` 是完全不可变的数据载体，字段全是 final，没有 `copy`；Kotlin 的 data class 支持 `var` 属性、`copy()`、默认参数和具名参数，灵活度更高。两者都自动生成 `equals`/`hashCode`/`toString`。

**Q4：解构声明的原理是什么？有什么坑？**

依赖 `componentN()` 操作符函数，`val (a, b) = obj` 会被编译成 `val a = obj.component1(); val b = obj.component2()`。坑在于**按位置匹配而非按名字**：调整主构造器参数顺序后，所有解构处不会报错但值会错位，重构时要格外小心。

**Q5：⭐密封类的作用是什么？和枚举有什么区别？**

密封类限定了**子类的种类**，编译器掌握全部子类，因此 `when` 可以做**穷尽检查**、省略 `else`，新增子类时所有未覆盖的 `when` 直接编译报错。与枚举的区别：枚举限制的是**实例个数**，每个常量结构相同、只能是单例；密封类的每个子类可以携带**不同的数据**，且可以有多个实例（如 `Success(data)`）。

**Q6：密封类的子类可以定义在哪里？**

Kotlin 1.5 之前必须与父类在**同一文件**；1.5 起放宽为**同一模块、同一包**下的任意文件。这个限制是穷尽检查的前提——编译器必须能看到全部子类。

**Q7：为什么密封类的 `when` 不建议写 `else`？**

写了 `else` 之后，新增子类时 `when` 依然能编译通过，只是悄悄走进 `else` 分支，漏处理的问题会拖到运行时才暴露。不写 `else` 才能享受编译器的穷尽检查——这正是使用密封类的主要动机。

**Q8：data class 用 `var` 属性有什么风险？**

对象被放入 `HashSet` 或作为 `HashMap` 的键之后，如果修改了参与 `hashCode` 的属性，哈希值会改变，导致**再也无法查到该对象**，也无法正常删除，是典型的内存泄漏与逻辑错误来源。因此 data class 属性推荐一律用 `val`，需要变更就用 `copy()` 生成新对象。

**Q9：`entries` 和 `values()` 有什么区别？**

`values()` 每次调用都会**复制一份新数组**（防止外部修改），在循环里频繁调用有性能损耗；`entries`（Kotlin 1.9+）返回一个不可变的 `List`，无需拷贝，还能直接使用集合 API。新代码一律用 `entries`。

---

## 十八、对象声明、伴生对象与单例

> Kotlin 没有 `static`，`object` 这一个关键字承担了**单例、静态成员、匿名内部类**三件事。

### 1. object 的三种用途 ⭐

| 写法 | 名称 | 作用 |
| ---- | ---- | ---- |
| `object Xxx { }` | 对象声明 | ⭐**单例** |
| `companion object { }` | 伴生对象 | ⭐替代 `static` |
| `object : Type { }` | 对象表达式 | ⭐匿名内部类 |

---

### 2. 对象声明：天生的单例 ⭐

```kotlin
object UserManager {
    private val users = mutableListOf<User>()

    var currentUser: User? = null            // ⭐可以有属性

    fun login(user: User) { currentUser = user }
}

// 使用：直接用类名，不需要 getInstance()
UserManager.login(user)
UserManager.currentUser
```

**特点**：

- **不能有构造器**（既然是单例，也就无从传参），但可以有 `init` 块。
- ⭐**线程安全 + 懒加载**：编译成 `static final INSTANCE` 字段并在静态代码块中初始化，由 **JVM 类加载机制**保证只初始化一次；类在首次被访问时才加载，所以也是懒的。
- 可以继承类、实现接口：

```kotlin
object DefaultClickListener : View.OnClickListener {
    override fun onClick(v: View) { }
}
```

**Java 侧调用**：`UserManager.INSTANCE.login(user)`。

---

### 3. 伴生对象：Kotlin 的"静态成员" ⭐

```kotlin
class User private constructor(val name: String) {

    companion object {                        // ⭐一个类最多一个伴生对象
        const val TAG = "User"                // 编译期常量

        fun create(name: String): User = User(name)      // ⭐工厂方法
    }
}

User.TAG              // 像静态成员一样访问
User.create("Tom")
```

**三个关键认知**：

**① 伴生对象是一个真实的对象**，因此可以实现接口、可以有名字、可以被扩展：

```kotlin
class User {
    companion object Factory : JsonParser<User> {         // ⭐能实现接口，static 做不到
        override fun parse(json: String): User = ...
    }
}

fun User.Companion.fromCache(): User = ...                // ⭐还能给它写扩展函数
```

**② 它不是真正的静态成员**。编译后是外部类里一个名为 `Companion` 的静态内部类实例，Java 侧要写 `User.Companion.create("Tom")`。想让 Java 侧写成 `User.create(...)`，需要注解：

| 注解 | 效果 |
| ---- | ---- |
| ⭐`@JvmStatic` | 生成真正的静态方法/属性访问器 |
| `@JvmField` | 生成公开静态字段 |
| `const val` | 直接生成静态常量（仅基本类型和 String） |

**③ 伴生对象的初始化时机**：随外部类的加载而初始化，因此**别在里面放重量级对象**，否则会拖慢首次使用外部类的速度。

---

### 4. 对象表达式：匿名内部类 ⭐

```kotlin
// ① 实现接口（Java 中的匿名内部类）
button.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View) { }
})

// ② 继承类并重写
val listener = object : AnimatorListenerAdapter() {
    override fun onAnimationEnd(animation: Animator) { }
}

// ③ ⭐临时对象，不需要专门定义一个类
val point = object {
    val x = 1
    val y = 2
}
```

**与 Java 匿名内部类的两点差异**：

- ⭐可以**访问并修改**外部的局部变量（Java 要求 final）。
- 每次执行**都会创建新实例**（`object 声明` 是唯一实例，注意区分）。

> 只有一个抽象方法的 Java 接口，直接用 Lambda 更简洁（见第九章 SAM 转换），对象表达式主要用于**多方法接口**或**需要继承类**的场景。

---

### 5. 单例的几种写法 ⭐

**① 无参单例：直接用 `object`（首选）**

```kotlin
object ConfigManager { }
```

**② 带参数的单例（面试常考）**

`object` 不能有构造器，因此需要"伴生对象 + 双重检查锁"：

```kotlin
class Database private constructor(private val context: Context) {

    companion object {
        @Volatile private var instance: Database? = null       // ⭐volatile 保证可见性

        fun getInstance(context: Context): Database =
            instance ?: synchronized(this) {                   // ⭐双重检查
                instance ?: Database(context).also { instance = it }
            }
    }
}
```

**③ 无参但需要延迟初始化的重对象：`by lazy`**

```kotlin
object ApiClient {
    val retrofit: Retrofit by lazy { Retrofit.Builder().build() }   // ⭐首次访问才创建
}
```

---

### 6. 与 Java 静态的对应关系

| Kotlin | 编译后 | Java 侧调用 |
| ---- | ---- | ---- |
| 顶层函数 | `FileNameKt` 的静态方法 | `UtilsKt.format()` |
| `object Xxx` | `Xxx.INSTANCE` 单例 | `Xxx.INSTANCE.foo()` |
| `companion object` 成员 | `Xxx$Companion` 内部类 | `Xxx.Companion.foo()` |
| `@JvmStatic` 成员 | 真正的静态方法 | ⭐`Xxx.foo()` |
| `const val` | 静态常量字段 | `Xxx.TAG` |

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐无参单例一律用 `object` | 不要手写双检锁 |
| ⭐工具函数优先顶层函数，而非 `object` 包一层 | `object Utils { fun a() }` 是 Java 思维 |
| 常量用伴生对象里的 `const val` | 或直接顶层 `const val` |
| 供 Java 调用的成员加 `@JvmStatic` | 否则 Java 侧要写 `.Companion.` |
| ⚠ 单例中不要持有 Context/Activity 引用 | Android 内存泄漏第一大来源，必须用 `applicationContext` |
| 单例的重字段用 `by lazy` | 避免类加载时就创建 |
| ⚠ 单例的可变状态要考虑线程安全 | 用 JUC 集合或加锁 |
| 需要多实例、需要注入时不要用单例 | 单例难以测试和替换 |

---

### 8. 面试重点

**Q1：⭐Kotlin 的 `object` 单例是怎么保证线程安全的？是懒加载吗？**

`object` 编译后是一个 `final` 类，带 `public static final INSTANCE` 字段，在**静态初始化块**中创建。JVM 的类加载机制保证静态初始化只执行一次且线程安全（类似 Java 的静态内部类/饿汉写法）。同时类是**首次被主动使用时才加载**，因此也具备懒加载特性——本质等价于 Java 的"静态内部类单例模式"。

**Q2：`object` 和 `companion object` 有什么区别？**

`object` 是**独立的单例对象**，有自己的名字，可以单独存在；`companion object` 是**依附于某个类**的伴生对象，一个类只能有一个，用来提供该类的"静态成员"和工厂方法，能访问该类的私有构造器。

**Q3：Kotlin 为什么没有 `static`？伴生对象是静态的吗？**

Kotlin 用顶层声明和伴生对象替代 `static`，好处是"静态成员"也是真实对象，**可以实现接口、被扩展、参与多态**，设计上更统一。伴生对象**不是真正的静态**：编译后是外部类中一个名为 `Companion` 的静态内部类实例，成员是该实例的方法。要生成真正的静态成员需加 `@JvmStatic` / `@JvmField` / `const`。

**Q4：怎么实现一个需要传参的单例？**

`object` 不能有构造器，标准做法是私有构造器 + 伴生对象中的**双重检查锁**：`@Volatile` 修饰实例字段，`instance ?: synchronized(this) { instance ?: create().also { instance = it } }`。`@Volatile` 保证多线程下的可见性和禁止指令重排。

**Q5：对象表达式和对象声明的区别？**

`object : Type { }` 是**对象表达式**（匿名内部类），每次执行都会**创建新实例**；`object Name { }` 是**对象声明**，全局**只有一个实例**。前者用于临时实现接口/继承类，后者用于单例。

**Q6：对象表达式相比 Java 匿名内部类有什么优势？**

① 可以**访问并修改**外部的局部变量，Java 要求捕获的变量必须是 final；② 可以同时继承一个类并实现多个接口；③ 配合 Kotlin 的类型推断，语法更简洁。但对单抽象方法接口，直接用 Lambda 更好。

**Q7：`companion object` 里的重对象会有什么问题？**

伴生对象随外部类加载而初始化，如果里面放了重量级对象（数据库、网络客户端、大集合），会拖慢首次使用该类的耗时，Android 上还可能影响启动速度。应改用 `by lazy` 延迟到首次访问时再创建。

**Q8：单例持有 Context 为什么会内存泄漏？**

单例的生命周期与进程一致，如果它持有 Activity 的 Context，该 Activity 销毁后仍被单例引用，无法被 GC 回收，就产生泄漏。正确做法是传入 `applicationContext`，或干脆不在单例中持有 Context。

---

## 十九、可见性修饰符与嵌套类、内部类

### 1. 四种可见性修饰符 ⭐

| 修饰符 | 类成员 | 顶层声明 |
| ---- | ---- | ---- |
| ⭐`public`（**默认**） | 处处可见 | 处处可见 |
| ⭐`private` | 仅**本类**内部 | 仅**本文件**内 |
| `protected` | 本类 + **子类** | ✘ 不可用 |
| ⭐`internal` | 同一**模块**内 | 同一**模块**内 |

```kotlin
// 顶层
private fun helper() { }          // 只有本文件能用
internal const val VERSION = 1    // 本模块可用

class User {
    private val id = 1L           // 只有 User 内部
    protected open fun init() { } // User 和其子类
    internal fun debug() { }      // 本模块
    fun getName() = ""            // ⭐默认 public
}
```

---

### 2. 与 Java 的三个关键差异 ⭐

| 差异 | Java | Kotlin |
| ---- | ---- | ---- |
| 默认可见性 | 包私有（package-private） | ⭐`public` |
| ⭐`protected` 范围 | 子类 **+ 同包** | ⭐**只有子类**（不含同包） |
| 模块级可见性 | 无（只有包级） | ⭐`internal` |

> Kotlin **没有"包私有"**，因为包不是可靠的封装边界（任何人都能声明同名包侵入）。取而代之的是 `internal`。

---

### 3. internal：Kotlin 特有 ⭐

**模块（module）= 一次编译产生的代码集合**，实际就是一个 Gradle module / Maven 模块。

```kotlin
internal class NetworkClient        // 本模块内随便用，其他模块看不见
internal fun parseInternal() { }
```

**典型用途**：多模块工程中，`base`/`network` 等基础模块只对外暴露少量 API，实现细节全部标记为 `internal`，防止上层模块乱用。

**⚠ 面试点**：`internal` 成员在**字节码里其实是 `public`**，因为 JVM 没有"模块可见性"这个概念。Kotlin 通过给方法名做**名称修饰**（name mangling，如 `foo$app_debug`）来阻止 Java 侧误调用——所以 **Java 代码理论上仍能访问 `internal` 成员**，`internal` 只是 Kotlin 编译期的保护。

---

### 4. 构造器与其他细节

```kotlin
class User private constructor(val name: String)     // ⭐私有构造器，配合工厂方法

class Foo internal constructor()                      // 只能在本模块内创建
```

- 局部变量、局部函数**不能**加可见性修饰符（作用域天然受限）。
- 重写成员时，可见性不能比父类更严格。

---

### 5. 嵌套类与内部类 ⭐

**这是与 Java 相反的默认行为，必考。**

```kotlin
class Outer {
    private val value = 10

    // ① ⭐嵌套类（默认）—— 不持有外部类引用，相当于 Java 的 static class
    class Nested {
        fun show() = println("nested")
        // println(value)   ✘ 访问不到外部类成员
    }

    // ② ⭐内部类 —— 加 inner 才持有外部类引用
    inner class Inner {
        fun show() = println(value)              // ✔ 可以访问外部类成员
        fun outer() = this@Outer                 // ⭐用 this@外部类名 引用外部实例
    }
}

Outer.Nested().show()          // ⭐嵌套类：不需要外部实例
Outer().Inner().show()         // 内部类：必须先有外部实例
```

| 对比项 | ⭐嵌套类（默认） | ⭐内部类 `inner` |
| ---- | ---- | ---- |
| 是否持有外部引用 | ✘ | ✔ |
| 能否访问外部成员 | ✘ | ✔ |
| 创建方式 | `Outer.Nested()` | `Outer().Inner()` |
| 对应 Java | `static class` | 普通内部类 |
| 内存泄漏风险 | 无 | ⭐**有** |

> ⭐ **Kotlin 与 Java 的默认值相反**：Java 的内部类默认持有外部引用（要加 `static` 才不持有），Kotlin 默认**不持有**（要加 `inner` 才持有）。这是 Kotlin 有意为之的安全设计——**内存泄漏的默认选项应该是安全的那个**。

---

### 6. 内部类与内存泄漏 ⚠

```kotlin
class MainActivity : Activity() {
    // ⚠ 内部类持有 Activity 引用，Handler 消息队列又持有它，
    //    延迟消息未执行完时 Activity 无法被回收
    inner class MyHandler : Handler() { }
}
```

**规避方式**：改用嵌套类（去掉 `inner`）+ 弱引用，或用协程 + 生命周期感知的作用域（Android 中用 `lifecycleScope`）。

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐**最小可见性原则** | 能 `private` 就不 `internal`，能 `internal` 就不 `public` |
| ⭐多模块工程善用 `internal` | 只暴露必要 API，实现细节全部隐藏 |
| 对外公开的 API 显式写 `public` | 表明这是刻意设计的对外接口（配合 explicit API 模式） |
| 类默认写成嵌套类 | 确实需要访问外部状态时才加 `inner` |
| ⚠ Android 中慎用 `inner class` | 长生命周期对象持有 Activity 会泄漏 |
| 工厂模式用私有构造器 | `private constructor` + 伴生对象 |
| 不要依赖 `internal` 做安全边界 | 它对 Java 侧和反射不设防 |

---

### 8. 面试重点

**Q1：Kotlin 有哪几种可见性修饰符？和 Java 有什么区别？**

`public`（默认）、`private`、`protected`、`internal`。差异有三：① Kotlin 默认是 `public`，Java 默认是包私有；② Kotlin 的 `protected` **只对子类可见，不包含同包**，Java 的 `protected` 包含同包；③ Kotlin 用 **`internal`（模块级）** 取代了 Java 的包私有，因为包不是可靠的封装边界。

**Q2：⭐`internal` 是什么？它安全吗？**

模块内可见——模块指一次编译产生的代码集合（一个 Gradle module）。用于多模块工程中隐藏实现细节。它**不是安全边界**：JVM 没有模块可见性概念，`internal` 成员在字节码中是 `public`，只是名字被做了修饰（如 `foo$app_debug`）来阻止 Java 侧正常调用，通过 Java 或反射依然可以访问。

**Q3：⭐嵌套类和内部类的区别？Kotlin 的默认值为什么和 Java 相反？**

Kotlin 的嵌套类**默认不持有**外部类引用（等价于 Java 的 `static class`），加 `inner` 才持有并能访问外部成员。Java 恰好相反，默认持有、加 `static` 才断开。Kotlin 这样设计是因为持有外部引用是**内存泄漏的主要来源**，默认选项应该是安全的那个，需要引用时再显式声明意图。

**Q4：内部类怎么访问外部类的实例？**

用限定 `this`：`this@Outer`。同名成员冲突时也靠它区分，例如 `this@Outer.name` 与 `this.name`。

**Q5：Kotlin 顶层声明能用 `protected` 吗？`private` 顶层声明的范围是什么？**

不能用 `protected`——顶层没有"子类"这个概念。`private` 的顶层声明**作用域是当前文件**，同一文件内的其他声明可以访问，其他文件不行。这是组织"文件内私有工具函数"的常用方式。

**Q6：为什么 Android 里不建议用 `inner class` 写 Handler？**

`inner class` 持有外部 Activity 的引用，而 Handler 发出的延迟消息会被消息队列持有，进而持有 Handler、持有 Activity。Activity 销毁后仍无法回收，造成内存泄漏。解决方式是改用普通嵌套类 + 弱引用，或直接使用协程配合 `lifecycleScope` 自动取消。

**Q7：什么是最小可见性原则？为什么重要？**

只暴露必须暴露的东西：能 `private` 就不用 `internal`，能 `internal` 就不用 `public`。好处是缩小了 API 面，重构时影响范围可控，也避免上层代码依赖不该依赖的实现细节——尤其在多模块工程和对外提供 SDK 时至关重要。

---

## 二十、委托（类委托与属性委托）

> 委托 = **把活儿交给别人干**。设计模式里要手写一堆转发代码，Kotlin 用一个 `by` 关键字在**语言层面**支持。

### 1. 类委托 ⭐

**场景**：想给一个接口的实现加点功能，但不想（或不能）继承。

```kotlin
interface Repository {
    fun save(user: User)
    fun find(id: Long): User?
    fun delete(id: Long)
}

class DbRepository : Repository { /* 三个方法的真实实现 */ }

// ⭐by inner：所有接口方法自动转发给 inner，一行代替三个转发方法
class LogRepository(private val inner: Repository) : Repository by inner {

    override fun save(user: User) {        // 只重写关心的那个
        println("保存前打日志")
        inner.save(user)
    }
    // find / delete 自动转发，不用写
}
```

这就是**装饰器/代理模式**的零样板实现。接口方法越多，`by` 省下的代码越多。

**⚠ 经典陷阱**：被委托对象内部的自调用**不会**走你的重写：

```kotlin
class Base : Repository {
    override fun save(user: User) { }
    override fun saveAll(list: List<User>) {
        list.forEach { save(it) }        // ⭐这里调用的是 Base 自己的 save
    }
}

class Logged(inner: Repository) : Repository by inner {
    override fun save(user: User) { println("日志") }    // ⚠ saveAll 不会触发它
}
```

> 原因：委托**不是继承**，`Base` 根本不知道自己被委托了，没有多态可言。

---

### 2. 属性委托 ⭐

把属性的 **get / set 逻辑**交给另一个对象：

```kotlin
class User {
    val name: String by lazy { loadName() }        // ⭐by lazy 就是属性委托
    var age: Int by Delegates.observable(0) { _, old, new ->
        println("age: $old -> $new")
    }
}
```

**原理**：编译器把 `by X` 翻译成对 `X` 的 `getValue` / `setValue` 调用。

```kotlin
class UpperCaseDelegate {
    private var value = ""

    // ⭐只读属性需要 getValue
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String = value

    // ⭐可写属性还需要 setValue
    operator fun setValue(thisRef: Any?, property: KProperty<*>, newValue: String) {
        println("属性 ${property.name} 被赋值")     // ⭐能拿到属性名
        value = newValue.uppercase()
    }
}

class Foo { var name: String by UpperCaseDelegate() }

Foo().apply { name = "tom" }.name        // "TOM"
```

也可以直接实现标准接口，更规范：

```kotlin
class Trimmed : ReadWriteProperty<Any?, String> {
    private var value = ""
    override fun getValue(thisRef: Any?, property: KProperty<*>) = value
    override fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        this.value = value.trim()
    }
}
```

---

### 3. 标准库提供的委托 ⭐

```kotlin
// ① lazy：延迟初始化（见第十五章）
val config by lazy { loadConfig() }

// ② ⭐observable：属性变化时回调，做数据监听
var name: String by Delegates.observable("默认") { prop, old, new ->
    println("${prop.name}: $old -> $new")
    notifyUi()
}

// ③ ⭐vetoable：可以【否决】赋值
var age: Int by Delegates.vetoable(0) { _, _, new ->
    new >= 0                       // 返回 false 则本次赋值不生效
}

// ④ notNull：类似 lateinit，但支持基本类型
var count: Int by Delegates.notNull()

// ⑤ ⭐Map 委托：属性值从 Map 中读取（解析 JSON、动态配置时好用）
class User(private val map: Map<String, Any?>) {
    val name: String by map        // 自动取 map["name"]
    val age: Int by map
}
User(mapOf("name" to "Tom", "age" to 20)).name    // "Tom"
```

---

### 4. 实战：SharedPreferences 委托 ⭐（Android 经典）

```kotlin
class PrefDelegate<T>(
    private val key: String,
    private val default: T
) : ReadWriteProperty<Any?, T> {

    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return when (default) {
            is String -> sp.getString(key, default) as T
            is Int -> sp.getInt(key, default) as T
            is Boolean -> sp.getBoolean(key, default) as T
            else -> throw IllegalArgumentException("不支持的类型")
        }
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        sp.edit().apply {
            when (value) {
                is String -> putString(key, value)
                is Int -> putInt(key, value)
                is Boolean -> putBoolean(key, value)
            }
        }.apply()
    }
}

object Settings {
    var token: String by PrefDelegate("token", "")     // ⭐像普通属性一样读写
    var isFirst: Boolean by PrefDelegate("first", true)
}

Settings.token = "abc"        // 自动写入 SharedPreferences
val t = Settings.token        // 自动读取
```

> 同类思路还可用于：数据库字段映射、配置中心、埋点属性、参数校验——**凡是"读写属性时要做点额外的事"，都适合属性委托**。

---

### 5. 编译原理（加分项）

```kotlin
class Foo { val bar by lazy { "x" } }
```

编译后大致是：

```java
private final Lazy bar$delegate = LazyKt.lazy(...);      // ⭐每个委托属性一个字段
public final String getBar() {
    return (String) this.bar$delegate.getValue();        // getter 转发给委托对象
}
```

要点：

- 每个委托属性生成一个 `xxx$delegate` 字段，属性访问器转发给它的 `getValue`/`setValue`。
- 属性的元信息（`KProperty`）会被生成为静态数组，因此**委托属性有轻微的额外开销**（一个对象 + 一次转发），在超高频访问路径上需要留意。

---

### 6. 委托 vs 继承 ⭐

| 对比项 | 继承 | ⭐委托 |
| ---- | ---- | ---- |
| 耦合度 | 高（依赖父类实现细节） | 低（只依赖接口） |
| 灵活性 | 编译期固定 | ⭐运行时可替换被委托对象 |
| 多重能力 | 单继承受限 | ⭐可委托给多个对象 |
| 父类方法内部自调用 | 走子类重写（多态） | ⚠ **不走**重写 |
| 设计建议 | 真正的 is-a 关系 | ⭐**优先使用**（组合优于继承） |

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐优先组合/委托，而非继承 | 尤其是给第三方实现加功能时 |
| ⭐重复的属性读写逻辑抽成委托 | SP、配置、缓存、校验、埋点 |
| 自定义委托实现 `ReadWriteProperty` 接口 | 比裸写 `operator fun` 更规范、可复用 |
| 属性监听用 `observable`，校验用 `vetoable` | 不要在 setter 里堆逻辑 |
| ⚠ 注意类委托不参与多态 | 被委托对象的内部自调用不会走你的重写 |
| 高频访问的属性慎用委托 | 存在一次额外转发和对象开销 |
| 委托类保持无状态或线程安全 | 多个属性可能共用一个委托实例 |

---

### 8. 面试重点

**Q1：Kotlin 的委托分几种？各解决什么问题？**

两种。**类委托**（`class B : A by a`）：把接口方法的实现转发给另一个对象，一行代码实现装饰器/代理模式，免去手写大量转发方法。**属性委托**（`val x by D()`）：把属性的 get/set 逻辑交给委托对象，用来复用"读写属性时要做的额外工作"，如懒加载、监听、持久化。

**Q2：⭐属性委托的原理是什么？**

编译器要求委托对象提供 `operator fun getValue(thisRef, property)`（可写属性还要 `setValue`），然后把属性访问器转发给它。字节码中会为每个委托属性生成一个 `xxx$delegate` 字段和对应的 `KProperty` 元信息，getter/setter 内部调用委托对象的方法。

**Q3：自定义一个属性委托怎么写？**

实现 `ReadOnlyProperty<T, V>` 或 `ReadWriteProperty<T, V>` 接口（或直接写带 `operator` 的 `getValue`/`setValue` 函数）。`thisRef` 是属性所属对象，`property` 是属性的元信息，可以通过 `property.name` 拿到属性名——这正是 SharedPreferences 委托能自动用属性名当 key 的原理。

**Q4：`by lazy` 是什么？它和属性委托的关系？**

`lazy` 是标准库函数，返回一个 `Lazy<T>` 对象，该对象提供了 `getValue` 扩展函数，因此可以作为属性委托使用。即 `by lazy` 本身就是属性委托的一个标准实现，负责"首次访问时计算并缓存"。

**Q5：⭐类委托有什么坑？**

被委托对象**内部的自调用不会走委托类的重写**。因为委托不是继承，被委托对象持有的 `this` 是它自己，完全不知道外面还包了一层，也就没有多态。需要这种效果只能用继承，或者让被委托对象通过接口回调外层。

**Q6：`observable` 和 `vetoable` 有什么区别？**

`observable` 在赋值**之后**回调，用于监听变化、触发 UI 刷新；`vetoable` 在赋值**之前**回调，返回 `Boolean` 决定这次赋值是否生效，用于校验拦截。

**Q7：属性委托有性能开销吗？**

有轻微开销：多一个委托对象、多一次方法转发，`KProperty` 元信息也会额外生成。绝大多数业务场景可以忽略，但在每帧执行或超高频读取的路径上应避免使用，改为普通属性 + 显式方法。

**Q8：委托和继承怎么选？**

优先委托（组合优于继承）：耦合更低、可以在运行时替换被委托对象、还能同时委托给多个对象，不受单继承限制。只有确实是 **is-a 关系**、需要利用多态、或需要复用父类状态时才用继承。

---

## 二十一、泛型与型变（in、out、where）

### 1. 泛型基础

```kotlin
// 泛型类
class Box<T>(val value: T)

// 泛型函数（类型参数写在 fun 后面）
fun <T> listOf(vararg items: T): List<T> = items.toList()

// ⭐泛型约束：上界
fun <T : Comparable<T>> max(a: T, b: T): T = if (a > b) a else b

// ⭐非空约束（默认上界是 Any?，即允许可空）
fun <T : Any> notNull(value: T) { }

// ⭐多个约束用 where
fun <T> process(item: T) where T : Comparable<T>, T : Serializable { }
```

---

### 2. 型变问题：为什么 List\<String\> 不是 List\<Any\> ⭐

先看 Java 数组的历史错误：

```java
Object[] objs = new String[1];    // Java 允许（数组协变）
objs[0] = 1;                      // ⚠ 编译通过，运行时抛 ArrayStoreException
```

**类型安全被破坏了**。所以泛型默认设计成**不变（invariant）**：

```kotlin
val strs: MutableList<String> = mutableListOf()
val anys: MutableList<Any> = strs      // ✘ 编译错误（幸好）
// 否则就能 anys.add(1)，而 strs 里就混进了 Int
```

> 但这样一来，`List<String>` 想传给接收 `List<Any>` 的函数也不行了——**型变就是用来在保证安全的前提下放开这个限制的**。

---

### 3. 协变 out：只读、只出 ⭐

`out T` 表示 **T 只作为输出（返回值）**，此时 `Box<String>` 就可以当作 `Box<Any>` 用：

```kotlin
class Box<out T>(private val value: T) {
    fun get(): T = value            // ✔ T 在返回值位置
    // fun set(v: T) { }            // ✘ 编译错误：T 不能出现在参数位置
}

val box: Box<Any> = Box<String>("abc")     // ⭐协变成立
```

**Kotlin 的 `List` 正是协变的**：

```kotlin
public interface List<out E> : Collection<E>       // ⭐只读，所以可以协变

val list: List<Any> = listOf("a", "b")             // ✔
val mList: MutableList<Any> = mutableListOf("a")   // ✘ MutableList<E> 不变
```

> 记忆：**只读集合协变，可变集合不变**——因为能写入就可能写进错误类型。

---

### 4. 逆变 in：只写、只入 ⭐

`in T` 表示 **T 只作为输入（参数）**，此时 `Box<Any>` 可以当作 `Box<String>` 用：

```kotlin
class Consumer<in T> {
    fun consume(item: T) { }        // ✔ T 在参数位置
    // fun produce(): T             // ✘ 不能作为返回值
}

val consumer: Consumer<String> = Consumer<Any>()    // ⭐逆变成立
```

**直觉解释**：一个"能处理任意 `Any`"的消费者，当然也能处理 `String`。

标准库例子：`Comparable<in T>`、`Comparator<in T>`。

---

### 5. PECS 口诀 ⭐

| 记忆 | 关键字 | 位置 | 含义 |
| ---- | ---- | ---- | ---- |
| **P**roducer | ⭐`out` | 返回值 | **生产者**，只能取出，协变 |
| **C**onsumer | ⭐`in` | 参数 | **消费者**，只能放入，逆变 |

> **Producer-out，Consumer-in**：只往外给数据用 `out`，只往里收数据用 `in`。

```kotlin
fun copy(from: List<out Any>, to: MutableList<in Any>) { }   // 从生产者读，往消费者写
```

---

### 6. 声明处型变 vs 使用处型变 ⭐

**这是 Kotlin 相比 Java 的重要改进。**

```kotlin
// ⭐① 声明处型变：在类定义时就声明好，所有使用点自动生效（Java 没有）
class Box<out T>

// ⭐② 使用处型变（类型投影）：类本身不变，在使用时临时放开
fun copy(from: Array<out Any>, to: Array<Any>) {
    // from 只能读，不能写 —— 等价于 Java 的 Array<? extends Any>
}
```

| | Java | Kotlin |
| ---- | ---- | ---- |
| 声明处型变 | ✘ 不支持 | ⭐✔ `class Box<out T>` |
| 使用处型变 | `? extends T` / `? super T` | `Array<out T>` / `Array<in T>` |
| 结果 | 每个使用点都要写通配符 | ⭐声明一次，处处可用 |

**对应关系**：`out` ≈ `? extends`，`in` ≈ `? super`，`*` ≈ `?`。

---

### 7. 星投影 `*` ⭐

不关心具体类型参数时用 `*`：

```kotlin
fun printSize(list: List<*>) {          // ⭐相当于 Java 的 List<?>
    println(list.size)
    val item: Any? = list[0]            // ⭐读出来是 Any?
    // list.add(...)                    // ✘ 不能写入（不知道具体类型）
}

if (obj is List<*>) { }                 // ⭐类型判断只能这么写（泛型已擦除）
```

---

### 8. 泛型擦除 ⭐

JVM 运行时**没有泛型信息**，`List<String>` 和 `List<Int>` 都是 `List`：

```kotlin
if (list is List<String>) { }     // ✘ 编译错误
if (list is List<*>) { }          // ✔ 只能判断是不是 List

fun <T> create(): T = T()         // ✘ 拿不到 T 的构造器
```

**突破方式：`inline` + `reified`**（见第十章）：

```kotlin
inline fun <reified T> parse(json: String): T = Gson().fromJson(json, T::class.java)
```

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐只读返回类型用 `out` | 对外暴露 `List<out T>` 类型的接口更易用 |
| ⭐遵循 PECS | 只读参数用 `out`，只写参数用 `in` |
| 泛型约束尽量精确 | `<T : Number>` 优于裸 `<T>`，能在编译期拦住错误 |
| 需要非空时写 `<T : Any>` | 否则默认上界是 `Any?`，允许传 null |
| 需要运行时类型用 `inline` + `reified` | 别到处传 `Class<T>` 参数 |
| 不关心类型参数时用 `*` | 不要用 `Any` 蒙混过关 |
| 泛型嵌套超过两层要重新审视设计 | `Map<String, List<Pair<A, B>>>` 应抽成 data class |

---

### 10. 面试重点

**Q1：⭐什么是型变？为什么泛型默认是不变的？**

型变指的是"`Box<String>` 与 `Box<Any>` 之间是否存在子类型关系"。默认**不变**是为了类型安全：如果 `MutableList<String>` 能当作 `MutableList<Any>` 用，就可以往里 `add(1)`，破坏原集合的类型约束。Java 的数组恰恰犯了这个错（数组协变），代价是运行时才抛 `ArrayStoreException`。

**Q2：⭐`in` 和 `out` 的含义和使用规则？**

`out T` 是**协变**，T 只能出现在**输出位置**（返回值），此时 `Box<子类>` 是 `Box<父类>` 的子类型；`in T` 是**逆变**，T 只能出现在**输入位置**（参数），此时 `Box<父类>` 是 `Box<子类>` 的子类型。口诀是 **PECS**：生产者用 `out`，消费者用 `in`。

**Q3：为什么 `List` 是协变的，`MutableList` 不是？**

`List<out E>` 是只读接口，E 只出现在返回值位置，取出来的一定是 E 或其子类，不会有类型安全问题，因此可以协变；`MutableList<E>` 有 `add(E)` 这类写入方法，E 出现在参数位置，如果协变就能写入错误类型，所以必须保持不变。

**Q4：⭐Kotlin 的型变和 Java 的通配符有什么区别？**

Java 只有**使用处型变**（`? extends` / `? super`），每个使用点都要重复写通配符；Kotlin 既支持**声明处型变**（`class Box<out T>`，在类定义时声明一次，所有使用点自动生效），也支持使用处型变（类型投影 `Array<out T>`）。声明处型变让 API 更清晰、调用方更省心，是 Kotlin 泛型的主要改进。

**Q5：星投影 `*` 是什么？**

表示"不关心具体类型参数"，等价于 Java 的 `?`。`List<*>` 读取时元素类型是 `Any?`，且**不能写入**（除了 null），因为编译器不知道实际类型。典型用法是 `if (obj is List<*>)`——由于泛型擦除，这也是唯一能写的类型判断形式。

**Q6：什么是泛型擦除？怎么突破？**

JVM 在运行时会擦除泛型信息，`List<String>` 和 `List<Int>` 都退化为 `List`，因此无法做 `is List<String>` 判断，也无法通过 `T` 创建对象。突破方式是 `inline` + `reified`：编译期把函数展开并把 `T` 替换成真实类型，从而可以使用 `T::class`、`is T`。

**Q7：`<T : Any>` 和 `<T>` 有什么区别？**

泛型参数的**默认上界是 `Any?`**，因此 `<T>` 允许传入可空类型；写成 `<T : Any>` 才限定为非空类型。需要保证泛型参数不为 null 时（比如内部要调用其成员）必须显式写 `<T : Any>`。

**Q8：多个泛型约束怎么写？**

一个约束直接写在尖括号里：`<T : Comparable<T>>`；多个约束用 `where` 子句：`fun <T> f(x: T) where T : Comparable<T>, T : Serializable`。

---

## 二十二、协程基础（挂起函数、CoroutineScope、Dispatchers）

> 协程是 Kotlin 最核心的竞争力，也是**面试占比最高**的部分。本章讲清"是什么、怎么用"，第二十三章讲并发结构与异常，第二十四章讲 Flow。

### 1. 为什么需要协程 ⭐

**线程的问题**：创建成本高（约 1MB 栈内存）、切换有内核态开销、数量受限；一旦阻塞就白白占着资源。

**回调的问题**：异步嵌套异步，形成回调地狱，且错误处理极其分散。

```kotlin
// ✘ 回调地狱
getUser(id) { user ->
    getOrders(user.id) { orders ->
        getDetail(orders[0].id) { detail ->
            runOnUiThread { show(detail) }
        }
    }
}

// ⭐协程：用同步的写法做异步的事
lifecycleScope.launch {
    val user = getUser(id)              // 挂起，不阻塞线程
    val orders = getOrders(user.id)
    val detail = getDetail(orders[0].id)
    show(detail)                        // 自动回到主线程
}
```

---

### 2. 协程是什么 ⭐

**一句话：协程是一段可以"挂起"和"恢复"的计算任务，运行在线程之上。**

| 对比项 | 线程 | ⭐协程 |
| ---- | ---- | ---- |
| 本质 | 操作系统资源 | ⭐**语言层面的编译期变换**，不是系统资源 |
| 创建成本 | 高（约 1MB 栈） | ⭐极低（一个对象），可轻松创建十万个 |
| 调度 | 操作系统内核调度，涉及内核态切换 | ⭐**用户态**由 Kotlin 运行时调度 |
| 阻塞代价 | 占用线程，资源浪费 | ⭐挂起时**释放线程**，线程可去做别的事 |
| 关系 | — | 多个协程可以复用同一个线程 |

> ⚠ **协程不是"更轻的线程"，它本身不产生并发能力**——并发依然靠线程池，协程解决的是"**异步代码的写法**"和"**线程的高效复用**"。

---

### 3. 挂起函数 suspend ⭐

```kotlin
suspend fun getUser(id: Long): User {        // ⭐suspend 标记：这个函数可能会挂起
    delay(1000)                              // 非阻塞地"等待"
    return User(id)
}
```

**两条规则**：

1. `suspend` 函数**只能在协程内或其他 `suspend` 函数中调用**。
2. ⭐**挂起 ≠ 阻塞**：

| | 阻塞（`Thread.sleep`） | ⭐挂起（`delay`） |
| ---- | ---- | ---- |
| 线程状态 | 被占住，什么也干不了 | ⭐**被释放**，去执行其他任务 |
| 恢复方式 | 时间到自动继续 | 由调度器重新安排执行（可能换线程） |
| 代价 | 高 | 极低 |

**挂起的原理（面试高频）**：编译器做 **CPS 变换**（续体传递风格）——

```kotlin
suspend fun getUser(id: Long): User
// ⭐编译后大致等价于：
fun getUser(id: Long, continuation: Continuation<User>): Any
```

1. 每个 `suspend` 函数编译后**多出一个 `Continuation` 参数**（"接下来要做什么"的回调）。
2. 函数体被拆分成一个**状态机**：以每个挂起点为界切分成若干状态，用 `label` 记录执行到哪一步。
3. 挂起时保存状态并返回特殊标记 `COROUTINE_SUSPENDED`，释放线程；恢复时通过 `Continuation` 从上次的 `label` 继续执行。

> 所以协程"同步的写法"本质是**编译器帮你写回调 + 状态机**，运行时并没有魔法。

---

### 4. 三个协程构建器 ⭐

```kotlin
// ① ⭐launch：启动协程，不返回结果，返回 Job（用于取消/等待）
val job = scope.launch {
    doSomething()
}

// ② ⭐async：并发执行并返回结果，返回 Deferred<T>
val deferred = scope.async { getUser() }
val user = deferred.await()          // 挂起等待结果

// ③ runBlocking：⚠ 阻塞当前线程直到协程完成，只用于 main 函数和单元测试
fun main() = runBlocking {
    launch { println("hi") }
}
```

| 构建器 | 返回值 | 是否阻塞线程 | 用途 |
| ---- | ---- | ---- | ---- |
| ⭐`launch` | `Job` | ✘ | "发射后不管"，执行不需要结果的任务 |
| ⭐`async` | `Deferred<T>` | ✘ | 需要返回值，配合 `await()` |
| `runBlocking` | 结果 | ⭐**✔ 阻塞** | ⚠ 仅限 `main()` 和测试，业务代码禁用 |

**async 的核心价值：并行**

```kotlin
// ✘ 串行，总耗时 = 1s + 1s = 2s
val a = getUserA()
val b = getUserB()

// ⭐并行，总耗时 ≈ 1s
val da = async { getUserA() }
val db = async { getUserB() }
val result = da.await() + db.await()      // 两个请求同时发出
```

---

### 5. CoroutineScope：协程的作用域 ⭐

协程必须启动在某个作用域内，**作用域负责统一管理这些协程的生命周期**（尤其是取消）。

```kotlin
// Android 常用的现成作用域
lifecycleScope.launch { }      // ⭐跟随 Activity/Fragment 生命周期，销毁时自动取消
viewModelScope.launch { }      // ⭐跟随 ViewModel，onCleared 时自动取消

// 自定义作用域（后端/工具类常用）
class MyService {
    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.IO)

    fun stop() = scope.cancel()      // ⭐必须在合适的时机取消，否则泄漏
}

// 挂起函数内部创建子作用域
suspend fun loadAll() = coroutineScope {      // ⭐等待内部所有协程完成才返回
    val a = async { loadA() }
    val b = async { loadB() }
    a.await() to b.await()
}
```

**⚠ 不要用 `GlobalScope`**：它的生命周期与整个应用一致，不受结构化并发管理，**页面销毁后协程仍在运行**，是内存泄漏和野任务的主要来源（官方已标记为 `@DelicateCoroutinesApi`）。

---

### 6. Dispatchers：决定协程在哪个线程执行 ⭐

| 调度器 | 线程池 | ⭐使用场景 |
| ---- | ---- | ---- |
| ⭐`Dispatchers.Main` | 主线程/UI 线程 | 更新 UI（Android） |
| ⭐`Dispatchers.IO` | 弹性线程池，默认上限 64 | **阻塞式 IO**：网络、数据库、文件读写 |
| ⭐`Dispatchers.Default` | 线程数 = CPU 核心数 | **CPU 密集**：排序、解析、图片处理、复杂计算 |
| `Dispatchers.Unconfined` | 不限定 | 特殊场景，业务代码几乎不用 |

**`withContext` 切换线程（最常用）**：

```kotlin
lifecycleScope.launch {                     // 默认在 Main
    val data = withContext(Dispatchers.IO) {  // ⭐切到 IO 执行
        api.getUser()
    }                                        // ⭐执行完自动切回 Main
    textView.text = data.name                // 安全更新 UI
}
```

**⭐ 主安全（main-safe）原则**：**切线程的责任在 suspend 函数自己**，而不是调用方。

```kotlin
// ✔ 正确：函数内部保证自己是主安全的
suspend fun getUser(): User = withContext(Dispatchers.IO) {
    api.getUser()
}

// 调用方不需要关心线程
lifecycleScope.launch { val user = getUser() }
```

---

### 7. CoroutineContext：协程的上下文 ⭐

协程上下文是一组元素的集合，用 `+` 组合：

```kotlin
scope.launch(Dispatchers.IO + CoroutineName("loadUser") + handler) { }
```

| 元素 | 作用 |
| ---- | ---- |
| ⭐`Job` | 生命周期与取消控制 |
| ⭐`CoroutineDispatcher` | 在哪个线程执行 |
| `CoroutineName` | 名称，便于调试 |
| ⭐`CoroutineExceptionHandler` | 未捕获异常的兜底处理（见第二十三章） |

> **子协程默认继承父协程的上下文**（`Job` 除外，子协程会新建自己的 Job 并挂到父 Job 下）。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐**禁用 `GlobalScope`** | 用 `lifecycleScope` / `viewModelScope` / 自定义可取消的作用域 |
| ⭐`suspend` 函数必须主安全 | 内部用 `withContext` 切线程，不把切线程的责任丢给调用方 |
| ⭐业务代码禁用 `runBlocking` | 只允许出现在 `main()` 和单元测试中 |
| 网络/数据库用 `IO`，计算用 `Default` | 用错会拖慢线程池 |
| 并行任务用 `async`，注意统一 `await` | 串行 `await` 等于没并行 |
| 自定义作用域必须在销毁时 `cancel()` | 否则协程泄漏 |
| 挂起函数中不要用 `Thread.sleep` | 用 `delay`，否则线程被白白阻塞 |
| 协程命名（`CoroutineName`）便于排查 | 线上日志能定位到具体任务 |

---

### 9. 面试重点

**Q1：⭐协程是什么？和线程有什么区别？**

协程是一段**可挂起和恢复的计算任务**，由 Kotlin 编译器和运行时在**用户态**调度，本身不是操作系统资源。线程创建成本高（约 1MB 栈）、切换涉及内核态；协程只是一个对象，可创建成千上万个，挂起时**释放所在线程**供其他任务使用。协程运行在线程之上，并发能力仍来自线程池——它解决的是**异步代码的写法**和**线程复用效率**。

**Q2：⭐挂起和阻塞的区别？**

阻塞（`Thread.sleep`）会占住线程，线程在这期间什么都做不了；挂起（`delay`）会**保存当前执行状态并释放线程**，线程可以去执行其他协程，等到恢复条件满足时再由调度器安排继续执行（可能是另一个线程）。这就是协程能用少量线程支撑大量并发任务的原因。

**Q3：⭐`suspend` 关键字的原理是什么？**

编译器对挂起函数做 **CPS（续体传递）变换**：① 函数签名多出一个 `Continuation` 参数；② 函数体按挂起点被拆分成**状态机**，用 `label` 记录当前执行到哪一段；③ 挂起时保存局部变量与 `label`、返回 `COROUTINE_SUSPENDED` 标记并释放线程，恢复时通过 `Continuation.resumeWith` 从对应状态继续执行。所谓"同步写异步"本质是编译器自动生成了回调和状态机。

**Q4：`launch`、`async`、`runBlocking` 的区别？**

`launch` 启动协程但不返回结果，返回 `Job` 用于取消和等待；`async` 返回 `Deferred<T>`，通过 `await()` 挂起获取结果，用于**并行执行多个任务**；`runBlocking` 会**阻塞当前线程**直到内部协程完成，只应用于 `main()` 函数和单元测试，业务代码中使用会退化成同步阻塞。

**Q5：⭐几个 Dispatchers 的区别和选择？**

`Main` 是 UI 线程，只用于更新界面；`IO` 是弹性线程池（默认上限 64），用于**阻塞式 IO**（网络、数据库、文件）；`Default` 的线程数等于 CPU 核心数，用于**CPU 密集计算**（排序、解析、图像处理）；`Unconfined` 不限定线程，恢复后在触发恢复的线程执行，业务中基本不用。选错会导致线程池被占满或计算任务抢占 IO 资源。

**Q6：`withContext` 做了什么？**

挂起当前协程，把代码块切换到指定的调度器上执行，执行完毕后**自动切回原来的上下文**继续往下走。它是"切线程"的标准做法，比嵌套 `launch` 更清晰，也保证了顺序执行的语义。

**Q7：⭐为什么不能用 `GlobalScope`？**

`GlobalScope` 的生命周期与整个应用相同，**不受结构化并发管理**：没有父 Job，无法随页面销毁而取消，容易造成协程泄漏、无效的网络请求和 UI 崩溃。应改用与生命周期绑定的作用域（`lifecycleScope`、`viewModelScope`），或自定义作用域并在销毁时显式 `cancel()`。

**Q8：什么是主安全（main-safe）？**

指一个挂起函数无论在哪个线程被调用都是安全的——**切线程的责任由函数自身承担**，内部用 `withContext(Dispatchers.IO)` 包住阻塞操作。这样调用方（尤其是 UI 层）无需关心线程细节，也避免了在主线程误执行耗时操作。

**Q9：协程一定是异步/多线程的吗？**

不一定。协程与线程是**正交**的：`Dispatchers.Main` 上的多个协程都在同一个线程上交替执行，属于并发但非并行。协程提供的是挂起/恢复的能力，真正的多线程并行取决于所用的调度器。

---

## 二十三、结构化并发（Job、取消与异常处理）

### 1. 什么是结构化并发 ⭐

协程之间存在**父子层级**，作用域内启动的协程都是子协程。由此产生三条铁律：

| 规则 | 说明 |
| ---- | ---- |
| ⭐父等子 | 父协程会等待**所有子协程**完成后才算完成 |
| ⭐父取消 → 子全取消 | 取消会自动向下传播，不会有"野协程"残留 |
| ⭐子失败 → 父取消 | 默认行为，一个子协程异常会取消父及所有兄弟（除非用 `SupervisorJob`） |

> 这就是协程相比"随手 new 一个线程"的最大优势：**任务的生命周期是可管理、可追踪、可整体取消的**。

---

### 2. Job：协程的句柄 ⭐

```kotlin
val job = scope.launch { doWork() }

job.start()          // 启动（懒启动时）
job.join()           // ⭐挂起等待完成
job.cancel()         // ⭐取消
job.cancelAndJoin()  // 取消并等待结束
job.isActive         // 是否活跃
job.isCancelled
job.isCompleted
job.children         // 所有子协程

job.invokeOnCompletion { throwable -> }   // 完成/取消时的回调
```

**状态流转**：

```text
New →  Active  →  Completing  →  Completed
         ↓             ↓
      Cancelling  →  Cancelled
```

---

### 3. 取消是"协作式"的 ⭐（高频面试题）

**`cancel()` 不会强行终止代码，它只是打个标记**——协程必须自己配合检查。

```kotlin
// ⚠ 取消无效：纯计算循环不检查取消状态
val job = launch(Dispatchers.Default) {
    var i = 0
    while (i < 1_000_000) { i++ }        // ⭐cancel() 后它还在跑
}
job.cancel()
```

**三种正确的配合方式**：

```kotlin
// ① ⭐检查 isActive
while (isActive) { doWork() }

// ② ⭐ensureActive()：已取消则直接抛 CancellationException
while (i < n) { ensureActive(); compute() }

// ③ yield()：让出执行权，同时检查取消
while (i < n) { yield(); compute() }
```

> ⭐ **所有 kotlinx.coroutines 的挂起函数（`delay`、`withContext`、`await`…）内部都会检查取消**，所以正常的异步代码天然可取消；只有**纯 CPU 循环**需要手动配合。

**取消时的资源清理**：

```kotlin
val job = launch {
    try {
        doWork()
    } finally {
        // ⚠ 协程已取消，这里不能再调用普通挂起函数
        withContext(NonCancellable) {      // ⭐必须包在 NonCancellable 里
            closeResource()
        }
    }
}
```

---

### 4. CancellationException 的特殊性 ⭐

取消是通过抛出 `CancellationException` 实现的，它被视为**协程的正常结束信号**：

- 不会被当作错误向上传播，**不会**触发 `CoroutineExceptionHandler`。
- ⚠ **不要吞掉它**：

```kotlin
// ✘ 危险：catch (e: Exception) 会把 CancellationException 也吃掉，导致取消失效
try {
    delay(1000)
} catch (e: Exception) {
    log(e)
}

// ✔ 正确写法
try {
    delay(1000)
} catch (e: CancellationException) {
    throw e                      // ⭐取消异常必须重新抛出
} catch (e: Exception) {
    log(e)
}
```

---

### 5. 超时控制 ⭐

```kotlin
// ① 超时抛 TimeoutCancellationException
val data = withTimeout(3000) { api.getUser() }

// ② ⭐超时返回 null，业务中更常用
val data = withTimeoutOrNull(3000) { api.getUser() } ?: defaultUser
```

---

### 6. 异常处理 ⭐（协程最难的部分）

**launch 与 async 的异常行为完全不同**：

| | ⭐`launch` | ⭐`async` |
| ---- | ---- | ---- |
| 异常何时抛出 | **立即**向父协程传播 | ⭐**存储在 `Deferred` 中，`await()` 时才抛出** |
| 未处理的后果 | 触发 `CoroutineExceptionHandler`，否则崩溃 | 不 `await` 就可能被静默忽略 |
| 捕获方式 | 内部 try-catch 或 Handler | ⭐在 `await()` 处 try-catch |

```kotlin
// launch：内部捕获
scope.launch {
    try { risky() } catch (e: Exception) { handle(e) }
}

// ⭐async：在 await 处捕获
val deferred = scope.async { risky() }
try { deferred.await() } catch (e: Exception) { handle(e) }
```

**⭐ CoroutineExceptionHandler**（兜底处理）：

```kotlin
val handler = CoroutineExceptionHandler { _, e -> log("协程异常: $e") }
val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main + handler)

scope.launch { throw RuntimeException("boom") }     // ⭐被 handler 捕获
```

⚠ 两个限制：

1. **只对 `launch` 有效**，`async` 的异常始终在 `await()` 处抛出。
2. **只在根协程上生效**——装在子协程上不起作用，因为异常会先传给父协程。

---

### 7. SupervisorJob：让子协程互不影响 ⭐

默认情况下**一个子协程失败会取消所有兄弟**。要隔离，就用监督作业：

```kotlin
// ① 作用域级别
val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main)
scope.launch { throw Exception() }    // ⭐失败
scope.launch { doWork() }             // ⭐不受影响，继续执行

// ② 局部作用域
supervisorScope {
    launch { loadBanner() }           // 广告失败
    launch { loadList() }             // ⭐列表照常加载
}
```

**`coroutineScope` vs `supervisorScope`** ⭐：

| | `coroutineScope` | ⭐`supervisorScope` |
| ---- | ---- | ---- |
| 子协程失败 | 取消其他所有子协程，异常上抛 | ⭐只有失败的那个结束，兄弟继续 |
| 适用场景 | 所有任务**缺一不可**（都要成功才有意义） | ⭐任务**相互独立**（某块加载失败不影响页面其他部分） |

---

### 8. 一个完整示例

```kotlin
class UserViewModel : ViewModel() {

    fun load(id: Long) {
        viewModelScope.launch {                     // ⭐随 ViewModel 自动取消
            try {
                val result = withTimeoutOrNull(5000) {
                    supervisorScope {               // 两个请求互不影响
                        val user = async { repo.getUser(id) }
                        val orders = async { repo.getOrders(id) }
                        user.await() to orders.await()
                    }
                } ?: return@launch showTimeout()

                showData(result)
            } catch (e: CancellationException) {
                throw e                             // ⭐取消异常必须重抛
            } catch (e: Exception) {
                showError(e.message)
            }
        }
    }
}
```

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐所有协程必须归属于可取消的作用域 | 页面/服务销毁时统一 `cancel()` |
| ⭐CPU 密集循环中检查 `isActive` / `ensureActive()` | 否则取消不生效 |
| ⭐`catch (e: Exception)` 时重抛 `CancellationException` | 否则会破坏取消机制 |
| `finally` 中的挂起调用包 `NonCancellable` | 保证资源正常释放 |
| 独立任务用 `supervisorScope`，关联任务用 `coroutineScope` | 按业务语义选择 |
| 根作用域配 `CoroutineExceptionHandler` 兜底 | 避免未捕获异常导致崩溃 |
| ⭐`async` 必须 `await`，否则异常被吞 | 或明确使用 `SupervisorJob` 并处理 |
| 网络请求统一加 `withTimeoutOrNull` | 避免任务永久挂起 |

---

### 10. 面试重点

**Q1：⭐什么是结构化并发？有什么好处？**

协程之间存在父子层级，子协程的生命周期受父作用域约束：父协程等待所有子协程完成；父被取消时所有子协程自动取消；子协程失败默认会取消父及兄弟。好处是**不会产生失控的野任务**，资源可以随页面/服务生命周期整体回收，错误也有明确的传播路径。

**Q2：⭐协程的取消为什么是"协作式"的？**

`cancel()` 只是把 Job 状态置为取消并抛出 `CancellationException`，它**无法强行中断正在执行的代码**（强行中断会导致状态不一致，Java 的 `Thread.stop` 就是因此被废弃）。协程必须主动配合：调用任意 kotlinx 挂起函数（内部会检查），或在计算循环中检查 `isActive`、调用 `ensureActive()` / `yield()`。

**Q3：一个纯计算的 while 循环，`cancel()` 后还会继续跑吗？**

会。因为循环中没有任何挂起点，也没有检查取消标记。必须改成 `while (isActive)`，或在循环体里调用 `ensureActive()`/`yield()`。这是协程取消最经典的面试题。

**Q4：⭐`launch` 和 `async` 的异常处理有什么不同？**

`launch` 的异常会**立即向父协程传播**，未处理时触发 `CoroutineExceptionHandler` 或导致崩溃；`async` 的异常被**保存在 `Deferred` 中，直到调用 `await()` 才抛出**，如果从不 `await`，异常可能被静默忽略。所以 `async` 要在 `await()` 处 try-catch。

**Q5：⭐`CoroutineExceptionHandler` 在什么情况下不生效？**

① 用在 `async` 上——`async` 的异常只在 `await()` 处抛出；② 装在**非根协程**上——异常会先传播给父协程，只有根协程（作用域的直接子协程）上的 handler 才会被调用；③ 对 `CancellationException` 无效，它被视为正常取消。

**Q6：⭐`coroutineScope` 和 `supervisorScope` 的区别？**

`coroutineScope` 中任一子协程失败会**取消其他所有子协程**并向上抛出异常，适合"所有任务必须都成功"的场景；`supervisorScope` 使用监督作业，**子协程之间互相独立**，一个失败不影响兄弟，适合首页多模块加载这类"某块失败不影响其他"的场景。

**Q7：`SupervisorJob` 有什么用？**

改变异常的传播方向：普通 `Job` 下子协程失败会向上取消父和所有兄弟；`SupervisorJob` 下失败**只影响自身**。常用于长期存在的作用域（如 `viewModelScope` 内部就用了 `SupervisorJob`），保证一次失败不会让整个作用域失效。

**Q8：为什么不能捕获 `CancellationException`？**

它是协程取消的**正常控制信号**而非错误。捕获后不重新抛出，协程会以为自己没被取消而继续执行，导致取消失效、资源无法释放。规范写法是在 `catch (e: Exception)` 之前先 `catch (e: CancellationException) { throw e }`。

**Q9：协程取消后，`finally` 里还能调用挂起函数吗？**

不能直接调用——协程已处于取消状态，任何挂起函数会立即抛出 `CancellationException`。需要在 `finally` 中做挂起清理（如关闭连接、写日志）时，必须包一层 `withContext(NonCancellable) { }`。

**Q10：`withTimeout` 和 `withTimeoutOrNull` 的区别？**

`withTimeout` 超时会抛出 `TimeoutCancellationException`（属于 `CancellationException` 的子类），需要 try-catch；`withTimeoutOrNull` 超时返回 `null`，配合 `?:` 给默认值，业务代码中更常用也更安全。

---

## 二十四、Flow 与冷热流（StateFlow、SharedFlow）

### 1. 为什么需要 Flow ⭐

`suspend` 函数只能返回**一个**值；需要**连续多个异步值**（列表分页、搜索联想、实时状态、数据库监听）时就要用 Flow。

| 场景 | 方案 |
| ---- | ---- |
| 单个值，同步 | 普通函数 |
| 单个值，异步 | ⭐`suspend fun` |
| 多个值，同步 | `Sequence`（阻塞） |
| ⭐多个值，异步 | ⭐**`Flow`** |

```kotlin
fun getUsers(): Flow<User> = flow {
    repeat(3) {
        delay(1000)              // ⭐可以挂起
        emit(User(it))           // ⭐发射一个值
    }
}

lifecycleScope.launch {
    getUsers().collect { user -> println(user) }     // ⭐collect 是挂起函数
}
```

---

### 2. 冷流：不 collect 就不执行 ⭐

```kotlin
val flow = flow { println("开始"); emit(1) }     // ⭐此刻什么都没发生

flow.collect { }     // 打印"开始"
flow.collect { }     // ⭐再打印一次 —— 每个收集者都会独立执行一遍
```

**冷流的三个特性**：惰性（有人收集才执行）、每个收集者**独立执行**、收集者取消则生产也停止。

**创建方式**：

```kotlin
flow { emit(1) }                          // ⭐通用构建器
flowOf(1, 2, 3)                           // 固定值
listOf(1, 2).asFlow()                     // 集合转换

// ⭐callbackFlow：把回调式 API 转成 Flow（实战高频）
fun locationFlow(): Flow<Location> = callbackFlow {
    val listener = object : LocationListener {
        override fun onChanged(loc: Location) { trySend(loc) }
    }
    manager.register(listener)
    awaitClose { manager.unregister(listener) }      // ⭐必须写，用于释放资源
}
```

---

### 3. 常用操作符 ⭐

```kotlin
flow
    // —— 中间操作（惰性，不触发执行）——
    .map { it * 2 }
    .filter { it > 0 }
    .onEach { log(it) }                    // ⭐副作用，不改变数据
    .take(10)
    .distinctUntilChanged()                // ⭐去掉连续重复值
    .debounce(300)                         // ⭐防抖：搜索联想标配
    .flatMapLatest { query -> search(query) }   // ⭐新值到来就取消上一个请求
    .flowOn(Dispatchers.IO)                // ⭐指定【上游】的执行线程
    .catch { e -> emit(default) }          // ⭐捕获【上游】异常
    .onCompletion { log("结束") }
    .retry(3)                              // ⭐失败重试
    // —— 末端操作（触发执行）——
    .collect { }                           // 收集
```

**组合多个 Flow**：

```kotlin
flowA.zip(flowB) { a, b -> a + b }        // 一一配对，等齐才发
flowA.combine(flowB) { a, b -> a + b }    // ⭐任一变化就重新合并（表单校验常用）
```

**⭐ `flowOn` 与上下文保持**：

```kotlin
flow {
    // ✘ 禁止在 flow{} 内部用 withContext 切线程（违反上下文保持原则，会抛异常）
    emit(api.getUser())
}
.flowOn(Dispatchers.IO)        // ⭐正确做法：只影响它【上游】的操作
.collect { }                   // collect 运行在调用方的上下文（如 Main）
```

---

### 4. 异常与背压

```kotlin
// ⭐catch 只捕获【上游】异常，捕获不到 collect 块里的异常
flow.catch { e -> emit(fallback) }
    .collect { risky() }        // ⚠ 这里的异常要用 try-catch

// 背压处理
flow.buffer()          // ⭐生产与消费并发，用缓冲区解耦
    .conflate()        // ⭐只保留最新值，丢弃处理不过来的中间值
flow.collectLatest { } // ⭐新值到来时取消上一次未完成的处理
```

---

### 5. 热流：StateFlow 与 SharedFlow ⭐

**热流不依赖收集者，自己就在发射**，且**多个收集者共享同一数据源**。

**① StateFlow —— 状态持有者**

```kotlin
class UserViewModel : ViewModel() {
    private val _state = MutableStateFlow<UiState>(UiState.Loading)   // ⭐必须有初始值
    val state: StateFlow<UiState> = _state.asStateFlow()              // ⭐对外只读

    fun load() = viewModelScope.launch {
        _state.value = UiState.Loading
        _state.value = try { UiState.Success(repo.get()) }
                       catch (e: Exception) { UiState.Error(e.message) }
    }
}

// 收集（Android 标准写法）
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {    // ⭐后台时自动停止收集
        viewModel.state.collect { render(it) }
    }
}
```

特性：**必须有初始值**、`value` 可直接读、**自动去重**（值相等不发射）、**粘性**（新收集者立即收到当前值）。

**② SharedFlow —— 事件广播**

```kotlin
private val _events = MutableSharedFlow<Event>()      // ⭐默认 replay = 0
val events = _events.asSharedFlow()

suspend fun toast(msg: String) = _events.emit(Event.Toast(msg))
```

用于**一次性事件**（Toast、导航、弹窗）——这类事件不应该在旋转屏幕后重放，因此不能用 StateFlow。

**③ 冷流转热流**：

```kotlin
val state = repo.userFlow()
    .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), initial)   // ⭐→ StateFlow
    // WhileSubscribed(5000)：无人订阅 5 秒后停止上游，避免后台白跑
```

---

### 6. 三张对比表 ⭐

**冷流 vs 热流**：

| 对比项 | ⭐冷流 `Flow` | ⭐热流 `StateFlow`/`SharedFlow` |
| ---- | ---- | ---- |
| 何时开始发射 | 有人 `collect` 时 | ⭐独立于收集者，一直存在 |
| 多个收集者 | 各自独立执行一份 | ⭐**共享**同一份数据 |
| 是否持有值 | 否 | `StateFlow` 持有 |
| 典型用途 | 一次性请求、数据库查询 | ⭐UI 状态、事件总线 |

**StateFlow vs SharedFlow**：

| 对比项 | ⭐`StateFlow` | ⭐`SharedFlow` |
| ---- | ---- | ---- |
| 初始值 | **必须有** | 无 |
| 重放 | 固定重放最新 1 个（粘性） | ⭐`replay` 可配置（默认 0） |
| 去重 | ⭐自动（值相等不发射） | 不去重 |
| 读取当前值 | `.value` | 不支持 |
| 用途 | ⭐**状态**（UI State） | ⭐**事件**（Toast、导航） |

**StateFlow vs LiveData（Android 面试常问）**：

| 对比项 | `LiveData` | ⭐`StateFlow` |
| ---- | ---- | ---- |
| 归属 | Android 架构组件 | ⭐纯 Kotlin，平台无关，可用于后端 |
| 生命周期感知 | ⭐内置 | 需配合 `repeatOnLifecycle` |
| 初始值 | 可空 | 必须有 |
| 操作符 | 少（`map`/`switchMap`） | ⭐完整的 Flow 操作符 |
| 线程 | 只能主线程更新（`postValue` 除外） | 任意线程 |

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐UI 状态用 `StateFlow`，一次性事件用 `SharedFlow` | 用错会导致事件重放（旋屏后又弹一次 Toast） |
| ⭐对外暴露只读类型 | `asStateFlow()` / `asSharedFlow()`，禁止暴露 `Mutable` 版本 |
| ⭐切线程用 `flowOn`，禁止在 `flow{}` 内 `withContext` | 违反上下文保持会直接抛异常 |
| Android 收集用 `repeatOnLifecycle` | 避免页面不可见时仍在刷新 UI |
| 冷流转热流用 `stateIn` + `WhileSubscribed(5000)` | 无人订阅时自动停止上游 |
| 搜索类输入配 `debounce` + `flatMapLatest` | 防抖 + 自动取消过期请求 |
| ⚠ `StateFlow` 的去重要注意 | 相等的新值不会发射；用 data class 时若字段没变则不会刷新 |
| `callbackFlow` 必须写 `awaitClose` | 否则回调无法反注册，造成泄漏 |

---

### 8. 面试重点

**Q1：⭐Flow 是什么？和 suspend 函数、Sequence 有什么区别？**

Flow 是**异步的数据流**，可以按时间先后发射多个值。`suspend` 函数只能返回一个值；`Sequence` 虽然能产生多个值但是**同步阻塞**的，不能在其中调用挂起函数。Flow 兼具"多个值"和"异步非阻塞"两个特性。

**Q2：⭐什么是冷流？Flow 的冷特性体现在哪？**

冷流指**没有收集者时不执行**，每次 `collect` 都会**独立执行一遍**完整的生产逻辑。比如两个地方收集同一个网络请求 Flow，会发出两次请求。热流则相反：独立于收集者存在，多个收集者共享同一份数据。

**Q3：⭐`flowOn` 的作用？为什么不能在 `flow{}` 里用 `withContext`？**

`flowOn` 指定**它上游**操作的执行调度器，下游（包括 `collect`）不受影响，`collect` 始终运行在调用方的上下文中。Flow 有**上下文保持原则**：发射必须发生在收集者的上下文中，直接在 `flow{}` 里 `withContext` 切线程会破坏这一约束，运行时直接抛异常，因此必须用 `flowOn`。

**Q4：⭐StateFlow 和 SharedFlow 的区别？分别用在什么场景？**

`StateFlow` 是**状态持有者**：必须有初始值、通过 `.value` 读取、自动重放最新值给新订阅者（粘性）、并且**值相等时不发射**（自动去重）。`SharedFlow` 是**事件广播**：无初始值、`replay` 可配置（默认 0，不重放）、不去重。所以 **UI 状态用 StateFlow，一次性事件（Toast、导航）用 SharedFlow**——后者避免旋屏后事件被重复消费。

**Q5：⭐StateFlow 和 LiveData 怎么选？**

`LiveData` 是 Android 组件，内置生命周期感知，但操作符匮乏、只能在主线程更新；`StateFlow` 是纯 Kotlin 实现，平台无关（后端也能用）、拥有完整的 Flow 操作符和线程调度能力，但需要配合 `repeatOnLifecycle` 才能实现生命周期感知。新项目推荐 `StateFlow`。

**Q6：`catch` 能捕获所有异常吗？**

不能。`catch` 只捕获**上游**（它之前的操作符和数据源）抛出的异常，`collect` 代码块内部的异常需要用 try-catch 自行处理。此外 `catch` 中还可以 `emit` 一个兜底值让流继续，或配合 `retry`/`retryWhen` 做重试。

**Q7：Flow 怎么处理背压？**

`buffer()` 让生产与消费并发执行，用缓冲区解耦；`conflate()` 只保留最新值、丢弃处理不过来的中间值；`collectLatest {}` 在新值到来时**取消上一次未完成的处理**。此外 Flow 本身基于挂起，默认就是"生产者等消费者"的天然背压。

**Q8：`debounce` + `flatMapLatest` 组合是干什么的？**

搜索联想的标准写法：`debounce(300)` 让用户停止输入 300ms 后才发起请求，避免每敲一个字符就请求一次；`flatMapLatest` 保证新的关键词到来时**自动取消上一个未完成的请求**，防止旧结果覆盖新结果。

**Q9：`stateIn` 中 `WhileSubscribed(5000)` 是什么意思？**

冷流转热流时的共享策略：有订阅者时启动上游，**最后一个订阅者消失 5 秒后停止**上游。5 秒的缓冲是为了应对屏幕旋转等短暂重建，既避免重新请求，也避免页面退到后台后上游仍在空跑。

**Q10：为什么 StateFlow 的值明明"更新了"，UI 却没刷新？**

因为 `StateFlow` 会用 `equals` **去重**：新值与旧值相等就不发射。常见于 data class 里改了一个可变属性但对象仍相等，或者复用了同一个对象实例。解决办法是保证每次都用 `copy()` 生成**内容不同的新对象**，避免持有可变状态。

---

## 二十五、Channel 与并发安全（Mutex、原子类）

### 1. Channel：协程间的通信管道 ⭐

可以理解为**挂起版的 `BlockingQueue`**：满了/空了不阻塞线程，而是挂起协程。

```kotlin
val channel = Channel<Int>()

launch {                                  // 生产者
    repeat(3) { channel.send(it) }        // ⭐挂起式发送
    channel.close()                       // ⭐关闭，通知消费者结束
}

launch {                                  // 消费者
    for (item in channel) println(item)   // ⭐close 后循环自动结束
}
```

**四种容量策略**：

| 类型 | 行为 |
| ---- | ---- |
| ⭐`RENDEZVOUS`（默认，容量 0） | `send` 挂起直到有人 `receive`，完全同步交接 |
| `BUFFERED` / 指定容量 | 缓冲区满了才挂起 |
| `UNLIMITED` | 永不挂起（⚠ 注意内存） |
| ⭐`CONFLATED` | 只保留最新值，旧值被丢弃 |

**`produce` 构建器**（生产者的简写）：

```kotlin
fun CoroutineScope.produceNumbers() = produce {
    var i = 0
    while (true) { send(i++); delay(100) }
}

produceNumbers().consumeEach { println(it) }
```

---

### 2. Channel vs Flow ⭐（高频对比）

| 对比项 | ⭐`Channel` | ⭐`Flow` |
| ---- | ---- | ---- |
| 冷热 | **热**（创建即存在） | 冷（collect 才执行） |
| 元素分配 | ⭐**每个元素只被一个消费者拿到**（扇出/分摊） | ⭐每个收集者都收到**全部**元素 |
| 定位 | 协程间**通信**（通道） | 数据**流**处理 |
| 操作符 | 很少 | ⭐丰富 |
| 典型场景 | 任务分发、工作队列、多消费者竞争 | 数据加载、状态、事件流 |

> **选择原则**：单纯"多个值的数据流"用 **Flow**；需要"多个协程之间投递任务、互相通信"才用 **Channel**。业务代码里 Flow 用得远多于 Channel。

---

### 3. 并发安全问题依然存在 ⚠

**协程并不自动解决线程安全**——只要调度器是多线程的，共享可变状态就会出问题：

```kotlin
var counter = 0

runBlocking {
    withContext(Dispatchers.Default) {
        repeat(100_000) {
            launch { counter++ }        // ⚠ 结果通常小于 100000
        }
    }
}
```

`counter++` 不是原子操作（读-改-写三步），多线程下会丢失更新。

---

### 4. 五种解决方案 ⭐

**① 限制在单线程（最推荐：不共享就没有竞争）**

```kotlin
val singleThread = Dispatchers.Default.limitedParallelism(1)   // ⭐并行度限制为 1

withContext(singleThread) { counter++ }
```

**② 原子类（简单计数首选）**

```kotlin
val counter = AtomicInteger(0)
counter.incrementAndGet()
```

**③ ⭐Mutex：协程的锁**

```kotlin
val mutex = Mutex()
var counter = 0

mutex.withLock {          // ⭐挂起而不是阻塞线程
    counter++
}
```

**④ 并发集合（JUC）**

```kotlin
val map = ConcurrentHashMap<String, User>()
```

**⑤ 干脆不共享可变状态**

用不可变数据 + `StateFlow`/`Channel` 串行处理，把状态收敛到单个协程内部维护——这是协程编程推荐的思路。

---

### 5. Mutex vs synchronized ⭐

| 对比项 | `synchronized` | ⭐`Mutex` |
| ---- | ---- | ---- |
| 阻塞方式 | ⭐**阻塞线程** | ⭐**挂起协程**，线程可去干别的 |
| 能否在其中调用挂起函数 | ⭐**不能** | ⭐能 |
| 可重入 | ✔ 可重入 | ⚠ **不可重入**（重复加锁会死锁） |
| 适用 | 普通多线程代码 | ⭐协程代码 |

> ⚠ **协程代码里不要用 `synchronized` 包住挂起调用**：一是编译就通不过（挂起函数不能在同步块中调用），二是即便包住普通代码也会白白阻塞线程，违背协程初衷。

---

### 6. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐优先"不共享"，而非"加锁" | 状态收敛到单个协程/单线程调度器内维护 |
| 数据流用 Flow，任务分发才用 Channel | 不要用 Channel 模拟数据流 |
| ⭐协程内加锁一律用 `Mutex`，禁用 `synchronized` | 避免阻塞线程 |
| ⚠ `Mutex` 不可重入 | 加锁范围内不要再次调用会加同一把锁的函数 |
| 简单计数用原子类 | 比加锁更轻量 |
| 锁的粒度尽量小 | `withLock` 内只放必要的临界区 |
| `Channel` 使用完必须 `close()` | 否则消费者的 `for` 循环永不结束 |
| ⚠ `UNLIMITED` 容量要评估内存 | 生产快于消费时会无限堆积 |

---

### 7. 面试重点

**Q1：Channel 是什么？和 BlockingQueue 有什么区别？**

Channel 是协程间通信的管道，API 类似阻塞队列，但在缓冲区满或空时**挂起协程而不是阻塞线程**，因此不会浪费线程资源。它是热的，创建后即存在，元素通过 `send`/`receive` 传递，用完需要 `close()`。

**Q2：⭐Channel 和 Flow 的区别？该怎么选？**

Channel 是**热流**且元素**只会被一个消费者接收**（多个消费者是"分摊"关系），定位是协程间通信；Flow 默认是**冷流**，每个收集者都会收到全部元素，定位是数据流处理，且操作符丰富。数据流场景一律用 Flow，只有需要多协程之间投递任务、竞争消费时才用 Channel。

**Q3：Channel 有哪几种容量策略？**

`RENDEZVOUS`（默认，容量 0，发送方挂起等待接收方，同步交接）、指定容量的 `BUFFERED`（缓冲满才挂起）、`UNLIMITED`（永不挂起，注意内存）、`CONFLATED`（只保留最新值，旧值丢弃，适合状态更新）。

**Q4：⭐协程是线程安全的吗？**

不是。协程只是调度单元，只要运行在多线程调度器（`Default`/`IO`）上，**共享可变状态依然会有竞态**。只有把协程限制在单线程（如 `Main` 或 `limitedParallelism(1)`）时才天然安全。

**Q5：⭐协程中怎么保证并发安全？**

按优先级：① **避免共享**，用不可变数据、把状态收敛到单个协程内；② 限制在**单线程**调度器上执行；③ 用**原子类**做简单计数；④ 用 **`Mutex.withLock`** 保护临界区；⑤ 用 JUC 的并发集合。

**Q6：⭐`Mutex` 和 `synchronized` 有什么区别？**

`synchronized` **阻塞线程**，且**不能在同步块中调用挂起函数**；`Mutex.withLock` 是挂起式的，等待锁时释放线程，块内可以安全调用挂起函数。区别代价是 `Mutex` **不可重入**——同一协程重复获取同一把锁会死锁，而 `synchronized` 是可重入的。

**Q7：为什么协程里不建议用 `synchronized`？**

① 挂起函数无法在同步块中调用，写不了异步逻辑；② 它阻塞的是线程，而协程的价值就在于挂起时释放线程，用 `synchronized` 等于把线程占死，可能拖垮整个调度器线程池，极端情况下还会造成死锁。

**Q8：Channel 忘记 `close()` 会怎样？**

消费端的 `for (item in channel)` 或 `consumeEach` 会**永远挂起等待**下一个元素，协程无法正常结束，作用域也就一直不完成，造成资源泄漏。生产者结束时必须 `close()`，异常路径可用 `close(cause)` 传递错误。

---

## 二十六、Kotlin 与 Java 互操作

> 国内绝大多数项目是**混合工程**（老代码 Java、新代码 Kotlin），这一章是实际工作中踩坑最多、面试也很爱问的部分。

### 1. Kotlin 调用 Java ⭐

基本可以无缝调用，但有四个要注意的地方：

**① 平台类型：空安全的缺口**（第三章详述）

```kotlin
val name = javaObj.getName()      // 推断为 String!（平台类型），编译器不做空检查
val name: String? = javaObj.name  // ⭐规范做法：显式声明为可空
```

**② getter/setter 自动变成属性**

```kotlin
javaObj.getName()          // 可以这么写
javaObj.name               // ⭐更推荐：符合 JavaBean 规范的方法自动变属性
javaObj.name = "Tom"       // 调用 setName()
```

**③ Kotlin 关键字冲突用反引号**

```kotlin
javaObj.`is`()             // ⭐Java 的 is()、in()、object() 等要用反引号转义
```

**④ 其他差异**

| 项 | 说明 |
| ---- | ---- |
| 受检异常 | ⭐Kotlin 不强制处理，`catch` 与否自行决定 |
| 数组 | Java 的 `int[]` 对应 `IntArray`，`String[]` 对应 `Array<String!>` |
| 静态成员 | 直接 `JavaClass.staticMethod()` |
| SAM 接口 | ⭐可直接传 Lambda（见第九章） |

---

### 2. Java 调用 Kotlin：注解大全 ⭐

这是面试重点——**Kotlin 的语法糖在 Java 侧往往很难用，需要注解来"翻译"**。

| Kotlin 写法 | Java 侧默认看到的 | ⭐加注解后 |
| ---- | ---- | ---- |
| 顶层函数 `fun format()` | `UtilsKt.format()` | `@file:JvmName("Utils")` → `Utils.format()` |
| 默认参数 `fun f(a: Int = 1)` | ⭐**用不了默认值** | `@JvmOverloads` → 生成全套重载 |
| `object Manager` | `Manager.INSTANCE.foo()` | `@JvmStatic` → `Manager.foo()` |
| `companion object` 成员 | `User.Companion.create()` | `@JvmStatic` → `User.create()` |
| 属性 `var name` | `getName()` / `setName()` | `@JvmField` → 直接 `user.name` |
| `const val TAG` | ⭐`User.TAG`（本来就是静态常量） | — |
| 抛异常的函数 | ⭐Java 侧**不知道**要 catch | `@Throws(IOException::class)` |
| `internal fun` | 名字被修饰（`foo$module`） | 不建议给 Java 用 |

```kotlin
@file:JvmName("StringUtils")            // ⭐文件级注解，必须写在 package 之前
package com.example

fun isPhone(s: String): Boolean = ...   // Java: StringUtils.isPhone(s)

class User(val name: String) {
    companion object {
        @JvmStatic                       // ⭐Java: User.create("Tom")
        fun create(name: String) = User(name)
    }

    @Throws(IOException::class)          // ⭐Java 侧才能 catch
    fun save() { }
}
```

---

### 3. 空安全的边界 ⭐

```kotlin
fun greet(name: String) { println(name.length) }     // 参数是非空类型
```

```java
Greeter.greet(null);     // ⚠ 运行时抛 NullPointerException
```

原因：编译器在 public 函数入口插入了 `Intrinsics.checkNotNullParameter()` 检查——**Kotlin 的非空约束在 Java 侧是运行期断言，而不是编译期保证**。

> 因此混合工程的原则是：**Kotlin 侧把 Java 来的值都当可空处理；Java 侧调用 Kotlin 前自行保证非空**。给 Java 用的 API 建议在 Java 侧加 `@Nullable`/`@NonNull` 注解形成双向约定。

---

### 4. 协程的互操作 ⚠

**Java 无法直接调用 `suspend` 函数**——它编译后多了一个 `Continuation` 参数，Java 侧根本没法构造。

桥接方式：

```kotlin
// ① 提供一个回调版本给 Java 调用
fun getUserAsync(id: Long, callback: (User) -> Unit) {
    scope.launch { callback(getUser(id)) }
}

// ② 后端项目：转成 CompletableFuture（kotlinx-coroutines-jdk8）
fun getUserFuture(id: Long): CompletableFuture<User> =
    scope.future { getUser(id) }
```

---

### 5. 其他常见坑 ⚠

| 坑 | 说明 |
| ---- | ---- |
| ⭐`isXxx` 属性名 | `val isVip` 的 getter 是 `isVip()`；写成 `vip` 则是 `getVip()`，序列化框架（Jackson/Gson）行为会不同 |
| ⭐data class 无无参构造器 | JPA/Jackson 需要，用 `no-arg` 插件或给所有参数加默认值 |
| Kotlin 类默认 `final` | Spring AOP、Mockito 需要能继承，用 `all-open` / `kotlin-spring` 插件 |
| 泛型通配符 | Kotlin 的 `List<out T>` 在 Java 侧是 `List<? extends T>`，必要时用 `@JvmSuppressWildcards` |
| 接口默认实现 | Java 类实现 Kotlin 接口时拿不到默认方法，需 `-Xjvm-default=all`（见第十六章） |
| `Unit` 返回值 | Java 侧是 `void`，但作为泛型参数时是 `Unit.INSTANCE` |

---

### 6. 迁移策略 ⭐

企业里的常见做法：

1. **新代码一律 Kotlin**，老代码不动——两者可在同一模块内混合编译。
2. **由外向内迁移**：先迁移 Model/工具类等叶子节点，再迁移业务逻辑，最后是基础框架。
3. ⚠ **IDE 的 "Convert Java File to Kotlin" 只是起点**：它会产出大量 `!!`、平台类型和啰嗦的写法，转换后必须人工重构——把 `!!` 换成可空类型处理、把 setter 链改成 `apply`、把工具类改成扩展函数、把静态常量改成 `const val`。
4. **对外 API 边界补注解**：`@JvmStatic`、`@JvmOverloads`、`@Throws`，减少 Java 侧调用摩擦。

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐Java 返回值一律按可空处理 | 平台类型是 NPE 的主要来源 |
| ⭐供 Java 调用的 API 补齐注解 | `@JvmStatic` / `@JvmOverloads` / `@JvmName` / `@Throws` |
| Java 侧接口加 `@Nullable`/`@NonNull` | Kotlin 能识别，转成真正的可空类型 |
| ⭐挂起函数不直接暴露给 Java | 提供回调或 `CompletableFuture` 版本 |
| Spring/JPA 项目引入 `kotlin-spring`、`no-arg` 插件 | 解决 `final` 和无参构造器问题 |
| 转换工具生成的代码必须人工审查 | 尤其是满屏 `!!` 的地方 |
| 混合工程统一序列化属性命名规范 | 避免 `isXxx` 引发的字段名不一致 |

---

### 8. 面试重点

**Q1：Kotlin 和 Java 能混合编译吗？原理是什么？**

能，同一个模块里 `.kt` 和 `.java` 可以互相引用。原因是 Kotlin/JVM 编译产物就是标准的 `.class` 字节码，Kotlin 编译器会先解析 Java 源码/字节码得到符号信息，Java 编译器再编译 Java 代码，两者最终都是 JVM 类。

**Q2：⭐什么是平台类型？为什么危险？**

Kotlin 调用 Java 时因无法确定可空性而产生的类型（IDE 显示为 `String!`），编译器对它**不做空检查**。危险在于 Java 返回 null 时你却当非空使用，编译期毫无提示、运行时直接 NPE。应对：接收时显式声明为可空类型，或在 Java 侧加 `@Nullable`/`@NonNull` 注解。

**Q3：⭐Java 调用 Kotlin 常用哪些注解？各解决什么问题？**

`@JvmStatic`（把伴生对象/object 成员变成真正的静态方法，免写 `.Companion.`）、`@JvmOverloads`（为默认参数生成全套重载，否则 Java 用不了默认值）、`@JvmField`（属性暴露为公开字段，不生成 getter/setter）、`@JvmName`（重命名文件生成的类名或函数名）、`@Throws`（声明受检异常，Java 侧才能 catch）。

**Q4：Kotlin 的默认参数为什么 Java 用不了？**

因为默认参数编译后是**一个带位掩码的合成方法 `xxx$default`**，而不是多个重载方法，Java 侧无法正常调用。加 `@JvmOverloads` 后编译器会额外生成各参数组合的重载方法。

**Q5：Java 传 null 给 Kotlin 的非空参数会怎样？**

抛出 **NPE**。因为 Kotlin 会在 public 函数入口插入 `Intrinsics.checkNotNullParameter()` 断言。这说明 Kotlin 的非空约束对 Java 侧只是**运行期检查**，而不是编译期保证——这也是"Kotlin 还会不会 NPE"的标准答案之一。

**Q6：⭐Java 能调用 `suspend` 函数吗？**

不能直接调用。挂起函数编译后会多出一个 `Continuation` 参数并被改写成状态机，Java 侧无法构造。实践中要提供桥接：暴露一个回调版本的方法，或用 `kotlinx-coroutines-jdk8` 的 `scope.future { }` 转成 `CompletableFuture`。

**Q7：Kotlin 项目用 Spring/JPA 有哪些坑？**

① 类和方法默认 `final`，Spring AOP 的 CGLIB 代理和 Mockito 无法继承——用 `kotlin-spring`（all-open）插件；② data class 没有无参构造器，JPA/Jackson 反序列化失败——用 `no-arg` 插件或给所有参数加默认值；③ 实体类用 data class 时 `equals`/`hashCode` 基于全部主构造属性，与 JPA 的按主键判等语义冲突，实体建议用普通类。

**Q8：IDE 的 Java 转 Kotlin 工具能直接用吗？**

只能作为**起点**。转换结果通常保守：大量 `!!` 和平台类型、保留 Java 式的工具类和 getter/setter 调用、没有用到 data class、扩展函数、作用域函数等 Kotlin 特性。必须人工重构，否则会得到"用 Kotlin 语法写的 Java 代码"，既不安全也不简洁。

---

## 二十七、反射、注解与 DSL 构建

## 注解

### 1. 定义与使用

```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)   // 能用在哪
@Retention(AnnotationRetention.RUNTIME)                       // 保留到什么时候
annotation class Api(val path: String, val desc: String = "")

@Api("/user", desc = "用户接口")
class UserController
```

| 元注解 | 作用 |
| ---- | ---- |
| `@Target` | 可用位置：`CLASS`、`FUNCTION`、`PROPERTY`、`FIELD`、`VALUE_PARAMETER` 等 |
| ⭐`@Retention` | `SOURCE`（编译后丢弃）/ `BINARY`（保留但反射不可见）/ ⭐`RUNTIME`（默认，反射可读） |
| `@Repeatable` | 允许在同一位置重复使用 |

> 注解参数只能是基本类型、`String`、枚举、`KClass`、其他注解及它们的数组。

---

### 2. ⭐使用点目标（Kotlin 特有）

一个 Kotlin 属性会生成**字段 + getter + setter + 构造参数**四个位置，注解到底加在哪个上面必须说清楚：

```kotlin
class User(
    @field:JsonProperty("user_name")     // ⭐加到字段上
    @get:NotNull                          // 加到 getter 上
    @param:NotBlank                       // 加到构造参数上
    val name: String
)
```

> ⚠ 这是混合工程中**序列化/校验框架不生效**的头号原因——注解加错了位置。

---

### 3. 常用内置注解

| 注解 | 作用 |
| ---- | ---- |
| ⭐`@Deprecated` | 标记废弃，配 `ReplaceWith` 可一键替换 |
| ⭐`@Suppress("...")` | 抑制指定的编译告警 |
| `@Volatile` / `@Synchronized` | 对应 Java 的 `volatile` / `synchronized` |
| `@JvmStatic` / `@JvmField` / `@JvmName` / `@Throws` | Java 互操作（见第二十六章） |

```kotlin
@Deprecated("请使用 newApi()", ReplaceWith("newApi(id)"))
fun oldApi(id: Long) { }
```

---

## 反射

### 4. Kotlin 反射 vs Java 反射 ⭐

```kotlin
val kClass: KClass<User> = User::class          // ⭐Kotlin 反射入口
val jClass: Class<User> = User::class.java      // ⭐Java 反射入口（转换）

val kClass2 = user::class                        // 从实例获取
```

**⚠ 完整的 Kotlin 反射能力需要单独引入 `kotlin-reflect` 依赖**（约 2~3MB），否则只能使用 `simpleName` 等基础功能。

```kotlin
// KClass 常用 API（需要 kotlin-reflect）
User::class.simpleName          // "User"
User::class.qualifiedName       // "com.example.User"
User::class.isData              // ⭐是否是 data class
User::class.memberProperties    // 所有属性
User::class.declaredFunctions   // 所有函数
User::class.constructors
User::class.objectInstance      // object 的单例实例
User::class.createInstance()    // 无参构造实例化
```

**属性与函数引用**：

```kotlin
val prop = User::name                   // KProperty1<User, String>
prop.get(user)                          // 取值
prop.name                               // "name"

val func = User::sayHi                  // KFunction
func.call(user)

// 访问私有成员
prop.isAccessible = true
```

---

### 5. 实战：对象转 Map

```kotlin
fun Any.toMap(): Map<String, Any?> =
    this::class.memberProperties.associate { prop ->
        prop.name to (prop as KProperty1<Any, *>).get(this)
    }

User(1, "Tom").toMap()      // {id=1, name=Tom}
```

**⚠ 反射的代价**：

| 问题 | 说明 |
| ---- | ---- |
| ⭐性能差 | 比直接调用慢一到两个数量级，Kotlin 反射还比 Java 反射多一层封装 |
| ⭐包体积 | `kotlin-reflect` 有 2~3MB，Android 项目通常**不引入** |
| 混淆风险 | ProGuard/R8 混淆后反射按名字找不到成员，需要 keep 规则 |
| 编译期无检查 | 名字写错只有运行时才暴露 |

> **替代方案**：能用 `inline` + `reified` 就不用反射；序列化用 `kotlinx.serialization`（编译期生成，无反射）；需要生成代码用 **KSP**（Kotlin Symbol Processing）注解处理器。

---

## DSL 构建 ⭐

### 6. 什么是 Kotlin DSL

用 Kotlin 语法写出的"领域专用语言"，实际都是**普通函数调用**：

```kotlin
// Gradle KTS
dependencies { implementation("...") }

// Jetpack Compose
Column { Text("hello") }

// Ktor 路由
routing { get("/user") { call.respond(user) } }
```

### 7. 三大支柱 ⭐

| 支柱 | 作用 |
| ---- | ---- |
| ⭐**带接收者的 Lambda** `T.() -> Unit` | Lambda 内部的 `this` 是 T，可以直接调用它的成员 |
| ⭐**尾随 Lambda** | 把 `{ }` 移出括号，看起来像语言结构 |
| **扩展函数 / `infix` / `operator`** | 让调用读起来像自然语言 |

**手写一个配置 DSL**：

```kotlin
class Request {
    var url: String = ""
    var method: String = "GET"
    private val headers = mutableMapOf<String, String>()

    fun header(k: String, v: String) { headers[k] = v }        // 嵌套构建
}

// ⭐关键：参数类型是 Request.() -> Unit（带接收者的 Lambda）
fun request(block: Request.() -> Unit): Request = Request().apply(block)

// 使用：像写配置文件一样
val req = request {
    url = "https://api.xxx.com"        // ⭐this 就是 Request，可直接访问属性
    method = "POST"
    header("token", "abc")
}
```

> 对比第十二章：`apply` 的签名正是 `T.(T.() -> Unit): T`——**标准库的作用域函数本身就是最小的 DSL**。

### 8. @DslMarker：防止作用域串味 ⭐

嵌套 DSL 时，内层 Lambda 能隐式访问外层接收者，容易误用：

```kotlin
@DslMarker
annotation class HtmlDsl              // ⭐自定义标记注解

@HtmlDsl class HTML { fun body(block: BODY.() -> Unit) { } }
@HtmlDsl class BODY { }

html {
    body {
        body { }        // ⭐编译错误：被 @DslMarker 拦住了，避免误调用外层的方法
    }
}
```

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐能不用反射就不用 | 优先 `reified`、密封类、KSP 代码生成 |
| ⭐Android 项目慎引 `kotlin-reflect` | 包体积增加 2~3MB |
| 反射涉及的类要加混淆 keep 规则 | 否则线上必崩 |
| 序列化用 `kotlinx.serialization` | 编译期生成，无反射、性能好、支持多平台 |
| ⭐属性注解务必写清使用点目标 | `@field:`/`@get:`/`@param:` |
| DSL 只用于**配置类、声明式**场景 | 业务逻辑别硬写成 DSL，可读性反而更差 |
| 嵌套 DSL 必须加 `@DslMarker` | 防止作用域混淆 |
| 废弃 API 用 `@Deprecated` + `ReplaceWith` | 便于调用方一键迁移 |

---

### 10. 面试重点

**Q1：Kotlin 反射和 Java 反射有什么区别？**

入口不同：`User::class` 得到 `KClass`，`User::class.java` 得到 `Class`。Kotlin 反射能感知 Kotlin 特有的概念（属性、可空性、data class、object 单例、默认参数），Java 反射只能看到擦除后的字节码视角。代价是**必须额外引入 `kotlin-reflect` 依赖（2~3MB）**，且性能比 Java 反射更慢，Android 项目一般不用。

**Q2：⭐为什么框架注解加在 Kotlin 属性上会"不生效"？**

因为一个属性会生成字段、getter、setter、构造参数多个元素，不指定**使用点目标**时注解可能落在你不期望的位置上。要显式写 `@field:`、`@get:`、`@param:` 等前缀。这是 Jackson/Gson/Hibernate 校验注解在 Kotlin 项目里失效的最常见原因。

**Q3：`@Retention` 有哪几种？默认是什么？**

`SOURCE`（只在源码，编译后丢弃，如 `@Suppress`）、`BINARY`（保留在字节码但反射读不到）、`RUNTIME`（保留且反射可读）。**Kotlin 默认是 `RUNTIME`**（Java 默认是 `CLASS`/BINARY，这点相反）。

**Q4：反射有什么代价？有哪些替代方案？**

性能比直接调用慢一到两个数量级、增加包体积、混淆后按名字查找会失败、编译期没有类型检查。替代方案：`inline` + `reified` 拿类型信息、`kotlinx.serialization` 做序列化（编译期生成代码）、**KSP** 注解处理器在编译期生成代码、用密封类替代类型判断。

**Q5：⭐Kotlin DSL 的实现原理是什么？**

三个语言特性的组合：① **带接收者的 Lambda**（`T.() -> Unit`），使 Lambda 内的 `this` 变成目标对象，可直接访问其成员；② **尾随 Lambda**，把 `{}` 移出括号，让函数调用看起来像语言结构；③ **扩展函数 / `infix` / `operator`**，让语法更自然。Gradle KTS、Compose、Ktor 都基于这套机制。

**Q6：`T.() -> Unit` 和 `(T) -> Unit` 有什么区别？**

前者是**带接收者的函数类型**，Lambda 内用 `this` 隐式访问 T 的成员（可省略），如 `apply`；后者是普通函数类型，T 作为参数传入，要用 `it` 访问，如 `also`。DSL 依赖前者才能写出"直接写属性名"的效果。

**Q7：`@DslMarker` 解决什么问题？**

嵌套 DSL 时，内层 Lambda 默认可以隐式访问**外层接收者**的成员，容易误调用（比如在 `body` 里又写了一个 `body`）。用 `@DslMarker` 标记的注解修饰各层接收者类后，编译器会禁止跨层的隐式访问，把这类错误提前到编译期。

---

## 二十八、编译原理与字节码剖析

> 面试中区分"会写 Kotlin"和"懂 Kotlin"的一章。核心认知只有一句：**Kotlin 的绝大多数特性都是编译期语法糖，运行时并没有魔法。**

### 1. 一个必备技能：看字节码 ⭐

IntelliJ IDEA / Android Studio：**Tools → Kotlin → Show Kotlin Bytecode → 点 Decompile**，就能看到这段 Kotlin 对应的 Java 代码。

> 遇到"这个语法到底做了什么""为什么慢""为什么 Java 侧调不到"，反编译一看便知——这也是面试时最有说服力的回答方式。

**编译流程**：

```text
.kt 源码 → 前端解析（语法/类型/推断，生成 FIR）→ IR 中间表示 → 后端生成 .class
                                    ⭐Kotlin 2.0 起默认使用 K2 编译器，编译速度显著提升
```

---

### 2. 语法糖的编译产物速查 ⭐（本章核心）

| Kotlin 语法 | 编译后的真实模样 |
| ---- | ---- |
| 顶层函数 | `FileNameKt` 类的 `static` 方法 |
| 属性 `var name` | `private` 字段 + `getName()` / `setName()` |
| ⭐`data class` | 自动生成 `equals`/`hashCode`/`toString`/`copy`/`componentN` |
| ⭐`object` | `static final INSTANCE` 字段，静态块中初始化 |
| ⭐`companion object` | 外部类中的 `Companion` 静态内部类实例 |
| ⭐扩展函数 | **静态方法**，接收者作为第一个参数 |
| ⭐默认参数 | `xxx$default` 合成方法 + `int` 位掩码 |
| ⭐空安全 | 函数入口插入 `Intrinsics.checkNotNullParameter()`；类型上加 `@NotNull`/`@Nullable` |
| `?.` | `if (x == null) null else x.foo()`，**零额外开销** |
| 字符串模板 | `StringBuilder` 拼接（JVM 9+ 走 `invokedynamic`） |
| ⭐Lambda | `FunctionN` 接口实现类（Kotlin 2.x 默认走 `invokedynamic` 运行时生成） |
| ⭐`inline` 函数 | 函数体和 Lambda 体**直接展开**到调用处 |
| ⭐委托属性 `by` | `xxx$delegate` 字段 + 访问器转发 `getValue`/`setValue` |
| `when` | `tableswitch`/`lookupswitch`（String 走 `hashCode` + switch）或 if-else 链 |
| ⭐`value class` | **擦除为底层类型**，运行时没有包装对象 |
| 闭包捕获 `var` | 包装成 `Ref.IntRef` / `ObjectRef` 对象放到堆上 |
| 局部函数 | 带 `$` 前缀的静态方法 |
| ⭐`suspend` 函数 | 多一个 `Continuation` 参数 + **状态机** |

---

### 3. 三个最值得展开的例子 ⭐

**① data class**

```kotlin
data class User(val id: Long, val name: String)
```

```java
public final class User {
    private final long id;
    private final String name;
    public final long getId() { ... }
    public final User copy(long id, String name) { ... }      // ⭐copy
    public final long component1() { ... }                     // ⭐解构用
    public String toString() { return "User(id=" + id + ...; }
    public int hashCode() { ... }                              // 基于所有主构造属性
    public boolean equals(Object o) { ... }
}
```

> 由此立刻能解释：为什么类体中的属性不参与 `equals`——生成器只看主构造器。

**② suspend 函数（面试最高频）**

```kotlin
suspend fun loadUser(): User {
    val a = api.getUser()      // 挂起点 1
    val b = api.getProfile()   // 挂起点 2
    return merge(a, b)
}
```

编译后大致：

```java
Object loadUser(Continuation $completion) {
    switch ($continuation.label) {                 // ⭐状态机
        case 0:
            $continuation.label = 1;
            result = api.getUser($continuation);
            if (result == COROUTINE_SUSPENDED) return COROUTINE_SUSPENDED;  // ⭐挂起，释放线程
        case 1:
            $continuation.label = 2;
            ...
    }
}
```

要点：**挂起点把函数切成若干片段，用 `label` 记录进度**；挂起时保存状态并立即返回，线程被释放；恢复时通过 `Continuation` 跳回对应分支继续执行。所谓"同步写异步"，本质是编译器替你写了回调和状态机。

**③ 扩展函数**

```kotlin
fun String.isPhone(): Boolean = length == 11
```

```java
public static boolean isPhone(String $this$isPhone) { ... }    // ⭐静态方法
```

> 由此立刻能解释第十一章那三条结论：静态解析（无多态）、成员函数优先、访问不到 private 成员。

---

### 4. Kotlin 相比 Java 的额外开销 ⭐

绝大多数特性是零开销的，但以下几处确实有成本：

| 开销点 | 说明 | 建议 |
| ---- | ---- | ---- |
| ⭐空检查插桩 | public 函数入口的 `Intrinsics` 校验 | 可忽略；`internal`/`private` 函数不插桩 |
| ⭐Lambda 对象 | 非 `inline` 的高阶函数会创建对象 | 高阶函数加 `inline` |
| ⭐装箱 | `Int?`、`List<Int>`、泛型参数会装箱成 `Integer` | 大数组用 `IntArray`，热点路径避免可空基本类型 |
| 委托属性 | 多一个委托对象 + 一次转发 | 超高频访问的属性不用委托 |
| 集合链式操作 | 每步生成中间集合 | 大数据量用 `asSequence()` |
| `data class` 的 `equals` | 属性多时逐个比较 | 热点路径注意 |
| 伴生对象 | 多一个 `Companion` 对象 | 常量用 `const val`（直接内联） |

> 结论：**日常业务代码不必担心 Kotlin 的性能**，只在高频循环、大数据量、移动端启动路径上关注上述几点。

---

### 5. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐拿不准就反编译看一眼 | Show Kotlin Bytecode → Decompile |
| 高阶函数加 `inline` | 消除 Lambda 对象 |
| 热点路径避免可空基本类型和泛型装箱 | 用 `IntArray`、非空类型 |
| 常量用 `const val` | 编译期内联，无方法调用 |
| 供 Java 调用的 API 关注编译产物 | 提前用注解调整（见第二十六章） |
| 升级到 Kotlin 2.x 使用 K2 编译器 | 编译速度和类型推断都有明显改善 |

---

### 6. 面试重点

**Q1：⭐`suspend` 函数编译后是什么样的？**

编译器做 **CPS 变换**：① 函数签名末尾多一个 `Continuation` 参数，返回类型变成 `Object`；② 函数体按挂起点切分成状态机，用 `label` 字段记录执行进度；③ 遇到挂起点时保存局部变量和 `label`，返回 `COROUTINE_SUSPENDED` 并释放线程；④ 恢复时通过 `Continuation.resumeWith` 回到对应的 `case` 继续执行。

**Q2：⭐扩展函数为什么没有多态？**

因为它编译成**静态方法**，接收者作为第一个参数。静态方法在编译期就完成绑定，只看变量的声明类型，不看运行时类型，所以既没有多态，也会被同名成员函数遮蔽，还访问不到接收者的私有成员。

**Q3：`data class` 的方法是怎么生成的？为什么类体属性不参与？**

编译器在生成 `equals`/`hashCode`/`toString`/`copy`/`componentN` 时**只读取主构造器的参数列表**，类体中声明的属性不在这个列表里，自然不会被纳入。这也是"两个 name 不同的对象却相等"这个经典现象的根源。

**Q4：`object` 单例的线程安全是怎么保证的？**

编译成一个带 `static final INSTANCE` 字段的类，在**静态初始化块**中赋值。JVM 保证类的静态初始化只执行一次且是线程安全的，同时类在首次主动使用时才加载，因此天然是"懒加载 + 线程安全"，等价于 Java 的静态内部类单例写法。

**Q5：Kotlin 的空安全在运行时还剩什么？**

主要是两样：public 函数入口的 `Intrinsics.checkNotNullParameter()` 断言（防止 Java 侧传 null），以及字节码中保留的 `@NotNull`/`@Nullable` 注解元信息（供其他 Kotlin 模块和 IDE 使用）。类型本身在字节码中并无差别，`String` 与 `String?` 都是 `java.lang.String`。

**Q6：Kotlin 有哪些地方比 Java 多开销？**

非内联 Lambda 的对象创建、可空基本类型和泛型导致的装箱、委托属性的转发、集合链式操作的中间集合、public 函数入口的空检查插桩、伴生对象实例。绝大多数在业务代码中可忽略，只在高频路径和移动端需要针对性优化。

**Q7：怎么验证某个 Kotlin 语法的实现？**

IDEA 里 **Tools → Kotlin → Show Kotlin Bytecode**，再点 **Decompile** 转成等价 Java 代码。这是排查"Java 调不到这个方法""为什么这里有对象分配""默认参数怎么实现"等问题最直接的手段。

**Q8：K2 编译器带来了什么？**

Kotlin 2.0 起默认启用的新一代编译器前端，重写了解析与类型推断（基于 FIR），带来**明显更快的编译速度**、更强更一致的智能转换能力，以及为多平台和后续语言特性打下的统一基础。日常开发感受最明显的就是编译变快、IDE 分析更准。

---

## 二十九、工程实践与生态（Gradle KTS、序列化、单元测试）

### 1. Gradle KTS ⭐

`build.gradle.kts` 用 Kotlin 写构建脚本，取代 Groovy：

```kotlin
plugins {
    kotlin("jvm") version "2.0.0"
    kotlin("plugin.serialization") version "2.0.0"
}

dependencies {
    implementation(libs.kotlinx.coroutines)      // ⭐版本目录引用
    testImplementation(kotlin("test"))
}

kotlin { jvmToolchain(17) }
```

| 对比 | Groovy DSL | ⭐Kotlin DSL |
| ---- | ---- | ---- |
| 类型检查 | 运行时才报错 | ⭐编译期 |
| IDE 补全 / 跳转 | 弱 | ⭐完整 |
| 首次编译速度 | 快 | 略慢（有缓存后差别不大） |

**版本目录 `gradle/libs.versions.toml`**（现在的标准做法，多模块统一版本）：

```toml
[versions]
coroutines = "1.8.0"

[libraries]
kotlinx-coroutines = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "coroutines" }
```

---

### 2. 序列化：kotlinx.serialization ⭐

```kotlin
@Serializable
data class User(
    val id: Long,
    @SerialName("user_name") val name: String,      // ⭐字段改名
    val age: Int = 18                                // 默认值
)

val json = Json {
    ignoreUnknownKeys = true      // ⭐忽略后端多返回的字段（必开）
    coerceInputValues = true      // ⭐null 转成默认值，防止非空字段被写入 null
    encodeDefaults = false
}

val str = json.encodeToString(user)
val obj = json.decodeFromString<User>(str)          // ⭐reified，无需传 Class
```

**与 Gson / Jackson 的对比** ⭐：

| 对比项 | Gson / Jackson | ⭐kotlinx.serialization |
| ---- | ---- | ---- |
| 实现方式 | **运行时反射** | ⭐**编译期生成**序列化器 |
| 性能 / 包体 | 反射有开销 | ⭐更快，无 `kotlin-reflect` 依赖 |
| ⚠ 空安全 | ⭐**能绕过非空检查**，让非空字段变成 null | ⭐尊重类型系统，缺字段直接报错 |
| 默认参数 | 不生效（走 Unsafe 实例化） | ⭐正确使用 |
| 多平台 | 仅 JVM | ⭐支持 KMP |

> ⚠ **Gson 最大的坑**：它通过 `Unsafe` 创建对象、反射赋值，完全绕过 Kotlin 的构造器和空检查。后端少返回一个字段，你声明为 `String` 的非空属性运行时就是 `null`，之后在任意一处访问时崩溃且堆栈莫名其妙。新项目一律用 kotlinx.serialization 或 Moshi（codegen 模式）。

---

### 3. 单元测试 ⭐

**基础：**

```kotlin
class UserServiceTest {
    @Test
    fun `用户名为空时应该抛异常`() {          // ⭐反引号方法名，测试用例可读性极佳
        assertFailsWith<IllegalArgumentException> { service.create("") }
    }
}
```

**Mock：用 MockK（Kotlin 首选）** ⭐

```kotlin
val repo = mockk<UserRepository>()
coEvery { repo.getUser(1) } returns User(1, "Tom")     // ⭐coEvery 支持挂起函数
coVerify { repo.getUser(1) }
```

> 为什么不用 Mockito？Kotlin 的类和方法**默认 `final`**，Mockito 需要额外配置 `mock-maker-inline` 才能 mock，且对 `object`、扩展函数、挂起函数支持不佳。**MockK 原生支持**这些 Kotlin 特性。

**协程测试** ⭐：

```kotlin
@Test
fun test() = runTest {                    // ⭐kotlinx-coroutines-test，自动跳过 delay
    val result = service.loadUser()        // delay(10_000) 会被虚拟时间瞬间推进
    assertEquals("Tom", result.name)
}

// 替换主调度器（Android ViewModel 测试必备）
Dispatchers.setMain(StandardTestDispatcher())
```

**Flow 测试**：用 **Turbine**

```kotlin
viewModel.state.test {
    assertEquals(UiState.Loading, awaitItem())
    assertTrue(awaitItem() is UiState.Success)
    cancelAndIgnoreRemainingEvents()
}
```

---

### 4. 代码质量工具 ⭐

| 工具 | 作用 |
| ---- | ---- |
| ⭐**ktlint** | 格式化 + 风格检查，可接入 Git hook / CI |
| ⭐**detekt** | 静态分析：复杂度、坏味道、潜在 bug（如滥用 `!!`） |
| **explicit API mode** | 库开发时强制显式写可见性和返回类型 |
| 官方 code style | `kotlin.code.style=official`，团队统一 |

> 面试加分点：能说出"我们在 CI 里接了 ktlint + detekt，`!!` 和圈复杂度超标会直接卡住 MR"，比说"我注意代码规范"有力得多。

---

### 5. 主流生态速查 ⭐

**Android（Kotlin 岗位主战场）**：

| 领域 | 主流选择 |
| ---- | ---- |
| UI | ⭐Jetpack Compose（声明式，纯 Kotlin DSL） |
| 架构 | ⭐ViewModel + StateFlow + Repository |
| 异步 | ⭐协程 + Flow |
| 网络 | Retrofit（原生支持 `suspend`）+ OkHttp |
| 本地存储 | Room（支持 `suspend` / Flow 返回）、DataStore |
| 依赖注入 | Hilt / Koin |
| 图片 | Coil（Kotlin 编写，协程友好） |

**服务端**：

| 领域 | 主流选择 |
| ---- | ---- |
| 框架 | ⭐Spring Boot（官方支持 Kotlin）/ Ktor（JetBrains，协程原生） |
| 必备插件 | ⭐`kotlin-spring`（all-open）、`kotlin-jpa`（no-arg） |
| 异步 | 协程（Spring 6 的 WebFlux 支持挂起函数控制器） |

**跨平台（KMP）**：共享业务逻辑与网络层，UI 用 Compose Multiplatform 或各平台原生；目前处于**稳步增长期**，大厂已有落地，中小公司仍以观望为主。

---

### 6. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐新项目构建脚本用 KTS + 版本目录 | 类型安全、多模块版本统一 |
| ⭐序列化用 kotlinx.serialization | 避免 Gson 绕过空安全的坑 |
| ⭐`Json` 必开 `ignoreUnknownKeys` | 后端加字段不会导致解析崩溃 |
| Mock 用 MockK，协程测试用 `runTest` | 与 Kotlin 特性天然契合 |
| CI 接入 ktlint + detekt | 把规范落到流程而不是口头约定 |
| 统一 Kotlin 与协程版本 | 多模块版本不一致会引发诡异的运行时错误 |
| 及时升级到 Kotlin 2.x | 享受 K2 编译速度与新特性 |

---

### 7. 面试准备建议 ⭐

**技术侧重点**（按被问频率排序）：

1. ⭐**协程**：挂起原理（CPS + 状态机）、结构化并发、取消机制、异常处理、Flow 冷热流 —— **占比最高，务必吃透第 22~25 章**。
2. ⭐**空安全**：类型系统设计、平台类型、`!!` 的危害。
3. ⭐**与 Java 的差异**：data class、扩展函数、密封类、委托、默认参数的编译产物。
4. **函数式**：高阶函数、`inline`/`reified`、作用域函数、集合与 Sequence。
5. **工程能力**：混合工程互操作、序列化选型、测试、代码质量工具。

**回答技巧**：

- 尽量讲到**编译产物**（"它编译后其实是一个静态方法…"），这是区分度最高的信号。
- 结合**项目场景**（"我们用 `sealed class` 封装接口结果，新增状态时编译器会强制我们处理所有分支"）。
- 主动说出**坑和权衡**（"`inline` 不能滥用，函数体大了会字节码膨胀"），比只背优点更可信。

---

### 8. 面试重点

**Q1：kotlinx.serialization 和 Gson 有什么区别？为什么推荐前者？**

kotlinx.serialization 在**编译期生成序列化代码**，无反射、性能更好、支持多平台，且**尊重 Kotlin 的类型系统**：非空字段缺失会直接报错，默认参数也能正确生效。Gson 通过反射和 `Unsafe` 创建对象，**完全绕过构造器和空检查**——后端少返回一个字段，声明为非空的属性运行时就是 `null`，会在无关的地方崩溃，是混合项目里最隐蔽的坑之一。

**Q2：Kotlin 项目为什么推荐 MockK 而不是 Mockito？**

Kotlin 的类和方法默认 `final`，Mockito 需要额外的 inline mock maker 才能处理，且对 `object` 单例、伴生对象、扩展函数、挂起函数支持不佳。MockK 是为 Kotlin 设计的，原生支持这些特性，并提供 `coEvery`/`coVerify` 专门处理挂起函数。

**Q3：协程怎么做单元测试？**

用 `kotlinx-coroutines-test` 的 `runTest`：它提供**虚拟时间**，`delay` 会被瞬间推进，因此测试既快又确定。测试 Android ViewModel 时用 `Dispatchers.setMain(StandardTestDispatcher())` 替换主调度器；测试 Flow 推荐 Turbine，用 `awaitItem()` 逐个断言发射值。

**Q4：Gradle KTS 相比 Groovy 有什么优势和代价？**

优势是类型安全（配置写错在编译期就报错）、IDE 补全和跳转完整、与项目代码同语言。代价是首次编译/脚本编译略慢。配合版本目录 `libs.versions.toml` 可以统一多模块依赖版本，是目前新项目的标准做法。

**Q5：Kotlin 用于 Spring Boot 项目要注意什么？**

必须引入 `kotlin-spring`（all-open）插件解决类默认 `final` 导致 AOP 代理失败的问题，以及 `kotlin-jpa`（no-arg）插件为实体类生成无参构造器；实体类不建议用 data class（`equals` 语义与 JPA 按主键判等冲突）；DTO 用 data class 很合适；注解要注意使用点目标（`@field:`/`@get:`）。

**Q6：你在项目中怎么保证 Kotlin 代码质量？**

统一 `kotlin.code.style=official`，CI 中接入 **ktlint**（格式）与 **detekt**（静态分析，可配置禁止 `!!`、限制圈复杂度），MR 未通过检查不允许合并；配合单元测试（MockK + `runTest` + Turbine）覆盖核心逻辑；对外提供的模块开启 explicit API mode，强制显式声明可见性和返回类型。

---

## 结语

全书 29 章覆盖了 Kotlin 从语法到原理的完整体系。若时间有限，按这个优先级复习：

**必须吃透**：空安全（三）→ 高阶函数与 inline（九、十）→ 扩展函数（十一）→ 作用域函数（十二）→ 数据类与密封类（十七）→ **协程全部（二十二 ~ 二十五）**

**加分项**：编译原理与字节码（二十八）→ Java 互操作（二十六）→ 委托（二十）→ 泛型型变（二十一）
