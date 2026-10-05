# Java

> **版本基线**：全文以 **JDK 17（LTS）** 为基线。JDK 8 用不了的语法和 API 都在原处标注了版本号（如 `record`（JDK 16+）），JDK 8 的项目按标注取舍即可；版本演进的完整对照见**第二十四章**。
>
> **符号约定**：⭐ = 工作中最常用 / 首选写法；🔥 = 面试高频，🔥🔥 = 几乎必问。
>
> **范围说明**：本篇只覆盖 **Java SE（语言 + 核心类库 + JVM）**。后端面试中同样占大头的 MySQL、Redis、Spring / Spring Boot、MyBatis、消息队列、分布式等内容不在本篇，需另行准备。

## 目录

**基础语法**

- [一、关键字与保留字](#一关键字与保留字)
- [二、变量与基本数据类型](#二变量与基本数据类型) 🔥
- [三、基本数据类型转换](#三基本数据类型转换)
- [四、运算符](#四运算符) 🔥
- [五、流程控制（分支与循环）](#五流程控制分支与循环)
- [六、字符串](#六字符串) 🔥🔥
- [七、数组](#七数组)

**面向对象**

- [八、面向对象基础（类、对象、构造器、方法）](#八面向对象基础类对象构造器方法)
  - [实体类的实际写法](#7-实体类的实际写法) · [⚠️ Lombok 的坑](#8-lombok-的坑重点)
- [九、封装与 static](#九封装与-static) 🔥
- [十、继承与多态](#十继承与多态) 🔥
- [十一、抽象类与接口](#十一抽象类与接口) 🔥
- [十二、枚举与内部类](#十二枚举与内部类)
- [十三、Object 类与包装类](#十三object-类与包装类) 🔥🔥

**核心类库**

- [十四、异常（Exception）](#十四异常exception) 🔥
- [十五、泛型](#十五泛型) 🔥
- [十六、集合类](#十六集合类) 🔥🔥
  - [集合体系](#1-集合体系) · [List](#2-list) · [Set](#3-set) · [HashMap](#4-hashmap重中之重) · [LinkedHashMap 与 TreeMap](#5-linkedhashmap-与-treemap)
  - [排序：Comparable 与 Comparator](#6-排序comparable-与-comparator) · [线程安全的集合](#7-线程安全的集合) · [常见坑](#8-集合的常见坑) · [怎么选](#9-怎么选)
- [十七、Lambda 与 Stream API](#十七lambda-与-stream-api) 🔥
  - [本质与语法](#1-本质与语法) · [方法引用](#2-方法引用) · [常用函数式接口](#3-常用函数式接口) · [Stream 是什么](#4-stream-是什么)
  - [中间操作](#5-中间操作) · [终端操作](#6-终端操作) · [Collectors](#7-collectors实际用得最多的部分) · [Optional](#8-optional) · [Stream 的坑](#9-stream-的坑)
- [十八、常用工具类（BigDecimal、日期时间、Optional）](#十八常用工具类bigdecimal日期时间optional)
- [十九、IO 流与网络编程](#十九io-流与网络编程) 🔥
- [二十、多线程与并发（JUC）](#二十多线程与并发juc) 🔥🔥
  - [线程基础与中断](#1-线程基础) · [JMM 与 happens-before](#2-线程安全的三个核心问题) · [synchronized](#3-synchronized) · [volatile](#4-volatile)
  - [ReentrantLock 与 AQS](#5-synchronized-vs-reentrantlock) · [CAS 与原子类](#6-cas-与原子类) · [线程池](#7-线程池实际开发核心--面试重点)
  - [并发工具与阻塞队列](#8-常用并发工具) · [ThreadLocal](#9-threadlocal) · [虚拟线程](#10-虚拟线程jdk-21) · [并发的坑](#11-并发的坑)

**进阶与原理**

- [二十一、反射与注解](#二十一反射与注解) 🔥
- [二十二、JVM（内存结构、GC、类加载）](#二十二jvm内存结构gc类加载) 🔥🔥
  - [运行时内存结构](#1-运行时内存结构) · [对象的创建与分配](#2-对象的创建与分配) · [垃圾回收与四种引用](#3-垃圾回收)
  - [类加载与 SPI](#4-类加载机制) · [常用启动参数](#5-常用启动参数) · [线上问题排查](#6-线上问题排查实战题)
- [二十三、常用设计模式](#二十三常用设计模式) 🔥
- [二十四、JDK 版本演进与新特性总览](#二十四jdk-版本演进与新特性总览) 🔥

---

## 一、关键字与保留字

### 1. 概念

**关键字**：已被赋予特殊含义的单词，如 `public`、`class`。

**保留字**：预留但未启用，只有 `goto`、`const` 两个。

两者都不能用作标识符（变量名、类名等）。

> `true`、`false`、`null` **不是关键字**，属于字面量（Literal）

---

### 2. 企业开发常用关键字

| 分类 | 关键字 |
| ---- | ------ |
| 类型定义 | ⭐`class`、⭐`interface`、`enum`、`record`（JDK 16+） |
| 访问控制 | ⭐`public`、⭐`private`、`protected` |
| 对象 | ⭐`new`、⭐`this`、`super` |
| 继承 | ⭐`extends`、⭐`implements` |
| 流程控制 | ⭐`if`/`else`、⭐`for`、`while`、`switch`、`break`、`continue`、⭐`return` |
| 异常 | ⭐`try`/⭐`catch`、`finally`、`throw`、`throws` |
| 修饰符 | ⭐`static`、⭐`final`、`abstract`、`synchronized`、`volatile`、`transient` |
| 包管理 | `package`、`import` |

---

### 3. 访问修饰符的作用范围（高频）

| 修饰符 | 本类 | 同包 | 子类（跨包） | 任意位置 |
| ------ | :--: | :--: | :----------: | :------: |
| `private` | ✔ | ✘ | ✘ | ✘ |
| 默认（不写） | ✔ | ✔ | ✘ | ✘ |
| `protected` | ✔ | ✔ | ✔ | ✘ |
| `public` | ✔ | ✔ | ✔ | ✔ |

> 企业规范：**字段一律 `private`，方法按需暴露 `public`**。`protected` 只在设计继承体系时用，默认修饰符基本不用。

---

### 4. 面试重点

**Q：Java 为什么保留 `goto` 却不实现？**

为避免程序流程混乱、提升可读性与可维护性。需要跳出多重循环时用**标签（label）** 代替（见第五章）。

---

## 二、变量与基本数据类型

### 1. 变量的分类与默认值

```text
所有变量
├─ 成员变量（类中定义）
│  ├─ 实例变量（无 static）—— 存在堆中，随对象创建
│  └─ 类变量  （有 static）—— 随类加载，全类共享一份
└─ 局部变量（方法/构造器/代码块内定义，含形参）—— 存在栈中，方法结束即销毁
```

**核心区别**：成员变量有**默认值**，不初始化也能用；**局部变量没有默认值，必须先赋值才能使用**，否则编译报错。

| 类型 | 默认值 |
| ---- | ---- |
| `byte` `short` `int` | `0` |
| `long` | `0L` |
| `float` / `double` | `0.0f` / `0.0d` |
| `char` | 空字符（码点 0） |
| `boolean` | `false` |
| **所有引用类型** | `null` |

---

### 2. 基本数据类型（8 种）

| 分类 | 类型 | 占用空间 | 取值范围 / 说明 |
| ---- | ---- | -------- | --------------- |
| 整型 | `byte` | 1 字节 | -128 ~ 127 |
| | `short` | 2 字节 | -2¹⁵ ~ 2¹⁵-1 |
| | ⭐`int` | 4 字节 | -2³¹ ~ 2³¹-1（约 **21 亿**） |
| | `long` | 8 字节 | -2⁶³ ~ 2⁶³-1 |
| 浮点型 | `float` | 4 字节 | 约 7 位有效数字 |
| | ⭐`double` | 8 字节 | 约 15~16 位有效数字 |
| 字符型 | `char` | 2 字节 | 0 ~ 65535，UTF-16 编码 |
| 布尔型 | `boolean` | 规范未定义 | 只有 `true`/`false`，不能与其他类型互转 |

> `boolean` 的大小 JVM 规范并没有规定：HotSpot 中**单独的 `boolean` 按 `int` 处理，占 4 字节**；而 `boolean[]` 数组里**每个元素占 1 字节**。偶尔会被问到。

选型只需记三条：**整数默认 `int`，超 21 亿用 `long`（时间戳、雪花 ID）；小数默认 `double`；金额一律 `BigDecimal`**。

**字面量的默认类型**：整数字面量是 `int`，小数字面量是 `double`，所以：

```java
long num = 10000000000L;   // 必须加 L，否则字面量先按 int 解析就已经溢出
float f = 3.14F;           // 必须加 F，否则是 double，无法直接赋给 float

char c1 = 'A';             // char 用单引号，双引号是 String
char c2 = '中';
char c3 = '\u0041';        // Unicode 写法，即 'A'
```

**常用转义字符**：`\n` 换行、`\t` 制表符、`\"` 双引号、`\\` 反斜杠，以及上面那种 4 位十六进制的 Unicode 写法。

---

### 3. 基本类型 vs 引用类型（高频）

| 对比项 | 基本类型 | 引用类型 |
| ------ | -------- | -------- |
| 存的内容 | 值本身 | 对象在堆中的地址 |
| 对象位置 | — | 实体永远在堆中 |
| 默认值 | `0` / `false` 等 | `null` |
| 比较方式 | `==` 比值 | `==` 比地址，`equals()` 比内容 |
| 能否为 null | 不能 | 能 —— NPE 的来源 |

```java
int a = 10;         // 变量里直接存 10
String s = "abc";   // 变量里存的是地址，指向堆/常量池中的 "abc"
```

> **方法传参永远是值传递**：基本类型传值的副本，引用类型传地址的副本 —— 所以方法内修改对象的属性会影响外部，但把形参重新指向新对象不会。

---

### 4. 企业开发规范

| 规范 | 说明 / 示例 |
| ---- | ---- |
| 小驼峰命名，见名知意 | `userName`、`orderId`、`createTime` |
| 英文命名，禁止拼音、禁止单字母 | ✘ `String mingzi;`、✘ `int a;` |
| 常量全大写下划线 | `static final int MAX_SIZE = 100;` |
| 声明时直接初始化 | `int age = 20;` |
| 作用域最小化 | 用在哪个方法就在哪个方法里定义 |
| `boolean` 字段不加 `is` 前缀 | 用 `deleted` 而非 `isDeleted`，避免序列化框架取错属性名 |

---

### 5. 面试重点

**Q1：局部变量和成员变量的区别？**

**位置**：成员变量定义在类中（实例变量在堆、静态变量随类），局部变量定义在方法中、存在栈里。

**生命周期**：成员变量随对象/类存亡，局部变量随方法调用结束销毁。

**初始化**：成员变量有默认值，局部变量必须手动初始化。

**Q2：`char` 为什么占 2 字节？能存中文吗？**

Java 用 UTF-16 编码，一个 `char` 是一个代码单元，占 2 字节。常用汉字能存；但 emoji、生僻字属于增补字符，需要 2 个 `char`（代理对）才能表示。

**Q3：Java 是值传递还是引用传递？**

**只有值传递**。传引用类型时传的是地址的拷贝，因此方法内可以修改对象内容，但重新给形参赋值不会影响调用方。

**Q4：`float` / `double` 为什么不能精确表示小数？**

IEEE 754 用二进制表示小数，`0.1` 这类十进制小数换算成二进制是无限循环，只能取近似值。金额计算必须用 `BigDecimal`。

---

## 三、基本数据类型转换

### 1. 转换规则

`boolean` 不参与转换，其余 7 种沿箭头方向（小 → 大）**自动转换**，逆方向必须**强制转换**：

```text
byte → short → int → long → float → double
             ↑
           char
```

| 对比项 | 自动转换（隐式） | 强制转换（显式） |
| ------ | ---------------- | ---------------- |
| 方向 | 小类型 → 大类型 | 大类型 → 小类型 |
| 写法 | 直接赋值 | `(目标类型) 变量` |
| 风险 | 一般不丢数据 | 精度丢失、数据溢出 |

```java
int num = 10;
double d = num;            // 自动 → 10.0

double price = 9.8;
int value = (int) price;   // 强制 → 9
```

---

### 2. 四个易错点（重点）

**① 强转直接截断，不四舍五入**

```java
(int) 3.99;     // 3，不是 4
(int) -3.99;    // -3，向零取整
Math.round(3.99);   // 4，要四舍五入用这个
```

**② 溢出不报错，静默出错**

```java
byte b = (byte) 128;             // -128
int  i = (int) 10000000000L;     // 1410065408，只保留低 32 位，数据已经废了
```

**③ `byte`/`short`/`char` 参与运算会自动提升为 `int`**

```java
byte a = 1, b = 2;
byte c = a + b;            // ✘ 编译错误：a + b 的结果是 int
byte c = (byte) (a + b);   // ✔
```

**④ 整数表达式的中间结果也会溢出**

```java
long ms = 1000 * 60 * 60 * 24 * 30;    // ✘ 右边全是 int，先溢出再赋给 long
long ms = 1000L * 60 * 60 * 24 * 30;   // ✔ 第一个操作数写成 long，后面全部提升
```

> ④ 是工作中真正会踩的坑：算毫秒数、字节数、总金额时，只要中间结果可能超过 21 亿，就必须让第一个操作数是 `long`。

---

### 3. char 与 int

`char` 本质是一个整数（Unicode 码点）：

```java
int code = 'A';                // 65，自动转换
char c1 = (char) 66;           // 'B'，强制转换
char c2 = (char) ('a' + 1);    // 'b'，'a' + 1 的结果是 int，必须强转回来
```

---

### 4. 面试重点

**Q1：`short s = 1; s = s + 1;` 和 `s += 1;` 有什么区别？**

前者编译报错，因为 `s + 1` 会提升为 `int`；后者能通过，复合赋值运算符隐含强制类型转换，等价于 `s = (short) (s + 1)`。

**Q2：`int i = (int) 10000000000L` 会发生什么？**

不报错也不抛异常，直接截取低 32 位得到错误结果 `1410065408` —— 这是强转最危险的地方，**强转前必须自己确认取值范围**。

**Q3：为什么 `byte b1 = 1, b2 = 2; byte b3 = b1 + b2;` 编译不通过，但 `byte b = 1 + 2;` 可以？**

`b1 + b2` 是变量运算，结果提升为 `int`；`1 + 2` 是编译期常量，编译器会做常量折叠并检查结果 `3` 在 `byte` 范围内，因此允许隐式窄化。

---

## 四、运算符

### 1. 分类总览

| 分类 | 运算符 | 说明 |
| ---- | ------ | ---- |
| 算术 | `+` `-` `*` `/` `%` `++` `--` | 整数除法会截断小数 |
| 赋值 | `=` `+=` `-=` `*=` `/=` `%=` | 复合赋值隐含强制类型转换 |
| 比较 | `>` `<` `>=` `<=` `==` `!=` | 结果是 `boolean` |
| 逻辑 | `&&` `\|\|` `!` | 短路运算 |
| 位运算 | `&` `\|` `^` `~` `<<` `>>` `>>>` | 按二进制位操作 |
| 三元 | `? :` | Java 中唯一的三目运算符 |

---

### 2. 算术运算符

**整数除法截断，除数为 0 抛异常**

```java
5 / 2        // 2，不是 2.5
5 / 2.0      // 2.5，有一个操作数是浮点就按浮点算
5 % 2        // 1
-5 % 2       // -1，取模结果的符号跟随被除数

int a = 1 / 0;         // ✘ 抛 ArithmeticException: / by zero
double d = 1.0 / 0;    // ✔ Infinity，浮点除零不抛异常
0.0 / 0.0              // NaN，且 NaN != NaN 恒成立
```

**浮点运算有精度误差**

```java
0.1 + 0.2            // 0.30000000000000004
0.1 + 0.2 == 0.3     // false
```

> 金额、比例一律用 `BigDecimal`，且**必须用字符串构造** —— `new BigDecimal("0.1")` 才精确，`new BigDecimal(0.1)` 照样有误差。判断浮点数相等用误差范围 `Math.abs(a - b) < 1e-6`。

**`i++` 与 `++i`**

```java
int i = 0;
int a = i++;   // a = 0，i = 1  先取值后自增
int b = ++i;   // b = 2，i = 2  先自增后取值

int j = 0;
j = j++;       // ✘ j 仍然是 0：旧值 0 先被暂存，自增后又被这个旧值覆盖回去
```

**复合赋值隐含强制类型转换**

```java
byte b = 10;
b = b + 1;   // ✘ 编译错误：b + 1 提升为 int
b += 1;      // ✔ 等价于 b = (byte) (b + 1)，注意可能悄悄溢出
```

---

### 3. 比较运算符：`==` 与 `equals()`

| 比较对象 | 用什么 | 说明 |
| -------- | ------ | ---- |
| 基本数据类型 | `==` | 比较值 |
| 引用数据类型 | `equals()` | `==` 比较的是地址，即是否为同一个对象 |

```java
String s1 = "abc";
String s2 = new String("abc");
s1 == s2;         // false，不是同一个对象
s1.equals(s2);    // true，内容相同

Integer a = 127, b = 127;
Integer c = 128, d = 128;
a == b;   // true，-128 ~ 127 走 Integer 缓存池
c == d;   // false，超出缓存范围，是两个新对象
```

> 企业规范：包装类型一律用 `equals()` 比较。常量写在前面可避免 NPE —— `"admin".equals(role)`，或直接用 `Objects.equals(a, b)`。

---

### 4. 逻辑运算符与短路

`&&`、`||` 是**短路**运算：左边已能确定结果就不再计算右边。`&`、`|` 不短路，两边都会算，实际开发几乎不用。

短路最主要的用途是**空值防护**：

```java
if (user != null && user.getAge() > 18) { }    // ✔ user 为 null 时右边根本不执行
if (user.getAge() > 18 && user != null) { }    // ✘ 顺序写反，直接 NPE

if (list == null || list.isEmpty()) { return; }   // ✔ 判空的标准写法
```

---

### 5. 位运算符

| 运算符 | 含义 | 示例 |
| ------ | ---- | ---- |
| `&` | 按位与 | `6 & 3` → `2` |
| `\|` | 按位或 | `6 \| 3` → `7` |
| `^` | 按位异或 | `6 ^ 3` → `5` |
| `~` | 按位取反 | `~6` → `-7` |
| `<<` | 左移，相当于 ×2ⁿ | `6 << 1` → `12` |
| `>>` | 右移，补符号位，相当于 ÷2ⁿ | `-8 >> 1` → `-4` |
| `>>>` | 无符号右移，补 0 | `-8 >>> 1` → `2147483644` |

位运算平时手写得少，但**框架源码里到处都是**，看得懂比写得出更重要：

```java
(n & 1) == 1              // 判断奇数，比 n % 2 快
(left + right) >>> 1      // 二分查找求中点，顺带避免 left + right 溢出

// 用位标记表示权限/状态，比一堆 boolean 字段省空间，框架源码里很常见
int READ = 1, WRITE = 1 << 1, EXEC = 1 << 2;
int perm = READ | WRITE;      // 授予读 + 写
(perm & WRITE) != 0;          // 判断是否有写权限
perm &= ~WRITE;               // 移除写权限
```

> `HashMap` 用 `(n - 1) & hash` 代替取模来定位桶，前提是容量为 2 的幂 —— 位运算比 `%` 快得多。

---

### 6. 三元运算符与优先级

```java
String level = score >= 60 ? "及格" : "不及格";
```

两个坑：

```java
// 1. 两个分支类型不同时会自动类型提升
Object o = true ? 1 : 2.0;    // 1.0，不是 1

// 2. 分支中出现包装类型会自动拆箱，null 会 NPE
Integer x = null;
int y = flag ? x : 0;         // flag 为 true 时抛 NullPointerException
```

优先级大致顺序：`!` > 算术 > 比较 > `&&` > `||` > 赋值。**不确定就加括号**：

```java
if (a & b == 0) { ... }     // ✘ 实际解析成 a & (b == 0)，编译错误
if ((a & b) == 0) { ... }   // ✔
```

> 三元嵌套超过一层就改用 `if-else`，可读性优先。

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 引用类型用 `equals()` | 推荐 `Objects.equals(a, b)`，同时避免 NPE |
| 金额、比例用 `BigDecimal` | 浮点有精度误差，且构造要传字符串 |
| 除法前判断除数是否为 0 | 避免 `ArithmeticException` |
| 判空条件写在 `&&` 左边 | 利用短路做空值防护 |
| 复杂表达式加括号 | 不依赖对优先级的记忆 |
| 并发下不要用 `i++` | 非原子操作，改用 `AtomicInteger` 或加锁 |

---

### 8. 面试重点

**Q1：`==` 和 `equals()` 的区别？**

`==` 比较基本类型的值、引用类型的地址；`equals()` 是 `Object` 的方法，默认实现也是比地址，`String`、包装类等重写后才比较内容。

**Q2：为什么 `Integer a = 128, b = 128;` 时 `a == b` 是 false？**

`Integer` 只缓存 -128 ~ 127（`IntegerCache`），超出范围自动装箱会创建新对象，`==` 比的是地址。

**Q3：`i++` 线程安全吗？**

不安全。`i++` 包含读取、加一、写回三步，并发下会丢更新，需要 `AtomicInteger` 或加锁。

**Q4：`>>` 和 `>>>` 的区别？**

`>>` 右移补符号位（负数补 1），`>>>` 无符号右移补 0，所以负数 `>>>` 之后会变成很大的正整数。

**Q5：`int j = 0; j = j++;` 之后 j 是多少？**

还是 `0`。`j++` 会先把原值 `0` 压栈，再对 `j` 自增为 1，最后把栈里的 `0` 赋回给 `j`。

---

## 五、流程控制（分支与循环）

### 1. 分支控制

#### if-else

```java
if (score >= 90) {
    level = "A";
} else if (score >= 60) {
    level = "B";
} else {
    level = "C";
}
```

**只有一行也必须写大括号** —— 否则后来的人加一行代码就会引入 bug：

```java
if (flag) doA();
          doB();     // ✘ 看着像在 if 里，实际无论如何都会执行
```

**用卫语句（Guard Clause）代替嵌套**，这是可读性差别最大的一条：

```java
// ✘ 嵌套地狱，主流程被推到最深处
public void handle(Order o) {
    if (o != null) {
        if (o.getStatus() == PAID) {
            if (o.getAmount() > 0) {
                doBusiness(o);
            }
        }
    }
}

// ✔ 不满足条件立刻 return，主流程始终在最外层
public void handle(Order o) {
    if (o == null)               return;
    if (o.getStatus() != PAID)   return;
    if (o.getAmount() <= 0)      return;

    doBusiness(o);
}
```

#### switch

支持的类型：`byte`、`short`、`int`、`char`、`String`（JDK 7+）、枚举。**不支持 `long`、`float`、`double`、`boolean`**。

传统写法的最大问题是 **case 穿透** —— 漏写 `break` 会继续执行下一个分支：

```java
switch (type) {
    case 1:
        doA();
        break;     // ✘ 漏写就会继续执行 case 2
    case 2:
        doB();
        break;
    default:
        doC();
}
```

⭐ **switch 表达式（JDK 14+），现在的首选**：箭头写法不会穿透、不用写 `break`，还能直接返回值。

```java
String desc = switch (status) {
    case "PAID"      -> "已支付";
    case "REFUNDED"  -> "已退款";
    case "A", "B"    -> "多个值合并写";
    default          -> "未知";
};

// 需要多行逻辑时，用 yield 返回值
int score = switch (level) {
    case "A" -> 100;
    default  -> {
        log.warn("未知等级 {}", level);
        yield 0;
    }
};
```

两个注意点：

- **switch 的条件为 `null` 会抛 NPE**（`String`、包装类、枚举都一样），要先判空
- 用作**表达式**（等号右边）时必须覆盖所有分支，否则编译不通过

> 分支超过 5 个、且是纯粹的"值 → 值"映射时，改用 `Map` 或策略模式，比堆一长串 `case` 好维护得多。

---

### 2. 三种基本循环

| 循环 | 适用场景 | 特点 |
| ---- | -------- | ---- |
| `for` | 循环次数明确 | 计数器作用域仅限循环内 |
| `while` | 次数不定，先判断 | 可能一次都不执行 |
| `do-while` | 次数不定，至少执行一次 | 先执行后判断，实际用得少 |

```java
for (int i = 0; i < 10; i++) { }   // i 出了循环就不可见，优先用这种

int i = 0;
while (i < 10) { i++; }            // i 在循环外依然占着作用域
```

---

### 3. 增强 for（for-each）

```java
for (String name : list) {
    System.out.println(name);
}
```

底层是语法糖：**数组**编译成下标循环，**集合**编译成 `Iterator` 遍历。

三个限制：

- 拿不到索引
- 只能读，给循环变量重新赋值不会影响原集合
- 遍历过程中增删元素会抛 `ConcurrentModificationException`

```java
for (String s : list) {
    if ("b".equals(s)) list.remove(s);   // ✘ ConcurrentModificationException
}
```

---

### 4. 遍历中删除元素

```java
// 把 list 中所有等于 "b" 的元素删除掉。

// ✔ 方式一：removeIf()，JDK 8+，最简洁
list.removeIf(s -> "b".equals(s));

// ✔ 方式二：Iterator.remove()
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if ("b".equals(it.next())) it.remove();
}

// ✔ 方式三：倒序下标遍历，删除后不影响尚未遍历的下标
for (int i = list.size() - 1; i >= 0; i--) {
    if ("b".equals(list.get(i))) list.remove(i);
}
```

> 原理：集合内部用 `modCount` 记录结构性修改次数，迭代器创建时记下 `expectedModCount`，每次 `next()` 都校验两者是否相等。直接调 `list.remove()` 只改了 `modCount`，所以抛异常；`Iterator.remove()` 会同步更新两个值。

---

### 5. 遍历 Map

```java
// ✔ 推荐：一次拿到 key 和 value
for (Map.Entry<String, Integer> e : map.entrySet()) {
    e.getKey();
    e.getValue();
}

// ✘ 不推荐：每次 get 都要重新做一次哈希查找
for (String key : map.keySet()) {
    map.get(key);
}
```

---

### 6. break、continue 与标签

```java
break;      // 结束整个循环
continue;   // 跳过本次，进入下一次

// 跳出多重循环用标签，比设 flag 变量清晰
outer:
for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
        if (matrix[i][j] == target) break outer;
    }
}
```

---

### 7. 循环里的性能坑（重点）

**循环内拼接字符串**

```java
String s = "";
for (String x : list) s += x;          // ✘ 每次都新建 String 对象，O(n²)

StringBuilder sb = new StringBuilder();
for (String x : list) sb.append(x);    // ✔
```

**循环内查库 / 调远程接口（N+1 问题）**

```java
for (Long id : ids) {
    userMapper.selectById(id);         // ✘ 100 个 id 就是 100 次网络往返
}

List<User> users = userMapper.selectBatchIds(ids);   // ✔ 一次批量查询
```

**嵌套循环做匹配**

```java
for (Order o : orders)                 // ✘ O(n×m)
    for (User u : users)
        if (o.getUserId().equals(u.getId())) { ... }

Map<Long, User> userMap = users.stream()             // ✔ O(n+m)
        .collect(Collectors.toMap(User::getId, u -> u));
for (Order o : orders) userMap.get(o.getUserId());
```

**循环内重复创建对象**

```java
for (String s : list) {
    Pattern p = Pattern.compile("\\d+");                       // ✘ 每次都重新编译正则
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd"); // ✘ 每次都新建
}
```

> 正则的 `Pattern` 应提为静态常量复用；日期格式化用线程安全的 `DateTimeFormatter`，不要用 `SimpleDateFormat`。

---

### 8. forEach 与 Stream（JDK 8+）

```java
list.forEach(System.out::println);
list.stream().filter(u -> u.getAge() > 18).forEach(...);
```

注意：

- lambda 里**不能 `break` / `continue`**，`return` 只相当于 `continue`
- 需要提前终止改用 `anyMatch()`、`findFirst()`
- 简单遍历用 for-each 更直观、断点也好打；多级过滤、分组、聚合才用 Stream

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 优先用 for-each | 只在需要索引或倒序删除时才用下标循环 |
| 遍历中删除用 `removeIf()` | 直接 `list.remove()` 会抛 CME |
| 遍历 Map 用 `entrySet()` | 避免二次哈希查找 |
| 循环内禁止 IO / 远程调用 | 改批量接口，杜绝 N+1 |
| 循环内拼接字符串用 `StringBuilder` | 避免 O(n²) |
| 嵌套不超过两层 | 更深的嵌套改用 Map 预先建索引 |
| 循环变量作用域最小化 | 能声明在 `for` 里就不要提到外面 |
| `if` 必须写大括号 | 单行也要写，防止后续加行出 bug |
| 用卫语句代替嵌套 | 嵌套层数控制在 2 层以内 |
| `switch` 优先用箭头写法 | 不会穿透，也不用写 `break` |

---

### 10. 面试重点

**Q1：`switch` 支持哪些类型？**

`byte`、`short`、`int`、`char`、`String`（JDK 7+）、枚举。不支持 `long`、`float`、`double`、`boolean`。底层对 `String` 是先 `switch(hashCode)` 再用 `equals()` 二次校验。

**Q2：`for` 和 for-each 有什么区别？**

for-each 是语法糖，数组编译成下标循环，集合编译成 `Iterator` 遍历；缺点是拿不到索引，且不能在遍历中增删元素。

**Q3：为什么遍历时删除元素会抛 `ConcurrentModificationException`？**

集合用 `modCount` 记录结构性修改次数，迭代器创建时记下 `expectedModCount`，每次 `next()` 校验两者是否一致。直接 `list.remove()` 只改 `modCount`，导致校验失败。

**Q4：如何正确地在遍历中删除元素？**

`removeIf()`、`Iterator.remove()`，或倒序下标遍历。

**Q5：Stream 的 `forEach` 里能 `break` 吗？**

不能。需要提前终止就改用 `anyMatch()`、`findFirst()`，或 JDK 9+ 的 `takeWhile()`。

---

## 六、字符串

### 1. String 的本质与不可变性

`String` 是**引用数据类型**，底层用一个数组保存字符：

```java
// JDK 8
private final char[] value;

// JDK 9+（紧凑字符串 Compact Strings）
private final byte[] value;
private final byte coder;   // 0 = LATIN1（1 字节/字符），1 = UTF16（2 字节/字符）
```

> JDK 9 之后纯 ASCII 字符串每个字符只占 1 字节，内存直接省一半，这是升级 JDK 9+ 白捡的收益。

**不可变（Immutable）**：`String` 被 `final` 修饰不可继承，内部数组也是 `final` 且不对外暴露。所有看起来"修改"字符串的方法，实际都是**返回一个新对象**：

```java
String s = "hello";
s.toUpperCase();              // ✘ 毫无效果，返回值被丢弃了
System.out.println(s);        // hello

s = s.toUpperCase();          // ✔ 必须重新接收返回值
System.out.println(s);        // HELLO
```

为什么要设计成不可变：

| 原因 | 说明 |
| ---- | ---- |
| 常量池可共享 | 内容相同的字面量共用一个对象，省内存 |
| 线程安全 | 不可变对象天然线程安全，多线程共享无需加锁 |
| hashCode 可缓存 | 内部缓存 `hash` 字段，只算一次，作 `HashMap` 的 key 性能极好 |
| 安全性 | 类名、URL、数据库连接串、文件路径作参数传递时不会被中途篡改 |

---

### 2. 字符串常量池

```java
String s1 = "abc";                // 字面量，编译期就进常量池
String s2 = "abc";                // 复用池中已有对象
String s3 = new String("abc");    // 堆中新建对象，内容指向池中字符
String s4 = s3.intern();          // 手动入池，返回池中引用

s1 == s2;   // true
s1 == s3;   // false ← 经典面试题
s1 == s4;   // true
```

**编译期常量折叠**：拼接的双方如果都是编译期常量，编译器会直接算好结果当字面量处理。

```java
String a = "he" + "llo";      // 编译后就是 "hello"
"hello" == a;                 // true

String b = "he";
String c = b + "llo";         // 运行期拼接，堆中新对象
"hello" == c;                 // false

final String d = "he";        // final 局部变量是编译期常量
String e = d + "llo";         // 仍然折叠
"hello" == e;                 // true
```

> JDK 7 起常量池从方法区（永久代）移到了堆中，`intern()` 不再复制对象，而是把堆中已有对象的引用登记进池。
>
> 实际开发中几乎不会手动调 `intern()`，掌握原理是为了应付面试、以及排查 `==` 比较字符串导致的诡异 bug。

---

### 3. 常用 API

```java
String s1 = "hello";                                     // ⭐ 首选
String s2 = new String("hello");                         // ✘ 多余，白白多创建一个对象
String s3 = String.valueOf(123);                         // ⭐ 任意类型转 String，传 null 也不抛异常
String s4 = new String(bytes, StandardCharsets.UTF_8);   // 字节数组转字符串，必须显式指定字符集
```

| 用途 | 方法 | 说明 / 示例 |
| ---- | ---- | ---- |
| 长度判空 | ⭐`length()` | 字符个数 |
| | `isEmpty()` | 长度是否为 0 —— `"  ".isEmpty()` 是 `false` |
| | ⭐`isBlank()` | 空或全空白 —— `"  ".isBlank()` 是 `true`（11+） |
| 比较 | ⭐`equals()` / `equalsIgnoreCase()` | 比内容 / 忽略大小写比内容 |
| | `compareTo()` | 字典序，用于排序 |
| 查找 | ⭐`contains(s)` | `"hello".contains("ell")` → `true` |
| | `indexOf(s)` / `lastIndexOf(s)` | 返回下标，不存在返回 `-1` |
| | `startsWith` / `endsWith` | `"a.txt".endsWith(".txt")` → `true` |
| | `charAt(i)` | `"hello".charAt(0)` → `'h'` |
| 截取 | ⭐`substring(b[, e])` | **左闭右开** —— `"hello".substring(1, 3)` → `"el"` |
| 替换 | ⭐`replace(a, b)` | **字面量**替换全部 |
| | `replaceAll(re, b)` / `replaceFirst(re, b)` | **正则**替换 |
| 去空白 | `trim()` / ⭐`strip()` | `strip()` 是 Unicode 感知版，能去全角空格（11+） |
| 大小写 | `toUpperCase()` / `toLowerCase()` | — |
| 格式化 | `String.format("%s-%d", a, b)` | `%s` 字符串、`%d` 整数、`%.2f` 保留两位小数 |
| 其他 | `toCharArray()`、`matches(re)`、`repeat(n)`（11+） | — |

**拆分与拼接**：

```java
String[] arr = "a,b,c".split(",");                  // ⭐ 参数是正则，坑见第 7 节
String s1 = String.join(",", list);                 // ⭐ 集合拼接，最常用

list.stream().map(User::getName)                    // ⭐ 流式拼接
    .collect(Collectors.joining(", ", "[", "]"));   // "[张三, 李四]"
```

---

### 4. 字符串拼接的三种方式

```java
// 1. + 号：可读性最好，用在单次拼接
String s = "Hello, " + name + "!";

// 2. StringBuilder：⭐ 循环 / 分支中反复拼接必须用它
StringBuilder sb = new StringBuilder();
for (String x : list) sb.append(x).append(",");

// 3. String.join / Collectors.joining：⭐ 拼接集合首选，自动处理分隔符
String s2 = String.join(",", list);
```

**`+` 的编译结果**（面试高频）：JDK 8 编译成 `new StringBuilder().append(a).append(b).toString()`；JDK 9+ 改用 `invokedynamic` 调用 `StringConcatFactory` 动态生成拼接代码，更快。

但无论哪个版本，**循环里用 `+` 都是灾难** —— 每次迭代都新建一个 `StringBuilder` 并把已有内容整体拷贝一遍，复杂度 O(n²)：

```java
String s = "";
for (String x : list) s += x;          // ✘ 一万条数据能卡几秒

StringBuilder sb = new StringBuilder();
for (String x : list) sb.append(x);    // ✔
```

> 日志同理：用占位符而不是拼接 —— `log.info("userId={}", id)`，日志级别未开启时根本不会执行拼接。

---

### 5. String / StringBuilder / StringBuffer

| 对比项 | `String` | `StringBuilder` | `StringBuffer` |
| ------ | -------- | --------------- | -------------- |
| 可变性 | 不可变 | 可变 | 可变 |
| 线程安全 | 安全（因为不可变） | **不安全** | 安全（方法加 `synchronized`） |
| 性能 | 拼接时最差 | ⭐ 最快 | 比 StringBuilder 慢 |
| 使用场景 | 内容基本不变 | ⭐ 单线程内拼接，绝大多数场景 | 几乎不用 |

> `StringBuffer` 是 JDK 1.0 的遗留类，除了维护老代码，现在基本不会主动选它：真需要多线程共享一个可变字符串，往往说明设计本身有问题；正确做法是每个线程各用各的 `StringBuilder`，最后再汇总。**默认无脑选 `StringBuilder`**。

---

### 6. StringBuilder 常用方法

```java
StringBuilder sb1 = new StringBuilder();          // 空内容，默认容量 16
StringBuilder sb2 = new StringBuilder("Hello");   // 指定初始内容
StringBuilder sb3 = new StringBuilder(100);       // ⭐ 已知大致长度时指定容量，避免反复扩容拷贝
```

| 方法 | 说明 | 示例（初始 `"Hello"`） |
| ---- | ---- | ---- |
| ⭐`append(x)` | 追加，**返回自身**可链式调用 | `sb.append(" World")` → `Hello World` |
| ⭐`toString()` | 转为 `String` | — |
| `insert(i, x)` | 在下标 i 处插入 | `sb.insert(5, "!")` → `Hello!` |
| `delete(b, e)` | 删除区间 `[b, e)` | `sb.delete(1, 3)` → `Hlo` |
| `deleteCharAt(i)` | 删除单个字符 | `sb.deleteCharAt(1)` → `Hllo` |
| `replace(b, e, x)` | 替换区间，长度可不等 | `sb.replace(0, 5, "Java")` → `Java` |
| `reverse()` | 反转 | `sb.reverse()` → `olleH` |
| `charAt(i)` / `setCharAt(i, c)` | 读 / 写单个字符 | — |
| `length()` | 长度 | — |
| `setLength(0)` | ⭐ 清空并复用对象 | — |

`append()` 返回自身，因此可以链式调用：

```java
String sql = new StringBuilder()
        .append("SELECT * FROM user")
        .append(" WHERE age > ").append(18)
        .append(" ORDER BY id")
        .toString();
```

---

### 7. 高频踩坑点（重点）

**① `split()` 的参数是正则，且默认丢弃尾部空串**

```java
"192.168.1.1".split(".");        // ✘ 返回长度为 0 的数组！. 在正则里匹配任意字符
"192.168.1.1".split("\\.");      // ✔ [192, 168, 1, 1]

"a,b,,,".split(",");             // [a, b]         ← 尾部空串被丢掉了
"a,b,,,".split(",", -1);         // ✔ [a, b, , , ]  limit 传 -1 才全保留
```

> 需要转义的正则元字符：`. | * + ? ^ $ ( ) [ ] { } \`。不确定就用 `Pattern.quote(".")`。

**② `replace()` 是字面量替换，`replaceAll()` 是正则替换**

```java
"a.b.c".replace(".", "-");       // ✔ a-b-c
"a.b.c".replaceAll(".", "-");    // ✘ ----- 每个字符都被替换了
```

**③ `substring()` 越界抛异常**

```java
"abc".substring(1, 10);          // ✘ StringIndexOutOfBoundsException
```

> 截取前先判 `length()`，或用 `StringUtils.substring()`（越界自动截到末尾，不抛异常）。

**④ `equals` 把常量写在前面防 NPE**

```java
role.equals("admin");            // ✘ role 为 null 时 NPE
"admin".equals(role);            // ✔
Objects.equals(role, "admin");   // ✔ 更推荐，两边都可能为 null 时也安全
```

**⑤ `length()` 数的是 char，不是"人眼看到的字符"**

```java
"中文".length();                          // 2  ✔ 常用汉字在 UTF-16 里是 1 个 char
"👍".length();                            // 2  ✘ emoji 是代理对，占 2 个 char
"👍".codePointCount(0, "👍".length());     // ✔ 1
```

> 数据库字段长度校验、短信字数统计遇到 emoji 会出问题，改用 `codePointCount()`。

**⑥ 字符相加是数值相加**

```java
System.out.println('a' + 'b');        // 195 ← char 提升为 int
System.out.println("" + 'a' + 'b');   // ab  ← 有 String 参与才是拼接
```

**⑦ `switch` 支持 String（JDK 7+）**

```java
switch (status) {
    case "PAID"     -> handlePaid();       // JDK 14+ 箭头写法，不用写 break
    case "REFUNDED" -> handleRefund();
    default         -> throw new IllegalArgumentException(status);
}
```

> 底层是先对 `hashCode()` 做 switch，再用 `equals()` 二次校验排除哈希冲突。`status` 为 null 会 NPE（JDK 21 的模式匹配 switch 才支持 `case null`）。

**⑧ 密码不要用 String 存**

`String` 不可变，用完无法主动清除，会一直留在内存/常量池里直到被 GC，堆转储（heap dump）时存在泄露风险。安全场景用 `char[]`，用完 `Arrays.fill(arr, '0')` 覆写 —— 这也是 `JPasswordField.getPassword()` 返回 `char[]` 的原因。

---

### 8. 文本块（Text Block，JDK 15+）

写多行 SQL、JSON 时不用再拼 `\n` 和转义引号：

```java
String json = """
        {
            "name": "张三",
            "age": 18
        }
        """;
```

> 结尾 `"""` 的缩进位置决定公共缩进量，这部分缩进会被自动剥离。

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 比较内容一律用 `equals()` | 推荐 `Objects.equals(a, b)`，两边都防 NPE |
| 判空用 `StringUtils.isBlank()` | 一次覆盖 `null`、`""`、全空格三种情况 |
| 循环内拼接用 `StringBuilder` | `+` 在循环里是 O(n²) |
| 拼接集合用 `String.join()` / `Collectors.joining()` | 不用手动处理末尾分隔符 |
| 日志用占位符不用 `+` | `log.info("id={}", id)` |
| `split()` 记得转义、`limit` 传 `-1` | `.`、`\|` 等是正则元字符 |
| 字节数组转换显式指定字符集 | 用 `StandardCharsets.UTF_8`，不依赖平台默认编码 |
| 常量字符串定义为 `static final` | 避免散落的魔法值 |
| 不用 `new String("x")` | 直接写字面量，避免多余对象 |

**工具类**：判空、截取、脱敏直接用现成工具，不要自己造轮子。

```java
// Apache Commons Lang3（最通用，Spring Boot 项目基本都有）
StringUtils.isBlank(s);                // null / "" / "   " 都返回 true
StringUtils.defaultIfBlank(s, "默认值");
StringUtils.substring(s, 0, 10);       // 越界不抛异常

// Spring 自带
org.springframework.util.StringUtils.hasText(s);
```

---

### 10. 面试重点

**Q1：String 为什么是不可变的？有什么好处？**

`String` 类被 `final` 修饰，内部 `value` 数组也是 `final` 且不对外暴露。好处：常量池可共享省内存、天然线程安全、hashCode 可缓存（适合做 HashMap 的 key）、作为参数传递时不会被篡改。

**Q2：`String s = new String("abc")` 创建了几个对象？**

1 个或 2 个。堆中必然新建 1 个 `String` 对象；如果常量池中还没有 `"abc"`，则还要在池中创建 1 个，共 2 个；已存在则只创建 1 个。

**Q3：String、StringBuilder、StringBuffer 的区别？**

`String` 不可变，拼接会不断产生新对象；`StringBuilder` 可变、线程不安全但最快，是单线程拼接的首选；`StringBuffer` 可变、方法加了 `synchronized` 所以线程安全但更慢，现在基本已被淘汰。

**Q4：`String s = "a" + "b" + "c"` 效率低吗？**

不低。三个都是编译期常量，编译器会直接折叠成 `"abc"`，运行期没有任何拼接开销。真正低效的是**循环内**或**有变量参与**的拼接。

**Q5：`intern()` 方法的作用？**

把字符串登记进常量池并返回池中引用。JDK 7 之前常量池在永久代，`intern()` 会复制一份对象进去；JDK 7 起常量池移到堆中，`intern()` 只登记堆中已有对象的引用。

**Q6：为什么密码建议用 `char[]` 而不是 `String`？**

`String` 不可变，用完无法主动清空，会留在内存和常量池中直到被 GC，堆转储时存在泄露风险；`char[]` 用完可以用 `Arrays.fill()` 覆写掉。

---

## 七、数组

### 1. 声明与初始化

```java
int[] arr1 = new int[5];             // ⭐ 动态初始化：只给长度，元素为默认值 0
int[] arr2 = {1, 2, 3};              // ⭐ 静态初始化：只能在声明时这样写
int[] arr3 = new int[]{1, 2, 3};     // 完整写法，赋值给已有变量或直接传参时用
```

三个基本事实：

- 取长度用 **`arr.length`**（属性，没有括号）—— `String` 是 `s.length()`，集合是 `list.size()`
- 数组一旦创建，**长度不可变**；所谓"扩容"都是新建一个数组再把数据拷过去
- 元素默认值和成员变量一致：数值 `0`、`boolean` `false`、**引用类型 `null`**

```java
String[] names = new String[3];
names[0].length();    // ✘ NPE，元素默认是 null，不是空字符串
```

---

### 2. Arrays 工具类

数组本身几乎没有方法，所有操作都靠 `java.util.Arrays`：

| 方法 | 说明 |
| ---- | ---- |
| ⭐`Arrays.toString(arr)` | 打印数组内容 |
| `Arrays.deepToString(arr)` | 打印二维数组 |
| ⭐`Arrays.sort(arr)` | 升序排序 |
| `Arrays.sort(arr, cmp)` | 自定义排序，**只能用于对象数组** |
| ⭐`Arrays.copyOf(arr, n)` | 复制并指定新长度（扩容 / 截断） |
| `Arrays.copyOfRange(arr, b, e)` | 区间复制，**左闭右开** |
| ⭐`Arrays.fill(arr, v)` | 全部填充为某个值 |
| `Arrays.binarySearch(arr, k)` | 二分查找，**必须先排好序** |
| ⭐`Arrays.equals(a, b)` | 比较内容是否相同 |
| `Arrays.asList(arr)` | 转成 List —— **有坑，见第 5 节** |
| `Arrays.stream(arr)` | 转成 Stream |

```java
int[] arr = {3, 1, 2};

System.out.println(arr);                    // ✘ [I@1b6d3586，打印的是地址
System.out.println(Arrays.toString(arr));   // ✔ [3, 1, 2]

Arrays.sort(arr);                           // [1, 2, 3]

// 基本类型数组要降序，只能转成包装类型，或者先升序再反转
Integer[] box = {3, 1, 2};
Arrays.sort(box, Comparator.reverseOrder());   // [3, 2, 1]
```

---

### 3. 数组的复制

```java
int[] b = Arrays.copyOf(a, a.length);        // ⭐ 首选，可读性最好
int[] c = a.clone();                         // 等价，写法更短
System.arraycopy(a, 0, b, 0, a.length);      // 底层实现，需要控制起止位置时用

int[] d = a;                                 // ✘ 这不是复制！只是多了一个指向同一数组的引用
```

> 三种复制方式**都是浅拷贝**：对象数组复制后，新旧两个数组里的元素指向的是同一批对象，改其中一个对象的属性，另一个数组也会"跟着变"。

---

### 4. 二维数组

```java
int[][] a = new int[3][4];          // 3 行 4 列
int[][] b = {{1, 2}, {3, 4, 5}};    // 不规则数组，每行长度可以不同

a.length;        // 3，行数
a[0].length;     // 4，第 0 行的列数

for (int[] row : a) {
    for (int v : row) { }
}
```

> 本质是**"数组的数组"** —— 外层数组的每个元素是一个一维数组的引用，行与行在内存中并不连续。

---

### 5. 高频踩坑点（重点）

**① `Arrays.asList()` 返回的不是 `ArrayList`**

```java
List<String> list = Arrays.asList("a", "b");
list.set(0, "x");   // ✔ 可以改
list.add("c");      // ✘ UnsupportedOperationException
list.remove(0);     // ✘ UnsupportedOperationException
```

它返回的是 `Arrays` 的一个内部类（同名叫 `ArrayList`，但不是 `java.util.ArrayList`），本质是**原数组的固定长度视图**，只能改不能增删。要可变集合必须再包一层：

```java
List<String> list = new ArrayList<>(Arrays.asList("a", "b"));   // ✔
List<String> list = new ArrayList<>(List.of("a", "b"));         // ✔ JDK 9+
```

**② `Arrays.asList()` 传基本类型数组会只得到 1 个元素**

```java
int[] arr = {1, 2, 3};
List<int[]> l1 = Arrays.asList(arr);      // ✘ size = 1，整个数组成了唯一的元素

Integer[] box = {1, 2, 3};
List<Integer> l2 = Arrays.asList(box);    // ✔ size = 3

List<Integer> l3 = Arrays.stream(arr)     // ✔ 从 int[] 转 List 的正确写法
        .boxed().collect(Collectors.toList());
```

> 原因：泛型不支持基本类型，`int[]` 整体被当成了一个 `T`。

**③ 数组比较内容不能用 `==` 或 `equals()`**

```java
int[] a = {1, 2}, b = {1, 2};

a == b;                    // false，比的是地址
a.equals(b);               // false，数组没重写 equals，本质还是 ==
Arrays.equals(a, b);       // ✔ true
Arrays.deepEquals(x, y);   // ✔ 二维数组用这个
```

**④ 越界与空数组**

```java
arr[arr.length];      // ✘ ArrayIndexOutOfBoundsException，下标从 0 到 length-1

int[] arr = null;
arr.length;           // ✘ NPE
```

> 企业规范：方法返回数组或集合时，**没有数据就返回空数组 / 空集合，绝不返回 `null`**，让调用方不必判空。

**⑤ `length` / `length()` / `size()` 别记混**

数组用属性 `arr.length`，`String` 用方法 `s.length()`，集合用方法 `list.size()`。

---

### 6. 数组 vs ArrayList

| 对比项 | 数组 | `ArrayList` |
| ------ | ---- | ----------- |
| 长度 | 创建后固定 | 自动扩容 |
| 元素类型 | 基本类型、对象都可以 | 只能是对象（基本类型自动装箱） |
| 取长度 | `length`（属性） | `size()`（方法） |
| 增删元素 | 只能自己新建数组重拷 | `add()` / `remove()` |
| 性能与内存 | 无装箱，内存紧凑，更快 | 有装箱和扩容开销 |
| 使用场景 | 长度固定、性能敏感、底层实现 | ⭐ 业务代码默认用它 |

> `ArrayList` 内部就是用数组实现的（`Object[] elementData`），扩容时新建 1.5 倍长度的数组再 `Arrays.copyOf` 拷过去 —— 这也是**集合章的高频面试题**。

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 业务代码优先用 `List` | 数组只在长度固定或性能敏感处用 |
| 声明写 `int[] arr` | 不写 C 风格的 `int arr[]` |
| 打印用 `Arrays.toString()` | 直接打印是地址 |
| 比较内容用 `Arrays.equals()` | `==` 和 `equals()` 都是比地址 |
| 返回空数组不返回 `null` | 调用方免判空 |
| `Arrays.asList()` 的结果不要增删 | 要可变就 `new ArrayList<>(...)` 包一层 |
| 遍历前判空 | `arr != null && arr.length > 0` |

---

### 8. 面试重点

**Q1：数组和 `ArrayList` 的区别？**

数组长度固定、能存基本类型、用 `length` 取长度、无装箱开销；`ArrayList` 长度可自动扩容、只能存对象、用 `size()` 取长度，底层其实就是数组，扩容时新建 1.5 倍数组再拷贝。业务代码默认用 `ArrayList`。

**Q2：`Arrays.asList()` 返回的 List 能 `add()` 吗？为什么？**

不能，会抛 `UnsupportedOperationException`。它返回的是 `Arrays` 的内部类，只是原数组的一个固定长度视图，没有实现增删方法。需要可变集合要 `new ArrayList<>(Arrays.asList(...))`。

**Q3：怎么判断两个数组内容相同？**

用 `Arrays.equals()`；二维数组用 `Arrays.deepEquals()`。`==` 和数组默认的 `equals()` 比的都是地址。

**Q4：`Arrays.copyOf()` 是深拷贝还是浅拷贝？**

浅拷贝。对基本类型数组等价于深拷贝；对**对象数组**，复制的只是引用，新旧数组的元素仍指向同一批对象。

**Q5：数组能存基本类型吗？集合呢？**

数组可以（`int[]`）；集合不行，只能存对象，`List<Integer>` 靠自动装箱，因此有额外的内存和性能开销。

---

## 八、面向对象基础（类、对象、构造器、方法）

### 1. 类与对象

**类是模板，对象是按模板造出来的实例**。类里放两种东西：字段（属性）和方法（行为）。

```java
public class User {
    private Long id;              // 字段 / 成员变量
    private String name;

    public String greet() {       // 方法
        return "Hi, " + name;
    }
}

User u = new User();              // 创建对象
u.greet();
```

`new User()` 做了三件事：**在堆中开辟空间并给字段赋默认值 → 执行初始化和构造器 → 把对象地址返回给栈上的变量 `u`**。所以 `u` 存的是地址，对象实体在堆里。

一个 `.java` 文件里**最多只能有一个 `public` 类，且类名必须和文件名相同**。

---

### 2. 构造器（构造方法）

用来在创建对象时完成初始化。三个硬性规则：

- 方法名必须**与类名完全相同**
- **没有返回值**，连 `void` 都不能写
- 不能被 `static`、`final`、`abstract` 修饰

```java
public class User {
    private Long id;
    private String name;

    public User() { }                    // ⭐ 无参构造，一定要显式保留

    public User(Long id) {
        this.id = id;
    }

    public User(Long id, String name) {
        this(id);                        // 复用另一个构造器，必须写在第一行
        this.name = name;
    }
}
```

**最容易踩的坑：一旦自己写了有参构造，编译器就不再生成默认的无参构造。**

```java
public class User {
    public User(Long id) { this.id = id; }
}

new User();      // ✘ 编译错误，无参构造已经没了
```

> 这不只是编译问题：**Jackson 反序列化 JSON、MyBatis 映射查询结果、JPA 实体，都要求类有无参构造**，否则运行期直接报错。写了有参构造就务必把无参的补回来（或用 Lombok 的 `@NoArgsConstructor`）。

| 对比项 | 构造器 | 普通方法 |
| ------ | ------ | -------- |
| 方法名 | 必须和类名相同 | 任意 |
| 返回值 | 不写，连 `void` 都不行 | 必须写 |
| 调用时机 | `new` 的时候自动调用一次 | 手动调用，可多次 |
| 能否继承 | 不能被继承，但可被 `super()` 调用 | 可以被继承和重写 |

---

### 3. this 关键字

`this` 指向**当前正在操作的这个对象**，三个用途：

```java
public class User {
    private String name;

    // ① 区分同名的字段和形参 —— 最常见的用法
    public void setName(String name) {
        this.name = name;              // 左边是字段，右边是形参
    }

    // ② 调用本类的其他构造器，必须是构造器里的第一句
    public User(String name) {
        this();
        this.name = name;
    }

    // ③ 返回自身，实现链式调用（Builder 模式的基础）
    public User withName(String name) {
        this.name = name;
        return this;
    }
}
```

---

### 4. 方法

```java
修饰符 返回值类型 方法名(参数列表) {
    方法体
    return 返回值;
}
```

**方法签名 = 方法名 + 参数列表**，**不包含返回值类型** —— 这是判断重载的依据。

**可变参数**：本质是数组，一个方法最多一个，且必须放在参数列表最后。

```java
public int sum(int... nums) {          // nums 就是一个 int[]
    int total = 0;
    for (int n : nums) total += n;
    return total;
}

sum();           // 0，可以不传
sum(1, 2, 3);    // 6
```

#### 方法重载（Overload）

**同一个类中，方法名相同、参数列表不同**（类型、个数、顺序至少有一项不同）。

```java
public void print(int a) { }
public void print(String s) { }        // ✔ 类型不同
public void print(int a, int b) { }    // ✔ 个数不同

public int print(int a) { }            // ✘ 只有返回值不同，不构成重载，编译报错
```

> 重载**与返回值类型、访问修饰符、异常声明都无关**，只看参数列表 —— 面试高频。

| 对比项 | 重载 Overload | 重写 Override |
| ------ | ------------- | ------------- |
| 发生位置 | 同一个类中 | 子类重写父类方法 |
| 方法名 | 相同 | 相同 |
| 参数列表 | **必须不同** | **必须相同** |
| 返回值 | 无所谓 | 相同或是父类返回值的子类 |
| 访问修饰符 | 无所谓 | 不能比父类更严格 |

> 重写的细节（`@Override`、`super`、动态绑定）见**第十章 继承与多态**。

---

### 5. 代码块与初始化顺序（面试高频）

```java
public class Demo {
    static { System.out.println("1 静态代码块"); }   // 类加载时执行，只执行一次
    { System.out.println("2 实例代码块"); }          // 每次 new 都执行，在构造器之前
    public Demo() { System.out.println("3 构造器"); }
}

new Demo();
new Demo();
// 输出：1 2 3 2 3
```

**单个类的初始化顺序**：静态变量 / 静态代码块（只一次，按定义先后）→ 实例变量 / 实例代码块 → 构造器。

**有继承时**（面试常考的完整顺序）：

```text
父类静态 → 子类静态 → 父类实例块 → 父类构造器 → 子类实例块 → 子类构造器
```

> 实际开发中实例代码块几乎不用（初始化逻辑写在构造器里更清楚）；静态代码块用于加载配置、注册驱动这类**只需执行一次**的场景。

---

### 6. 包（package）与实体类分层

包名全小写，用倒置域名：`com.company.project.module`。企业项目的典型分层：

```text
com.example.order
├─ controller    接收请求
├─ service       业务逻辑
├─ mapper        数据库访问
├─ entity        与数据库表对应的实体
└─ dto / vo      传输对象、返回给前端的对象
```

| 类型 | 全称 | 用途 |
| ---- | ---- | ---- |
| Entity / DO | Data Object | 和**数据库表字段一一对应** |
| DTO | Data Transfer Object | 服务之间、接口之间传输数据 |
| VO | View Object | 返回给前端展示的数据 |

> 为什么要分开：数据库实体里有密码、删除标记、内部状态等字段，**直接返回给前端就是安全事故**。业务上也常需要拼装多张表的数据，Entity 装不下。

---

### 7. 实体类的实际写法

```java
// 传统写法：private 字段 + getter/setter + 无参构造，几十行样板代码
public class User {
    private Long id;
    private String name;

    public User() { }
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    // ... 省略
}
```

```java
// ⭐ 实际项目：用 Lombok，编译期自动生成上面那堆代码
@Data                    // getter/setter/toString/equals/hashCode
@NoArgsConstructor       // 无参构造 —— 框架需要
@AllArgsConstructor
public class User {
    private Long id;
    private String name;
}
```

```java
// 纯粹的不可变数据载体，用 record（JDK 16+）
// 自动生成全参构造、访问器、equals/hashCode/toString，字段全部是 final
public record Point(int x, int y) { }

Point p = new Point(1, 2);
p.x();      // 注意访问器是 x() 而不是 getX()
```

> `record` 适合 DTO、返回值封装这类"只读数据包"；需要 setter 的实体类仍然用普通类 + Lombok。

---

### 8. Lombok 的坑（重点）

Lombok 的本质是**编译期注解处理器**：编译时把 getter/setter/`equals` 等方法直接织入字节码，`.java` 源码里看不到，反编译 `.class` 才能看见。正因为"生成的代码看不见"，它的坑**全部是静默的** —— 不报错，只是行为和你以为的不一样。

**常用注解**：

| 注解 | 作用 |
| ---- | ---- |
| ⭐`@Data` | = `@Getter` + `@Setter` + `@ToString` + `@EqualsAndHashCode` + `@RequiredArgsConstructor` |
| ⭐`@NoArgsConstructor` / `@AllArgsConstructor` | 无参 / 全参构造器 |
| ⭐`@Slf4j` | 生成 `private static final Logger log`，直接 `log.info()` |
| `@Builder` | 建造者模式（见第二十三章） |
| `@RequiredArgsConstructor` | 为 `final` 字段生成构造器 —— ⭐ Spring **构造器注入**的标准写法 |
| `@SneakyThrows` | 偷偷把受检异常当运行时异常抛出 |

---

**① `@Data` + 继承：`equals()` 默认不比较父类字段**

```java
@Data
public class Base { private Long id; }

@Data
public class User extends Base { private String name; }

// 两个 id 完全不同的对象，只要 name 相同就被判定为相等 ⚠️
new User(1L, "张三").equals(new User(2L, "张三"));   // true
```

`@Data` 生成的 `equals`/`hashCode` **只包含本类字段**，直接忽略父类。必须显式声明：

```java
@Data
@EqualsAndHashCode(callSuper = true)     // ✔ 把父类字段也算进去
@ToString(callSuper = true)
public class User extends Base { }
```

> IDEA 会对这种情况给出警告，**不要忽略它**。团队规范里最省事的做法是：**实体类不用继承**，或者干脆不用 `@Data`，改成 `@Getter @Setter @ToString`。

**② `@Data` 的 `equals`/`hashCode` 用了全部字段 —— 作为 Map key 会失效**

```java
Set<User> set = new HashSet<>();
User u = new User(1L, "张三");
set.add(u);

u.setName("李四");        // ⚠️ 字段一改，hashCode 就变了
set.contains(u);         // false！对象还在 set 里，但已经永远找不回来了
```

对象放进 `Set`、当作 `HashMap` 的 key 时，**判等只应该依赖业务主键**：

```java
@Data
@EqualsAndHashCode(of = "id")     // ✔ 只用 id 判等，字段怎么改都不影响
public class User { }
```

**③ `@Builder` 会吞掉字段默认值**

```java
@Data
@Builder
public class Order {
    private String status = "INIT";       // ⚠️ 用 builder 构建时，这个默认值失效
}

Order.builder().build().getStatus();      // null，不是 "INIT"
```

原因是 `@Builder` 走的是全参构造，没赋值的字段就是 `null`。修法是加 `@Builder.Default`：

```java
@Builder.Default
private String status = "INIT";           // ✔
```

**④ `@Builder` 会顺手干掉无参构造器 —— 框架反序列化直接失败**

`@Builder` 会生成一个全参构造，**一旦有了显式构造器，编译器就不再提供默认无参构造**。而 Jackson 反序列化 JSON、MyBatis 映射结果集、Spring 实例化 Bean 都需要无参构造：

```java
@Data
@Builder
@NoArgsConstructor        // ⭐ 用了 @Builder 就必须补这两个
@AllArgsConstructor
public class OrderDTO { }
```

> 报错通常是 `InvalidDefinitionException: Cannot construct instance ... no Creators, like default constructor, exist`。这是接口层最高频的 Lombok 事故。

**⑤ 双向关联时 `toString()` / `equals()` 无限递归 → `StackOverflowError`**

```java
@Data class Order { private User user; }
@Data class User  { private List<Order> orders; }

order.toString();        // ✘ Order → User → Order → ... 栈溢出
```

在其中一侧排除掉这个字段：

```java
@ToString.Exclude
@EqualsAndHashCode.Exclude
private User user;
```

> 同一个 `@ToString` 还有两个老问题（呼应第十三章）：**不要打印密码、身份证、手机号等敏感字段**（日志会被采集留存，属于合规问题），**不要打印大集合**（一条日志几万字符会拖垮日志系统）。

**⑥ `boolean` 字段加 `is` 前缀会导致属性名对不上**

```java
private boolean isDeleted;      // ✘ Lombok 生成的方法是 isDeleted()
```

Jackson 按 JavaBean 规范去掉 `is` 前缀，认为属性名是 `deleted`，于是**前端拿到的字段名和实体类字段名不一致**，序列化、反序列化两头对不上。所以规范是 **`boolean` 字段直接命名为 `deleted`**（呼应第二章）。另外注意 `boolean` 生成的是 `isXxx()`，而包装类 `Boolean` 生成的是 `getXxx()`，混用同样会出问题。

**⑦ `@SneakyThrows` 慎用**

它把受检异常绕过编译器检查直接抛出，调用方**在方法签名上完全看不到**这个异常，也就不会去处理。只适合写工具类和测试代码，业务代码里应当老老实实包装成自定义业务异常（见第十四章）。

**⑧ 环境层面的两个前提**

- **IDE 必须装 Lombok 插件**，否则满屏"找不到 getXxx() 方法"的红线（编译其实是好的）
- ⚠️ **升级 JDK 必须同步升级 Lombok**：它依赖 javac 的内部 API，版本不匹配会直接编译失败（`java.lang.NoSuchFieldError: com.sun.tools.javac...`）。这是 **JDK 8 升 17 / 21 时最常见的编译报错**，遇到先看 Lombok 版本

---

> **团队规范建议**：`@Data` 只用在 DTO / VO 这类扁平的数据对象上；实体类（有继承、要进集合、要当 Map key）写明确的 `@Getter @Setter @ToString`，判等用 `@EqualsAndHashCode(of = "id")`；用 `@Builder` 就必须补 `@NoArgsConstructor @AllArgsConstructor`。

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 类名用大驼峰 | `UserService`、`OrderDetail` |
| 字段一律 `private` | 通过 getter/setter 访问 |
| 写了有参构造就补上无参构造 | 否则框架反序列化会失败 |
| 实体类用 Lombok，别手写样板代码 | `@Data` + `@NoArgsConstructor` |
| 有继承关系时加 `@EqualsAndHashCode(callSuper = true)` | 否则父类字段不参与判等 |
| 用 `@Builder` 就补上 `@NoArgsConstructor` + `@AllArgsConstructor` | 否则框架反序列化失败 |
| 字段默认值配 `@Builder.Default` | 否则 builder 构建时默认值失效 |
| Entity 不直接返回给前端 | 转成 DTO / VO |
| 一个类只做一件事 | 类过大就拆分，方法不超过 80 行 |
| 方法参数不超过 5 个 | 再多就封装成一个对象 |

---

### 10. 面试重点

**Q1：构造器和普通方法有什么区别？**

构造器名必须与类名相同、没有返回值、在 `new` 时自动调用一次、不能被继承和重写；普通方法则相反。

**Q2：一个类如果不写构造器会怎样？写了有参构造呢？**

不写时编译器自动生成一个 `public` 无参构造；**一旦自己写了任意构造器，默认无参构造就不再生成**。这会导致 Jackson、MyBatis、JPA 等依赖反射调无参构造的框架报错，所以要显式补回来。

**Q3：方法重载的判断依据是什么？返回值不同算重载吗？**

只看**参数列表**（类型、个数、顺序），与返回值类型、访问修饰符、异常声明无关。仅返回值不同**不构成重载，会直接编译报错**。

**Q4：静态代码块、实例代码块、构造器的执行顺序？**

同一个类：静态代码块（类加载时一次）→ 实例代码块 → 构造器。有继承时：父类静态 → 子类静态 → 父类实例块 → 父类构造 → 子类实例块 → 子类构造。

**Q5：`this` 有哪些用途？**

区分同名的字段和形参、调用本类其他构造器（`this(...)`，必须在第一行）、返回当前对象实现链式调用。

**Q6：一个 `.java` 文件可以有几个类？**

可以有多个类，但**最多一个 `public` 类，且必须与文件名同名**。

**Q7：Lombok 的原理是什么？用它踩过什么坑？**

Lombok 是**编译期注解处理器**，在编译阶段修改 AST 把 getter/setter/`equals` 等方法织入字节码，源码里看不到、反编译 `.class` 才能看见，所以运行期没有任何性能损耗，但 **IDE 必须装插件**，且**升级 JDK 时必须同步升级 Lombok**（它依赖 javac 内部 API）。

踩过的坑按严重程度：`@Data` 遇到继承时 `equals`/`hashCode` **不包含父类字段**（要加 `@EqualsAndHashCode(callSuper = true)`）；`@Data` 默认用**全部字段**判等，对象进了 `Set` 后改一个字段就再也找不到（应改成 `@EqualsAndHashCode(of = "id")`）；`@Builder` 会**吞掉字段默认值**（要加 `@Builder.Default`）并**干掉无参构造**导致 Jackson 反序列化失败（要补 `@NoArgsConstructor` + `@AllArgsConstructor`）；双向关联时 `toString()` 相互调用**栈溢出**（用 `@ToString.Exclude` 排除）。

---

## 九、封装与 static

### 1. 封装

**隐藏内部实现，只暴露必要的访问入口**。做法很简单：字段 `private`，需要访问就提供 `public` 的 getter/setter。

真正的价值不在于"写 getter/setter"这个动作，而在于**留出了加逻辑的位置**：

```java
public class User {
    private int age;

    public int getAge() { return age; }

    public void setAge(int age) {
        if (age < 0 || age > 150) {                    // ⭐ 校验、日志、转换都能加在这
            throw new IllegalArgumentException("年龄非法: " + age);
        }
        this.age = age;
    }
}
```

如果字段是 `public`，任何人都能写 `user.age = -1`，你连拦都拦不住；而且以后想改字段名或内部存储方式，所有调用方都得跟着改。

**只写 getter 不写 setter，就是只读字段。**

**集合字段的 getter 要做防御性拷贝**（容易被忽略）：

```java
public List<String> getRoles() { return roles; }                   // ✘ 外部拿到后能直接 add/remove
public List<String> getRoles() { return new ArrayList<>(roles); }  // ✔ 返回副本
public List<String> getRoles() { return Collections.unmodifiableList(roles); }  // ✔ 返回只读视图
```

> 访问修饰符的作用范围见**第一章第 3 节**。企业规范：**字段一律 `private`，方法按需 `public`**。

---

### 2. static 的含义

`static` 修饰的成员**属于类，不属于任何对象**：随类加载初始化一次，所有对象共享同一份。

```java
public class Counter {
    private static int total = 0;      // 类变量：全类共享一份
    private int id;                    // 实例变量：每个对象各一份

    public Counter() {
        total++;
        this.id = total;
    }
}

new Counter(); new Counter();
Counter.total;      // 2 ⭐ 用类名访问，不要用对象名访问
```

| 对比项 | 静态变量（类变量） | 实例变量 |
| ------ | ------------------ | -------- |
| 归属 | 类，全局一份 | 对象，每个对象一份 |
| 初始化时机 | 类加载时，只一次 | `new` 对象时 |
| 访问方式 | ⭐`类名.变量` | `对象.变量` |
| 生命周期 | 类卸载才销毁 | 对象被 GC 时销毁 |

**静态方法的两条限制**：

```java
public class Demo {
    private String name;

    public static void print() {
        System.out.println(this.name);   // ✘ 编译错误：静态方法里没有 this
        System.out.println(name);        // ✘ 编译错误：不能直接访问实例成员
    }
}
```

原因很直白：静态方法可能在**任何对象都还没创建**时就被调用，自然没有"当前对象"可言。反过来，**实例方法可以随便访问静态成员**。

---

### 3. static 的典型用途

**① 工具类** —— 最常见的用法：

```java
public final class DateUtils {           // final：不允许被继承
    private DateUtils() { }              // ⭐ 私有构造：不允许被 new，工具类不需要实例

    public static String format(LocalDate date) { ... }
}

DateUtils.format(LocalDate.now());       // 直接类名调用
```

**② 常量**：

```java
public static final int    MAX_RETRY = 3;
public static final String CACHE_PREFIX = "order:";
```

**③ 静态代码块** —— 加载配置、注册驱动等只需执行一次的初始化（执行顺序见第八章第 5 节）。

---

### 4. static 的坑（重点）

**① 静态变量是全局共享的，并发下不安全**

```java
private static int count = 0;
count++;                        // ✘ 非原子操作，多线程会丢更新

private static final AtomicInteger COUNT = new AtomicInteger();   // ✔
```

```java
private static SimpleDateFormat SDF = new SimpleDateFormat("yyyy-MM-dd");    // ✘ 线程不安全，
                                                                             //   高并发下会返回错误日期
private static final DateTimeFormatter FMT =
        DateTimeFormatter.ofPattern("yyyy-MM-dd");                           // ✔ 不可变，线程安全
```

> `SimpleDateFormat` 定义成静态字段是**线上事故的常见来源**：它内部用成员变量存中间状态，多线程共用会互相覆盖，解析出的日期直接是错的。

**② 静态字段不会被回收，容易内存泄漏**

```java
private static Map<Long, User> CACHE = new HashMap<>();   // ✘ 只进不出，数据越堆越多直到 OOM
```

静态变量的生命周期和类一样长，进程不结束就不会被 GC。要做缓存请用有淘汰策略的方案（Caffeine、Redis）。

**③ Spring 里静态字段注入不进去**

```java
@Autowired
private static UserService userService;    // ✘ 永远是 null
```

Spring 是给**对象实例**注入依赖的，静态字段不属于任何实例。改成普通字段即可；确实需要在静态方法里用，就通过 setter 注入再赋给静态字段。

**④ 静态方法不能被重写，只会被"隐藏"**

```java
class Parent { static void f() { System.out.println("父"); } }
class Child extends Parent { static void f() { System.out.println("子"); } }

Parent p = new Child();
p.f();      // 输出"父" —— static 是编译期按引用类型绑定的，没有多态
```

---

### 5. final 与常量

| 修饰对象 | 效果 |
| -------- | ---- |
| 变量 | 只能赋值一次 |
| 方法 | 不能被子类重写 |
| 类 | 不能被继承（`String`、`Integer` 都是 `final` 类） |

**`final` 修饰引用类型时，锁的是引用，不是内容**（高频考点）：

```java
final List<String> list = new ArrayList<>();
list.add("a");                // ✔ 内容可以随便改
list = new ArrayList<>();     // ✘ 编译错误，引用不能重新指向
```

要让内容也不可变，得用 `List.of(...)`（JDK 9+）或 `Collections.unmodifiableList(...)`。

**常量的标准写法**是 `public static final` + 全大写下划线命名：

```java
// ✔ 常量类：final 类 + 私有构造
public final class OrderConstants {
    private OrderConstants() { }
    public static final String STATUS_PAID = "PAID";
}

// ✘ 阿里规约不推荐用接口存常量：接口的字段天生就是 public static final，
//    但常量会被实现类继承，容易造成滥用和命名污染
public interface Constants { String STATUS_PAID = "PAID"; }
```

> 一组有限且相关的取值（订单状态、性别、审批结果），**优先用枚举而不是常量**，枚举有类型安全和 `switch` 支持（见第十二章）。

---

### 6. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 字段一律 `private` | 通过 getter/setter 访问 |
| 静态成员用类名访问 | `Counter.total`，不要写 `obj.total` |
| 工具类：`final` 类 + 私有构造 + 全静态方法 | 不允许被继承和实例化 |
| 静态字段不做可变缓存 | 不会被 GC，改用 Caffeine / Redis |
| 静态字段要么 `final`，要么线程安全 | 共享可变状态必须加锁或用原子类 |
| 不用 `SimpleDateFormat` 做静态字段 | 改用 `DateTimeFormatter` |
| 常量用 `public static final` + 全大写 | 一组相关取值优先用枚举 |
| Spring Bean 不用静态字段接收注入 | 注入不进去，永远是 `null` |

---

### 7. 面试重点

**Q1：为什么要封装？只是为了写 getter/setter 吗？**

不是。封装的价值在于**留出可控的入口**：可以在 setter 里加校验、日志、格式转换；内部字段名或存储结构变化时不影响调用方；还能通过只提供 getter 实现只读。

**Q2：静态方法为什么不能使用 `this` 和实例变量？**

静态方法属于类，可能在任何对象创建之前就被调用，此时不存在"当前对象"，所以没有 `this`，也无法访问依赖对象存在的实例成员。反过来，实例方法可以访问静态成员。

**Q3：静态方法能被重写吗？**

不能。子类定义同名静态方法叫**隐藏（hiding）**，不是重写。静态方法在编译期就按**引用类型**绑定，不参与运行期的动态分派，因此没有多态效果。

**Q4：静态变量存在哪里？**

JDK 7 之前在方法区（永久代）；JDK 8 之后永久代被元空间取代，**元空间只存类的元数据，静态变量实际存放在堆中的 Class 对象里**。

**Q5：`final` 修饰引用类型时，对象内容还能改吗？**

能。`final` 只保证**引用不能重新指向别的对象**，对象自身的内容不受限制。要内容也不可变得用不可变集合或不可变类。

**Q6：工具类为什么要写私有构造器？**

工具类全是静态方法，不需要也不应该被实例化。私有构造能阻止外部 `new`，配合 `final` 修饰类还能防止被继承。

---

## 十、继承与多态

### 1. 继承（extends）

子类继承父类的字段和方法，实现代码复用并建立 **is-a** 关系。本章统一用"支付"这个真实场景举例：

```java
public class Payment {
    protected String orderNo;                       // protected：子类可访问

    public boolean pay(BigDecimal amount) {
        System.out.println("通用支付");
        return true;
    }
}

public class AliPay extends Payment {
    public void aliRefund() { }                     // 子类特有的方法
}
```

四个必须记住的点：

- **Java 类只能单继承**（一个 `extends`），但可以实现多个接口
- **所有类都隐式继承 `Object`**，不写 `extends` 就是 `extends Object`
- 子类**不能继承父类的构造器**；`private` 成员会被继承但子类不可见
- `final` 修饰的类不能被继承（`String`、`Integer` 都是）

---

### 2. super 关键字

```java
public class AliPay extends Payment {

    public AliPay(String orderNo) {
        super(orderNo);              // ① 调用父类构造器，必须是第一句
    }

    @Override
    public boolean pay(BigDecimal amount) {
        super.pay(amount);           // ② 调用父类被重写的方法
        System.out.println("支付宝支付");
        return true;
    }
}
```

**子类构造器一定会调用父类构造器**：不写 `super(...)` 时编译器会自动插入一句无参的 `super()`。

由此产生一个常见的编译错误：

```java
public class Payment {
    public Payment(String orderNo) { }      // 只有有参构造，无参构造被"吃掉"了
}

public class AliPay extends Payment {
    public AliPay() { }                     // ✘ 编译错误：父类没有无参构造，
}                                           //   隐式的 super() 找不到目标
```

> 解决办法：子类显式写 `super(orderNo)`，或者给父类补一个无参构造。这和第八章"有参构造会吃掉默认无参构造"是同一个坑的延伸。
>
> 继承体系下的完整初始化顺序见**第八章第 5 节**。

---

### 3. 方法重写（Override）

子类重新实现父类的方法。规则可以背成 **两同、两小、一大**：

| 口诀 | 含义 |
| ---- | ---- |
| 两同 | **方法名**、**参数列表**必须完全相同 |
| 两小 | **返回值类型**相同或是父类返回值的子类（协变返回）；**抛出的受检异常**范围不能更大 |
| 一大 | **访问权限**不能比父类更严格（父类 `public`，子类不能改成 `protected`） |

**不能被重写的三种方法**：`private`（子类根本看不见）、`static`（只是隐藏，见第九章）、`final`。

**`@Override` 注解必须写**：它不影响运行，但能让编译器帮你检查 —— 方法名拼错、参数写错时立刻报错，而不是悄悄变成一个新方法。

```java
public class AliPay extends Payment {
    @Override
    public boolean pay(BigDecimal amount) { ... }    // ✔

    @Override
    public boolean pay(double amount) { ... }        // ✘ 参数类型不同，编译器直接报错
}                                                    //   不写 @Override 就变成了重载，
                                                     //   运行时才发现方法没生效
```

---

### 4. 多态

**父类引用指向子类对象，调用方法时执行的是子类的实现。**

三个前提缺一不可：**有继承（或实现接口）→ 有方法重写 → 父类引用指向子类对象**。

```java
Payment payment = new AliPay();      // 父类引用指向子类对象
payment.pay(amount);                 // 运行时执行的是 AliPay 的 pay()
```

#### 编译看左边，运行看右边

- **编译期**：能调用哪些方法，由**引用类型（左边）**决定
- **运行期**：实际执行哪份实现，由**对象类型（右边）**决定

```java
Payment p = new AliPay();
p.pay(amount);       // ✔ Payment 声明了 pay()，运行时执行 AliPay 的版本
p.aliRefund();       // ✘ 编译错误：Payment 里没有这个方法，编译期就过不去
```

#### 字段没有多态

```java
class Payment { String type = "父"; }
class AliPay extends Payment { String type = "子"; }

Payment p = new AliPay();
p.type;      // "父" ← 字段是编译期按引用类型访问的，不参与动态绑定
```

> 只有**实例方法**有多态。字段、静态方法、`private` 方法都是编译期绑定。

#### 多态到底解决什么问题

这是这一章最该理解的地方：

```java
// ✘ 没有多态：每加一种支付方式，都要回来改这个方法
public void doPay(String type, BigDecimal amount) {
    if ("ali".equals(type))      new AliPay().pay(amount);
    else if ("wx".equals(type))  new WechatPay().pay(amount);
    // 加一种就改一次，改一次就要重新测一次 —— 违反开闭原则
}

// ✔ 有多态：新增支付方式完全不用动这个方法
public void doPay(Payment payment, BigDecimal amount) {
    payment.pay(amount);
}
```

> Spring 里写 `@Autowired List<Payment> payments;` 就能把所有实现类自动注入进来，再配合 `Map<String, Payment>` 按名字路由 —— 这就是多态在框架里最典型的落地方式。

---

### 5. 向上转型与向下转型

```java
Payment p = new AliPay();          // 向上转型：自动完成，永远安全
AliPay ali = (AliPay) p;           // 向下转型：必须强转，可能抛 ClassCastException

Payment p2 = new WechatPay();
AliPay bad = (AliPay) p2;          // ✘ ClassCastException，运行期才炸
```

**强转前一律先用 `instanceof` 判断**：

```java
if (p instanceof AliPay) {
    AliPay a = (AliPay) p;
    a.aliRefund();
}

// ⭐ JDK 16+ 的模式匹配，判断和转型一步到位，现在的推荐写法
if (p instanceof AliPay a) {
    a.aliRefund();
}
```

> 代码里频繁出现向下转型，通常说明**抽象设计有问题** —— 应该把需要的行为提到父类/接口上，而不是转成具体类型再调。

---

### 6. 优先组合，而非继承

继承是**最强的耦合关系**：父类改一个方法，所有子类都可能受影响；而且 Java 只能单继承，继承名额用掉就没了。

```java
// ✘ 只为了复用几个方法就继承 —— OrderService 和 BaseUtils 根本不是 is-a 关系
public class OrderService extends BaseUtils { }

// ✔ 组合：把它当成一个成员持有
public class OrderService {
    private final BaseUtils utils;
}
```

判断标准很简单：**能说通"子类 is a 父类"才用继承，只是"想用父类的功能"就用组合。**

实际项目中，继承主要出现在这三种场景：

- 实现框架/父类定下的模板（如继承 `RuntimeException` 定义业务异常）
- 模板方法模式：父类定流程骨架，子类填具体步骤
- 抽象类 + 多态，做策略分发（下一章展开）

其余绝大多数情况，用**接口 + 组合**更合适。

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 重写方法必须加 `@Override` | 让编译器帮你检查签名 |
| 继承层次不超过 3 层 | 层次太深难以理解和调试 |
| 优先组合，而非继承 | 只有真正的 is-a 关系才用继承 |
| 强转前先 `instanceof` | 或直接用 JDK 16+ 的模式匹配写法 |
| 不要在构造器里调用可被重写的方法 | 此时子类字段还没初始化，会拿到 `null` / `0` |
| 父类保留无参构造 | 避免子类被迫每个构造器都写 `super(...)` |
| 面向父类/接口编程 | 参数、返回值、变量都声明成父类型 |

---

### 8. 面试重点

**Q1：多态的三个前提是什么？**

继承（或实现接口）、子类重写父类方法、父类引用指向子类对象。三者缺一都不构成多态。

**Q2：方法重写的规则？**

两同两小一大：方法名和参数列表相同；返回值类型和抛出的受检异常范围不能更大；访问权限不能更小。`private`、`static`、`final` 方法不能被重写。

**Q3："编译看左边，运行看右边"是什么意思？**

编译期能调用哪些方法由引用类型（等号左边）决定，编译器只认左边声明了什么；运行期实际执行哪份代码由对象的真实类型（等号右边）决定。

**Q4：为什么字段没有多态？**

字段访问是**编译期静态绑定**的，编译器按引用类型去找字段；只有实例方法通过虚方法表在运行期动态分派。所以父类引用访问同名字段拿到的永远是父类的那份。

**Q5：多态的底层是怎么实现的？**

JVM 为每个类维护一张**虚方法表（vtable）**，表中存放各方法实际入口地址，子类重写会覆盖父类对应的表项。调用实例方法时用 `invokevirtual` 指令，运行期根据对象实际类型查表分派，这就是动态绑定。

**Q6：子类构造器一定会调用父类构造器吗？**

会。不写 `super(...)` 时编译器自动插入无参的 `super()`。如果父类只有有参构造，子类就必须显式调用 `super(参数)`，否则编译报错。

**Q7：为什么推荐"组合优于继承"？**

继承耦合最强，父类改动会波及所有子类，还会暴露父类实现细节（破坏封装），且 Java 单继承名额只有一个。组合只依赖对方的公开接口，更灵活、更好替换和测试。

**Q8：为什么不能在构造器里调用可被重写的方法？**

父类构造器先于子类字段初始化执行。如果父类构造器里调用了被子类重写的方法，该方法会访问到**尚未初始化的子类字段**，拿到 `null` 或 `0`，属于典型的隐蔽 bug。

---

## 十一、抽象类与接口

### 1. 抽象类

用 `abstract` 修饰，代表**"未完成的类"** —— 定义了共性，但把变化的部分留给子类实现。

```java
public abstract class Payment {
    protected String orderNo;                  // ⭐ 可以有普通字段（状态）

    public Payment(String orderNo) {           // ⭐ 有构造器，供子类 super() 调用
        this.orderNo = orderNo;
    }

    protected abstract boolean doPay(BigDecimal amount);   // 抽象方法：只有声明，没有方法体

    public void printOrderNo() {               // ⭐ 可以有普通方法
        System.out.println(orderNo);
    }
}
```

五个要点：

- **不能被实例化**（`new Payment()` 编译报错），但**有构造器**，供子类通过 `super()` 调用
- 有抽象方法的类必须声明为抽象类；反过来，**抽象类可以一个抽象方法都没有**
- 子类必须实现全部抽象方法，否则子类也得声明为 `abstract`
- `abstract` 不能和 `final`、`static`、`private` 同时使用（含义互相冲突）
- 抽象类同样受单继承限制

#### 最典型的用法：模板方法模式

```java
public abstract class Payment {

    // final：流程骨架不允许子类改
    public final boolean execute(BigDecimal amount) {
        check(amount);                     // 固定步骤
        boolean ok = doPay(amount);        // ⭐ 变化点，交给子类
        log(ok);                           // 固定步骤
        return ok;
    }

    protected abstract boolean doPay(BigDecimal amount);

    private void check(BigDecimal amount) { }
    private void log(boolean ok) { }
}
```

> **父类定流程，子类填步骤**。Spring 的 `AbstractApplicationContext`、`JdbcTemplate` 都是这个套路 —— 这也是抽象类相比接口最不可替代的价值。

---

### 2. 接口（interface）

接口定义**"能做什么"**，不关心怎么做，是一份纯粹的能力契约。

```java
public interface Payment {
    String VERSION = "1.0";                  // 隐式 public static final（常量）

    boolean pay(BigDecimal amount);          // 隐式 public abstract
}

public class AliPay implements Payment {
    @Override
    public boolean pay(BigDecimal amount) { return true; }
}
```

- 字段隐式是 `public static final`，**接口没有实例字段，也就没有状态**
- 抽象方法隐式是 `public abstract`，实现类重写时**必须是 `public`**
- 一个类可以 `implements A, B, C` 实现多个接口
- 接口之间可以**多继承**：`interface C extends A, B`
- 接口**没有构造器**

---

### 3. JDK 8+ 接口的新能力（重点）

这部分是老教材里没有的，也是"抽象类和接口的区别"这道题现在的正确答案基础。

| 版本 | 新增 | 作用 |
| ---- | ---- | ---- |
| JDK 8 | `default` 方法 | 带方法体，实现类可直接用，也可重写 |
| JDK 8 | `static` 方法 | 接口自己的工具方法，只能用接口名调用 |
| JDK 9 | `private` 方法 | 给 `default` 方法抽取公共逻辑，不对外暴露 |

```java
public interface Payment {
    boolean pay(BigDecimal amount);

    default boolean refund(BigDecimal amount) {     // 有默认实现，实现类不写也能用
        log("默认不支持退款");
        return false;
    }

    static Payment of(String type) {                // 工具方法：Payment.of("ali")
        return "ali".equals(type) ? new AliPay() : new WechatPay();
    }

    private void log(String msg) { }                // JDK 9+，只给 default 方法内部用
}
```

**为什么 JDK 8 要加 `default`？** 因为要给 `Collection` 接口新增 `stream()`、`forEach()`。如果按老规则加抽象方法，**全世界所有实现类都会编译失败**。`default` 让接口能够**向后兼容地演进** —— 这是它被引入的唯一根本原因，不是为了让你随便往接口里塞实现。

**菱形问题**：实现的两个接口有同名 `default` 方法时，编译器无法决定用哪个，**必须重写**：

```java
interface A { default void f() { } }
interface B { default void f() { } }

class C implements A, B {
    @Override
    public void f() {
        A.super.f();          // ⭐ 用「接口名.super.方法()」指定
    }
}
```

---

### 4. 抽象类 vs 接口（面试必问）

| 对比项 | 抽象类 | 接口 |
| ------ | ------ | ---- |
| 关键字 | `abstract class` / `extends` | `interface` / `implements` |
| 数量限制 | 只能继承 **1 个** | 可以实现**多个** |
| 字段 | 任意字段，**可以有状态** | 只能是 `public static final` 常量，**无状态** |
| 方法 | 抽象方法 + 普通方法 + 静态方法 | 抽象方法 + `default` + `static` + `private`(9+) |
| 构造器 | ⭐**有**（供子类调用） | ⭐**没有** |
| 访问修饰符 | 任意 | 方法默认 `public` |
| 设计语义 | **is-a**，共性的抽象 | **can-do**，能力的约定 |

**怎么选：**

- 需要共享**字段/状态**，或需要构造逻辑、模板流程 → **抽象类**
- 只定义**能力契约**，实现类可能来自完全不同的继承体系 → **接口**
- **拿不准就用接口** —— Java 单继承名额只有一个，用掉就没了，而接口不占名额

---

### 5. 实际项目里的组合用法

**接口定契约 + 抽象类提供骨架实现**，是 Java 集合框架的经典套路：

```text
List（接口，定义契约）
  └─ AbstractList（抽象类，把大部分方法实现掉）
       └─ ArrayList（只需实现少数几个核心方法）
```

**Spring 项目的分层**：

```java
public interface UserService {
    User getById(Long id);
}

@Service
public class UserServiceImpl implements UserService {
    @Override
    public User getById(Long id) { ... }
}
```

> **Service 要不要抽接口是有争议的**：只有一个实现类时，接口就只是多一个文件。早期必须抽，是因为 Spring AOP 走 JDK 动态代理需要接口；现在 Spring Boot 默认用 CGLIB 代理类，**不写接口 AOP 照样生效**。
>
> 判断标准：**有多个实现、需要对外暴露契约、或要方便 mock 测试就抽；否则可以不抽。** 跟着团队现有规范走即可。

---

### 6. 函数式接口（铺垫）

**只有一个抽象方法**的接口叫函数式接口，可以用 Lambda 表达式来实现。`default`、`static` 方法不算在内。

```java
@FunctionalInterface                       // 可选，加上让编译器校验"只有一个抽象方法"
public interface Handler {
    void handle(String msg);
}

Handler h = msg -> System.out.println(msg);      // Lambda 直接当实现
```

> `Runnable`、`Comparator`、`Function`、`Predicate` 都是函数式接口。详见**第十七章 Lambda 与 Stream API**。

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 优先用接口 | 不占用单继承名额，扩展性更好 |
| 接口只定义行为，不放常量 | 常量放常量类或枚举（见第九章） |
| 实现方法必须加 `@Override` | 让编译器校验签名 |
| `default` 方法只用于接口演进 | 不要拿它当"多继承"来堆业务逻辑 |
| 抽象类命名以 `Abstract` 开头 | `AbstractPayment`，一眼看出是骨架类 |
| 接口不加 `I` 前缀 | Java 的习惯是 `UserService` + `UserServiceImpl`，不写 `IUserService` |
| 模板方法用 `final` 修饰 | 防止子类改掉流程骨架 |

---

### 8. 面试重点

**Q1：抽象类和接口的区别？**

抽象类是 **is-a** 的共性抽象，可以有实例字段（状态）、构造器、任意访问修饰符的方法，但只能单继承；接口是 **can-do** 的能力契约，没有实例字段和构造器，方法默认 `public`，可以多实现。JDK 8 后接口也能有 `default`、`static` 方法，两者的差距主要剩下**「有没有状态」和「能不能多继承」**。

**Q2：抽象类能被实例化吗？有构造器吗？**

不能实例化，但**有构造器**。构造器不是用来 `new` 自己的，而是供子类通过 `super()` 调用，完成父类字段的初始化。

**Q3：抽象类可以没有抽象方法吗？**

可以。只要用 `abstract` 修饰，类就不能被实例化 —— 有时正是为了阻止实例化才这么写。

**Q4：JDK 8 为什么要给接口加 `default` 方法？**

为了**接口的向后兼容演进**。要给 `Collection` 加 `stream()`、`forEach()`，若按抽象方法添加会导致所有已有实现类编译失败，`default` 提供了默认实现，老代码不用改。

**Q5：一个类实现的两个接口有同名 `default` 方法会怎样？**

编译报错（菱形问题）。必须在实现类中重写该方法，需要复用某个接口的实现时用 `接口名.super.方法()` 显式指定。

**Q6：接口里的字段和方法默认是什么修饰符？**

字段是 `public static final`（常量），抽象方法是 `public abstract`，因此实现类重写时**必须写成 `public`**，写成 `protected` 会因为"访问权限变小"而编译报错。

**Q7：什么时候用抽象类，什么时候用接口？**

需要共享状态、构造逻辑或固定流程骨架（模板方法）用抽象类；只描述能力、且实现类可能来自不同继承体系用接口。实际开发中优先接口，抽象类主要用来配合接口提供骨架实现。

---

## 十二、枚举与内部类

**A · 枚举（enum）**

### 1. 基本用法与本质

```java
public enum OrderStatus {
    PAID, REFUNDED, CANCELLED
}

OrderStatus s = OrderStatus.PAID;
```

**本质**：编译后是一个 `final class extends java.lang.Enum`，每个枚举常量都是这个类的一个 `public static final` 实例，由 JVM 在类加载时创建。所以枚举天生**单例、线程安全、可以用 `==` 比较**。

| 方法 | 说明 |
| ---- | ---- |
| ⭐`values()` | 返回所有枚举常量的数组 |
| `valueOf(String)` | 按名字获取，**找不到抛 `IllegalArgumentException`**（不是返回 null） |
| `name()` | 常量名字符串 |
| `ordinal()` | 声明顺序，从 0 开始 —— ⚠️ **绝不要持久化** |

---

### 2. 带字段和方法的枚举（⭐ 行业最常用的形态）

实际项目里的枚举几乎都长这样 —— 一个业务码 + 一个描述：

```java
public enum OrderStatus {

    PAID(1, "已支付"),
    REFUNDED(2, "已退款"),
    CANCELLED(3, "已取消");

    private final int code;
    private final String desc;

    OrderStatus(int code, String desc) {      // 构造器隐式 private，不能被外部调用
        this.code = code;
        this.desc = desc;
    }

    public int getCode()     { return code; }
    public String getDesc()  { return desc; }

    // ⭐ 按 code 反查，落库/接口交互时都要用到
    public static OrderStatus of(int code) {
        for (OrderStatus s : values()) {
            if (s.code == code) return s;
        }
        throw new IllegalArgumentException("未知订单状态: " + code);
    }
}
```

> **数据库里存自己定义的 `code`，绝不要存 `ordinal()`** —— 有人在枚举中间插入一个新常量，后面所有值的序号全部错位，数据直接对不上，而且不会报错。

---

### 3. 用枚举替代常量：类型安全

```java
// ✘ int 常量：编译器管不住你
public static final int STATUS_PAID = 1;
handle(999);                    // 编译通过，运行才出错

// ✔ 枚举：类型安全，非法值编译期就被拦下
handle(OrderStatus.PAID);
handle(999);                    // ✘ 编译错误
```

**在 `switch` 中使用枚举，`case` 后面不写枚举类名**：

```java
switch (status) {
    case PAID      -> handlePaid();        // ✔ 不是 OrderStatus.PAID
    case REFUNDED  -> handleRefund();
    default        -> throw new IllegalStateException();
}
```

---

### 4. 枚举 + 抽象方法：替代 if-else 分发

```java
public enum Operation {

    ADD("+") { public int apply(int a, int b) { return a + b; } },
    SUB("-") { public int apply(int a, int b) { return a - b; } };

    private final String symbol;
    Operation(String symbol) { this.symbol = symbol; }

    public abstract int apply(int a, int b);
}

Operation.ADD.apply(1, 2);      // 3
```

> 新增一种运算只需要加一个枚举常量，**不用回去改任何 `if-else`**。这是策略模式的轻量写法，业务里的"审批类型""结算方式"这类分发很适合。

---

### 5. 枚举的其他实战用途

**① 错误码枚举** —— 几乎每个项目都有：

```java
public enum ErrorCode {
    PARAM_INVALID(400, "参数错误"),
    UNAUTHORIZED(401, "未登录"),
    SERVER_ERROR(500, "系统异常");
    // 字段、构造器、getter 同上
}
```

**② 单例** —— 枚举单例是《Effective Java》推荐的写法：

```java
public enum ConfigManager {
    INSTANCE;

    public String get(String key) { ... }
}

ConfigManager.INSTANCE.get("url");
```

原因：JVM 保证枚举实例只创建一次，**天然防止反射攻击和反序列化破坏单例**，而双重检查锁写法要靠 `volatile` 加一堆样板代码。

**③ `EnumMap` / `EnumSet`** —— 以枚举为 key 时，比 `HashMap` 快得多（内部就是数组）：

```java
Map<OrderStatus, String> map = new EnumMap<>(OrderStatus.class);
Set<OrderStatus> set = EnumSet.of(OrderStatus.PAID, OrderStatus.REFUNDED);
```

> 枚举**可以实现接口，但不能继承类**（它已经继承了 `Enum`）。

---

**B · 内部类**

### 6. 四种内部类

| 类型 | 定义位置 | 使用频率 |
| ---- | -------- | -------- |
| ⭐**静态内部类** | 类内部，加 `static` | 高，首选 |
| 成员内部类 | 类内部，不加 `static` | 低，有内存泄漏风险 |
| 局部内部类 | 方法内部 | 基本不用 |
| ⭐**匿名内部类** | 方法内部，`new 接口(){...}` | 中，被 Lambda 大量替代 |

#### 静态内部类

```java
public class Outer {
    private static int a;

    public static class Inner {        // ⭐ 不持有外部类实例的引用
        void f() { }
    }
}

Outer.Inner in = new Outer.Inner();    // 直接创建，不需要外部类实例
```

#### 成员内部类

```java
public class Outer {
    private int a = 1;

    public class Inner {
        void f() {
            System.out.println(a);              // 能直接访问外部类的私有成员
            System.out.println(Outer.this.a);   // 同名时用 外部类.this 区分
        }
    }
}

Outer.Inner in = new Outer().new Inner();       // 注意这个奇怪的创建语法
```

> **成员内部类会隐式持有外部类的引用**。如果内部类实例被放进静态集合、或者交给了一个长时间运行的异步任务，外部类对象就永远无法被 GC —— 这是 Java 内存泄漏的经典成因之一。
>
> **结论：能用静态内部类就用静态内部类**，除非确实需要访问外部类的实例成员。

#### 匿名内部类

```java
// 老写法：创建一个"只用一次"的实现类
Runnable r = new Runnable() {
    @Override
    public void run() { System.out.println("run"); }
};

// ⭐ 函数式接口用 Lambda，等价且简洁得多
Runnable r2 = () -> System.out.println("run");
```

**捕获的局部变量必须是 `final` 或 effectively final（事实上不可变）**：

```java
int count = 0;
Runnable r = () -> System.out.println(count);
count++;      // ✘ 编译错误：count 被修改了，不再是 effectively final
```

原因：**内部类捕获的是变量值的副本，不是变量本身**。局部变量在栈上、方法结束就销毁，而内部类对象在堆上可能活得更久，所以只能拷贝值；既然是副本，就必须禁止修改，否则内外两份值会不一致。

| 对比项 | 匿名内部类 | Lambda |
| ------ | ---------- | ------ |
| 编译产物 | 生成 `Outer$1.class` 文件 | 用 `invokedynamic`，不生成额外 class |
| 适用范围 | 任意接口 / 抽象类 | **只能是函数式接口**（一个抽象方法） |
| `this` 指向 | 匿名内部类**自身** | **外部类**实例 |

---

### 7. 内部类的实际用途

**静态内部类最典型的用法是 Builder 模式**：

```java
public class User {
    private final String name;

    private User(Builder b) { this.name = b.name; }

    public static class Builder {
        private String name;
        public Builder name(String name) { this.name = name; return this; }
        public User build() { return new User(this); }
    }
}

User u = new User.Builder().name("张三").build();
```

> 实际项目里直接用 Lombok 的 `@Builder` 注解生成，但要看得懂这个结构 —— 面试问 Builder 模式时手写的就是它。⚠️ `@Builder` 有两个必踩的坑（吞掉字段默认值、干掉无参构造），见**第八章第 8 节**。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 状态、类型、错误码一律用枚举 | 而不是 `int` / `String` 常量 |
| 枚举自定义 `code` 字段落库 | **绝不用 `ordinal()`** |
| 枚举提供静态反查方法 | `OrderStatus.of(code)` |
| 单例优先用枚举实现 | 天然防反射和反序列化破坏 |
| 内部类优先用 `static` | 避免隐式持有外部类引用导致内存泄漏 |
| 函数式接口用 Lambda 不用匿名内部类 | 更简洁，且不生成额外 class 文件 |
| 内部类超过 50 行就抽成独立类 | 内部类过大会让外部类难以阅读 |

---

### 9. 面试重点

**Q1：枚举的本质是什么？**

编译后是一个继承 `java.lang.Enum` 的 `final` 类，每个枚举常量是该类的一个 `public static final` 实例，由 JVM 在类加载时创建。因此枚举天然单例、线程安全，可以直接用 `==` 比较。

**Q2：为什么说枚举是实现单例最好的方式？**

JVM 保证枚举实例只会被创建一次，**反射无法创建枚举实例**（`newInstance` 对枚举类型直接抛异常），**反序列化也不会生成新对象**（按名字查找已有实例）。相比之下双重检查锁需要 `volatile` 且防不住反射。

**Q3：枚举的 `ordinal()` 能存数据库吗？**

不能。`ordinal()` 是声明顺序，一旦有人在中间插入或调整常量顺序，所有已存数据的含义全部错位，而且不会报任何错。必须自己定义 `code` 字段来持久化。

**Q4：静态内部类和成员内部类的区别？**

静态内部类不持有外部类实例的引用，可以直接 `new Outer.Inner()` 创建，也能有静态成员；成员内部类隐式持有外部类引用，必须通过 `new Outer().new Inner()` 创建，能访问外部类的实例成员，但可能导致外部类对象无法被 GC。**优先用静态内部类。**

**Q5：为什么匿名内部类/Lambda 访问的局部变量必须是 final 或 effectively final？**

因为捕获的是**变量值的副本**。局部变量在栈上、方法返回即销毁，而内部类对象在堆上生命周期更长，只能拷贝值过去。既然是两份独立的数据，就必须禁止修改，否则会出现内外不一致的问题。

**Q6：Lambda 和匿名内部类的区别？**

匿名内部类会编译出独立的 `.class` 文件，可用于任意接口或抽象类，其中的 `this` 指向匿名类自身；Lambda 基于 `invokedynamic` 在运行期生成，不产生额外 class 文件，**只能用于函数式接口**，其中的 `this` 指向外部类实例。

---

## 十三、Object 类与包装类

**A · Object 类**

### 1. 所有类的父类

不写 `extends` 的类都隐式继承 `Object`，因此下面这些方法任何对象都能调用：

| 方法 | 说明 |
| ---- | ---- |
| ⭐`equals(Object)` | 比较是否相等，**默认实现就是 `==`（比地址）** |
| ⭐`hashCode()` | 返回哈希码，`HashMap`/`HashSet` 定位用 |
| ⭐`toString()` | 字符串表示，默认是 `类名@十六进制哈希` |
| `getClass()` | 获取运行时类对象，**反射的入口** |
| `clone()` | 浅拷贝，`protected`，需实现 `Cloneable` |
| `wait()` / `notify()` / `notifyAll()` | 线程间通信（见第二十章） |
| `finalize()` | 已废弃（JDK 9 起 `@Deprecated`），永远不要用 |

---

### 2. equals 与 hashCode（本章重点）

`Object` 的默认实现就是比较地址：

```java
public boolean equals(Object obj) { return (this == obj); }
```

所以**要按业务含义比较内容，就必须重写 `equals()`**。而重写 `equals()` 时，**必须同时重写 `hashCode()`**。

#### 契约

- 两个对象 `equals()` 相等 → **`hashCode()` 必须相等**
- `hashCode()` 相等 → `equals()` **不一定**相等（哈希冲突是正常的）

#### 只重写 equals 不重写 hashCode 的后果

```java
Set<User> set = new HashSet<>();
set.add(new User(1L, "张三"));
set.contains(new User(1L, "张三"));      // ✘ false

Map<User, String> map = new HashMap<>();
map.put(new User(1L), "a");
map.get(new User(1L));                   // ✘ null
```

原因：`HashMap` **先用 `hashCode()` 定位桶，再在桶内用 `equals()` 比较**。两个对象没重写 `hashCode`，哈希码是各自的地址，落在完全不同的桶里 —— 根本走不到 `equals()` 那一步。

#### 标准写法

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;                              // 同一个对象，直接 true
    if (o == null || getClass() != o.getClass()) return false;
    User user = (User) o;
    return Objects.equals(id, user.id);                      // 按业务主键比较
}

@Override
public int hashCode() {
    return Objects.hash(id);                                 // 参与比较的字段要一致
}
```

> 实际开发中用 IDEA 自动生成，或直接用 Lombok 的 `@Data` / `@EqualsAndHashCode`。关键是**两个方法参与计算的字段必须一致**。

#### 一个隐蔽的坑

**对象放进 `HashMap`/`HashSet` 之后，不要再修改参与 `hashCode()` 计算的字段** —— 哈希码变了，就会去错误的桶里找，导致这个元素既取不出来也删不掉，等于内存泄漏。

> ⚠️ 这个坑在用 Lombok 时特别容易踩：`@Data` 默认拿**全部字段**算 `equals`/`hashCode`，随便 `setXxx()` 一下哈希码就变了。要进集合的对象应写成 `@EqualsAndHashCode(of = "id")`，只用业务主键判等；有继承时还要 `callSuper = true`。详见**第八章第 8 节**。

---

### 3. toString()

默认输出 `com.example.User@1b6d3586`，等于没输出。**实体类必须重写**，否则日志里打印对象毫无价值。

```java
@Override
public String toString() {
    return "User{id=" + id + ", name='" + name + "'}";
}
```

用 Lombok 的 `@Data` 会自动生成。两个注意点：

- **不要在 `toString()` 里打印敏感字段**（密码、身份证、手机号）—— 日志会被采集和留存，这是常见的合规问题。Lombok 可以用 `@ToString.Exclude` 排除。
- 不要打印大集合字段，一条日志几万字符会拖垮日志系统。

---

### 4. getClass() 与 clone()

```java
user.getClass().getName();        // 反射入口，见第二十一章
```

`clone()` 是 `protected` 的**浅拷贝**，需要实现 `Cloneable` 接口，实际项目里基本不用。

对象复制的常见做法：

```java
BeanUtils.copyProperties(source, target);     // Spring 工具，⚠️ 也是浅拷贝
```

> 无论 `clone()` 还是 `copyProperties()` 都是**浅拷贝** —— 对象里的引用字段（`List`、嵌套对象）仍然共享同一份，改一个另一个跟着变。

**深拷贝的三种做法**：

| 方式 | 写法 | 评价 |
| ---- | ---- | ---- |
| 手动逐层 `new` | 在 `clone()` 里把每个引用字段也拷一遍 | 最快、最可控，字段一多就难维护 |
| 序列化转一圈 | `SerializationUtils.clone(obj)`（需实现 `Serializable`） | 简单，但性能差，且受原生序列化的限制（见第十九章） |
| ⭐JSON 转一圈 | `JSON.parseObject(JSON.toJSONString(obj), User.class)` | ⭐ 实际项目最常用；⚠️ 泛型要用 `TypeReference`，`LocalDateTime`、`BigDecimal` 要确认序列化配置正确 |

> 另一个高频坑：`BeanUtils.copyProperties()` 有两个同名工具类，**Spring 的参数是 `(源, 目标)`，Apache Commons 的是 `(目标, 源)`，顺序正好相反**，导包导错会把数据反向覆盖，而且不报错。项目里统一只用 Spring 那个。

---

**B · 包装类**

### 5. 八种包装类与自动装箱

| 基本类型 | 包装类 | | 基本类型 | 包装类 |
| -------- | ------ | - | -------- | ------ |
| `byte` | `Byte` | | `long` | `Long` |
| `short` | `Short` | | `float` | `Float` |
| `int` | ⭐`Integer` | | `double` | `Double` |
| `char` | ⭐`Character` | | `boolean` | `Boolean` |

> 只有 `int → Integer`、`char → Character` 名字不是简单首字母大写，容易写错。

**自动装箱 / 拆箱**（JDK 5+）是编译器的语法糖：

```java
Integer a = 10;      // 装箱 → 编译成 Integer.valueOf(10)
int b = a;           // 拆箱 → 编译成 a.intValue()
```

---

### 6. 包装类的三个坑（重点）

#### ① Integer 缓存：`-128 ~ 127`

```java
Integer a = 127, b = 127;
Integer c = 128, d = 128;

a == b;   // true  ← 走 IntegerCache，是同一个对象
c == d;   // false ← 超出缓存范围，valueOf 各 new 了一个
```

缓存范围：`Byte`/`Short`/`Integer`/`Long` 是 `-128 ~ 127`，`Character` 是 `0 ~ 127`，`Boolean` 全缓存，**`Float`/`Double` 没有缓存**。

> **结论：包装类比较内容一律用 `equals()` 或 `Objects.equals()`，绝不用 `==`。**

#### ② 自动拆箱导致 NPE —— 线上最常见的 NPE 之一

```java
Integer count = map.get("key");     // key 不存在时返回 null
if (count > 0) { }                  // ✘ NPE：null 被自动拆箱成 int

Integer x = null;
int y = flag ? x : 0;               // ✘ NPE：三元表达式两个分支类型不同，
                                    //   统一提升为 int，x 被拆箱
```

> 包装类参与算术运算、比较运算前，**必须先判空**。

#### ③ 什么时候用包装类，什么时候用基本类型

```java
// ✘ int 无法表达"没有值"
private int age;         // 前端没传 age 时值是 0，你分不清是"没填"还是"真的填了 0"

// ✔ 包装类可以为 null
private Integer age;
```

| 场景 | 用什么 | 原因 |
| ---- | ------ | ---- |
| **实体类/DTO 字段** | ⭐包装类 | 需要区分"没传值(`null`)"和"传了 0"，数据库字段也可能是 `NULL` |
| **接口的参数和返回值** | ⭐包装类 | 同上，能表达"无数据" |
| **局部变量、循环计数** | ⭐基本类型 | 没有装箱开销，也不会 NPE |

> 这是阿里规约的明文要求：**POJO 类属性、RPC 方法的返回值和参数必须使用包装类型；局部变量推荐使用基本类型。**

---

### 7. 常用方法

```java
Integer.parseInt("123");       // ⭐ String → int
Integer.valueOf("123");        // String → Integer（走缓存）
String.valueOf(123);           // ⭐ 任意类型 → String，null 安全

Integer.MAX_VALUE;             // 2147483647，约 21 亿
Integer.MIN_VALUE;
Integer.compare(a, b);         // 比较大小，用于排序
```

> `parseInt()` 传 `null`、`""`、`"abc"` 都会抛 `NumberFormatException`，接收外部输入时必须先校验或 try-catch。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 重写 `equals()` 必须重写 `hashCode()` | 否则 `HashMap`/`HashSet` 完全失效 |
| 两个方法参与计算的字段保持一致 | 用 IDEA 生成或 Lombok `@Data` |
| 作为 Map key 后不要改字段 | 哈希码变了就再也找不到 |
| 实体类重写 `toString()` | 但排除密码、身份证等敏感字段 |
| POJO 字段、接口出入参用包装类 | 能表达 `null`，区分"未设置" |
| 局部变量用基本类型 | 无装箱开销，无 NPE 风险 |
| 包装类比较用 `equals()` | `==` 只在 `-128~127` 内"碰巧"正确 |
| 拆箱前先判空 | 参与运算的包装类一律先检查 `null` |

---

### 9. 面试重点

**Q1：为什么重写 `equals()` 必须同时重写 `hashCode()`？**

因为 `HashMap`/`HashSet` 是先用 `hashCode()` 定位桶、再在桶内用 `equals()` 比较。只重写 `equals()` 的话，两个"业务上相等"的对象哈希码不同，会落到不同的桶，`equals()` 根本不会被调用，导致 `contains()` 返回 `false`、`get()` 返回 `null`。

**Q2：`hashCode()` 相同的两个对象一定 `equals()` 吗？**

不一定，这叫哈希冲突，是正常现象。反过来才是硬性约定：`equals()` 相等则 `hashCode()` 必须相等。

**Q3：`Integer a = 127, b = 127; a == b` 是什么结果？128 呢？**

127 时是 `true`，128 时是 `false`。`Integer.valueOf()` 对 `-128 ~ 127` 走 `IntegerCache` 返回同一个对象，超出范围则新建对象，`==` 比的是地址。所以包装类比较一律用 `equals()`。

**Q4：什么时候用包装类，什么时候用基本类型？**

POJO 字段、接口的参数和返回值用包装类 —— 需要用 `null` 表达"没有值"，`int` 的默认值 `0` 会和真实的 0 混淆；局部变量和循环计数用基本类型 —— 没有装箱开销，也不存在拆箱 NPE。

**Q5：自动装箱和拆箱的原理是什么？**

编译器的语法糖：装箱编译成 `Integer.valueOf(int)`，拆箱编译成 `intValue()`。因为装箱走的是 `valueOf()`，才有了 `-128 ~ 127` 的缓存行为。

**Q6：`Object` 类有哪些方法？**

`equals()`、`hashCode()`、`toString()`、`getClass()`、`clone()`、`wait()`/`notify()`/`notifyAll()`，以及已废弃的 `finalize()`。

---

## 十四、异常（Exception）

### 1. 异常体系

```text
Throwable
├─ Error ————————————— 虚拟机层面的错误，程序不该也无法处理
│   OutOfMemoryError、StackOverflowError
│
└─ Exception
    ├─ RuntimeException ——— 非受检异常（unchecked），编译器不强制处理
    │   NullPointerException、IndexOutOfBoundsException、
    │   ClassCastException、NumberFormatException、ArithmeticException
    │
    └─ 其他 Exception ————— 受检异常（checked），编译器强制处理
        IOException、SQLException、ClassNotFoundException
```

| 对比项 | 受检异常（Checked） | 非受检异常（Unchecked） |
| ------ | ------------------- | ----------------------- |
| 编译器 | **强制**处理：`try-catch` 或 `throws` | 不强制 |
| 范围 | `Exception` 中除 `RuntimeException` 外的部分 | `RuntimeException` 及其子类、`Error` |
| 语义 | 外部原因导致，**可预期、可恢复** | 程序 bug 或不可恢复 |
| 例子 | 文件不存在、网络中断、数据库连接失败 | 空指针、下标越界、类型转换错误 |

> **`Error` 不要 catch**。`OutOfMemoryError` 捕获了也没用，此时 JVM 已经没有内存可分配了；正确做法是让它挂掉、留下堆转储去分析。

---

### 2. try-catch-finally

```java
try {
    // 可能抛异常的代码
} catch (FileNotFoundException e) {          // ⭐ 子类异常必须写在父类前面
    log.error("文件不存在", e);
} catch (IOException | SQLException e) {     // JDK 7+ 多异常合并捕获
    log.error("IO 或数据库异常", e);
} finally {
    // 无论是否异常都会执行，用于释放资源
}
```

三条规则：

- **`catch` 的顺序必须从子类到父类**，把 `Exception` 写在前面会导致后面的 `catch` 编译报错（unreachable）
- **`finally` 一定执行**，除非调用了 `System.exit()` 或 JVM 直接崩溃
- **`finally` 里绝对不要写 `return`**：

```java
public int f() {
    try {
        return 1;
    } finally {
        return 2;      // ✘ 最终返回 2 —— try 的返回值被覆盖，抛出的异常也被彻底吞掉
    }
}
```

---

### 3. try-with-resources（JDK 7+，⭐ 现在的标准写法）

```java
// ✘ 老写法：冗长，而且 close() 自己还会抛异常，还得再套一层 try
InputStream in = null;
try {
    in = new FileInputStream("a.txt");
} finally {
    if (in != null) {
        try { in.close(); } catch (IOException ignored) { }
    }
}

// ✔ 自动关闭，凡是实现了 AutoCloseable 的资源都能用
try (InputStream  in  = new FileInputStream("a.txt");
     OutputStream out = new FileOutputStream("b.txt")) {
    in.transferTo(out);
}
```

**原理**：编译器自动生成 `finally` 块调用 `close()`，**关闭顺序与声明顺序相反**。如果 try 块和 close() 都抛了异常，close() 的异常会被记录为"被抑制的异常"（`getSuppressed()`），不会覆盖主异常 —— 这正是老写法做不到的。

---

### 4. throw 与 throws

| 关键字 | 位置 | 作用 |
| ------ | ---- | ---- |
| `throw` | 方法体内 | 实际**抛出**一个异常对象 |
| `throws` | 方法签名上 | **声明**这个方法可能抛出哪些异常，由调用方处理 |

```java
public void read(String path) throws IOException {     // 声明
    if (path == null) {
        throw new IllegalArgumentException("路径不能为空");   // 抛出
    }
}
```

---

### 5. 自定义业务异常（⭐ 行业标配）

配合第十二章的错误码枚举使用：

```java
public class BizException extends RuntimeException {

    private final int code;

    public BizException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.code = errorCode.getCode();
    }

    public int getCode() { return code; }
}
```

```java
// 业务代码里直接抛，不用 try-catch
User user = userMapper.selectById(id);
if (user == null) {
    throw new BizException(ErrorCode.USER_NOT_FOUND);
}
```

**为什么继承 `RuntimeException` 而不是 `Exception`？**

- 受检异常会**污染整条调用链** —— Controller 调 Service 调 Mapper，每一层都被迫 `throws` 或 `try`，代码里全是噪音
- **Spring 事务默认只对 `RuntimeException` 回滚**（见第 7 节），继承 `Exception` 会导致抛异常了事务却不回滚

---

### 6. 全局异常处理（Spring 项目的标准做法）

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BizException.class)
    public Result<?> handleBiz(BizException e) {
        log.warn("业务异常: {}", e.getMessage());          // 业务异常不用打堆栈
        return Result.fail(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public Result<?> handleAll(Exception e) {
        log.error("系统异常", e);                          // ⭐ 系统异常必须打完整堆栈
        return Result.fail(500, "系统繁忙，请稍后重试");   // 不要把堆栈信息返回给前端
    }
}
```

> 有了全局处理器，**业务代码里只管 `throw`，不需要到处写 `try-catch`**。这是现在 Spring Boot 项目的通用架构，面试问"你们项目怎么处理异常"，答的就是这套。

---

### 7. 异常处理的六个坑（重点）

**① 吞异常 —— 最恶劣的写法**

```java
try { ... } catch (Exception e) { }                          // ✘ 出了问题永远查不到
try { ... } catch (Exception e) { e.printStackTrace(); }     // ✘ 生产环境没人看控制台，
                                                             //   而且它不进日志文件
try { ... } catch (Exception e) { log.error("操作失败", e); } // ✔
```

**② 日志丢了堆栈**

```java
log.error("查询失败: " + e.getMessage());       // ✘ 只有一句话，不知道哪一行出的错
log.error("查询失败", e);                       // ✔ 异常对象放最后一个参数，打印完整堆栈
log.error("查询失败, userId={}", userId, e);    // ✔ 占位符 + 异常对象，两者可以共存
```

> `NullPointerException` 的 `getMessage()` 经常是 `null`，只打 message 等于什么都没打。**异常对象一定要作为最后一个参数传给日志方法。**

**③ `catch (Exception e)` 范围过大**

会把 `NullPointerException` 这类**代码 bug** 一起吞掉，掩盖真正的问题。应该精确捕获自己能处理的异常类型。

**④ `try` 该放在循环内还是循环外**

```java
// 单条失败不影响其他条 → try 放循环内
for (Order o : orders) {
    try { handle(o); } catch (Exception e) { log.error("订单处理失败 id={}", o.getId(), e); }
}

// 一条失败就整批终止 → try 包住循环
try {
    for (Order o : orders) { handle(o); }
} catch (Exception e) { ... }
```

按**业务语义**选，不是按性能选。

**⑤ Spring 事务不回滚的两种情况**

```java
// 情况一：抛的是受检异常，默认不回滚
@Transactional
public void save() throws Exception {
    throw new Exception("失败");                       // ✘ 事务不会回滚
}

@Transactional(rollbackFor = Exception.class)          // ✔ 显式指定
public void save() throws Exception { ... }
```

```java
// 情况二：自己 catch 了却不再抛出
@Transactional
public void save() {
    try {
        insert();
        throw new RuntimeException("失败");
    } catch (Exception e) {
        log.error("失败", e);       // ✘ 异常被吃掉了，Spring 感知不到，事务照常提交
    }
}
```

> 这两个是**真实事故的高发点**：数据写了一半没回滚，比直接报错更难排查。

**⑥ 不要用异常控制业务流程**

创建异常对象时要填充调用栈（`fillInStackTrace()`），开销比一次 `if` 判断大几个数量级。能用 `if` 判断的就不要靠抛异常。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 自定义异常继承 `RuntimeException` | 避免污染调用链，也保证事务能回滚 |
| 统一用 `@RestControllerAdvice` 全局处理 | 业务代码只 `throw`，不到处 `try-catch` |
| 捕获后必须记日志，且带上异常对象 | `log.error("msg", e)`，禁止 `printStackTrace()` |
| 禁止空 `catch` 块 | 确实要忽略，也要写注释说明原因 |
| 精确捕获，不要一律 `catch (Exception e)` | 避免掩盖代码 bug |
| 资源关闭用 try-with-resources | 不手写 `finally { close() }` |
| `finally` 里不写 `return` | 会吞掉返回值和异常 |
| 不 catch `Error` | 捕获了也无法处理 |
| 不返回堆栈给前端 | 泄露内部结构，属于安全问题 |
| `@Transactional` 加 `rollbackFor = Exception.class` | 覆盖受检异常 |

---

### 9. 面试重点

**Q1：`Error` 和 `Exception` 的区别？**

`Error` 是 JVM 层面的严重错误（`OutOfMemoryError`、`StackOverflowError`），程序无法也不应该处理；`Exception` 是程序可以捕获处理的异常。两者都继承 `Throwable`。

**Q2：受检异常和非受检异常的区别？**

受检异常在编译期强制要求 `try-catch` 或 `throws`（如 `IOException`），语义是"外部原因导致、可预期可恢复"；非受检异常是 `RuntimeException` 及其子类，编译器不强制处理，通常代表程序 bug。

**Q3：`finally` 一定会执行吗？里面写 `return` 会怎样？**

一定执行，除非 `System.exit()` 或 JVM 崩溃。`finally` 里写 `return` 会**覆盖 try 中的返回值，并吞掉正在抛出的异常**，属于严重坏味道，绝对禁止。

**Q4：try-with-resources 的原理？**

编译器自动生成 `finally` 块调用资源的 `close()`，要求资源实现 `AutoCloseable`。关闭顺序与声明顺序相反；若 try 块和 `close()` 都抛异常，`close()` 的异常会被记录为**被抑制的异常**（`getSuppressed()`），不会覆盖主异常。

**Q5：自定义异常应该继承 `Exception` 还是 `RuntimeException`？**

绝大多数情况继承 `RuntimeException`。受检异常会强制每一层调用方都 `throws` 或 `try`，严重污染代码；而且 Spring 事务默认只对 `RuntimeException` 回滚，继承 `Exception` 会导致抛了异常事务却提交。

**Q6：`throw` 和 `throws` 的区别？**

`throw` 用在方法体内，实际抛出一个异常对象；`throws` 用在方法签名上，声明该方法可能抛出哪些异常，把处理责任交给调用方。

**Q7：Spring 事务在什么情况下不会回滚？**

① 抛的是受检异常且没配 `rollbackFor`；② 异常在方法内部被 `catch` 了没有再抛出；③ 方法不是 `public`；④ 同一个类内部方法直接调用（没走代理）；⑤ 异常被全局处理器提前拦截。

---

## 十五、泛型

### 1. 泛型解决什么问题

```java
// JDK 5 之前：集合里存 Object，取出来要强转，类型错了只能运行期才发现
List list = new ArrayList();
list.add("abc");
list.add(123);
String s = (String) list.get(1);      // ✘ ClassCastException，运行期才炸

// 有了泛型：编译期就拦住
List<String> list = new ArrayList<>();
list.add(123);                        // ✘ 编译错误
String s = list.get(0);               // ✔ 不需要强转
```

泛型的两个作用：**把类型检查提前到编译期** + **免去强制类型转换**。

---

### 2. 泛型类、泛型接口、泛型方法

#### 泛型类

```java
public class Result<T> {
    private int code;
    private String msg;
    private T data;                        // T 由使用方决定

    public T getData() { return data; }
    public void setData(T data) { this.data = data; }
}

Result<User> r = new Result<>();           // 菱形语法（JDK 7+），右边不用重复写类型
```

> 统一返回值 `Result<T>` 是每个项目都有的类，也是泛型最常见的落地场景。

#### 泛型方法

```java
public static <T> T getFirst(List<T> list) {
    return list.isEmpty() ? null : list.get(0);
}
```

> **`<T>` 必须写在返回值类型前面**，这才是泛型方法。泛型类里的普通方法用到类上的 `T`，不算泛型方法。

#### 类型参数的命名惯例

`T` Type、`E` Element、`K` Key、`V` Value、`R` Result。

---

### 3. 类型擦除（面试必问）

**Java 泛型只存在于编译期**，编译后类型参数会被擦除：

- `List<String>` 和 `List<Integer>` 擦除后**都是 `List`**
- `<T>` 擦除为 `Object`；`<T extends Number>` 擦除为上界 `Number`
- 需要的地方由编译器自动插入强制转换

```java
List<String>  a = new ArrayList<>();
List<Integer> b = new ArrayList<>();

a.getClass() == b.getClass();      // true ← 运行期它们就是同一个类
```

**擦除带来的五个限制**（面试常考）：

```java
// ① 不能 new 泛型对象和泛型数组
T t = new T();                 // ✘ 运行期不知道 T 是什么
T[] arr = new T[10];           // ✘

// ② 不能用基本类型作类型参数
List<int> list;                // ✘ 只能写 List<Integer>

// ③ 不能靠泛型重载（擦除后签名完全相同）
void f(List<String> l)  { }
void f(List<Integer> l) { }    // ✘ 编译错误

// ④ 不能对具体泛型做 instanceof
if (obj instanceof List<String>) { }   // ✘ 运行期没有这个信息

// ⑤ 静态成员不能使用类的类型参数
static T data;                 // ✘ T 属于实例，static 属于类
```

> 要在运行期拿到泛型类型，只能通过反射读取**类/字段/方法签名**里保留的信息 —— Jackson 的 `TypeReference`、Spring 的 `ParameterizedTypeReference` 就是靠创建匿名子类，让泛型信息保留在类签名上再反射取出。

---

### 4. 通配符与 PECS（重点）

#### 三种通配符

```java
List<?>              // 无界：不知道具体类型，只能读成 Object，不能写
List<? extends Number>   // 上界：Number 或其子类
List<? super Integer>    // 下界：Integer 或其父类
```

```java
// <? extends T> —— 能读不能写
List<? extends Number> list = new ArrayList<Integer>();
Number n = list.get(0);      // ✔ 读出来一定是 Number
list.add(1);                 // ✘ 编译器不知道它到底是 List<Integer> 还是 List<Double>

// <? super T> —— 能写不能（安全地）读
List<? super Integer> list2 = new ArrayList<Number>();
list2.add(1);                // ✔ 无论实际类型是什么，装得下 Integer
Integer i = list2.get(0);    // ✘ 读出来只能当 Object
```

#### PECS 原则

**Producer Extends, Consumer Super** —— 站在参数的角度看：

- 这个集合是**生产者**（你要**从里面读**数据）→ 用 `<? extends T>`
- 这个集合是**消费者**（你要**往里面写**数据）→ 用 `<? super T>`

```java
// JDK 源码里的经典范例
public static <T> void copy(List<? super T> dest, List<? extends T> src)
//                               消费者(写入)          生产者(读取)
```

#### 为什么 `List<String>` 不是 `List<Object>` 的子类

```java
List<Object> list = new ArrayList<String>();   // ✘ 编译错误
```

如果允许，就能通过 `list.add(123)` 往一个 `List<String>` 里塞 `Integer`，类型安全彻底失守。

> 对比：**数组是协变的**（`String[]` 是 `Object[]` 的子类），代价是把错误推迟到运行期抛 `ArrayStoreException`；**泛型是不变的**，把同样的问题拦在了编译期。

---

### 5. 上界约束 extends

```java
public <T extends Comparable<T>> T max(List<T> list) {     // T 必须可比较
    ...
}

public <T extends Number & Serializable> void f(T t) { }   // 多个上界，类必须写在最前
```

> 这里的 `extends` 表示"上界"，接口也用 `extends` 而不是 `implements`。

---

### 6. 实际项目里的泛型

```java
// ① 统一返回结果 —— 最常见
public class Result<T> {
    private int code;
    private String msg;
    private T data;

    public static <T> Result<T> ok(T data)   { ... }
    public static <T> Result<T> fail(int code, String msg) { ... }
}

// ② 分页封装
public class PageResult<T> {
    private List<T> records;
    private long total;
}

// ③ 框架接口：泛型让一份代码适配所有实体
public interface UserMapper extends BaseMapper<User> { }        // MyBatis-Plus
public interface UserRepo extends JpaRepository<User, Long> { } // Spring Data
```

> 日常开发中**使用泛型**远多于**定义泛型**。能看懂 `BaseMapper<T>` 这类声明、会写 `Result<T>` 就够用了。

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 集合声明一律带泛型 | 裸类型 `List list` 会丢失所有类型检查 |
| 用菱形语法 | `new ArrayList<>()`，不重复写类型 |
| 类型参数用惯例字母 | `T`/`E`/`K`/`V`/`R`，不要用 `A`、`Data` 这类名字 |
| 按 PECS 选通配符 | 只读用 `extends`，只写用 `super` |
| 方法参数优先用通配符 | `List<? extends Number>` 比 `List<Number>` 适用范围大得多 |
| 不要用 `@SuppressWarnings` 掩盖警告 | 泛型警告通常意味着真的有类型隐患 |

---

### 8. 面试重点

**Q1：什么是类型擦除？带来哪些限制？**

Java 泛型只在编译期生效，编译后类型参数被擦除：`<T>` 擦成 `Object`，有上界则擦成上界，必要处由编译器插入强转。限制包括：不能 `new T()` 和 `new T[]`、不能用基本类型作类型参数、不能靠泛型重载、不能对具体泛型做 `instanceof`、静态成员不能用类的类型参数。

**Q2：`List<String>` 和 `List<Integer>` 是同一个类吗？**

是。擦除后两者的 `getClass()` 完全相同，运行期都是 `java.util.ArrayList`。

**Q3：PECS 原则是什么？**

Producer Extends, Consumer Super。要从集合中**读数据**（集合是生产者）用 `<? extends T>`；要往集合中**写数据**（集合是消费者）用 `<? super T>`。

**Q4：`List<Object>`、`List<?>`、`List` 有什么区别？**

`List<Object>` 明确装 `Object`，可以 `add` 任意对象，但**只能接收 `List<Object>` 实参**；`List<?>` 能接收任何 `List`，但**除了 `null` 什么都不能 add**；`List` 是裸类型，能接收任何 `List` 也能随便 add，**完全丧失类型检查**，属于遗留写法，禁止使用。

**Q5：为什么 `List<String>` 不是 `List<Object>` 的子类？**

因为泛型是**不变（invariant）**的。如果允许，就能把 `Integer` 塞进 `List<String>`，破坏类型安全。数组是协变的，代价就是运行期才抛 `ArrayStoreException`。

**Q6：泛型能用基本类型吗？**

不能。擦除后类型参数是 `Object`，而基本类型不是对象。只能用包装类，如 `List<Integer>`，因此会有自动装箱的开销。

---

## 十六、集合类

### 1. 集合体系

```text
Collection（单列集合）
├─ List ——— 有序、可重复、有索引
│   ├─ ⭐ArrayList        动态数组，查询快
│   ├─ LinkedList        双向链表，实际很少用
│   └─ Vector            已淘汰（方法加 synchronized）
├─ Set ———— 无序、不可重复
│   ├─ ⭐HashSet          底层就是 HashMap
│   ├─ LinkedHashSet     保持插入顺序
│   └─ TreeSet           按大小排序（红黑树）
└─ Queue ——— 队列
    ├─ ⭐ArrayDeque       用作栈和队列的首选
    └─ PriorityQueue     优先队列（小顶堆）

Map（双列集合，不属于 Collection）
├─ ⭐HashMap              数组 + 链表 + 红黑树
├─ LinkedHashMap         保持插入/访问顺序，可实现 LRU
├─ TreeMap               按 key 排序（红黑树）
├─ Hashtable             已淘汰
└─ ⭐ConcurrentHashMap    并发场景首选
```

> **`Map` 不是 `Collection` 的子接口**，这是个常考的细节。
>
> JDK 21 新增了 `SequencedCollection` / `SequencedMap` 接口，给"有确定顺序"的集合（`List`、`LinkedHashMap`、`ArrayDeque` 等）统一了 `getFirst()`、`getLast()`、`addFirst()`、`reversed()` 这几个方法 —— 以前取第一个元素 `List` 写 `get(0)`、`Deque` 写 `peekFirst()`、`LinkedHashSet` 只能靠迭代器，现在写法一致了。

---

### 2. List

| 对比项 | `ArrayList` | `LinkedList` |
| ------ | ----------- | ------------ |
| 底层结构 | 动态数组 `Object[]` | 双向链表 |
| 随机访问 `get(i)` | ⭐**O(1)** | O(n)，要从头遍历 |
| 尾部添加 | O(1)（均摊） | O(1) |
| 中间插入/删除 | O(n)，要整体移动元素 | 找位置 O(n) + 改指针 O(1) |
| 内存开销 | 小，连续存储 | 大，每个节点多两个指针 |
| 使用场景 | ⭐**默认就用它** | 几乎不用 |

> `LinkedList` 看起来"增删快"，但**要先 O(n) 找到位置**，实际很少比 `ArrayList` 快。它唯一还算合理的场景是**只在头尾频繁增删、且完全不做随机访问**（队列 / 栈 / LRU 链表），而这种需求用 `ArrayDeque` 更快也更省内存。**除此之外业务代码无脑用 `ArrayList`。**

#### ArrayList 扩容机制（面试必问）

- 底层是 `Object[] elementData`
- 无参构造时是**空数组**，**第一次 `add()` 才真正分配长度 10**（懒加载）
- 扩容公式：`newCapacity = oldCapacity + (oldCapacity >> 1)`，即 **1.5 倍**
- 扩容通过 `Arrays.copyOf()` 复制整个数组，**代价很高**

```java
List<User> list = new ArrayList<>(1000);   // ⭐ 已知数据量就指定初始容量，避免反复扩容拷贝
```

---

### 3. Set

| 实现类 | 底层 | 特点 |
| ------ | ---- | ---- |
| ⭐`HashSet` | `HashMap`（value 是一个固定常量） | 无序，O(1) |
| `LinkedHashSet` | `LinkedHashMap` | 保持**插入顺序** |
| `TreeSet` | 红黑树 | 按大小**排序**，O(log n)，元素需实现 `Comparable` |

**去重靠 `hashCode()` + `equals()`** —— 自定义对象放进 `Set` 前必须重写这两个方法，否则去重完全失效（原因见第十三章）。

---

### 4. HashMap（重中之重）

#### 底层结构

```text
JDK 7：数组 + 链表        （头插法，并发扩容会形成环形链表 → CPU 100%）
JDK 8：数组 + 链表 + 红黑树（改为尾插法）
```

| 参数 | 值 |
| ---- | ---- |
| 默认初始容量 | 16 |
| 负载因子 | 0.75 |
| 扩容阈值 | 容量 × 0.75（默认 12） |
| 扩容倍数 | **2 倍** |
| 树化条件 | 链表长度 ≥ **8** **且** 数组长度 ≥ **64** |
| 退化条件 | 树节点数 ≤ **6** 退回链表 |

> 注意树化的**两个条件**：链表到 8 但数组长度不足 64 时，**优先扩容而不是树化**。只答"链表超过 8 就变红黑树"是不完整的。

#### put 流程

1. **算 hash**：`(h = key.hashCode()) ^ (h >>> 16)` —— 扰动函数，让高 16 位也参与运算，减少冲突
2. **定位桶**：`(n - 1) & hash`
3. 桶为空 → 直接放入
4. 桶不为空 → 比较 `hash` 和 `equals()`，相同则**覆盖 value**
5. 是红黑树 → 走树插入；是链表 → **尾插**，达到树化条件则树化
6. `size` 超过阈值 → 扩容

#### 三个"为什么"

**为什么容量必须是 2 的幂？**

为了用 `(n - 1) & hash` 代替 `hash % n`。`n` 是 2 的幂时 `n - 1` 的低位全是 1，与运算的结果和取模完全等价，但**位运算快得多**。传入非 2 的幂时，`HashMap` 会用 `tableSizeFor()` 向上取到最近的 2 的幂。

**为什么负载因子是 0.75？**

时间和空间的折中：太小浪费空间且频繁扩容，太大则哈希冲突增多、链表变长导致查询变慢。0.75 是基于泊松分布的经验值。

**为什么树化阈值是 8？**

负载因子 0.75 时，按泊松分布计算，一个桶里出现 8 个元素的概率约**千万分之一**。这个阈值是**兜底设计**，正常情况根本不会触发树化 —— 因为红黑树节点占用的空间是链表节点的两倍，不到万不得已不用。

#### HashMap 不是线程安全的

JDK 7 头插法在并发扩容时会导致链表反转、形成**环形链表**，后续 `get()` 直接 CPU 100%。JDK 8 改成尾插解决了成环问题，**但仍然不线程安全** —— 并发 `put` 会丢数据。并发场景必须用 `ConcurrentHashMap`。

---

### 5. LinkedHashMap 与 TreeMap

**`LinkedHashMap`** 继承 `HashMap`，额外用双向链表维护顺序。默认是插入顺序，构造时传 `accessOrder = true` 变为**访问顺序**，可以直接实现 **LRU 缓存**：

```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);          // ⭐ true = 按访问顺序排列
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;              // 超出容量就淘汰最久未访问的
    }
}
```

> 手写 LRU 是高频面试题，用 `LinkedHashMap` 只要这十行。

**`TreeMap`** 底层红黑树，按 key 排序，增删查都是 O(log n)。需要**排序或范围查询**（`firstKey()`、`headMap()`、`subMap()`）时用。

---

### 6. 排序：Comparable 与 Comparator

排序是业务里最高频的操作之一，两者的分工是固定的：

| 对比项 | `Comparable` | `Comparator` |
| ------ | ------------ | ------------ |
| 位置 | 类**自己实现**，`compareTo(o)` | **外部**定义，`compare(a, b)` |
| 语义 | 类的**自然顺序**，只能有一种 | 临时的排序规则，可以有无数种 |
| 所在包 | `java.lang` | `java.util` |
| 典型场景 | `TreeMap` / `TreeSet` 的元素、`Collections.sort(list)` | ⭐`list.sort(cmp)`、`stream().sorted(cmp)`、`Arrays.sort(arr, cmp)` |

> 一句话选型：**改不了源码、或者同一个类要按多种规则排序，就用 `Comparator`**；只有"这个类天生就有一种顺序"（如金额、版本号）才实现 `Comparable`。

**返回值约定**：`a < b` 返回负数、相等返回 0、`a > b` 返回正数。业务代码不用自己写比较逻辑，直接用链式 API：

```java
list.sort(Comparator.comparing(User::getAge));                        // ⭐ 升序
list.sort(Comparator.comparing(User::getAge).reversed());             // 降序
list.sort(Comparator.comparing(User::getAge)
                    .thenComparing(User::getName));                   // ⭐ 多级排序
list.sort(Comparator.comparingInt(User::getAge));                     // 基本类型用 comparingInt/Long/Double，避免装箱
list.sort(Comparator.comparing(User::getName,
          Comparator.nullsLast(Comparator.naturalOrder())));          // 字段可能为 null
```

三个坑：

**① 不要用减法实现 `compare`**

```java
(a, b) -> a.getAge() - b.getAge();          // ✘ 两数相差超过 21 亿时 int 溢出，排序结果错乱
(a, b) -> Integer.compare(a.getAge(), b.getAge());   // ✔
```

**② 比较器必须满足传递性**

自己写 `if-else` 返回 1 / -1、漏掉"相等返回 0"的情况时，JDK 7+ 的 TimSort 会直接抛：

```text
IllegalArgumentException: Comparison method violates its general contract!
```

这个异常**只在数据量大到触发归并时才出现**，测试环境常常发现不了，是典型的线上偶发问题。

**③ `TreeSet` / `TreeMap` 判重靠比较结果，不靠 `equals()`**

```java
Set<User> set = new TreeSet<>(Comparator.comparing(User::getAge));
set.add(user18a);
set.add(user18b);        // ⚠️ 年龄相同 → compare 返回 0 → 被当成同一个元素，size 仍是 1
```

> 这正是 `TreeSet` 和 `HashSet` 的根本区别：`HashSet` 用 `hashCode()` + `equals()` 去重，`TreeSet` 用**比较结果是否为 0** 去重。所以给 `TreeSet` 传比较器时，一定要保证"业务上不同的对象比较结果不为 0"，必要时加上 id 作为最后一级比较。

---

### 7. 线程安全的集合

| 方案 | 评价 |
| ---- | ---- |
| `Vector` / `Hashtable` | ✘ 已淘汰，整表加锁，性能极差 |
| `Collections.synchronizedList(...)` | 包装类，仍是整表锁，遍历还要自己加锁 |
| ⭐`CopyOnWriteArrayList` | **读多写极少**时用（配置、白名单） |
| ⭐`ConcurrentHashMap` | 并发 Map 的唯一选择 |

#### ConcurrentHashMap

| 版本 | 实现方式 | 锁粒度 |
| ---- | -------- | ------ |
| JDK 7 | `Segment` **分段锁**，默认 16 段 | 一段（1/16 的数据） |
| JDK 8 | 放弃分段锁，改为 **CAS + `synchronized`** | ⭐**单个桶的头节点** |

JDK 8 的做法：桶为空时用 **CAS** 直接插入，桶非空时只 `synchronized` 锁住**该桶的头节点**，锁粒度从"一段"降到"一个桶"，并发度大幅提升。

**为什么 `ConcurrentHashMap` 的 key 和 value 都不允许为 `null`？**

因为 `get()` 返回 `null` 时无法区分"key 不存在"还是"value 本身就是 null"。单线程下可以用 `containsKey()` 二次确认，但并发环境中两次调用之间状态可能已经变了，这个歧义**无法消除**，所以干脆禁止。（`HashMap` 允许一个 `null` key 和多个 `null` value。）

#### CopyOnWriteArrayList

写操作时复制一份新数组、改完再替换引用，**读操作完全无锁**。适合读多写少；写频繁会疯狂产生新数组，GC 压力极大。

---

### 8. 集合的常见坑

**① `subList()` 返回的是视图，不是新集合**

```java
List<Integer> sub = list.subList(0, 3);
sub.set(0, 99);        // ⚠️ 原 list 也被改了
list.add(1);
sub.get(0);            // ✘ ConcurrentModificationException，原集合结构一变视图就失效

List<Integer> copy = new ArrayList<>(list.subList(0, 3));   // ✔ 要独立副本就拷一份
```

**② `List.of()` / `Map.of()` 是不可变集合**（JDK 9+）

```java
List<String> list = List.of("a", "b");
list.add("c");         // ✘ UnsupportedOperationException
```

和 `Arrays.asList()` 一样，要可变就 `new ArrayList<>(...)` 包一层（见第七章）。

**③ `HashMap` 指定初始容量要考虑负载因子**

```java
new HashMap<>(16);     // ✘ 存到第 13 个就扩容了

// ✔ 初始容量 = 预期元素数 / 0.75 + 1
new HashMap<>((int) (expectedSize / 0.75f) + 1);
Maps.newHashMapWithExpectedSize(expectedSize);              // Guava 提供的写法
```

**④ `Collectors.toMap()` 的两个雷**

```java
// key 重复 → IllegalStateException
users.stream().collect(Collectors.toMap(User::getName, u -> u));
users.stream().collect(Collectors.toMap(User::getName, u -> u, (a, b) -> a));  // ✔ 指定冲突策略

// value 为 null → NPE（HashMap 允许 null value，但 toMap 内部用了 merge 会抛）
```

**⑤ 遍历中删除元素抛 CME（fail-fast）** —— 见第五章，用 `removeIf()`。

> **术语**：`ArrayList`、`HashMap` 这类集合的迭代器是 **fail-fast（快速失败）** —— 一旦发现结构被并发修改（`modCount != expectedModCount`）就立刻抛 `ConcurrentModificationException`，宁可报错也不返回错误数据；`CopyOnWriteArrayList`、`ConcurrentHashMap` 的迭代器是 **fail-safe（安全失败）** —— 遍历的是副本或不校验 `modCount`，边遍历边改不会抛异常，但**可能读不到最新数据**。面试常直接用这两个术语提问。

---

### 9. 怎么选

| 需求 | 用什么 |
| ---- | ---- |
| 一般的列表 | ⭐`ArrayList` |
| 去重 | ⭐`HashSet` |
| 去重 + 保持插入顺序 | `LinkedHashSet` |
| 去重 + 排序 | `TreeSet` |
| 一般的键值对 | ⭐`HashMap` |
| 键值对 + 保持顺序 / LRU | `LinkedHashMap` |
| 键值对 + 按 key 排序或范围查询 | `TreeMap` |
| 并发场景的 Map | ⭐`ConcurrentHashMap` |
| 队列 / 栈 | ⭐`ArrayDeque` |
| 读多写极少的列表 | `CopyOnWriteArrayList` |

---

### 10. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 声明用接口类型 | `List<User> list = new ArrayList<>();` |
| 已知数据量就指定初始容量 | 避免反复扩容拷贝 |
| `HashMap` 初始容量按 `n / 0.75 + 1` 算 | 直接传 `n` 仍然会扩容 |
| 并发场景用 `ConcurrentHashMap` | 不用 `Hashtable`、不用 `synchronizedMap` |
| 自定义对象作 key / 存 Set 必须重写 `equals` + `hashCode` | 否则去重和查找全部失效 |
| 遍历 Map 用 `entrySet()` | 避免二次哈希查找 |
| 遍历中删除用 `removeIf()` | 直接 `remove()` 抛 CME |
| `Collectors.toMap()` 必须传冲突策略 | 否则 key 重复直接抛异常 |
| 返回空集合而不是 `null` | `Collections.emptyList()` |

---

### 11. 面试重点

**Q1：`ArrayList` 和 `LinkedList` 的区别？**

`ArrayList` 底层是动态数组，随机访问 O(1)，中间增删要移动元素 O(n)，内存连续开销小；`LinkedList` 是双向链表，随机访问 O(n)，增删指针本身 O(1) 但**找位置仍需 O(n)**，每个节点还要多存两个指针。实际开发几乎总是用 `ArrayList`。

**Q2：`ArrayList` 的扩容机制？**

无参构造时是空数组，第一次 `add()` 才分配长度 10；容量不足时按 **1.5 倍**扩容（`oldCapacity + (oldCapacity >> 1)`），通过 `Arrays.copyOf()` 复制整个数组。所以已知数据量应当指定初始容量。

**Q3：`HashMap` 的底层结构？put 的流程？**

JDK 8 起是**数组 + 链表 + 红黑树**。put 时先用扰动函数算 hash（`hashCode() ^ (h >>> 16)`），再用 `(n-1) & hash` 定位桶；桶空则直接放，非空则比较 hash 和 `equals()`，相同则覆盖，否则尾插到链表或插入红黑树；最后判断是否需要扩容。

**Q4：`HashMap` 什么时候转红黑树？**

**链表长度 ≥ 8 且数组长度 ≥ 64** 时才树化；如果数组长度不足 64，会**优先扩容**而不是树化。树节点数 ≤ 6 时退回链表。

**Q5：`HashMap` 的容量为什么必须是 2 的幂？**

为了用 `(n - 1) & hash` 代替 `hash % n`。`n` 为 2 的幂时 `n - 1` 低位全是 1，与运算和取模等价，但位运算快得多，同时能让元素分布更均匀。

**Q6：`HashMap` 为什么线程不安全？JDK 7 和 8 的区别？**

JDK 7 用头插法，并发扩容会导致链表反转形成**环形链表**，后续 `get()` 陷入死循环导致 CPU 100%；JDK 8 改为尾插法解决了成环，但**并发 put 仍会丢失数据**，依然不是线程安全的。

**Q7：`ConcurrentHashMap` 是怎么保证线程安全的？**

JDK 7 用 `Segment` 分段锁，默认 16 段；JDK 8 放弃分段锁，改为**桶为空时 CAS 插入、桶非空时 `synchronized` 锁住桶的头节点**，把锁粒度从"一段"降到"一个桶"，并发度大幅提升。

**Q8：`ConcurrentHashMap` 为什么不允许 null 键值？**

`get()` 返回 `null` 时无法区分"key 不存在"和"value 就是 null"。单线程可以用 `containsKey()` 二次确认，但并发下两次调用之间状态可能已变，歧义无法消除，因此直接禁止。

**Q9：怎么用 `LinkedHashMap` 实现 LRU？**

继承 `LinkedHashMap`，构造时传 `accessOrder = true` 让它按访问顺序排列，再重写 `removeEldestEntry()`，当 `size() > capacity` 时返回 `true` 自动淘汰最久未访问的元素。

**Q10：`HashSet` 是怎么去重的？**

`HashSet` 底层就是一个 `HashMap`（value 是固定常量）。添加元素时先比 `hashCode()` 定位桶，再用 `equals()` 判断是否相同。因此自定义对象**必须同时重写 `equals()` 和 `hashCode()`**，否则去重失效。

**Q11：`Comparable` 和 `Comparator` 的区别？**

`Comparable` 由类自己实现 `compareTo()`，定义的是**唯一的自然顺序**，在 `java.lang` 下；`Comparator` 是外部定义的 `compare()`，可以有多个、可随时切换，在 `java.util` 下。改不了源码或需要多种排序规则时用 `Comparator`。注意 `compare` 不要用减法（会溢出），且必须满足传递性，否则 TimSort 会抛 `Comparison method violates its general contract`。

**Q12：什么是 fail-fast？哪些集合是 fail-safe？**

fail-fast 指迭代过程中一旦检测到集合被结构性修改（`modCount != expectedModCount`）立刻抛 `ConcurrentModificationException`，`ArrayList`、`HashMap` 都是这种；fail-safe 指遍历副本或不做校验，边遍历边改不抛异常但可能读到旧数据，`CopyOnWriteArrayList`、`ConcurrentHashMap` 属于这一类。

---

## 十七、Lambda 与 Stream API

**A · Lambda 表达式**

### 1. 本质与语法

Lambda 的本质是**函数式接口的实例**，用来替代啰嗦的匿名内部类。

```java
// 匿名内部类
Runnable r = new Runnable() {
    @Override public void run() { System.out.println("hi"); }
};

// Lambda —— 完全等价
Runnable r = () -> System.out.println("hi");
```

```java
()      -> System.out.println("hi")     // 无参
x       -> x * 2                        // 单参数可省略括号
(x, y)  -> x + y                        // 多参数
(x, y)  -> { return x + y; }            // 多行必须写 {} 和 return
```

参数类型可以省略，由编译器推断。**Lambda 只能用于函数式接口**（有且仅有一个抽象方法）。

---

### 2. 方法引用

Lambda 体只是"调用一个已有方法"时，可以进一步简写为 `::`：

| 形式 | 示例 | 等价的 Lambda |
| ---- | ---- | ------------- |
| 类名::静态方法 | `Integer::parseInt` | `s -> Integer.parseInt(s)` |
| 对象::实例方法 | `System.out::println` | `x -> System.out.println(x)` |
| ⭐类名::实例方法 | `User::getName` | `u -> u.getName()` |
| 类名::new | `User::new` | `() -> new User()` |

> 第三种（类名::实例方法）在 Stream 里出现频率最高，`map(User::getName)` 就是它。

---

### 3. 常用函数式接口

都在 `java.util.function` 包下，记住这四个基本够用：

| 接口 | 抽象方法 | 语义 | 出现在 |
| ---- | -------- | ---- | ------ |
| ⭐`Function<T,R>` | `R apply(T)` | 转换 | `map()` |
| ⭐`Predicate<T>` | `boolean test(T)` | 判断 | `filter()` |
| ⭐`Consumer<T>` | `void accept(T)` | 消费 | `forEach()` |
| ⭐`Supplier<T>` | `T get()` | 提供 | `orElseGet()` |
| `BiFunction<T,U,R>` | `R apply(T,U)` | 两参转换 | `reduce()` |

---

**B · Stream API**

### 4. Stream 是什么

**Stream 不是数据结构，是对数据源的一条操作管道**。固定三段式：

```java
list.stream()                        // ① 创建
    .filter(u -> u.getAge() > 18)    // ② 中间操作（可多个，惰性）
    .map(User::getName)              // ② 中间操作
    .collect(Collectors.toList());   // ③ 终端操作（触发执行）
```

三个关键特性：

- **惰性求值**：没有终端操作时，中间操作**一行都不会执行**
- **一次性**：Stream 用完即废，再次使用抛 `IllegalStateException`
- **不修改源数据**：返回新结果，原集合不变

创建方式：

```java
list.stream();                  // ⭐ 从集合创建
Arrays.stream(arr);             // 从数组
Stream.of("a", "b");            // 从若干元素
IntStream.range(0, 10);         // 数值区间
```

---

### 5. 中间操作

| 方法 | 说明 |
| ---- | ---- |
| ⭐`filter` | 按条件过滤 |
| ⭐`map` | **一对一**转换 |
| `flatMap` | **一对多**展平：`List<List<T>>` → `List<T>` |
| `distinct` | 去重（依赖 `equals`/`hashCode`） |
| `sorted` | 排序 |
| `limit` / `skip` | 取前 n 个 / 跳过前 n 个 |
| `peek` | 仅用于调试，不要放业务逻辑 |

```java
// flatMap：把每个订单的明细摊平成一个列表
List<String> itemNames = orders.stream()
        .flatMap(o -> o.getItems().stream())     // Stream<List<Item>> → Stream<Item>
        .map(Item::getName)
        .collect(Collectors.toList());

// 排序
.sorted(Comparator.comparing(User::getAge))                              // 升序
.sorted(Comparator.comparing(User::getAge).reversed())                   // 降序
.sorted(Comparator.comparing(User::getAge).thenComparing(User::getName)) // 多级排序
```

---

### 6. 终端操作

| 方法 | 说明 |
| ---- | ---- |
| ⭐`collect` | 收集成集合 / Map |
| `forEach` | 遍历 |
| ⭐`anyMatch` / `allMatch` / `noneMatch` | 条件判断，**短路** |
| ⭐`findFirst` / `findAny` | 取一个元素，返回 `Optional` |
| `count` | 计数 |
| `min` / `max` | 最值 |
| `reduce` | 归约 |

---

### 7. Collectors（实际用得最多的部分）

```java
// 转集合
.collect(Collectors.toList());
.collect(Collectors.toSet());

// ⭐ 转 Map —— 业务里最高频，必须传冲突策略
.collect(Collectors.toMap(User::getId, u -> u));
.collect(Collectors.toMap(User::getId, u -> u, (a, b) -> a));    // 重复 key 保留前者

// ⭐ 分组 —— 替代手写 for + Map.computeIfAbsent
Map<Integer, List<User>> byAge =
        users.stream().collect(Collectors.groupingBy(User::getAge));

// 分组 + 统计
Map<Integer, Long> countByAge =
        users.stream().collect(Collectors.groupingBy(User::getAge, Collectors.counting()));

// 分区：按 boolean 分成两组
Map<Boolean, List<User>> parts =
        users.stream().collect(Collectors.partitioningBy(u -> u.getAge() > 18));

// 拼接字符串
.collect(Collectors.joining(", ", "[", "]"));

// 求和 / 平均
.collect(Collectors.summingInt(User::getAge));
.collect(Collectors.averagingInt(User::getAge));
```

> 业务代码里 90% 的场景就三个：**`toList`、`toMap`、`groupingBy`**。

**最典型的实战用法** —— 批量查询后转 Map，消灭 N+1（呼应第五章）：

```java
Map<Long, User> userMap = userMapper.selectBatchIds(ids).stream()
        .collect(Collectors.toMap(User::getId, u -> u));

for (Order o : orders) {
    User u = userMap.get(o.getUserId());     // O(1) 查找，不再循环查库
}
```

---

### 8. Optional

用来显式表达"这个值可能不存在"，替代到处 `if (x != null)`。

```java
Optional<User> opt = Optional.ofNullable(user);

opt.orElse(defaultUser);                          // 为空返回默认值
opt.orElseGet(() -> buildDefault());              // ⭐ 为空才执行，惰性
opt.orElseThrow(() -> new BizException(...));     // ⭐ 为空抛异常，最常用
opt.map(User::getName).orElse("未知");
opt.ifPresent(u -> log.info("{}", u));
```

**`orElse` 和 `orElseGet` 的区别**（面试常问）：

```java
opt.orElse(buildDefault());          // ✘ 无论 opt 是否有值，buildDefault() 都会执行
opt.orElseGet(() -> buildDefault()); // ✔ 只有 opt 为空时才执行
```

默认值是常量用 `orElse`，是**方法调用或耗时操作**必须用 `orElseGet`。

三条使用纪律：

- **禁止裸用 `get()`** —— 不判空直接 `get()` 只是把 NPE 换了个名字（`NoSuchElementException`）
- `Optional` **只作为返回值**，不要用作字段和方法参数
- 别为了用而用，简单的 `if (x != null)` 该写还是写

---

### 9. Stream 的坑

**① Stream 是一次性的**

```java
Stream<User> s = list.stream();
s.filter(...).count();
s.map(...).count();       // ✘ IllegalStateException: stream has already been operated upon
```

**② 不能 `break`**

Lambda 里 `return` 只相当于 `continue`。需要提前终止用 `anyMatch()`、`findFirst()`，或 JDK 9+ 的 `takeWhile()`。

**③ 并行流 `parallelStream()` 要慎用**

```java
list.parallelStream().forEach(this::callRemoteApi);     // ✘ 高风险
```

三个原因：

- 默认使用 **`ForkJoinPool.commonPool()`，是整个 JVM 全局共享的线程池**，你的慢任务会拖垮其他所有并行流
- 里面做 **IO 或远程调用**会长时间占住线程，直接把公共池打满
- 数据量小的时候，**拆分和合并的开销比串行还大**

> **唯一合适的场景**：纯 CPU 计算、数据量在十万级以上、且中间不涉及任何 IO —— 例如一次性对大数组做数学运算或排序。**Web 请求线程里一律不要用**；真要并行请用自己的线程池 + `CompletableFuture`（见第二十章）。

**④ `Collectors.toMap()` 的 value 不能为 `null`**，否则抛 NPE（`HashMap` 本身是允许的，但 `toMap` 内部走 `merge` 会检查）。

**⑤ 不要为了用而用**

```java
list.stream().forEach(System.out::println);    // ✘ 绕了一圈
list.forEach(System.out::println);             // ✔ 集合自带 forEach
for (User u : list) { }                        // ✔ 简单遍历这样最直观，断点也好打
```

> **判断标准**：涉及多级过滤、转换、分组、聚合才用 Stream；单纯遍历用 for-each。Stream 链过长（超过 5 步）时可读性和调试体验都会变差。

---

### 10. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 简单遍历用 for-each | 多级过滤/分组/聚合才用 Stream |
| 能用方法引用就用 | `map(User::getName)` 比 Lambda 更清晰 |
| `Collectors.toMap()` 必须传冲突策略 | 否则 key 重复直接抛异常 |
| 禁止使用 `parallelStream()` | 共享公共线程池，风险远大于收益 |
| 禁止裸用 `Optional.get()` | 用 `orElse` / `orElseGet` / `orElseThrow` |
| `Optional` 只用作返回值 | 不做字段、不做方法参数 |
| 默认值是方法调用时用 `orElseGet` | `orElse` 的参数一定会被求值 |
| Stream 链不超过 5 步 | 过长就拆成多个变量，便于调试 |

---

### 11. 面试重点

**Q1：Lambda 的本质是什么？和匿名内部类的区别？**

Lambda 是函数式接口的实例。区别：匿名内部类会编译出独立的 `.class` 文件、可用于任意接口和抽象类、其中 `this` 指向自身；Lambda 基于 `invokedynamic` 在运行期生成，不产生额外 class，只能用于函数式接口，`this` 指向外部类。

**Q2：Stream 的中间操作和终端操作有什么区别？**

中间操作返回 Stream、可以链式调用、是**惰性**的 —— 没有终端操作就一行都不会执行；终端操作触发整条管道真正执行并产出结果，执行后 Stream 即失效。

**Q3：`map` 和 `flatMap` 的区别？**

`map` 是一对一转换，`Stream<T>` → `Stream<R>`；`flatMap` 是一对多展平，把每个元素映射成一个 Stream 再合并，常用于 `List<List<T>>` → `List<T>`。

**Q4：`parallelStream()` 有什么风险？**

它默认使用全局共享的 `ForkJoinPool.commonPool()`，慢任务会拖垮 JVM 内所有并行流；里面做 IO 或远程调用会长时间占用线程打满公共池；数据量小时拆分合并开销反而比串行更大。生产代码应使用自定义线程池而非并行流。

**Q5：`orElse` 和 `orElseGet` 的区别？**

`orElse(x)` 的参数 `x` **无论 Optional 是否有值都会被求值**；`orElseGet(supplier)` 只有为空时才执行。默认值需要方法调用或耗时计算时必须用 `orElseGet`。

**Q6：Stream 可以复用吗？**

不可以。Stream 是一次性的，执行过终端操作后再次使用会抛 `IllegalStateException`。需要多次处理就重新从数据源创建。

---

## 十八、常用工具类（BigDecimal、日期时间、Optional）

**A · BigDecimal（金额计算）**

### 1. 为什么金额必须用 BigDecimal

```java
0.1 + 0.2;      // 0.30000000000000004
1.0 - 0.9;      // 0.09999999999999998
```

`float`/`double` 遵循 IEEE 754 用二进制表示小数，`0.1` 换算成二进制是无限循环，只能取近似值。**金额差一分钱就是生产事故**，所以涉及钱、比例、税率的计算一律用 `BigDecimal`。

> 数据库对应 `DECIMAL(20, 2)`，Java 侧用 `BigDecimal`。**绝不用 `double` 存金额。**

---

### 2. 创建：必须用字符串构造

```java
new BigDecimal(0.1);        // ✘ 0.1000000000000000055511151231257827...
                            //   传进去的 double 本身就已经不精确了
new BigDecimal("0.1");      // ✔ 精确的 0.1
BigDecimal.valueOf(0.1);    // ✔ 内部走 Double.toString()，也是精确的

BigDecimal.ZERO;  BigDecimal.ONE;  BigDecimal.TEN;      // 内置常量
```

---

### 3. 运算：不可变，必须接收返回值

```java
BigDecimal a = new BigDecimal("10.00");

a.add(new BigDecimal("5"));         // ✘ 毫无效果，a 还是 10.00
a = a.add(new BigDecimal("5"));     // ✔
```

| 方法 | 运算 |
| ---- | ---- |
| `add(b)` | 加 |
| `subtract(b)` | 减 |
| `multiply(b)` | 乘 |
| ⭐`divide(b, scale, RoundingMode)` | 除 |

**除法必须指定精度和舍入模式**，否则除不尽时（如 `10 / 3`）会抛 `ArithmeticException: Non-terminating decimal expansion`。

```java
a.divide(b, 2, RoundingMode.HALF_UP);      // 保留 2 位，四舍五入
```

| 舍入模式 | 说明 |
| -------- | ---- |
| ⭐`HALF_UP` | 四舍五入，最常用 |
| `HALF_EVEN` | 银行家舍入，能减少大批量累计误差 |
| `DOWN` | 直接截断 |
| `UP` | 远离零方向进位 |

---

### 4. 比较：用 compareTo，不用 equals

```java
new BigDecimal("1.0").equals(new BigDecimal("1.00"));      // ✘ false！equals 会连精度一起比
new BigDecimal("1.0").compareTo(new BigDecimal("1.00"));   // ✔ 0，值相等

// 判断是否为 0 的正确写法
a.compareTo(BigDecimal.ZERO) == 0;      // ✔
a.equals(BigDecimal.ZERO);              // ✘ "0.00" 会返回 false
```

> `BigDecimal.equals()` 要求**值和 scale（小数位数）都相同**。这是最容易踩的坑之一。

---

### 5. 格式化

```java
a.setScale(2, RoundingMode.HALF_UP);    // ⭐ 保留两位小数
a.stripTrailingZeros();                 // 去掉末尾多余的 0
a.toPlainString();                      // ⭐ 避免科学计数法
```

> 大数字或小数位很多时，`toString()` 可能输出 `1E+10` 这样的科学计数法，展示给用户或拼接参数时要用 `toPlainString()`。

---

**B · 日期时间 API（JDK 8 `java.time`）**

### 6. 为什么不用 Date / SimpleDateFormat

| 老 API | 问题 |
| ------ | ---- |
| `Date` | 年份从 1900 起、月份从 0 起，设计混乱；**可变**，谁都能改 |
| `Calendar` | API 极其难用 |
| `SimpleDateFormat` | ⭐**线程不安全** —— 定义成静态字段共享时会解析出**错误的日期** |

> 新代码一律用 `java.time`。这套 API 的所有类都是**不可变、线程安全**的。

---

### 7. 核心类

| 类 | 用途 |
| ---- | ---- |
| ⭐`LocalDate` | 日期，如 `2026-08-11` |
| ⭐`LocalDateTime` | 日期 + 时间 |
| `LocalTime` | 时间 |
| `Instant` | 时间戳（UTC 时刻） |
| `Duration` | 时间间隔（时分秒） |
| `Period` | 日期间隔（年月日） |
| ⭐`DateTimeFormatter` | 格式化 / 解析，**线程安全** |
| `ZonedDateTime` | 带时区的日期时间 |

---

### 8. 常用操作

```java
LocalDate today = LocalDate.now();
LocalDate d     = LocalDate.of(2026, 8, 11);
LocalDateTime now = LocalDateTime.now();

// 加减：返回新对象，原对象不变
today.plusDays(7);
today.minusMonths(1);
today.withDayOfMonth(1);                              // 本月第一天
today.with(TemporalAdjusters.lastDayOfMonth());       // 本月最后一天

// 比较
d1.isBefore(d2);  d1.isAfter(d2);  d1.isEqual(d2);

// 间隔
ChronoUnit.DAYS.between(d1, d2);                      // ⭐ 相差多少天
Duration.between(t1, t2).toMinutes();                 // 相差多少分钟
```

---

### 9. 格式化

```java
// ⭐ 定义成静态常量：不可变、线程安全、可复用
private static final DateTimeFormatter FMT =
        DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

String s = FMT.format(LocalDateTime.now());
LocalDateTime t = LocalDateTime.parse("2026-08-11 10:00:00", FMT);
```

格式符：`yyyy` 年、`MM` 月、`dd` 日、`HH` 24 小时制、`mm` 分、`ss` 秒。

> ⚠️ **绝对不要用大写 `YYYY`**。它表示"基于周的年份"，在跨年的那一周会算成下一年 —— 比如 2019-12-30 用 `YYYY` 格式化会输出 **2020**。这是被反复报道过的真实线上事故。

---

### 10. 与时间戳 / Date 互转

```java
// LocalDateTime → 毫秒时间戳
long ms = ldt.atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();

// 毫秒时间戳 → LocalDateTime
LocalDateTime.ofInstant(Instant.ofEpochMilli(ms), ZoneId.systemDefault());

// Date → LocalDateTime（对接老代码时用）
date.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
```

---

**C · 其他常用工具**

### 11. Math、Random、UUID

```java
Math.max(a, b);  Math.min(a, b);  Math.abs(x);
Math.ceil(3.1);      // 4.0 向上取整
Math.floor(3.9);     // 3.0 向下取整
Math.round(3.5);     // 4
Math.round(-3.5);    // ⚠️ -3，不是 -4！round 的定义是 floor(x + 0.5)

ThreadLocalRandom.current().nextInt(100);    // ⭐ 并发场景用它，不用 Random
UUID.randomUUID().toString();                // 36 位带横杠的唯一串
```

> `Random` 多线程共享时内部靠 CAS 竞争种子，并发下性能很差，**多线程一律用 `ThreadLocalRandom`**。

---

### 12. Objects

```java
Objects.equals(a, b);                       // ⭐ 两边都可能为 null 时的安全比较
Objects.hash(id, name);                     // ⭐ 生成 hashCode（见第十三章）
Objects.requireNonNull(x, "x 不能为空");     // 参数校验，为空抛 NPE 并带上说明
Objects.isNull(x);  Objects.nonNull(x);
```

---

### 13. Optional

见**第十七章第 8 节**。要点回顾：只作为返回值使用，禁止裸用 `get()`，默认值是方法调用时用 `orElseGet` 而非 `orElse`。

---

### 14. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 金额、比例一律用 `BigDecimal` | 数据库用 `DECIMAL`，禁止 `double` |
| `BigDecimal` 用字符串构造 | `new BigDecimal("0.1")` 或 `valueOf()` |
| `BigDecimal` 比较用 `compareTo` | `equals()` 会连小数位数一起比 |
| `divide()` 必须传精度和舍入模式 | 否则除不尽时抛异常 |
| 日期时间用 `java.time` | 禁止新写 `Date` / `Calendar` |
| 禁止 `SimpleDateFormat` 做共享字段 | 线程不安全，改用 `DateTimeFormatter` |
| 格式化年份用小写 `yyyy` | `YYYY` 跨年会算错 |
| `DateTimeFormatter` 提为静态常量 | 不可变、可复用，避免重复创建 |
| 并发取随机数用 `ThreadLocalRandom` | `Random` 有 CAS 竞争 |
| 判空比较用 `Objects.equals()` | 双向防 NPE |

---

### 15. 面试重点

**Q1：为什么金额不能用 `double`？**

`double` 遵循 IEEE 754 用二进制近似表示小数，`0.1 + 0.2` 会得到 `0.30000000000000004`。金额计算要求绝对精确，必须用 `BigDecimal`，且数据库侧用 `DECIMAL` 类型。

**Q2：`new BigDecimal(0.1)` 和 `new BigDecimal("0.1")` 有什么区别？**

前者传入的 `double` 本身就已经不精确，构造出来是一长串近似值；后者按字符串精确解析，得到真正的 `0.1`。**必须用字符串构造或 `BigDecimal.valueOf()`。**

**Q3：`BigDecimal` 比较大小为什么不能用 `equals()`？**

`equals()` 要求值和 scale（小数位数）都相同，所以 `1.0` 和 `1.00` 会返回 `false`。判断值相等必须用 `compareTo(...) == 0`。

**Q4：`SimpleDateFormat` 为什么线程不安全？怎么解决？**

它内部用成员变量 `Calendar` 保存解析中间状态，多线程共享同一个实例时会互相覆盖，解析出错误日期。解决办法：改用 `DateTimeFormatter`（不可变、线程安全），或每次新建实例、或用 `ThreadLocal` 隔离。

**Q5：`java.time` 相比 `Date` 好在哪？**

所有类都是**不可变且线程安全**的；API 语义清晰（`LocalDate`/`LocalDateTime`/`Instant` 职责分明）；月份从 1 开始不再从 0 开始；日期计算和格式化的方法完善且可链式调用。

---

## 十九、IO 流与网络编程

**A · IO 流**

### 1. 流的分类

IO 流的类有几十个，但**只要记住分类逻辑，看名字就知道是什么**，不需要背 API。

| 维度 | 分类 |
| ---- | ---- |
| 数据单位 | **字节流**（8 bit，任意文件）/ **字符流**（16 bit，只能处理文本） |
| 流向 | 输入流 / 输出流 |
| 功能 | **节点流**（直接连数据源）/ **处理流**（包装别的流，如缓冲、转换） |

四大抽象基类：

| | 输入 | 输出 |
| ---- | ---- | ---- |
| **字节流** | `InputStream` | `OutputStream` |
| **字符流** | `Reader` | `Writer` |

命名规律：`XxxInputStream`、`XxxReader` —— **看后缀就知道类别，看前缀就知道功能**。

> 怎么选：**纯文本用字符流，其他（图片、视频、压缩包）用字节流**，不确定就用字节流。

---

### 2. 常用实现类

| 类 | 说明 |
| ---- | ---- |
| `FileInputStream` / `FileOutputStream` | 文件字节流（节点流） |
| ⭐`BufferedInputStream` / `BufferedOutputStream` | 字节缓冲流，默认 8KB |
| `FileReader` / `FileWriter` | 文件字符流，⚠️ **无法指定字符集** |
| ⭐`InputStreamReader` / `OutputStreamWriter` | **转换流**：字节 ↔ 字符，可指定字符集 |
| ⭐`BufferedReader` / `BufferedWriter` | 字符缓冲流，`readLine()` 按行读 |
| `ObjectInputStream` / `ObjectOutputStream` | 对象序列化 |

**为什么必须加缓冲流**：不带缓冲时每读写一个字节就是一次系统调用（用户态 ↔ 内核态切换），加了缓冲后攒够 8KB 才操作一次，**性能相差几十倍**。

---

### 3. 现代写法：Files / Paths（⭐ 现在的标准做法）

实际项目里读写文件**不用手写流**，JDK 7+ 的 `java.nio.file` 一行搞定：

```java
Path path = Paths.get("d:/data/a.txt");

// 读
String content      = Files.readString(path, StandardCharsets.UTF_8);      // JDK 11+
List<String> lines  = Files.readAllLines(path, StandardCharsets.UTF_8);
byte[] bytes        = Files.readAllBytes(path);

// 写
Files.writeString(path, content, StandardCharsets.UTF_8);                  // JDK 11+
Files.write(path, bytes);

// ⭐ 大文件按行流式处理，不一次性加载进内存
try (Stream<String> stream = Files.lines(path, StandardCharsets.UTF_8)) {
    stream.filter(s -> s.contains("ERROR")).forEach(System.out::println);
}

// 其他
Files.exists(path);   Files.size(path);   Files.delete(path);
Files.createDirectories(path);            Files.copy(src, dst);
```

> ⚠️ **`readString()` / `readAllLines()` 会把整个文件读进内存**，处理大文件（日志、导出文件）必须用 `Files.lines()` 流式读取，否则直接 OOM。

---

### 4. 传统写法（要能看懂老代码）

```java
// ⭐ try-with-resources 自动关闭（见第十四章）
try (BufferedReader br = new BufferedReader(
         new InputStreamReader(
             new FileInputStream("a.txt"), StandardCharsets.UTF_8))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
}
```

这个**三层包装**是 IO 的经典结构，从内到外：

```text
FileInputStream      节点流：连上文件，拿到字节
    ↓
InputStreamReader    转换流：字节 → 字符，指定字符集
    ↓
BufferedReader       处理流：加缓冲，支持 readLine()
```

复制文件：

```java
try (InputStream in = new FileInputStream("a"); OutputStream out = new FileOutputStream("b")) {
    in.transferTo(out);        // JDK 9+，一行搞定，内部自带缓冲
}
```

---

### 5. IO 的坑

**① 字符集必须显式指定 —— 跨平台乱码的头号原因**

```java
new FileReader("a.txt");        // ✘ 使用平台默认编码：Windows 是 GBK，Linux 是 UTF-8
                                //   同一份代码在两个环境跑出来结果不一样

new InputStreamReader(new FileInputStream("a.txt"), StandardCharsets.UTF_8);   // ✔
```

> 所有**字节 ↔ 字符**转换的地方都要显式传字符集：文件流、`String.getBytes()`、`new String(bytes)`、`URLEncoder`。

**② 流必须关闭** —— 用 try-with-resources，不手写 `finally { close() }`。

**③ 大文件不要一次性读进内存** —— 用 `Files.lines()` 或 `BufferedReader.readLine()`。

**④ 不加缓冲的单字节循环读写** —— 性能差几十倍。

---

**B · 序列化**

### 6. Java 原生序列化

```java
public class User implements Serializable {

    private static final long serialVersionUID = 1L;    // ⭐ 必须手动指定

    private String name;
    private transient String password;                  // transient：不参与序列化
}
```

| 要点 | 说明 |
| ---- | ---- |
| `Serializable` | 标记接口，没有任何方法，只是给 JVM 一个标记 |
| ⭐`serialVersionUID` | **必须手动写死** |
| `transient` | 修饰的字段不参与序列化（密码、临时状态、大对象） |
| `static` 字段 | 也不参与 —— 它属于类，不属于对象 |

**为什么 `serialVersionUID` 必须手写**：不写的话编译器会根据类结构自动生成，**加一个字段这个值就变了**，导致旧数据反序列化时抛 `InvalidClassException`。写死为 `1L` 后，类结构变化也能兼容反序列化（新增字段取默认值）。

> **实际项目基本不用 Java 原生序列化**：跨语言不通、字节体积大、还存在反序列化 RCE 安全漏洞。现在统一用 **JSON**（Jackson / Fastjson2），追求性能用 Protobuf。
>
> 但 Redis 存对象、Dubbo RPC 传参时仍会用到，所以实体类习惯上还是会实现 `Serializable`。

---

**C · 网络编程与 IO 模型**

### 7. 三种 IO 模型（面试重点）

| 模型 | 全称 | 工作方式 | 代表 |
| ---- | ---- | -------- | ---- |
| **BIO** | 同步阻塞 | **一个连接一个线程**，线程阻塞等数据 | `ServerSocket`，早期 Tomcat |
| ⭐**NIO** | 同步非阻塞（多路复用） | **一个线程用 Selector 管理多个连接** | `java.nio`，Netty、Redis、Nginx |
| **AIO** | 异步非阻塞 | 内核处理完后**回调**通知 | Linux 支持不佳，几乎不用 |

**BIO 的致命问题**：1 万个连接就要 1 万个线程，线程创建和上下文切换的开销直接压垮系统 —— 这就是著名的 **C10K 问题**。

#### NIO 的三大核心组件

| 组件 | 作用 |
| ---- | ---- |
| `Channel`（通道） | 双向读写，替代单向的 Stream |
| `Buffer`（缓冲区） | 数据容器，读写都要经过它 |
| ⭐`Selector`（选择器） | **一个线程监听多个 Channel**，哪个就绪就处理哪个 |

> `Selector` 底层就是操作系统的多路复用机制 —— **Linux 上是 epoll**，这是 NIO 能扛住海量连接的根源。

#### 为什么生产环境不直接写 NIO

原生 NIO 的 API 非常复杂，还存在 epoll 空轮询导致 CPU 100% 的 bug，粘包拆包也要自己处理。**实际都用 Netty** —— Dubbo、RocketMQ、Elasticsearch、Spring WebFlux 全部基于它。

---

### 8. Socket（了解即可）

```java
// 服务端
try (ServerSocket server = new ServerSocket(8080)) {
    Socket socket = server.accept();          // 阻塞，直到有客户端连上来
}

// 客户端
try (Socket socket = new Socket("127.0.0.1", 8080)) { }
```

> 业务开发几乎不会手写 Socket：HTTP 调用用 `RestTemplate` / `OkHttp` / `WebClient`，服务间调用用 Dubbo / gRPC。这部分**只需理解模型，不需要会手撕**。

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 读写文件优先用 `Files` / `Paths` | 不手写三层流包装 |
| 一律显式指定字符集 | `StandardCharsets.UTF_8`，禁止依赖平台默认 |
| 禁用 `FileReader` / `FileWriter` | 无法指定字符集 |
| 流的关闭用 try-with-resources | 不手写 `finally { close() }` |
| 手写流必须加缓冲 | 否则性能差几十倍 |
| 大文件用 `Files.lines()` 流式处理 | 避免一次性加载导致 OOM |
| 实体类显式声明 `serialVersionUID` | 避免类结构变化后反序列化失败 |
| 敏感字段加 `transient` | 密码、密钥不参与序列化 |
| 数据交换用 JSON | 不用 Java 原生序列化 |

---

### 10. 面试重点

**Q1：字节流和字符流的区别？怎么选？**

字节流以 8 bit 为单位，能处理任意类型文件；字符流以 16 bit 字符为单位，底层仍是字节流 + 字符集解码，只能处理文本。纯文本用字符流，图片视频等二进制文件用字节流。

**Q2：为什么要用缓冲流？**

不带缓冲时每次读写都触发一次系统调用，涉及用户态和内核态切换，开销极大。缓冲流在内存中攒够一批（默认 8KB）再一次性操作，性能可以相差几十倍。

**Q3：`serialVersionUID` 有什么用？不写会怎样？**

它是序列化的版本标识。不显式声明时编译器会根据类结构自动生成，**只要类加一个字段这个值就会变**，导致旧数据反序列化抛 `InvalidClassException`。手动写死后，类结构演进也能兼容。

**Q4：`transient` 的作用？**

标记该字段不参与序列化，反序列化后取类型默认值。用于密码、密钥等敏感信息，以及不需要持久化的临时状态。`static` 字段同样不参与序列化。

**Q5：BIO、NIO、AIO 的区别？**

BIO 同步阻塞，一个连接一个线程，连接数一多线程开销就压垮系统；NIO 同步非阻塞，用 `Selector` 让一个线程管理多个连接，靠操作系统的多路复用（Linux 上是 epoll）实现；AIO 异步非阻塞，由内核完成后回调，但 Linux 支持不佳，实际很少使用。

**Q6：NIO 的三大核心组件是什么？**

`Channel`（双向通道）、`Buffer`（数据缓冲区）、`Selector`（选择器，一个线程监听多个 Channel 的就绪事件）。

**Q7：为什么用 Netty 而不是原生 NIO？**

原生 NIO API 复杂、存在 epoll 空轮询导致 CPU 100% 的 bug、粘包拆包和心跳重连都要自己实现。Netty 封装了这些问题并做了大量性能优化，是事实标准 —— Dubbo、RocketMQ、Elasticsearch 都基于它。

---

## 二十、多线程与并发（JUC）

### 1. 线程基础

#### 创建线程的方式

```java
// ① 实现 Runnable —— 无返回值
new Thread(() -> System.out.println("run")).start();

// ② 实现 Callable —— 有返回值、能抛异常
Callable<String> task = () -> "result";
FutureTask<String> ft = new FutureTask<>(task);
new Thread(ft).start();
String r = ft.get();          // 阻塞等待结果

// ③ ⭐ 线程池 —— 实际开发中唯一正确的方式
executor.submit(() -> doSomething());
```

> 继承 `Thread` 的方式因为占用了单继承名额，实际不用。**`start()` 才会开新线程，直接调 `run()` 只是普通方法调用**，仍在当前线程里跑。

#### 线程的六种状态

```text
NEW → RUNNABLE ⇄ BLOCKED / WAITING / TIMED_WAITING → TERMINATED
```

| 状态 | 触发条件 |
| ---- | -------- |
| `NEW` | 创建了但还没 `start()` |
| `RUNNABLE` | 运行中或等待 CPU 调度 |
| `BLOCKED` | 等待获取 `synchronized` 锁 |
| `WAITING` | `wait()` / `join()` / `LockSupport.park()` |
| `TIMED_WAITING` | `sleep(n)` / `wait(n)` |
| `TERMINATED` | 执行结束 |

#### sleep 与 wait 的区别（必问）

| 对比项 | `sleep()` | `wait()` |
| ------ | --------- | -------- |
| 所属 | `Thread` 的静态方法 | `Object` 的方法 |
| **是否释放锁** | ⭐**不释放** | ⭐**释放** |
| 唤醒方式 | 时间到自动醒 | `notify()`/`notifyAll()` 或超时 |
| 使用位置 | 任意位置 | **必须在 `synchronized` 块内** |

#### 线程中断（interrupt）

Java **没有**强行杀死线程的手段（`stop()` 早已废弃，它会直接释放锁导致数据不一致）。取消一个线程只有一种正确方式：**发中断信号，由线程自己决定何时退出**。

```java
thread.interrupt();          // 只是把中断标志位置为 true，不会真的中断什么

// 线程内部自己响应
while (!Thread.currentThread().isInterrupted()) {
    // 干活
}
```

| 方法 | 作用 | 是否清除标志位 |
| ---- | ---- | ---- |
| `interrupt()` | 设置中断标志 | — |
| `isInterrupted()` | 查询标志（实例方法） | ✘ 不清除 |
| `Thread.interrupted()` | 查询标志（静态方法） | ⚠️ **会清除** |

**关键规则**：`sleep()`、`wait()`、`join()` 这类阻塞方法收到中断会**抛出 `InterruptedException`，同时清除中断标志**。所以：

```java
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    // ✘ 最常见的错误：吞掉异常。标志位已被清除，上层再也不知道被中断过
}

try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();   // ✔ 恢复中断标志
    return;                               // 或直接向上抛出
}
```

> 线程池的 `shutdownNow()` 正是靠给每个工作线程发中断来停止任务的 —— **任务里吞掉 `InterruptedException`，线程池就永远优雅关不掉**，应用停机时会一直卡住。

---

### 2. 线程安全的三个核心问题

| 问题 | 含义 | 解决 |
| ---- | ---- | ---- |
| **原子性** | `i++` 是读-改-写三步，会丢更新 | `synchronized`、原子类 |
| **可见性** | 一个线程改了，另一个线程看不到 | `volatile`、`synchronized` |
| **有序性** | 编译器和 CPU 会指令重排 | `volatile`、`synchronized` |

**JMM（Java 内存模型）**：共享变量存在**主内存**，每个线程有自己的**工作内存**（对应 CPU 缓存）。线程操作的是变量副本，不刷回主内存，其他线程就看不见 —— 这就是可见性问题的根源。

#### happens-before（面试必问）

只靠"主内存 / 工作内存"这个模型，没法判断一段并发代码到底对不对。JMM 真正对外做出的承诺是 **happens-before**：如果操作 A happens-before 操作 B，那么 **A 的所有结果对 B 一定可见**（就执行结果而言，A 排在 B 前面）。

八条规则，记住加粗的五条就够用：

| 规则 | 内容 |
| ---- | ---- |
| **程序顺序** | 同一个线程内，写在前面的操作 happens-before 写在后面的 |
| ⭐**锁规则** | 对一把锁的 `unlock` happens-before 后续对这把锁的 `lock` |
| ⭐**volatile 规则** | 对 `volatile` 变量的写 happens-before 后续对它的读 |
| ⭐**传递性** | A hb B、B hb C ⇒ A hb C |
| ⭐**线程启动** | `t.start()` happens-before 线程 t 内的所有操作 |
| 线程终止 | 线程 t 内的所有操作 happens-before 其他线程从 `t.join()` 返回 |
| 中断 | `t.interrupt()` happens-before 线程 t 检测到中断 |
| 对象终结 | 构造器执行完 happens-before `finalize()` |

有了**传递性**，`volatile` 的威力才完整：**写 `volatile` 变量之前的所有普通写操作，对读到该 `volatile` 变量的线程也全部可见**。DCL 单例、状态标志位能生效，靠的都是这一条。

```java
int data = 0;
volatile boolean ready = false;

// 线程 A                          // 线程 B
data = 42;                        if (ready) {
ready = true;   // volatile 写         print(data);   // ✔ 一定是 42，不会是 0
                                  }
```

> **as-if-serial**：单线程内无论怎么重排，最终结果必须和顺序执行一致。所以指令重排在单线程下永远不会出错，**只有多线程共享变量时才会暴露**。

---

### 3. synchronized

```java
synchronized void f() { }             // 锁当前实例 this
static synchronized void g() { }      // 锁 Class 对象（全类共享一把锁）
synchronized (lock) { }               // ⭐ 锁指定对象，粒度可控，推荐
```

**锁升级**（JDK 6 起的优化，只能升不能降）：

```text
无锁 → 偏向锁 → 轻量级锁（CAS 自旋） → 重量级锁（操作系统互斥量，线程挂起）
```

底层靠对象头里的 **Mark Word** 记录锁状态，重量级锁通过 `monitorenter` / `monitorexit` 指令操作 monitor。

> 偏向锁在 **JDK 15 起被废弃、JDK 18 已彻底移除**（维护成本高、收益有限，且在 Lambda / 并发集合大量使用的现代代码里经常需要昂贵的撤销）。面试时提一句是加分项。

`synchronized` 是**可重入锁**：同一线程可以重复获得同一把锁，避免自己把自己锁死。

---

### 4. volatile

两个作用：

1. **保证可见性** —— 写操作立即刷回主内存，读操作强制从主内存读
2. **禁止指令重排** —— 通过插入内存屏障实现

**但它不保证原子性**：

```java
volatile int i = 0;
i++;              // ✘ 依然线程不安全，读-改-写三步中间会被打断
```

两个典型用途：

```java
// ① 状态标志位 —— volatile 最经典的正确用法
private volatile boolean running = true;
while (running) { ... }             // 别的线程改成 false 后能立刻看到

// ② 双重检查锁单例 —— volatile 防止指令重排
private static volatile Singleton instance;
```

> 单例这里如果不加 `volatile`：`new` 对象分为「分配内存 → 初始化 → 赋值引用」三步，重排后可能先赋值再初始化，**别的线程会拿到一个半初始化的对象**。

---

### 5. synchronized vs ReentrantLock

| 对比项 | `synchronized` | `ReentrantLock` |
| ------ | -------------- | --------------- |
| 层面 | JVM 关键字 | JDK 类（基于 AQS） |
| 释放锁 | 自动释放 | ⚠️ **必须手动 `unlock()`** |
| 可中断 | ✘ | ✔ `lockInterruptibly()` |
| 超时获取 | ✘ | ✔ `tryLock(时间)` |
| 公平锁 | ✘ 只能非公平 | ✔ 可选 |
| 条件变量 | 一个（`wait`/`notify`） | 多个 `Condition` |

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // 业务
} finally {
    lock.unlock();        // ⭐ 必须写在 finally，否则异常时锁永远不释放
}
```

> **没有特殊需求就用 `synchronized`** —— JDK 6 优化后性能已经接近，而且不会忘记释放。需要超时、可中断、公平锁时才用 `ReentrantLock`。
>
> 读多写少的场景用 `ReentrantReadWriteLock`，读锁之间不互斥。

#### AQS：整个 JUC 的底座（面试必问）

`AbstractQueuedSynchronizer`（AQS）是 `ReentrantLock`、`CountDownLatch`、`Semaphore`、`ReentrantReadWriteLock`、线程池的 `Worker` **共同的父类**，它把"管理线程排队"这件事抽象成了一套模板。

两个核心结构：

| 结构 | 作用 |
| ---- | ---- |
| ⭐`volatile int state` | 同步状态。`ReentrantLock` 用它记**重入次数**（0 = 无人持有）；`Semaphore` 用它记**剩余许可数**；`CountDownLatch` 用它记**剩余计数** |
| ⭐CLH 变体：**双向链表队列** | 抢不到锁的线程被包装成 `Node` 入队，用 `LockSupport.park()` 挂起；持有者释放锁时 `unpark()` 唤醒队首 |

以 `ReentrantLock.lock()` 为例的流程：

1. CAS 尝试把 `state` 从 0 改成 1，成功就直接持有锁（**非公平锁就是在这一步插队**）
2. 失败则判断持有者是不是自己（是 → `state + 1`，这就是**可重入**的实现）
3. 仍然失败 → 包装成 `Node` 入队 → `park()` 挂起
4. 持有者 `unlock()` 把 `state` 减到 0 → `unpark()` 唤醒队首线程重新竞争

**公平 vs 非公平**：公平锁进来先看队列里有没有人在等，有就排到队尾；非公平锁上来就先 CAS 抢一次。非公平**吞吐量更高**（少一次线程挂起/唤醒的切换），所以是**默认**，代价是可能出现线程饥饿。

> AQS 用的是**模板方法模式**（第二十三章）：排队、挂起、唤醒全部由 AQS 实现，子类只需要重写 `tryAcquire()` / `tryRelease()` 定义"什么算拿到锁"。被问到"JDK 里哪里用了设计模式"，这是最好的答案之一。

---

### 6. CAS 与原子类

**CAS（Compare And Swap）** 是 CPU 级别的原子指令：比较内存值和预期值，相同才更新。它是**无锁**的，靠自旋重试实现并发安全。

```java
AtomicInteger count = new AtomicInteger();
count.incrementAndGet();      // ⭐ 原子自增，替代 synchronized
```

**CAS 的三个问题**：

| 问题 | 说明 | 解决 |
| ---- | ---- | ---- |
| **ABA 问题** | 值从 A 改成 B 又改回 A，CAS 察觉不到 | `AtomicStampedReference` 加版本号 |
| 自旋开销 | 竞争激烈时一直失败一直空转，浪费 CPU | 限制自旋次数或改用锁 |
| 只能保证单个变量 | 多个变量的复合操作无法原子 | 用锁，或封装成一个对象配 `AtomicReference` |

> 高并发计数用 **`LongAdder`** 比 `AtomicLong` 快得多：它把热点分散到多个 Cell 分段累加，取值时求和，大幅减少 CAS 冲突。

---

### 7. 线程池（实际开发核心 + 面试重点）

**为什么用线程池**：复用线程避免频繁创建销毁的开销、控制最大并发数防止资源耗尽、便于统一管理和监控。

#### 七个核心参数

```java
new ThreadPoolExecutor(
    corePoolSize,        // 核心线程数
    maximumPoolSize,     // 最大线程数
    keepAliveTime, unit, // 非核心线程的空闲存活时间
    workQueue,           // ⭐ 任务队列
    threadFactory,       // 线程工厂 —— 用来给线程起名，排查问题必备
    handler              // ⭐ 拒绝策略
);
```

#### 执行流程（必背）

1. 核心线程数未满 → **创建核心线程**执行
2. 核心线程满了 → **任务进队列排队**
3. 队列也满了 → **创建非核心线程**（直到 `maximumPoolSize`）
4. 线程数达到最大且队列满 → **执行拒绝策略**

> ⚠️ 注意顺序：**先排队，队列满了才扩线程**。很多人误以为是先把线程扩到 max 再排队。

#### 四种拒绝策略

| 策略 | 行为 |
| ---- | ---- |
| ⭐`AbortPolicy` | 默认，直接抛 `RejectedExecutionException` |
| `CallerRunsPolicy` | 让**提交任务的线程**自己执行，起到降速反压的作用 |
| `DiscardPolicy` | 静默丢弃，不报错（危险） |
| `DiscardOldestPolicy` | 丢掉队列里最老的任务 |

#### 为什么禁止用 Executors 创建线程池

这是阿里规约的强制条款：

| 方法 | 隐患 |
| ---- | ---- |
| `newFixedThreadPool` / `newSingleThreadExecutor` | 队列是**无界**的 `LinkedBlockingQueue`，任务无限堆积 → **OOM** |
| `newScheduledThreadPool` | 队列是**无界**的 `DelayedWorkQueue`，同样是任务堆积 → **OOM** |
| `newCachedThreadPool` | 最大线程数是 `Integer.MAX_VALUE` + `SynchronousQueue`，来一个任务开一个线程 → **OOM** |

**必须手动 `new ThreadPoolExecutor(...)` 并指定有界队列。**

> 细节修正：阿里规约把 `newScheduledThreadPool` 和 `newCachedThreadPool` 一起归入"允许创建的线程数量为 `Integer.MAX_VALUE`"。严格说 `newScheduledThreadPool` 的核心线程数是固定的，**因为队列无界，线程数根本扩不上去**，它真正的隐患和 `newFixedThreadPool` 一样是任务堆积。结论不变，但答面试时说清楚更显功底。

#### 线程数怎么定

- **CPU 密集型**：核数 + 1
- **IO 密集型**：核数 × 2 起步

> 公式只是起点，**实际必须靠压测和监控来调**。别把公式当答案。

---

### 8. 常用并发工具

| 工具 | 用途 |
| ---- | ---- |
| `CountDownLatch` | 主线程等待 N 个任务完成（**一次性**） |
| `CyclicBarrier` | N 个线程互相等待，全到齐再一起走（**可重复**） |
| `Semaphore` | 限流，控制同时访问的线程数 |
| ⭐`CompletableFuture` | 异步编排，替代 `Future` |

```java
// 并行调用多个接口，总耗时 = 最慢的那个
CompletableFuture<User>  f1 = CompletableFuture.supplyAsync(() -> getUser(id),  pool);
CompletableFuture<Order> f2 = CompletableFuture.supplyAsync(() -> getOrder(id), pool);

CompletableFuture.allOf(f1, f2).join();
User user = f1.join();
```

**常用编排方法**：

```java
// 串行依赖：先查用户，再用用户 id 查订单
CompletableFuture<List<Order>> f = CompletableFuture
        .supplyAsync(() -> getUser(id), pool)
        .thenApply(User::getId)                          // 同步转换，返回普通值
        .thenCompose(uid -> queryOrders(uid, pool));     // ⭐ 返回值本身也是 CF 时用它，否则会套成两层

// 并行合并：两个互不依赖的调用，结果合起来
CompletableFuture<Result> r = f1.thenCombine(f2, (user, order) -> merge(user, order));

// ⭐ 异常兜底 + 超时保护
f.exceptionally(e -> { log.error("查询订单失败", e); return Collections.emptyList(); })
 .orTimeout(3, TimeUnit.SECONDS)                         // JDK 9+
 .join();
```

| 方法 | 用途 |
| ---- | ---- |
| `thenApply` | 转换结果（返回普通值） |
| ⭐`thenCompose` | 串行依赖，回调返回值也是 `CompletableFuture` |
| ⭐`thenCombine` | 合并两个并行任务的结果 |
| `thenAccept` / `thenRun` | 只消费结果 / 什么都不接收 |
| ⭐`exceptionally` | 出异常时返回一个兜底值 |
| `handle` / `whenComplete` | 无论成功失败都执行，能同时拿到结果和异常 |
| `allOf` / `anyOf` | 等全部完成 / 等任意一个完成 |

三个必须注意的点：

- ⚠️ `supplyAsync()` **不传线程池时会用 `ForkJoinPool.commonPool()`**（JVM 全局共享），和 `parallelStream()` 是同一个坑（见第十七章）。**每个带 `Async` 的方法都要传自己的线程池。**
- ⚠️ **不写 `exceptionally` / `handle`，异常会被封在 `CompletableFuture` 里**，不 `join()` 就永远看不到，接口直接返回空结果 —— 和线程池 `submit()` 吞异常是同一类问题。
- 带 `Async` 后缀的方法才会换线程执行，不带的可能直接跑在上一个任务的线程里。

#### 阻塞队列（BlockingQueue）

线程池的 `workQueue` 就是它，同时它也是**生产者-消费者模型**的标准实现 —— 队列满时生产者阻塞、队列空时消费者阻塞，不用自己写 `wait()` / `notify()`。

| 实现类 | 特点 | 用在哪 |
| ---- | ---- | ---- |
| ⭐`ArrayBlockingQueue` | **有界**，数组实现，一把锁 | ⭐ 线程池首选，容量必须显式指定 |
| `LinkedBlockingQueue` | 链表实现，**默认无界**（`Integer.MAX_VALUE`），读写各一把锁吞吐更高 | 用它**必须传容量**，否则就是上面那个 OOM 陷阱 |
| `SynchronousQueue` | **不存元素**，一次 `put` 必须等一次 `take` | `newCachedThreadPool` 用的就是它 |
| `PriorityBlockingQueue` | 按优先级出队，**无界** | 任务有优先级时 |
| `DelayQueue` | 元素到期才能取出 | 延时任务、订单超时关闭 |

四组 API 行为完全不同，选错就是 bug：

| 场景 | 抛异常 | 返回特殊值 | ⭐阻塞 | 超时 |
| ---- | ---- | ---- | ---- | ---- |
| 入队 | `add` | `offer` | `put` | `offer(e, time, unit)` |
| 出队 | `remove` | `poll` | `take` | `poll(time, unit)` |

```java
BlockingQueue<Order> queue = new ArrayBlockingQueue<>(1000);

queue.put(order);            // 生产者：队列满则阻塞，天然起到限流 / 反压的作用
Order o = queue.take();      // 消费者：队列空则阻塞，不用空转轮询
```

---

### 9. ThreadLocal

给每个线程一份独立的变量副本，常用于保存**用户上下文**、事务连接、链路追踪 ID。

```java
private static final ThreadLocal<User> CONTEXT = new ThreadLocal<>();

try {
    CONTEXT.set(user);
    // 业务代码里任意位置都能 CONTEXT.get()
} finally {
    CONTEXT.remove();        // ⭐ 必须！
}
```

**为什么必须 `remove()`**：`ThreadLocalMap` 的 key 是**弱引用**、value 是**强引用**。线程池中的线程会被无限复用、永不销毁，value 就一直无法回收 —— 既造成**内存泄漏**，还会让下一个请求**读到上一个用户的数据**（串数据，是安全事故）。

**线程池场景下的上下文传递**：

| 类 | 行为 |
| ---- | ---- |
| `ThreadLocal` | 只在当前线程内可见，子线程 / 线程池线程都拿不到 |
| `InheritableThreadLocal` | **子线程创建的那一刻**从父线程拷贝一份 —— 但线程池的线程是复用的，**创建时拷到的值会一直沿用下去，等于串数据** |
| ⭐`TransmittableThreadLocal`（阿里 TTL） | **任务提交时**捕获、**执行时**回放，是"线程池 + 上下文传递"的正确解法 |

> 结论：**在用线程池的项目里，`InheritableThreadLocal` 是错的**。要么把上下文（traceId、用户信息、租户）当参数显式传进任务，要么引入 TTL 并用它包装线程池（`TtlExecutors.getTtlExecutorService(pool)`）。

---

### 10. 虚拟线程（JDK 21）

JDK 21 转正的重量级特性，也是目前并发部分最容易被问到的新题。

**解决什么问题**：传统 `Thread` 是**平台线程**，1:1 映射到操作系统线程，一个约占 1MB 栈内存，几千个就到顶了。所以"一个连接一个线程"的写法撑不到高并发，再往上只能转异步 / 响应式（Netty、WebFlux），代价是代码彻底变形、堆栈和调试全废。

**虚拟线程**由 JVM 调度，栈保存在堆上，创建成本极低，**可以轻松开几百万个**：

```java
// 创建：和普通线程一样简单
Thread.startVirtualThread(() -> doSomething());

// ⭐ 实际用法：每个任务一个虚拟线程，不池化
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (Order o : orders) {
        executor.submit(() -> handle(o));      // 十万个任务也没问题
    }
}                                              // close() 会等所有任务结束
```

| 对比项 | 平台线程 | ⭐虚拟线程 |
| ------ | -------- | -------- |
| 映射关系 | 1:1 映射 OS 线程 | 由 JVM 调度，多路复用到少量**载体线程**（carrier thread） |
| 数量级 | 几千 | **几百万** |
| 阻塞代价 | 占住一个 OS 线程 | 阻塞时**自动卸载**，把载体线程让给别的任务 |
| 池化 | ⭐必须池化 | ⚠️ **禁止池化**，用完即弃 |
| 适用场景 | CPU 密集型 | ⭐**IO 密集型**（查库、调接口、读文件） |

四个必须知道的点：

- **不要池化**。线程池存在的意义是复用昂贵资源，虚拟线程很便宜，池化反而白白引入排队限制。要限流请用 `Semaphore`。
- ⚠️ **`synchronized` 会导致 pinning（钉住）**：虚拟线程在 `synchronized` 块里阻塞时无法卸载，会连载体线程一起占住，并发度瞬间退化回平台线程。JDK 21 下**涉及阻塞的临界区应改用 `ReentrantLock`**（该限制在 JDK 24 已修复）。
- **`ThreadLocal` 仍可用但要克制**：虚拟线程数量巨大，每个都挂一份副本会吃掉大量内存。
- **CPU 密集型任务没有收益**：它省的是"阻塞等待"的开销，不是计算的开销。

> 一句话回答面试：**虚拟线程让"同步阻塞"这种简单写法重新获得了异步编程的吞吐量**。Spring Boot 3.2+ 只需一行 `spring.threads.virtual.enabled=true` 即可开启。

---

### 11. 并发的坑

| # | 坑 | 后果 |
| - | ---- | ---- |
| ① | 用 `Executors` 创建线程池 | 无界队列或无限线程 → OOM |
| ② | `ThreadLocal` 用完不 `remove()` | 内存泄漏 + 线程复用串数据 |
| ③ | 线程池不设置线程名 | 出问题看堆栈全是 `pool-1-thread-3`，无从排查 |
| ④ | 在 `@Transactional` 方法里开异步线程 | **事务绑定当前线程**，新线程不在事务内，不会回滚 |
| ⑤ | 双重检查锁不加 `volatile` | 指令重排导致拿到半初始化对象 |
| ⑥ | `synchronized` 锁错对象 | 锁 `String`/`Integer` 等有缓存的对象会误锁；锁可变对象则锁根本不生效 |
| ⑦ | 多个锁的获取顺序不一致 | **死锁** |
| ⑧ | 异步任务不处理异常 | `submit()` 提交的任务异常会被吞进 `Future`，不 `get()` 就永远看不到 |

> ④ 和 ⑧ 是最容易在生产环境静默出错的两个 —— 都不会报错，只是结果不对。

---

### 12. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 线程池手动 `new ThreadPoolExecutor` | 禁止用 `Executors`，必须用有界队列 |
| 线程池必须命名 | 用 `ThreadFactory` 起业务名，方便排查 |
| 不同业务用不同线程池 | 隔离开，避免互相拖垮 |
| `ThreadLocal` 用完在 `finally` 里 `remove()` | 防泄漏、防串数据 |
| `CompletableFuture` 必须传自定义线程池 | 否则用全局的 `commonPool` |
| `lock()` 后 `unlock()` 写在 `finally` | 否则异常时锁不释放 |
| 优先用 `synchronized` | 简单、自动释放，性能已经足够 |
| 计数用 `AtomicInteger` / `LongAdder` | 不用 `volatile int` |
| 并发集合用 `ConcurrentHashMap` | 见第十六章 |
| 加锁顺序全局统一 | 避免死锁 |
| 不吞掉 `InterruptedException` | 要么向上抛，要么 `Thread.currentThread().interrupt()` 恢复标志 |
| `CompletableFuture` 必须接 `exceptionally` / `handle` | 否则异常被静默吞掉 |
| 线程池 + 上下文传递用 TTL | 不要用 `InheritableThreadLocal` |

---

### 13. 面试重点

**Q1：`sleep()` 和 `wait()` 的区别？**

`sleep()` 是 `Thread` 的静态方法，**不释放锁**，时间到自动唤醒，可在任意位置调用；`wait()` 是 `Object` 的方法，**会释放锁**，需要 `notify()`/`notifyAll()` 唤醒，且**必须在 `synchronized` 块内**调用。

**Q2：`volatile` 能保证线程安全吗？**

不能。它只保证**可见性**和**禁止指令重排**，不保证**原子性** —— `volatile int i; i++` 依然会丢更新。适合状态标志位这类"一写多读"的场景，计数请用 `AtomicInteger`。

**Q3：`synchronized` 的锁升级过程？**

无锁 → 偏向锁 → 轻量级锁（CAS 自旋）→ 重量级锁（操作系统互斥量，线程挂起），**只能升级不能降级**。锁状态记录在对象头的 Mark Word 中。JDK 15 起偏向锁已被废弃。

**Q4：`synchronized` 和 `ReentrantLock` 怎么选？**

`synchronized` 是 JVM 关键字、自动释放锁、写法简单，JDK 6 之后性能已经很好，**默认选它**；`ReentrantLock` 基于 AQS，支持超时获取、可中断、公平锁和多个 `Condition`，但必须在 `finally` 里手动 `unlock()`。有这些特殊需求才用。

**Q5：线程池的执行流程是什么？**

核心线程未满就创建核心线程执行 → 核心满了任务**先进队列排队** → 队列满了才创建非核心线程（到 `maximumPoolSize`）→ 线程和队列都满了执行拒绝策略。**关键是"先排队，后扩容"**。

**Q6：为什么阿里规约禁止用 `Executors` 创建线程池？**

`newFixedThreadPool`、`newSingleThreadExecutor` 用的是**无界的 `LinkedBlockingQueue`**，`newScheduledThreadPool` 用的是**无界的 `DelayedWorkQueue`**，任务无限堆积会 OOM；`newCachedThreadPool` 的最大线程数是 `Integer.MAX_VALUE` 且配 `SynchronousQueue`，来一个任务开一个线程，同样会 OOM。必须手动 `new ThreadPoolExecutor` 并使用有界队列。

**Q7：CAS 是什么？有什么问题？**

Compare And Swap，比较内存值与预期值，相同才更新，是 CPU 原子指令，属于无锁并发。三个问题：**ABA 问题**（用 `AtomicStampedReference` 加版本号解决）、竞争激烈时**自旋开销大**、**只能保证单个变量**的原子性。

**Q8：`ThreadLocal` 为什么会内存泄漏？怎么避免？**

`ThreadLocalMap` 的 key 是弱引用、**value 是强引用**。线程池中的线程被长期复用不会销毁，value 无法被回收，既泄漏内存又会导致下一个请求读到上个用户的数据。**必须在 `finally` 中调用 `remove()`**。

**Q9：什么是死锁？怎么避免？**

多个线程互相持有对方需要的锁并等待对方释放。四个必要条件：互斥、持有并等待、不可剥夺、循环等待。最实用的避免方法是**规定全局统一的加锁顺序**，其次是用 `tryLock(超时)` 避免无限等待。

**Q10：`@Transactional` 方法里开异步线程，事务还有效吗？**

无效。Spring 事务通过 `ThreadLocal` 绑定在**当前线程**上，新开的线程拿不到事务上下文，它的数据库操作不在原事务内，主线程回滚也不会影响它。需要异步内部事务就在异步方法上单独加 `@Transactional`。

**Q11：什么是 happens-before？**

JMM 对外的可见性承诺：若 A happens-before B，则 A 的所有操作结果对 B 一定可见。常用的几条是程序顺序规则、锁规则（`unlock` 先于后续 `lock`）、`volatile` 规则（写先于后续读）、线程启动规则（`start()` 先于线程内所有操作）以及**传递性**。正因为有传递性，写 `volatile` 之前的普通写对读到它的线程也可见 —— 这是 DCL 单例和状态标志位成立的理论依据。补充一条 **as-if-serial**：单线程内重排后的结果必须与顺序执行一致，所以重排只在多线程共享变量时才会出问题。

**Q12：AQS 是什么？`ReentrantLock` 怎么基于它实现？**

AQS 是 JUC 的底座，`ReentrantLock`、`CountDownLatch`、`Semaphore`、`ReentrantReadWriteLock` 都基于它。核心是一个 `volatile int state` 加一个 CLH 变体的双向等待队列：`ReentrantLock` 用 `state` 记重入次数，CAS 把它从 0 改成 1 就算拿到锁，失败的线程包装成 Node 入队并 `LockSupport.park()` 挂起，释放时唤醒队首。公平锁先查队列再抢，非公平锁直接 CAS 插队、吞吐更高所以是默认。AQS 本身是**模板方法模式**，子类只重写 `tryAcquire` / `tryRelease`。

**Q13：`BlockingQueue` 有哪些实现？线程池该选哪个？**

`ArrayBlockingQueue`（有界数组）、`LinkedBlockingQueue`（链表，**默认无界**）、`SynchronousQueue`（不存元素，直接交付）、`PriorityBlockingQueue`（优先级，无界）、`DelayQueue`（延时）。线程池应选 `ArrayBlockingQueue` 或**显式指定容量**的 `LinkedBlockingQueue`，绝不能用无界队列。另外要区分 `add`/`offer`/`put`/`offer(超时)` 四组 API：只有 `put`、`take` 是阻塞的。

**Q14：怎么优雅地停止一个线程？**

不能用 `stop()`（已废弃，会直接释放锁导致数据不一致）。正确方式是 `interrupt()` 发中断信号，线程内部通过 `isInterrupted()` 轮询或响应 `InterruptedException` 自行退出。注意阻塞方法抛出 `InterruptedException` 时会**清除中断标志**，所以 catch 里必须 `Thread.currentThread().interrupt()` 恢复标志或直接向上抛 —— 吞掉它会导致线程池 `shutdownNow()` 永远关不掉。

**Q15：虚拟线程解决了什么问题？用它要注意什么？**

平台线程 1:1 映射 OS 线程，单个约占 1MB，几千个就到极限；虚拟线程由 JVM 调度、栈在堆上，可以创建几百万个，阻塞时自动从载体线程卸载，因此**用同步阻塞的简单写法就能拿到异步的吞吐量**，适合 IO 密集型。注意三点：**不要池化**（用 `newVirtualThreadPerTaskExecutor`，限流用 `Semaphore`）、**JDK 21 下 `synchronized` 内阻塞会 pinning 住载体线程**（改用 `ReentrantLock`）、CPU 密集型没有收益。

---

## 二十一、反射与注解

**A · 反射**

### 1. 反射是什么

**在运行期动态获取类的信息、创建对象、调用方法、访问字段**，不需要在编译期知道类名。

```java
User u = new User();                                  // 正常：编译期就确定了类型
Object o = Class.forName(className).getDeclaredConstructor().newInstance();
                                                      // 反射：类名可以是运行期才拿到的字符串
```

**为什么必须学**：Spring 的 IOC/DI、MyBatis 的结果映射、Jackson 的 JSON 转换、JUnit 找 `@Test` 方法 —— **底层全是反射**。不懂反射就看不懂任何框架的原理。

---

### 2. 获取 Class 对象

```java
Class<?> c1 = User.class;                          // 类名.class，编译期已知时用
Class<?> c2 = user.getClass();                     // 对象.getClass()
Class<?> c3 = Class.forName("com.example.User");   // ⭐ 全限定名，最灵活，会触发类初始化
```

> 一个类在 JVM 中**只有一个 `Class` 对象**，三种方式拿到的是同一个。

---

### 3. 核心 API

```java
Class<?> clazz = User.class;

// 创建对象
Object obj = clazz.getDeclaredConstructor().newInstance();   // ⭐ 需要无参构造（见第八章）

// 字段
Field field = clazz.getDeclaredField("name");
field.setAccessible(true);                    // ⭐ 突破 private 访问限制
field.set(obj, "张三");
Object value = field.get(obj);

// 方法
Method method = clazz.getDeclaredMethod("setName", String.class);
method.setAccessible(true);
method.invoke(obj, "张三");

// 注解
if (clazz.isAnnotationPresent(Table.class)) {
    Table table = clazz.getAnnotation(Table.class);
}
```

**`getXxx()` 和 `getDeclaredXxx()` 的区别**（容易搞混）：

| 方法 | 范围 |
| ---- | ---- |
| `getFields()` / `getMethods()` | 本类 **+ 父类**的 **`public`** 成员 |
| ⭐`getDeclaredFields()` / `getDeclaredMethods()` | **本类**的**所有**成员（含 `private`），**不含父类** |

> 实际用得最多的是 `getDeclaredXxx()` + `setAccessible(true)`。
>
> `clazz.newInstance()` 已在 JDK 9 废弃（会吞掉构造器异常），统一用 `getDeclaredConstructor().newInstance()`。

---

### 4. 框架是怎么用反射的

```java
// 简化版：模拟 Spring 的依赖注入
for (Field field : bean.getClass().getDeclaredFields()) {
    if (field.isAnnotationPresent(Autowired.class)) {
        field.setAccessible(true);
        field.set(bean, getBean(field.getType()));      // 把容器里的 Bean 塞进去
    }
}
```

| 框架 | 用反射做什么 |
| ---- | ---- |
| Spring | 扫描注解、创建 Bean、依赖注入、AOP 代理 |
| MyBatis | 把 `ResultSet` 映射成实体对象（**所以要求实体有无参构造**） |
| Jackson | 读写字段完成 JSON 序列化/反序列化 |
| JUnit | 找到标了 `@Test` 的方法并调用 |

> 这就解释了第八章那个坑：**写了有参构造导致无参构造消失，框架反射创建对象时直接报错。**

---

### 5. 反射的代价

| 代价 | 说明 |
| ---- | ---- |
| **性能开销** | 比直接调用慢：需要做访问检查，且 JIT 难以内联优化 |
| **破坏封装** | `setAccessible(true)` 能访问 `private`，绕过一切访问控制 |
| **失去编译期检查** | 方法名、字段名都是字符串，改名后编译不报错，**运行时才炸** |

优化手段：`setAccessible(true)` 可跳过访问检查，把 `Field`/`Method` 对象**缓存起来复用**（框架都是这么做的）。

> **业务代码里不要用反射。** 它是写框架和工具的手段，不是写业务的手段。

---

**B · 注解**

### 6. 注解的本质

**注解本身不产生任何行为**，它只是贴在代码上的一个"标签"。真正干活的是**读取注解的那段代码**（框架、反射、注解处理器）。

```java
@Service            // 这行本身什么也不做
public class UserServiceImpl { }
// 是 Spring 在启动时扫描到它，才去创建 Bean
```

| 分类 | 例子 |
| ---- | ---- |
| JDK 内置 | `@Override`、`@Deprecated`、`@SuppressWarnings`、`@FunctionalInterface` |
| **元注解**（修饰注解的注解） | `@Target`、`@Retention`、`@Documented`、`@Inherited` |
| 框架 / 自定义 | `@Service`、`@Transactional`、`@Data` |

---

### 7. 元注解

**`@Target`** —— 这个注解能用在哪里：

| 值 | 位置 |
| ---- | ---- |
| `TYPE` | 类、接口、枚举 |
| `FIELD` | 字段 |
| `METHOD` | 方法 |
| `PARAMETER` | 方法参数 |

**`@Retention`** —— 注解保留到什么阶段（⭐ 最关键）：

| 值 | 保留到 | 典型用途 |
| ---- | ---- | ---- |
| `SOURCE` | 仅源码，编译后丢弃 | `@Override`、Lombok |
| `CLASS` | 保留到字节码，运行时读不到（**默认值**） | 极少用 |
| ⭐`RUNTIME` | 运行时可通过反射读取 | **几乎所有框架注解** |

> **想用反射读取注解，必须声明为 `RUNTIME`。** 不写 `@Retention` 时默认是 `CLASS`，反射根本读不到 —— 这是自定义注解最常犯的错误。

---

### 8. 自定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)      // ⭐ 必须 RUNTIME
public @interface Log {
    String value() default "";           // 属性写成"方法"的形式
    boolean enabled() default true;
}
```

```java
@Log("下单")            // 只有一个属性且名为 value 时，可省略属性名
public void createOrder() { }
```

> 属性类型只能是：基本类型、`String`、`Class`、枚举、注解，以及这些类型的数组。

---

### 9. 注解 + AOP（自定义注解最主要的落地场景）

```java
// ① 定义注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
    String value();
}

// ② 写切面读注解
@Aspect
@Component
public class LogAspect {

    @Around("@annotation(log)")
    public Object around(ProceedingJoinPoint pjp, Log log) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        logger.info("{} 耗时 {}ms", log.value(), System.currentTimeMillis() - start);
        return result;
    }
}

// ③ 使用：业务代码只贴一行注解，逻辑完全解耦
@Log("下单")
public void createOrder() { ... }
```

> **操作日志、接口限流、权限校验、幂等控制、数据脱敏** —— 全是这个套路。面试问"你写过自定义注解吗"，答的就是它。

---

### 10. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 业务代码不用反射 | 反射是框架的手段，业务里用会牺牲可读性和性能 |
| 自定义注解必须 `@Retention(RUNTIME)` | 否则反射读不到 |
| 必须写 `@Target` 限定使用位置 | 防止被贴到不该贴的地方 |
| 反射的 `Field`/`Method` 要缓存复用 | 每次重新查找开销大 |
| 重写方法必须加 `@Override` | 让编译器校验签名（见第十章） |
| 废弃的 API 加 `@Deprecated` 并说明替代方案 | — |
| 自定义注解配合 AOP 使用 | 注解只是标记，逻辑放在切面里 |

---

### 11. 面试重点

**Q1：什么是反射？它有什么用？**

反射是在运行期动态获取类信息、创建对象、调用方法、访问字段的能力，不需要编译期知道具体类型。它是所有框架的基石 —— Spring 的 IOC/DI、MyBatis 的结果映射、Jackson 的 JSON 转换、JUnit 的用例执行全部依赖反射。

**Q2：获取 `Class` 对象有哪几种方式？**

`类名.class`、`对象.getClass()`、`Class.forName("全限定名")`。第三种最灵活且会触发类初始化。一个类在 JVM 中只有一个 `Class` 对象。

**Q3：`getFields()` 和 `getDeclaredFields()` 有什么区别？**

`getFields()` 返回本类和父类的所有 **`public`** 字段；`getDeclaredFields()` 返回**本类**声明的**全部**字段（包括 `private`），但不含父类。配合 `setAccessible(true)` 才能访问 `private` 成员。

**Q4：反射有什么缺点？**

性能比直接调用差（需做访问检查、JIT 难以内联）；破坏封装，能绕过 `private`；把方法名字段名变成字符串，失去编译期检查，改名后要到运行时才报错。

**Q5：`@Retention` 的三个取值有什么区别？**

`SOURCE` 只在源码阶段存在，编译后丢弃（如 `@Override`、Lombok）；`CLASS` 保留在字节码但运行时读不到，是**默认值**；`RUNTIME` 运行时可被反射读取，框架注解都用它。

**Q6：注解本身有功能吗？**

没有。注解只是元数据标记，不会产生任何行为。真正起作用的是读取注解的代码 —— 框架通过反射发现注解后执行相应逻辑。

**Q7：自定义注解怎么落地？**

定义时用 `@Target` 限定位置、`@Retention(RUNTIME)` 保证可被反射读取；使用时贴在目标方法上；最后写一个 AOP 切面用 `@annotation(xxx)` 切入，在切面里实现日志、限流、鉴权等横切逻辑。

---

## 二十二、JVM（内存结构、GC、类加载）

### 1. 运行时内存结构

```text
线程私有
├─ 程序计数器 ——— 当前执行到的字节码行号
├─ 虚拟机栈 ————— 每调一个方法压一个栈帧（局部变量表、操作数栈…）
└─ 本地方法栈 ——— native 方法用

线程共享
├─ 堆（Heap）—————————— ⭐ 对象实例，GC 的主战场
└─ 方法区 / 元空间 ————— 类元信息、运行时常量池、静态变量
```

| 区域 | 存什么 | 线程 | 可能的异常 |
| ---- | ---- | ---- | ---- |
| 程序计数器 | 字节码行号 | 私有 | **唯一不会 OOM 的区域** |
| 虚拟机栈 | 栈帧、局部变量 | 私有 | `StackOverflowError` |
| 本地方法栈 | native 方法 | 私有 | 同上 |
| ⭐**堆** | 对象实例、数组 | 共享 | `OOM: Java heap space` |
| ⭐**元空间** | 类元信息、常量池 | 共享 | `OOM: Metaspace` |

**JDK 8 的重大变化**：永久代（PermGen）被**元空间（Metaspace）**取代，元空间使用**本地内存**而非堆内存，默认不设上限。

> 注意：**字符串常量池和静态变量在 JDK 7 就已经移到堆中**了，不在元空间（呼应第六章、第九章）。

**堆的分代结构**：

```text
堆
├─ 新生代（约 1/3）
│   ├─ Eden      ┐
│   ├─ Survivor0 ├─ 默认比例 8 : 1 : 1
│   └─ Survivor1 ┘
└─ 老年代（约 2/3）
```

> ⚠️ 上面这张图是 **Parallel / CMS 的经典布局**（`-XX:NewRatio=2` 决定新生代占 1/3，`-XX:SurvivorRatio=8` 决定 8:1:1）。**G1（JDK 9+ 默认）已经不是这个结构**：堆被切成 2048 个左右大小相等的 **Region**，每个 Region 动态扮演 Eden / Survivor / Old / **Humongous**（存放超过半个 Region 的大对象）之一，新生代占比由 JVM 按暂停时间目标（`-XX:MaxGCPauseMillis`）自行调整，**默认在 5% ~ 60% 之间浮动**。所以在 G1 下 `8:1:1` 不再适用，**手动设 `-Xmn` 反而会关掉它的自适应能力**。

---

### 2. 对象的创建与分配

创建过程：**类加载检查 → 分配内存 → 零值初始化 → 设置对象头 → 执行构造器**。

内存分配的流转：

1. 优先在 **Eden** 分配
2. Eden 满了触发 **Minor GC**，存活对象移到 Survivor，**年龄 +1**
3. 年龄达到 **15**（默认）晋升**老年代**
4. **大对象直接进老年代**（避免在两个 Survivor 之间反复复制）
5. 老年代空间不足触发 **Full GC**

> **逃逸分析**：如果对象没有逃出方法作用域，JIT 可能直接**在栈上分配**，方法结束就随栈帧销毁。所以"对象一定在堆上"这句话并不严谨。

---

### 3. 垃圾回收

#### 怎么判断对象已死

| 算法 | 说明 |
| ---- | ---- |
| 引用计数 | 简单，但**无法解决循环引用**，Java **不使用** |
| ⭐**可达性分析** | 从 **GC Roots** 出发遍历，不可达的对象判定为可回收 |

**GC Roots 包括**：虚拟机栈中局部变量引用的对象、**静态变量引用的对象**、常量引用的对象、本地方法栈引用的对象。

> 这正好解释了第九章的坑：**静态变量是 GC Roots**，它引用的对象永远可达、永远不会被回收 —— 所以静态 `Map` 做缓存必然内存泄漏。

#### 四种引用强度（面试高频）

可达性分析只回答"可达 / 不可达"，但"可达"本身还分强弱：

| 引用类型 | 写法 | 回收时机 | 典型用途 |
| -------- | ---- | -------- | -------- |
| ⭐**强引用** | `Object o = new Object()` | **永不回收**，宁可 OOM | 平时写的所有代码 |
| **软引用** | `new SoftReference<>(obj)` | **内存不足时**才回收 | 本地缓存（图片、页面片段） |
| **弱引用** | `new WeakReference<>(obj)` | **下一次 GC 必回收** | `ThreadLocalMap` 的 key、`WeakHashMap` |
| 虚引用 | `new PhantomReference<>(obj, queue)` | 随时可能回收，`get()` 永远返回 `null` | 仅用于在对象被回收时收到通知，管理堆外内存（`DirectByteBuffer`） |

```java
// 软引用做缓存：内存紧张时 JVM 会自动清掉，不至于因为缓存把堆撑爆
SoftReference<byte[]> cache = new SoftReference<>(bigData);
byte[] data = cache.get();       // ⚠️ 随时可能是 null，必须判空后重新加载
```

> 这解释了第二十章 `ThreadLocal` 的设计：`ThreadLocalMap` 的 **key 用弱引用**，是为了线程还活着时 `ThreadLocal` 对象本身仍能被回收；但 **value 是强引用**，所以不 `remove()` 一定泄漏。
>
> 补充一个概念区分：**内存泄漏**（Memory Leak）是对象已经没用了却还被 GC Roots 引用着，回收不掉；**内存溢出**（OOM）是内存真的不够用了。泄漏积累到一定程度就会表现为溢出。

#### 回收算法

| 算法 | 特点 | 用于 |
| ---- | ---- | ---- |
| 标记-清除 | 会产生内存碎片 | 老年代 |
| ⭐**复制** | 无碎片，但浪费一半空间 | **新生代** |
| 标记-整理 | 无碎片，但移动对象效率低 | 老年代 |
| ⭐**分代收集** | 不同区域用不同算法 | 现代 JVM 的整体策略 |

**为什么新生代用复制算法**：新生代里 98% 的对象朝生夕死，只需复制存活的那一小部分，代价极低。而 Eden : S0 : S1 = 8 : 1 : 1 的划分，让每次只浪费 10% 的空间，而不是传统复制算法的 50%。

#### 垃圾收集器

| 收集器 | 特点 | 状态 |
| ---- | ---- | ---- |
| Serial | 单线程，STW 长 | 客户端场景 |
| Parallel Scavenge / Old | 多线程，**吞吐量优先** | **JDK 8 默认** |
| CMS | 并发标记清除，低延迟，有碎片 | **JDK 14 已移除** |
| ⭐**G1** | 按 Region 分区，**可预测停顿时间** | ⭐ **JDK 9+ 默认，当前主流** |
| ZGC / Shenandoah | 停顿 < 10ms，支持超大堆 | JDK 15+ 生产可用 |

**STW（Stop The World）**：GC 时暂停所有用户线程。**所有收集器都有 STW**，区别只在长短 —— GC 调优的本质就是减少 STW 时间。

#### 三色标记与并发标记（了解）

CMS、G1、ZGC 能做到"一边跑用户线程一边标记"，靠的都是**三色标记法**：

| 颜色 | 含义 |
| ---- | ---- |
| 白 | 还没被访问；标记结束仍是白色 → **判定为垃圾，回收** |
| 灰 | 自己已被访问，但**它引用的对象还没扫完** |
| 黑 | 自己和它引用的对象都扫完了 → 存活 |

**并发标记的风险**：标记的同时用户线程还在改引用关系，可能把一个本该存活的对象**漏标**，当成垃圾回收掉（那就是直接的数据错误）。两种解法：

- **SATB（原始快照）** —— G1 用：记录被**删除**的引用，一切以标记开始那一刻的快照为准。代价是可能把已经死掉的对象当成活的（**浮动垃圾**，留到下次 GC 再收），但绝不会误删。
- **增量更新** —— CMS 用：记录**新增**的引用，重新扫描这部分。

> 面试答到"漏标 → SATB / 增量更新 → 浮动垃圾"这个链条就足够了，不必深入写屏障的实现细节。

#### Minor GC 与 Full GC

| 对比项 | Minor GC（Young GC） | Full GC |
| ---- | ---- | ---- |
| 回收范围 | 仅新生代 | 整个堆 + 方法区 |
| 频率 | 高 | 低 |
| 耗时 | 短（几毫秒） | ⭐**长（几百毫秒到数秒）** |

> **Full GC 频繁是最典型的线上性能问题** —— 表现为接口大面积超时、响应时间毛刺。

---

### 4. 类加载机制

#### 生命周期

**加载 → 验证 → 准备 → 解析 → 初始化 → 使用 → 卸载**（验证、准备、解析合称"连接"）

| 阶段 | 做什么 |
| ---- | ---- |
| 加载 | 读取 `.class` 字节流，生成 `Class` 对象 |
| 验证 | 校验字节码是否合法 |
| ⭐**准备** | 给**静态变量**分配内存并赋**零值** |
| 解析 | 符号引用 → 直接引用 |
| ⭐**初始化** | 执行 `<clinit>()`：静态变量赋**真正的值** + 执行静态代码块 |

```java
static int a = 10;
// 准备阶段：a = 0
// 初始化阶段：a = 10   ← 这是常考的区分点
```

#### 双亲委派模型

```text
Bootstrap ClassLoader     启动类加载器（C++ 实现，加载 java.* 核心类）
        ↑
Platform / Ext ClassLoader   扩展类加载器
        ↑
Application ClassLoader   应用类加载器（加载 classpath 下的类）
        ↑
自定义 ClassLoader
```

**双亲委派**：收到加载请求时，先**向上委托给父加载器**，父加载器加载不了才自己动手。

**为什么这么设计**：

1. **安全** —— 防止核心类库被篡改。你自己写一个 `java.lang.String` 也加载不进去，因为请求会被 Bootstrap 优先处理
2. **避免重复加载** —— 保证同一个类在 JVM 中只有一份

> 打破双亲委派的典型场景：**Tomcat**（每个 webapp 一个独立加载器实现应用隔离）、**JDBC 的 SPI**（用线程上下文类加载器反向加载驱动实现）、**热部署**。

#### SPI 与 ServiceLoader

**SPI（Service Provider Interface）**：接口由一方定义，实现由另一方在自己的 jar 里提供，运行期自动发现。JDBC、Dubbo、SLF4J、Spring Boot 自动装配全都建立在这个思路上。

用法只有三步：

```text
① 定义接口：            com.xxx.PayService
② 实现方在自己 jar 里放：META-INF/services/com.xxx.PayService
   文件内容为实现类全限定名：com.yyy.AliPayServiceImpl
③ 调用方加载
```

```java
ServiceLoader<PayService> loader = ServiceLoader.load(PayService.class);
for (PayService s : loader) { s.pay(); }      // 自动发现所有 jar 中的实现
```

**为什么 SPI 会打破双亲委派**：`DriverManager` 是核心类（由 Bootstrap 加载），但它要加载的 MySQL 驱动在 classpath 下（只有 Application ClassLoader 能加载）。**父加载器需要用到子加载器才能加载的类，委派方向反了**，只能靠**线程上下文类加载器**（`Thread.currentThread().getContextClassLoader()`）绕过双亲委派。

> JDK 原生 SPI 的缺点是**一次性把所有实现全部实例化，不能按需加载、不能按名字取**。所以 Dubbo 自己做了增强版（`@SPI` + `@Adaptive`，可按 key 取实现），Spring Boot 则用 `spring.factories`（2.7 之后是 `META-INF/spring/...AutoConfiguration.imports`）走同一套思路。

---

### 5. 常用启动参数

```bash
-Xms2g -Xmx2g                        # ⭐ 堆的初始值和最大值，生产环境设成相同，避免动态扩容抖动
-Xmn512m                             # 新生代大小
-Xss512k                             # 每个线程的栈大小
-XX:MetaspaceSize=256m
-XX:+UseG1GC                         # 使用 G1 收集器
-XX:+HeapDumpOnOutOfMemoryError      # ⭐ OOM 时自动导出堆快照，事后分析全靠它
-XX:HeapDumpPath=/data/dump
```

---

### 6. 线上问题排查（实战题）

#### ① OOM 怎么排查

```bash
jps                                       # 找到 java 进程 pid
jmap -heap <pid>                          # 看各区域使用情况
jmap -histo:live <pid> | head -20         # 看对象数量/占用排行
jmap -dump:format=b,file=heap.hprof <pid> # 导出堆快照
# 再用 MAT / JProfiler 分析 hprof，找占用最大的对象及其 GC Roots 引用链
```

| OOM 类型 | 常见原因 |
| ---- | ---- |
| `Java heap space` | 内存泄漏（**静态集合、`ThreadLocal` 不 remove**）、一次性加载海量数据 |
| `Metaspace` | 动态生成类过多（大量反射、CGLIB 代理） |
| `StackOverflowError` | **递归没有终止条件** |
| `unable to create new native thread` | 线程创建过多（**没用线程池**） |

> 这几条原因分别对应第九章、第二十章、第二十一章的坑 —— **它们最终都会表现为 OOM**。

#### ② CPU 100% 怎么排查

```bash
top                                       # 找到 CPU 高的 java 进程 pid
top -Hp <pid>                             # 找到该进程内 CPU 最高的线程 tid
printf "%x\n" <tid>                       # 线程 id 转成十六进制
jstack <pid> | grep <十六进制tid> -A 30    # 在线程栈中定位到具体代码行
```

常见原因：**死循环、频繁 Full GC、正则回溯爆炸、大量线程锁竞争**。

> 现在生产环境更常用 **Arthas**：`thread -n 3` 一条命令直接列出最忙的三个线程及其堆栈，比上面这串命令方便得多。

---

### 7. 面试重点

**Q1：JVM 的内存结构？哪些线程私有，哪些共享？**

线程私有：程序计数器、虚拟机栈、本地方法栈；线程共享：堆、方法区（JDK 8 后是元空间）。其中程序计数器是**唯一不会发生 OOM** 的区域。

**Q2：堆和栈的区别？**

栈是线程私有的，存放栈帧和局部变量，随方法调用入栈、结束出栈，空间不足抛 `StackOverflowError`；堆是线程共享的，存放对象实例，是 GC 的主要区域，空间不足抛 `OutOfMemoryError`。

**Q3：JDK 8 为什么用元空间取代永久代？**

永久代在堆内且大小固定，容易因加载类过多而 OOM，且难以调优；元空间使用**本地内存**，默认不设上限，能显著降低 `OOM: PermGen` 的发生。另外这也是 HotSpot 与 JRockit 融合的需要。

**Q4：怎么判断对象可以被回收？GC Roots 有哪些？**

用**可达性分析**：从 GC Roots 出发遍历引用链，不可达的对象判定为可回收。引用计数法因为无法解决循环引用，Java 不采用。GC Roots 包括虚拟机栈中局部变量引用的对象、**静态变量引用的对象**、常量引用的对象、本地方法栈引用的对象。

**Q5：为什么新生代用复制算法？**

新生代对象绝大多数朝生夕死，存活率极低，复制存活的一小部分成本很低且**不产生内存碎片**。通过 Eden : S0 : S1 = 8:1:1 的划分，每次只浪费 10% 的空间，而非传统复制算法的一半。

**Q6：Minor GC 和 Full GC 的区别？**

Minor GC 只回收新生代，频率高但耗时短（毫秒级）；Full GC 回收整个堆和方法区，频率低但**耗时可达数百毫秒甚至数秒**，会造成明显的 STW，是线上接口超时的常见原因。

**Q7：什么是双亲委派？为什么要这么设计？**

类加载请求先逐级向上委托给父加载器，父加载器无法加载时才由自己加载。目的是**保证核心类库的安全**（自定义的 `java.lang.String` 无法覆盖官方实现）和**避免类的重复加载**。Tomcat、JDBC 的 SPI、热部署都会打破它。

**Q8：类加载的过程？"准备"和"初始化"有什么区别？**

加载 → 验证 → 准备 → 解析 → 初始化。**准备**阶段给静态变量分配内存并赋**零值**；**初始化**阶段执行 `<clinit>()`，才把静态变量赋成代码里写的值并执行静态代码块。

**Q9：线上 OOM 怎么排查？**

先用 `-XX:+HeapDumpOnOutOfMemoryError` 保证能拿到堆快照；用 `jmap -histo:live` 看对象排行做初判，用 `jmap -dump` 导出后在 MAT 中分析大对象及其 GC Roots 引用链，定位到具体的泄漏点。常见根因是静态集合、`ThreadLocal` 未清理、一次性加载大量数据。

**Q10：线上 CPU 飙到 100% 怎么排查？**

`top` 定位进程 → `top -Hp <pid>` 定位线程 → `printf "%x"` 转十六进制 → `jstack` 中搜索该线程栈定位到代码。常见原因是死循环、频繁 Full GC、正则回溯、锁竞争。生产环境更推荐用 Arthas 的 `thread -n 3`。

**Q11：Java 的四种引用分别是什么？各用在哪？**

**强引用**（普通 `new`）永不回收，宁可 OOM；**软引用**（`SoftReference`）内存不足时回收，适合做本地缓存；**弱引用**（`WeakReference`）下次 GC 必回收，`ThreadLocalMap` 的 key 和 `WeakHashMap` 用的就是它；**虚引用**（`PhantomReference`）`get()` 永远返回 null，只用于在对象被回收时收到通知，管理堆外内存。顺带区分：**内存泄漏**是没用的对象还被 GC Roots 引用着回收不掉，**内存溢出**是内存真的不够用，泄漏积累会导致溢出。

**Q12：G1 和 CMS 的区别？G1 的堆结构是怎样的？**

CMS 是"标记-清除"的并发收集器，低延迟但**会产生碎片**且 JDK 14 已移除；G1 把堆划分成约 2048 个大小相等的 **Region**，每个 Region 动态扮演 Eden / Survivor / Old / Humongous，整体是"标记-整理"因此**没有碎片**，并且可以通过 `-XX:MaxGCPauseMillis` 设定**可预测的停顿时间**，优先回收垃圾最多的 Region（Garbage First 名字的由来）。G1 下新生代占比由 JVM 自适应，不要再手动设 `-Xmn`。

**Q13：并发标记为什么会漏标？怎么解决？**

并发标记时用户线程还在改引用关系，可能导致本该存活的对象被标成白色回收掉。解决方案有两种：G1 用 **SATB（原始快照）**，记录被删除的引用、以标记开始时的快照为准，代价是产生**浮动垃圾**；CMS 用**增量更新**，记录新增的引用并重新扫描。

**Q14：什么是 SPI？它和双亲委派有什么关系？**

SPI 是"接口与实现分离、运行期自动发现实现"的扩展机制，实现方在 `META-INF/services/` 下声明实现类，调用方用 `ServiceLoader.load()` 加载。它**打破了双亲委派**：`DriverManager` 这类核心类由 Bootstrap 加载，却需要加载 classpath 下的驱动实现，委派方向是反的，只能借助**线程上下文类加载器**完成。JDBC、Dubbo、SLF4J、Spring Boot 自动装配都基于这个思路。

---

## 二十三、常用设计模式

> 23 种设计模式不需要全学。这里只讲**面试会问、项目真会用**的 9 个，每个都配上 JDK 或 Spring 里的真实出处。

### 1. 设计原则（SOLID）

| 原则 | 一句话 |
| ---- | ---- |
| 单一职责 | 一个类只做一件事 |
| ⭐**开闭原则** | **对扩展开放，对修改关闭** —— 所有设计模式的共同目标 |
| 里氏替换 | 子类必须能完全替换父类 |
| 接口隔离 | 接口要小而专，不强迫实现类依赖用不到的方法 |
| 依赖倒置 | 面向接口编程，不依赖具体实现 |
| 迪米特法则 | 只和直接朋友通信，降低耦合 |

> 记住**开闭原则**这一条就够了：判断一个设计好不好，看**新增需求时要不要改老代码**。

---

### 2. 单例（Singleton）

```java
// ① 饿汉式：类加载时就创建，天然线程安全 —— ⭐ 大多数场景足够
public class Config {
    private static final Config INSTANCE = new Config();
    private Config() { }
    public static Config getInstance() { return INSTANCE; }
}
```

```java
// ② 双重检查锁（DCL）：懒加载 + 线程安全
public class Config {
    private static volatile Config instance;         // ⭐ volatile 绝对不能省

    private Config() { }

    public static Config getInstance() {
        if (instance == null) {                      // 第一次检查：避免每次都加锁
            synchronized (Config.class) {
                if (instance == null) {              // 第二次检查：防止重复创建
                    instance = new Config();
                }
            }
        }
        return instance;
    }
}
```

```java
// ③ 静态内部类：懒加载 + 线程安全 + 无锁 —— ⭐ 写法最优雅
public class Config {
    private Config() { }

    private static class Holder {
        private static final Config INSTANCE = new Config();
    }

    public static Config getInstance() { return Holder.INSTANCE; }
}
```

```java
// ④ 枚举：⭐ 最安全（见第十二章）
public enum Config {
    INSTANCE;
}
```

| 写法 | 懒加载 | 线程安全 | 防反射/反序列化 |
| ---- | :----: | :------: | :-------------: |
| 饿汉式 | ✘ | ✔ | ✘ |
| 双重检查锁 | ✔ | ✔（需 `volatile`） | ✘ |
| 静态内部类 | ✔ | ✔ | ✘ |
| **枚举** | ✘ | ✔ | ⭐**✔** |

> **DCL 为什么必须加 `volatile`**：`new` 分为「分配内存 → 初始化 → 赋值引用」三步，指令重排后可能先赋值再初始化，导致别的线程拿到**半初始化对象**（见第二十章）。
>
> 实际项目里几乎不手写单例 —— **Spring 的 Bean 默认就是单例**。

---

### 3. 工厂（Factory）

把 `new` 的逻辑集中到一处，调用方不关心具体实现类。

```java
public class PaymentFactory {
    public static Payment create(String type) {
        return switch (type) {
            case "ali" -> new AliPay();
            case "wx"  -> new WechatPay();
            default    -> throw new IllegalArgumentException(type);
        };
    }
}
```

> **出处**：JDK 的 `Calendar.getInstance()`、`LocalDate.of()`；Spring 的 `BeanFactory`。
>
> 简单工厂本身没解决开闭原则（加一种支付还是要改 `switch`），实际项目中常被下面的**策略模式**取代。

---

### 4. 建造者（Builder）

参数多、可选参数多时，避免构造器爆炸。

```java
User u = User.builder().id(1L).name("张三").age(18).build();
```

> **出处**：`StringBuilder`、OkHttp 的 `Request.Builder`。手写实现见第十二章第 7 节，实际项目直接用 Lombok 的 `@Builder`。

---

### 5. 策略（Strategy）—— ⭐ 业务里最常用

**用来消灭一长串 `if-else`，新增分支不用改老代码。**

```java
// ① 定义策略接口
public interface PayStrategy {
    String type();
    void pay(BigDecimal amount);
}

// ② 每种实现独立一个类，交给 Spring 管理
@Component
public class AliPayStrategy implements PayStrategy {
    public String type() { return "ali"; }
    public void pay(BigDecimal amount) { ... }
}

// ③ ⭐ Spring 把所有实现注入进来，按 type 路由
@Service
public class PayService {

    private final Map<String, PayStrategy> strategies;

    public PayService(List<PayStrategy> list) {           // 构造器注入所有实现类
        this.strategies = list.stream()
                .collect(Collectors.toMap(PayStrategy::type, s -> s));
    }

    public void pay(String type, BigDecimal amount) {
        PayStrategy strategy = strategies.get(type);
        if (strategy == null) throw new BizException(ErrorCode.UNSUPPORTED_PAY_TYPE);
        strategy.pay(amount);
    }
}
```

> 新增一种支付方式，只需要**加一个 `@Component` 实现类**，`PayService` 一行都不用改 —— 这就是开闭原则的实际落地，也是多态（第十章）最典型的应用。
>
> 面试问"你在项目里用过什么设计模式"，这是最好的答案。

---

### 6. 模板方法（Template Method）

父类定义流程骨架并用 `final` 锁住，子类只填变化的步骤（完整代码见第十一章第 1 节）。

> **出处**：JDK 的 `AbstractList`；Spring 的 `JdbcTemplate`、`RestTemplate`、`AbstractApplicationContext`。

---

### 7. 代理（Proxy）—— ⭐ AOP 的原理

在不修改原对象的前提下增强其功能。**静态代理**要手写代理类，太麻烦；实际都用**动态代理**（运行期生成代理类）。

| 方式 | 前提 | 原理 | 用在 |
| ---- | ---- | ---- | ---- |
| ⭐**JDK 动态代理** | **目标类必须有接口** | 反射 + `InvocationHandler` | Spring AOP（有接口时） |
| ⭐**CGLIB** | 无需接口 | **生成目标类的子类** → `final` 类和 `final` 方法无法代理 | **Spring Boot 默认** |

```java
Object proxy = Proxy.newProxyInstance(
        target.getClass().getClassLoader(),
        target.getClass().getInterfaces(),
        (p, method, args) -> {
            System.out.println("前置增强");
            Object result = method.invoke(target, args);
            System.out.println("后置增强");
            return result;
        });
```

> **出处**：Spring AOP、`@Transactional`、MyBatis 的 Mapper 接口（只有接口没有实现类，全靠动态代理）。
>
> 这也解释了第十四章的坑：**同一个类内部直接调用自己的方法不会经过代理对象，所以 `@Transactional` 会失效。**

---

### 8. 责任链（Chain of Responsibility）

请求沿着链条传递，每个节点自行决定是处理掉还是往下传。

> **出处**：Servlet 的 `Filter`、Spring MVC 的 `HandlerInterceptor`、Netty 的 `ChannelPipeline`、Spring Gateway 的过滤器链。
>
> 业务中适合做**多级审批、参数校验链、风控规则链**。

---

### 9. 观察者（Observer）

一对多的通知机制：状态变化时自动通知所有订阅者，让发布方和订阅方解耦。

```java
// 下单后只发一个事件，不用在下单代码里硬编码"发短信、加积分、送优惠券"
publisher.publishEvent(new OrderCreatedEvent(order));

@EventListener
public void onOrderCreated(OrderCreatedEvent event) {
    smsService.send(event.getOrder());
}
```

> **出处**：Spring 的 `ApplicationEvent` + `@EventListener`。新增一个下单后动作，只要加一个监听器，下单主流程完全不用动。

---

### 10. 装饰器与适配器（认识即可）

| 模式 | 作用 | 出处 |
| ---- | ---- | ---- |
| **装饰器** | 动态给对象叠加功能，不改原类 | ⭐**IO 流的三层包装**（第十九章） |
| **适配器** | 转换接口，让不兼容的类协作 | `InputStreamReader`（字节流 → 字符流）、Spring MVC 的 `HandlerAdapter` |

> 第十九章那个 `new BufferedReader(new InputStreamReader(new FileInputStream(...)))`，同时用到了这两个模式 —— **适配器负责字节转字符，装饰器负责加缓冲**。

---

### 11. 面试重点

**Q1：手写单例，几种写法怎么选？**

饿汉式最简单、类加载即创建、天然线程安全，大多数场景够用；需要懒加载用**静态内部类**（无锁、优雅）或**双重检查锁**（必须加 `volatile`）；要求绝对安全用**枚举**，它天然防反射和反序列化破坏。

**Q2：双重检查锁为什么必须加 `volatile`？**

`instance = new Config()` 不是原子操作，分为分配内存、初始化对象、把引用指向内存三步。JVM 可能重排成 1-3-2，此时另一个线程在第一次检查时看到 `instance != null` 就直接返回，拿到的是**尚未初始化完成的对象**。`volatile` 禁止这种重排。

**Q3：JDK 动态代理和 CGLIB 有什么区别？**

JDK 动态代理基于反射和 `InvocationHandler`，**要求目标类实现接口**；CGLIB 通过生成目标类的**子类**来代理，不需要接口，但**无法代理 `final` 类和 `final` 方法**。Spring AOP 有接口时用 JDK 代理，**Spring Boot 默认统一使用 CGLIB**。

**Q4：你在项目里用过哪些设计模式？**

最实用的三个：**策略模式**（用 Spring 注入所有实现类到 Map，按类型路由，消灭 `if-else`）、**模板方法**（父类定流程、子类填步骤）、**代理模式**（自定义注解 + AOP 实现日志、限流、鉴权）。

**Q5：Spring 中用到了哪些设计模式？**

| 模式 | 体现 |
| ---- | ---- |
| 单例 | Bean 默认作用域 |
| 工厂 | `BeanFactory`、`FactoryBean` |
| 代理 | AOP、`@Transactional` |
| 模板方法 | `JdbcTemplate`、`RestTemplate` |
| 观察者 | `ApplicationEvent` + `@EventListener` |
| 责任链 | `HandlerInterceptor`、`Filter` |
| 适配器 | `HandlerAdapter` |

**Q6：策略模式怎么在 Spring 里落地？**

策略接口里定义一个 `type()` 方法，每个实现类加 `@Component`；在调用方用构造器注入 `List<策略接口>`，用 `Collectors.toMap()` 转成 `Map<type, 策略>`，运行时按 key 取出对应策略执行。新增策略只需加实现类，调用方代码完全不用修改。

---

## 二十四、JDK 版本演进与新特性总览

> 前面各章的新语法是按知识点分散讲的，这一章把它们**按版本收拢成一张表**，解决一个很实际的问题：**这个写法我们项目到底能不能用**。

### 1. 版本选型

| 版本 | 发布 | 现状 |
| ---- | ---- | ---- |
| **JDK 8** | 2014，LTS | ⚠️ 存量最大的版本，Lambda / Stream / `LocalDate` 都出自这里；公共免费更新早已结束 |
| JDK 11 | 2018，LTS | 过渡版本，新项目没必要停在这 |
| ⭐**JDK 17** | 2021，LTS | ⭐ **当前新项目的主流基线**，Spring Boot 3.x **强制要求 17+** |
| ⭐**JDK 21** | 2023，LTS | ⭐ 虚拟线程转正，正在快速普及 |

> 只需记两条组合：**Spring Boot 2.x + JDK 8 是存量，Spring Boot 3.x + JDK 17/21 是增量**。升级最大的收益其实不是语法，而是 **GC（G1 / ZGC）和整体性能**，以及安全更新。

### 2. 语法与 API 速查

| 特性 | 版本 | 一句话 | 详见 |
| ---- | ---- | ---- | ---- |
| Lambda / Stream / 函数式接口 | 8 | 集合处理的标准写法 | 第十七章 |
| `LocalDate` / `LocalDateTime` | 8 | 替代 `Date` / `SimpleDateFormat` | 第十八章 |
| `Optional` | 8 | 显式表达"值可能不存在" | 第十七章 |
| 接口 `default` / `static` 方法 | 8 | 接口能加方法而不破坏已有实现类 | 第十一章 |
| `ConcurrentHashMap` 改 CAS + `synchronized` | 8 | 锁粒度降到单个桶 | 第十六章 |
| 元空间取代永久代 | 8 | 类元信息改用本地内存 | 第二十二章 |
| `var` 局部变量类型推断 | 10 | `var list = new ArrayList<User>();`；⚠️ 只能用于局部变量，右边看不出类型时别用 | — |
| `String.isBlank/strip/repeat/lines` | 11 | 常用字符串 API 补齐 | 第六章 |
| `Files.readString` / `writeString` | 11 | 一行读写文件 | 第十九章 |
| ⭐`switch` 表达式（箭头 + `yield`） | 14 | 不穿透、能返回值 | 第五章 |
| ⭐文本块 `"""` | 15 | 多行 SQL / JSON 不用拼 `\n` | 第六章 |
| 偏向锁废弃（18 移除） | 15 | 锁升级只到轻量级锁 | 第二十章 |
| ⭐`instanceof` 模式匹配 | 16 | 判断和转型一步到位 | 第十章 |
| ⭐`record` | 16 | 一行定义不可变数据载体 | 第八章、见下 |
| `sealed` 密封类 | 17 | 限定谁能继承 / 实现 | 见下 |
| ⭐`switch` 模式匹配 + `case null` | 21 | 按类型分支，替代一串 `instanceof` | 见下 |
| ⭐**虚拟线程** | 21 | 同步的写法，异步的吞吐 | 第二十章 |
| `SequencedCollection` | 21 | 集合统一了 `getFirst()` / `getLast()` / `reversed()` | 第十六章 |

### 3. 三个值得单独看一眼的

**① `record` —— DTO / VO 的正确写法（JDK 16+）**

```java
public record OrderVO(Long id, String name, BigDecimal amount) { }
// 自动生成：全参构造器、id()/name()/amount() 读取方法、equals / hashCode / toString
```

> 两个注意点：字段是 `final` 的（**没有 setter**），读取方法**不带 `get` 前缀**（是 `id()` 不是 `getId()`）。
>
> 适合接口返回值、值对象、方法内的临时数据结构；**需要 setter、需要继承、或者框架强依赖 JavaBean 规范的地方仍然用普通类 + Lombok**。

**② 密封类 `sealed`（JDK 17+）**

```java
public sealed interface Result permits Success, Failure { }
```

限定只有列出的类型能实现它。真正的价值是配合下面的模式匹配：**编译器能检查分支是否穷举**，新增一种子类型时，所有没处理它的 `switch` 都会编译报错，逼你补全 —— 比运行期才发现漏分支安全得多。

**③ `switch` 模式匹配（JDK 21）**

```java
String desc = switch (shape) {
    case null                       -> "空";                        // ⭐ 终于能处理 null 了
    case Circle c                   -> "圆，半径 " + c.radius();
    case Rect r when r.w() == r.h() -> "正方形";                     // when 守卫条件
    case Rect r                     -> "矩形";
};
```

> 什么时候用它、什么时候用多态（第十章）：**行为属于对象自己**（`pay()`、`validate()`）就用多态；**只是外部对不同类型做展示或转换**，用模式匹配更直接，不必为此往实体类里塞方法。

### 4. 面试重点

**Q1：你们项目用的是哪个 JDK 版本？为什么？**

按实际情况答，但要说出**理由**：新项目用 JDK 17，因为 Spring Boot 3.x 强制要求 17+，默认 G1，且能用 `record`、文本块、模式匹配、`switch` 表达式；用 21 的最大动机是**虚拟线程**。老项目留在 8 是因为升级要回归全部依赖，尤其是 `javax` → `jakarta` 的包名变更。

**Q2：JDK 8 升级到 17，收益和风险分别是什么？**

收益：GC 与整体性能（G1 默认、ZGC 可用）、`String` 紧凑存储省内存、一批实用语法（`record`、文本块、模式匹配、`switch` 表达式）、以及 JDK 8 免费公共更新结束带来的安全问题。风险：`javax` → `jakarta` 包名变更、模块系统限制反射访问 JDK 内部 API（需要 `--add-opens`）、老版本 CGLIB / Lombok / 字节码增强类工具不兼容。

**Q3：`record` 和 Lombok 的 `@Data` 有什么区别？**

`record` 是**语言层面**的不可变数据载体，字段全部 `final`、没有 setter、读取方法不带 `get` 前缀，由编译器直接生成；`@Data` 是**注解处理器**在编译期插入 getter/setter，对象是可变的。传输不可变数据（VO、返回值、值对象）用 `record`，需要可变状态的实体类用普通类 + Lombok。另外 `record` 的 `equals`/`hashCode` 由语言规范保证，不存在 `@Data` 那些继承、默认值、无参构造的坑（见第八章第 8 节）——不需要可变性时，`record` 是更安全的选择。

**Q4：`var` 可以随便用吗？**

只能用于**局部变量**（不能用于字段、方法参数、返回值），且必须在声明时初始化。适合右边已经写清楚类型的场景（`var user = new User();`、`var entry : map.entrySet()`）；如果右边是方法调用（`var result = service.query();`），读代码的人看不出类型，反而降低可读性，这种情况不要用。
