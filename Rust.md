# Rust

## 目录

**基础语法**

- [一、变量、常量与可变性](#一变量常量与可变性)
- [二、基本数据类型](#二基本数据类型)
- [三、运算符与类型转换](#三运算符与类型转换)
- [四、流程控制（分支、循环、match）](#四流程控制分支循环match)
- [五、函数、闭包与函数式编程](#五函数闭包与函数式编程)
- [六、字符串（String 与 &str）](#六字符串string-与-str)
- [七、复合类型（元组、数组、切片、Vec）](#七复合类型元组数组切片vec)

**核心机制（Rust 的灵魂）**

- [八、所有权（Ownership）](#八所有权ownership)
- [九、借用与引用（Borrowing）](#九借用与引用borrowing)
- [十、生命周期（Lifetime）](#十生命周期lifetime)
- [十一、结构体与方法](#十一结构体与方法)
- [十二、枚举与模式匹配](#十二枚举与模式匹配)
- [十三、错误处理（Option 与 Result）](#十三错误处理option-与-result)

**抽象与泛型**

- [十四、泛型](#十四泛型)
- [十五、Trait 特征](#十五trait-特征)
- [十六、智能指针（Box、Rc、RefCell）](#十六智能指针boxrcrefcell)

**标准库与工程化**

- [十七、集合类型（Vec、HashMap、HashSet）](#十七集合类型vechashmaphashset)
- [十八、迭代器](#十八迭代器)
- [十九、模块系统与 Cargo 工程管理](#十九模块系统与-cargo-工程管理)
- [二十、测试、文档与常用生态库](#二十测试文档与常用生态库)
- [二十一、宏（macro_rules! 与过程宏）](#二十一宏macro_rules-与过程宏)

**并发与进阶**

- [二十二、多线程与并发](#二十二多线程与并发)
- [二十三、异步编程（async/await 与 Tokio）](#二十三异步编程asyncawait-与-tokio)
- [二十四、Unsafe Rust 与 FFI](#二十四unsafe-rust-与-ffi)
- [二十五、内存布局与性能优化](#二十五内存布局与性能优化)
- [二十六、常见设计模式与工程实践](#二十六常见设计模式与工程实践)

---

## 一、变量、常量与可变性

### 1. let：默认不可变

Rust 用 `let` 声明变量，**默认不可变（immutable）**，要修改必须显式加 `mut`：

```rust
let x = 5;
x = 6;              // ✘ 编译错误：cannot assign twice to immutable variable

let mut y = 5;
y = 6;              // ✔
```

这是 Rust 与 Java/C++ 最直观的一个区别：其他语言默认可变、想要只读要加 `final`/`const`；Rust 把默认值反过来了。

| 对比项 | Rust `let` | Rust `let mut` | Java 变量 |
| ------ | ---------- | -------------- | --------- |
| 默认语义 | 不可变 | 可变 | 可变 |
| 只读写法 | 默认就是 | — | 手动加 `final` |
| 编译器态度 | 可变才是特例，需显式声明 | — | 不可变才是特例 |

> **为什么默认不可变**：不可变是并发安全和编译器优化的前提 —— 一个值只要不会被改，就可以放心地被多处共享；`mut` 相当于一个显式的"这里会发生修改"的标记，代码审查和编译器都能一眼看到。

**术语提醒**：Rust 里 `let x = 5` 叫**绑定（binding）**，不叫"赋值"。`mut` 是绑定的属性，不是类型的属性 —— 这个区别在第 7 节会变成一个高频坑。

---

### 2. 变量遮蔽（Shadowing）

用 `let` 重复声明同名变量，会**遮蔽**掉前一个，这是完全合法的：

```rust
let x = 5;
let x = x + 1;          // 6，遮蔽前一个 x
let x = x * 2;          // 12
println!("{x}");        // 12
```

遮蔽和 `mut` 看起来像，但完全是两回事：

| 对比项 | 遮蔽（`let` 再声明） | `mut`（可变绑定） |
| ------ | ------------------ | ---------------- |
| 本质 | 创建一个**新变量**，旧的被覆盖 | 修改**同一个**变量的值 |
| 能否改类型 | ⭐ **能** | 不能 |
| 作用域 | 受作用域限制，出了块就恢复 | 无此行为 |

```rust
// ⭐ 遮蔽最实用的场景：类型转换后仍用同一个名字
let input = "42";                       // &str
let input: i32 = input.parse().unwrap();  // i32，名字不用改成 input_num

let spaces = "   ";
let spaces = spaces.len();              // ✔ &str → usize

let mut spaces = "   ";
spaces = spaces.len();                  // ✘ mut 不能改变类型
```

**作用域内遮蔽**：

```rust
let x = 5;
{
    let x = x * 2;
    println!("{x}");    // 10
}
println!("{x}");        // 5，出了内层块，外层的 x 依然是 5
```

> 遮蔽在业务代码中主要用来做**"清洗后覆盖原名"**：`let config = load(); let config = validate(config)?;` —— 避免出现 `config1`、`config2`、`configChecked` 这类命名。滥用（在同一函数里遮蔽七八次）反而降低可读性。

---

### 3. const 与 static

除了 `let`，Rust 还有两种真正意义上的"常量"：

```rust
const MAX_POINTS: u32 = 100_000;         // ⭐ 编译期常量，最常用
static APP_NAME: &str = "my-service";    // 静态变量，有固定内存地址
```

| 对比项 | `const` | `static` | `let`（不可变） |
| ------ | ------- | -------- | -------------- |
| 类型标注 | ⭐ **必须写** | ⭐ **必须写** | 可省略（自动推断） |
| 求值时机 | 编译期 | 编译期 | 运行期 |
| 内存 | **没有固定地址**，编译时内联到每个使用处 | 有唯一固定地址，`'static` 生命周期 | 栈上，随作用域销毁 |
| 可变性 | 永不可变 | `static mut` 可变，但**需要 `unsafe`** | 加 `mut` 即可变 |
| 声明位置 | 任意作用域（含全局） | 任意作用域（含全局） | 只能在函数内 |
| 命名规范 | `SCREAMING_SNAKE_CASE` | `SCREAMING_SNAKE_CASE` | `snake_case` |

**如何选择**：⭐ **默认一律用 `const`**。只有两种情况才用 `static`：需要一个全局唯一地址（比如要取 `&'static` 引用、或数据很大不想被内联复制），以及需要全局可变状态（此时也不该用 `static mut`，见第 7 节）。

**`const` 只能是编译期可求值的表达式**：

```rust
const A: u32 = 3 * 60;              // ✔ 字面量运算
const B: [u8; 4] = [0; 4];          // ✔ 数组
const C: u32 = compute();           // ✘ 除非 compute 是 const fn

const fn compute() -> u32 { 42 }    // ⭐ const fn：可在编译期求值的函数
const D: u32 = compute();           // ✔
```

**数字字面量可以用 `_` 分隔**，`100_000` 比 `100000` 好读，这是 Rust 的常见写法。

---

### 4. 类型推断与类型标注

Rust 是静态强类型语言，但绝大多数情况下**类型可以自动推断**，不用手写：

```rust
let x = 5;              // 推断为 i32（整数默认类型）
let y = 3.14;           // 推断为 f64（浮点默认类型）
let s = "hello";        // &str
let flag = true;        // bool
```

**必须手写类型标注的三种情况**：

```rust
// ① 编译器无法确定时，比如 parse 的目标类型
let n: i32 = "42".parse().unwrap();
let n = "42".parse::<i32>().unwrap();     // ⭐ 或用 turbofish 写在方法上

// ② 空集合，元素类型无从推断
let v: Vec<String> = Vec::new();
let v = Vec::<String>::new();             // 等价

// ③ 需要非默认的数字类型
let big: u64 = 1;
let small: u8 = 255;
```

> **规范**：函数签名的参数和返回值**必须**写类型（Rust 强制要求，不推断），函数体内部则能推就推。局部变量到处写 `let x: i32 = 5;` 属于噪音。

---

### 5. 解构与延迟初始化

**解构绑定**：`let` 左边可以是任意模式（pattern），这是第十二章模式匹配的预习：

```rust
let (a, b, c) = (1, 2.0, "three");      // 元组解构
let [x, y, z] = [1, 2, 3];              // 数组解构

let Point { x, y } = point;             // 结构体解构
let (mut a, b) = (1, 2);                // ⭐ mut 可以只加在其中一个上

let (first, ..) = (1, 2, 3);            // .. 忽略剩余
```

**延迟初始化**：`let` 可以先声明后赋值，编译器会通过控制流分析确保"使用前一定已初始化"：

```rust
let level;                              // 注意：这里不需要 mut
if score >= 60 {
    level = "pass";
} else {
    level = "fail";
}
println!("{level}");                    // ✔ 两个分支都赋了值

let x;
if cond { x = 1; }
println!("{x}");                        // ✘ 编译错误：可能未初始化
```

> 首次赋值不算"修改"，所以不需要 `mut`。但更地道的写法是**直接用 if 表达式赋值**（第四章会讲）：
> ```rust
> let level = if score >= 60 { "pass" } else { "fail" };
> ```

---

### 6. 作用域与析构顺序

变量在离开作用域时自动释放（调用 `Drop`），**释放顺序与声明顺序相反**（后声明的先释放）：

```rust
{
    let a = Resource::new("A");
    let b = Resource::new("B");
}   // 先 drop b，再 drop a
```

这条规则在持有锁、数据库事务、文件句柄时很关键 —— 依赖释放顺序的代码要注意声明顺序。需要提前释放可以用 `drop(x)` 手动调用，或者新开一个 `{}` 块把作用域缩小。

```rust
let data = expensive_load();
let result = process(&data);
drop(data);                 // ⭐ 提前释放，后面还有耗时操作时能省内存
long_running_task(result);
```

---

### 7. 高频踩坑点（重点）

**① `let _ = ...` 会立即丢弃值（实战最容易出的 bug）**

`_` 是"不绑定"，值当场就被 drop 了；`_name` 才是真正的绑定，只是不产生"未使用"警告。

```rust
let _ = mutex.lock().unwrap();        // ✘ 锁当场就释放了，等于没加锁
let _guard = mutex.lock().unwrap();   // ✔ 守卫活到作用域结束，锁才有效
```

> 同理适用于 `MutexGuard`、`tracing` 的 span guard、临时文件句柄等所有 RAII 守卫类型。这类 bug 编译能过、测试也常常过，只在并发下偶发，非常难查。

**② `mut` 加错位置：可变绑定 ≠ 可变引用**

```rust
let mut r = &v1;      // 绑定可变：能改 r 指向谁，不能通过 r 改数据
r = &v2;              // ✔

let r = &mut v;       // 引用可变：能通过 r 改数据，不能改 r 指向谁
*r += 1;              // ✔

let mut r = &mut v;   // 两者都可变
```

| 写法 | 能改指向 | 能改数据 |
| ---- | :------: | :------: |
| `let r = &T` | ✘ | ✘ |
| `let mut r = &T` | ✔ | ✘ |
| `let r = &mut T` | ✘ | ✔ |
| `let mut r = &mut T` | ✔ | ✔ |

**③ 不可变绑定不能借出可变引用**

```rust
let s = String::from("hi");
s.push_str("!");        // ✘ push_str 需要 &mut self，但 s 不是 mut

let mut s = String::from("hi");
s.push_str("!");        // ✔
```

> 报错信息 `cannot borrow as mutable` 一般就是这个原因：不是借用规则的问题，只是漏写了 `mut`。

**④ `static mut` 是历史包袱，不要用**

```rust
static mut COUNTER: u32 = 0;
unsafe { COUNTER += 1; }              // ✘ 每次读写都要 unsafe，且并发下 UB
```

Rust 2024 edition 起，创建 `static mut` 的引用已是硬错误。全局可变状态的正确做法：

```rust
use std::sync::atomic::{AtomicU32, Ordering};
use std::sync::{Mutex, OnceLock};

static COUNTER: AtomicU32 = AtomicU32::new(0);        // ⭐ 计数器用原子类型
COUNTER.fetch_add(1, Ordering::Relaxed);

static CACHE: Mutex<Vec<String>> = Mutex::new(Vec::new());   // 复杂数据加锁

static CONFIG: OnceLock<Config> = OnceLock::new();    // ⭐ 只初始化一次的全局配置
CONFIG.get_or_init(|| load_config());
```

**⑤ 未使用变量的警告**

```rust
let count = 10;             // ⚠ warning: unused variable: `count`
let _count = 10;            // ✔ 下划线前缀，明确表示"故意不用"
```

> 工程规范里 CI 一般会开 `-D warnings`，把警告当错误。不要用 `#[allow(unused)]` 一关了之，未使用的变量往往意味着逻辑漏写。

**⑥ `const` 里的堆分配类型是个陷阱**

```rust
const NAMES: Vec<&str> = Vec::new();   // 编译能过，但每次使用都是一个新的空 Vec
```

`const` 会在每个使用处内联展开一份，所以它不适合表达"共享的一份数据"。这种场景要用 `static` 或 `OnceLock`。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 默认不加 `mut` | 编译器提示需要才加，可变范围最小化 |
| 变量 `snake_case`，常量 `SCREAMING_SNAKE_CASE` | 类型/Trait 用 `PascalCase`，与 Java 的小驼峰不同 |
| 魔法值一律提为 `const` | `const MAX_RETRY: u8 = 3;`，不要散落字面量 |
| 全局常量用 `const`，全局状态用 `OnceLock`/原子类型 | 禁用 `static mut` |
| RAII 守卫必须绑定到具名变量 | 写 `let _guard = ...`，绝不写 `let _ = ...` |
| 局部变量少写类型标注 | 靠推断，函数签名才必须写全 |
| 遮蔽用于"同一语义的类型转换" | 不要用遮蔽做多个不相干的变量 |
| 数字字面量用 `_` 分隔 | `86_400`、`1_000_000` |
| 作用域最小化 | 大对象用 `{}` 块或 `drop()` 提前释放 |

---

### 9. 面试重点

**Q1：Rust 变量为什么默认不可变？**

不可变是安全和优化的基础：不会被修改的值可以被安全共享（多线程无需加锁），编译器也能做更激进的优化。同时 `mut` 成为一个显式标记，代码里所有会发生修改的地方一眼可见，降低了心智负担。

**Q2：`mut` 和 shadowing 有什么区别？**

`mut` 是修改同一个变量的值，**类型不能变**；shadowing 是用 `let` 创建一个全新的变量把旧的遮蔽掉，**可以改变类型**，并且受作用域约束，出了块就恢复成外层的值。

**Q3：`const` 和 `static` 的区别？**

`const` 是编译期常量，没有固定内存地址，编译时被内联到每一处使用点；`static` 有唯一的固定地址和 `'static` 生命周期，可以取引用。两者都必须显式标注类型。默认用 `const`，需要唯一地址、大对象或全局状态时才用 `static`。

**Q4：能定义全局可变变量吗？**

语法上有 `static mut`，但读写都需要 `unsafe`，且不做任何并发保护，极易出现数据竞争（UB），Rust 2024 起对它的引用已是硬错误。正确做法是用 `AtomicXxx`（计数器）、`Mutex`/`RwLock`（复杂数据）、`OnceLock`/`LazyLock`（只初始化一次的全局配置）。

**Q5：`let _ = foo();` 和 `let _x = foo();` 有什么区别？**

`_` 不是绑定，值在语句结束时立刻被 drop；`_x` 是正常绑定，只是加下划线来抑制未使用警告，值会活到作用域结束。所以对 `MutexGuard` 这类 RAII 守卫，写成 `let _ = lock()` 等于完全没有加锁。

**Q6：`let x;` 不加 `mut` 为什么后面还能赋值？**

首次初始化不算"修改"。编译器会做控制流分析，保证变量在被读取前的每条路径上都恰好被赋值一次；如果存在可能未初始化的路径，或试图赋第二次值，都会编译报错。

**Q7：变量的析构顺序是怎样的？**

同一作用域内按**声明顺序的逆序**析构（后声明先释放）。这在持锁、事务、文件句柄等依赖释放时机的场景很关键，必要时用 `drop(x)` 或额外的 `{}` 块显式控制。

---

## 二、基本数据类型

### 1. 类型总览

Rust 的基本类型分**标量（scalar）**和**复合（compound）**两类，本章只讲标量，复合类型见第七章。

| 分类 | 类型 | 说明 |
| ---- | ---- | ---- |
| 整型 | `i8` `i16` ⭐`i32` `i64` `i128` `isize` | 有符号 |
| | `u8` `u16` `u32` `u64` `u128` ⭐`usize` | 无符号 |
| 浮点 | `f32` ⭐`f64` | IEEE 754 |
| 布尔 | `bool` | 只有 `true` / `false`，占 1 字节 |
| 字符 | `char` | ⭐ **4 字节**，一个 Unicode 标量值 |
| 单元 | `()` | 零大小，表示"没有值" |
| 永不返回 | `!` | never 类型，`panic!`、`loop` 的类型 |

**默认类型**：整数字面量默认 `i32`，浮点字面量默认 `f64`。`i32` 在绝大多数平台上性能最好，`f64` 精度更高且现代 CPU 上和 `f32` 速度相当。

---

### 2. 整型

| 类型 | 大小 | 取值范围 |
| ---- | ---- | -------- |
| `i8` / `u8` | 1 字节 | -128 ~ 127 / 0 ~ 255 |
| `i16` / `u16` | 2 字节 | -2¹⁵ ~ 2¹⁵-1 / 0 ~ 65535 |
| ⭐`i32` / `u32` | 4 字节 | 约 ±21 亿 / 0 ~ 约 42 亿 |
| `i64` / `u64` | 8 字节 | 约 ±9.2×10¹⁸ |
| `i128` / `u128` | 16 字节 | 极少用，大整数场景 |
| `isize` / ⭐`usize` | **跟随平台指针宽度**（64 位机上 8 字节） | 集合长度、索引专用 |

**`usize` 是必须记住的一个类型**：所有集合的 `len()`、下标索引、`Vec` 容量都是 `usize`。这也是 Rust 新人最常见的类型不匹配来源。

```rust
let v = vec![1, 2, 3];
let n: i32 = 1;
v[n];                       // ✘ 索引必须是 usize
v[n as usize];              // ✔
```

**字面量写法**：

```rust
let a = 42;                 // i32
let b = 42u8;               // ⭐ 后缀直接指定类型
let c = 1_000_000;          // _ 是分隔符，可读性
let d = 0xff;               // 十六进制 255
let e = 0o77;               // 八进制 63
let f = 0b1010_1010;        // 二进制 170
let g = b'A';               // ⭐ 字节字面量，u8 类型，值为 65
```

**每种类型自带常量与方法**：

```rust
i32::MAX;  i32::MIN;  u8::MAX;        // 边界值
i32::from_str_radix("ff", 16);        // 按进制解析
10_i32.pow(3);                        // 1000
(-5_i32).abs();
```

---

### 3. 整数溢出（重点）

这是 Rust 与 Java/C 差别最大的一个点：

| 场景 | Rust 行为 | Java 行为 |
| ---- | --------- | --------- |
| debug 模式 | ⭐ **直接 panic**，程序崩溃 | 静默回绕 |
| release 模式 | 按二进制补码**回绕**（定义好的行为，不是 UB） | 静默回绕 |
| 编译期能算出的常量溢出 | ⭐ **编译错误** | 编译期截断 |

```rust
let a: u8 = 255;
let b = a + 1;              // debug: panic: attempt to add with overflow
                            // release: 0

let c: u8 = 256;            // ✘ 编译错误，字面量超出范围
```

> 这个设计意味着 **debug 下能提前暴露溢出 bug，release 下不会有 UB**。但也意味着 debug 和 release 行为不一致，压测/线上问题排查时要注意。

**⭐ 四组溢出处理方法**（工程代码里处理外部输入必须用）：

| 方法族 | 返回值 | 溢出时行为 | 适用场景 |
| ------ | ------ | ---------- | -------- |
| ⭐`checked_add` | `Option<T>` | 返回 `None` | 需要感知并处理溢出 |
| ⭐`saturating_add` | `T` | 停在 `MAX`/`MIN` | 计数器、进度、剩余量 |
| `wrapping_add` | `T` | 回绕 | 哈希、随机数、密码学 |
| `overflowing_add` | `(T, bool)` | 回绕 + 是否溢出标志 | 需要值也需要标志 |

```rust
let a: u8 = 250;

a.checked_add(10);          // None
a.saturating_add(10);       // 255
a.wrapping_add(10);         // 4
a.overflowing_add(10);      // (4, true)

// 实战：金额累加，避免恶意输入把服务打 panic
let total = items.iter()
    .try_fold(0u64, |acc, it| acc.checked_add(it.price))
    .ok_or("金额溢出")?;
```

`sub`、`mul`、`div`、`pow`、`neg` 都有对应的四个版本。

**除法与取余**：

```rust
7 / 2;                      // 3，整数除法向零截断
-7 / 2;                     // -3
-7 % 2;                     // -1，余数符号跟随被除数
7 / 0;                      // ✘ 编译错误（常量）
a / b;                      // 运行时 b 为 0 → panic: divide by zero

a.checked_div(b);           // ⭐ 除数为 0 返回 None，不 panic
```

---

### 4. 浮点型

```rust
let x = 2.0;                // f64（默认）
let y: f32 = 3.14;
let z = 3.14f32;
```

`f32` 约 7 位有效数字，`f64` 约 15~16 位。**除非有明确的内存/带宽压力（如图形、大规模数组），一律用 `f64`**。

**浮点的三个特殊值 + 精度问题**：

```rust
1.0 / 0.0;                  // inf，浮点除零不 panic
-1.0 / 0.0;                 // -inf
0.0 / 0.0;                  // NaN
f64::NAN == f64::NAN;       // ⭐ false，NaN 不等于自身

0.1 + 0.2 == 0.3;           // false
0.1 + 0.2;                  // 0.30000000000000004

// ✔ 浮点比较用误差范围
(a - b).abs() < f64::EPSILON;
```

**⭐ 浮点没有实现 `Ord`，只有 `PartialOrd`** —— 因为 NaN 破坏了全序关系。直接后果是浮点数组不能用 `sort()`：

```rust
let mut v = vec![3.0, 1.0, 2.0];
v.sort();                                   // ✘ 编译错误：f64 未实现 Ord
v.sort_by(|a, b| a.partial_cmp(b).unwrap());   // ✔ 但有 NaN 会 panic
v.sort_by(f64::total_cmp);                  // ⭐ 最稳妥，NaN 也有确定顺序
```

同理，`f64` 不能直接作 `HashMap` 的 key（未实现 `Eq` / `Hash`）。

> **金额绝不用浮点**。Rust 生态的做法：用 `i64` 存最小货币单位（分），或用 `rust_decimal::Decimal`（对应 Java 的 `BigDecimal`）。

---

### 5. 布尔与字符

**`bool`**：只有 `true` / `false`，占 1 字节。**不能和整数隐式互转**：

```rust
if 1 { }                    // ✘ 编译错误，条件必须是 bool
if x != 0 { }               // ✔

let n = true as i32;        // 1，显式转换可以
```

**`char`**：⭐ **占 4 字节**，存的是一个 **Unicode 标量值**（`U+0000`~`U+D7FF` 和 `U+E000`~`U+10FFFF`，不含代理项）。

```rust
let c = 'A';
let cn = '中';              // ✔ 一个 char
let emoji = '😀';           // ✔ 一个 char —— 注意这点和 Java 完全不同
let heart = '\u{1F600}';    // Unicode 转义

c.is_alphabetic();
c.is_numeric();
c.to_ascii_uppercase();
c as u32;                   // 65，char → 码点
char::from_u32(65);         // Option<char>
```

| 对比 | Rust `char` | Java `char` |
| ---- | ----------- | ----------- |
| 大小 | 4 字节 | 2 字节 |
| 编码 | Unicode 标量值 | UTF-16 代码单元 |
| emoji | ⭐ 1 个 `char` 装得下 | 需要 2 个 `char`（代理对） |

> Rust 的 `char` 设计避免了 Java 的"代理对"问题。但注意 `char` **仍不等于"人眼看到的一个字符"** —— 带组合符号的字形（如 `é` 可能是 `e` + 重音符两个 `char`）、国旗 emoji 依然要多个 `char`，这类需求要用 `unicode-segmentation` 库按字素簇（grapheme cluster）处理。

**字符串不是 `char` 数组**：`String` 底层是 UTF-8 字节，不能按下标取 `char`，详见第六章。

---

### 6. 单元类型 `()` 与 never 类型 `!`

**`()`（unit）**：零大小类型，表示"没有有意义的返回值"，相当于其他语言的 `void`，但它是一个**真实的值和类型**：

```rust
fn log(msg: &str) { }               // 返回类型省略即 ()
fn log(msg: &str) -> () { }         // 等价写法

let x = ();                         // 合法
let r: Result<(), Error> = Ok(());  // ⭐ 极常见：只关心成功/失败，不关心返回值
let set: HashMap<K, ()>;            // 用 () 占位，零内存开销
```

**`!`（never）**：永不返回的类型，可以强制转换成任意类型。`panic!`、`loop {}`、`return`、`std::process::exit()` 的类型都是 `!`：

```rust
let x: i32 = match opt {
    Some(v) => v,
    None => panic!("no value"),     // ⭐ panic 的类型是 !，能匹配 i32 分支
};

fn fatal(msg: &str) -> ! {          // 明确声明"这个函数不会返回"
    eprintln!("{msg}");
    std::process::exit(1);
}
```

---

### 7. 类型别名与 newtype

```rust
type Kilometers = i32;              // ⭐ 只是别名，和 i32 完全等价，无类型安全
let d: Kilometers = 5;
let n: i32 = d;                     // ✔ 可以互相赋值

struct Meters(i32);                 // ⭐ newtype：一个全新的类型
let m = Meters(5);
let n: i32 = m;                     // ✘ 编译错误，类型不同
```

| 用法 | 作用 | 场景 |
| ---- | ---- | ---- |
| `type` 别名 | 缩短冗长类型，**不产生新类型** | ⭐`type Result<T> = std::result::Result<T, MyError>;` |
| newtype | 产生**新类型**，编译期区分 | 区分 `UserId(u64)` 和 `OrderId(u64)`，防止参数传反 |

> newtype 是 Rust 工程中极常用的模式：把 `u64` 包一层成 `UserId`，编译器就能帮你拦住"把订单 ID 传给了要用户 ID 的参数"这类 bug。同时它还是给外部类型实现外部 Trait 的唯一办法（孤儿规则，见第十五章）。

---

### 8. 高频踩坑点（重点）

**① 不同整数类型之间不存在隐式转换**

```rust
let a: i32 = 5;
let b: i64 = a;             // ✘ 编译错误，即使是"小转大"也不行
let b: i64 = a as i64;      // ✔ 必须显式
let b: i64 = a.into();      // ⭐ 更推荐，无损转换用 into
```

> Rust 没有 Java 那种"自动向上转型"。好处是永远不会有意外的隐式转换，代价是要多写 `as`。**无损转换优先用 `into()`，可能有损的才用 `as` 或 `try_into()`**（详见第三章）。

**② `as` 转换会静默截断**

```rust
let a: i32 = 300;
let b = a as u8;            // 44，高位被直接砍掉，不报错不警告

let big: i64 = 10_000_000_000;
let small = big as i32;     // 1410065408，数据已经废了

// ✔ 需要安全转换用 try_into
let small: i32 = big.try_into().map_err(|_| "溢出")?;
```

**③ 浮点转整数是饱和转换，不是截断**

```rust
let x = 3.9_f64 as i32;         // 3，向零截断
let x = -3.9_f64 as i32;        // -3
let x = 1e20_f64 as i32;        // ⭐ i32::MAX，饱和而非回绕（Rust 1.45+）
let x = f64::NAN as i32;        // ⭐ 0
```

**④ debug 和 release 的溢出行为不一致**

本地 debug 测试正常（溢出会 panic 被发现），线上 release 却静默回绕出错误数据。**处理外部输入的算术必须用 `checked_*` / `saturating_*`**，不要依赖 debug 的 panic 兜底。

也可以在 `Cargo.toml` 里让 release 也保持 panic：

```toml
[profile.release]
overflow-checks = true          # 牺牲一点性能，换取线上溢出可见
```

**⑤ `usize` 相减会溢出 panic**

```rust
let v: Vec<i32> = vec![];
let last = v.len() - 1;         // ✘ 0 - 1 在 usize 上溢出 → panic

// ✔ 正确写法
let last = v.len().checked_sub(1);          // Option<usize>
let last = v.len().saturating_sub(1);       // 0
if !v.is_empty() { let last = v.len() - 1; }
```

> 这是 Rust 新人最常遇到的运行时 panic 之一，**凡是对 `len()` 做减法都要先判空**。

**⑥ 浮点作 key、作排序键**

`f64` 没有 `Ord`/`Eq`/`Hash`，不能进 `HashMap` 的 key，`sort()` 也用不了。要么改用整数（比如放大 100 倍存分），要么用 `ordered_float` crate。

**⑦ 数值类型在字面量上会"传染推断"**

```rust
let a = 1;                  // 此处还不确定类型
let b: u8 = 200;
let c = a + b;              // ⭐ a 被推断为 u8
let d = a + 1000;           // ✘ 此时 a 已经是 u8，1000 超范围
```

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 默认用 `i32`；长度/索引用 `usize` | 不要为了"省内存"随便选 `i8`/`u16` |
| ID、时间戳用 `i64`/`u64` | 雪花 ID、毫秒时间戳会超 `i32` |
| 金额禁用浮点 | 用 `i64` 存"分"，或 `rust_decimal::Decimal` |
| 外部输入的算术用 `checked_*` | 防止恶意数据把服务打 panic |
| `len()` 做减法前先判空 | 或用 `saturating_sub` / `checked_sub` |
| 无损转换用 `into()`，有损用 `try_into()` | `as` 只在明确不会越界时用 |
| 浮点比较用 `EPSILON`，排序用 `total_cmp` | 不要写 `a == b` |
| release 建议开 `overflow-checks` | 金融/计费类服务尤其应该开 |
| 语义不同的同底层类型用 newtype 区分 | `UserId(u64)` vs `OrderId(u64)` |
| 长类型用 `type` 别名 | `type Result<T> = std::result::Result<T, AppError>;` |

---

### 10. 面试重点

**Q1：Rust 整数溢出会发生什么？**

debug 模式下直接 panic；release 模式下按二进制补码回绕 —— 注意这是**定义明确的行为，不是 UB**（这点和 C/C++ 的有符号溢出不同）。编译期能算出的常量溢出直接编译报错。生产代码应显式使用 `checked_*`、`saturating_*`、`wrapping_*` 表达意图。

**Q2：`usize` 是什么？为什么要单独有这个类型？**

跟随平台指针宽度的无符号整数（64 位机上 8 字节）。所有集合的长度和索引都是 `usize`，因为它保证能表示内存中任何对象的大小和任何有效索引。注意 `usize` 是无符号的，`len() - 1` 在空集合上会溢出 panic。

**Q3：Rust 的 `char` 和 Java 的 `char` 有什么区别？**

Rust 的 `char` 占 4 字节，存一个 Unicode 标量值，emoji 也是一个 `char`；Java 的 `char` 占 2 字节，是一个 UTF-16 代码单元，emoji 需要代理对（2 个 char）。但 Rust 的 `char` 仍不等于"人眼看到的一个字符"，字素簇需要专门的库处理。

**Q4：为什么 `f64` 不能用 `sort()`？**

`sort()` 要求元素实现 `Ord`（全序），而 `f64` 只实现了 `PartialOrd`（偏序）—— 因为 `NaN` 与任何值（包括自身）比较都返回 `false`，破坏了全序。实际排序用 `sort_by(f64::total_cmp)`。同理 `f64` 也不能作 `HashMap` 的 key。

**Q5：`as` 转换安全吗？**

不安全。`as` 在整数间会静默截断高位，不报错也不警告；浮点转整数则是饱和转换（超范围停在 `MAX`/`MIN`，`NaN` 转成 0）。安全做法：无损转换用 `From`/`Into`，可能有损的用 `TryFrom`/`try_into()` 显式处理失败。

**Q6：`()` 和 `!` 分别是什么？**

`()` 是单元类型，零大小，表示"没有有意义的值"，是一个真实存在的类型和值，常见于 `Result<(), E>`；`!` 是 never 类型，表示"永不返回"，`panic!`、`loop {}`、`process::exit()` 的类型都是 `!`，它可以强制转换为任意类型，所以能出现在 `match` 的任意分支里。

**Q7：`type` 别名和 newtype 有什么区别？**

`type` 只是换个名字，和原类型完全等价，可以互相赋值，纯粹为了缩短冗长的类型名；newtype（元组结构体包一层）会产生一个**全新的类型**，编译器能区分，用来防止语义混淆（`UserId` vs `OrderId`），也是绕过孤儿规则给外部类型实现外部 Trait 的标准手段。

---

## 三、运算符与类型转换

### 1. 运算符总览

| 分类 | 运算符 | 说明 |
| ---- | ------ | ---- |
| 算术 | `+` `-` `*` `/` `%` | ⭐ **没有 `++` / `--`** |
| 赋值 | `=` `+=` `-=` `*=` `/=` `%=` 等 | ⭐ 赋值是语句，值为 `()` |
| 比较 | `==` `!=` `>` `<` `>=` `<=` | 需要 `PartialEq` / `PartialOrd` |
| 逻辑 | `&&` `\|\|` `!` | 短路；`!` 对 `bool` 是逻辑非 |
| 位运算 | `&` `\|` `^` `!` `<<` `>>` | ⭐ 取反是 `!` 不是 `~`，且**没有 `>>>`** |
| 类型转换 | `as` | 只用于原始类型 |
| 引用 | `&` `&mut` `*` | 借用与解引用 |
| 区间 | `..` `..=` | `0..5`（左闭右开）、`0..=5`（全闭） |
| 错误传播 | `?` | 见第十三章 |

**Rust 没有的运算符**：`++` / `--`（易与所有权/借用产生歧义）、`~`（取反统一用 `!`）、`>>>`（有符号数天然算术右移）、三元 `? :`（用 `if` 表达式代替）。

```rust
i += 1;                             // ✔ 代替 i++
let x = if a > b { a } else { b };  // ✔ 代替三元运算符
```

---

### 2. 算术运算符

**基本规则和第二章一致**：整数除法向零截断、除零 panic、溢出在 debug 下 panic。

```rust
7 / 2;              // 3
-7 % 2;             // -1，余数符号跟随被除数
2.0_f64.powi(10);   // 幂运算没有 ** 运算符，用方法
10_i32.pow(3);
```

**⭐ 不同类型不能直接运算**，这是与 Java 差别最大的一点：

```rust
let a: i32 = 1;
let b: i64 = 2;
a + b;                  // ✘ 编译错误：类型不匹配，没有自动提升
a as i64 + b;           // ✔

let n: usize = 3;
let m: i32 = 2;
n - m as usize;         // 必须手动统一类型
```

> Rust 不做任何隐式数值提升。看似麻烦，但彻底杜绝了 Java 里 `1000 * 60 * 60 * 24 * 30` 先按 `int` 溢出再赋给 `long` 那类静默 bug。

**赋值是语句不是表达式**，其值为 `()`：

```rust
let a = (b = 5);        // a 的类型是 ()，不是 5
if x = 5 { }            // ✘ 编译错误：期望 bool，得到 ()
                        //    C/Java 里 = 和 == 写混的经典 bug 在 Rust 里编译不过
```

**运算符可以作用在引用上**：

```rust
let a = &1;
let b = &2;
let c = a + b;          // ✔ 3，标准库为 &i32 也实现了 Add
```

---

### 3. 比较运算符

**⭐ Rust 的 `==` 比较的是"值"，不是"地址"** —— 这是 Java 程序员最容易搞反的一点：

```rust
let s1 = String::from("abc");
let s2 = String::from("abc");
s1 == s2;                       // ⭐ true！Rust 里 == 就是内容比较
                                //    Java 里这写法是 false（比地址）

let r1 = &s1;
let r2 = &s2;
r1 == r2;                       // ⭐ 仍然 true，引用比较会自动比较所指的值

std::ptr::eq(r1, r2);           // false，要比地址必须显式用 ptr::eq
```

Rust 没有"引用相等"这个默认语义，`==` 由 `PartialEq` trait 定义，等价于 Java 的 `equals()`。

**四个比较相关的 Trait**：

| Trait | 提供 | 说明 |
| ----- | ---- | ---- |
| `PartialEq` | `==` `!=` | 部分等价关系，允许"自身不等于自身"（`NaN`） |
| `Eq` | — | 标记 trait，表示是完全等价关系。`f64` 没有 |
| `PartialOrd` | `<` `>` `<=` `>=` | 偏序，返回 `Option<Ordering>` |
| `Ord` | `cmp()` | 全序，排序、`BTreeMap` 的 key 要求它。`f64` 没有 |

```rust
#[derive(PartialEq, Eq, PartialOrd, Ord)]    // ⭐ 一般直接 derive
struct Version { major: u32, minor: u32 }    // 按字段声明顺序逐个比较
```

**⭐ 比较运算不能链式**：

```rust
if 1 < x < 10 { }               // ✘ 编译错误
if 1 < x && x < 10 { }          // ✔
if (1..10).contains(&x) { }     // ⭐ 更地道
```

---

### 4. 逻辑与位运算

**逻辑运算**：`&&`、`||` 短路，`!` 取反。**操作数必须是 `bool`**，整数、`Option` 都不行。

```rust
if user.is_some() && user.unwrap().age > 18 { }     // 短路保证右边安全
if list.is_empty() || list[0] == 0 { }
```

**位运算**：

| 运算符 | 含义 | 示例 |
| ------ | ---- | ---- |
| `&` | 按位与 | `6 & 3` → `2` |
| `\|` | 按位或 | `6 \| 3` → `7` |
| `^` | 按位异或 | `6 ^ 3` → `5` |
| `!` | ⭐ 按位取反（不是 `~`） | `!6u8` → `249` |
| `<<` | 左移 | `6 << 1` → `12` |
| `>>` | 右移：有符号补符号位，无符号补 0 | `-8 >> 1` → `-4`；`8u8 >> 1` → `4` |

Rust 不需要 `>>>`：类型本身就带符号信息，`u32` 的 `>>` 天然是逻辑右移。

**⭐ 移位量超过位宽会 panic**（debug）/ 取模（release）：

```rust
let x: u8 = 1;
x << 8;                 // debug: panic: attempt to shift left with overflow
x.checked_shl(8);       // ✔ None
x.wrapping_shl(8);      // 回绕：等价于 x << (8 % 8) = x << 0 = 1
```

**位标志的常见用法**：

```rust
const READ:  u8 = 1 << 0;
const WRITE: u8 = 1 << 1;
const EXEC:  u8 = 1 << 2;

let mut perm = READ | WRITE;
perm & WRITE != 0;      // 判断是否有写权限
perm &= !WRITE;         // 移除写权限（注意是 ! 不是 ~）
```

> 生产项目里位标志建议直接用 `bitflags` crate，它把这套写法封装成了类型安全的结构体。

**⭐ Rust 修正了 C/Java 的优先级坑**：`&` `^` `|` 的优先级**高于**比较运算符。

```rust
if a & b == 0 { }       // ⭐ Rust 解析为 (a & b) == 0，符合直觉
                        //    C/Java 解析为 a & (b == 0)，是编译错误
```

**完整优先级**（从高到低）：方法调用/字段访问 → `?` → 一元 `-` `!` `*` `&` → `as` → `*` `/` `%` → `+` `-` → `<<` `>>` → `&` → `^` → `|` → 比较 → `&&` → `||` → `..` → `=` 及复合赋值。

---

### 5. 运算符重载

Rust 的运算符本质是 `std::ops` 下的 Trait，实现对应 Trait 即可重载：

| 运算符 | Trait | 运算符 | Trait |
| ------ | ----- | ------ | ----- |
| `+` | `Add` | `+=` | `AddAssign` |
| `-` | `Sub` | `-` (取负) | `Neg` |
| `*` `/` `%` | `Mul` `Div` `Rem` | `!` | `Not` |
| `[]` | `Index` / `IndexMut` | `*` (解引用) | `Deref` / `DerefMut` |
| `==` | `PartialEq` | `<` `>` | `PartialOrd` |

```rust
use std::ops::Add;

#[derive(Debug, Clone, Copy)]
struct Point { x: i32, y: i32 }

impl Add for Point {
    type Output = Point;                        // ⭐ 关联类型，指定返回类型
    fn add(self, rhs: Point) -> Point {
        Point { x: self.x + rhs.x, y: self.y + rhs.y }
    }
}

let p = Point { x: 1, y: 2 } + Point { x: 3, y: 4 };   // Point { x: 4, y: 6 }
```

> 注意 `Add::add` 的签名是 `fn add(self, ...)` —— **会消耗所有权**。所以自定义类型要么 `#[derive(Clone, Copy)]`，要么额外为 `&T` 实现 `Add`。这也是 `String + &str` 会消耗左侧 `String` 的原因（见第 7 节）。

---

### 6. 类型转换体系（重点）

Rust 的类型转换有明确的分工，选错方式是新人代码"不地道"的主要表现：

| 方式 | 用途 | 失败处理 | 推荐度 |
| ---- | ---- | -------- | ------ |
| ⭐`From` / `Into` | 无损转换 | 不会失败 | 首选 |
| ⭐`TryFrom` / `TryInto` | 可能失败的转换 | 返回 `Result` | 首选 |
| `as` | 原始类型之间 | ⭐ **静默截断** | 明确安全时才用 |
| `parse()` | 字符串 → 任意类型 | 返回 `Result` | 字符串专用 |
| `to_string()` | 任意类型 → `String` | 不会失败 | 需实现 `Display` |
| `AsRef` / `Deref` | 引用之间的轻量转换 | 不会失败 | 写通用函数用 |

**① `From` / `Into`：无损转换的标准做法**

⭐ **只需实现 `From`，`Into` 会自动获得**（标准库有全局实现 `impl<T, U: From<T>> Into<U> for T`）：

```rust
let a: i32 = 5;
let b: i64 = a.into();              // ⭐ 无损，推荐
let b = i64::from(a);               // 等价

let s: String = "hello".into();     // &str → String
let s = String::from("hello");      // 等价，更常见

// 自定义类型
struct Celsius(f64);
struct Fahrenheit(f64);

impl From<Celsius> for Fahrenheit {
    fn from(c: Celsius) -> Self {
        Fahrenheit(c.0 * 9.0 / 5.0 + 32.0)
    }
}

let f: Fahrenheit = Celsius(100.0).into();
```

> `From` 在错误处理中作用巨大：为自定义错误实现 `From<io::Error>` 后，`?` 运算符就能自动完成错误类型转换（第十三章）。

**② `TryFrom` / `TryInto`：可能失败的转换**

```rust
use std::convert::TryInto;

let big: i64 = 300;
let small: u8 = big.try_into().unwrap();        // ✘ 运行时 Err → panic

let small: Result<u8, _> = big.try_into();      // ✔ 显式处理
match small {
    Ok(v)  => println!("{v}"),
    Err(e) => println!("超出范围: {e}"),
}

let n: u8 = big.try_into().map_err(|_| "数值溢出")?;    // ⭐ 工程写法
```

**③ `as`：只用于原始类型**

```rust
let a = 3.9_f64 as i32;         // 3
let b = 300_i32 as u8;          // ⭐ 44，静默截断
let c = 'A' as u32;             // 65
let d = 65_u8 as char;          // 'A'，⭐ 只有 u8 能 as char
let e = true as i32;            // 1
```

`as` 的转换规则汇总：

| 转换 | 行为 |
| ---- | ---- |
| 整数 → 更小整数 | ⭐ 截断高位，不报错 |
| 整数 → 更大整数 | 有符号补符号位，无符号补 0 |
| 浮点 → 整数 | 向零截断；超范围**饱和**到 `MAX`/`MIN`；`NaN` → `0` |
| 整数 → 浮点 | 可能丢精度（如 `i64` → `f32`） |
| `char` → 整数 | 取 Unicode 码点 |
| `u8` → `char` | 合法；其他整数类型不行，要用 `char::from_u32` |
| `bool` → 整数 | `false` → 0，`true` → 1 |

**④ 字符串与其他类型互转**

```rust
// 字符串 → 数值：parse()，依赖 FromStr trait
let n: i32 = "42".parse()?;
let n = "42".parse::<i32>()?;               // turbofish 写法
let n = "abc".parse::<i32>();               // Err(ParseIntError)

// 任意类型 → String：依赖 Display trait
let s = 42.to_string();
let s = format!("{}-{}", a, b);             // ⭐ 拼接首选
let s = format!("{a:?}");                   // Debug 格式

// String ↔ &str
let s: String = String::from("hi");
let r: &str = &s;                           // ⭐ Deref 强制转换，自动发生
let r: &str = s.as_str();                   // 等价，显式写法
```

**⑤ Deref 强制转换（隐式，唯一的"自动转换"）**

Rust 只有两种隐式转换，其中最常见的是 **Deref coercion**：

```rust
&String   → &str
&Vec<T>   → &[T]
&Box<T>   → &T
&String   → &str → &[u8]（多级也能自动发生）
```

```rust
fn print(s: &str) { println!("{s}"); }

let s = String::from("hi");
print(&s);              // ⭐ &String 自动转成 &str，不用写 s.as_str()
```

> ⭐ **工程规范**：函数参数优先用 `&str` 而非 `&String`、用 `&[T]` 而非 `&Vec<T>` —— 前者能同时接受两种类型，适用面更广。

---

### 7. 高频踩坑点（重点）

**① `String + &str` 会消耗左侧的所有权**

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world");
let s3 = s1 + &s2;              // ⭐ s1 被移走，之后不能再用
println!("{s1}");               // ✘ 编译错误：value borrowed after move

let s3 = format!("{s1}{s2}");   // ✔ 两个都还能用
```

原因是 `Add for String` 的签名是 `fn add(self, other: &str) -> String`，`self` 按值传入。**拼接优先用 `format!`**，循环内拼接用 `push_str`。

**② `as` 静默截断，别用来做"安全转换"**

```rust
let len = data.len();               // usize
let n = len as i32;                 // ✘ 数据量大时会变成负数
let n: i32 = len.try_into()?;       // ✔
```

**③ `%` 对负数的结果是"余数"不是"模"**

```rust
-7 % 3;                     // -1（余数，符号跟被除数）
(-7_i32).rem_euclid(3);     // ⭐ 2（欧几里得取模，结果恒非负）
```

> 做环形索引、哈希分桶时必须用 `rem_euclid`，否则负数会算出负下标直接 panic。

**④ `!` 对整数是按位取反，容易和逻辑非混**

```rust
let flag = true;
!flag;                      // false，逻辑非

let n: u8 = 0b0000_0001;
!n;                         // 0b1111_1110 = 254，按位取反
if !n { }                   // ✘ 编译错误：期望 bool
```

**⑤ 整数除法在浮点场景下的意外截断**

```rust
let rate = 1 / 3;                       // 0，两边都是整数
let rate = 1.0 / 3.0;                   // ✔ 0.333...
let rate = a as f64 / b as f64;         // ✔ 计算百分比的正确写法
```

**⑥ `into()` 有时推断不出目标类型**

```rust
let x = 5_i32.into();               // ✘ 编译错误：type annotations needed
let x: i64 = 5_i32.into();          // ✔ 标注目标类型
let x = i64::from(5_i32);           // ✔ 或用 From 显式写
```

**⑦ 比较浮点和跨类型比较**

```rust
let a: i32 = 5;
let b: i64 = 5;
a == b;                             // ✘ 编译错误，类型必须一致
a as i64 == b;                      // ✔
```

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 无损转换用 `into()` / `from()` | 语义清晰，编译器保证不丢数据 |
| 可能失败的转换用 `try_into()` | 显式处理 `Result`，不要 `unwrap()` |
| `as` 只在明确不越界时用 | 尤其禁止 `usize as i32` 这类 |
| 字符串拼接用 `format!` | `+` 会移走左侧所有权 |
| 函数参数用 `&str` / `&[T]` | 而非 `&String` / `&Vec<T>`，适用面更广 |
| 自定义错误实现 `From` | 让 `?` 自动完成错误转换 |
| 取模用 `rem_euclid` | 涉及负数时 `%` 会给出负结果 |
| 复杂表达式加括号 | 虽然 Rust 优先级更符合直觉，仍不依赖记忆 |
| 位标志用 `bitflags` crate | 比手写常量更类型安全 |
| 判断区间用 `(a..b).contains(&x)` | 比较不能链式写 |

---

### 9. 面试重点

**Q1：Rust 的 `==` 比较的是值还是地址？**

**值**。`==` 由 `PartialEq` trait 定义，语义上等价于 Java 的 `equals()`；对引用做 `==` 也会自动比较所指向的值。要比较指针地址必须显式用 `std::ptr::eq()`。Rust 里不存在 Java 那种 `==` 比地址的陷阱。

**Q2：`PartialEq` 和 `Eq`、`PartialOrd` 和 `Ord` 的区别？**

`PartialEq`/`PartialOrd` 表示部分等价/偏序关系，允许存在无法比较的值；`Eq`/`Ord` 是更强的完全等价/全序约束，本身不提供额外方法（`Eq` 是纯标记 trait）。典型区别就是 `f64`：因为 `NaN != NaN`、且 `NaN` 与任何值都无法排序，所以 `f64` 只实现了 `PartialEq`/`PartialOrd`，不能用于 `sort()`、`BTreeMap` 的 key。

**Q3：Rust 为什么没有 `++` 和 `--`？**

一是它们的前缀/后缀语义容易写出难懂的代码，二是会与 Rust 的所有权和借用检查产生歧义（自增本质是"读 + 写"，需要可变借用），设计上用 `i += 1` 表达同样的意思且没有歧义。

**Q4：`as`、`From/Into`、`TryFrom/TryInto` 该怎么选？**

无损转换用 `From`/`Into`（如 `i32 → i64`），编译器保证不丢数据；可能失败的转换用 `TryFrom`/`TryInto`，返回 `Result` 强制处理；`as` 只用于原始类型且会**静默截断**，仅在明确不会越界或确实需要截断语义时用。只实现 `From` 就能自动获得 `Into`。

**Q5：什么是 Deref 强制转换？**

Rust 唯二的隐式转换之一：当传参类型不匹配时，编译器会自动对引用做 `Deref`，把 `&String` 转成 `&str`、`&Vec<T>` 转成 `&[T]`、`&Box<T>` 转成 `&T`，并且可以多级连续转换。它让 `fn f(s: &str)` 可以直接接受 `&String`，因此函数参数应优先声明为 `&str` / `&[T]`。

**Q6：`let s3 = s1 + &s2;` 之后 `s1` 还能用吗？**

不能。`impl Add for String` 的签名是 `fn add(self, other: &str) -> String`，`self` 按值传入，所以 `s1` 的所有权被移走了。要保留两者用 `format!("{s1}{s2}")`，循环内拼接用 `push_str`。

**Q7：Rust 位运算符的优先级和 C/Java 有什么不同？**

Rust 里 `&`、`^`、`|` 的优先级**高于**比较运算符，所以 `a & b == 0` 会正确解析成 `(a & b) == 0`；而 C/Java 里 `&` 低于 `==`，同样的写法会解析成 `a & (b == 0)`。Rust 修正了这个历史遗留的优先级设计问题。

---

## 四、流程控制（分支、循环、match）

### 1. 一切皆表达式

⭐ **理解 Rust 流程控制的前提：`if`、`match`、`loop`、`{}` 块都是表达式，都有返回值。**

```rust
// 块表达式：最后一个"不带分号"的表达式就是它的值
let y = {
    let x = 3;
    x * 2               // ⭐ 没有分号 → 这是返回值
};                      // y = 6

let y = {
    let x = 3;
    x * 2;              // ⭐ 加了分号 → 变成语句，块的值是 ()
};                      // y = ()
```

| 概念 | 说明 |
| ---- | ---- |
| 表达式（expression） | 求值产生一个值，**不带分号** |
| 语句（statement） | 执行动作，不产生值，**带分号**；值为 `()` |

**函数返回值同理**：最后一个表达式不加分号即为返回值，`return` 只在提前返回时用。

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b               // ⭐ 地道写法，不写 return
}

fn add(a: i32, b: i32) -> i32 {
    a + b;              // ✘ 编译错误：加了分号，返回 () 而非 i32
}
```

> **新人最高频的编译错误就是多写/少写一个分号**。报错 `expected i32, found ()` 基本都是这个原因。

---

### 2. if 分支

```rust
if score >= 90 {
    "A"
} else if score >= 60 {
    "B"
} else {
    "C"
};
```

三条硬性规则：

```rust
if (x > 5) { }          // 条件不需要括号（写了会有 warning）
if x { }                // ✘ 条件必须是 bool，整数/Option 都不行
if x > 5 doA();         // ✘ 大括号不能省略，Rust 强制要求
```

**⭐ `if` 作为表达式时，所有分支类型必须一致，且必须有 `else`**：

```rust
let level = if score >= 60 { "pass" } else { "fail" };      // ✔

let x = if cond { 1 };              // ✘ 缺 else，另一条路径是 () 与 i32 冲突
let x = if cond { 1 } else { "a" }; // ✘ 分支类型不一致
```

Rust 没有三元运算符，`if` 表达式就是它的替代品，而且可读性更好。

**卫语句（提前返回）** 同样是 Rust 的主流风格，配合 `?` 运算符能让主流程始终在最外层：

```rust
fn handle(order: &Order) -> Result<(), Error> {
    if order.status != Status::Paid { return Ok(()); }
    if order.amount <= 0 { return Err(Error::InvalidAmount); }

    do_business(order)
}
```

---

### 3. if let 与 let else

**`if let`**：只关心一种匹配情况时，比 `match` 简洁 —— 日常代码里出现频率极高：

```rust
// 啰嗦写法
match config.port {
    Some(p) => println!("端口 {p}"),
    None => {}
}

// ⭐ if let
if let Some(p) = config.port {
    println!("端口 {p}");
}

// 可以带 else
if let Some(p) = config.port {
    println!("端口 {p}");
} else {
    println!("使用默认端口");
}
```

**⭐ `let else`（Rust 1.65+）**：匹配失败就走 `else` 分支，且 `else` 分支**必须发散**（`return`/`break`/`continue`/`panic!`）。它是消除嵌套的利器：

```rust
// ✘ if let 会把主流程推进嵌套
fn process(input: &str) {
    if let Ok(n) = input.parse::<i32>() {
        if let Some(user) = find_user(n) {
            do_something(user);
        }
    }
}

// ✔ let else：主流程保持在最外层，n 和 user 在后续作用域直接可用
fn process(input: &str) {
    let Ok(n) = input.parse::<i32>() else { return; };
    let Some(user) = find_user(n) else { return; };

    do_something(user);
}
```

**`let` 链（Rust 1.88+，edition 2024）**：`if let` 可以用 `&&` 串联：

```rust
if let Some(u) = user && u.age > 18 && let Some(e) = u.email {
    send(e);
}
```

---

### 4. 三种循环

| 循环 | 适用场景 | 特点 |
| ---- | -------- | ---- |
| ⭐`for` | 遍历迭代器/集合/区间 | 绝大多数场景用它 |
| `while` | 条件驱动，次数不定 | 先判断后执行 |
| ⭐`loop` | 无限循环 / 需要 `break` 带值 | 类型是 `!`，编译器知道它不会正常结束 |

```rust
for i in 0..5 { }               // 0,1,2,3,4（左闭右开）
for i in 0..=5 { }              // 0,1,2,3,4,5（全闭）
for i in (0..5).rev() { }       // 倒序
for i in (0..10).step_by(2) { } // 0,2,4,6,8

for item in &vec { }            // ⭐ 借用遍历，vec 之后还能用
for item in &mut vec { }        // 可变借用，可以修改元素
for item in vec { }             // ⭐ 消耗所有权，vec 之后不可用

for (i, item) in vec.iter().enumerate() { }     // ⭐ 带索引
for (k, v) in &map { }                          // 遍历 Map

let mut n = 0;
while n < 5 { n += 1; }

loop {
    if done { break; }
}
```

**⭐ Rust 没有 `do-while`**，用 `loop` + 末尾判断代替：

```rust
loop {
    do_work();
    if !should_continue() { break; }
}
```

**⭐ 无限循环一律用 `loop` 而不是 `while true`**：`loop` 的类型是 `!`，编译器知道它不会正常退出，因而能做更好的类型推断和可达性分析。

```rust
let x = loop { break 5; };       // ✔ x = 5
let x = while true { break 5; }; // ✘ while 不能 break 带值
```

---

### 5. break 带值与循环标签

**`loop` 可以用 `break` 返回值** —— "重试直到成功"的标准写法：

```rust
let result = loop {
    match try_connect() {
        Ok(conn) => break conn,             // ⭐ break 带值
        Err(_) => {
            retries += 1;
            if retries > 3 { break_with_error(); }
        }
    }
};
```

**循环标签**：跳出多层循环，比设 flag 变量清晰得多：

```rust
'outer: for i in 0..n {
    for j in 0..m {
        if matrix[i][j] == target {
            break 'outer;               // ⭐ 直接跳出外层
        }
        if skip(j) { continue 'outer; }
    }
}

// 标签还能带值
let found = 'search: loop {
    for x in &data {
        if x.is_valid() { break 'search Some(x); }
    }
    break None;
};
```

> 标签也可以用在普通块上（`'block: { ... break 'block value; }`），相当于其他语言的 `goto end`，但作用域受限、安全可控。

---

### 6. while let

**循环版的 `if let`**：模式匹配成功就继续循环，失败就退出。处理栈、队列、流式数据的标准写法：

```rust
let mut stack = vec![1, 2, 3];
while let Some(top) = stack.pop() {         // ⭐ pop 返回 None 时自动结束
    println!("{top}");
}

while let Some(line) = reader.next_line().await? {
    handle(line);
}
```

---

### 7. match：Rust 的核心控制流

`match` 相当于加强版的 `switch`，但有三个本质区别：**必须穷尽、是表达式、支持模式解构**。

```rust
let desc = match status {
    Status::Paid     => "已支付",
    Status::Refunded => "已退款",
    Status::Pending  => "待支付",
};                                  // ⭐ 枚举全部覆盖，不需要 _
```

**⭐ 穷尽性检查（Exhaustiveness）是 `match` 最大的价值**：漏掉任何一种情况都编译不过。后续给枚举加新成员时，编译器会把所有需要更新的 `match` 全部报出来 —— 这是 Rust 重构敢下手的底气，Java 的 `switch` 做不到。

**各种匹配写法**：

```rust
let msg = match n {
    0            => "零",
    1 | 2 | 3    => "小",                       // ⭐ 多值用 |
    4..=9        => "中",                       // ⭐ 范围（必须用 ..=）
    x if x < 0   => "负数",                     // ⭐ 匹配守卫 if
    x @ 10..=99  => "两位数",                   // ⭐ @ 绑定：既限定范围又拿到值
    _            => "其他",                     // ⭐ 通配符，必须放最后
};

// 解构
match point {
    Point { x: 0, y: 0 } => "原点",
    Point { x, y: 0 }    => "在 X 轴上",
    Point { x, y }       => "其他位置",
};

// 处理 Result / Option
match result {
    Ok(v)  => println!("成功: {v}"),
    Err(e) => eprintln!("失败: {e}"),
}
```

**多行分支用 `{}`**，`match` 分支之间**不会穿透**，也不需要 `break`：

```rust
let n = match level {
    "A" => 100,
    _ => {
        warn!("未知等级 {level}");
        0                           // ⭐ 块的最后一个表达式就是分支的值
    }
};
```

**⭐ `matches!` 宏**：只需要一个 `bool` 结果时的简写：

```rust
if matches!(status, Status::Paid | Status::Shipped) { }

// 等价于
if match status { Status::Paid | Status::Shipped => true, _ => false } { }
```

> 模式匹配的完整能力（嵌套解构、引用模式、绑定语义等）见第十二章，这里只覆盖它作为控制流的部分。

---

### 8. 高频踩坑点（重点）

**① `for x in vec` 会移走所有权**

```rust
let v = vec![1, 2, 3];
for x in v { }                  // ⭐ v 被移动（IntoIterator）
println!("{:?}", v);            // ✘ 编译错误：value borrowed after move

for x in &v { }                 // ✔ 借用，v 还能用
for x in v.iter() { }           // ✔ 等价
```

| 写法 | 迭代出的类型 | 集合归属 |
| ---- | ------------ | -------- |
| `for x in v` / `v.into_iter()` | `T` | ⭐ 被消耗 |
| `for x in &v` / `v.iter()` | `&T` | 保留 |
| `for x in &mut v` / `v.iter_mut()` | `&mut T` | 保留，可修改 |

**② 分支类型不一致 / 多写分号**

```rust
let x = if cond { 1 } else { 2; };   // ✘ else 分支带分号，类型是 ()

let x = match n {
    0 => "zero",
    _ => 1,                          // ✘ 分支类型不一致
};
```

**③ 遍历时修改集合 —— Rust 在编译期就拦住**

```rust
let mut v = vec![1, 2, 3];
for x in &v {
    v.push(*x);                 // ✘ 编译错误：已被不可变借用，不能可变借用
}

// ✔ 正确做法
v.retain(|x| *x != 2);                          // 条件删除
let extra: Vec<_> = v.iter().map(|x| x * 2).collect();
v.extend(extra);                                // 先收集再修改
```

> 这就是 Rust 相对 Java 的直接优势：Java 的 `ConcurrentModificationException` 是**运行时**才抛出的异常，Rust 的借用检查在**编译期**就杜绝了这类问题。

**④ 通配符 `_` 会掩盖新增的枚举成员**

```rust
match status {
    Status::Paid => handle_paid(),
    _ => {}                     // ⚠ 以后给 Status 加了新成员，这里会静默走到 _
}
```

> **工程规范：对自己定义的、会演进的枚举，尽量列全所有分支，不写 `_`**，让编译器在加新成员时替你找出所有遗漏点。`_` 留给外部类型或确实无穷的情况（如整数匹配）。

**⑤ 范围模式必须用 `..=`**

```rust
match n {
    1..5 => "",                 // ⚠ 早期版本不支持，语义也易混
    1..=4 => "",                // ✔ 模式里用全闭区间
}
```

注意区分：`for i in 0..5` 用的是区间**表达式**（左闭右开），`match` 里的 `1..=4` 是区间**模式**。

**⑥ `while let` 里的借用不能长期持有**

```rust
while let Some(x) = map.get(&key) {     // ✘ map 一直被借用，循环内不能改 map
    map.remove(&key);
}
```

**⑦ 循环变量的作用域**

```rust
for i in 0..5 { }
println!("{i}");                // ✘ i 出了循环就不存在了

let mut i = 0;
while i < 5 { i += 1; }
println!("{i}");                // ✔ 但污染了外层作用域
```

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 优先用 `for` + 迭代器 | 而不是下标循环，无越界风险且更快（省去边界检查） |
| 遍历默认写 `&v` | 需要消耗所有权时才写 `for x in v` |
| 无限循环用 `loop` | 不写 `while true` |
| 单分支匹配用 `if let` | 不要为一种情况写完整 `match` |
| 提前返回用 `let else` | 消除 `if let` 嵌套，主流程留在最外层 |
| 自定义枚举的 `match` 列全分支 | 不写 `_`，让编译器守住新增成员 |
| 布尔判断用 `matches!` | 比手写 `match ... => true, _ => false` 清晰 |
| 多层循环用标签 | 不设 `flag` 变量 |
| 嵌套不超过两层 | 更深就抽函数或用 `?` / `let else` 拍平 |
| 善用表达式返回值 | `let x = if/match {...}`，避免先声明再赋值 |

---

### 10. 面试重点

**Q1：Rust 里表达式和语句的区别？**

表达式求值产生一个值（不带分号），语句执行动作不产生值（带分号，值为 `()`）。`if`、`match`、`loop`、`{}` 块在 Rust 里都是表达式，可以直接赋值给变量。函数最后一个不带分号的表达式就是返回值，多写一个分号会让返回类型变成 `()` —— 这是新人最常见的编译错误。

**Q2：`match` 和 Java 的 `switch` 有什么区别？**

三点：① `match` **必须穷尽**所有情况，漏掉编译不过；② `match` 是**表达式**，有返回值；③ 支持**模式解构**（结构体、枚举、范围、守卫、绑定）。此外 `match` 分支不会穿透，不需要写 `break`。穷尽性检查的实际价值是：给枚举加新成员时，编译器会把所有需要更新的地方全部报出来。

**Q3：`if let`、`while let`、`let else` 分别什么时候用？**

`if let` 用于只关心一种匹配情况；`while let` 用于"匹配成功就继续循环"，典型是 `while let Some(x) = stack.pop()`；`let else` 用于"匹配失败就提前返回"，`else` 分支必须发散，作用是消除嵌套、让绑定的变量在后续作用域直接可用。

**Q4：`loop` 和 `while true` 有什么区别？**

`loop` 的类型是 `!`（never），编译器知道它不会正常退出，因此能做更精确的类型推断和可达性分析，而且 `loop` 可以用 `break value` 返回值。`while true` 编译器仍按普通循环处理，也不能 `break` 带值。无限循环一律用 `loop`。

**Q5：`for x in v` 和 `for x in &v` 有什么区别？**

`for x in v` 调用 `into_iter()`，**移走集合所有权**，迭代出 `T`，循环后 `v` 不可用；`for x in &v` 调用 `iter()`，只是借用，迭代出 `&T`，`v` 之后仍可用；`&mut v` 对应 `iter_mut()`，迭代出 `&mut T` 可修改元素。

**Q6：Rust 为什么不会有 `ConcurrentModificationException`？**

因为遍历时集合被借用检查器"锁住"了：`for x in &v` 持有 `v` 的不可变借用，此时再调用 `v.push()`（需要可变借用）会直接**编译错误**。Java 靠 `modCount` 在运行时抛异常，Rust 把同一类问题提前到了编译期。

**Q7：`match` 里的 `_` 有什么风险？**

`_` 会吞掉所有未列出的情况。对自己定义的、后续会扩展的枚举，写 `_` 意味着以后新增成员时不会有任何编译提示，逻辑静默走到默认分支。工程上应尽量列全分支，把 `_` 留给外部类型或真正无穷的取值域。

---

## 五、函数、闭包与函数式编程

### 1. 函数定义

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b                       // ⭐ 尾表达式即返回值，不写 return
}

fn log(msg: &str) { }           // 无返回值，等价于 -> ()

fn find(id: u64) -> Option<User> {
    if id == 0 { return None; } // 提前返回才用 return
    db.get(id)
}
```

四条规则：

| 规则 | 说明 |
| ---- | ---- |
| 参数**必须**标注类型 | 函数签名不做类型推断，这是 API 稳定性的保证 |
| 返回值类型必须标注 | 省略即 `()` |
| 命名用 `snake_case` | 与 Java 的小驼峰不同 |
| 定义位置任意 | 不像 C 需要先声明，函数在调用点之后定义也可以 |

**⭐ Rust 不支持的三件事**（面试常问）：

```rust
fn add(a: i32) {}
fn add(a: i32, b: i32) {}       // ✘ 不支持函数重载

fn conn(timeout: u32 = 30) {}   // ✘ 不支持默认参数

fn sum(nums: ...) {}            // ✘ 不支持可变参数
```

**替代方案**：

```rust
// ① 参数用 Option，调用方传 None 表示"用默认值"
fn conn(url: &str, timeout: Option<u32>) {
    let timeout = timeout.unwrap_or(30);
}

// ② ⭐ 参数多/可选时用 Builder 模式（工程主流做法）
Client::builder().url(url).timeout(30).build()?;

// ③ 参数结构体 + Default
#[derive(Default)]
struct Opts { timeout: u32, retry: u8 }
fn conn(url: &str, opts: Opts) {}
conn(url, Opts { timeout: 30, ..Default::default() });

// ④ 可变参数用切片或宏
fn sum(nums: &[i32]) -> i32 { nums.iter().sum() }
println!("{} {}", a, b);        // 真正的可变参数只有宏能做到
```

**嵌套函数**：可以在函数内定义函数，但**不能捕获外层变量**（需要捕获就得用闭包）：

```rust
fn outer() {
    let x = 5;
    fn inner() { println!("{x}"); }     // ✘ 嵌套函数无法捕获环境
    let inner = || println!("{x}");     // ✔ 闭包可以
}
```

---

### 2. 函数是一等公民

函数可以像值一样传递，类型是 **`fn`（小写）指针类型**：

```rust
fn double(x: i32) -> i32 { x * 2 }

let f: fn(i32) -> i32 = double;         // ⭐ 函数指针类型
println!("{}", f(5));                   // 10

// 直接把函数名传给需要闭包的地方
let v: Vec<i32> = vec![1, 2, 3].iter().map(|x| double(*x)).collect();
let lens: Vec<usize> = names.iter().map(|s| s.len()).collect();
let lens: Vec<usize> = names.iter().map(String::len).collect();   // ⭐ 更简洁

let nums: Vec<String> = vec![1, 2].iter().map(ToString::to_string).collect();
let opts: Vec<Option<i32>> = vec![1, 2].into_iter().map(Some).collect();  // ⭐ 枚举构造器也是函数
```

**高阶函数**：接收或返回函数/闭包的函数。

```rust
fn apply(f: fn(i32) -> i32, v: i32) -> i32 { f(v) }
apply(double, 5);
```

---

### 3. 闭包基础

**闭包（closure）= 可以捕获环境变量的匿名函数**：

```rust
let add = |a: i32, b: i32| -> i32 { a + b };
let add = |a, b| a + b;                 // ⭐ 类型可推断，单表达式可省大括号

let factor = 3;
let scale = |x| x * factor;             // ⭐ 捕获了外部变量 factor
println!("{}", scale(5));               // 15
```

| 对比项 | 函数 `fn` | 闭包 |
| ------ | --------- | ---- |
| 捕获环境 | ✘ | ⭐ ✔ |
| 类型标注 | 必须写 | 可推断 |
| 类型 | `fn(i32) -> i32`，具体类型 | ⭐ **匿名类型**，每个闭包各不相同 |
| 用途 | 通用逻辑 | 迭代器、回调、线程体 |

**⭐ 每个闭包都是一个独立的匿名类型**，即使签名完全一样：

```rust
let a = |x: i32| x + 1;
let b = |x: i32| x + 1;
let v = vec![a, b];             // ✘ 编译错误：两者类型不同
let v: Vec<Box<dyn Fn(i32) -> i32>> = vec![Box::new(a), Box::new(b)];   // ✔
```

**闭包的类型推断只发生一次**：

```rust
let f = |x| x;
f(5);                           // 此处把 x 推断为 i32
f("abc");                       // ✘ 类型已锁定为 i32
```

---

### 4. 三个闭包 Trait：Fn / FnMut / FnOnce（重点）

闭包按**捕获和使用环境的方式**自动实现三个 Trait，这是闭包最核心的考点：

| Trait | 捕获方式 | 调用次数 | 方法签名 |
| ----- | -------- | -------- | -------- |
| ⭐`Fn` | 不可变借用 `&T` | 多次 | `fn call(&self)` |
| ⭐`FnMut` | 可变借用 `&mut T` | 多次，会修改环境 | `fn call_mut(&mut self)` |
| ⭐`FnOnce` | 取得所有权 `T` | ⭐ **只能一次** | `fn call_once(self)` |

**三者是包含关系**：`Fn` ⊂ `FnMut` ⊂ `FnOnce`。所有闭包都实现 `FnOnce`；不消耗捕获值的实现 `FnMut`；不修改捕获值的实现 `Fn`。

```rust
let s = String::from("hi");

// Fn：只读取
let print = || println!("{s}");
print(); print();                       // ✔ 可以多次调用

// FnMut：修改环境 → 闭包变量本身必须是 mut
let mut count = 0;
let mut inc = || count += 1;            // ⭐ 注意 let mut
inc(); inc();

// FnOnce：消耗捕获的值
let consume = || drop(s);               // s 的所有权被移进闭包
consume();
consume();                              // ✘ 编译错误：闭包已被消耗
```

**编译器自动按"够用就好"的原则选择捕获方式**：能借用就借用，非要所有权时才移动。

> Rust 2021 起是**按字段（disjoint）捕获**：闭包只捕获实际用到的字段，而不是整个结构体。这解决了老版本里"闭包用了 `s.a`，却导致整个 `s` 被借走"的问题。

---

### 5. move 闭包

`move` 关键字**强制**闭包取得所有捕获变量的所有权：

```rust
let s = String::from("hi");
let f = move || println!("{s}");        // ⭐ s 的所有权移入闭包
println!("{s}");                        // ✘ s 已被移走
```

**⭐ `move` 的主战场是线程和 async**（闭包可能比创建它的作用域活得更久）：

```rust
let data = vec![1, 2, 3];

thread::spawn(move || {                 // ⭐ 不加 move 编译不过
    println!("{:?}", data);             //    编译器无法证明 data 活得够久
}).join().unwrap();

tokio::spawn(async move {               // async 同理
    handle(data).await;
});
```

**⭐ 关键区分：`move` 决定"怎么捕获"，三个 Trait 决定"怎么调用"**。加了 `move` 的闭包如果只读取捕获值，依然实现 `Fn`：

```rust
let s = String::from("hi");
let f = move || println!("{s}");        // move 捕获，但只读 → 仍是 Fn
f(); f();                               // ✔ 可以多次调用
```

**`Copy` 类型被 `move` 的是副本**：

```rust
let n = 5;
let f = move || println!("{n}");
println!("{n}");                        // ✔ i32 是 Copy，原变量仍可用
```

---

### 6. 闭包作为参数与返回值

**作为参数**：三种写法，⭐ **优先用 `impl Trait`**。

```rust
// ① impl Trait（最简洁，静态分发，零开销）⭐ 首选
fn call(f: impl Fn(i32) -> i32) -> i32 { f(1) }

// ② 泛型（等价于 ①，需要在多处引用 F 时用）
fn call<F: Fn(i32) -> i32>(f: F) -> i32 { f(1) }
fn call<F>(f: F) -> i32 where F: Fn(i32) -> i32 { f(1) }

// ③ Box<dyn>（动态分发，有虚表开销，要存起来时才用）
fn call(f: Box<dyn Fn(i32) -> i32>) -> i32 { f(1) }
```

**⭐ 约束选最宽松的那个**：只调用不修改用 `Fn`，需要修改环境用 `FnMut`，只调用一次（如线程体）用 `FnOnce`。约束越宽松，调用方能传的闭包越多。

```rust
fn retry<F: FnMut() -> bool>(mut f: F, times: u8) {     // ⭐ 参数需要 mut
    for _ in 0..times {
        if f() { return; }
    }
}
```

**作为返回值**：

```rust
// ① impl Fn：⭐ 首选，静态分发无开销
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n                      // ⭐ 必须 move，否则 n 是局部变量会悬垂
}

// ② Box<dyn Fn>：需要返回不同闭包时唯一的选择
fn make_op(kind: &str) -> Box<dyn Fn(i32) -> i32> {
    match kind {
        "double" => Box::new(|x| x * 2),
        _        => Box::new(|x| x + 1),    // ⭐ 两个分支类型不同，impl Fn 不行
    }
}
```

| 返回方式 | 分发 | 性能 | 限制 |
| -------- | ---- | ---- | ---- |
| ⭐`impl Fn` | 静态 | 零开销，可内联 | 只能返回**一种**闭包类型 |
| `Box<dyn Fn>` | 动态 | 堆分配 + 虚表调用 | 可返回不同分支的闭包 |

---

### 7. 特殊函数形式

```rust
const fn square(n: u32) -> u32 { n * n }        // ⭐ 可在编译期求值
const SIZE: u32 = square(4);                    // 编译期算出 16

async fn fetch(url: &str) -> Result<String> { } // 异步函数，见第二十三章

fn fatal(msg: &str) -> ! {                      // 发散函数，永不返回
    panic!("{msg}")
}

unsafe fn raw_read(p: *const u8) -> u8 { *p }   // 见第二十四章

fn largest<T: PartialOrd>(list: &[T]) -> &T { } // 泛型函数，见第十四章
```

---

### 8. 高频踩坑点（重点）

**① 闭包捕获引发的借用冲突**

```rust
let mut v = vec![1, 2, 3];
let mut push = || v.push(4);            // 闭包可变借用了 v
println!("{:?}", v);                    // ✘ v 已被可变借用，不能再读
push();

// ✔ 缩小闭包的作用域
{
    let mut push = || v.push(4);
    push();
}
println!("{:?}", v);                    // ✔ 闭包已销毁，借用结束
```

**② `FnMut` 闭包忘记加 `mut`**

```rust
let mut count = 0;
let inc = || count += 1;                // ✘ cannot borrow as mutable
let mut inc = || count += 1;            // ✔ 闭包变量本身必须是 mut
```

**③ 循环里 spawn 线程忘记 clone**

```rust
let data = Arc::new(vec![1, 2, 3]);

for i in 0..3 {
    thread::spawn(move || {             // ✘ 第二次迭代时 data 已被移走
        println!("{:?}", data);
    });
}

for i in 0..3 {
    let data = Arc::clone(&data);       // ⭐ 每个线程一份 Arc 克隆
    thread::spawn(move || {
        println!("{:?}", data);
    });
}
```

**④ 返回闭包忘记 `move`**

```rust
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    |x| x + n                           // ✘ n 是局部变量，函数返回后就没了
}
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n                      // ✔
}
```

**⑤ 不同闭包类型不能混在一个集合里**

```rust
let ops = vec![|x: i32| x + 1, |x: i32| x * 2];      // ✘ 类型不同
let ops: Vec<Box<dyn Fn(i32) -> i32>> =             // ✔
    vec![Box::new(|x| x + 1), Box::new(|x| x * 2)];
```

**⑥ 闭包不能递归**

```rust
let fact = |n: u64| if n == 0 { 1 } else { n * fact(n - 1) };   // ✘ 引用了自身
fn fact(n: u64) -> u64 { if n == 0 { 1 } else { n * fact(n - 1) } }  // ✔ 用函数
```

**⑦ 闭包参数带类型时必须写大括号**

```rust
let f = |x: i32| x + 1;                 // ✔
let f = |x: i32| -> i32 x + 1;          // ✘ 写了返回类型就必须加 {}
let f = |x: i32| -> i32 { x + 1 };      // ✔
```

**⑧ `move` 不等于 `FnOnce`**

面试高频误解。`move` 只描述**捕获方式**（取得所有权），不决定能调用几次。`move` 闭包如果只读取捕获的值，依然是 `Fn`，可以多次调用。

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 函数参数用 `&str` / `&[T]` | 而非 `&String` / `&Vec<T>` |
| 参数超过 3~4 个改用结构体或 Builder | Rust 没有默认参数和重载 |
| 接收闭包优先用 `impl Fn` | 静态分发零开销，比 `Box<dyn Fn>` 快 |
| 闭包约束选最宽松的 | 能用 `Fn` 就不写 `FnMut`，能用 `FnMut` 就不写 `FnOnce` |
| 返回闭包默认 `impl Fn` | 只有多分支返回不同闭包才用 `Box<dyn Fn>` |
| 线程/async 闭包必须 `move` + `Arc::clone` | 共享数据用 `Arc`，不要克隆整份数据 |
| 简单转换直接传函数名 | `map(String::len)` 比 `map(\|s\| s.len())` 清晰 |
| 闭包体超过 10 行就抽成函数 | 长闭包严重影响可读性和错误定位 |
| 编译期可算的用 `const fn` | 减少运行期开销 |
| 公开 API 的函数签名要稳定 | 参数改用 `impl Into<String>` 等可提升易用性 |

---

### 10. 面试重点

**Q1：函数和闭包的区别？**

闭包能**捕获环境变量**，函数不能（函数内嵌套定义的函数也不能）。闭包的类型是编译器生成的**匿名类型**，每个闭包都独一无二，即使签名相同也不是同一类型；函数有具体的 `fn(T) -> U` 指针类型。闭包的参数和返回类型可以推断，函数签名必须显式标注。

**Q2：`Fn`、`FnMut`、`FnOnce` 有什么区别？**

按闭包对捕获环境的使用方式划分：`Fn` 以不可变借用捕获，可多次调用；`FnMut` 以可变借用捕获，可多次调用并修改环境；`FnOnce` 取得捕获值的所有权，**只能调用一次**。三者是包含关系 `Fn ⊂ FnMut ⊂ FnOnce`，所有闭包都实现 `FnOnce`。写函数约束时应选最宽松的那个，调用方能传的闭包才最多。

**Q3：`move` 关键字的作用？加了 `move` 就是 `FnOnce` 吗？**

`move` 强制闭包取得所有捕获变量的**所有权**，主要用于线程和 async —— 闭包会比创建它的作用域活得更久，编译器无法证明借用有效。**`move` 不等于 `FnOnce`**：`move` 描述的是捕获方式，Trait 描述的是调用方式。`move` 闭包只要不消耗捕获的值，依然实现 `Fn`，可以多次调用。

**Q4：为什么两个长得一样的闭包不能放进同一个 `Vec`？**

每个闭包都是编译器生成的独立匿名类型（结构体），即使签名完全相同也是不同类型，而 `Vec<T>` 要求元素同类型。解决办法是装箱成 trait 对象：`Vec<Box<dyn Fn(i32) -> i32>>`，代价是堆分配和动态分发。

**Q5：`impl Fn` 和 `Box<dyn Fn>` 怎么选？**

`impl Fn` 是静态分发，编译期单态化，零运行时开销、可内联，但一个函数只能返回**一种**具体闭包类型；`Box<dyn Fn>` 是动态分发，有堆分配和虚表调用开销，但可以在不同分支返回不同的闭包、也可以存进集合。默认用 `impl Fn`，确实需要类型擦除时才用 `Box<dyn Fn>`。

**Q6：Rust 为什么不支持函数重载和默认参数？**

重载会让类型推断和 trait 解析变得极其复杂（Rust 的推断是双向的），也会影响函数指针的唯一性；默认参数则会破坏函数签名的稳定性。Rust 的替代方案是 Builder 模式、`Option` 参数、参数结构体 + `Default`，以及用泛型 `impl Into<T>` 提升易用性。

**Q7：为什么 `let mut count = 0; let inc = || count += 1;` 编译不过？**

闭包修改了捕获的环境，所以它是 `FnMut`，调用它需要 `&mut self`，因此**闭包变量自身**也必须声明为 `mut`：`let mut inc = || count += 1;`。

---

## 六、字符串（String 与 &str）

### 1. 两种字符串类型

Rust 的字符串是新人最容易困惑的地方，根源在于它把**"拥有"和"借用"分成了两个类型**：

| 对比项 | ⭐`String` | ⭐`&str`（字符串切片） |
| ------ | --------- | -------------------- |
| 所有权 | 拥有 | 借用 |
| 存储位置 | 堆 | 指向堆 / 栈 / 二进制只读段 |
| 可变性 | 可变（加 `mut`） | 不可变 |
| 大小 | 24 字节（`ptr`+`len`+`cap`） | ⭐ 16 字节胖指针（`ptr`+`len`） |
| 本质 | `Vec<u8>` 的封装 | `&[u8]` 的封装 |
| 典型来源 | `String::from()`、`format!()` | 字面量 `"abc"`、`&s[..]` |
| 用途 | ⭐ 存储、返回值、需要修改 | ⭐ 函数参数、只读传递 |

```text
let s = String::from("hello");

栈                          堆
┌──────────────┐           ┌───┬───┬───┬───┬───┐
│ ptr      ────┼──────────►│ h │ e │ l │ l │ o │
│ len   = 5    │           └───┴───┴───┴───┴───┘
│ cap   = 5    │
└──────────────┘

let r: &str = &s[0..2];     胖指针：{ ptr → 'h' 处, len = 2 }
```

**字符串字面量的类型是 `&'static str`**，数据编译进二进制的只读段，程序全程有效：

```rust
let s: &'static str = "hello";      // 不在堆上，不需要分配
```

**⭐ 一句话记住选型**：**参数用 `&str`，存储和返回用 `String`**。

```rust
fn greet(name: &str) -> String {    // ⭐ 标准签名
    format!("Hello, {name}")
}

greet("world");                     // ✔ &str
greet(&String::from("world"));      // ✔ &String 自动 Deref 成 &str
```

---

### 2. 创建与相互转换

```rust
// &str → String
let s = String::from("hi");
let s = "hi".to_string();           // ⭐ 最常用
let s = "hi".to_owned();            // 等价，泛型场景语义更准确
let s: String = "hi".into();
let s = format!("{}-{}", a, b);     // ⭐ 拼接场景首选
let s = String::with_capacity(64);  // ⭐ 已知大致长度，预分配避免反复扩容

// String → &str
let r: &str = &s;                   // ⭐ Deref 强制转换，最常用
let r: &str = s.as_str();           // 显式写法
let r: &str = &s[..];               // 切片写法

// 数字 ↔ 字符串
let s = 42.to_string();
let n: i32 = "42".parse()?;

// 字节 ↔ 字符串
let bytes = s.as_bytes();                       // &[u8]，零开销
let bytes = s.into_bytes();                     // Vec<u8>，消耗 String
let s = String::from_utf8(bytes)?;              // ⭐ 校验 UTF-8，非法则 Err
let s = String::from_utf8_lossy(&bytes);        // ⭐ 非法字节替换成 �，不会失败
```

**特殊字面量写法**：

```rust
let s = "换行\n制表\t引号\"";
let s = r"C:\path\file";            // ⭐ 原始字符串，不处理转义
let s = r#"包含"引号"的内容"#;      // ⭐ 内容里有引号时加 #
let b = b"bytes";                   // 字节串，类型是 &[u8; 6]
let s = "多行字符串\
         用反斜杠续行";              // 续行不产生换行符
```

---

### 3. UTF-8 与索引（核心难点）

⭐ **Rust 字符串底层是 UTF-8 变长编码**，这决定了它所有"反直觉"的行为：

| 字符 | UTF-8 字节数 |
| ---- | ------------ |
| ASCII（`a`、`1`） | 1 |
| 拉丁扩展、希腊、西里尔 | 2 |
| ⭐ 中日韩汉字 | **3** |
| ⭐ emoji | **4** |

**⭐ 不能用下标索引字符串**：

```rust
let s = String::from("hello");
let c = s[0];                       // ✘ 编译错误：String 没有实现 Index<usize>
```

三个原因：① UTF-8 变长，无法 O(1) 定位第 n 个字符；② 返回值该是字节还是字符，语义不清；③ 按字节取会切出无效字符。

**`len()` 返回的是字节数，不是字符数**：

```rust
"hello".len();              // 5
"中文".len();               // ⭐ 6（每个汉字 3 字节）
"👍".len();                 // ⭐ 4

"中文".chars().count();     // ⭐ 2，真正的字符数（O(n) 遍历）
```

**切片按字节位置，切到字符中间会 panic**：

```rust
let s = "中文abc";
&s[0..3];                   // ✔ "中"（正好是一个完整汉字）
&s[0..2];                   // ✘ panic: byte index 2 is not a char boundary

s.get(0..2);                // ⭐ None，返回 Option 不会 panic
s.is_char_boundary(2);      // ⭐ false，可提前检查
```

> ⭐ **只要字符串可能含中文/emoji，就不要写死字节下标**。要按字符截取用 `chars().take(n)`，要安全切片用 `get()`。

---

### 4. 遍历字符串

```rust
let s = "中a👍";

for c in s.chars() { }              // ⭐ 按字符：'中', 'a', '👍'
for b in s.bytes() { }              // 按字节：9 个 u8
for (i, c) in s.char_indices() { }  // ⭐ (字节索引, 字符)：(0,'中'), (3,'a'), (4,'👍')
for line in text.lines() { }        // ⭐ 按行，自动处理 \n 和 \r\n
for w in s.split_whitespace() { }   // 按空白切分
```

**按"人眼看到的字符"（字素簇）遍历需要第三方库**：

```rust
// Cargo.toml: unicode-segmentation = "1"
use unicode_segmentation::UnicodeSegmentation;

let s = "y̆es";                      // y + 组合符号
s.chars().count();                  // 4（y、组合符、e、s）
s.graphemes(true).count();          // ⭐ 3，符合直觉
```

| 层级 | 方法 | `"👨‍👩‍👦"`（家庭 emoji）的结果 |
| ---- | ---- | ---- |
| 字节 | `.len()` | 18 |
| 字符（码点） | `.chars().count()` | 5 |
| ⭐ 字素簇 | `.graphemes(true).count()` | **1** |

> 数据库字段长度校验、短信字数统计、UI 截断都应按**字素簇**算，否则用户输入 emoji 会出问题。

---

### 5. 常用 API

| 用途 | 方法 | 说明 |
| ---- | ---- | ---- |
| 长度判空 | ⭐`len()` | **字节数** |
| | ⭐`is_empty()` | 是否为空串 |
| | `chars().count()` | 字符数（O(n)） |
| 查找 | ⭐`contains(p)` | `"hello".contains("ell")` → `true` |
| | `starts_with` / `ends_with` | `"a.txt".ends_with(".txt")` |
| | `find(p)` / `rfind(p)` | ⭐ 返回**字节索引** `Option<usize>` |
| 截取 | ⭐`&s[a..b]` | 按字节，越界或切坏字符会 panic |
| | ⭐`get(a..b)` | 返回 `Option<&str>`，安全 |
| 修改 | ⭐`push_str(s)` / `push(c)` | 追加（需 `mut`） |
| | `insert_str(i, s)` / `remove(i)` | 按字节位置插入/删除 |
| | `truncate(n)` / `clear()` / `pop()` | 截断 / 清空 / 弹出末尾字符 |
| 替换 | ⭐`replace(a, b)` | ⭐ **字面量**替换全部，不是正则 |
| | `replacen(a, b, n)` | 替换前 n 个 |
| 去空白 | ⭐`trim()` | `trim_start()` / `trim_end()` |
| | `trim_matches(p)` | 去掉指定字符 |
| 大小写 | `to_uppercase()` / `to_lowercase()` | ⭐ 返回 `String`（长度可能变） |
| | `to_ascii_uppercase()` | 只处理 ASCII，更快 |
| 切分 | ⭐`split(p)` | 返回迭代器 |
| | `splitn(n, p)` / `rsplit(p)` | 限制数量 / 从右切 |
| | ⭐`split_whitespace()` / `lines()` | 按空白 / 按行 |
| | `split_once(p)` | ⭐ 切一次，返回 `Option<(&str, &str)>` |
| 转换 | ⭐`parse::<T>()` | 返回 `Result` |
| | `as_bytes()` / `chars()` | — |
| 重复 | `repeat(n)` | `"ab".repeat(3)` → `"ababab"` |

**⭐ 与 Java 的关键差异：`split()` 的参数不是正则**，而是模式（字符、字符串、闭包），所以没有 Java 那个 `"192.168.1.1".split(".")` 的坑：

```rust
"192.168.1.1".split('.');                   // ✔ 直接就是按点切分
"a,b;c".split(|c| c == ',' || c == ';');    // ⭐ 闭包做模式，很灵活
"key=value".split_once('=');                // ⭐ Some(("key", "value"))

let parts: Vec<&str> = "a,b,c".split(',').collect();    // 切分结果是借用，零拷贝
```

要用正则得引入 `regex` crate（不在标准库里）：

```rust
static RE: LazyLock<Regex> = LazyLock::new(|| Regex::new(r"\d+").unwrap());
RE.replace_all(text, "N");          // ⭐ Regex 编译很贵，必须复用
```

---

### 6. 拼接与格式化

```rust
// ① ⭐ format!：可读性最好，单次拼接首选
let s = format!("{name} 今年 {age} 岁");           // 内联变量（Rust 1.58+）
let s = format!("{}-{}", a, b);

// ② ⭐ push_str：循环中拼接必须用它
let mut s = String::with_capacity(1024);
for item in &items {
    s.push_str(&item.name);
    s.push(',');
}

// ③ + 运算符：⭐ 会移走左侧所有权
let s3 = s1 + &s2;                  // s1 被消耗，之后不可用

// ④ 集合拼接
let s = vec!["a", "b"].join(",");           // ⭐ "a,b"，最常用
let s = vec!["a", "b"].concat();            // "ab"
let s: String = items.iter().map(|i| i.name.as_str())
                    .collect::<Vec<_>>().join(", ");
```

**⭐ 循环内用 `+` 或 `format!` 是 O(n²)**，和 Java 的 `String +=` 同理：

```rust
let mut s = String::new();
for x in &list { s = s + x; }               // ✘ 每次都重新分配
for x in &list { s.push_str(x); }           // ✔
```

**格式化占位符**：

```rust
format!("{}", x);           // Display，面向用户
format!("{:?}", x);         // ⭐ Debug，面向调试（需 #[derive(Debug)]）
format!("{:#?}", x);        // ⭐ Debug 美化换行，打印结构体常用
format!("{:.2}", 3.14159);  // 3.14，保留两位小数
format!("{:>8}", x);        // 右对齐宽度 8；< 左对齐，^ 居中
format!("{:08.2}", 3.1);    // 00003.10，补零
format!("{:x}", 255);       // ff；{:b} 二进制，{:o} 八进制
format!("{:+}", 5);         // +5
```

> `println!`、`write!`、`log::info!` 等都用同一套语法。日志一律用占位符而不是先 `format!` 再传入 —— 日志级别未开启时不会产生任何格式化开销。

---

### 7. 其他字符串类型

实际工程里还会遇到这几个，知道什么时候用即可：

| 类型 | 用途 |
| ---- | ---- |
| ⭐`Cow<'a, str>` | "可能修改也可能不改"：不改时零拷贝借用，改了才分配 |
| `Box<str>` | 不再需要增长的字符串，比 `String` 少 8 字节（无 `capacity`） |
| ⭐`Rc<str>` / `Arc<str>` | 多处共享同一份只读字符串，比克隆 `String` 省内存 |
| ⭐`PathBuf` / `&Path` | 文件路径专用，处理平台差异，**不要用 String 拼路径** |
| `OsString` / `&OsStr` | 操作系统原生字符串，可能不是合法 UTF-8 |
| `CString` / `&CStr` | 与 C 交互，以 `\0` 结尾 |

```rust
use std::borrow::Cow;

fn sanitize(input: &str) -> Cow<'_, str> {
    if input.contains(' ') {
        Cow::Owned(input.replace(' ', "_"))     // 需要改 → 分配新 String
    } else {
        Cow::Borrowed(input)                    // ⭐ 不需要改 → 零拷贝
    }
}
```

---

### 8. 高频踩坑点（重点）

**① `len()` 是字节数，不是字符数**

```rust
if username.len() > 20 { }          // ✘ 中文用户名 7 个字就超了
if username.chars().count() > 20 { }        // ✔ 按字符
if username.graphemes(true).count() > 20 { } // ✔ 按字素簇（最准确）
```

**② 切片切到字符中间直接 panic**

```rust
let s = "你好世界";
let short = &s[0..5];               // ✘ panic，5 不是字符边界

// ✔ 按字符安全截取
let short: String = s.chars().take(2).collect();        // "你好"
// ✔ 或用 get 返回 Option
let short = s.get(0..6).unwrap_or(s);                   // "你好"
```

**③ `+` 移走左侧所有权**

```rust
let s3 = s1 + &s2;
println!("{s1}");                   // ✘ value borrowed after move
let s3 = format!("{s1}{s2}");       // ✔
```

`impl Add for String` 的签名是 `fn add(self, other: &str)`，`self` 按值传入。同时注意右侧必须是 `&str`：`s1 + s2` 编译不过，得写 `s1 + &s2`。

**④ 参数写成 `&String` 而不是 `&str`**

```rust
fn greet(name: &String) { }         // ✘ 只能接受 &String
greet("hi");                        // ✘ 编译错误

fn greet(name: &str) { }            // ✔ 两种都能接受
greet("hi");
greet(&String::from("hi"));
```

> Clippy 会直接报 `ptr_arg` 警告。同理 `&Vec<T>` 应写成 `&[T]`。

**⑤ 到处 `.clone()` / `.to_string()`**

```rust
fn process(s: String) { }
process(text.clone());              // ✘ 每次调用都深拷贝整个字符串

fn process(s: &str) { }             // ✔ 借用，零拷贝
process(&text);
```

> 新人为了"绕过借用检查器"疯狂 `clone()` 是最常见的性能问题。正确做法是理解所有权（第八章），而不是靠 `clone` 硬闯。

**⑥ `replace()` 是字面量替换，不是正则**

这点反而比 Java 省心 —— `"a.b.c".replace(".", "-")` 在 Rust 里就是 `"a-b-c"`。要正则得用 `regex` crate，且 `Regex::new()` 很贵，必须提为静态变量复用。

**⑦ 大小写转换后长度可能变**

```rust
"ß".to_uppercase();                 // "SS"，1 个字符变成 2 个
"İ".to_lowercase();                 // 长度也会变
```

所以 `to_uppercase()` 返回 `String` 而不是原地修改。只处理 ASCII 时用 `to_ascii_uppercase()` 更快。

**⑧ `find()` 返回的是字节索引**

```rust
let s = "中文abc";
let i = s.find('a').unwrap();       // ⭐ 6，不是 2
&s[i..];                            // ✔ 直接用于切片是安全的（本来就是字节位置）
```

**⑨ 反转字符串**

```rust
let r: String = s.chars().rev().collect();      // ⭐ 按字符反转
// 注意：含组合字符、emoji 时按字符反转仍会出错，需按字素簇反转
```

**⑩ `String::from_utf8` vs `from_utf8_lossy`**

```rust
String::from_utf8(bytes)?;                  // ⭐ 严格校验，非法返回 Err
String::from_utf8_lossy(&bytes);            // ⭐ 非法字节替换成 �，返回 Cow
unsafe { String::from_utf8_unchecked(b) };  // ✘ 非法 UTF-8 会导致 UB
```

处理网络/文件的不可信输入时用前两个，绝不用 `unchecked`。

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 参数用 `&str`，返回/存储用 `String` | 参数写 `&String` 会被 Clippy 警告 |
| 循环内拼接用 `push_str` | `+` 和 `format!` 在循环里是 O(n²) |
| 已知长度用 `with_capacity` | 避免反复扩容拷贝 |
| 拼接集合用 `join()` | 不手动处理末尾分隔符 |
| 日志用占位符 | `info!("id={}", id)`，不预先 `format!` |
| 长度校验按 `chars().count()` | 或字素簇，`len()` 是字节数 |
| 截取用 `chars().take()` 或 `get()` | 直接切片会 panic |
| 正则 `Regex` 提为静态变量 | `LazyLock` / `OnceLock` 缓存，编译很贵 |
| 路径用 `PathBuf` / `Path` | 不用 `String` 拼 `/`，跨平台会出问题 |
| 不可信字节用 `from_utf8` 并处理错误 | 禁用 `from_utf8_unchecked` |
| 只读共享用 `Arc<str>` | 比到处 `clone()` `String` 省内存 |
| 少写 `clone()` | 先想清楚所有权，而不是靠 clone 绕过编译器 |

---

### 10. 面试重点

**Q1：`String` 和 `&str` 有什么区别？**

`String` 拥有所有权，数据在堆上，可变，本质是 `Vec<u8>` 的封装，栈上占 24 字节（指针+长度+容量）；`&str` 是字符串切片，只是借用，不可变，是 16 字节的胖指针（指针+长度），可以指向堆、栈或二进制只读段。字符串字面量的类型是 `&'static str`。工程惯例是**参数用 `&str`，存储和返回用 `String`**。

**Q2：为什么 Rust 的字符串不能用下标索引？**

因为底层是 UTF-8 变长编码：ASCII 占 1 字节、汉字 3 字节、emoji 4 字节。① 无法 O(1) 定位第 n 个字符；② 返回字节还是字符语义不明；③ 按字节取会切出无效 UTF-8。所以 Rust 只提供 `chars()`、`bytes()`、`char_indices()` 显式遍历，以及按字节的切片（切坏边界会 panic）。

**Q3：`"中文".len()` 返回多少？**

**6**。`len()` 返回的是 UTF-8 字节数，每个汉字占 3 字节。字符数要用 `chars().count()`（O(n)），"人眼看到的字符"要用 `unicode-segmentation` 的 `graphemes()`。

**Q4：字符串拼接有哪些方式？性能怎样？**

`format!`（可读性最好，单次拼接首选）、`push_str`（循环内必须用它）、`+`（会移走左侧 `String` 的所有权）、`join()`/`concat()`（拼接集合）。循环里用 `+` 或 `format!` 会不断重新分配，复杂度 O(n²)；正确做法是 `String::with_capacity()` 预分配后 `push_str`。

**Q5：为什么函数参数应该用 `&str` 而不是 `&String`？**

`&str` 更通用：由于 Deref 强制转换，`&String` 会自动转成 `&str`，所以 `&str` 参数能同时接受字面量、`String` 的借用和其他切片；而 `&String` 参数只能接受 `String` 的引用，连字符串字面量都传不进去。同理 `&[T]` 优于 `&Vec<T>`。

**Q6：`s[0..2]` 什么时候会 panic？怎么避免？**

切片下标是**字节位置**，如果落在多字节字符的中间（不是 char boundary）就会 panic。避免方法：用 `s.get(0..2)` 返回 `Option<&str>`、用 `s.is_char_boundary(i)` 提前判断，或者干脆按字符操作 `s.chars().take(n).collect::<String>()`。

**Q7：什么是 `Cow<str>`，什么时候用？**

`Cow`（Clone on Write）是一个枚举，要么借用（`Borrowed`）要么拥有（`Owned`）。用于"大多数情况不需要修改，少数情况才需要修改"的场景：不修改时零拷贝直接借用，需要修改时才分配新的 `String`。典型场景是字符串清洗、转义、路径规范化这类函数的返回值。

**Q8：Rust 的 `split()` 和 Java 的有什么区别？**

Java 的 `split()` 参数是**正则表达式**，所以 `split(".")` 会切出空数组，必须转义成 `\\.`；Rust 的 `split()` 参数是**模式**（字符、字符串或闭包），`split('.')` 就是按点切分，没有这个坑。Rust 标准库不含正则，需要正则得引入 `regex` crate。另外 Rust 的切分结果是 `&str` 借用，零拷贝。

---

## 七、复合类型（元组、数组、切片、Vec）

### 1. 四种类型总览

| 类型 | 写法 | 长度 | 元素类型 | 存储 | 所有权 |
| ---- | ---- | ---- | -------- | ---- | ------ |
| 元组 | `(i32, &str)` | 固定 | ⭐ 可不同 | 栈 | 拥有 |
| 数组 | `[i32; 5]` | ⭐ 固定，**是类型的一部分** | 相同 | 栈 | 拥有 |
| 切片 | `&[i32]` | 运行时决定 | 相同 | ⭐ 不拥有数据 | 借用 |
| ⭐`Vec` | `Vec<i32>` | ⭐ 可增长 | 相同 | 堆 | 拥有 |

**⭐ 三条选型铁律**：

- **业务代码默认用 `Vec<T>`**，数组只在长度固定且很小、或性能极敏感时用
- **函数参数一律用 `&[T]`**，而不是 `&Vec<T>` 或 `&[T; N]`
- **元组超过 3 个元素就改用结构体**

---

### 2. 元组（Tuple）

**固定长度、元素类型可以不同**，主要用于多返回值和临时组合：

```rust
let t: (i32, f64, &str) = (1, 2.0, "three");

t.0;                                // ⭐ 用下标访问，注意是 . 不是 []
let (a, b, c) = t;                  // ⭐ 解构，更常用
let (a, ..) = t;                    // 只取第一个

// ⭐ 最主要的用途：多返回值
fn divide(a: i32, b: i32) -> (i32, i32) { (a / b, a % b) }
let (quotient, remainder) = divide(7, 2);

// 迭代器里到处是元组
for (i, item) in v.iter().enumerate() { }
for (k, v) in &map { }
map.insert(k, v);
```

**两个特殊形式**：

```rust
let unit = ();                      // 空元组 = 单元类型
let single = (5,);                  // ⭐ 单元素元组必须加逗号，(5) 就是 i32
```

> 标准库只为**最多 12 个元素**的元组实现了 `Debug`、`Clone` 等 trait。但实际上超过 3 个元素时 `t.0`/`t.3` 已经完全丧失可读性，**应该改用结构体**（第十一章）。

---

### 3. 数组（Array）

**长度固定，且长度是类型的一部分**，默认分配在**栈**上：

```rust
let a = [1, 2, 3];                  // [i32; 3]
let a: [i32; 5] = [1, 2, 3, 4, 5];
let a = [0; 5];                     // ⭐ [0, 0, 0, 0, 0]，重复初始化
let a = [String::new(); 3];         // ✘ 元素必须实现 Copy 才能这样写

a.len();                            // 5（编译期常量）
a[0];                               // 索引访问
```

**⭐ `[i32; 3]` 和 `[i32; 4]` 是完全不同的类型**：

```rust
fn sum(arr: [i32; 3]) -> i32 { }
sum([1, 2, 3, 4]);                  // ✘ 编译错误：类型不匹配

fn sum(arr: &[i32]) -> i32 { }      // ⭐ 用切片参数，任意长度都能接受
sum(&[1, 2, 3, 4]);
```

**越界检查**：

```rust
let a = [1, 2, 3];
a[5];                               // ⭐ 编译错误（常量下标编译期就能查出来）

let i = get_index();
a[i];                               // ⭐ 运行时 panic: index out of bounds
a.get(i);                           // ✔ Option<&i32>，安全
```

> Rust 会对运行时索引插入边界检查，这是它相对 C 的安全性来源。用迭代器（`for x in &a`）时编译器能证明不会越界，会把检查优化掉 —— 这就是**「用迭代器比用下标更快」**的原因。

**数组常用于固定大小的缓冲区**：

```rust
let mut buf = [0u8; 1024];          // ⭐ IO 读缓冲的标准写法
let n = reader.read(&mut buf)?;
```

---

### 4. 切片（Slice）

**切片是对一段连续内存的借用**，本身不拥有数据，是 `{ ptr, len }` 的**胖指针**（16 字节）：

```rust
let v = vec![1, 2, 3, 4, 5];

let s = &v[1..3];                   // ⭐ [2, 3]，左闭右开
let s = &v[..];                     // 全部
let s = &v[2..];                    // 从下标 2 到末尾
let s = &mut v[..2];                // 可变切片，可以修改元素

let a = [1, 2, 3];
let s: &[i32] = &a;                 // ⭐ 数组自动转切片
```

**⭐ 切片是函数参数的标准形式**，一次兼容数组、`Vec` 和其他切片：

```rust
fn sum(data: &[i32]) -> i32 { data.iter().sum() }

sum(&[1, 2, 3]);                    // 数组
sum(&vec![1, 2, 3]);                // ⭐ Vec 自动 Deref 成切片
sum(&v[1..3]);                      // 切片
```

**切片的常用方法**（`Vec` 通过 `Deref` 全部继承）：

```rust
s.first();  s.last();               // Option<&T>
s.get(i);   s.get(1..3);            // ⭐ 安全访问，返回 Option
s.iter();   s.iter_mut();
s.contains(&x);
s.split_at(i);                      // ⭐ 一分为二，返回 (&[T], &[T])
s.chunks(3);                        // ⭐ 每 3 个一组（批处理常用）
s.windows(2);                       // ⭐ 滑动窗口，[1,2],[2,3],[3,4]
s.concat();  s.join(", ");
s.swap(i, j);  s.reverse();  s.fill(0);
```

---

### 5. Vec\<T\>：最常用的集合

**堆分配、可增长**，内存布局是 `{ ptr, len, capacity }`（栈上 24 字节）：

```text
let v = vec![1, 2, 3];

栈                          堆
┌──────────────┐           ┌───┬───┬───┬╌╌╌┬╌╌╌┐
│ ptr      ────┼──────────►│ 1 │ 2 │ 3 │   │   │
│ len   = 3    │           └───┴───┴───┴╌╌╌┴╌╌╌┘
│ cap   = 5    │                          未使用容量
└──────────────┘
```

**创建**：

```rust
let v = vec![1, 2, 3];                      // ⭐ 宏，最常用
let v: Vec<i32> = Vec::new();               // 空 Vec，必须标类型
let v = Vec::with_capacity(1000);           // ⭐ 已知大小时预分配
let v = vec![0; 100];                       // 100 个 0
let v: Vec<i32> = (1..=5).collect();        // ⭐ 从迭代器收集
let v: Vec<_> = items.iter().map(|x| x.id).collect();
```

**增删改查**：

| 操作 | 方法 | 复杂度 | 说明 |
| ---- | ---- | ------ | ---- |
| 尾部添加 | ⭐`push(x)` | O(1) 均摊 | — |
| 尾部弹出 | `pop()` | O(1) | 返回 `Option<T>` |
| 指定位置插入 | `insert(i, x)` | ⭐ O(n) | 后面元素整体后移 |
| 删除并保序 | `remove(i)` | ⭐ O(n) | 返回被删元素 |
| 删除不保序 | ⭐`swap_remove(i)` | ⭐ **O(1)** | 用最后一个元素填补空位 |
| 条件删除 | ⭐`retain(\|x\| ...)` | O(n) | 保留满足条件的 |
| 批量删除 | `drain(a..b)` | O(n) | 返回被删元素的迭代器 |
| 访问 | `v[i]` | O(1) | ⭐ 越界 panic |
| 安全访问 | ⭐`get(i)` | O(1) | 返回 `Option<&T>` |
| 追加 | `extend(iter)` / `append(&mut v2)` | O(m) | — |
| 清空 | `clear()` / `truncate(n)` | O(n) | 不释放容量 |
| 查找 | `contains(&x)` / `iter().position(...)` | O(n) | — |
| 二分查找 | `binary_search(&x)` | O(log n) | ⭐ 必须先排序 |

**排序与去重**：

```rust
let mut v = vec![3, 1, 2];

v.sort();                                   // ⭐ 稳定排序，需要 T: Ord
v.sort_unstable();                          // ⭐ 不稳定但更快，元素无副作用时首选
v.sort_by(|a, b| b.cmp(a));                 // 降序
v.sort_by_key(|x| x.age);                   // ⭐ 按字段排序，最常用
v.sort_by(|a, b| a.score.total_cmp(&b.score));   // 浮点排序

v.sort();
v.dedup();                                  // ⭐ 只去除"连续"重复，必须先排序
v.reverse();
```

**扩容机制**：容量不足时**申请约 2 倍的新内存并把数据整体拷贝过去**，所以 `push` 是**均摊 O(1)**。这也意味着：

```rust
let mut v = Vec::new();
for i in 0..10_000 { v.push(i); }               // ⚠ 发生了约 14 次重分配 + 拷贝

let mut v = Vec::with_capacity(10_000);         // ⭐ 一次到位
for i in 0..10_000 { v.push(i); }
```

---

### 6. 三者关系与内存对比

```text
Vec<T>      拥有 + 可增长   →  堆   [ptr | len | cap]   24 字节
Box<[T]>    拥有 + 固定长   →  堆   [ptr | len]         16 字节
&[T]        借用 + 固定长   →  任意 [ptr | len]         16 字节
[T; N]      拥有 + 编译期长 →  栈   数据本身             N × size_of::<T>()
```

```rust
let v: Vec<i32> = vec![1, 2, 3];
let s: &[i32] = &v;                 // ⭐ Deref 强制转换，自动发生
let b: Box<[i32]> = v.into_boxed_slice();       // 确定不再增长时，省 8 字节
let v2: Vec<i32> = b.into_vec();                // 转回来，零拷贝
```

> `Vec` 通过 `Deref<Target = [T]>` 继承了切片的**全部**方法 —— 所以 `v.iter()`、`v.sort()`、`v.first()` 实际上都是切片的方法。这也解释了为什么函数参数写 `&[T]` 就够了。

---

### 7. 高频踩坑点（重点）

**① 数组长度是类型的一部分**

```rust
let mut a: [i32; 3] = [1, 2, 3];
a = [1, 2, 3, 4];                   // ✘ 类型不同

// ✔ 需要变长就用 Vec，函数参数就用 &[T]
```

**② 大数组会爆栈**

```rust
let big = [0u8; 10_000_000];        // ✘ 栈溢出（默认主线程栈 8MB）
let big = vec![0u8; 10_000_000];    // ✔ 堆分配
```

**③ 索引越界 panic**

```rust
let v: Vec<i32> = vec![];
v[0];                               // ✘ panic
v.first();                          // ✔ None
v.get(0);                           // ✔ None

let last = v.len() - 1;             // ✘ usize 溢出 panic（第二章）
let last = v.len().saturating_sub(1);   // ✔
```

**④ 遍历时修改 —— 编译期拦截**

```rust
for x in &v {
    v.push(*x);                     // ✘ 编译错误：不能同时不可变借用和可变借用
}

v.retain(|x| *x > 0);                       // ✔ 条件删除
let extra: Vec<_> = v.iter().map(|x| x * 2).collect();
v.extend(extra);                            // ✔ 先收集再修改
for x in v.iter_mut() { *x += 1; }          // ✔ 只改元素值是允许的
```

**⑤ `remove(0)` 是 O(n)**

```rust
while !v.is_empty() { v.remove(0); }        // ✘ 整体 O(n²)

v.swap_remove(0);                           // ✔ O(1)，但不保序
let mut q = VecDeque::from(v);              // ✔ 需要头部频繁进出用 VecDeque
q.pop_front();                              // O(1)
```

**⑥ `dedup()` 只去除连续重复**

```rust
let mut v = vec![1, 2, 1, 2];
v.dedup();                                  // ✘ 还是 [1, 2, 1, 2]

v.sort();  v.dedup();                       // ✔ [1, 2]
let set: HashSet<_> = v.into_iter().collect();      // ✔ 不需要保序时更快
```

**⑦ `Vec<Vec<T>>` 内存不连续**

```rust
let grid = vec![vec![0; cols]; rows];       // ⚠ 每一行都是独立的堆分配

// ⭐ 性能敏感时用一维 Vec 模拟二维，内存连续，缓存友好
let grid = vec![0; rows * cols];
grid[row * cols + col];
```

**⑧ 迭代器的三种形式选错**

```rust
for x in v { }                      // ⭐ 消耗 v，迭代出 T
for x in &v { }                     // 借用，迭代出 &T
for x in &mut v { }                 // 可变借用，迭代出 &mut T

let sum: i32 = v.iter().sum();      // ⭐ &T 求和，v 还能用
let sum: i32 = v.into_iter().sum(); // v 被消耗
```

> **数组的特殊之处**：Rust 2021 起 `for x in arr`（数组按值）迭代出 `T` 而不是 `&T`，与 `Vec` 行为一致。

**⑨ `collect()` 需要类型标注**

```rust
let v = (1..5).collect();                       // ✘ 收集成什么？编译器不知道
let v: Vec<i32> = (1..5).collect();             // ✔
let v = (1..5).collect::<Vec<i32>>();           // ✔ turbofish
let v: Vec<_> = (1..5).collect();               // ⭐ 元素类型可推断时用 _
```

**⑩ 切片区间越界**

```rust
let v = vec![1, 2, 3];
&v[1..10];                          // ✘ panic: range end index out of bounds
v.get(1..10);                        // ✔ None
```

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 函数参数用 `&[T]` | 不用 `&Vec<T>`（Clippy `ptr_arg` 警告）或 `&[T; N]` |
| 业务代码默认 `Vec<T>` | 数组只用于小的固定缓冲区 |
| 已知大小用 `with_capacity` | 避免多次重分配拷贝 |
| 访问用 `get()` / `first()` / `last()` | 而不是 `v[i]`，避免 panic |
| 不保序删除用 `swap_remove` | `remove(0)` 是 O(n) |
| 条件删除用 `retain` | 不要在循环里 `remove` |
| 头尾频繁进出用 `VecDeque` | `Vec` 的头部操作是 O(n) |
| 排序默认 `sort_unstable` | 除非需要稳定性；按字段用 `sort_by_key` |
| 去重先 `sort` 再 `dedup` | 或用 `HashSet` |
| 二维数据用一维 `Vec` 模拟 | `Vec<Vec<T>>` 内存不连续，缓存不友好 |
| 优先用迭代器而不是下标 | 无越界检查开销，且更不容易写错 |
| 返回空 `Vec` 不返回 `Option<Vec>` | 空集合本身就表达了"没有数据" |
| 元组超过 3 个元素改结构体 | `t.0`/`t.3` 完全丧失可读性 |

---

### 9. 面试重点

**Q1：数组、切片、`Vec` 有什么区别？**

数组 `[T; N]` 长度固定且**长度是类型的一部分**，通常在栈上，`[i32; 3]` 和 `[i32; 4]` 是不同类型；切片 `&[T]` 是对一段连续内存的**借用**，本身不拥有数据，是 `{ptr, len}` 的 16 字节胖指针；`Vec<T>` 在堆上分配、可动态增长，是 `{ptr, len, cap}` 的 24 字节结构。三者关系是：`Vec` 和数组都能自动转成切片，所以函数参数一律写 `&[T]`。

**Q2：`Vec` 的扩容机制是怎样的？**

容量不足时，申请一块约为原容量 **2 倍**的新内存，把已有元素拷贝过去，再释放旧内存。因此单次 `push` 最坏是 O(n)，但**均摊复杂度是 O(1)**。已知数据规模时应该用 `Vec::with_capacity(n)` 一次分配到位，避免多次重分配和拷贝。

**Q3：为什么函数参数要用 `&[T]` 而不是 `&Vec<T>`？**

`&[T]` 更通用：`Vec` 实现了 `Deref<Target = [T]>`，`&Vec<T>` 会自动转成 `&[T]`，所以 `&[T]` 参数能同时接受 `Vec`、数组和其他切片；而 `&Vec<T>` 只能接受 `Vec`。而且 `&Vec<T>` 是双重间接寻址，`&[T]` 更直接。这和「参数用 `&str` 不用 `&String`」是同一条规则。

**Q4：`remove()` 和 `swap_remove()` 的区别？**

`remove(i)` 删除后把后面所有元素往前移，保持顺序，复杂度 **O(n)**；`swap_remove(i)` 直接把最后一个元素搬到位置 `i`，复杂度 **O(1)**，但**不保证顺序**。不关心顺序时应优先用 `swap_remove`。需要频繁在头部增删则应该换成 `VecDeque`。

**Q5：Rust 遍历集合时删除元素会怎样？**

**编译不过**。`for x in &v` 持有 `v` 的不可变借用，此时调用 `v.push()`/`v.remove()`（需要可变借用）违反借用规则，编译器直接拒绝。这比 Java 运行时才抛 `ConcurrentModificationException` 安全得多。正确做法是 `retain()`、`drain()`，或先收集再修改。

**Q6：`sort()` 和 `sort_unstable()` 怎么选？**

`sort()` 是**稳定**排序（相等元素保持原有相对顺序），内部是归并排序的变体，需要额外内存；`sort_unstable()` 是**不稳定**排序（基于模式选择的快排），不需要额外分配，通常更快。**不依赖相等元素顺序时一律用 `sort_unstable()`**。两者都要求元素实现 `Ord`，所以 `f64` 不能直接用，得走 `sort_by(f64::total_cmp)`。

**Q7：为什么说"用迭代器比用下标快"？**

用 `v[i]` 索引时编译器必须插入边界检查（越界要 panic）；而用 `for x in &v` / `iter()` 时，迭代器的实现保证了不会越界，编译器能证明这一点并把边界检查**完全优化掉**。加上迭代器适配器会被内联和融合成单次遍历，最终代码往往和手写 C 循环一样快 —— 这就是 Rust 的**零成本抽象**。

---

## 八、所有权（Ownership）

> ⭐ 所有权是 Rust 最核心、也是面试必问的机制。它让 Rust **不用垃圾回收也能保证内存安全**，是理解后续所有章节的基础。

### 1. 为什么需要所有权

内存管理历来只有三种方案：

| 方案 | 代表语言 | 内存安全 | 运行时开销 | 主要问题 |
| ---- | -------- | :------: | ---------- | -------- |
| 手动管理 | C / C++ | ✘ | 无 | 悬垂指针、二次释放、内存泄漏 |
| 垃圾回收（GC） | Java / Go / C# | ✔ | ⭐ 有（STW 停顿、额外内存） | 延迟不可预测，不适合实时/嵌入式 |
| ⭐ 所有权 | **Rust** | ✔ | ⭐ **零**（编译期检查） | 学习曲线陡，编译器"啰嗦" |

Rust 的选择是：**把内存管理规则做成类型系统的一部分，在编译期强制检查**。代价是要多想一层，收益是既有 C 的性能，又有 Java 的安全。

它一次性消灭了这几类问题：

```text
✔ 悬垂指针（use after free）       ✔ 二次释放（double free）
✔ 缓冲区溢出                       ✔ 迭代器失效
✔ 数据竞争（data race）            ✔ 大部分内存泄漏
```

---

### 2. 栈与堆

理解所有权前必须分清两种内存：

| 对比项 | 栈（Stack） | 堆（Heap） |
| ------ | ----------- | ---------- |
| 大小 | ⭐ 编译期已知、固定 | 运行时决定、可变 |
| 分配速度 | ⭐ 极快（移动栈指针） | 慢（需要分配器查找空闲块） |
| 访问速度 | 快（缓存友好） | 慢（指针跳转） |
| 管理方式 | 自动，函数返回即弹出 | ⭐ 需要显式管理 —— 所有权要解决的正是这个 |
| 典型类型 | `i32`、`bool`、`[T; N]`、引用 | `String`、`Vec<T>`、`Box<T>` |

```text
let s = String::from("hi");

栈                       堆
┌────────────┐          ┌───┬───┐
│ ptr    ────┼─────────►│ h │ i │
│ len  = 2   │          └───┴───┘
│ cap  = 2   │
└────────────┘
 变量 s（所有者）          实际数据
```

**所有权管的就是堆上那部分数据的生命周期。**

---

### 3. 所有权三原则（必背）

> ⭐ ① Rust 中每一个值都有一个**所有者（owner）**
> ⭐ ② 同一时刻，一个值**有且只有一个**所有者
> ⭐ ③ 当所有者离开作用域时，这个值**被自动丢弃（drop）**

```rust
{
    let s = String::from("hi");      // s 成为所有者，堆内存被分配
    // 使用 s
}                                    // ⭐ s 离开作用域，自动调用 drop，堆内存被释放
```

这就是 C++ 的 **RAII**，但 Rust 把它变成了**编译器强制**的规则，而不是靠程序员自觉。

---

### 4. 移动（Move）

**⭐ 赋值、传参、返回值，对非 `Copy` 类型默认都是"移动"而非"拷贝"**：

```rust
let s1 = String::from("hi");
let s2 = s1;                        // ⭐ 移动，不是拷贝
println!("{s1}");                   // ✘ 编译错误：borrow of moved value: `s1`
```

```text
移动前                              移动后
s1 ──► [ptr|len|cap] ──► 堆"hi"     s1 ✘ 已失效（编译器标记）
                                    s2 ──► [ptr|len|cap] ──► 堆"hi"
```

**为什么必须这样设计**：如果 `s1` 和 `s2` 都有效，它们指向同一块堆内存，作用域结束时会被 **drop 两次 → double free**，这正是 C++ 里最经典的内存错误。Rust 的做法是让 `s1` 直接失效，从根上杜绝。

> ⭐ 移动只拷贝**栈上那 24 字节**（指针、长度、容量），堆数据一个字节都不动，所以移动**非常廉价**，和拷贝一个指针的成本相同。

**⭐ 五种会发生所有权转移的场景**：

```rust
let s2 = s1;                        // ① 赋值
takes(s1);                          // ② 传参
fn gives() -> String { s }          // ③ 返回值
v.push(s1);                         // ④ 放入集合
let f = move || println!("{s1}");   // ⑤ 闭包 move 捕获
```

---

### 5. Copy 与 Clone

**`Copy` trait：完全存储在栈上的类型，赋值时按位复制，原变量仍然可用**：

```rust
let a = 5;
let b = a;                          // ⭐ 复制，不是移动
println!("{a}");                    // ✔ a 依然有效
```

| 是 `Copy` | 不是 `Copy` |
| --------- | ----------- |
| 所有整数、浮点、`bool`、`char` | ⭐`String`、`Vec<T>`、`Box<T>`、`HashMap` |
| ⭐ 共享引用 `&T` | ⭐ 可变引用 `&mut T` |
| 元素全是 `Copy` 的元组 `(i32, bool)` | 含非 `Copy` 元素的元组 |
| 元素是 `Copy` 的数组 `[i32; 3]` | ⭐ 任何实现了 `Drop` 的类型 |

**⭐ `Copy` 和 `Drop` 互斥**：需要自定义清理逻辑（`Drop`）的类型不可能是 `Copy`，因为按位复制会导致资源被清理两次。

**`Clone`：显式的深拷贝**，可能很昂贵，所以必须手动调用：

```rust
let s1 = String::from("hi");
let s2 = s1.clone();                // ⭐ 深拷贝：堆数据也复制一份
println!("{s1} {s2}");              // ✔ 两个都有效
```

| 对比项 | `Copy` | `Clone` |
| ------ | ------ | ------- |
| 触发 | ⭐ 隐式（赋值时自动） | ⭐ 显式调用 `.clone()` |
| 语义 | 按位复制栈数据 | 可自定义，通常深拷贝 |
| 开销 | 极小 | ⭐ 可能很大 |
| 关系 | ⭐ `Copy` 必须同时实现 `Clone` | 可单独实现 |

```rust
#[derive(Clone, Copy, Debug)]       // ⭐ Copy 必须和 Clone 一起 derive
struct Point { x: i32, y: i32 }
```

> **`&T` 是 `Copy`，`&mut T` 不是** —— 因为可变引用必须唯一（第九章），能随便复制就破坏了这个保证。

---

### 6. 所有权与函数

```rust
fn takes(s: String) { }             // s 的所有权被移入
fn borrows(s: &str) { }             // ⭐ 只借用，不转移
fn gives() -> String { String::from("hi") }     // 所有权转移给调用方

let s = String::from("hi");
takes(s);
println!("{s}");                    // ✘ s 已被移走

let s = String::from("hi");
borrows(&s);
println!("{s}");                    // ✔ 只是借用
```

**⭐ 用"所有权转移"来传递参数是新手最容易写错的地方**。三种签名的选择：

| 签名 | 含义 | 何时用 |
| ---- | ---- | ---- |
| ⭐`fn f(x: &T)` | 只读借用 | ⭐ **默认选它** |
| `fn f(x: &mut T)` | 可变借用 | 需要修改调用方的数据 |
| `fn f(x: T)` | 取得所有权 | 需要存储、转换或销毁这个值 |

**方法的三种 `self` 同理**（第十一章详述）：

```rust
fn get(&self) -> &str { }           // ⭐ 只读，最常见
fn set(&mut self, v: String) { }    // 修改
fn into_inner(self) -> String { }   // ⭐ 消耗自身，方法名常以 into_ 开头
```

> **命名惯例**：`as_xxx`（借用转换，零开销）、`to_xxx`（克隆转换，有开销）、`into_xxx`（消耗所有权的转换）。看到 `into_` 开头就知道调用后原对象不可用了。

---

### 7. Drop 与 RAII

值离开作用域时自动调用 `Drop::drop`，这就是 Rust 版的 RAII：

```rust
struct Conn { name: String }

impl Drop for Conn {
    fn drop(&mut self) {
        println!("关闭连接 {}", self.name);      // ⭐ 自动释放资源
    }
}

{
    let a = Conn { name: "A".into() };
    let b = Conn { name: "B".into() };
}   // ⭐ 输出顺序：关闭连接 B，关闭连接 A（逆序析构）
```

**三个要点**：

```rust
let s = String::from("hi");
s.drop();                           // ✘ 不能手动调用 drop 方法
drop(s);                            // ⭐ 用 std::mem::drop 提前释放（其实就是把所有权移走）
```

- 析构顺序是**声明的逆序**（第一章第 6 节）
- `Drop` 与 `Copy` 互斥
- 数据库连接、文件句柄、锁守卫、临时目录都靠 `Drop` 自动清理 —— **Rust 不需要 `try-with-resources` 或 `finally`**

---

### 8. 部分移动与常用技巧

**部分移动（partial move）**：从结构体中移出某个字段后，整体不可用，但**其他字段仍可访问**：

```rust
struct User { name: String, age: u32 }
let u = User { name: "张三".into(), age: 20 };

let name = u.name;                  // ⭐ 移出 name 字段
println!("{}", u.age);              // ✔ 其他字段还能用
println!("{:?}", u);                // ✘ 整体已被部分移动，不能整体使用
```

**⭐ 三个必备技巧：`std::mem` 家族**

从 `&mut self` 里"偷走"一个值是极常见的需求，直接移动会编译报错（不能移出借用），标准做法是：

```rust
use std::mem;

// ① take：取出值，原位置留下 Default::default()
let old = mem::take(&mut self.buffer);      // ⭐ buffer 变成空 Vec/String

// ② replace：取出值，原位置放入指定的新值
let old = mem::replace(&mut self.state, State::Done);

// ③ swap：交换两个值
mem::swap(&mut a, &mut b);

// ④ Option 专用：取出并留下 None
let v = self.inner.take();                  // ⭐ Option<T> → Option<T>，原位置变 None
```

**从集合中取出元素**：

```rust
let v = vec![String::from("a")];
let s = v[0];                       // ✘ 不能从索引位置移出（会留下"空洞"）

let s = &v[0];                      // ✔ 借用
let s = v[0].clone();               // ✔ 克隆
let s = v.remove(0);                // ✔ 移除并取得所有权
let s = v.swap_remove(0);           // ✔ O(1)，不保序
for s in v { }                      // ✔ into_iter 消耗整个 Vec
let s = std::mem::take(&mut v[0]);  // ✔ 取出并留下默认值
```

---

### 9. 高频踩坑点（重点）

**① 传参后原变量失效**

```rust
let s = String::from("hi");
print_it(s);
print_it(s);                        // ✘ 第二次就报 use of moved value

fn print_it(s: &str) { }            // ✔ 改成借用
print_it(&s); print_it(&s);
```

**② `for x in v` 消耗整个集合**

```rust
let v = vec![1, 2, 3];
for x in v { }
println!("{:?}", v);                // ✘ v 已被移走
for x in &v { }                     // ✔
```

**③ 循环里移动同一个值**

```rust
let s = String::from("hi");
for _ in 0..3 {
    takes(s);                       // ✘ 第二次迭代时 s 已被移走
}

for _ in 0..3 { takes(s.clone()); } // ✔ 每次克隆（有开销）
for _ in 0..3 { borrows(&s); }      // ⭐ 更好：改成借用
```

**④ 结构体方法误用 `self`**

```rust
impl Config {
    fn name(self) -> String { self.name }       // ✘ 调用一次后 config 就没了
    fn name(&self) -> &str { &self.name }       // ✔
}
```

**⑤ 闭包意外捕获所有权**

```rust
let data = vec![1, 2, 3];
let f = move || println!("{:?}", data);
println!("{:?}", data);             // ✘ 已被 move 进闭包
```

**⑥ 用 `clone()` 硬闯借用检查（最常见的反模式）**

```rust
// ✘ 新人写法：编译器一报错就 clone
fn process(data: Vec<Item>) -> Vec<Item> {
    let a = data.clone();
    let b = data.clone();
    ...
}

// ✔ 先想清楚：这个函数真的需要所有权吗？
fn process(data: &[Item]) -> Vec<Item> { }
```

> ⭐ **`clone()` 不是解决所有权问题的答案，只是把问题变成了性能问题。** 遇到借用错误时先问三个问题：这个函数需要所有权吗（多半不需要 → 改 `&T`）？能不能缩小借用作用域？是否真的需要共享（→ `Rc`/`Arc`，第十六章）？

**⑦ `match` / `if let` 中的移动**

```rust
let opt = Some(String::from("hi"));
match opt {
    Some(s) => println!("{s}"),     // ⭐ s 移出了 opt
    None => {}
}
println!("{:?}", opt);              // ✘ opt 已被部分移动

match &opt {                        // ✔ 匹配引用
    Some(s) => println!("{s}"),
    None => {}
}
if let Some(s) = opt.as_ref() { }   // ✔ 等价写法
```

**⑧ 返回局部变量的引用**

```rust
fn bad() -> &String {
    let s = String::from("hi");
    &s                              // ✘ s 在函数结束时被 drop，引用会悬垂
}
fn good() -> String { String::from("hi") }      // ✔ 转移所有权
```

---

### 10. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 参数默认用 `&T` / `&str` / `&[T]` | 只在需要存储或消耗时才取所有权 |
| 需要修改才用 `&mut T` | 可变借用的限制更多（第九章） |
| 方法默认 `&self` | `self` 只用于 `into_xxx` 这类消耗型转换 |
| `clone()` 要有明确理由 | 不是为了绕过编译器，代码 review 重点关注 |
| 共享数据用 `Rc` / `Arc` | 而不是到处 `clone()` 整份数据 |
| 从 `&mut self` 取值用 `mem::take` / `Option::take` | 不要为此把签名改成 `self` |
| 资源清理写在 `Drop` 里 | 不要暴露 `close()` 让调用方手动调 |
| 命名遵循 `as_` / `to_` / `into_` 惯例 | 让调用方一眼看出所有权语义 |
| 大对象避免按值传递到深层调用链 | 顶层拿所有权，内部逐层借用 |
| 用 `#[derive(Clone)]` 而非手写 | 除非有特殊的深浅拷贝需求 |

---

### 11. 面试重点

**Q1：什么是所有权？三条规则是什么？**

所有权是 Rust 在编译期管理内存的机制。三条规则：① 每个值都有一个所有者；② 同一时刻一个值只能有一个所有者；③ 所有者离开作用域时值被自动 drop。它让 Rust 既不需要 GC，也不需要手动 `free`，且**没有任何运行时开销**——所有检查都在编译期完成。

**Q2：Rust 为什么不需要垃圾回收？**

因为每个值的释放时机在**编译期就完全确定**了：所有者离开作用域，值就被 drop。编译器在编译时插入释放代码，运行时不需要追踪对象存活、不需要扫描堆、没有 STW 停顿。代价是程序员要接受所有权规则的约束。相比 GC，优势是延迟可预测、内存占用低，适合系统编程、嵌入式和延迟敏感的服务。

**Q3：移动（move）和拷贝（copy）的区别？**

对**非 `Copy` 类型**（如 `String`、`Vec`），赋值/传参是**移动**：只复制栈上的指针、长度、容量，堆数据不动，同时**原变量失效**——这样保证同一块堆内存只有一个所有者，不会被 drop 两次。对 `Copy` 类型（整数、`bool`、`char`、`&T` 等完全在栈上的类型），赋值是**按位复制**，原变量依然可用。

**Q4：`Copy` 和 `Clone` 有什么区别？**

`Copy` 是隐式的、按位复制栈上数据、开销极小，赋值时自动发生；`Clone` 是显式调用 `.clone()`、可以自定义（通常是深拷贝）、可能很昂贵。`Copy` 要求必须同时实现 `Clone`。**`Copy` 和 `Drop` 互斥**：需要自定义析构的类型不能是 `Copy`，否则资源会被释放多次。

**Q5：为什么 `String` 不是 `Copy`？**

因为 `String` 的数据在堆上。如果按位复制，两个 `String` 会指向同一块堆内存，各自离开作用域时都会 drop，导致 **double free**。所以 `String` 只能移动（原变量失效）或显式 `clone()`（连堆数据一起复制）。

**Q6：什么是部分移动？**

从结构体中移出某个字段的所有权后，该结构体**整体不可再用**（不能整体传递或打印），但**其他未被移动的字段仍可单独访问**。要避免部分移动可以改用借用、`clone()`，或用 `std::mem::take` 取值并留下默认值。

**Q7：`std::mem::take` / `replace` / `swap` 解决什么问题？**

解决"想从 `&mut` 借用里拿走所有权"的困境 —— 直接移出会破坏借用的完整性，编译器不允许。`take` 取出值并留下 `Default::default()`，`replace` 取出并留下指定新值，`swap` 交换两个值。`Option::take()` 是最常用的特化版本，取出内部值并留下 `None`，链表、状态机、缓冲区复用里到处都是。

**Q8：遇到借用/所有权报错，为什么不建议直接 `clone()`？**

`clone()` 只是把编译错误变成了运行时的性能损耗，往往说明设计上没想清楚谁该拥有数据。正确的排查顺序是：① 这个函数真的需要所有权吗（大多数只需要 `&T`）；② 能否缩小借用的作用域；③ 是否确实需要多方共享（那就用 `Rc`/`Arc`，只克隆引用计数而非数据）。只有在确认需要一份独立数据时才 `clone()`。

---

## 九、借用与引用（Borrowing）

> 所有权解决了"值归谁"，借用解决了"**不转移所有权也能用**"。这两章合起来就是 Rust 的地基。

### 1. 引用的本质

**借用 = 创建一个引用，临时使用值但不获得所有权**：

```rust
fn calc_len(s: &String) -> usize {      // ⭐ & 表示借用
    s.len()
}                                       // ⭐ s 离开作用域，但因为不拥有值，什么也不会发生

let s = String::from("hi");
let len = calc_len(&s);                 // ⭐ 传引用
println!("{s} 的长度是 {len}");          // ✔ s 依然可用
```

```text
栈                    栈                     堆
┌──────────┐         ┌────────────┐        ┌───┬───┐
│ r    ────┼────────►│ ptr    ────┼───────►│ h │ i │
└──────────┘         │ len  = 2   │        └───┴───┘
  &String            │ cap  = 2   │
                     └────────────┘
                      String s（所有者）
```

引用在底层就是一个指针，**但编译器保证它永远指向有效数据** —— 这是它和 C 指针的根本区别，且**零运行时开销**。

| 类型 | 写法 | 别名 | 数量限制 |
| ---- | ---- | ---- | -------- |
| ⭐ 不可变借用 | `&T` | 共享引用 | ⭐ 可以有**任意多个** |
| ⭐ 可变借用 | `&mut T` | 独占引用 | ⭐ 同一时刻**只能有一个** |

---

### 2. 借用规则（必背）

> ⭐ **在任意给定时刻，对同一份数据，要么只能存在一个可变引用，要么只能存在任意多个不可变引用 —— 两者不能同时存在。**
> ⭐ **引用必须始终有效（不能悬垂）。**

**记忆方法：这就是读写锁**。多个读者可以并发读，写者必须独占。区别在于 Rust 是**编译期**检查，没有任何运行时开销。

```rust
let mut s = String::from("hi");

// ✔ 多个不可变借用
let r1 = &s;
let r2 = &s;
println!("{r1} {r2}");

// ✘ 可变与不可变同时存在
let r1 = &s;
let r2 = &mut s;                    // ✘ cannot borrow `s` as mutable
println!("{r1}");

// ✘ 两个可变借用
let r1 = &mut s;
let r2 = &mut s;                    // ✘ cannot borrow `s` as mutable more than once
println!("{r1}");
```

**⭐ 这条规则的价值：在编译期消灭数据竞争**。数据竞争需要三个条件同时成立 —— 两个以上指针访问同一数据、至少一个在写、没有同步机制。借用规则直接让第一和第二条无法同时成立，所以 **Rust 的多线程代码天生没有数据竞争**（第二十二章）。

**可变借用还要求原变量本身是 `mut`**：

```rust
let s = String::from("hi");
let r = &mut s;                     // ✘ 不可变绑定不能借出可变引用
let mut s = String::from("hi");
let r = &mut s;                     // ✔
```

---

### 3. NLL：借用在"最后一次使用"就结束

⭐ **借用的作用域不是到花括号结束，而是到它最后一次被使用为止** —— 这叫 **NLL（Non-Lexical Lifetimes，非词法生命周期）**，Rust 2018 引入。理解它能解释掉一大半"为什么这段能编译过"的疑惑：

```rust
let mut s = String::from("hi");

let r1 = &s;
let r2 = &s;
println!("{r1} {r2}");              // ⭐ r1、r2 的借用到此结束

let r3 = &mut s;                    // ✔ 编译通过！前面的不可变借用已经结束
println!("{r3}");
```

```rust
// ✘ 但如果不可变借用在后面还要用，就冲突了
let r1 = &s;
let r3 = &mut s;                    // ✘
println!("{r1}");                   // ← 因为这里还用到 r1，r1 的借用还活着
```

> ⭐ **调整代码顺序、让借用尽早结束，是解决借用冲突最常用的手段**，比 `clone()` 好得多。

---

### 4. 解引用与自动解引用

```rust
let x = 5;
let r = &x;
*r;                                 // ⭐ 解引用取值
*r == 5;                            // true
r == &5;                            // ✔ 引用之间也能直接比较

let mut n = 5;
let r = &mut n;
*r += 1;                            // ⭐ 通过可变引用修改，必须解引用
```

**⭐ 方法调用会自动解引用**，所以日常代码里很少需要手写 `*`：

```rust
let s = String::from("hi");
let r = &&&s;
r.len();                            // ⭐ 自动解引用三层，等价于 (***r).len()
```

Rust 在方法调用时会自动插入 `&`、`&mut` 或 `*` 来匹配方法签名，这叫**自动引用/解引用（auto-ref/auto-deref）**，配合第三章的 `Deref` 强制转换，让 `&String` 能当 `&str` 用、`&Vec<T>` 能当 `&[T]` 用。

**闭包和迭代器里需要手动解引用的场景**：

```rust
let v = vec![1, 2, 3];
v.iter().filter(|x| **x > 1);           // x 是 &&i32
v.iter().filter(|&&x| x > 1);           // ⭐ 模式解构更清晰
v.iter().copied().filter(|x| *x > 1);   // ⭐ copied() 把 &i32 变成 i32
```

**重借用（reborrow）**：`&mut T` 不是 `Copy`，但把它传给函数时编译器会自动**重借用** `&mut *r`，所以调用后原引用仍可用：

```rust
fn modify(v: &mut Vec<i32>) { v.push(1); }

let r = &mut v;
modify(r);                          // ⭐ 自动重借用，不是移动
modify(r);                          // ✔ r 还能继续用
```

---

### 5. 悬垂引用（Dangling Reference）

**编译器保证引用永远不会指向已释放的内存**：

```rust
fn dangle() -> &String {
    let s = String::from("hi");
    &s                              // ✘ 编译错误：s 在函数结束时被 drop
}                                   //    missing lifetime specifier

fn no_dangle() -> String {          // ✔ 转移所有权出去
    String::from("hi")
}
```

**⭐ 经典案例：为什么 `push` 会让之前的引用失效**

```rust
let mut v = vec![1, 2, 3];
let first = &v[0];                  // 借用了 v
v.push(4);                          // ✘ cannot borrow `v` as mutable
println!("{first}");
```

原因不是编译器死板，而是**真实的内存安全问题**：`push` 可能触发扩容，`Vec` 会申请新内存、拷贝数据、释放旧内存 —— 此时 `first` 就变成了指向已释放内存的悬垂指针。C++ 里这正是 `vector` 迭代器失效的经典 UB，Rust 在编译期就拦住了。

---

### 6. 常见借用冲突与解法（重点）

**① 借用期间修改 —— 缩小借用作用域**

```rust
// ✘
let first = &v[0];
v.push(4);
println!("{first}");

// ✔ 方案一：调整顺序，让借用先结束（NLL）
let first = &v[0];
println!("{first}");
v.push(4);

// ✔ 方案二：不持有引用，复制出值
let first = v[0];                   // i32 是 Copy
v.push(4);

// ✔ 方案三：用块显式限制作用域
{
    let first = &v[0];
    println!("{first}");
}
v.push(4);
```

**② 遍历时修改集合**

```rust
for x in &v {
    v.push(*x);                     // ✘
}

v.retain(|x| *x > 0);                               // ✔ 条件删除
for x in v.iter_mut() { *x += 1; }                  // ✔ 只改元素值
let extra: Vec<_> = v.iter().map(|x| x * 2).collect();
v.extend(extra);                                    // ✔ 先收集再修改
let len = v.len();
for i in 0..len { v[i] += 1; }                      // ✔ 索引不持有借用
```

**③ 同时可变借用两个元素 —— `split_at_mut`**

```rust
let mut v = vec![1, 2, 3, 4];
let a = &mut v[0];
let b = &mut v[1];                  // ✘ 编译器无法证明下标不同

let (left, right) = v.split_at_mut(2);      // ⭐ 标准库提供的安全拆分
left[0] += 1;
right[0] += 1;                              // ✔

v.swap(0, 1);                               // ⭐ 只是交换的话用 swap
```

**④ 结构体字段的分别借用**

```rust
struct S { a: Vec<i32>, b: Vec<i32> }

let a = &mut s.a;
let b = &mut s.b;                   // ⭐ ✔ 直接访问字段时，编译器能分辨是不同字段

// ✘ 但通过方法就不行了 —— 方法签名借用的是整个 self
let a = s.get_a();                  // fn get_a(&mut self) -> &mut Vec<i32>
let b = s.get_b();                  // ✘ self 已被可变借用
```

> 解法：直接访问字段、把结构体拆成两个、或者提供一个同时返回两者的方法 `fn get_both(&mut self) -> (&mut Vec<i32>, &mut Vec<i32>)`。

**⑤ 循环中调用 `&mut self` 方法**

```rust
impl Engine {
    fn run(&mut self) {
        for task in &self.tasks {           // ✘ 不可变借用了 self.tasks
            self.execute(task);             //    execute 需要 &mut self
        }
    }
}

// ✔ 方案一：先取出数据（第八章的 mem::take）
let tasks = std::mem::take(&mut self.tasks);
for task in &tasks { self.execute(task); }
self.tasks = tasks;

// ✔ 方案二：先收集要处理的索引/数据，再统一处理
let ids: Vec<_> = self.tasks.iter().map(|t| t.id).collect();
for id in ids { self.execute_by_id(id); }
```

**⑥ HashMap 的"查了再插" —— 用 `entry` API**

```rust
match map.get(&key) {
    Some(v) => *v,
    None => { map.insert(key, 0); 0 }       // ✘ map 仍被 get 借用
}

*map.entry(key).or_insert(0) += 1;          // ⭐ ✔ 一次查找，标准写法
map.entry(key).or_insert_with(Vec::new).push(item);
```

**⑦ 确实需要"共享 + 可变" —— 内部可变性**

上面所有办法都不行时（典型是图、树、观察者模式、缓存），就该用**内部可变性**：

```rust
Cell<T>          // Copy 类型的简单场景，无运行时检查
RefCell<T>       // ⭐ 单线程，把借用检查推迟到运行时，违规则 panic
Rc<RefCell<T>>   // ⭐ 单线程共享可变
Mutex<T>         // ⭐ 多线程
Arc<Mutex<T>>    // ⭐ 多线程共享可变
```

详见第十六章和第二十二章。**注意这是最后手段，不是遇到报错就上的捷径**。

---

### 7. 高频踩坑点

**① 引用不是"指针的别名"，它有生命周期**

```rust
let r;
{
    let x = 5;
    r = &x;                         // ✘ x 的生命周期比 r 短
}
println!("{r}");
```

**② `&mut` 传参后原引用还能用（重借用），但赋值给别人就是移动**

```rust
let r = &mut v;
let r2 = r;                         // ⭐ 移动，r 之后不可用
let r2 = &mut *r;                   // ✔ 显式重借用，r 之后仍可用
```

**③ 迭代器闭包里的多层引用**

```rust
v.iter().filter(|x| x > 1);         // ✘ x 是 &&i32，不能和 i32 比
v.iter().filter(|&&x| x > 1);       // ✔
v.iter().any(|&x| x == 3);          // ✔
```

**④ 不能同时读写同一个集合的不同表达式**

```rust
v.push(v.len());                    // ✘ v.len() 是不可变借用，push 是可变借用
let n = v.len();                    // ✔ 先算出来
v.push(n);
```

**⑤ 自引用结构体做不到**

```rust
struct Bad<'a> { data: String, ptr: &'a str }   // ⭐ 无法安全地指向自己的字段
```

Rust 不支持自引用结构体（除非用 `Pin` + `unsafe`，或第三方库 `ouroboros`）。实践中改用索引（存下标而不是引用）、或 `Rc`/`Arc`。

**⑥ 返回内部数据的引用会锁住整个结构体**

```rust
let name = config.get_name();       // 返回 &str，config 被借用
config.reload();                    // ✘ 需要 &mut self
println!("{name}");
```

> 解法：返回克隆的值，或者调整调用顺序让借用先结束。

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 默认借用，需要时才取所有权 | 参数写 `&T`、`&str`、`&[T]` |
| 借用作用域尽量小 | 利用 NLL，把借用放在使用点附近 |
| 优先调整代码顺序而非 `clone()` | 大多数借用冲突靠重排就能解决 |
| 需要两个可变引用用 `split_at_mut` | 不要用 `unsafe` 硬绕 |
| HashMap 更新用 `entry` API | 避免二次查找和借用冲突 |
| 内部可变性是最后手段 | `RefCell` 把编译期错误变成运行期 panic |
| 返回引用要谨慎 | 会锁住整个结构体，考虑返回克隆值或 `Arc` |
| 结构体设计避免自引用 | 用索引或 `Rc` 代替内部指针 |
| 迭代器闭包用 `|&x|` 模式解构 | 比写 `**x` 清晰 |
| `&mut self` 方法内避免长时间持有字段借用 | 必要时 `mem::take` 取出再放回 |

---

### 9. 面试重点

**Q1：Rust 的借用规则是什么？**

在任意时刻，对同一份数据，**要么只能有一个可变引用（`&mut T`），要么可以有任意多个不可变引用（`&T`），两者不能同时存在**；并且所有引用必须始终有效，不能悬垂。可以类比读写锁：多读并发、写者独占，区别是 Rust 在**编译期**完成检查，零运行时开销。

**Q2：借用规则是怎么防止数据竞争的？**

数据竞争需要三个条件同时成立：多个指针访问同一数据、至少一个在写、没有同步。借用规则保证"有可变引用时不能有任何其他引用"，直接让前两个条件无法同时成立。所以 Rust 的多线程代码**在编译期就杜绝了数据竞争**，这就是"无畏并发"的由来。

**Q3：什么是 NLL？**

Non-Lexical Lifetimes，非词法生命周期。Rust 2018 之前借用的作用域是到花括号结束，之后改成了**到最后一次使用为止**。这让大量原本合法但被拒绝的代码能通过编译，比如先创建两个不可变引用用完，再创建可变引用。实践意义是：**通过调整语句顺序让借用尽早结束**，是解决借用冲突最优先的手段。

**Q4：为什么 `let first = &v[0]; v.push(4);` 编译不过？**

因为 `push` 可能触发扩容：`Vec` 会申请新内存、拷贝元素、释放旧内存，此时 `first` 就指向了已释放的内存。这在 C++ 里就是经典的迭代器失效 UB，Rust 通过"持有不可变借用时不能可变借用"在**编译期**拦住了它。

**Q5：`&T` 和 `&mut T` 除了可变性还有什么区别？**

数量上：`&T` 可以同时存在多个，`&mut T` 同一时刻只能有一个且排斥所有 `&T`。类型上：**`&T` 实现了 `Copy`，`&mut T` 没有** —— 因为可变引用必须唯一，能随意复制就破坏了独占性。传参时 `&mut T` 靠编译器自动**重借用**（`&mut *r`）而非移动，所以调用后原引用仍可用。

**Q6：遇到"cannot borrow as mutable more than once"怎么办？**

按优先级：① 调整顺序让前一个借用尽早结束（NLL）；② 缩小借用作用域（加 `{}` 或提前 `drop`）；③ 需要同时可变访问集合的多个元素用 `split_at_mut`、`iter_mut`、`swap`；④ 结构体多字段改为直接访问字段而非通过 `&mut self` 方法；⑤ 用 `mem::take` 把数据先取出来；⑥ 确实需要"共享 + 可变"才上 `RefCell` / `Mutex`。

**Q7：什么是内部可变性？什么时候用？**

通过 `Cell`/`RefCell`/`Mutex` 等类型，在只持有 `&T`（不可变引用）的情况下修改内部数据 —— 它们把借用检查从编译期推迟到运行期（`RefCell` 违规会 panic）或用锁保证。用于编译期无法证明安全、但逻辑上确实安全的场景：图/树结构、观察者模式、缓存、`Rc<RefCell<T>>` 共享可变状态。**是最后手段**，因为它把编译错误变成了运行时错误。

**Q8：Rust 为什么不支持自引用结构体？**

结构体一旦移动（Rust 中移动是按位复制），内部指向自身字段的引用就会指向旧地址，变成悬垂指针。要安全地表达自引用需要 `Pin` 保证对象不被移动（异步 `Future` 就是这么做的），或者用 `unsafe`/第三方库。实践中通常改用**索引**（存下标而非引用）或 `Rc`/`Arc` 来表达内部关联。

---

## 十、生命周期（Lifetime）

> 生命周期是 Rust 学习曲线最陡的一段，但实际业务代码里 **90% 的场景靠省略规则自动推导，根本不用手写**。真正需要理解的是：它在解决什么问题、什么时候必须手写。

### 1. 生命周期是什么

**生命周期 = 引用保持有效的作用域范围**。每个引用都有生命周期，绝大多数时候由编译器自动推导。

⭐ **它的唯一目的：防止悬垂引用。**

```rust
let r;
{
    let x = 5;
    r = &x;                 // ✘ `x` does not live long enough
}                           // x 在这里被 drop
println!("{r}");            // r 会指向已释放的内存
```

```text
'a ┌─ r 的生命周期
   │  'b ┌─ x 的生命周期
   │     │
   │     └─ x 被 drop
   └─ 这里还在用 r        ⭐ 'b 比 'a 短 → 编译器拒绝
```

**⭐ 三个关键认知**（面试常考）：

1. 生命周期标注**不改变**任何引用的实际存活时间，它只是**告诉编译器多个引用之间的关系**
2. 生命周期**纯编译期概念**，不产生任何运行时代码，零开销
3. 标注是给编译器的**约束声明**，编译器据此验证；标注错了会报错，但不会"延长"什么

---

### 2. 标注语法

```rust
&i32            // 引用
&'a i32         // ⭐ 带显式生命周期标注的引用
&'a mut i32     // 带标注的可变引用
```

生命周期参数以 `'` 开头，通常用 `'a`、`'b`，声明在泛型参数列表里：

```rust
fn foo<'a>(x: &'a str) -> &'a str { x }
struct S<'a> { data: &'a str }
impl<'a> S<'a> { }
```

---

### 3. 函数中的生命周期

**为什么需要标注**：编译器无法确定返回的引用来自哪个参数。

```rust
fn longest(x: &str, y: &str) -> &str {          // ✘ missing lifetime specifier
    if x.len() > y.len() { x } else { y }
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {     // ✔
    if x.len() > y.len() { x } else { y }
}
```

⭐ **这个标注的含义**：返回的引用，其有效期是 `x` 和 `y` 两者中**较短的那个**。编译器据此在调用点做检查：

```rust
let s1 = String::from("long string");
let result;
{
    let s2 = String::from("short");
    result = longest(&s1, &s2);     // 'a 被推导为 s2 的（较短的）生命周期
}
println!("{result}");               // ✘ s2 已销毁，result 不再有效
```

**⭐ 如果返回值只依赖某一个参数，就该分开标注** —— 约束越精确，调用方越自由：

```rust
fn first<'a>(x: &'a str, y: &str) -> &'a str { x }      // ⭐ y 不参与
// 等价于 fn first<'a, 'b>(x: &'a str, y: &'b str) -> &'a str
```

**返回值必须来自参数**：

```rust
fn bad<'a>(x: &'a str) -> &'a str {
    let s = String::from("hi");
    &s                              // ✘ s 是局部变量，函数返回后就没了
}
fn good(x: &str) -> String {        // ✔ 返回所有权
    String::from("hi")
}
```

---

### 4. 生命周期省略规则（重点）

⭐ **绝大多数函数不用写生命周期**，因为编译器有三条省略规则（elision rules）：

> ⭐ **① 每个引用参数各自分配一个生命周期参数**
> ⭐ **② 如果只有一个输入生命周期参数，它被赋给所有输出**
> ⭐ **③ 如果参数里有 `&self` 或 `&mut self`，`self` 的生命周期被赋给所有输出**

按顺序套用，能推导出全部输出生命周期就不用写；否则必须手写。

```rust
// 规则 ②：唯一输入 → 输出
fn first_word(s: &str) -> &str { }
// 编译器看到的：fn first_word<'a>(s: &'a str) -> &'a str

// 规则 ③：有 &self → 用 self 的
impl Parser<'_> {
    fn peek(&self) -> &str { }
    // 编译器看到的：fn peek<'s>(&'s self) -> &'s str
}

// ✘ 两个输入引用，无 self，规则 ②③ 都不适用 → 必须手写
fn longest(x: &str, y: &str) -> &str { }
```

> ⭐ **规则 ③ 有个重要副作用**：方法返回的引用默认绑定在 `self` 上，所以只要返回值还活着，整个 `self` 就一直被借用（第九章踩坑 ⑥）。

---

### 5. 结构体中的生命周期

**结构体持有引用时，必须标注生命周期**，含义是：**结构体实例不能比它引用的数据活得更久**。

```rust
struct Parser<'a> {
    input: &'a str,                 // ⭐ 借用了外部数据
    pos: usize,
}

impl<'a> Parser<'a> {
    fn new(input: &'a str) -> Self { Parser { input, pos: 0 } }
    fn rest(&self) -> &str { &self.input[self.pos..] }       // 规则 ③ 自动推导
}

let text = String::from("hello");
let p = Parser::new(&text);
drop(text);                         // ✘ text 被 drop，但 p 还借着它
println!("{}", p.rest());
```

**⭐ 工程建议：能用所有权就别在结构体里存引用。**

| 方案 | 优点 | 缺点 |
| ---- | ---- | ---- |
| `struct S<'a> { s: &'a str }` | 零拷贝，最快 | ⭐ 生命周期会"传染"到所有用到它的地方 |
| ⭐`struct S { s: String }` | 简单，无生命周期负担 | 有一次拷贝 |
| ⭐`struct S { s: Arc<str> }` | 共享只读，克隆极廉价 | 有引用计数开销 |

> 引用型结构体适合**短生命周期的解析器、迭代器、视图**（如 `std::str::Split`）；长期存活的业务结构体（配置、实体、DTO）一律用 `String`/`Arc`，否则生命周期参数会一路传染到整个调用链，代码极难维护。

**匿名生命周期 `'_`**：不关心具体是什么时用它，让代码更简洁：

```rust
impl Parser<'_> { }                             // ⭐ 等价于 impl<'a> Parser<'a>
fn parse(s: &str) -> Parser<'_> { }             // 明确表示返回值借用了参数
```

---

### 6. `'static`（高频误解点）

`'static` 表示**生命周期覆盖整个程序运行期**，但它有**两种完全不同的含义**：

**① 作为引用的生命周期 `&'static T`**：数据在整个程序期间有效。

```rust
let s: &'static str = "hello";      // ⭐ 字符串字面量存在二进制只读段
static CONFIG: &str = "prod";
let leaked: &'static str = Box::leak(String::from("hi").into_boxed_str());
```

**② ⭐ 作为泛型约束 `T: 'static`**：意思是"**该类型不包含任何生命周期短于 `'static` 的引用**"。

⭐ **关键：所有拥有所有权的类型都满足 `T: 'static`**，包括 `String`、`Vec<T>`、`i32`：

```rust
fn needs_static<T: 'static>(x: T) { }

needs_static(String::from("hi"));   // ⭐ ✔ String 拥有数据，满足 T: 'static
needs_static(42);                   // ✔
let s = String::from("hi");
needs_static(&s);                   // ✘ &'a String 含有非 'static 的引用
```

> ⭐ **最常见的误解**：看到 `thread::spawn` 要求 `F: 'static` 就以为"必须是全局变量或字面量"。实际含义是"闭包不能借用任何可能提前销毁的外部数据"—— 所以用 `move` 把所有权移进闭包就满足了。

```rust
let data = vec![1, 2, 3];
thread::spawn(move || println!("{:?}", data));   // ✔ data 的所有权进了闭包
thread::spawn(|| println!("{:?}", data));        // ✘ 只是借用，不满足 'static
```

---

### 7. 生命周期约束与进阶

**生命周期约束**：

```rust
fn f<'a, 'b: 'a>(x: &'a str, y: &'b str) -> &'a str { y }
//        ^^^^^ ⭐ 'b: 'a 读作 "'b 至少和 'a 一样长"（b outlives a）

struct Wrapper<'a, T: 'a> { data: &'a T }       // T 中的引用必须活得比 'a 长
```

**协变（covariance）**：长生命周期的引用可以当短生命周期用，所以这样是合法的：

```rust
fn use_short<'a>(s: &'a str) { }
let s: &'static str = "hi";
use_short(s);                       // ✔ 'static 可以"降级"成任意 'a
```

**HRTB（高阶 trait 约束）**：用于"对任意生命周期都成立"的场景，写闭包参数时会遇到：

```rust
fn apply<F>(f: F) where F: for<'a> Fn(&'a str) -> &'a str { }
//                         ^^^^^^^ ⭐ 对所有可能的 'a 都成立
```

> 这两个在业务代码里很少手写，但看到 `for<'a>` 或 `'b: 'a` 时要知道是什么意思。

---

### 8. 高频踩坑点（重点）

**① 以为标注能"延长"生命周期**

```rust
fn bad<'a>() -> &'a str {
    let s = String::from("hi");
    &s                              // ✘ 标了 'a 也没用，s 就是会被 drop
}
```

⭐ 标注只是**描述关系**，不是**创造存活期**。返回引用必须来自参数或 `'static` 数据。

**② 结构体生命周期的"传染"**

```rust
struct Config<'a> { name: &'a str }
struct App<'a> { config: Config<'a> }           // ⭐ 被迫也加上 'a
struct Server<'a> { app: App<'a> }              // ⭐ 一路传染

// ✔ 顶层业务结构体直接用 String / Arc<str>
struct Config { name: String }
```

**③ `'static` 约束理解错**

```rust
fn spawn_task<F: FnOnce() + Send + 'static>(f: F) { }

let data = vec![1, 2, 3];
spawn_task(move || println!("{:?}", data));     // ⭐ ✔ move 后满足 'static
```

`T: 'static` 不要求"全局存在"，只要求"不借用外部数据"。

**④ 方法返回引用会锁住整个 `self`**

```rust
impl Cache {
    fn get(&self) -> &str { &self.value }       // 规则 ③：绑定 self 的生命周期
}
let v = cache.get();
cache.refresh();                    // ✘ 需要 &mut self，但 cache 还被借用
println!("{v}");
```

> 解法：返回 `String`（克隆）、返回 `Arc<str>`，或调整调用顺序让借用先结束。

**⑤ 迭代器/闭包返回引用**

```rust
fn names(&self) -> impl Iterator<Item = &str> { }       // ⭐ 需要写成 + '_
fn names(&self) -> impl Iterator<Item = &str> + '_ { self.items.iter().map(|i| i.name.as_str()) }
```

**⑥ 多个生命周期用了同一个参数**

```rust
fn parse<'a>(input: &'a str, sep: &'a str) -> Vec<&'a str> { }   // ⚠ 过度约束
fn parse<'a>(input: &'a str, sep: &str) -> Vec<&'a str> { }      // ✔ 返回值只来自 input
```

用同一个 `'a` 会让编译器把两者的生命周期取交集，调用方被迫传入活得一样久的参数。

**⑦ 常见报错的含义**

| 报错 | 含义 | 常见解法 |
| ---- | ---- | ---- |
| `missing lifetime specifier` | 返回引用但推不出来自哪 | 手写生命周期标注 |
| `does not live long enough` | 引用比被引用的数据活得久 | 调整作用域，或改用所有权 |
| `lifetime may not live long enough` | 约束不满足 | 补 `'b: 'a` 约束 |
| `borrowed value does not live long enough` | 典型的返回局部变量引用 | 返回 `String` 等所有权类型 |

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 优先靠省略规则，不手写 | 90% 的函数不需要标注 |
| 业务结构体用 `String` / `Arc<str>` | 不要在长期存活的类型里存引用 |
| 引用型结构体只用于解析器/视图 | 短生命周期、零拷贝的场景 |
| 返回值只依赖一个参数就分开标注 | 别图省事全用 `'a`，会过度约束调用方 |
| 不确定就用 `'_` | 比省略更明确，也比写 `'a` 简洁 |
| 线程/async 闭包用 `move` 满足 `'static` | 不是"必须用全局变量" |
| 方法尽量不返回内部引用 | 会锁住整个 `self`；必要时返回克隆或 `Arc` |
| 生命周期参数超过两个就重新设计 | 通常说明数据所有权划分有问题 |
| 需要"跨作用域共享"用 `Arc` | 而不是硬凑生命周期标注 |

---

### 10. 面试重点

**Q1：什么是生命周期？为什么需要它？**

生命周期是引用保持有效的作用域范围，唯一目的是**防止悬垂引用**。编译器在多数情况下能自动推导，但当函数返回引用、而编译器无法确定它来自哪个参数时，就需要程序员用 `'a` 显式标注引用之间的关系。生命周期是**纯编译期概念，不产生任何运行时代码**。

**Q2：生命周期标注会改变引用的存活时间吗？**

**不会**。标注只是向编译器**描述多个引用之间的关系约束**，编译器据此验证代码是否安全。给返回局部变量的引用加上 `'a` 并不能让它活得更久，照样编译报错。

**Q3：生命周期省略的三条规则是什么？**

① 每个引用参数各自分配一个生命周期参数；② 如果只有一个输入生命周期参数，就把它赋给所有输出；③ 如果参数中有 `&self` 或 `&mut self`，把 `self` 的生命周期赋给所有输出。按顺序套用后能确定所有输出生命周期，就不用手写；像 `fn longest(x: &str, y: &str) -> &str` 这种两个输入引用又没有 `self` 的情况，就必须显式标注。

**Q4：`'static` 是什么意思？`T: 'static` 呢？**

两种含义要分清：`&'static T` 表示**引用指向的数据在整个程序运行期都有效**（如字符串字面量、`static` 变量、`Box::leak` 的结果）；`T: 'static` 作为**泛型约束**时表示"**该类型不含任何非 `'static` 的引用**"，**所有拥有所有权的类型都满足它**，包括 `String`、`Vec<T>`。所以 `thread::spawn` 要求 `F: 'static` 并不意味着必须用全局变量，用 `move` 把数据所有权移进闭包就满足了。

**Q5：结构体里能存引用吗？有什么代价？**

能，但必须标注生命周期 `struct S<'a> { x: &'a str }`，语义是"结构体实例不能比它引用的数据活得久"。代价是生命周期参数会**传染**到所有包含它的类型和函数签名上，调用链一长就极难维护。所以引用型结构体只适合解析器、迭代器、视图这类短生命周期场景，业务实体应该用 `String` / `Arc<str>` 持有所有权。

**Q6：为什么方法返回引用后，就不能再调用 `&mut self` 的方法了？**

因为省略规则 ③：方法返回的引用默认绑定到 `self` 的生命周期上。只要这个返回值还在被使用，`self` 就一直处于被借用状态，此时再调用需要 `&mut self` 的方法会违反借用规则。解法是返回克隆的值或 `Arc`，或者调整调用顺序让借用先结束。

**Q7：`'b: 'a` 是什么意思？**

读作 "`'b` outlives `'a`"，即 **`'b` 至少和 `'a` 一样长**。用于表达多个生命周期之间的包含关系，比如函数需要把 `&'b` 的数据作为 `&'a` 返回时就要加这个约束。同理 `T: 'a` 表示类型 `T` 中的所有引用都至少活得和 `'a` 一样久。

---

## 十一、结构体与方法

### 1. 三种结构体

```rust
// ① ⭐ 具名字段结构体 —— 最常用
struct User {
    name: String,
    age: u32,
    active: bool,
}

// ② ⭐ 元组结构体 —— 字段无名，用下标访问
struct Point(i32, i32);
struct UserId(u64);                 // ⭐ newtype 模式（第二章）

// ③ 单元结构体 —— 没有任何字段
struct Marker;                      // ⭐ 零大小类型（ZST），占 0 字节
```

| 类型 | 适用场景 |
| ---- | -------- |
| 具名字段 | ⭐ 绝大多数业务实体 |
| 元组结构体 | 字段含义由位置明确时（`Point(x, y)`、`Rgb(r,g,b)`）；⭐ newtype 包装 |
| 单元结构体 | 只需要为某个类型实现 trait，本身不带数据（标记类型、策略对象） |

**⭐ Rust 的结构体没有继承**，代码复用靠**组合 + trait**（第十五章）。这是与 Java 面向对象最根本的差异。

---

### 2. 实例化与字段访问

```rust
let mut u = User {
    name: String::from("张三"),
    age: 20,
    active: true,
};                                  // ⭐ 必须初始化所有字段，没有默认值

u.name;                             // 访问
u.age = 21;                         // ⭐ 修改需要整个实例是 mut
```

**⭐ 字段初始化简写**：变量名和字段名相同时可省略：

```rust
fn new_user(name: String, age: u32) -> User {
    User { name, age, active: true }        // ⭐ 等价于 name: name, age: age
}
```

**⭐ 结构体更新语法**：从另一个实例取剩余字段：

```rust
let u2 = User {
    age: 30,
    ..u                             // ⭐ 其余字段来自 u，必须放最后
};
```

> ⚠ **坑**：`..u` 对**非 `Copy` 字段是移动**，所以上面 `u.name` 被移走了，`u` 整体不可再用（部分移动）。要保留 `u` 就写 `..u.clone()`。

**元组结构体与单元结构体**：

```rust
let p = Point(1, 2);
p.0;  p.1;                          // ⭐ 下标访问
let Point(x, y) = p;                // 解构

let id = UserId(1001);
id.0;                               // 取出内部值

let m = Marker;
std::mem::size_of::<Marker>();      // ⭐ 0 字节
```

**⭐ 可变性是整个实例的属性，不能只让某个字段可变**：

```rust
struct S { a: i32, b: i32 }
let mut s = S { a: 1, b: 2 };       // ⭐ 要么全部可变，要么全部不可变
```

需要"整体不可变但某字段可变"就得用内部可变性（`Cell`/`RefCell`，第十六章）。

---

### 3. 方法与关联函数

方法定义在 `impl` 块里，**一个类型可以有多个 `impl` 块**：

```rust
impl User {
    // ⭐ 关联函数：没有 self 参数，用 :: 调用，相当于 Java 的静态方法
    fn new(name: String, age: u32) -> Self {        // ⭐ Self = User
        User { name, age, active: true }
    }

    // ⭐ 关联常量
    const MAX_AGE: u32 = 150;

    // 方法：第一个参数是 self 的某种形式
    fn name(&self) -> &str { &self.name }               // ⭐ 只读，最常见
    fn birthday(&mut self) { self.age += 1; }           // 修改
    fn into_name(self) -> String { self.name }          // ⭐ 消耗自身
}

let u = User::new("张三".into(), 20);       // ⭐ 关联函数用 ::
u.name();                                   // ⭐ 方法用 .
User::MAX_AGE;
```

**⭐ 三种 `self` 的选择**（面试必考）：

| 写法 | 全称 | 含义 | 使用场景 | 命名惯例 |
| ---- | ---- | ---- | -------- | -------- |
| ⭐`&self` | `self: &Self` | 不可变借用 | ⭐ **默认选它**，读取数据 | `get_x`、`is_x`、`as_x` |
| `&mut self` | `self: &mut Self` | 可变借用 | 修改自身状态 | `set_x`、`push`、`update` |
| `self` | `self: Self` | ⭐ 取得所有权，调用后原对象失效 | 转换、消耗、Builder 的 `build()` | ⭐`into_x`、`build` |

```rust
let u = User::new(...);
let name = u.into_name();
println!("{:?}", u);                // ✘ u 已被消耗
```

**⭐ 方法调用的自动引用**：Rust 会自动加 `&`、`&mut` 或 `*`，所以不用像 C++ 那样区分 `.` 和 `->`：

```rust
u.name();                           // ⭐ 自动变成 User::name(&u)
(&u).name();                        // 等价，但没人这么写
```

**链式调用**：返回 `Self` 或 `&mut Self` 即可：

```rust
impl Builder {
    fn timeout(mut self, t: u32) -> Self { self.timeout = t; self }      // ⭐ 消费型
    fn retry(&mut self, n: u8) -> &mut Self { self.retry = n; self }     // 借用型
}
```

---

### 4. 常用 derive

`#[derive]` 让编译器自动实现常见 trait，**这是 Rust 结构体最常写的一行**：

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash, Default, PartialOrd, Ord)]
struct User { name: String, age: u32 }
```

| Derive | 作用 | 备注 |
| ------ | ---- | ---- |
| ⭐`Debug` | 支持 `{:?}` / `{:#?}` 打印 | ⭐ **几乎所有结构体都该加** |
| ⭐`Clone` | 提供 `.clone()` 深拷贝 | 所有字段都要 `Clone` |
| `Copy` | 赋值时按位复制 | ⭐ 必须同时 derive `Clone`；字段全是 `Copy` 才行 |
| ⭐`PartialEq` / `Eq` | 支持 `==` | 逐字段比较 |
| ⭐`Hash` | 可作 `HashMap` 的 key | 需同时有 `Eq` |
| `PartialOrd` / `Ord` | 支持 `<`、排序 | ⭐ 按**字段声明顺序**逐个比较 |
| ⭐`Default` | 提供 `Default::default()` | 数值 0、`String` 空、`Option` `None` |
| ⭐`Serialize` / `Deserialize` | JSON 序列化（`serde` crate） | 后端开发必备 |

```rust
println!("{:?}", u);                // User { name: "张三", age: 20 }
println!("{:#?}", u);               // ⭐ 多行美化格式，调试首选
```

---

### 5. Default 与 Builder 模式

**`Default`：字段多、大部分有合理默认值时**：

```rust
#[derive(Debug, Default)]
struct Config { host: String, port: u16, timeout: u32 }

let c = Config::default();
let c = Config { port: 8080, ..Default::default() };        // ⭐ 只改关心的字段

// 自定义默认值
impl Default for Config {
    fn default() -> Self {
        Config { host: "127.0.0.1".into(), port: 8080, timeout: 30 }
    }
}
```

**⭐ Builder 模式：Rust 没有默认参数和重载，参数多的构造一律用它**：

```rust
#[derive(Default)]
struct ClientBuilder { url: String, timeout: u32, retry: u8 }

impl ClientBuilder {
    fn url(mut self, v: &str) -> Self { self.url = v.into(); self }
    fn timeout(mut self, v: u32) -> Self { self.timeout = v; self }
    fn build(self) -> Result<Client, Error> {                   // ⭐ build 消耗 self
        if self.url.is_empty() { return Err(Error::MissingUrl); }
        Ok(Client { url: self.url, timeout: self.timeout })
    }
}

let client = ClientBuilder::default()
    .url("https://api.example.com")
    .timeout(60)
    .build()?;
```

> 实际项目直接用 `derive_builder` 或 `bon` crate 自动生成，不必手写。

---

### 6. 可见性

**Rust 默认一切私有**，需要显式 `pub`：

```rust
pub struct User {                   // ⭐ 结构体公开
    pub name: String,               // ⭐ 字段要单独公开！
    age: u32,                       // 私有：外部模块无法访问
}
```

| 修饰符 | 可见范围 |
| ------ | -------- |
| （无） | ⭐ 仅当前模块及其子模块 |
| ⭐`pub(crate)` | 整个 crate 内可见（内部 API 常用） |
| `pub(super)` | 父模块可见 |
| `pub` | 完全公开 |

> ⭐ **`pub struct` 不等于字段公开**。字段私有 + 提供 `pub fn` 访问器，就是 Rust 版的封装。另一个副作用：**字段私有的结构体，外部无法用字面量构造**，必须走你提供的 `new()`，这正是保证不变量的手段。

---

### 7. 内存布局

⭐ **Rust 默认（`repr(Rust)`）会自动重排字段以减少内存填充（padding）**，不保证字段顺序：

```rust
struct A { a: u8, b: u32, c: u8 }

std::mem::size_of::<A>();           // ⭐ 8（Rust 重排成 u32 + u8 + u8 + 2 padding）

#[repr(C)]
struct B { a: u8, b: u32, c: u8 }   // 保持声明顺序
std::mem::size_of::<B>();           // ⭐ 12（u8 + 3 padding + u32 + u8 + 3 padding）
```

| 属性 | 作用 |
| ---- | ---- |
| （默认）`repr(Rust)` | ⭐ 自动重排优化大小，顺序不保证 |
| ⭐`#[repr(C)]` | 按 C 的规则布局，⭐ **FFI 必须加**（第二十四章） |
| `#[repr(transparent)]` | 保证和唯一字段的布局完全相同，newtype 常用 |
| `#[repr(packed)]` | 取消对齐填充，⚠ 会产生未对齐访问，慎用 |

```rust
std::mem::size_of::<T>();           // 类型大小
std::mem::align_of::<T>();          // 对齐要求
```

> **优化技巧**：字段多的结构体，把大字段放前面（虽然 Rust 会自己重排，但便于阅读）；`Option<Box<T>>`、`Option<&T>` 等有**空指针优化**，和 `Box<T>` 一样大，不额外占空间。

---

### 8. 高频踩坑点（重点）

**① 结构体更新语法会移动字段**

```rust
let u2 = User { age: 30, ..u };
println!("{}", u.name);             // ✘ name 已被移走
println!("{}", u.age);              // ✔ age 是 Copy，还能用

let u2 = User { age: 30, ..u.clone() };     // ✔ 保留 u
```

**② 忘记 `#[derive(Debug)]`**

```rust
println!("{:?}", u);                // ✘ `User` doesn't implement `Debug`
```

> 规范：**所有结构体默认加 `#[derive(Debug)]`**，否则出问题时连日志都打不出来。

**③ `pub struct` 的字段依然私有**

```rust
// lib.rs
pub struct Config { pub host: String, port: u16 }

// 外部
Config { host: "x".into(), port: 80 };      // ✘ port 私有，无法构造
Config::new("x", 80);                       // ✔ 走公开的构造函数
```

**④ 方法误用 `self` 消耗所有权**

```rust
impl Order {
    fn total(self) -> f64 { self.amount }   // ✘ 调用一次订单就没了
    fn total(&self) -> f64 { self.amount }  // ✔
}
```

**⑤ 递归结构体必须用 `Box`**

```rust
struct Node { value: i32, next: Node }              // ✘ 大小无限递归
struct Node { value: i32, next: Option<Box<Node>> } // ✔ Box 是固定大小的指针
```

**⑥ 同时可变借用两个字段 —— 直接访问字段可以，通过方法不行**

```rust
let a = &mut s.a;
let b = &mut s.b;                   // ✔ 编译器能分辨不同字段

let a = s.get_a();                  // fn get_a(&mut self) -> &mut Vec<i32>
let b = s.get_b();                  // ✘ 方法借用的是整个 self
```

**⑦ 结构体太大时按值传递有拷贝开销**

```rust
fn process(config: Config) { }      // ⚠ 大结构体会整体拷贝到栈上
fn process(config: &Config) { }     // ✔ 只传 8 字节指针
```

**⑧ `new()` 不是关键字**

`new` 只是社区惯例的关联函数名，没有任何语言层面的特殊性。同样地，Rust **没有构造函数、没有析构函数语法**（清理逻辑写在 `Drop` 里），也**没有 `null`**（用 `Option`）。

**⑨ 字段声明顺序影响 `derive(Ord)` 的比较顺序**

```rust
#[derive(PartialOrd, Ord, PartialEq, Eq)]
struct Version { major: u32, minor: u32 }       // ⭐ 先比 major，再比 minor
```

调整字段顺序会**静默改变排序语义**，重构时要注意。

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 所有结构体加 `#[derive(Debug)]` | 便于日志和排查 |
| 需要时再加 `Clone` | 不要无脑全加，`Clone` 意味着可能有隐藏的深拷贝 |
| 字段默认私有 + 提供访问器 | `pub` 字段只用于纯数据 DTO |
| 方法默认 `&self` | `self` 只用于 `into_xxx` / `build` |
| 参数多用 Builder 或 `Default` + 更新语法 | Rust 没有默认参数和重载 |
| 大结构体按引用传递 | 避免栈拷贝 |
| 业务实体用 `String` 不用 `&'a str` | 避免生命周期传染（第十章） |
| 递归结构用 `Option<Box<T>>` | 或 `Rc`/`Arc` |
| FFI 结构体必须 `#[repr(C)]` | 默认布局不保证字段顺序 |
| 用组合代替继承 | 共享行为靠 trait，共享数据靠内嵌字段 |
| DTO 加 `#[derive(Serialize, Deserialize)]` | serde 是事实标准 |
| 命名：类型 `PascalCase`，字段/方法 `snake_case` | — |

---

### 10. 面试重点

**Q1：Rust 的结构体和 Java 的类有什么区别？**

① **没有继承**，代码复用靠组合 + trait；② **数据与行为分离**：字段在 `struct` 里，方法在 `impl` 块里，一个类型可以有多个 `impl` 块（也可以在不同 trait 的 `impl` 里分组）；③ **没有构造/析构函数语法**，`new()` 只是惯例，清理逻辑写在 `Drop` trait 里；④ **没有 `null`**，可空用 `Option<T>` 表达；⑤ 默认私有，且 `pub struct` 不代表字段公开。

**Q2：`&self`、`&mut self`、`self` 怎么选？**

`&self` 是默认选择，只读访问；`&mut self` 需要修改自身状态时用；`self` 会**取得所有权**，调用后原对象失效，只用于消耗型转换（命名惯例是 `into_xxx`）和 Builder 的 `build()`。选错最典型的后果是把 getter 写成 `self`，导致对象调用一次就没了。

**Q3：关联函数和方法有什么区别？**

关联函数没有 `self` 参数，属于类型本身，用 `Type::func()` 调用，相当于 Java 的静态方法，典型是 `new()`；方法第一个参数是某种形式的 `self`，属于实例，用 `instance.method()` 调用。方法调用时 Rust 会**自动加引用或解引用**来匹配签名，所以不需要像 C++ 那样区分 `.` 和 `->`。

**Q4：结构体更新语法 `..other` 有什么坑？**

对**非 `Copy` 字段是移动而非复制**，所以 `let b = S { x: 1, ..a };` 之后 `a` 会因为部分移动而整体不可用（未被移动的 `Copy` 字段仍可单独访问）。要保留原实例得写 `..a.clone()`。另外 `..other` 必须放在最后。

**Q5：Rust 结构体的内存布局是怎样的？**

默认 `repr(Rust)` 下编译器会**自动重排字段**以最小化对齐填充，**不保证声明顺序**。所以 `{u8, u32, u8}` 实际只占 8 字节，而加了 `#[repr(C)]` 保持声明顺序则要 12 字节。与 C 交互（FFI）时必须加 `#[repr(C)]`，否则布局不匹配。另外 `Option<Box<T>>` 有空指针优化，和 `Box<T>` 一样大。

**Q6：为什么递归结构体必须用 `Box`？**

因为编译器需要在编译期确定每个类型的大小，`struct Node { next: Node }` 的大小会无限递归。`Box<Node>` 是一个固定大小的堆指针（8 字节），把递归"打断"在指针处。链表通常写成 `Option<Box<Node>>`，其中 `None` 表示末尾，且因为空指针优化不额外占空间。

**Q7：Rust 没有继承，怎么复用代码？**

**组合 + trait**。共享**数据**靠把公共部分做成字段内嵌进来；共享**行为**靠 trait —— trait 可以提供默认方法实现，多个类型实现同一 trait 即可获得统一接口；需要运行时多态就用 `dyn Trait`（第十五章）。相比继承，这种方式避免了脆弱基类问题和菱形继承，也让"能力"和"数据结构"解耦。

---

## 十二、枚举与模式匹配

> ⭐ Rust 的枚举远比 Java/C 的强大：**每个变体都可以携带不同类型的数据**，本质是"代数数据类型"（tagged union）。`Option` 和 `Result` 就是它的两个实例，Rust 的错误处理、状态机、AST 全建立在此之上。

### 1. 枚举定义

```rust
// ① 无数据枚举（类似 C 的 enum）
enum Direction { North, South, East, West }

// ② ⭐ 带数据的枚举 —— Rust 的精髓
enum Message {
    Quit,                               // 无数据
    Move { x: i32, y: i32 },            // ⭐ 具名字段（像结构体）
    Write(String),                      // 元组变体
    Color(u8, u8, u8),                  // 多个值
}

// ③ 指定判别值（只有无数据枚举可以）
enum Status { Ok = 200, NotFound = 404, ServerError = 500 }
Status::NotFound as i32;                // ⭐ 404，用 as 取判别值
```

**⭐ 枚举取代了 Java 里的"继承 + instanceof"**：

```rust
// Java：抽象类 Shape + 三个子类 + instanceof 判断
enum Shape {
    Circle { r: f64 },
    Rect { w: f64, h: f64 },
    Triangle { a: f64, b: f64, c: f64 },
}

impl Shape {
    fn area(&self) -> f64 {
        match self {                    // ⭐ 编译器保证覆盖所有形状
            Shape::Circle { r } => std::f64::consts::PI * r * r,
            Shape::Rect { w, h } => w * h,
            Shape::Triangle { a, b, c } => {
                let s = (a + b + c) / 2.0;
                (s * (s - a) * (s - b) * (s - c)).sqrt()
            }
        }
    }
}
```

**`use` 简化写法**：

```rust
use Message::*;
let m = Write("hi".into());             // ⭐ 不用写 Message::Write
```

---

### 2. 标准库的两个核心枚举

```rust
enum Option<T> { None, Some(T) }                // ⭐ 取代 null（第十三章）
enum Result<T, E> { Ok(T), Err(E) }             // ⭐ 取代异常（第十三章）
```

⭐ 这两个太常用，已被自动导入（prelude），`Some`、`None`、`Ok`、`Err` 都可以直接写。

---

### 3. 枚举的方法与内存布局

枚举同样可以有 `impl` 块：

```rust
impl Status {
    fn is_success(&self) -> bool { matches!(self, Status::Ok) }
    fn code(&self) -> u16 { *self as u16 }
}
```

**⭐ 内存布局 = 判别标签（tag）+ 最大变体的空间**：

```rust
enum E { A(u8), B(u64) }
std::mem::size_of::<E>();               // ⭐ 16（8 字节 tag+padding + 8 字节数据）
```

**⭐ 空位优化（niche optimization）** 是 Rust 的一个杀手锏：当某个变体没有数据、且另一个变体存在"不可能的值"时，编译器用那个空位表示标签，**不额外占空间**：

```rust
size_of::<Box<i32>>();                  // 8
size_of::<Option<Box<i32>>>();          // ⭐ 8！用空指针表示 None，零开销
size_of::<Option<&T>>();                // ⭐ 8
size_of::<Option<NonZeroU32>>();        // ⭐ 4
size_of::<Option<i32>>();               // 8（i32 没有空位，需要额外 tag）
```

> ⭐ 这就是"**Rust 的 `Option` 比 null 更安全，却没有额外开销**"的技术原因，是面试的加分点。

**⚠ 变体大小差异过大会浪费内存**：

```rust
enum Msg { Ping, Data([u8; 1024]) }     // ⚠ 每个 Msg 都占 1KB+
enum Msg { Ping, Data(Box<[u8; 1024]>) }// ✔ 大变体装箱，枚举只占 16 字节
```

Clippy 的 `large_enum_variant` lint 会提醒这一点。

---

### 4. 模式的种类（全览）

`match`、`if let`、`while let`、`let`、函数参数、`for` 里用的都是**模式**：

```rust
match value {
    1                    => {}         // 字面量
    'a'..='z'            => {}         // ⭐ 范围（Rust 1.80+ 也支持 1..5 半开区间）
    1 | 2 | 3            => {}         // ⭐ 多模式
    n if n > 100         => {}         // ⭐ 匹配守卫
    n @ 10..=99          => {}         // ⭐ @ 绑定：既限定范围又拿到值
    _                    => {}         // 通配符（不绑定，值立即丢弃）
    other                => {}         // ⭐ 变量绑定（会匹配一切！）
}
```

**结构体解构**：

```rust
let Point { x, y } = p;                             // 全部字段
let Point { x, .. } = p;                            // ⭐ .. 忽略其余
match p {
    Point { x: 0, y: 0 } => "原点",
    Point { x, y: 0 }    => "X 轴上",
    Point { x, y }       => "其他",
}
```

**枚举与嵌套解构**：

```rust
match msg {
    Message::Move { x, y }      => println!("移动到 ({x}, {y})"),
    Message::Write(text)        => println!("{text}"),
    Message::Color(r, g, b)     => println!("#{r:02x}{g:02x}{b:02x}"),
    Message::Quit               => return,
}

// ⭐ 嵌套解构，任意深度
match result {
    Ok(Some(User { name, age: 18..=30 })) => println!("{name} 是青年用户"),
    Ok(Some(user))  => println!("{}", user.name),
    Ok(None)        => println!("用户不存在"),
    Err(e)          => eprintln!("{e}"),
}
```

**元组与数组/切片模式**：

```rust
let (a, b, ..) = tuple;

match slice {
    []                  => "空",
    [only]              => "一个元素",
    [first, .., last]   => "首尾",              // ⭐ 切片模式
    [head, rest @ ..]   => "头 + 剩余",          // ⭐ rest 绑定为子切片
}
```

---

### 5. match ergonomics（默认绑定模式）

⭐ **匹配引用时，绑定的变量自动成为引用**，不需要手写 `&` 或 `ref`：

```rust
let opt = Some(String::from("hi"));

match &opt {
    Some(s) => println!("{s}"),         // ⭐ s 自动是 &String，没有移动
    None => {}
}
println!("{:?}", opt);                  // ✔ opt 仍可用

// 老写法（Rust 2015），现在不需要了
match opt {
    Some(ref s) => println!("{s}"),     // ref 显式借用
    None => {}
}
```

**三种避免移动的写法**：

```rust
match &opt { Some(s) => ..., None => ... }          // ⭐ 最常用
match opt.as_ref() { Some(s) => ..., None => ... }  // ⭐ Option<&T>
if let Some(s) = &opt { }                           // ⭐ 单分支
```

---

### 6. 可反驳与不可反驳模式

| 类型 | 含义 | 使用位置 |
| ---- | ---- | -------- |
| **不可反驳**（irrefutable） | 一定能匹配成功 | `let`、函数参数、`for` |
| **可反驳**（refutable） | 可能匹配失败 | `match` 分支、`if let`、`while let` |

```rust
let (a, b) = (1, 2);                    // ✔ 元组解构一定成功
let Some(x) = opt;                      // ✘ 编译错误：可能是 None
let Some(x) = opt else { return; };     // ✔ let else 处理失败分支
if let Some(x) = opt { }                // ✔
```

---

### 7. 高频踩坑点（重点）

**① 变量绑定会匹配一切（最隐蔽的坑）**

```rust
let expected = 5;
match n {
    expected => println!("匹配"),       // ⭐ ✘ expected 是新绑定，匹配任何值！
    _ => println!("不匹配"),            // ⚠ 永远走不到，编译器会警告 unreachable
}

// ✔ 正确写法
match n {
    x if x == expected => println!("匹配"),     // 用守卫
    _ => {}
}
const EXPECTED: i32 = 5;
match n { EXPECTED => {}, _ => {} }             // ⭐ 或用 const（常量才是字面量模式）
```

**② `match` 会移动所有权**

```rust
let opt = Some(String::from("hi"));
match opt {
    Some(s) => println!("{s}"),         // ⭐ String 被移出
    None => {}
}
println!("{:?}", opt);                  // ✘ 已被部分移动

match &opt { }                          // ✔ 匹配引用
```

**③ 枚举变体不是类型**

```rust
fn f(m: Message::Write) { }             // ✘ 变体不是独立类型
fn f(m: Message) { }                    // ✔ 只能用枚举本身
```

> 这与 Java 的"子类是独立类型"完全不同。需要"只接受某一种变体"就得拆成独立结构体，或在函数内 `match` 后返回错误。

**④ `_` 通配符掩盖新增变体**

```rust
match status {
    Status::Ok => {}
    _ => {}                             // ⚠ 以后加了新状态不会有编译提示
}
```

> **规范：自己定义的枚举尽量列全分支。** 反过来，如果你是**库作者**，希望给枚举加变体时不破坏下游代码，就标记 `#[non_exhaustive]` —— 强制下游必须写 `_` 分支。

**⑤ 大变体浪费内存**

```rust
enum Event { Tick, Payload([u8; 4096]) }        // ⚠ 每个 Event 都占 4KB
enum Event { Tick, Payload(Box<[u8; 4096]>) }   // ✔
```

**⑥ 匹配字符串要用 `&str`**

```rust
match s {                               // s: String
    "a" => {},                          // ✘ 类型不匹配
}
match s.as_str() {                      // ✔
    "a" => {},
    _ => {}
}
```

**⑦ 守卫不参与穷尽性检查**

```rust
match n {
    x if x > 0 => {},
    x if x <= 0 => {},                  // ✘ 编译器不做数学推理，仍报"未覆盖所有情况"
    // 必须补一个 _ 或无守卫的分支
}
```

**⑧ `@` 绑定与解构混用**

```rust
match msg {
    m @ Message::Move { x: 0, .. } => println!("{:?}", m),      // ⭐ 整体绑定
    Message::Move { x: n @ 1..=10, y } => println!("{n}"),      // ⭐ 字段绑定
    _ => {}
}
```

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 用枚举表达"有限的几种情况" | 状态、消息、错误类型、AST 节点 |
| 自定义枚举的 `match` 列全分支 | 不写 `_`，让编译器守住新增变体 |
| 公开 API 的枚举加 `#[non_exhaustive]` | 便于将来扩展而不破坏下游 |
| 大变体用 `Box` 包装 | 避免整个枚举被撑大（Clippy 会提示） |
| 匹配时优先 `match &x` / `x.as_ref()` | 避免不必要的所有权移动 |
| 用常量而非变量做匹配值 | 变量名会变成"匹配一切"的绑定 |
| 单分支用 `if let`，布尔判断用 `matches!` | 不写完整 `match` |
| 枚举加 `#[derive(Debug, Clone, PartialEq)]` | 便于日志与测试断言 |
| 封闭集合用枚举，开放扩展用 trait | 类型集合会随插件增长时选 `dyn Trait` |
| 状态机用枚举 + `mem::replace` 转移状态 | 编译期保证状态迁移合法 |

---

### 9. 面试重点

**Q1：Rust 的枚举和 Java 的枚举有什么区别？**

Java 的枚举本质是一组单例常量对象，所有成员类型相同；Rust 的枚举是**代数数据类型（tagged union）**，**每个变体可以携带不同类型和数量的数据**（无数据、元组、具名字段都行）。所以 Rust 的枚举能表达 Java 里要用"抽象类 + 多个子类 + instanceof"才能表达的东西，而且配合 `match` 的穷尽性检查，编译期就能保证所有情况都被处理。

**Q2：枚举在内存中怎么表示？**

判别标签（tag）+ 最大变体所需空间，再按对齐补齐。所以 `enum E { A(u8), B(u64) }` 占 16 字节。特别地，Rust 有**空位优化（niche optimization）**：当存在"不可能的值"可以借用作标签时，不额外占空间 —— `Option<Box<T>>`、`Option<&T>`、`Option<NonZeroU32>` 都和内部类型**一样大**。这就是 `Option` 相比 null 既安全又零开销的原因。

**Q3：什么是可反驳模式和不可反驳模式？**

不可反驳模式一定匹配成功（如 `let (a, b) = tuple`），用于 `let`、函数参数、`for`；可反驳模式可能失败（如 `Some(x)`），只能用于 `match` 分支、`if let`、`while let`。所以 `let Some(x) = opt;` 编译不过，得写成 `if let` 或 `let Some(x) = opt else { return; }`。

**Q4：`match` 会转移所有权吗？怎么避免？**

会。匹配非 `Copy` 类型时，绑定的变量会把数据从原值中**移出**，导致原值部分移动后不可用。避免方法：匹配引用 `match &opt`、用 `opt.as_ref()` 得到 `Option<&T>`、或者绑定时用 `ref`（老写法）。Rust 2018 起有 **match ergonomics**，匹配引用时绑定会自动成为引用，所以 `match &opt { Some(s) => ... }` 中的 `s` 自动是 `&String`。

**Q5：下面这段代码为什么不对？**

```rust
let expected = 5;
match n { expected => "yes", _ => "no" }
```

因为 `expected` 在模式位置是一个**新的变量绑定**，会匹配任何值并遮蔽外层的 `expected`，导致 `_` 分支永远不可达（编译器会给 unreachable 警告）。要和变量比较必须用**匹配守卫** `x if x == expected`，或者把它定义成 `const`（常量在模式中才是字面量）。

**Q6：什么时候用枚举，什么时候用 trait 对象？**

**类型集合封闭且已知**（状态、消息、错误种类）用枚举：性能好（无虚表、无堆分配）、可穷尽匹配、易于序列化；**类型集合开放、需要下游扩展**（插件、驱动、处理器）用 `Box<dyn Trait>`：可以在不改动原代码的前提下新增实现，代价是动态分发和堆分配。库的公开枚举可以加 `#[non_exhaustive]`，在两者之间取得一定折中。

**Q7：为什么大枚举变体要用 `Box`？**

枚举的大小由**最大的变体**决定，所以 `enum E { Tick, Data([u8; 4096]) }` 里即使是 `Tick` 也要占 4KB 以上。把大变体改成 `Box<[u8; 4096]>` 后，枚举只需存一个 8 字节指针，整体大小降到 16 字节，代价是访问大变体时多一次堆间接寻址。Clippy 的 `large_enum_variant` lint 会自动提示。

---

## 十三、错误处理（Option 与 Result）

> ⭐ **Rust 没有异常，没有 `try/catch`，也没有 `null`。** 错误是**返回值**，用类型系统强制你处理 —— 这是 Rust 工程可靠性的最大来源，也是面试必问。

### 1. 错误处理哲学

Rust 把错误分成两类，用两套机制：

| 类型 | 机制 | 含义 | 对应 Java |
| ---- | ---- | ---- | --------- |
| ⭐ 可恢复错误 | `Result<T, E>` | 预期内的失败：文件不存在、网络超时、参数非法 | 受检异常 |
| ⭐ 不可恢复错误 | `panic!` | 程序 bug：数组越界、断言失败、不变量被破坏 | `RuntimeException` / `Error` |
| ⭐ 值可能不存在 | `Option<T>` | 空值：查不到、列表为空 | `null` / `Optional` |

**与异常机制的关键差异**：

```rust
fn read_config(path: &str) -> Result<Config, Error>     // ⭐ 错误写在签名里，调用方一眼可见
```

- ⭐ **错误是类型签名的一部分**，不会像 Java 的 `RuntimeException` 那样在任意位置意外抛出
- ⭐ **忽略错误会有编译警告**（`Result` 标了 `#[must_use]`）
- ⭐ **没有栈展开成本**，`Result` 就是普通枚举，`?` 编译后就是一个 `match`，零开销

---

### 2. Option\<T\>：取代 null

```rust
enum Option<T> { Some(T), None }
```

⭐ **Rust 没有 `null`，所有"可能没有值"必须用 `Option` 显式表达** —— 这从根本上消灭了空指针异常（Tony Hoare 口中的"十亿美元的错误"）。

```rust
let v = vec![1, 2, 3];
v.get(10);                              // Option<&i32> → None，而不是越界或返回 null
map.get("key");                         // Option<&V>
"abc".find('z');                        // Option<usize>
```

**常用方法（按使用频率排）**：

| 方法 | 作用 | 示例 |
| ---- | ---- | ---- |
| ⭐`unwrap_or(d)` | 为 `None` 时返回默认值 | `opt.unwrap_or(0)` |
| ⭐`unwrap_or_else(f)` | ⭐ 默认值需要计算时用（惰性） | `opt.unwrap_or_else(\|\| expensive())` |
| `unwrap_or_default()` | 返回类型的默认值 | `opt.unwrap_or_default()` |
| ⭐`map(f)` | `Some` 时转换内部值 | `opt.map(\|x\| x * 2)` |
| ⭐`and_then(f)` | ⭐ f 也返回 `Option` 时用（扁平化） | `opt.and_then(\|x\| x.checked_add(1))` |
| `filter(p)` | 不满足条件则变 `None` | `opt.filter(\|x\| *x > 0)` |
| ⭐`ok_or(e)` / `ok_or_else(f)` | ⭐ `Option` → `Result` | `opt.ok_or("未找到")?` |
| ⭐`as_ref()` / `as_mut()` | `&Option<T>` → `Option<&T>` | 避免移动所有权 |
| ⭐`take()` | 取出值并留下 `None` | `self.inner.take()` |
| `replace(v)` | 换成新值，返回旧值 | — |
| `is_some()` / `is_none()` | 判断 | — |
| ⭐`unwrap()` / `expect(msg)` | 取值，`None` 时 ⭐ **panic** | 只用于确定不会失败处 |

**⭐ `map` vs `and_then`**（高频混淆）：

```rust
let s = Some("42");
s.map(|x| x.parse::<i32>());            // Option<Result<i32, _>> ← 嵌套了
s.and_then(|x| x.parse::<i32>().ok());  // ⭐ Option<i32> ← 扁平化
```

规律：闭包返回**普通值**用 `map`，闭包返回 **`Option`** 用 `and_then`。

**链式组合**：

```rust
let name = config
    .get("user")                        // Option<&Value>
    .and_then(|v| v.as_str())           // Option<&str>
    .map(|s| s.trim())
    .filter(|s| !s.is_empty())
    .unwrap_or("匿名");                 // ⭐ 全程无 if 嵌套，无 null 检查
```

---

### 3. Result\<T, E\>：取代异常

```rust
enum Result<T, E> { Ok(T), Err(E) }
```

| 方法 | 作用 |
| ---- | ---- |
| ⭐`?` | ⭐ `Ok` 解包，`Err` 提前返回（见下节） |
| ⭐`map(f)` / `map_err(f)` | 转换成功值 / ⭐ 转换错误类型 |
| ⭐`and_then(f)` | 链式调用下一个可能失败的操作 |
| ⭐`unwrap_or(d)` / `unwrap_or_else(f)` | 出错时给默认值 |
| ⭐`ok()` | ⭐ `Result` → `Option`（丢弃错误信息） |
| `err()` | 取出错误 |
| `is_ok()` / `is_err()` | 判断 |
| ⭐`unwrap()` / `expect(msg)` | ⭐ 出错则 **panic** |

```rust
let n: i32 = "42".parse().unwrap_or(0);
let n = "abc".parse::<i32>().map_err(|e| format!("解析失败: {e}"))?;
let opt: Option<i32> = "42".parse().ok();               // ⭐ 不关心错误细节时
```

**⭐ 批量处理：`collect` 到 `Result`**（非常实用）：

```rust
let nums: Result<Vec<i32>, _> = vec!["1", "2", "x"]
    .iter().map(|s| s.parse::<i32>()).collect();        // ⭐ 遇到第一个错误就返回 Err

let nums: Vec<i32> = vec!["1", "2", "x"]
    .iter().filter_map(|s| s.parse().ok()).collect();   // ⭐ 跳过失败项 → [1, 2]
```

---

### 4. `?` 运算符（核心）

⭐ **`?` 是 Rust 错误处理的主力**：成功就取出值继续，失败就**提前返回错误**。

```rust
fn read_username() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("user.txt")?.read_to_string(&mut s)?;    // ⭐ 两个可能失败的操作
    Ok(s)
}

// 等价于
fn read_username() -> Result<String, io::Error> {
    let mut f = match File::open("user.txt") {
        Ok(f) => f,
        Err(e) => return Err(e),                        // ⭐ ? 就是这个 match
    };
    ...
}
```

**⭐ `?` 会自动做错误类型转换**（通过 `From` trait），这是它最强大的地方：

```rust
fn process() -> Result<(), AppError> {
    let s = std::fs::read_to_string("f.txt")?;          // io::Error → AppError
    let n: i32 = s.trim().parse()?;                     // ParseIntError → AppError
    Ok(())
}
// 前提：impl From<io::Error> for AppError 和 impl From<ParseIntError> for AppError
```

**`?` 也能用于 `Option`**（`None` 时提前返回 `None`）：

```rust
fn first_char(s: &str) -> Option<char> {
    let first = s.chars().next()?;                      // ⭐ None 则直接返回 None
    Some(first.to_ascii_uppercase())
}
```

**三条限制**：

```rust
// ① 函数返回类型必须是 Result / Option（不能混用）
fn f() -> i32 { let x = may_fail()?; }                  // ✘

// ② ⭐ main 要用 ? 必须改签名
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let cfg = load_config()?;
    Ok(())                                              // ⭐ 别忘了这行
}

// ③ 错误类型必须能通过 From 转换到返回类型
```

---

### 5. 自定义错误类型

**手写方式**（理解原理）：

```rust
use std::fmt;

#[derive(Debug)]
enum AppError {
    NotFound(String),
    Io(std::io::Error),
    Invalid { field: String, reason: String },
}

impl fmt::Display for AppError {                        // ⭐ 面向用户的消息
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::NotFound(k) => write!(f, "未找到: {k}"),
            AppError::Io(e) => write!(f, "IO 错误: {e}"),
            AppError::Invalid { field, reason } => write!(f, "{field} 非法: {reason}"),
        }
    }
}

impl std::error::Error for AppError {                   // ⭐ 标准错误 trait
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        match self {
            AppError::Io(e) => Some(e),                 // ⭐ 错误链，保留底层原因
            _ => None,
        }
    }
}

impl From<std::io::Error> for AppError {                // ⭐ 让 ? 能自动转换
    fn from(e: std::io::Error) -> Self { AppError::Io(e) }
}
```

**⭐ 实际项目用 `thiserror`，上面所有样板代码一个宏搞定**：

```rust
// Cargo.toml: thiserror = "2"
use thiserror::Error;

#[derive(Debug, Error)]
pub enum AppError {
    #[error("未找到: {0}")]
    NotFound(String),

    #[error("IO 错误")]
    Io(#[from] std::io::Error),                         // ⭐ #[from] 自动生成 From 实现

    #[error("{field} 非法: {reason}")]
    Invalid { field: String, reason: String },
}
```

---

### 6. thiserror vs anyhow（工程标准）

⭐ **这是 Rust 错误处理的行业惯例，面试常问**：

| 场景 | 用什么 | 原因 |
| ---- | ------ | ---- |
| ⭐ **库（library）** | `thiserror` | 调用方需要**按类型分别处理**错误，必须暴露具体的错误枚举 |
| ⭐ **应用（binary）** | `anyhow` | 大多数错误只需要**向上传播 + 打日志**，不需要区分类型 |

```rust
// ⭐ 应用层：anyhow，一个类型吃所有错误
use anyhow::{Context, Result, bail};

fn load() -> Result<Config> {                           // ⭐ anyhow::Result<T> = Result<T, anyhow::Error>
    let s = std::fs::read_to_string("app.toml")
        .context("读取配置文件失败")?;                   // ⭐ 附加上下文，错误链更好排查

    if s.is_empty() { bail!("配置文件为空"); }           // ⭐ 快速返回错误

    let cfg: Config = toml::from_str(&s)
        .with_context(|| format!("解析失败，内容长度 {}", s.len()))?;
    Ok(cfg)
}
```

`context()` 的价值在于最终打印时能看到完整的错误链：

```text
Error: 读取配置文件失败

Caused by:
    No such file or directory (os error 2)
```

> ⚠ **不要在库里用 `anyhow`** —— 下游拿到的是一个不透明的类型，无法针对具体错误做重试、降级等处理。

---

### 7. panic! 与 unwrap 家族

**什么时候该 panic**：程序遇到了**不该发生的状态**，继续执行只会造成更大伤害。

```rust
panic!("状态非法: {state}");
unreachable!("枚举已穷尽");             // ⭐ 标记逻辑上不可达
todo!();                                // ⭐ 占位，编译能过，跑到就 panic
unimplemented!();
assert!(n > 0, "n 必须为正");
assert_eq!(a, b);                       // ⭐ 测试里最常用
debug_assert!(...);                     // 仅 debug 生效，release 零开销
```

**⭐ `unwrap()` vs `expect()`**：

```rust
opt.unwrap();                           // ⚠ panic 信息只有一句 "called `unwrap()` on a `None` value"
opt.expect("配置必须在启动时加载");      // ⭐ 带上下文，出问题时能直接定位
```

> ⭐ **规范：能用 `expect` 就不用 `unwrap`，且 `expect` 的消息应该写"为什么这里不可能失败"，而不是"出错了"。**

**允许 `unwrap` 的场合**：

```rust
// ① 测试代码
#[test]
fn test() { assert_eq!(parse("1").unwrap(), 1); }

// ② 编译期就能确定不会失败的常量
let re = Regex::new(r"\d+").unwrap();               // 正则字面量写死的

// ③ 已经检查过的分支
if opt.is_some() { let v = opt.unwrap(); }          // ⭐ 但更好的写法是 if let
```

**panic 的行为可配置**：

```toml
[profile.release]
panic = "abort"         # ⭐ 直接终止，不做栈展开：二进制更小、更快，但无法 catch_unwind
```

---

### 8. 高频踩坑点（重点）

**① 到处 `unwrap()` —— 生产事故第一名**

```rust
let user = db.find(id).unwrap();         // ✘ 用户不存在 → 整个服务 panic
let user = db.find(id).ok_or(AppError::NotFound)?;      // ✔
```

> ⭐ Web 服务里一个 `unwrap` 可能让整个请求处理线程 panic。工程上应该在 CI 加 `#![deny(clippy::unwrap_used)]`（测试代码除外）。

**② `?` 报错 "the trait `From` is not implemented"**

```rust
fn f() -> Result<(), AppError> {
    let s = std::fs::read_to_string("x")?;      // ✘ io::Error 无法转成 AppError
}
// ✔ 加上 #[from] 或手写 impl From<io::Error> for AppError
// ✔ 或临时用 .map_err(|e| AppError::Io(e))?
```

**③ `unwrap_or` 的参数会被立即求值**

```rust
opt.unwrap_or(expensive_default());      // ✘ 无论是否为 None 都会执行
opt.unwrap_or_else(|| expensive_default());     // ⭐ ✔ 惰性求值
opt.unwrap_or_default();                        // ✔ 用类型默认值
```

同理 `ok_or` / `ok_or_else`、`map_or` / `map_or_else`。

**④ 忽略 `Result` 返回值**

```rust
write_file(path);                        // ⚠ warning: unused `Result` that must be used
let _ = write_file(path);                // ⭐ 显式表示"确实不关心"（要有理由）
write_file(path)?;                       // ✔ 正常写法
if let Err(e) = write_file(path) { warn!("写入失败: {e}"); }     // ✔ 至少记日志
```

**⑤ `map` 和 `and_then` 用错导致嵌套**

```rust
opt.map(|x| x.parse::<i32>().ok())       // Option<Option<i32>>
   .flatten();                           // ⭐ 或直接用 and_then
```

**⑥ 在 `Option` 上用 `?` 却写在返回 `Result` 的函数里**

```rust
fn f() -> Result<i32, E> {
    let x = opt?;                        // ✘ Option 的 ? 不能用在返回 Result 的函数
    let x = opt.ok_or(E::NotFound)?;     // ✔ 先转成 Result
}
```

**⑦ 错误类型缺少 `Send + Sync + 'static`**

跨线程传递或放进 `anyhow::Error` 时，错误类型必须满足这三个约束。自定义错误里如果混进了 `Rc` 或引用就会失败。

**⑧ 在库里 panic**

库函数遇到非法输入应该返回 `Result`，而不是 `panic!` —— 下游没有任何办法恢复。只有"违反了函数的前置契约"（如索引越界）才 panic，并在文档里用 `# Panics` 注明。

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 业务代码禁止 `unwrap()` | CI 加 `clippy::unwrap_used`，测试代码除外 |
| 必须解包时用 `expect("原因")` | 消息写"为什么这里不会失败" |
| ⭐ 库用 `thiserror`，应用用 `anyhow` | 库要暴露具体错误类型供下游分支处理 |
| 错误定义在模块级 | 每个模块一个 `Error` 枚举，用 `#[from]` 串联 |
| 传播时用 `?` + `context()` | 附加上下文，形成可读的错误链 |
| 不吞掉错误 | 至少 `warn!` 记录，不写空的 `Err(_) => {}` |
| `Option` 转 `Result` 用 `ok_or_else` | 错误对象需要构造时用惰性版本 |
| 边界处理错误，内部只传播 | HTTP handler / main 统一转成响应或退出码 |
| 不用 `Result<T, String>` | 字符串错误无法分类处理，只适合原型 |
| 错误信息不泄露敏感数据 | 对外返回错误码，细节只进日志 |
| `main` 返回 `Result` | 可以直接用 `?`，退出码自动为 1 |
| 空值一律 `Option`，禁止用魔法值 | 不用 `-1`、`""` 表示"没有" |

---

### 10. 面试重点

**Q1：Rust 的错误处理和 Java 的异常有什么区别？**

Rust **没有异常机制**，错误是**普通的返回值**（`Result<T, E>`）。区别：① 错误类型写在**函数签名**里，调用方一眼可见，不会在任意位置意外抛出；② 忽略 `Result` 会有编译警告（`#[must_use]`）；③ 没有栈展开的运行时成本，`?` 编译后就是一个 `match`，**零开销**；④ 没有"受检异常导致到处 try-catch"的问题，`?` 让传播只需一个字符。不可恢复的程序 bug 才用 `panic!`，类比 Java 的 `RuntimeException`。

**Q2：`Option` 是怎么解决空指针问题的？**

Rust 没有 `null`，任何"可能没有值"的情况必须用 `Option<T>` 显式表达。因为 `Option<T>` 和 `T` 是**不同类型**，你不可能把 `Option<T>` 当 `T` 直接用，编译器强制你先处理 `None` 分支。而且借助**空位优化**，`Option<&T>`、`Option<Box<T>>` 与内部类型**同样大小**，所以是零开销抽象。

**Q3：`?` 运算符做了什么？**

对 `Result`：`Ok(v)` 则取出 `v` 继续执行；`Err(e)` 则**立即 return**，并通过 `From::from(e)` 把错误**自动转换**成函数声明的错误类型。对 `Option`：`Some(v)` 取值，`None` 则 return `None`。限制是函数返回类型必须匹配（`Result` 的 `?` 只能用在返回 `Result` 的函数里），且需要有对应的 `From` 实现。`main` 想用 `?` 就得声明成 `fn main() -> Result<(), Box<dyn Error>>`。

**Q4：`thiserror` 和 `anyhow` 分别什么时候用？**

⭐ **库用 `thiserror`，应用用 `anyhow`**。库需要暴露**具体的错误枚举**，让调用方能针对不同错误做重试、降级、转换等处理；`thiserror` 用宏自动生成 `Display`、`Error`、`From` 实现，消除样板代码。应用层大多数错误只需向上传播并打日志，`anyhow::Error` 是一个能装下任何错误的统一类型，配合 `.context()` 附加上下文形成错误链，排查非常方便。**不要在库里用 `anyhow`**，会让下游失去分类处理的能力。

**Q5：什么时候该用 `panic!`，什么时候该用 `Result`？**

`Result` 用于**预期内的失败**：文件不存在、网络超时、用户输入非法 —— 调用方有能力处理。`panic!` 用于**程序 bug 或不变量被破坏**：数组越界、状态机进入非法状态、`unreachable!` 分支 —— 继续执行只会造成更大伤害。原则：**库函数几乎永远返回 `Result`**（下游无法从 panic 恢复），只有违反前置契约时才 panic，并在文档的 `# Panics` 段注明。

**Q6：`unwrap()` 和 `expect()` 的区别？生产代码能用吗？**

两者都在失败时 panic，区别是 `expect` 能附带自定义消息。生产业务代码应该**禁止 `unwrap`**（CI 用 `clippy::unwrap_used` 拦截）；确实能证明不会失败的地方用 `expect`，且消息要写清楚**"为什么这里不可能失败"**。允许的场景：测试代码、写死的常量（如编译期确定的正则）、程序启动阶段的初始化（失败就该直接退出）。

**Q7：`map` 和 `and_then` 有什么区别？**

`map` 的闭包返回**普通值**，结果被自动包回 `Some`/`Ok`；`and_then` 的闭包返回 **`Option`/`Result`**，结果不会嵌套（相当于 flatMap）。写 `opt.map(|x| x.parse().ok())` 会得到 `Option<Option<T>>`，用 `and_then` 才是 `Option<T>`。同理 `unwrap_or` 是立即求值、`unwrap_or_else` 是惰性求值，默认值需要计算时必须用后者。

---

## 十四、泛型

### 1. 泛型函数

泛型用来消除"逻辑相同、只有类型不同"的重复代码：

```rust
fn largest_i32(list: &[i32]) -> &i32 { }
fn largest_char(list: &[char]) -> &char { }         // ✘ 重复

fn largest<T: PartialOrd>(list: &[T]) -> &T {       // ⭐ 一份代码适用所有可比较类型
    let mut max = &list[0];
    for item in list {
        if item > max { max = item; }
    }
    max
}
```

**⭐ 泛型参数默认"什么都不能做"**，必须通过 **trait 约束**告诉编译器它具备哪些能力：

```rust
fn largest<T>(list: &[T]) -> &T {
    if item > max { }                   // ✘ 编译错误：T 可能不支持比较
}
fn largest<T: PartialOrd>(list: &[T]) -> &T { }     // ✔
```

> ⭐ 这是与 Java 泛型的重要区别：Java 的 `<T>` 默认可以调 `Object` 的方法，Rust 的 `T` 默认**一个方法都没有**。约束写多少，能力就有多少 —— 好处是**函数签名即契约**，编译器能在定义处就检查函数体是否合法。

---

### 2. 泛型结构体、枚举与方法

```rust
struct Point<T> { x: T, y: T }
struct Pair<T, U> { first: T, second: U }           // ⭐ 多个类型参数

enum Option<T> { Some(T), None }                    // 标准库就是泛型枚举
enum Result<T, E> { Ok(T), Err(E) }

// ⭐ impl 后面也要声明泛型参数
impl<T> Point<T> {
    fn new(x: T, y: T) -> Self { Point { x, y } }
    fn x(&self) -> &T { &self.x }
}

// ⭐ 可以只为特定类型实现方法
impl Point<f64> {
    fn distance(&self) -> f64 { (self.x.powi(2) + self.y.powi(2)).sqrt() }
}
// Point<i32> 没有 distance 方法

// ⭐ 也可以按约束实现（最常用）
impl<T: std::fmt::Display> Point<T> {
    fn show(&self) { println!("({}, {})", self.x, self.y); }
}
```

**方法级泛型**：方法可以有自己独立于结构体的泛型参数：

```rust
impl<T> Point<T> {
    fn mix<U>(self, other: Point<U>) -> Pair<T, U> {
        Pair { first: self.x, second: other.y }
    }
}
```

---

### 3. Trait 约束的写法

```rust
// ① 内联约束
fn f<T: Display + Clone>(x: T) { }

// ② ⭐ where 子句 —— 约束多时可读性更好
fn f<T, U>(x: T, y: U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{ }

// ③ impl Trait 参数（第五章）—— 最简洁，但无法在多处引用同一个 T
fn f(x: impl Display + Clone) { }
```

**常用约束速查**：

| 约束 | 含义 |
| ---- | ---- |
| ⭐`Clone` / `Copy` | 可克隆 / 可按位复制 |
| ⭐`Debug` / `Display` | 可 `{:?}` / `{}` 打印 |
| ⭐`PartialEq` / `Eq` | 可 `==` |
| ⭐`PartialOrd` / `Ord` | 可比较大小、可排序 |
| ⭐`Hash` | 可作 HashMap 的 key（需配合 `Eq`） |
| ⭐`Default` | 有默认值 |
| ⭐`Send` / `Sync` | 可跨线程移动 / 可跨线程共享（第二十二章） |
| ⭐`'static` | 不含短生命周期引用（第十章） |
| `Into<T>` / `From<T>` | 可转换（`impl Into<String>` 让参数既收 `&str` 又收 `String`） |
| `AsRef<str>` | 可借用为 `&str`，比 `Into<String>` 更省（不分配） |
| `Iterator<Item = T>` | 是迭代器且元素类型为 T |
| `Fn` / `FnMut` / `FnOnce` | 可调用（第五章） |

**⭐ 提升 API 易用性的常见写法**：

```rust
fn set_name(name: impl Into<String>) { }            // ⭐ 既能传 &str 也能传 String
fn open(path: impl AsRef<Path>) { }                 // ⭐ 标准库文件 API 的写法
```

---

### 4. 单态化与零成本抽象（重点）

⭐ **Rust 泛型在编译期做"单态化（monomorphization）"：为每个实际用到的具体类型生成一份专门的代码。**

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T { }

largest(&[1, 2, 3]);            // 编译器生成 largest_i32
largest(&['a', 'b']);           // 编译器生成 largest_char
```

**⭐ 与 Java 泛型的核心区别**：

| 对比项 | ⭐ Rust（单态化） | Java（类型擦除） |
| ------ | ---------------- | ---------------- |
| 实现时机 | ⭐ 编译期展开 | 编译期擦除，运行时全是 `Object` |
| 运行时开销 | ⭐ **零**，等同手写具体类型 | 装箱拆箱 + 类型转换 |
| 基本类型 | ⭐ `Vec<i32>` 直接存 `i32` | `List<Integer>` 必须装箱 |
| 运行时类型信息 | 有（可单态化的都是具体类型） | ⭐ 被擦除，`List<String>` 和 `List<Int>` 运行时同一类型 |
| 能否内联优化 | ⭐ 能，完全展开后可继续优化 | 受限 |
| 代价 | ⭐ **二进制体积增大**（code bloat）、编译变慢 | 二进制小 |

> ⭐ 这就是**零成本抽象**：`Vec<T>`、`Option<T>`、迭代器链写得再抽象，编译后的机器码和手写具体类型的代码一样快。面试中"Rust 泛型和 Java 泛型的区别"几乎必问。

**代价是编译产物膨胀**，泛型函数被大量不同类型实例化时（尤其是大函数），二进制会明显变大。工程上的做法是**把与泛型无关的主体逻辑抽成非泛型内部函数**：

```rust
pub fn read<P: AsRef<Path>>(path: P) -> Result<String> {
    fn inner(path: &Path) -> Result<String> { /* 大段逻辑，只编译一份 */ }
    inner(path.as_ref())                            // ⭐ 泛型外壳只做转换
}
```

---

### 5. const 泛型

⭐ **把"数值"当泛型参数**，主要用于按长度泛化数组（Rust 1.51+）：

```rust
fn sum<const N: usize>(arr: [i32; N]) -> i32 {      // ⭐ 任意长度的数组
    arr.iter().sum()
}
sum([1, 2, 3]);         // N = 3
sum([1, 2, 3, 4]);      // N = 4

struct Matrix<const R: usize, const C: usize> {
    data: [[f64; C]; R],
}

impl<const N: usize> Buffer<N> {
    const SIZE: usize = N;
}
```

在 const 泛型之前，标准库只能为长度 0~32 的数组手动实现 trait，这也是老代码里 `[T; 33]` 用不了很多方法的原因。

---

### 6. impl Trait 与泛型的关系

```rust
// 参数位置（APIT）：等价于匿名泛型参数
fn f(x: impl Display) { }
fn f<T: Display>(x: T) { }              // ⭐ 基本等价，但 impl Trait 无法用 turbofish 指定

// ⭐ 返回位置（RPIT）：隐藏具体类型
fn make() -> impl Iterator<Item = i32> {
    (0..10).filter(|x| x % 2 == 0)      // ⭐ 真实类型极其复杂，用 impl Trait 隐藏
}
```

| 方式 | 分发 | 特点 |
| ---- | ---- | ---- |
| ⭐`impl Trait` / 泛型 | 静态 | 单态化，零开销，⭐ 只能是**一种**具体类型 |
| `Box<dyn Trait>` | 动态 | 虚表调用，可以是**多种**类型（第十五章） |

---

### 7. 高频踩坑点（重点）

**① 忘记加约束**

```rust
fn print_all<T>(v: &[T]) {
    for x in v { println!("{x}"); }     // ✘ T 不一定实现了 Display
}
fn print_all<T: Display>(v: &[T]) { }   // ✔
```

> 报错 `` `T` doesn't implement `std::fmt::Display` `` 就是这个原因。养成习惯：**先写函数体，编译器会告诉你缺哪些约束**。

**② 泛型参数过多导致签名不可读**

```rust
fn process<T, U, V, W>(a: T, b: U, c: V) -> W
where T: Clone + Debug + Send, U: Into<String>, ... { }     // ⚠ 已经很难读
```

> 超过 2~3 个泛型参数时，考虑改成具体类型、trait 对象，或用结构体打包参数。

**③ 代码膨胀**

大泛型函数被十几种类型实例化时，编译时间和二进制体积都会明显上升。解法见第 4 节的"泛型外壳 + 非泛型内核"。

**④ `T` 与 `&T` 混淆**

```rust
fn largest<T: PartialOrd>(list: &[T]) -> T {
    list[0]                             // ✘ 不能从切片移出（除非 T: Copy）
}
fn largest<T: PartialOrd>(list: &[T]) -> &T { &list[0] }            // ✔
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T { list[0] }       // ✔
```

**⑤ 返回 `impl Trait` 只能有一种具体类型**

```rust
fn make(flag: bool) -> impl Fn() {
    if flag { || println!("a") } else { || println!("b") }          // ✘ 两个闭包类型不同
}
fn make(flag: bool) -> Box<dyn Fn()> {                              // ✔ 类型擦除
    if flag { Box::new(|| println!("a")) } else { Box::new(|| println!("b")) }
}
```

**⑥ 数值运算的泛型很麻烦**

```rust
fn sum<T: std::ops::Add<Output = T> + Copy + Default>(v: &[T]) -> T {
    v.iter().fold(T::default(), |a, b| a + *b)      // ⭐ 需要一堆约束
}
```

> 标准库没有"数值类型"这个统一 trait，实际项目用 `num-traits` crate 的 `Num`、`Zero`、`One` 等。

**⑦ Rust 不支持泛型特化和"重载"**

```rust
impl<T> Foo for T { }
impl Foo for i32 { }                    // ✘ conflicting implementations
```

稳定版 Rust 没有特化（specialization 仍是 nightly 特性）。需要按类型走不同逻辑时，改用 trait 的默认方法 + 具体实现覆盖。

**⑧ 类型推断不出来时用 turbofish**

```rust
let v = Vec::new();                     // ✘ 元素类型未知
let v = Vec::<i32>::new();              // ✔
let n = "42".parse::<i32>()?;           // ⭐ turbofish
let v: Vec<i32> = iter.collect();       // ✔ 或标注变量类型
```

---

### 8. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 约束只写必需的 | 多余约束会限制调用方 |
| 约束多时用 `where` 子句 | 签名更清晰 |
| 泛型参数不超过 2~3 个 | 更多说明抽象层次有问题 |
| 简单参数用 `impl Trait` | `fn f(x: impl Display)` 比 `<T: Display>` 简洁 |
| 字符串参数用 `impl Into<String>` / `AsRef<str>` | 提升 API 易用性 |
| 路径参数用 `impl AsRef<Path>` | 与标准库风格一致 |
| 大泛型函数抽出非泛型内核 | 控制二进制体积和编译时间 |
| 泛型参数命名：`T`/`U`/`K`/`V`/`E` | 有语义时用完整名如 `Item` |
| 编译期已知类型用泛型，运行时才知道用 `dyn` | 静态分发优先 |
| 数值泛型用 `num-traits` | 不要手写一长串运算约束 |

---

### 9. 面试重点

**Q1：Rust 泛型和 Java 泛型有什么区别？**

⭐ **最核心的区别是实现机制**：Rust 用**单态化**（monomorphization）—— 编译期为每个实际使用的具体类型生成一份专门的代码，运行时**零开销**，`Vec<i32>` 直接存 `i32` 无需装箱，并且可以进一步内联优化；Java 用**类型擦除** —— 编译后泛型信息被抹掉，运行时全是 `Object`，基本类型必须装箱，有额外的内存和转换开销。另一个区别是 Rust 的 `T` 默认**没有任何方法**，必须用 trait 约束显式声明能力，而 Java 的 `T` 默认可以调 `Object` 的方法。Rust 单态化的代价是**二进制体积增大**和编译变慢。

**Q2：什么是零成本抽象？**

"不用的东西不产生开销，用了的东西不会比手写更慢"。Rust 的泛型、迭代器、`Option`、trait 静态分发都符合这一点：泛型编译期单态化，迭代器链会被内联融合成单次循环，`Option<&T>` 靠空位优化和裸指针一样大。所以用高层抽象写出的代码，编译后的机器码和手写底层代码性能相当。

**Q3：为什么泛型参数必须加 trait 约束？**

因为 Rust 在**定义处**就检查泛型函数体的合法性（而不是像 C++ 模板那样等到实例化时才报错）。`T` 默认不具备任何能力，只有加了约束编译器才知道 `T` 支持哪些操作。好处是：函数签名就是完整契约、错误信息清晰、调用方一眼看出需要满足什么条件。

**Q4：什么是 const 泛型？**

把**常量值**（而不是类型）作为泛型参数，写作 `<const N: usize>`。主要用于对数组长度泛化：`fn sum<const N: usize>(arr: [i32; N])` 可以接受任意长度的数组。在它稳定之前（Rust 1.51 前），标准库只能为长度 0~32 的数组手工实现 trait，这就是老代码中长数组用不了很多方法的原因。

**Q5：泛型（静态分发）和 trait 对象（动态分发）怎么选？**

**编译期能确定类型就用泛型**：零开销、可内联、类型安全信息更完整；**运行时才知道类型、或需要在一个集合里放多种类型**就用 `Box<dyn Trait>`：有虚表调用和堆分配开销，但可以类型擦除。典型例子：函数参数用 `impl Fn`（静态），要把多个不同闭包放进 `Vec` 就必须 `Box<dyn Fn>`（动态）。

**Q6：泛型会导致什么问题？怎么优化？**

**代码膨胀（code bloat）**：每个具体类型都生成一份机器码，泛型函数体大、实例化类型多时，二进制体积和编译时间显著增加。优化手段是"**泛型外壳 + 非泛型内核**"：外层泛型函数只负责把参数转成统一的具体类型（如 `&Path`、`&str`、`&dyn Trait`），把大段主体逻辑放进一个非泛型的内部函数，这样主体只编译一份。标准库的 `File::open<P: AsRef<Path>>` 就是这么写的。

---

## 十五、Trait 特征

> ⭐ Trait 是 Rust 的接口机制，也是**面向对象能力的全部答案**：没有继承，多态、抽象、代码复用全靠它。

### 1. 定义与实现

```rust
trait Summary {
    fn summarize(&self) -> String;                      // 必须实现的方法

    fn preview(&self) -> String {                       // ⭐ 默认实现
        format!("{}...", &self.summarize()[..20])       // ⭐ 可以调用其他方法
    }

    const MAX_LEN: usize = 100;                         // ⭐ 关联常量
}

struct Article { title: String, body: String }

impl Summary for Article {
    fn summarize(&self) -> String { format!("《{}》", self.title) }
    // preview 用默认实现，也可以覆盖
}

article.summarize();
article.preview();
```

**与 Java 接口的对比**：

| 对比项 | Rust Trait | Java Interface |
| ------ | ---------- | -------------- |
| 默认方法 | ⭐ 支持 | Java 8+ 支持 |
| 为**已有类型**实现 | ⭐ **可以**（哪怕是标准库类型） | ✘ 必须改类的定义 |
| 静态分发 | ⭐ 支持（泛型，零开销） | ✘ 永远是动态分发 |
| 关联类型/常量 | ⭐ 支持 | ✘ |
| 运算符重载 | ⭐ 支持（`Add`、`Index` 等） | ✘ |
| 字段 | ✘ 不能有字段 | ✘ |

⭐ **"为已有类型实现 trait"是 Rust 最强大的特性之一**：

```rust
trait Doubler { fn double(&self) -> Self; }

impl Doubler for i32 {                  // ⭐ 给标准库的 i32 加方法
    fn double(&self) -> i32 { self * 2 }
}
5.double();                             // 10
```

---

### 2. Trait 作为参数与返回值

```rust
// ① ⭐ impl Trait：最简洁
fn notify(item: &impl Summary) { }

// ② 泛型：需要多处引用同一类型时
fn notify<T: Summary>(item: &T) { }
fn notify<T: Summary + Display>(a: &T, b: &T) { }       // ⭐ 强制两个参数同类型

// ③ ⭐ trait 对象：运行时才知道具体类型
fn notify(item: &dyn Summary) { }
fn notify(items: &[Box<dyn Summary>]) { }
```

---

### 3. Trait 对象与动态分发

⭐ **当集合里要放多种类型、或运行时才决定类型时，用 `dyn Trait`**：

```rust
let items: Vec<Box<dyn Summary>> = vec![
    Box::new(Article { .. }),
    Box::new(Tweet { .. }),                     // ⭐ 不同类型放进同一个 Vec
];

for item in &items {
    println!("{}", item.summarize());           // ⭐ 运行时查虚表决定调哪个实现
}
```

**⭐ `dyn Trait` 是"胖指针"**（16 字节）：

```text
&dyn Summary
┌──────────────┐
│ data ptr ────┼──► 实际对象数据
│ vtable ptr ──┼──► ┌────────────────┐
└──────────────┘    │ drop           │   ⭐ 虚函数表
                    │ size / align   │
                    │ summarize()    │
                    │ preview()      │
                    └────────────────┘
```

**`dyn Trait` 是"不定大小类型"（DST）**，不能直接使用，必须放在指针后面：

```rust
let s: dyn Summary;                     // ✘ 大小未知
let s: &dyn Summary = &article;         // ✔
let s: Box<dyn Summary> = Box::new(article);        // ⭐ 最常用
let s: Rc<dyn Summary> / Arc<dyn Summary>;          // 需要共享时
```

---

### 4. 静态分发 vs 动态分发（重点）

| 对比项 | ⭐ 静态分发（泛型 / `impl Trait`） | ⭐ 动态分发（`dyn Trait`） |
| ------ | -------------------------------- | ------------------------ |
| 决定时机 | 编译期 | 运行时 |
| 实现方式 | 单态化，每个类型一份代码 | 虚表（vtable）查找 |
| 性能 | ⭐ 零开销，可内联 | 一次指针跳转，⭐ **无法内联** |
| 二进制体积 | ⭐ 会膨胀 | ⭐ 小，只有一份代码 |
| 集合中混放多类型 | ✘ | ⭐ ✔ |
| 编译速度 | 慢 | 快 |
| 限制 | 一个位置只能一种类型 | ⭐ 必须满足**对象安全** |

**⭐ 选型原则**：

```rust
// ✔ 默认用静态分发
fn process(item: &impl Summary) { }

// ✔ 需要异构集合、插件式扩展、或要减小二进制时用动态分发
struct App { handlers: Vec<Box<dyn Handler>> }
```

> 实践中动态分发的开销通常不是瓶颈（一次间接跳转），**真正的代价是丧失内联优化机会**。热路径上的小函数用静态分发，架构层的可扩展点用动态分发。

---

### 5. 对象安全（dyn 兼容性）

⭐ **不是所有 trait 都能做成 `dyn Trait`**。要能做 trait 对象，必须满足**对象安全**（Rust 2024 起官方术语改为 "dyn 兼容性"）：

| 规则 | 原因 |
| ---- | ---- |
| ⭐ 方法不能返回 `Self` | 运行时不知道 `Self` 具体是什么、多大 |
| ⭐ 方法不能有泛型参数 | 泛型需要单态化，虚表里放不下无限多份 |
| ⭐ 不能有关联函数（无 `self` 的方法） | 没有实例就无法查虚表 |
| `Self: Sized` 约束的方法可豁免 | 这些方法不会进虚表 |

```rust
trait Bad {
    fn clone_me(&self) -> Self;         // ✘ 返回 Self
    fn generic<T>(&self, x: T);         // ✘ 泛型方法
    fn create() -> Self;                // ✘ 无 self
}
let b: Box<dyn Bad>;                    // ✘ the trait `Bad` cannot be made into an object

trait Good {
    fn run(&self);
    fn clone_me(&self) -> Self where Self: Sized;       // ⭐ 加约束后可豁免
}
```

> ⭐ 这就是 **`Clone` 不能直接做 trait 对象**的原因（`fn clone(&self) -> Self`）。需要"可克隆的 trait 对象"时用 `dyn-clone` crate，或自己定义 `fn box_clone(&self) -> Box<dyn Trait>`。

---

### 6. 关联类型 vs 泛型参数

⭐ **高频面试题**。两者都能表达"trait 里有个待定类型"，区别是**能否为同一类型实现多次**：

```rust
// ① 关联类型：一个类型对一个 trait 只能实现一次
trait Iterator {
    type Item;                          // ⭐ 关联类型
    fn next(&mut self) -> Option<Self::Item>;
}

impl Iterator for Counter {
    type Item = u32;                    // ⭐ Counter 的迭代元素类型唯一确定
    fn next(&mut self) -> Option<u32> { }
}

// ② 泛型参数：可以为同一类型实现多次
trait From<T> { fn from(x: T) -> Self; }

impl From<i32> for MyType { }           // ⭐ 同一个类型
impl From<String> for MyType { }        // ⭐ 实现了多次
```

| 场景 | 选择 |
| ---- | ---- |
| 类型由实现者**唯一确定** | ⭐ 关联类型（`Iterator::Item`、`Deref::Target`） |
| 需要为同一类型**多次实现** | ⭐ 泛型参数（`From<T>`、`Add<Rhs>`） |

> 关联类型的另一个好处：调用方不用写 `Iterator<Item = u32>` 这样冗长的约束，写 `T: Iterator` 就够了。

---

### 7. 孤儿规则与 newtype

⭐ **孤儿规则（orphan rule）：`impl Trait for Type` 中，trait 和 Type 至少有一个必须定义在当前 crate 里。**

```rust
impl Display for Vec<i32> { }           // ✘ 两者都是标准库的
impl MyTrait for Vec<i32> { }           // ✔ MyTrait 是本地的
impl Display for MyType { }             // ✔ MyType 是本地的
```

**目的**：防止两个 crate 给同一类型实现同一 trait 导致冲突（Java 没有这个问题是因为实现必须写在类内部）。

**⭐ 绕过办法：newtype 包装**（第二章）：

```rust
struct Wrapper(Vec<String>);            // ⭐ 本地类型

impl Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

// ⭐ 配合 Deref 让 Wrapper 用起来像 Vec
impl std::ops::Deref for Wrapper {
    type Target = Vec<String>;
    fn deref(&self) -> &Vec<String> { &self.0 }
}
```

---

### 8. supertrait 与 blanket impl

**supertrait（trait 继承）**：要求实现者必须先实现另一个 trait：

```rust
trait Person: Display {                 // ⭐ 实现 Person 前必须先实现 Display
    fn name(&self) -> String;
    fn greet(&self) { println!("你好，{self}"); }       // ⭐ 可以用 Display 的能力
}
```

**⭐ blanket impl（覆盖实现）**：为**所有满足条件的类型**批量实现 trait，标准库大量使用：

```rust
impl<T: Display> ToString for T { }             // ⭐ 所有能 Display 的都自动有 to_string()
impl<T, U: From<T>> Into<U> for T { }           // ⭐ 实现 From 就自动获得 Into

// 自己写：给所有实现了 Summary 的类型加日志能力
trait Loggable { fn log(&self); }
impl<T: Summary> Loggable for T {
    fn log(&self) { println!("[LOG] {}", self.summarize()); }
}
```

---

### 9. 标准库常用 trait 速查

| 分类 | Trait | 作用 |
| ---- | ----- | ---- |
| 格式化 | ⭐`Debug` / `Display` | `{:?}` / `{}` |
| 复制 | ⭐`Clone` / `Copy` | 深拷贝 / 按位复制 |
| 比较 | ⭐`PartialEq`/`Eq`/`PartialOrd`/`Ord` | `==`、`<`、排序 |
| 哈希 | ⭐`Hash` | 作 HashMap 的 key |
| 默认值 | ⭐`Default` | `T::default()` |
| 转换 | ⭐`From`/`Into`/`TryFrom`/`TryInto` | 类型转换（第三章） |
| | ⭐`AsRef<T>` / `Borrow<T>` | 廉价引用转换 |
| 迭代 | ⭐`Iterator` / `IntoIterator` | `for` 循环的基础（第十八章） |
| 智能指针 | ⭐`Deref` / `DerefMut` | 自动解引用（第十六章） |
| 资源 | ⭐`Drop` | 析构（第八章） |
| 运算符 | `Add`/`Sub`/`Mul`/`Index`/`Neg`/`Not` | 运算符重载（第三章） |
| 调用 | `Fn`/`FnMut`/`FnOnce` | 闭包（第五章） |
| 并发 | ⭐`Send` / `Sync` | 可跨线程传递 / 共享（第二十二章） |
| 标记 | `Sized` / `Copy` / `Send` / `Sync` | ⭐ 无方法，只表达性质 |
| 错误 | ⭐`Error` | 自定义错误（第十三章） |

**自动 derive 的 trait**：`Debug`、`Clone`、`Copy`、`PartialEq`、`Eq`、`PartialOrd`、`Ord`、`Hash`、`Default`。

---

### 10. 高频踩坑点（重点）

**① 违反孤儿规则**

```rust
impl Serialize for chrono::DateTime<Utc> { }        // ✘ 两者都是外部 crate 的
```

> 解法：newtype 包装，或用目标 crate 提供的适配机制（如 serde 的 `#[serde(with = "...")]`）。

**② `Vec<dyn Trait>` 编译不过**

```rust
let v: Vec<dyn Summary>;                // ✘ dyn Trait 大小未知
let v: Vec<Box<dyn Summary>>;           // ✔
```

**③ trait 对象不能有泛型方法**

```rust
trait Store { fn save<T: Serialize>(&self, item: T); }      // ✘ 无法做 trait 对象
trait Store { fn save(&self, item: &dyn Serialize); }       // ✔ 参数也改成 trait 对象
```

**④ 多个 trait 有同名方法 —— 完全限定语法**

```rust
trait Pilot { fn fly(&self); }
trait Wizard { fn fly(&self); }

person.fly();                           // ✘ 有歧义
Pilot::fly(&person);                    // ✔
<Person as Pilot>::fly(&person);        // ⭐ 完全限定语法，关联函数必须这么写
```

**⑤ `Box<dyn Trait>` 无法 `clone()`**

```rust
#[derive(Clone)]
struct App { handlers: Vec<Box<dyn Handler>> }      // ✘ Box<dyn Handler> 不是 Clone
```

> 解法：`Rc<dyn Handler>` / `Arc<dyn Handler>`（克隆引用计数），或用 `dyn-clone` crate。

**⑥ trait 里的 `async fn`**

```rust
trait Service {
    async fn call(&self) -> Result<()>;             // ⭐ Rust 1.75+ 支持
}
```

但这样的 trait **不能做 trait 对象**（返回的 `Future` 类型不确定）。需要 `Box<dyn Service>` 时用 `#[async_trait]` 宏：

```rust
#[async_trait::async_trait]
trait Service { async fn call(&self) -> Result<()>; }       // ⭐ 宏会把返回值包成 Box<dyn Future>
```

**⑦ 忘记 `use` trait 就调不到方法**

```rust
let s = 5.to_string();                  // ✔ ToString 在 prelude 里
use std::io::Write;                     // ⭐ 不 use 就没有 write_all 方法
file.write_all(b"hi")?;
```

> 报错 `` method not found in `File` `` 但方法明明存在时，多半是漏了 `use` 对应的 trait。

**⑧ 为引用实现 trait**

```rust
impl Summary for Article { }
fn f(x: impl Summary) { }
f(&article);                            // ✘ &Article 没实现 Summary
f(article);                             // ✔
// 或者：impl Summary for &Article { }
```

---

### 11. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 默认静态分发（泛型 / `impl Trait`） | 需要异构集合或插件扩展时才用 `dyn` |
| trait 定义保持小而聚焦 | 一个 trait 一个职责，便于组合 |
| 尽量提供默认实现 | 降低实现者的负担 |
| 公开 trait 注意对象安全 | 不返回 `Self`、不用泛型方法，否则下游无法 `dyn` |
| 用关联类型表达"唯一确定"的类型 | `Item`、`Target`、`Error` |
| 用泛型参数表达"可多次实现" | `From<T>`、`Add<Rhs>` |
| 为外部类型实现外部 trait 用 newtype | 孤儿规则绕不过去 |
| 需要克隆的 trait 对象用 `Arc<dyn T>` | 而不是 `Box<dyn T>` + 手写克隆 |
| trait 对象集合优先 `Arc<dyn T + Send + Sync>` | 多线程环境下必须补这两个约束 |
| 异步 trait 要做 `dyn` 时用 `#[async_trait]` | 原生 `async fn in trait` 不支持 `dyn` |
| 善用 blanket impl 减少重复 | 但会影响下游的实现自由度，公开库慎用 |

---

### 12. 面试重点

**Q1：Trait 和 Java 的接口有什么区别？**

① ⭐ **可以为已有类型（包括标准库类型）实现 trait**，不需要修改类型定义 —— Java 接口必须在类声明时 `implements`；② ⭐ **支持静态分发**，泛型约束下会单态化，零运行时开销，Java 接口永远是虚调用；③ 支持**关联类型和关联常量**；④ 可用于**运算符重载**（`Add`、`Index`）；⑤ 都支持默认方法，但 trait 不能有字段。此外 Rust **没有继承**，trait 是唯一的抽象与多态机制。

**Q2：静态分发和动态分发的区别？怎么选？**

静态分发（泛型 / `impl Trait`）在**编译期单态化**，为每个具体类型生成一份代码，零开销且可内联，但二进制会膨胀，且一个位置只能是一种类型；动态分发（`dyn Trait`）通过**虚表在运行时查找**方法，只有一份代码，可以在集合中混放多种类型，代价是一次指针跳转且**无法内联优化**。选型：默认用静态分发，需要异构集合、插件式扩展、或要控制二进制体积时用动态分发。

**Q3：什么是对象安全？为什么 `Clone` 不能做 trait 对象？**

要能写成 `dyn Trait`，trait 必须满足对象安全（Rust 2024 起称 "dyn 兼容性"）：**方法不能返回 `Self`**（运行时不知道具体类型和大小）、**不能有泛型方法**（泛型要单态化，虚表放不下）、**不能有无 `self` 的关联函数**（没实例就无法查虚表）。`Clone` 的签名是 `fn clone(&self) -> Self`，返回了 `Self`，所以不满足。需要可克隆的 trait 对象时用 `Rc`/`Arc` 共享，或 `dyn-clone` crate。

**Q4：关联类型和泛型参数有什么区别？什么时候用哪个？**

关联类型（`type Item;`）表示"这个类型由实现者**唯一确定**"，一个类型对一个 trait **只能实现一次**，典型是 `Iterator::Item`、`Deref::Target`；泛型参数（`trait From<T>`）允许**同一类型多次实现**不同参数的版本，典型是 `From<i32>`、`From<String>`。判断标准：如果"某个类型只该有一种答案"就用关联类型，同时调用方的约束也更简洁（写 `T: Iterator` 而不是 `T: Iterator<Item = U>`）。

**Q5：什么是孤儿规则？怎么绕过？**

`impl Trait for Type` 要求 **trait 和 Type 至少有一个定义在当前 crate**。目的是防止两个不同的 crate 给同一个类型实现同一个 trait 造成冲突（Rust 的实现可以写在类型定义之外，所以需要这条规则）。绕过办法是 **newtype 模式**：用本地的元组结构体包一层，再为这个本地类型实现外部 trait，通常配合 `Deref` 让它用起来像被包装的类型。

**Q6：`dyn Trait` 在内存里是什么样的？**

是一个**胖指针**，占 16 字节（64 位平台）：一个指向实际数据的指针，一个指向**虚表（vtable）**的指针。虚表里存着该具体类型的 `drop` 函数、大小、对齐以及所有 trait 方法的函数指针。因为 `dyn Trait` 本身是不定大小类型（DST），必须放在 `&`、`Box`、`Rc`、`Arc` 后面使用。

**Q7：什么是 blanket impl？**

为**所有满足某个约束的类型**批量实现 trait，例如标准库的 `impl<T: Display> ToString for T` —— 任何实现了 `Display` 的类型都自动获得 `to_string()`；`impl<T, U: From<T>> Into<U> for T` —— 实现 `From` 就自动获得 `Into`。它能大幅减少重复代码，但会占用下游为自己类型手动实现该 trait 的自由度（会与 blanket impl 冲突），所以公开库要谨慎使用。

**Q8：Rust 没有继承，怎么实现多态？**

两条路：① **静态多态**——泛型 + trait 约束，编译期单态化，零开销；② **动态多态**——`dyn Trait` 对象，运行时虚表分发，等价于 Java 的接口多态。代码复用方面：行为复用靠 trait 的**默认方法**，数据复用靠**组合**（把公共部分作为字段内嵌），接口层次靠 **supertrait**（`trait A: B`）。这套组合避免了继承的脆弱基类问题和菱形继承。

---

## 十六、智能指针（Box、Rc、RefCell）

> 智能指针 = **拥有数据的指针**，通过实现 `Deref`（像引用一样使用）和 `Drop`（自动清理）提供额外能力。第七、八章的 `String` 和 `Vec<T>` 其实就是智能指针。

### 1. Box\<T\>：最简单的智能指针

**把数据放到堆上，保持唯一所有权**，除了一次堆分配外**没有任何额外开销**（大小就是一个指针）：

```rust
let b = Box::new(5);
println!("{}", *b);                     // ⭐ 解引用
```

**⭐ 四个使用场景**：

```rust
// ① 递归类型：编译期需要确定大小（第十一章）
struct Node { value: i32, next: Option<Box<Node>> }

enum Expr {                             // ⭐ AST / 表达式树的标准写法
    Num(i32),
    Add(Box<Expr>, Box<Expr>),
}

// ② ⭐ trait 对象：不定大小类型必须装箱
let items: Vec<Box<dyn Summary>> = vec![Box::new(a), Box::new(b)];

// ③ 大对象避免栈拷贝（转移所有权时只拷贝 8 字节指针）
let big = Box::new([0u8; 1_000_000]);

// ④ 大枚举变体装箱（第十二章）
enum Msg { Ping, Data(Box<[u8; 4096]>) }
```

```rust
let v = *b;                             // 取出所有权（Box 被消耗）
let leaked: &'static mut T = Box::leak(b);      // ⭐ 故意泄漏，换取 'static 生命周期
```

---

### 2. Deref 与自动解引用

实现 `Deref` 就能让自定义类型"用起来像引用"：

```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> Deref for MyBox<T> {
    type Target = T;                    // ⭐ 关联类型
    fn deref(&self) -> &T { &self.0 }
}

let b = MyBox(5);
*b;                                     // ⭐ 等价于 *(b.deref())
```

**⭐ Deref 强制转换（第三章）**：函数参数类型不匹配时编译器自动调用 `deref`：

```text
&String  → &str          &Vec<T>  → &[T]
&Box<T>  → &T            &Rc<T>   → &T          （可多级连续转换）
```

这就是"`&String` 能传给 `fn f(s: &str)`"、"`Box<T>` 能直接调用 `T` 的方法"的原理。

---

### 3. Rc\<T\> / Arc\<T\>：共享所有权

⭐ **当一份数据需要多个所有者时用引用计数**（图、树、缓存、配置共享）：

```rust
use std::rc::Rc;

let a = Rc::new(vec![1, 2, 3]);
let b = Rc::clone(&a);                  // ⭐ 只增加计数，不复制数据
let c = a.clone();                      // 等价，但 Rc::clone 写法更能表明意图

Rc::strong_count(&a);                   // 3
drop(b);                                // 计数减 1；归 0 时数据才真正释放
```

| 对比项 | ⭐`Rc<T>` | ⭐`Arc<T>` |
| ------ | -------- | --------- |
| 计数方式 | 普通整数 | ⭐ 原子操作（atomic） |
| 线程安全 | ✘ 单线程 | ⭐ ✔ 可跨线程 |
| 性能 | 快 | 略慢（原子操作有开销） |
| 使用场景 | 单线程树/图结构 | ⭐ 多线程共享（第二十二章） |

**⭐ `Rc`/`Arc` 里的数据是只读的**（因为可能有多个所有者，允许改就违反借用规则）：

```rust
let a = Rc::new(vec![1]);
a.push(2);                              // ✘ 不能通过 Rc 修改
Rc::get_mut(&mut a);                    // ⭐ 只有计数为 1 时才返回 Some(&mut T)
```

要"共享 + 可变"必须配合内部可变性 → `Rc<RefCell<T>>` / `Arc<Mutex<T>>`。

---

### 4. Cell\<T\> 与 RefCell\<T\>：内部可变性

⭐ **内部可变性：持有 `&T`（不可变引用）也能修改内部数据**，把借用检查从**编译期**推迟到**运行期**。

| 类型 | 适用 | 检查时机 | 违规后果 | 开销 |
| ---- | ---- | -------- | -------- | ---- |
| ⭐`Cell<T>` | `T: Copy` 的简单值 | ⭐ 无需检查（整体读写） | — | 零 |
| ⭐`RefCell<T>` | 任意类型 | ⭐ **运行期** | ⭐ **panic** | 一个计数器 |
| ⭐`Mutex<T>` | 多线程 | 运行期 | 阻塞等待 | 锁开销 |

```rust
use std::cell::{Cell, RefCell};

// Cell：整体替换，不给出引用，所以永远安全
struct Counter { count: Cell<u32> }
let c = Counter { count: Cell::new(0) };
c.count.set(c.count.get() + 1);         // ⭐ 只有 &c 也能改

// RefCell：运行时借用检查
let data = RefCell::new(vec![1, 2, 3]);
data.borrow_mut().push(4);              // ⭐ 可变借用
println!("{:?}", data.borrow());        // 不可变借用

// ✘ 违反借用规则 → 运行时 panic
let b1 = data.borrow_mut();
let b2 = data.borrow_mut();             // ⭐ panic: already mutably borrowed

data.try_borrow_mut();                  // ⭐ 返回 Result，不 panic
```

> ⭐ `borrow()` / `borrow_mut()` 返回的是**守卫对象**（`Ref` / `RefMut`），它们在 drop 时才释放借用 —— 所以要**尽量缩短守卫的存活范围**，别把它绑定到长生命周期的变量上。

---

### 5. 组合模式：共享 + 可变

⭐ **这两个组合是实际项目里最常见的写法**：

```rust
// ① 单线程共享可变
use std::{rc::Rc, cell::RefCell};
let shared = Rc::new(RefCell::new(vec![1, 2]));
let clone = Rc::clone(&shared);
clone.borrow_mut().push(3);
println!("{:?}", shared.borrow());      // [1, 2, 3]

// ② ⭐ 多线程共享可变（第二十二章）
use std::sync::{Arc, Mutex};
let shared = Arc::new(Mutex::new(0));
let clone = Arc::clone(&shared);
thread::spawn(move || { *clone.lock().unwrap() += 1; });
```

**⭐ 记忆口诀**：`Rc`/`Arc` 管**共享**（多个所有者），`RefCell`/`Mutex` 管**可变**（绕过借用检查），两者正交、按需组合。

---

### 6. Weak\<T\> 与循环引用

⭐ **Rust 保证内存安全，但不保证不泄漏** —— `Rc` 循环引用就会造成泄漏：

```text
parent ──Rc──► child
   ▲            │
   └────Rc──────┘        ⭐ 双方计数永远不为 0，内存永不释放
```

**解法：一个方向用 `Weak`（弱引用，不增加强计数）**：

```rust
use std::rc::{Rc, Weak};

struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,        // ⭐ 指向父节点用 Weak
    children: RefCell<Vec<Rc<Node>>>,   // ⭐ 指向子节点用 Rc
}

let parent = Rc::new(Node { .. });
let child = Rc::new(Node { .. });

*child.parent.borrow_mut() = Rc::downgrade(&parent);        // ⭐ Rc → Weak
parent.children.borrow_mut().push(Rc::clone(&child));

// ⭐ 使用弱引用必须先 upgrade（可能已被释放）
if let Some(p) = child.parent.borrow().upgrade() {          // Option<Rc<Node>>
    println!("{}", p.value);
}
```

**规律**：**"拥有"关系用 `Rc`（强引用），"反向引用/观察"关系用 `Weak`**。父子树、观察者模式、双向链表都适用。

---

### 7. 选型总表（重点）

| 需求 | 用什么 |
| ---- | ------ |
| 数据放堆上、唯一所有权 | ⭐`Box<T>` |
| 递归类型 / trait 对象 | ⭐`Box<T>` |
| 单线程多个所有者（只读） | ⭐`Rc<T>` |
| 多线程多个所有者（只读） | ⭐`Arc<T>` |
| 单线程共享 + 可变 | ⭐`Rc<RefCell<T>>` |
| 多线程共享 + 可变 | ⭐`Arc<Mutex<T>>` / `Arc<RwLock<T>>` |
| 只读全局共享大对象 | ⭐`Arc<T>` / `&'static` |
| 简单 `Copy` 值的内部可变 | ⭐`Cell<T>` |
| 多线程计数器 | ⭐`Arc<AtomicU64>` |
| 打破循环引用 | ⭐`Weak<T>` |
| "可能改也可能不改" | ⭐`Cow<'a, T>`（第六章） |
| 自引用 / 异步 Future | `Pin<Box<T>>`（第二十三章） |

---

### 8. 高频踩坑点（重点）

**① `RefCell` 的运行时 panic**

```rust
let d = RefCell::new(vec![1]);
let b = d.borrow();
d.borrow_mut().push(2);                 // ⭐ panic: already borrowed: BorrowMutError
```

> ⭐ 这是把编译期错误换成了**运行时崩溃**。规避：缩小 `borrow()` 守卫的作用域、必要时用 `try_borrow_mut()`。**能用编译期检查就不要用 `RefCell`。**

```rust
{
    let b = d.borrow();
    println!("{:?}", b);
}                                       // ⭐ 守卫在此释放
d.borrow_mut().push(2);                 // ✔
```

**② 循环引用导致内存泄漏**

```rust
a.next = Some(Rc::clone(&b));
b.next = Some(Rc::clone(&a));           // ⚠ 泄漏，计数永不归零
```

Rust 只保证**安全**，不保证**不泄漏**（`mem::forget`、`Box::leak` 都是安全函数）。反向引用必须用 `Weak`。

**③ `Rc` 不能跨线程**

```rust
let a = Rc::new(1);
thread::spawn(move || println!("{a}"));         // ✘ `Rc` cannot be sent between threads
let a = Arc::new(1);                            // ✔
```

> 报错里出现 `Rc<...> cannot be sent between threads safely` 时，改成 `Arc` 即可。

**④ 通过 `Rc` 无法直接修改数据**

```rust
let a = Rc::new(RefCell::new(vec![]));  // ✔ 需要修改就套 RefCell
```

**⑤ 过度使用 `Rc<RefCell<T>>`（架构坏味道）**

从 Java/Python 转来的人容易把所有对象都包成 `Rc<RefCell<T>>` 模拟"引用语义"。这会带来运行时 panic 风险、性能损耗，且丧失了 Rust 的编译期保证。

> ⭐ **先考虑：能不能用所有权 + 借用表达？能不能用索引（`Vec` 下标）代替对象引用？** 图、树等结构在 Rust 里的惯用做法是 **arena + 索引**（`Vec<Node>` + `usize` 下标），比 `Rc<RefCell<>>` 更快也更简单。

**⑥ `Box<dyn Trait>` 不能 `Clone`**

```rust
#[derive(Clone)]
struct S { h: Box<dyn Handler> }        // ✘
struct S { h: Arc<dyn Handler> }        // ✔ Arc 克隆的是计数
```

**⑦ 守卫跨 `await` 持有（异步场景）**

```rust
let guard = data.borrow_mut();          // 或 mutex.lock()
some_async_fn().await;                  // ⚠ 守卫跨越 await，容易死锁/panic
```

> 异步代码里必须在 `.await` 之前释放守卫（用 `{}` 包起来或显式 `drop`），详见第二十三章。

**⑧ `Rc::clone(&a)` 的写法惯例**

```rust
let b = a.clone();                      // ⚠ 看起来像深拷贝
let b = Rc::clone(&a);                  // ⭐ 明确表示"只是增加计数"
```

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 优先用所有权 + 借用 | 智能指针是解决特定问题的工具，不是默认选择 |
| 单线程用 `Rc`，多线程用 `Arc` | 不要无脑全用 `Arc`（原子操作有开销） |
| 共享用 `Rc`/`Arc`，可变用 `RefCell`/`Mutex` | 两者正交，按需组合 |
| `Rc::clone(&x)` 而非 `x.clone()` | 明确表达"增加计数"的意图 |
| 反向引用用 `Weak` | 防止循环引用泄漏 |
| `borrow()` 守卫作用域最小化 | 用 `{}` 包住，避免运行时 panic |
| 异步代码不跨 `await` 持有守卫 | 会导致死锁 |
| 图/树优先考虑 arena + 索引 | 比 `Rc<RefCell<>>` 更快、更简单、无 panic 风险 |
| trait 对象集合用 `Arc<dyn T + Send + Sync>` | 便于克隆和跨线程 |
| 大的只读配置用 `Arc` 全局共享 | 而不是每个模块 `clone` 一份 |

---

### 10. 面试重点

**Q1：什么是智能指针？Rust 有哪些？**

拥有数据所有权、并实现了 `Deref`（可以像引用一样使用）和 `Drop`（离开作用域自动清理）的类型。常见的有：`Box<T>`（堆分配、唯一所有权）、`Rc<T>`/`Arc<T>`（引用计数共享所有权）、`RefCell<T>`/`Cell<T>`（内部可变性）、`Mutex<T>`/`RwLock<T>`（线程安全的内部可变性）、`Cow<T>`（写时克隆）。其实 `String` 和 `Vec<T>` 也是智能指针。

**Q2：`Box<T>` 有什么用？**

① **递归类型**：编译期必须确定大小，`Box` 提供固定 8 字节的间接层（链表、AST）；② **trait 对象**：`dyn Trait` 是不定大小类型，必须放在 `Box`/`&`/`Rc` 后面；③ **大对象转移所有权**时只拷贝指针而非整块数据；④ **大枚举变体**装箱以缩小枚举整体大小。除一次堆分配外没有任何运行时开销。

**Q3：`Rc` 和 `Arc` 的区别？**

`Rc` 用普通整数做引用计数，**不是线程安全的**（没有实现 `Send`/`Sync`），只能单线程用；`Arc` 用**原子操作**维护计数，可以跨线程共享，代价是原子操作比普通加减慢。两者都只提供**共享只读**访问，需要修改必须配合内部可变性：单线程 `Rc<RefCell<T>>`，多线程 `Arc<Mutex<T>>`。

**Q4：什么是内部可变性？`Cell` 和 `RefCell` 的区别？**

内部可变性是指"通过不可变引用 `&T` 修改内部数据"的能力，本质是把借用检查从**编译期推迟到运行期**。`Cell<T>` 只支持 `Copy` 类型，通过整体 `get`/`set` 替换值，**从不交出引用**，所以无需运行时检查、零开销；`RefCell<T>` 支持任意类型，`borrow()`/`borrow_mut()` 会在**运行时**维护借用计数，**违反借用规则直接 panic**。多线程版本是 `Mutex`/`RwLock`（违规时是阻塞而非 panic）。

**Q5：`Rc<RefCell<T>>` 是什么组合？**

`Rc` 提供**多个所有者**（共享），`RefCell` 提供**通过不可变引用修改**（可变），组合起来就是"单线程下多方共享同一份可变数据"。多线程对应版本是 `Arc<Mutex<T>>`。两者职责正交，按需搭配。

**Q6：Rust 会内存泄漏吗？**

会。Rust 保证**内存安全**（无悬垂指针、无二次释放、无数据竞争），但**不保证不泄漏** —— 泄漏是安全的行为，`mem::forget`、`Box::leak` 都是安全函数。最典型的泄漏是 **`Rc` 循环引用**：两个节点互相持有 `Rc`，强引用计数永远不为 0。解法是把其中一个方向改成 `Weak`（弱引用不增加强计数，使用时通过 `upgrade()` 拿到 `Option<Rc<T>>`）。

**Q7：什么时候用 `Weak`？**

表达"**引用但不拥有**"的关系时：树结构中子节点指向父节点、观察者模式中被观察者持有观察者、缓存中指向可能已被回收的对象。规律是"拥有"方向用 `Rc`，"反向/观察"方向用 `Weak`。`Weak` 不增加强引用计数，所以不会阻止对象释放，使用前必须 `upgrade()` 检查目标是否还活着。

**Q8：`Rc<RefCell<T>>` 满天飞是好设计吗？**

不是，通常说明数据所有权关系没设计清楚。它带来三个代价：运行时 panic 风险（借用冲突）、引用计数与动态检查的性能开销、丧失编译期保证。应该先尝试用**所有权 + 借用**表达；图、树这类结构在 Rust 中的惯用做法是 **arena 模式**——把所有节点放在一个 `Vec` 里，节点之间用 `usize` 索引互相引用，既避开了循环引用，性能和缓存局部性也更好。

---

## 十七、集合类型（Vec、HashMap、HashSet）

> `Vec<T>` 已在第七章详述，本章重点是 Map、Set 和其他容器。

### 1. 集合总览与选型

| 集合 | 说明 | 有序性 | 典型场景 |
| ---- | ---- | ------ | -------- |
| ⭐`Vec<T>` | 动态数组 | 插入顺序 | ⭐ 默认选择 |
| ⭐`HashMap<K,V>` | 哈希表 | ⭐ **无序** | ⭐ 键值查找 |
| `BTreeMap<K,V>` | 有序映射（B 树） | ⭐ 按 key 排序 | 需要排序 / 范围查询 |
| ⭐`HashSet<T>` | 哈希集合 | 无序 | ⭐ 去重、成员判断 |
| `BTreeSet<T>` | 有序集合 | 按值排序 | 有序去重 |
| ⭐`VecDeque<T>` | 双端队列（环形缓冲） | 插入顺序 | ⭐ 队列、滑动窗口 |
| `BinaryHeap<T>` | 二叉堆（最大堆） | 堆序 | 优先队列、Top-K |
| `LinkedList<T>` | 双向链表 | 插入顺序 | ⚠ 几乎不用 |

**⭐ 选型口诀**：

```text
按下标访问、顺序遍历        → Vec
键值查找                    → HashMap（需要排序/范围查询才用 BTreeMap）
判断"在不在"、去重          → HashSet
两头都要进出                → VecDeque
每次取最大/最小             → BinaryHeap
```

**复杂度速查**：

| 操作 | `Vec` | `HashMap` | `BTreeMap` | `VecDeque` | `BinaryHeap` |
| ---- | ----- | --------- | ---------- | ---------- | ------------ |
| 查找 | O(n) | ⭐ O(1) | O(log n) | O(n) | — |
| 按下标 | ⭐ O(1) | — | — | ⭐ O(1) | — |
| 尾部插入 | ⭐ O(1)* | O(1)* | O(log n) | ⭐ O(1)* | O(log n) |
| 头部插入 | O(n) | — | — | ⭐ O(1)* | — |
| 删除 | O(n) | ⭐ O(1) | O(log n) | O(1) 两端 | O(log n) |
| 取最值 | O(n) | O(n) | ⭐ O(log n) | — | ⭐ O(1) peek |

（* 表示均摊复杂度）

---

### 2. HashMap

```rust
use std::collections::HashMap;

let mut map: HashMap<String, i32> = HashMap::new();
let mut map = HashMap::with_capacity(100);              // ⭐ 预分配
let map = HashMap::from([("a", 1), ("b", 2)]);          // ⭐ 字面量构造
let map: HashMap<_, _> = vec![("a", 1)].into_iter().collect();   // 从迭代器
```

| 操作 | 方法 | 返回 |
| ---- | ---- | ---- |
| ⭐ 插入 | `insert(k, v)` | ⭐`Option<V>`（被覆盖的旧值） |
| ⭐ 查询 | `get(&k)` | `Option<&V>` |
| | `get_mut(&k)` | `Option<&mut V>` |
| | `contains_key(&k)` | `bool` |
| ⭐ 删除 | `remove(&k)` | `Option<V>` |
| | `remove_entry(&k)` | `Option<(K, V)>` |
| ⭐ 更新 | `entry(k)` | ⭐ 见下节 |
| 遍历 | `iter()` / `keys()` / `values()` / `values_mut()` | — |
| 其他 | `len()` / `is_empty()` / `clear()` / `retain(f)` | — |

```rust
map.insert("a".into(), 1);
map.get("a");                           // ⭐ Some(&1)，K 是 String 但可以传 &str
map["a"];                               // ⚠ 键不存在会 panic，尽量别用
map.get("x").copied().unwrap_or(0);     // ✔ 带默认值

for (k, v) in &map { }                  // ⭐ 借用遍历，(&K, &V)
for (k, v) in &mut map { }              // (&K, &mut V)
for (k, v) in map { }                   // 消耗 map，(K, V)
map.retain(|_, v| *v > 0);              // ⭐ 条件删除
```

**⭐ `HashMap` 是无序的**，且 Rust 默认使用**随机种子**的 SipHash，所以**每次运行的迭代顺序都可能不同** —— 这是为了防哈希碰撞攻击（HashDoS），但意味着**绝不能依赖迭代顺序**。

---

### 3. entry API（必须掌握）

⭐ **"不存在就插入、存在就更新"的标准写法**，一次哈希查找搞定，同时避开借用冲突：

```rust
// ① 计数器 —— 最经典的用法
let mut counts: HashMap<&str, i32> = HashMap::new();
for word in text.split_whitespace() {
    *counts.entry(word).or_insert(0) += 1;              // ⭐
}

// ② 分组：HashMap<K, Vec<V>>
let mut groups: HashMap<String, Vec<Item>> = HashMap::new();
for item in items {
    groups.entry(item.category.clone()).or_default().push(item);    // ⭐ or_default
}

// ③ 默认值需要计算时用惰性版本
map.entry(k).or_insert_with(|| expensive_compute());    // ⭐ 只在缺失时才计算
map.entry(k).or_insert_with_key(|k| k.len());

// ④ 同时处理"存在"和"不存在"
map.entry(k)
   .and_modify(|v| *v += 1)             // ⭐ 存在时
   .or_insert(1);                       // 不存在时
```

**对比传统写法**：

```rust
// ✘ 两次哈希查找，且有借用冲突风险
if !map.contains_key(k) { map.insert(k, 0); }
*map.get_mut(k).unwrap() += 1;

// ✔ 一次查找
*map.entry(k).or_insert(0) += 1;
```

---

### 4. BTreeMap：有序映射

⭐ **需要"按 key 排序遍历"或"范围查询"时用它**，要求 `K: Ord`：

```rust
use std::collections::BTreeMap;

let mut m = BTreeMap::new();
m.insert(3, "c"); m.insert(1, "a"); m.insert(2, "b");

for (k, v) in &m { }                    // ⭐ 保证按 key 升序：1, 2, 3

m.range(2..=5);                         // ⭐ 范围查询，HashMap 做不到
m.first_key_value();                    // ⭐ 最小的键值对
m.last_key_value();                     // 最大的
```

| 对比 | ⭐`HashMap` | ⭐`BTreeMap` |
| ---- | ---------- | ----------- |
| 查找 | O(1) | O(log n) |
| 顺序 | ⭐ 无序（且每次运行不同） | ⭐ 按 key 升序 |
| 范围查询 | ✘ | ⭐ ✔ `range()` |
| key 要求 | `Eq + Hash` | `Ord` |
| 内存局部性 | 一般 | 较好 |
| 使用频率 | ⭐ 高 | 中 |

> 小数据量（几十个元素）时 `BTreeMap` 常常比 `HashMap` 更快（无需计算哈希，缓存友好）。需要稳定的序列化输出（如生成配置文件、计算签名）时必须用 `BTreeMap`。

---

### 5. HashSet 与 BTreeSet

```rust
use std::collections::HashSet;

let mut s: HashSet<i32> = HashSet::new();
s.insert(1);                            // ⭐ 返回 bool：是否是新元素
s.contains(&1);                         // ⭐ O(1)
s.remove(&1);

// ⭐ 去重最常用写法
let unique: HashSet<_> = v.into_iter().collect();
let unique: Vec<_> = v.into_iter().collect::<HashSet<_>>().into_iter().collect();

// ⭐ 集合运算
a.union(&b);                    // 并集
a.intersection(&b);             // 交集
a.difference(&b);               // 差集 a - b
a.symmetric_difference(&b);     // 对称差
a.is_subset(&b);  a.is_disjoint(&b);
```

> `HashSet<T>` 本质就是 `HashMap<T, ()>`，利用了 `()` 零大小的特性。

---

### 6. VecDeque 与 BinaryHeap

**`VecDeque`：双端队列**，头尾操作都是 O(1)（`Vec` 的头部操作是 O(n)）：

```rust
use std::collections::VecDeque;

let mut q = VecDeque::new();
q.push_back(1);                         // 尾入
q.push_front(0);                        // ⭐ 头入，O(1)
q.pop_front();                          // ⭐ 头出，队列语义
q.pop_back();                           // 栈语义
q.front();  q.back();
```

**`BinaryHeap`：优先队列（默认最大堆）**：

```rust
use std::{collections::BinaryHeap, cmp::Reverse};

let mut h = BinaryHeap::new();
h.push(3); h.push(1); h.push(5);
h.peek();                               // Some(&5)，⭐ 最大值
h.pop();                                // Some(5)

// ⭐ 最小堆：用 Reverse 包装
let mut min = BinaryHeap::new();
min.push(Reverse(3));
let Reverse(v) = min.pop().unwrap();    // 1

// ⭐ Top-K 的标准写法：维护大小为 k 的最小堆
for x in data {
    heap.push(Reverse(x));
    if heap.len() > k { heap.pop(); }
}
```

> `LinkedList` 在 Rust 里几乎没有使用场景 —— 缓存不友好，遍历慢，且没有稳定的"按引用插入"API。需要频繁头尾操作用 `VecDeque`，需要中间插入通常说明该换数据结构。

---

### 7. 作为 key 的类型要求

| 集合 | key 约束 |
| ---- | -------- |
| `HashMap` / `HashSet` | ⭐`Eq + Hash` |
| `BTreeMap` / `BTreeSet` | ⭐`Ord` |

```rust
#[derive(PartialEq, Eq, Hash)]          // ⭐ 自定义类型作 key 必须 derive
struct UserId(u64);

#[derive(PartialEq, Eq, PartialOrd, Ord)]
struct Version { major: u32, minor: u32 }       // BTreeMap 的 key
```

**⭐ 两条铁律**：

- **`f64` 不能作 key**（没有 `Eq`/`Ord`/`Hash`，因为 `NaN != NaN`）—— 要用得改成整数或 `ordered_float`
- **`Hash` 和 `Eq` 必须一致**：相等的对象哈希值必须相同。手写实现时如果只比较部分字段，哈希也必须只用那些字段，否则会出现"插进去却查不到"的诡异 bug

**⭐ 用 `&str` 查询 `String` 作 key 的 Map**（`Borrow` trait 的功劳）：

```rust
let mut m: HashMap<String, i32> = HashMap::new();
m.insert("a".to_string(), 1);
m.get("a");                             // ⭐ ✔ 不需要构造 String
```

---

### 8. 高频踩坑点（重点）

**① 依赖 `HashMap` 的迭代顺序**

```rust
for (k, v) in &map { }                  // ⚠ 顺序不确定，且每次运行都不同
```

> 需要确定顺序：用 `BTreeMap`，或收集后排序 `let mut v: Vec<_> = map.iter().collect(); v.sort();`。测试用例里比对 `HashMap` 的输出顺序是最常见的 flaky test 来源。

**② 默认哈希算法偏慢**

Rust 默认用 SipHash 1-3，抗 HashDoS 攻击但性能不是最优。**内部数据、性能敏感场景**可以换：

```toml
ahash = "0.8"           # 或 rustc-hash（FxHashMap）
```

```rust
use ahash::AHashMap;
let map: AHashMap<u64, i32> = AHashMap::new();          // ⭐ 快 2~5 倍
```

> ⚠ 只在 key 来自可信来源时替换 —— 对外暴露的接口（如 HTTP 参数作 key）保留默认 hasher 更安全。

**③ `map[key]` 会 panic**

```rust
map["missing"];                         // ✘ panic
map.get("missing");                     // ✔ Option
map.get("missing").copied().unwrap_or_default();        // ✔
```

**④ `or_insert` 的参数会立即求值**

```rust
map.entry(k).or_insert(expensive());            // ✘ 每次都执行
map.entry(k).or_insert_with(|| expensive());    // ⭐ ✔ 惰性
map.entry(k).or_default();                      // ✔ 用 Default
```

**⑤ 借用冲突：在插入时读取自身**

```rust
map.insert(k, map.len());               // ✘ map 同时被可变和不可变借用
let n = map.len();                      // ✔ 先取出来
map.insert(k, n);
```

**⑥ 集合遍历时修改**

```rust
for (k, v) in &map { map.remove(k); }   // ✘ 编译错误
map.retain(|k, v| *v > 0);              // ✔
let keys: Vec<_> = map.keys().cloned().collect();       // ✔ 先收集
for k in keys { map.remove(&k); }
```

**⑦ 嵌套集合的更新**

```rust
// ✘ 啰嗦
if !map.contains_key(&k) { map.insert(k.clone(), Vec::new()); }
map.get_mut(&k).unwrap().push(v);

// ⭐ ✔
map.entry(k).or_default().push(v);
```

**⑧ 修改了 key 的哈希值**

Rust 通过所有权规避了这个问题（key 插入后无法再取到 `&mut`），但如果 key 内部含 `RefCell`/`Cell` 就可能被改，导致元素永远查不到。**key 应该是不可变的纯数据**。

**⑨ 大量插入不预分配**

```rust
let mut m = HashMap::new();
for i in 0..100_000 { m.insert(i, i); }             // ⚠ 多次 rehash
let mut m = HashMap::with_capacity(100_000);        // ⭐ ✔
```

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 默认 `Vec`，键值查找才 `HashMap` | 小数据量线性查找往往更快 |
| 更新 map 一律用 `entry` API | 一次哈希，避免借用冲突 |
| 需要有序输出用 `BTreeMap` | 序列化、签名、日志比对场景 |
| 不依赖 `HashMap` 迭代顺序 | 测试里尤其注意，会造成 flaky test |
| 已知规模用 `with_capacity` | 避免反复 rehash |
| 内部高频 map 换 `ahash`/`FxHashMap` | 对外接口保留默认 hasher 防 HashDoS |
| 访问用 `get()` 不用 `map[k]` | 后者会 panic |
| 自定义 key derive `Eq + Hash` | 且保证 `Hash` 与 `Eq` 语义一致 |
| 头尾频繁进出用 `VecDeque` | 不要用 `Vec` 的 `remove(0)` |
| Top-K 用 `BinaryHeap` | 不要全排序 |
| 不用 `LinkedList` | Rust 中它几乎没有优势 |
| 返回空集合而非 `Option<集合>` | 空集合本身就表达了"没有数据" |

---

### 10. 面试重点

**Q1：`HashMap` 和 `BTreeMap` 怎么选？**

`HashMap` 是哈希表，增删查平均 **O(1)**，但**无序**；`BTreeMap` 是 B 树，操作 **O(log n)**，但**按 key 有序**且支持 `range()` 范围查询。默认用 `HashMap`；需要排序遍历、范围查询、或稳定的序列化输出（配置文件、签名计算）时用 `BTreeMap`。数据量很小时 `BTreeMap` 反而可能更快（省去哈希计算，缓存友好）。key 的约束也不同：`HashMap` 要 `Eq + Hash`，`BTreeMap` 要 `Ord`。

**Q2：`HashMap` 的迭代顺序为什么不固定？**

Rust 默认使用带**随机种子**的 SipHash 1-3 算法，每次程序启动种子都不同，所以同样的插入序列在不同运行中迭代顺序也不同。这是为了防止**哈希碰撞拒绝服务攻击（HashDoS）**——攻击者构造大量哈希冲突的 key 让哈希表退化成链表。代价是绝不能依赖迭代顺序，需要确定顺序得用 `BTreeMap` 或收集后排序。性能敏感且 key 可信时可以换 `ahash`/`FxHashMap`。

**Q3：`entry` API 有什么用？**

解决"不存在就插入、存在就更新"的场景，优势有三：① **只做一次哈希查找**（传统写法 `contains_key` + `insert` + `get_mut` 要查三次）；② **避开借用冲突**（不会出现"map 同时被不可变和可变借用"）；③ 语义清晰。常用组合：`or_insert(v)`、`or_insert_with(f)`（惰性）、`or_default()`、`and_modify(f).or_insert(v)`。词频统计写作 `*map.entry(word).or_insert(0) += 1;`，分组写作 `map.entry(k).or_default().push(v);`。

**Q4：什么类型可以作 `HashMap` 的 key？**

必须实现 `Eq + Hash`。基本类型、`String`、`&str`、元组、以及 `#[derive(PartialEq, Eq, Hash)]` 的自定义类型都可以。**`f64` 不行**——因为 `NaN != NaN` 破坏了等价关系，没有 `Eq`。另外 `Hash` 和 `Eq` 的实现必须**语义一致**（相等的值哈希必须相同），手写实现时如果 `Eq` 只比较部分字段，`Hash` 也只能用那些字段，否则会出现"插进去查不到"的 bug。

**Q5：`Vec` 和 `VecDeque` 的区别？**

`Vec` 是连续数组，尾部操作 O(1)，**头部插入/删除是 O(n)**（要整体搬移）；`VecDeque` 是**环形缓冲区**，头尾操作**都是 O(1)**，代价是内存不保证连续（不能直接当切片用，要用 `make_contiguous()`）。需要队列语义（FIFO）、滑动窗口、双端操作时用 `VecDeque`；纯尾部追加和随机访问用 `Vec`。

**Q6：怎么求 Top-K？**

用 `BinaryHeap` 维护一个大小为 K 的**最小堆**（用 `Reverse` 包装把最大堆变最小堆）：遍历数据不断 `push`，长度超过 K 就 `pop` 掉最小的。时间复杂度 O(n log k)，空间 O(k)，比全排序的 O(n log n) 更优，且不需要把所有数据装进内存。

**Q7：为什么 Rust 里几乎不用 `LinkedList`？**

链表节点分散在堆上，**缓存局部性极差**，实际性能通常远不如 `Vec`/`VecDeque`（即使理论复杂度更优）；同时 Rust 的所有权模型让链表的"任意位置插入"API 很难安全地暴露（需要持有节点引用，而借用检查不允许）。需要双端操作用 `VecDeque`，需要频繁中间插入通常意味着该换一种数据结构（如索引 + `Vec`）。

---

## 十八、迭代器

> ⭐ 迭代器是 Rust 代码风格的分水岭：会用的人写出的代码简洁、安全且**通常比手写循环更快**。面试常考它的**惰性**和**零成本抽象**。

### 1. Iterator trait 与惰性求值

整个迭代器体系只建立在**一个必需方法**上：

```rust
pub trait Iterator {
    type Item;                                  // ⭐ 关联类型
    fn next(&mut self) -> Option<Self::Item>;   // ⭐ 唯一必须实现的方法

    // 其余 70+ 个方法都有默认实现
}
```

**⭐ 迭代器是惰性（lazy）的：适配器不会执行任何计算，直到被消费器触发**：

```rust
let v = vec![1, 2, 3];

v.iter().map(|x| { println!("处理 {x}"); x * 2 });   // ⚠ 什么都不会打印
                                                     //    warning: unused `Map` that must be used

let r: Vec<i32> = v.iter().map(|x| x * 2).collect(); // ⭐ collect 触发执行
```

**惰性带来的两个好处**：

```rust
// ① 无限序列也能用
let v: Vec<u64> = (1..).map(|x| x * x).take(5).collect();       // [1,4,9,16,25]

// ② ⭐ 短路：找到就停，不会遍历完
let found = huge_list.iter().find(|x| x.id == target);          // 找到即返回
let has = huge_list.iter().any(|x| x.is_valid());               // 有一个满足就返回
```

---

### 2. 三种迭代方式

| 方法 | 迭代出 | 集合归属 | 使用场景 |
| ---- | ------ | -------- | -------- |
| ⭐`iter()` | `&T` | 保留 | ⭐ 只读遍历（默认选它） |
| `iter_mut()` | `&mut T` | 保留 | 就地修改元素 |
| ⭐`into_iter()` | `T` | ⭐ **被消耗** | 需要元素所有权 |

```rust
for x in &v      { }        // 等价 v.iter()
for x in &mut v  { }        // 等价 v.iter_mut()
for x in v       { }        // ⭐ 等价 v.into_iter()，v 之后不可用

v.iter_mut().for_each(|x| *x *= 2);         // ⭐ 就地翻倍
let owned: Vec<String> = v.into_iter().collect();       // 取得所有权
```

**`IntoIterator` 是 `for` 循环的底层机制**：任何实现了它的类型都能被 `for` 遍历。

---

### 3. 适配器（Adaptor）：返回新迭代器

| 适配器 | 作用 | 示例 |
| ------ | ---- | ---- |
| ⭐`map(f)` | 转换每个元素 | `.map(\|x\| x * 2)` |
| ⭐`filter(p)` | 保留满足条件的 | `.filter(\|x\| **x > 0)` |
| ⭐`filter_map(f)` | ⭐ 转换 + 过滤（返回 `Option`） | `.filter_map(\|s\| s.parse().ok())` |
| ⭐`flat_map(f)` | 转换成迭代器再展平 | `.flat_map(\|s\| s.chars())` |
| `flatten()` | 展平嵌套结构 | `Vec<Vec<T>>` → `Vec<T>` |
| ⭐`enumerate()` | 带上索引 | `.enumerate()` → `(i, x)` |
| ⭐`zip(other)` | 两个迭代器配对 | ⭐ 短的结束就停 |
| `chain(other)` | 首尾连接 | — |
| ⭐`take(n)` / `skip(n)` | 取前 n 个 / 跳过前 n 个 | 分页常用 |
| `take_while(p)` / `skip_while(p)` | 按条件取/跳 | — |
| `step_by(n)` | 每 n 个取一个 | — |
| ⭐`rev()` | 反转（需 `DoubleEndedIterator`） | — |
| ⭐`cloned()` / `copied()` | ⭐`&T` → `T` | `.iter().copied()` |
| `peekable()` | 可以"偷看"下一个 | 手写解析器常用 |
| `inspect(f)` | ⭐ 调试用，观察但不改变 | `.inspect(\|x\| println!("{x:?}"))` |
| `scan(init, f)` | 带状态的 map（如前缀和） | — |
| `windows(n)` / `chunks(n)` | ⭐ 切片方法：滑动窗口 / 分批 | 批量处理 |

```rust
// ⭐ filter_map：比 filter + map 更简洁，一步完成"解析成功的才保留"
let nums: Vec<i32> = lines.iter().filter_map(|s| s.parse().ok()).collect();

// ⭐ flat_map：一对多展开
let chars: Vec<char> = words.iter().flat_map(|w| w.chars()).collect();

// ⭐ zip + enumerate
for (i, (name, score)) in names.iter().zip(scores.iter()).enumerate() { }

// ⭐ chunks：批量入库，避免 N+1
for batch in items.chunks(500) {
    db.insert_batch(batch)?;
}
```

---

### 4. 消费器（Consumer）：触发执行并产出结果

| 消费器 | 作用 |
| ------ | ---- |
| ⭐`collect()` | ⭐ 收集成集合（见下节） |
| ⭐`sum()` / `product()` | 求和 / 求积 |
| ⭐`count()` | 计数（⚠ 会完整遍历） |
| ⭐`fold(init, f)` | ⭐ 折叠/归约，最通用 |
| `reduce(f)` | 无初值的 fold，返回 `Option` |
| ⭐`for_each(f)` | 逐个执行副作用 |
| ⭐`any(p)` / `all(p)` | ⭐ 存在 / 全部满足（短路） |
| ⭐`find(p)` / `position(p)` | 找元素 / 找索引（短路） |
| `find_map(f)` | 找到第一个转换成功的 |
| ⭐`max()` / `min()` | 最值（需 `Ord`） |
| ⭐`max_by_key(f)` / `min_by_key(f)` | ⭐ 按字段取最值 |
| `max_by(f)` / `min_by(f)` | 自定义比较（浮点用这个） |
| ⭐`partition(p)` | ⭐ 一分为二 `(Vec, Vec)` |
| `unzip()` | `Vec<(A,B)>` → `(Vec<A>, Vec<B>)` |
| `last()` / `nth(n)` | 最后一个 / 第 n 个 |
| ⭐`try_fold` / `try_for_each` | 可提前返回错误的版本 |

```rust
let total: i32 = v.iter().sum();
let n = v.iter().filter(|x| **x > 0).count();

// ⭐ fold：最通用的归约
let sum = v.iter().fold(0, |acc, x| acc + x);
let s = words.iter().fold(String::new(), |mut acc, w| { acc.push_str(w); acc });

// ⭐ 按字段取最值
let oldest = users.iter().max_by_key(|u| u.age);
let cheapest = items.iter().min_by(|a, b| a.price.total_cmp(&b.price));  // 浮点

// ⭐ 分组
let (adults, minors): (Vec<_>, Vec<_>) = users.into_iter().partition(|u| u.age >= 18);
```

---

### 5. collect：最强大的消费器

⭐ **`collect()` 的目标类型由上下文决定**，能收集成各种集合：

```rust
let v: Vec<i32>            = iter.collect();
let s: String              = chars.collect();               // ⭐ char 迭代器 → String
let m: HashMap<K, V>       = pairs.collect();               // ⭐ (K,V) 元组 → Map
let set: HashSet<i32>      = iter.collect();                // ⭐ 顺带去重
let v = iter.collect::<Vec<_>>();                           // turbofish 写法

// ⭐ 收集成 Result：遇到第一个 Err 就短路返回
let nums: Result<Vec<i32>, _> = strs.iter().map(|s| s.parse()).collect();

// ⭐ 收集成 Option：有一个 None 就返回 None
let vals: Option<Vec<_>> = keys.iter().map(|k| map.get(k)).collect();
```

**`Vec<Result<T,E>>` 与 `Result<Vec<T>,E>` 的转换是高频考点**：

```rust
// 全部成功才要结果 → Result<Vec<T>, E>
let all: Result<Vec<i32>, _> = inputs.iter().map(|s| s.parse()).collect();

// 只要成功的那些 → filter_map
let ok_only: Vec<i32> = inputs.iter().filter_map(|s| s.parse().ok()).collect();

// 成功和失败分开
let (ok, err): (Vec<_>, Vec<_>) = results.into_iter().partition(|r| r.is_ok());
```

**⭐ 建 Map 索引，解决 N+1 问题**：

```rust
let user_map: HashMap<u64, &User> = users.iter().map(|u| (u.id, u)).collect();
for order in &orders {
    user_map.get(&order.user_id);       // ⭐ O(1)，代替 O(n×m) 的嵌套循环
}
```

---

### 6. 自定义迭代器

只需实现 `next()`，其余 70 多个方法自动获得：

```rust
struct Fib { a: u64, b: u64 }

impl Iterator for Fib {
    type Item = u64;
    fn next(&mut self) -> Option<u64> {
        let r = self.a;
        self.a = self.b;
        self.b = r + self.b;
        Some(r)                         // ⭐ 无限迭代器，返回 None 表示结束
    }
}

let v: Vec<u64> = Fib { a: 0, b: 1 }.take(10).collect();    // ⭐ 直接用所有适配器
```

**让自己的集合支持 `for`**：实现 `IntoIterator`。

---

### 7. 性能：零成本抽象

⭐ **迭代器链通常和手写循环一样快，甚至更快**：

```rust
let sum: u64 = v.iter().filter(|x| *x % 2 == 0).map(|x| x * 2).sum();

// 编译后等价于（且没有边界检查）：
let mut sum = 0;
for x in v { if x % 2 == 0 { sum += x * 2; } }
```

原因：① 适配器都是 `#[inline]` 的零大小结构体，**编译器完全内联并融合成单次循环**；② 用下标 `v[i]` 需要插入**边界检查**，迭代器则由类型系统保证不越界，检查被完全消除。

**并行化只需改一个词**（`rayon` crate）：

```rust
use rayon::prelude::*;
let sum: u64 = v.par_iter().map(|x| expensive(x)).sum();    // ⭐ iter → par_iter
```

---

### 8. 高频踩坑点（重点）

**① 忘记消费，适配器不执行**

```rust
v.iter().map(|x| println!("{x}"));      // ⚠ 什么都不发生
v.iter().for_each(|x| println!("{x}")); // ✔
for x in &v { println!("{x}"); }        // ⭐ 纯副作用用 for 更直观
```

**② 迭代器只能消费一次**

```rust
let it = v.iter();
let a: Vec<_> = it.collect();
let b: Vec<_> = it.collect();           // ✘ it 已被移走
```

**③ `collect()` 需要类型标注**

```rust
let v = iter.collect();                 // ✘ collect 成什么？
let v: Vec<_> = iter.collect();         // ✔
let v = iter.collect::<Vec<_>>();       // ✔
```

**④ 闭包参数的引用层级**

```rust
v.iter().filter(|x| x > 1);             // ✘ x 是 &&i32
v.iter().filter(|&&x| x > 1);           // ✔ 模式解构
v.iter().filter(|x| **x > 1);           // ✔ 显式解引用
v.iter().copied().filter(|x| *x > 1);   // ⭐ copied() 后是 &i32，更清晰
```

**⑤ 无限迭代器忘记 `take`**

```rust
let v: Vec<u64> = (1..).collect();      // ✘ 死循环 + OOM
let v: Vec<u64> = (1..).take(10).collect();     // ✔
```

**⑥ 中间不必要的 `collect`**

```rust
let a: Vec<_> = v.iter().map(f).collect();      // ⚠ 多一次堆分配
let b: Vec<_> = a.iter().filter(g).collect();

let b: Vec<_> = v.iter().map(f).filter(g).collect();    // ⭐ ✔ 一次遍历，一次分配
```

**⑦ `count()` / `last()` 会完整遍历**

```rust
v.iter().count();                       // ⚠ O(n)
v.len();                                // ⭐ ✔ O(1)
```

**⑧ 链太长影响可读性和调试**

```rust
// ⚠ 六七级链式很难定位问题
data.iter().filter(..).map(..).flat_map(..).filter(..).take(..).collect()

// ✔ 拆成有名字的中间步骤，或用 inspect 调试
let valid = data.iter().filter(|x| x.is_valid());
let names: Vec<_> = valid.map(|x| x.name.clone()).collect();

data.iter().inspect(|x| println!("{x:?}")).filter(..)    // ⭐ 调试神器
```

**⑨ `zip` 会在较短的那个结束时停止**

```rust
[1,2,3].iter().zip([1,2].iter()).count();       // 2，不报错，静默丢弃第三个
```

**⑩ 迭代中修改集合**（编译期拦截，第九章）

```rust
for x in &v { v.push(*x); }             // ✘
v.retain(|x| *x > 0);                   // ✔
```

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 优先用迭代器而非下标循环 | 无边界检查开销，更不易写错 |
| 纯副作用用 `for`，数据变换用迭代器链 | `for_each` 只在链尾且必要时用 |
| 链式长度控制在 3~5 级 | 超过就拆分成有名字的步骤 |
| 避免中间 `collect()` | 每次 `collect` 都是一次堆分配 |
| 转换 + 过滤用 `filter_map` | 比 `filter().map()` 更清晰 |
| 提前终止用 `any`/`all`/`find` | 不要 `collect` 后再判断 |
| 批量处理用 `chunks(n)` | 杜绝循环内单条 IO（N+1 问题） |
| 建索引用 `map(...).collect::<HashMap<_,_>>()` | 代替嵌套循环 |
| 长度用 `len()` 不用 `count()` | 后者是 O(n) |
| 已知大小时先 `Vec::with_capacity` | `collect` 前预分配 |
| CPU 密集型批处理考虑 `rayon` | `iter()` → `par_iter()` 一行并行 |
| 复杂组合用 `itertools` crate | `group_by`、`unique`、`sorted`、`chunk_by` 等 |

---

### 10. 面试重点

**Q1：Rust 的迭代器是惰性的吗？什么意思？**

是。**适配器**（`map`、`filter`、`take` 等）只是构造一个新的迭代器结构体，不做任何实际计算；只有调用**消费器**（`collect`、`sum`、`for_each`、`any` 等）时才真正开始拉取元素。好处：① 可以表达无限序列（`(1..).take(5)`）；② 多个适配器融合成**一次遍历**，不产生中间集合；③ `find`/`any` 等能短路提前退出。副作用是只写适配器不消费的代码什么都不会发生，编译器会给 `unused must_use` 警告。

**Q2：`iter()`、`iter_mut()`、`into_iter()` 有什么区别？**

`iter()` 产出 `&T`（不可变借用，集合保留）；`iter_mut()` 产出 `&mut T`（可变借用，可就地修改）；`into_iter()` 产出 `T`（**消耗集合**，取得元素所有权）。对应 `for` 循环的三种写法：`for x in &v`、`for x in &mut v`、`for x in v`。

**Q3：为什么说迭代器是零成本抽象？**

因为编译后的机器码和手写循环相当，甚至更快。原因：① 所有适配器都是**零大小或极小的结构体**，方法标了 `#[inline]`，编译器会完全内联并把多级链**融合成单个循环**，不产生中间集合；② 用下标访问 `v[i]` 时编译器必须插入**边界检查**，而迭代器由类型系统保证不越界，检查被彻底消除。所以"用迭代器比用下标快"在 Rust 里是成立的。

**Q4：`map` 和 `filter_map` 有什么区别？**

`map` 一对一转换，元素数量不变；`filter_map` 的闭包返回 `Option<T>`，返回 `Some` 的保留并解包、返回 `None` 的丢弃 —— 相当于 `filter` + `map` 一步完成。典型用法是"解析成功的才保留"：`inputs.iter().filter_map(|s| s.parse().ok())`。

**Q5：怎么把 `Vec<Result<T, E>>` 变成 `Result<Vec<T>, E>`？**

直接 `collect()` 即可 —— 标准库为 `Result` 实现了 `FromIterator`：`let r: Result<Vec<T>, E> = iter.collect();`，遇到第一个 `Err` 就**短路返回**，不再继续遍历。`Option` 同理。如果只想保留成功的项，改用 `filter_map(|r| r.ok())`；如果要成功失败分开，用 `partition`。

**Q6：`fold` 和 `reduce` 有什么区别？**

`fold(init, f)` 需要显式提供初始值，累加器类型可以与元素类型不同（如把 `Vec<&str>` 折叠成 `String`），空迭代器返回初始值；`reduce(f)` 没有初始值，用第一个元素作为起点，累加器类型必须和元素相同，空迭代器返回 `None`。需要中途返回错误时用 `try_fold`。

**Q7：自定义迭代器要实现什么？**

只需为类型实现 `Iterator` trait 的 **`next()`** 方法并指定关联类型 `Item`，其余 70 多个适配器和消费器都由默认实现自动获得。如果希望自己的集合能被 `for` 直接遍历，则实现 `IntoIterator`；支持 `rev()` 要额外实现 `DoubleEndedIterator`，`len()` 要 `ExactSizeIterator`。

---

## 十九、模块系统与 Cargo 工程管理

### 1. 包、Crate、模块的层级

⭐ **三个概念必须分清**（面试常问）：

| 概念 | 说明 | 对应 |
| ---- | ---- | ---- |
| ⭐ Package（包） | 由 `Cargo.toml` 定义，可包含多个 crate | Maven 的一个 project |
| ⭐ Crate（单元包） | ⭐ **编译单元**，分 binary 和 library 两种 | 一个 jar |
| ⭐ Module（模块） | crate 内部的命名空间，控制可见性 | Java 的 package |

```text
my-app/                     ← Package
├── Cargo.toml
├── Cargo.lock
└── src/
    ├── main.rs             ← ⭐ binary crate 的根（crate root）
    ├── lib.rs              ← ⭐ library crate 的根
    ├── config.rs           ← 模块
    └── service/
        ├── mod.rs          ← 旧风格
        └── user.rs
```

**规则**：一个 package 最多有 **1 个 library crate**，可以有**多个 binary crate**（`src/bin/*.rs` 各自是一个）。

---

### 2. 模块定义与文件组织

⭐ **`mod` 是"声明模块存在"，不是"引入"** —— 这是新人最大的困惑点：

```rust
// src/main.rs
mod config;                 // ⭐ 声明：去找 src/config.rs 或 src/config/mod.rs
mod service;

use config::Settings;       // ⭐ 引入：把路径缩短

fn main() { let s = Settings::new(); }
```

> ⭐ **文件放在 `src/` 下但没有 `mod` 声明，就根本不会被编译**（连语法错误都不报）。

**两种目录风格**：

```text
① 旧风格（Rust 2015）          ② ⭐ 新风格（2018+，推荐）
src/                            src/
├── service/                    ├── service.rs        ← 内含 mod user;
│   ├── mod.rs                  └── service/
│   └── user.rs                     └── user.rs
```

**内联模块**（小模块或测试模块用）：

```rust
mod math {
    pub fn add(a: i32, b: i32) -> i32 { a + b }

    pub mod advanced {          // ⭐ 嵌套模块
        pub fn pow(a: i32) -> i32 { a * a }
    }
}

math::advanced::pow(2);
```

---

### 3. 路径与 use

```rust
use crate::config::Settings;        // ⭐ 绝对路径（从 crate 根）
use self::helper::format;           // 当前模块
use super::parent_fn;               // ⭐ 父模块

use std::collections::{HashMap, HashSet};       // ⭐ 嵌套引入
use std::io::{self, Read, Write};               // self 表示 io 本身
use std::fmt::Result as FmtResult;              // ⭐ 重命名，解决同名冲突
use std::collections::*;                        // ⚠ glob，只用于测试和 prelude

pub use crate::service::UserService;            // ⭐ 重导出：对外暴露更扁平的 API
```

**⭐ `pub use` 是设计公开 API 的关键**：内部结构可以很深，对外只暴露扁平路径：

```rust
// src/lib.rs
mod service;
mod model;

pub use service::user::UserService;         // ⭐ 用户写 my_crate::UserService 即可
pub use model::{User, Order};
```

---

### 4. 可见性规则

| 修饰符 | 可见范围 |
| ------ | -------- |
| （默认） | ⭐ 仅当前模块及其**子模块** |
| ⭐`pub(crate)` | ⭐ 整个 crate 内（内部 API 的主力） |
| `pub(super)` | 父模块 |
| `pub(in path)` | 指定路径内 |
| `pub` | 完全公开 |

**⭐ 两条核心规则**：

```rust
mod outer {
    fn private_fn() {}

    pub mod inner {
        pub fn call() {
            super::private_fn();        // ⭐ ✔ 子模块可以访问父模块的私有项
        }
    }
}
// ⭐ 但父模块不能访问子模块的私有项
```

```rust
pub struct User {
    pub name: String,           // ⭐ 结构体 pub 不代表字段 pub
    age: u32,                   // 私有
}

pub enum Status { Ok, Err }     // ⭐ 枚举 pub 则所有变体自动 pub
```

---

### 5. Cargo 基础

**项目结构约定**：

```text
my-app/
├── Cargo.toml          ⭐ 项目清单
├── Cargo.lock          ⭐ 精确依赖版本
├── src/
│   ├── main.rs         二进制入口
│   ├── lib.rs          库入口
│   └── bin/other.rs    ⭐ 额外的二进制目标
├── tests/              ⭐ 集成测试（只能用 pub API）
├── benches/            基准测试
├── examples/           ⭐ 示例代码（cargo run --example x）
├── build.rs            构建脚本
└── target/             ⭐ 编译产物（必须 gitignore）
```

**常用命令**：

| 命令 | 作用 |
| ---- | ---- |
| ⭐`cargo check` | ⭐ 只做类型检查，**比 build 快得多**，开发时高频用 |
| `cargo build` / `--release` | 编译 debug / ⭐ 优化版 |
| ⭐`cargo run` | 编译并运行 |
| ⭐`cargo test` | 运行所有测试 |
| ⭐`cargo clippy` | ⭐ 官方 lint，比编译器警告严格得多 |
| ⭐`cargo fmt` | 按 rustfmt 统一格式 |
| ⭐`cargo add xxx` | 添加依赖（自动写入 Cargo.toml） |
| `cargo update` | 更新依赖到兼容的最新版 |
| ⭐`cargo tree` | 查看依赖树（排查版本冲突） |
| `cargo doc --open` | 生成并打开文档 |
| `cargo bench` | 基准测试 |
| `cargo publish` | 发布到 crates.io |

> ⭐ **开发循环用 `cargo check` + `cargo clippy`**，只有真要运行时才 `cargo build`，能节省大量时间。

---

### 6. Cargo.toml 详解

```toml
[package]
name = "my-app"
version = "0.1.0"
edition = "2024"                    # ⭐ 版次：2015/2018/2021/2024
rust-version = "1.85"               # ⭐ MSRV，最低支持的 Rust 版本

[dependencies]
serde = { version = "1", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
anyhow = "1"
thiserror = "2"

# 本地/Git 依赖
my-lib = { path = "../my-lib" }
other = { git = "https://github.com/x/y", tag = "v1.0" }

[dev-dependencies]                  # ⭐ 只用于测试/示例，不会传递给下游
criterion = "0.5"

[build-dependencies]                # build.rs 用的依赖
cc = "1"

[features]
default = ["json"]                  # ⭐ 默认启用的特性
json = ["dep:serde_json"]           # ⭐ 特性可开启可选依赖
full = ["json", "async"]

[profile.release]
opt-level = 3                       # 优化级别 0~3 / "s" / "z"（体积优先）
lto = true                          # ⭐ 链接时优化，更快更小，但编译慢
codegen-units = 1                   # ⭐ 牺牲编译并行度换运行性能
panic = "abort"                     # ⭐ 不做栈展开，二进制更小
strip = true                        # 去掉符号表
```

**⭐ 版本号语义（SemVer）**：

| 写法 | 含义 |
| ---- | ---- |
| ⭐`"1.2.3"` | 等价于 `^1.2.3`：允许 `>=1.2.3, <2.0.0` |
| `"~1.2.3"` | 允许 `>=1.2.3, <1.3.0` |
| `"=1.2.3"` | 精确锁定 |
| `"*"` | 任意版本（⚠ 禁用） |

> ⭐ Rust 生态严格遵循 SemVer：**主版本号相同就保证 API 兼容**。`0.x` 版本例外，`0.1` 和 `0.2` 视为不兼容。

**Cargo.lock 要不要提交**：⭐ **二进制项目必须提交**（保证构建可复现）；库项目现在也**推荐提交**（便于 CI 复现问题，下游不受影响）。

---

### 7. Workspace（多包工程）

⭐ **中大型项目的标准组织方式**：多个 crate 共享一个 `target/` 和 `Cargo.lock`，编译产物复用，依赖版本统一。

```toml
# 根目录 Cargo.toml
[workspace]
resolver = "3"
members = ["crates/*"]

[workspace.dependencies]            # ⭐ 统一管理版本
serde = { version = "1", features = ["derive"] }
tokio = { version = "1", features = ["full"] }

[workspace.package]
version = "0.1.0"
edition = "2024"
```

```toml
# crates/api/Cargo.toml
[package]
name = "api"
version.workspace = true            # ⭐ 继承 workspace 配置
edition.workspace = true

[dependencies]
serde.workspace = true              # ⭐ 继承版本
domain = { path = "../domain" }
```

**典型分层**：

```text
crates/
├── domain/         纯业务模型，不依赖框架
├── infra/          数据库、缓存、外部接口
├── service/        业务逻辑
└── api/            HTTP 入口（binary）
```

> ⚠ **crate 之间不能循环依赖**（模块之间可以）。出现循环通常意味着分层有问题，应把公共部分抽到更底层的 crate。

---

### 8. 高频踩坑点（重点）

**① `mod` 和 `use` 搞混**

```rust
use config::Settings;               // ✘ 只 use 没 mod，编译器找不到 config
mod config;                         // ⭐ 必须先声明模块
use config::Settings;               // ✔
```

> 记忆：**`mod` = "把这个文件加入编译"，`use` = "给路径起个短名字"**。

**② 新建的文件没生效**

在 `src/` 下新建了 `foo.rs` 却忘了在父模块写 `mod foo;` —— 文件完全不会被编译，写什么都没反应。

**③ 忘记加 `pub`**

```rust
mod service { fn run() {} }
service::run();                     // ✘ private
```

> 注意子模块能访问父模块私有项，但反过来不行。

**④ 集成测试只能访问公开 API**

```rust
// tests/api.rs
use my_app::internal_fn;            // ✘ 除非它是 pub
```

`tests/` 下的每个文件都是独立 crate，只能用 `pub` 的东西。要测私有函数就写在 `src/` 内的 `#[cfg(test)] mod tests`（第二十章）。

**⑤ feature 统一（feature unification）**

同一个依赖被多处引用时，Cargo 会**合并所有启用的 feature**。所以某个依赖开了 `tokio/full`，整个 workspace 都会带上，导致编译变慢、二进制变大。用 `cargo tree -f "{p} {f}"` 排查。

**⑥ glob 导入污染命名空间**

```rust
use std::collections::*;            // ⚠ 名字来源不明，易冲突
```

只在 `#[cfg(test)] mod tests { use super::*; }` 和引入 prelude 时用。

**⑦ 依赖过多导致编译缓慢**

```toml
tokio = { version = "1", features = ["full"] }              # ⚠ 拉进所有模块
tokio = { version = "1", features = ["rt-multi-thread", "macros", "net"] }   # ✔ 按需
```

**⑧ `target/` 没有 gitignore**

`target/` 动辄几个 GB，必须忽略。`cargo new` 生成的 `.gitignore` 已包含。

**⑨ 二进制体积过大**

```toml
[profile.release]
opt-level = "z"     # 体积优先
lto = true
codegen-units = 1
panic = "abort"
strip = true
```

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 中大型项目用 workspace 分层 | domain / infra / service / api |
| 公共依赖版本写在 `[workspace.dependencies]` | 避免版本漂移 |
| 内部 API 用 `pub(crate)` | 只有真正对外的才 `pub` |
| 用 `pub use` 扁平化公开 API | 内部结构可以深，对外要简单 |
| 特性按需开启 | 尤其 `tokio`，不要无脑 `full` |
| `Cargo.lock` 提交进版本库 | 保证构建可复现 |
| CI 跑 `fmt --check` + `clippy -D warnings` + `test` | 三件套 |
| 指定 `rust-version`（MSRV） | 明确最低支持版本 |
| 定期 `cargo audit` / `cargo deny` | 检查依赖的安全漏洞与许可证 |
| 依赖数量要克制 | 每个依赖都是编译时间、体积和安全面 |
| release profile 开 `lto` | 性能和体积双收益 |
| 避免 crate 间循环依赖 | 出现循环说明分层有问题 |

---

### 10. 面试重点

**Q1：Package、Crate、Module 的区别？**

**Package** 由一个 `Cargo.toml` 定义，是 Cargo 管理的单位，可以包含多个 crate；**Crate** 是**编译单元**，分为 binary（有 `main`，根是 `src/main.rs`）和 library（根是 `src/lib.rs`）两种，一个 package 最多一个 lib crate、可以有多个 bin crate；**Module** 是 crate 内部的命名空间，用 `mod` 声明，负责代码组织和可见性控制。类比：package ≈ Maven project，crate ≈ jar，module ≈ Java package。

**Q2：`mod` 和 `use` 有什么区别？**

`mod foo;` 是**声明模块存在**，告诉编译器去加载 `foo.rs` 或 `foo/mod.rs` 并把它编译进来；`use` 只是**给已存在的路径起个短名字**，不引入任何新代码。新人最常见的错误是新建了文件却没写 `mod` 声明 —— 这个文件根本不会被编译，也不会报错。

**Q3：Rust 的可见性规则是怎样的？**

默认**一切私有**，只对当前模块及其**子模块**可见。修饰符有 `pub`（完全公开）、`pub(crate)`（crate 内可见，内部 API 主力）、`pub(super)`（父模块可见）、`pub(in path)`。两条特别规则：① **子模块可以访问父模块的私有项**，反之不行；② `pub struct` **不代表字段公开**，字段要单独加 `pub`（而 `pub enum` 的所有变体自动公开）。

**Q4：`Cargo.lock` 该不该提交？**

**二进制项目必须提交**——它锁定了整棵依赖树的精确版本，保证任何人任何时间构建出的产物一致。库项目历史上建议不提交，但现在官方也**推荐提交**：它只影响该库自己的 CI 和本地开发，下游使用时会重新解析，不受影响，而提交后能保证 CI 结果可复现。

**Q5：什么是 workspace？为什么用它？**

多个相关 crate 共享一个 `target/` 目录和一个 `Cargo.lock` 的组织方式。好处：① **编译产物复用**，公共依赖只编译一次；② **依赖版本统一**（`[workspace.dependencies]` 集中管理）；③ 便于按职责分层（domain / infra / service / api），强制模块边界。注意 crate 之间**不能循环依赖**。

**Q6：Cargo 的版本号 `"1.2"` 是什么意思？**

等价于 `^1.2`，即允许 `>=1.2.0, <2.0.0` —— 因为 Rust 生态严格遵循 **SemVer**：主版本号相同就保证 API 兼容。特例是 `0.x` 版本，此时次版本号被当作主版本号，`0.1` 和 `0.2` 视为不兼容。`~1.2.3` 只允许补丁版本升级，`=1.2.3` 精确锁定。

**Q7：Rust 编译慢有什么优化手段？**

① 开发时用 **`cargo check`** 而非 `cargo build`（跳过代码生成）；② **精简依赖和 feature**（不要无脑 `tokio/full`）；③ 用 **workspace** 让公共依赖只编译一次；④ 拆分大 crate，提高增量编译粒度；⑤ 减少泛型代码膨胀（泛型外壳 + 非泛型内核，第十四章）；⑥ 换更快的链接器（`lld`/`mold`）；⑦ CI 上做依赖缓存。反过来，release 追求运行性能时开 `lto = true` 和 `codegen-units = 1`，代价是编译更慢。

---

## 二十、测试、文档与常用生态库

> ⭐ Rust 把测试和文档做进了语言和工具链本身：不需要 JUnit，`cargo test` 开箱即用，连文档里的示例代码都会被自动执行。

### 1. 单元测试

⭐ **写在被测代码的同一个文件里**，这是 Rust 与 Java 最大的组织差异：

```rust
// src/math.rs
pub fn add(a: i32, b: i32) -> i32 { a + b }
fn internal(x: i32) -> i32 { x * 2 }        // 私有函数

#[cfg(test)]                                // ⭐ 只在 cargo test 时编译，不进发布产物
mod tests {
    use super::*;                           // ⭐ 引入父模块的所有项（包括私有的）

    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
    }

    #[test]
    fn test_internal() {
        assert_eq!(internal(2), 4);         // ⭐ 单元测试可以测私有函数
    }
}
```

**断言宏**：

```rust
assert!(cond, "自定义消息 {}", x);
assert_eq!(a, b);                           // ⭐ 失败时会打印两边的值（需要 Debug）
assert_ne!(a, b);
debug_assert!(cond);                        // ⭐ 仅 debug 生效，release 零开销
```

**四种常用属性**：

```rust
#[test]
#[should_panic(expected = "除数为零")]       // ⭐ 断言会 panic，且消息包含指定文本
fn test_panic() { divide(1, 0); }

#[test]
#[ignore = "耗时较长"]                       // ⭐ 默认跳过，cargo test -- --ignored 才跑
fn slow_test() { }

#[test]
fn test_result() -> Result<(), Box<dyn std::error::Error>> {
    let n: i32 = "42".parse()?;             // ⭐ 测试体内可以用 ?
    assert_eq!(n, 42);
    Ok(())
}

#[tokio::test]                              // ⭐ 异步测试（需要 tokio 的 macros feature）
async fn test_async() { fetch().await; }
```

---

### 2. 集成测试

⭐ **放在 `tests/` 目录，每个文件是一个独立的 crate，只能访问 `pub` API** —— 模拟真实用户的使用方式：

```text
tests/
├── api_test.rs             ← 独立 crate
├── db_test.rs              ← 独立 crate
└── common/
    └── mod.rs              ← ⭐ 共享辅助代码（放 common/mod.rs 才不会被当成测试 crate）
```

```rust
// tests/api_test.rs
mod common;

use my_app::UserService;                    // ⭐ 像外部用户一样引入

#[test]
fn test_create_user() {
    common::setup();
    let svc = UserService::new();
    assert!(svc.create("张三").is_ok());
}
```

| 对比 | ⭐ 单元测试 | ⭐ 集成测试 |
| ---- | ---------- | ---------- |
| 位置 | 源文件内 `#[cfg(test)] mod tests` | `tests/` 目录 |
| 可见性 | ⭐ 能测**私有**函数 | ⭐ 只能用 `pub` API |
| 编译单元 | 与被测代码同一 crate | 每个文件独立 crate |
| 用途 | 测单个函数/模块的逻辑 | 测模块协作、对外契约 |

---

### 3. 文档与文档测试

```rust
//! # 用户服务模块                              ⭐ //! 描述所在的模块/crate
//!
//! 提供用户的增删改查能力。

/// 计算两数之和。                              ⭐ /// 描述下面的项
///
/// # Examples
///
/// ```
/// use my_app::add;
/// assert_eq!(add(2, 3), 5);                  // ⭐ 这段代码会被 cargo test 真实执行！
/// ```
///
/// # Panics
/// 当结果溢出时 panic。                        ⭐ 标准小节：Panics / Errors / Safety
///
/// # Errors
/// 参数非法时返回 `Err`。
pub fn add(a: i32, b: i32) -> i32 { a + b }
```

⭐ **文档测试（doctest）是 Rust 的杀手锏**：文档里的示例代码会被 `cargo test` 编译并运行，**保证文档永远不会过时**。

```rust
/// ```
/// # let x = 5;              // ⭐ # 开头的行会执行但不显示在文档里
/// assert!(x > 0);
/// ```
///
/// ```no_run
/// // 只编译不运行（涉及网络、文件等）
/// ```
///
/// ```ignore
/// // 完全跳过（尽量别用）
/// ```
```

```bash
cargo doc --open            # ⭐ 生成 HTML 文档并打开
cargo test --doc            # 只跑文档测试
```

---

### 4. 运行测试的常用参数

```bash
cargo test                          # ⭐ 跑全部（单元 + 集成 + 文档）
cargo test test_add                 # ⭐ 按名字过滤（子串匹配）
cargo test --lib                    # 只跑单元测试
cargo test --test api_test          # 只跑指定集成测试文件
cargo test -- --ignored             # 跑被 ignore 的
cargo test -- --nocapture           # ⭐ 显示 println! 输出（默认被捕获）
cargo test -- --test-threads=1      # ⭐ 串行执行（测试有共享状态时）
cargo test --release                # 用优化版跑（测性能相关逻辑）
```

> ⭐ **`cargo test` 默认并行执行所有测试**，所以测试之间不能共享可变状态（数据库、文件、全局变量），否则会随机失败。需要串行就用 `--test-threads=1`，或者让每个测试用独立的临时目录/数据库。

**推荐工具**：

```bash
cargo install cargo-nextest         # ⭐ 更快的测试运行器，输出更清晰
cargo nextest run

cargo install cargo-llvm-cov        # 覆盖率
cargo llvm-cov --html
```

---

### 5. 基准测试

稳定版用 **`criterion`**（内置 `#[bench]` 需要 nightly）：

```toml
[dev-dependencies]
criterion = "0.5"

[[bench]]
name = "my_bench"
harness = false
```

```rust
// benches/my_bench.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn bench_add(c: &mut Criterion) {
    c.bench_function("add", |b| b.iter(|| add(black_box(2), black_box(3))));
}                                       // ⭐ black_box 防止编译器优化掉被测代码

criterion_group!(benches, bench_add);
criterion_main!(benches);
```

> ⚠ **性能测试必须用 `--release`**：debug 模式下 Rust 的性能可能相差 10~100 倍，用 debug 数据下结论毫无意义。

---

### 6. 常用生态库速查（重点）

⭐ Rust 标准库很小，很多能力靠生态。**这张表是面试"你用过哪些库"的答案来源**：

| 领域 | 库 | 说明 |
| ---- | -- | ---- |
| ⭐ 序列化 | ⭐`serde` + `serde_json` | ⭐ 事实标准，几乎所有库都基于它 |
| | `toml` / `serde_yaml` / `bincode` | 各种格式 |
| ⭐ 错误处理 | ⭐`thiserror`（库）/ ⭐`anyhow`（应用） | 第十三章 |
| ⭐ 异步运行时 | ⭐`tokio` | ⭐ 事实标准；`async-std` 已边缘化 |
| | `futures` | 异步工具组合子 |
| ⭐ Web 框架 | ⭐`axum` | ⭐ 当前主流，tokio 官方生态 |
| | `actix-web` / `rocket` | 性能强 / 易用性好 |
| ⭐ HTTP 客户端 | ⭐`reqwest` | — |
| ⭐ 数据库 | ⭐`sqlx` | ⭐ 异步，编译期校验 SQL |
| | `sea-orm` / `diesel` | ORM / 同步 ORM |
| | `redis` | — |
| ⭐ 日志 | ⭐`tracing` + `tracing-subscriber` | ⭐ 结构化日志 + 分布式追踪，新项目首选 |
| | `log` + `env_logger` | 传统门面，简单场景够用 |
| ⭐ CLI | ⭐`clap` | ⭐ 派生宏定义参数，功能完备 |
| 时间 | ⭐`chrono` / `time` | — |
| 正则 | ⭐`regex` | ⭐ 标准库没有正则 |
| 随机数 | ⭐`rand` | 标准库也没有随机数 |
| ⭐ 并行 | ⭐`rayon` | ⭐ `iter()` → `par_iter()` 一行并行 |
| 工具 | ⭐`itertools` | 补充迭代器能力 |
| | `uuid` / `bytes` / `dashmap` | UUID / 字节缓冲 / 并发 Map |
| 配置 | `config` / `dotenvy` | — |
| 加密 | `sha2` / `argon2` / `jsonwebtoken` | 哈希 / 密码 / JWT |
| ⭐ 测试 | ⭐`rstest`（参数化） | ⭐`mockall`（mock） |
| | `proptest`（属性测试）/ `insta`（快照） | `criterion`（基准） |

> ⚠ `once_cell` 的核心能力已进标准库：用 **`OnceLock`**（1.70+）和 **`LazyLock`**（1.80+）代替，新代码不必再引入这个依赖。

**一个典型后端服务的依赖组合**：

```toml
tokio = { version = "1", features = ["rt-multi-thread", "macros"] }
axum = "0.8"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres"] }
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
anyhow = "1"
thiserror = "2"
```

---

### 7. 日志与可观测性

⭐ **新项目一律用 `tracing`**（结构化 + 支持 span/分布式追踪），`log` 只适合小工具：

```rust
use tracing::{info, warn, error, instrument};

#[instrument(skip(password))]           // ⭐ 自动记录函数入参和耗时，跳过敏感字段
async fn login(user: &str, password: &str) -> Result<Token> {
    info!(user, "开始登录");            // ⭐ 结构化字段，而非拼接字符串
    warn!(retry = 3, "重试中");
    error!(?err, "登录失败");           // ? 表示用 Debug 格式
    Ok(token)
}

fn main() {
    tracing_subscriber::fmt()
        .with_env_filter("info,my_app=debug")       // ⭐ 通过 RUST_LOG 环境变量控制
        .json()                                      // 生产环境输出 JSON，便于采集
        .init();
}
```

> ⭐ **日志用结构化字段而不是字符串拼接** —— `info!(user_id = id, "创建成功")` 比 `info!("创建成功: {}", id)` 更利于检索和告警。

---

### 8. 高频踩坑点

**① 测试之间共享状态导致随机失败**

```rust
static mut COUNTER: i32 = 0;            // ✘ 并行测试会互相干扰
```

> 每个测试用独立的临时目录（`tempfile` crate）、独立的数据库 schema，或用 `--test-threads=1`。

**② `println!` 在测试里看不到输出**

```bash
cargo test -- --nocapture               # ⭐ 或 --show-output
```

**③ 集成测试访问不到内部函数**

`tests/` 只能用 `pub` API。要测私有逻辑就写单元测试，或用 `pub(crate)` + 单元测试。

**④ `tests/common.rs` 被当成测试 crate**

放成 `tests/common/mod.rs` 才是模块而不是独立测试目标。

**⑤ 用 debug 模式做性能测试**

debug 下没有优化，性能可能差 10~100 倍，结论完全不可信。必须 `--release`。

**⑥ 文档测试因为缺 `use` 而失败**

文档测试是**独立编译**的，示例里要写完整的 `use` 语句（不想显示就用 `#` 前缀隐藏）。

**⑦ 忘了 `#[cfg(test)]`**

测试模块没加 `#[cfg(test)]` 会被编译进发布产物，增大体积并可能引入测试依赖。

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| 单元测试写在源文件内 `#[cfg(test)] mod tests` | 能测私有函数，改代码时同步改测试 |
| 对外契约写集成测试 | 放 `tests/`，只用 `pub` API |
| 公开 API 必须有文档注释和示例 | 示例会被 `cargo test` 执行，永不过时 |
| 文档标准小节：`# Examples` / `# Panics` / `# Errors` | 会 panic 的函数必须注明 |
| 测试之间零共享状态 | 用临时目录/独立 schema，保证可并行 |
| CI 三件套：`fmt --check`、`clippy -D warnings`、`test` | 缺一不可 |
| 性能测试用 `criterion` + `--release` | 配 `black_box` 防优化 |
| 新项目日志用 `tracing` | 结构化字段，生产输出 JSON |
| 敏感信息不进日志 | `#[instrument(skip(password))]` |
| 依赖选型看维护状态和下载量 | 优先事实标准（serde/tokio/axum） |
| 定期 `cargo audit` | 检查依赖安全漏洞 |
| 用 `cargo-nextest` 加速测试 | 大型项目收益明显 |

---

### 10. 面试重点

**Q1：Rust 的单元测试和集成测试有什么区别？**

**单元测试**写在被测源文件内部的 `#[cfg(test)] mod tests` 里，与被测代码属于同一个 crate，因此**可以测试私有函数**；`#[cfg(test)]` 保证它只在 `cargo test` 时编译，不会进入发布产物。**集成测试**放在 `tests/` 目录，每个文件是**独立的 crate**，只能访问 `pub` API，用来验证对外契约和模块协作。

**Q2：什么是文档测试？**

Rust 会把文档注释（`///`）中 ` ``` ` 代码块里的代码**当作测试来编译和运行**（`cargo test` 会执行它们）。这保证了文档里的示例永远是可编译、可运行的，不会随着 API 变化而过时——这是 Rust 生态文档质量普遍很高的重要原因。可以用 `no_run`（只编译）、`ignore`（跳过）、`#` 前缀（执行但不显示）来控制。

**Q3：`cargo test` 是并行还是串行？有什么影响？**

**默认并行**（多线程执行不同测试）。影响是测试之间**不能共享可变状态** —— 共享数据库、写同一个文件、用全局变量都会导致随机失败（flaky test）。解决办法：每个测试用独立的临时目录（`tempfile`）或独立的数据库 schema；实在无法隔离时用 `cargo test -- --test-threads=1` 串行执行。另外测试默认**捕获标准输出**，要看 `println!` 得加 `-- --nocapture`。

**Q4：Rust 项目里常用哪些第三方库？**

序列化 `serde`/`serde_json`（事实标准）；错误处理 `thiserror`（库）+ `anyhow`（应用）；异步运行时 `tokio`；Web 框架 `axum`；HTTP 客户端 `reqwest`；数据库 `sqlx`（异步、编译期校验 SQL）；日志 `tracing` + `tracing-subscriber`；CLI `clap`；并行 `rayon`；正则 `regex`、随机数 `rand`（标准库都没有）；测试 `rstest`/`mockall`/`criterion`。

**Q5：`log` 和 `tracing` 怎么选？**

`log` 是传统的日志门面，只有级别 + 文本消息；`tracing` 提供**结构化字段**和 **span（跨度）**概念，能表达"一次请求经历了哪些阶段、每段耗时多少"，天然适配异步和分布式追踪（可对接 OpenTelemetry）。**新项目一律选 `tracing`**，配合 `#[instrument]` 宏可以自动记录函数入参和耗时，生产环境输出 JSON 便于日志采集与检索。

**Q6：为什么性能测试必须用 release 模式？**

debug 模式下编译器不做任何优化，还会插入整数溢出检查和额外的调试信息，性能可能比 release 慢 **10~100 倍**，而且各部分的相对开销也完全失真，得出的结论没有参考价值。另外基准测试要用 `criterion` 的 `black_box()` 包裹输入输出，防止编译器发现结果没被使用而把整段被测代码优化掉。

---

## 二十一、宏（macro_rules! 与过程宏）

> ⭐ 日常开发中**用宏远多于写宏**。面试要求通常是：说清宏和函数的区别、知道两大类宏、能写简单的 `macro_rules!`、看得懂框架里的宏。

### 1. 宏 vs 函数

⭐ **宏在编译期展开成代码，函数在运行期被调用**：

| 对比项 | 宏 | 函数 |
| ------ | -- | ---- |
| 执行时机 | ⭐ 编译期（展开成 AST） | 运行期 |
| 参数个数 | ⭐ **可变** | 固定 |
| 参数类型 | ⭐ 可接受任意语法片段（类型、表达式、语句） | 必须是值 |
| 能否生成新代码 | ⭐ 能（生成结构体、impl 块等） | 不能 |
| 定义位置 | ⭐`macro_rules!` 必须先定义后使用 | 任意 |
| 代价 | ⭐ 编译变慢、可读性差、报错难懂、IDE 支持弱 | — |

**Rust 为什么需要宏**：因为不支持函数重载、默认参数和可变参数 —— `println!("{} {}", a, b)` 这种事只有宏能做。

---

### 2. 常用内置宏

| 宏 | 作用 |
| -- | ---- |
| ⭐`println!` / `print!` / `eprintln!` | 标准输出 / 标准错误 |
| ⭐`format!` | 格式化成 `String` |
| ⭐`write!` / `writeln!` | 写入实现了 `Write` 的目标（自定义 `Display` 必用） |
| ⭐`vec!` | 创建 `Vec` |
| ⭐`panic!` / `assert!` / `assert_eq!` | 崩溃 / 断言 |
| ⭐`matches!` | 模式匹配判断（第十二章） |
| ⭐`todo!` / `unimplemented!` / `unreachable!` | 占位 / 未实现 / 不可达 |
| ⭐`dbg!` | ⭐ 打印表达式的值和行号并**返回该值**，调试神器 |
| `include_str!` / `include_bytes!` | 编译期嵌入文件内容 |
| `env!` / `option_env!` | 读取编译期环境变量 |
| `cfg!` | 编译期条件判断 |
| `concat!` / `stringify!` | 拼接字面量 / 把代码变成字符串 |
| `file!` / `line!` / `column!` | 当前位置信息 |

```rust
let x = dbg!(compute() * 2);            // ⭐ 打印 [src/main.rs:5] compute() * 2 = 10，并返回 10

const SQL: &str = include_str!("query.sql");        // ⭐ 把文件内容编译进二进制
let version = env!("CARGO_PKG_VERSION");            // ⭐ 读取 Cargo.toml 里的版本号
```

---

### 3. 声明宏 macro_rules!

⭐ **本质是"对语法做模式匹配 + 替换"**：

```rust
macro_rules! say_hello {
    () => {                             // ⭐ 匹配空参数
        println!("Hello!");
    };
}
say_hello!();
```

**带参数与多分支**：

```rust
macro_rules! calc {
    ($a:expr, $b:expr) => {
        $a + $b
    };
    (mul $a:expr, $b:expr) => {         // ⭐ 可以有多个规则，按顺序匹配
        $a * $b
    };
}

calc!(1, 2);            // 3
calc!(mul 3, 4);        // 12
```

**⭐ 片段说明符（fragment specifier）**：

| 说明符 | 匹配内容 | 示例 |
| ------ | -------- | ---- |
| ⭐`expr` | 表达式 | `1 + 2`、`foo()` |
| ⭐`ident` | 标识符 | 变量名、函数名 |
| ⭐`ty` | 类型 | `i32`、`Vec<String>` |
| `pat` | 模式 | `Some(x)` |
| `stmt` / `block` | 语句 / 代码块 | — |
| ⭐`item` | 条目 | 整个 `fn`、`struct`、`impl` |
| `literal` | 字面量 | `42`、`"abc"` |
| `path` | 路径 | `std::collections::HashMap` |
| `tt` | ⭐ 单个 token 树（最灵活，递归宏常用） | — |
| `meta` / `vis` / `lifetime` | 属性内容 / 可见性 / 生命周期 | — |

**⭐ 重复（repetition）**：`$( ... )` 加分隔符和次数符：

```rust
macro_rules! my_vec {
    () => { Vec::new() };

    ($($x:expr),+ $(,)?) => {{          // ⭐ 逗号分隔、至少一个、允许尾随逗号
        let mut v = Vec::new();
        $( v.push($x); )+               // ⭐ 对每个 $x 重复展开
        v
    }};
}

let v = my_vec![1, 2, 3];
```

| 次数符 | 含义 |
| ------ | ---- |
| `*` | 0 次或多次 |
| `+` | 1 次或多次 |
| `?` | 0 次或 1 次（⭐ 常用于允许尾随逗号） |

**实用示例：构造 HashMap**

```rust
macro_rules! hashmap {
    ($($k:expr => $v:expr),* $(,)?) => {{
        let mut m = std::collections::HashMap::new();
        $( m.insert($k, $v); )*
        m
    }};
}

let m = hashmap!{ "a" => 1, "b" => 2 };
```

**导出宏**：

```rust
#[macro_export]                         // ⭐ 导出到 crate 根，下游 use my_crate::hashmap;
macro_rules! hashmap { ... }
```

---

### 4. 卫生性（Hygiene）

⭐ **宏内部定义的局部变量不会和调用处的变量冲突**：

```rust
macro_rules! bad_swap {
    ($a:expr, $b:expr) => {{
        let tmp = $a;                   // ⭐ 这个 tmp 与外部的 tmp 是不同的变量
        $a = $b;
        $b = tmp;
    }};
}

let mut tmp = 1;
let mut x = 2;
bad_swap!(tmp, x);                      // ✔ 不会互相干扰（C 的宏在这里就出错了）
```

> 卫生性只对**局部变量**生效，对类型名、函数名、常量等 item 不生效（宏里写 `Vec::new()` 用的是调用处能看到的 `Vec`），所以宏里引用外部路径应写全 `::std::vec::Vec` 或 `$crate::`。

---

### 5. 过程宏（Procedural Macro）

⭐ **过程宏是"输入 TokenStream，输出 TokenStream"的编译期程序**，能力远超声明宏，必须放在**独立的 crate** 里：

```toml
# my-macros/Cargo.toml
[lib]
proc-macro = true

[dependencies]
syn = { version = "2", features = ["full"] }
quote = "1"
proc-macro2 = "1"
```

**三种类型**：

| 类型 | 写法 | 典型例子 |
| ---- | ---- | -------- |
| ⭐ 派生宏 | `#[derive(X)]` | ⭐`#[derive(Debug, Clone, Serialize)]` |
| ⭐ 属性宏 | `#[attr]` | ⭐`#[tokio::main]`、`#[async_trait]`、`#[instrument]` |
| 函数式宏 | `my_macro!(...)` | `sqlx::query!()`、`json!()` |

**一个最小的派生宏**：

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(Hello)]
pub fn derive_hello(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);      // ⭐ syn 解析成语法树
    let name = &ast.ident;

    quote! {                                                 // ⭐ quote 生成代码
        impl #name {
            fn hello() { println!("Hello, {}!", stringify!(#name)); }
        }
    }.into()
}
```

```rust
#[derive(Hello)]
struct World;
World::hello();                         // Hello, World!
```

**⭐ 框架里到处是过程宏，看懂它们做了什么很重要**：

```rust
#[derive(Serialize, Deserialize)]       // ⭐ 生成序列化/反序列化的 impl
struct User { name: String }

#[tokio::main]                          // ⭐ 把 async fn main 包装成同步 main + 启动运行时
async fn main() { }

#[derive(Parser)]                       // ⭐ clap：根据结构体字段生成命令行解析代码
struct Args { #[arg(short, long)] name: String }

let row = sqlx::query!("SELECT id FROM users WHERE id = $1", id);   // ⭐ 编译期校验 SQL
```

**⭐ 调试宏展开**：

```bash
cargo install cargo-expand
cargo expand                            # ⭐ 查看宏展开后的真实代码
```

---

### 6. 高频踩坑点

**① 三种括号等价，但有约定**

```rust
vec![1, 2, 3];          // ⭐ 集合类惯用 []
println!("{}", x);      // ⭐ 函数式惯用 ()
macro_rules! m { }      // 定义块用 {}
```

**② `macro_rules!` 顺序敏感**

```rust
say_hello!();                           // ✘ 定义在后面，找不到
macro_rules! say_hello { ... }
```

> 声明宏必须**先定义后使用**（过程宏不受此限制）。所以宏通常放在 `lib.rs` 顶部，或用 `#[macro_use] mod macros;`。

**③ 展开后的运算符优先级**

```rust
macro_rules! double { ($x:expr) => { $x * 2 }; }
double!(1 + 2);                         // ⭐ 实际是 (1+2)*2 = 6 ✔
```

⭐ `expr` 片段被捕获后是一个**整体**，Rust 宏不像 C 那样需要到处加括号。但宏体内自己拼接的表达式仍应加括号更保险。

**④ 递归宏的展开深度限制**

```rust
#![recursion_limit = "256"]             // 默认 128，深度递归的宏需要调大
```

**⑤ 宏里的路径不卫生**

```rust
macro_rules! new_vec { () => { Vec::new() }; }       // ⚠ 依赖调用处能看到 Vec
macro_rules! new_vec { () => { ::std::vec::Vec::new() }; }      // ✔ 写全路径
macro_rules! m { () => { $crate::helper() }; }       // ⭐ $crate 指向宏定义所在的 crate
```

**⑥ 过程宏拖慢编译**

`syn` + `quote` 本身编译就很重，大量使用派生宏（尤其 `serde`）会明显增加编译时间。可以按需开 feature，或对不需要的类型不 derive。

**⑦ 宏的报错信息很难懂**

宏展开后的错误往往指向宏内部而非调用处。排查时先 `cargo expand` 看展开结果。

**⑧ 滥用宏**

```rust
macro_rules! add { ($a:expr, $b:expr) => { $a + $b }; }   // ⚠ 函数就够了
fn add(a: i32, b: i32) -> i32 { a + b }                   // ✔
```

---

### 7. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 能用函数/泛型解决就不用宏 | 宏是最后手段 |
| 宏只用于消除**真正无法抽象**的样板 | 可变参数、生成 impl、编译期校验 |
| 声明宏放在专门的 `macros.rs` | 且在 `lib.rs` 顶部 `mod macros;` |
| 宏内引用路径写全或用 `$crate` | 避免依赖调用处的作用域 |
| 支持尾随逗号 `$(,)?` | 与标准库宏行为一致 |
| 公开宏用 `#[macro_export]` 并写文档 | 宏也能写文档注释和 doctest |
| 过程宏单独成 crate 并控制依赖 | 编译成本高 |
| 排查问题用 `cargo expand` | 别对着展开前的代码猜 |
| `dbg!` 不能进生产代码 | CI 用 clippy 拦截 |
| 谨慎 derive 大量宏 | 每个 derive 都增加编译时间 |

---

### 8. 面试重点

**Q1：宏和函数有什么区别？**

宏在**编译期展开**成代码（操作的是语法树），函数在**运行期调用**（操作的是值）。宏可以接受**可变数量的参数**、接受**任意语法片段**（类型、表达式、语句），还能**生成新的代码**（结构体、`impl` 块）——这些函数都做不到。Rust 之所以大量依赖宏，正是因为它不支持函数重载、默认参数和可变参数，`println!("{} {}", a, b)` 只能靠宏实现。代价是编译变慢、报错难懂、IDE 支持弱。

**Q2：Rust 有哪几类宏？**

两大类：① **声明宏**（`macro_rules!`）——基于**语法模式匹配 + 替换**，写起来简单，适合消除重复代码，如 `vec!`；② **过程宏**——本质是编译期运行的 Rust 程序，输入输出都是 `TokenStream`，必须放在独立的 `proc-macro` crate 里，又分三种：**派生宏** `#[derive(Serialize)]`、**属性宏** `#[tokio::main]`、**函数式宏** `sqlx::query!()`。过程宏能力强得多（可以解析整个类型定义并生成任意代码），代价是实现复杂、依赖 `syn`/`quote`、编译更慢。

**Q3：什么是宏的卫生性（hygiene）？**

宏内部定义的**局部变量**与调用处的同名变量互不干扰——编译器会给宏内的标识符打上不同的语法上下文标记。所以在 Rust 里写 `let tmp = ...;` 的交换宏不会像 C 的宏那样被外部的 `tmp` 破坏。注意卫生性**只覆盖局部变量**，不覆盖类型名、函数名、路径，所以宏内引用外部项应写全路径或用 `$crate`。

**Q4：`#[derive(Debug)]` 背后发生了什么？**

`derive` 是**派生宏**：编译器把结构体/枚举的定义作为 `TokenStream` 传给宏，宏解析出字段信息后生成一段 `impl Debug for XXX { ... }` 的代码，再插回到编译流程中。`Debug`、`Clone`、`Serialize` 都是这个机制。可以用 `cargo expand` 查看生成的真实代码。

**Q5：常见的片段说明符有哪些？**

`expr`（表达式）、`ident`（标识符）、`ty`（类型）、`pat`（模式）、`stmt`（语句）、`block`（代码块）、`item`（整个条目如 `fn`/`struct`）、`literal`（字面量）、`path`（路径）、`tt`（单个 token 树，最灵活，递归宏常用）、`meta`/`vis`/`lifetime`。配合重复语法 `$( ... ),*`（`*` 零次或多次、`+` 一次或多次、`?` 零次或一次）就能处理可变参数。

**Q6：什么时候应该写宏？**

**尽量不写**。优先用函数、泛型、trait 解决；只有当需求本质上无法用它们表达时才考虑宏：① 可变数量参数（`vec!`、`println!`）；② 需要**生成代码**（自动实现 trait、生成结构体）；③ 需要**编译期校验**（`sqlx::query!` 在编译期连数据库校验 SQL）；④ 需要捕获调用位置信息（`dbg!`、`assert!` 打印表达式原文）。滥用宏会严重损害可读性、编译速度和调试体验。

---

## 二十二、多线程与并发

> ⭐ **"无畏并发（Fearless Concurrency）"是 Rust 最响亮的卖点**：所有权 + 借用规则 + `Send`/`Sync` 让**数据竞争在编译期就无法通过**。这一章是高级岗位面试的重点。

### 1. 为什么 Rust 能"无畏并发"

数据竞争（data race）需要三个条件**同时**成立：

```text
① 两个以上线程同时访问同一份数据
② 至少有一个在写
③ 没有同步机制
```

⭐ **借用规则（第九章）直接让 ① 和 ② 无法共存**："有可变引用时不能有任何其他引用"。再加上 `Send`/`Sync` 约束跨线程传递的合法性，**Rust 把数据竞争从运行时的玄学问题变成了编译错误**。

> ⚠ 注意：Rust 消灭的是**数据竞争（data race）**，不是所有并发问题。**死锁、竞态条件（race condition）、活锁仍然可能发生** —— 这是面试常设的陷阱。

---

### 2. 创建线程

```rust
use std::thread;

let handle = thread::spawn(|| {
    println!("子线程");
    42                                  // ⭐ 返回值通过 join 拿到
});

let result = handle.join().unwrap();    // ⭐ 等待结束，返回 Result（子线程 panic 则是 Err）
```

**⭐ 闭包必须满足 `Send + 'static`**（第十章）：

```rust
let data = vec![1, 2, 3];

thread::spawn(|| println!("{:?}", data));           // ✘ 只是借用，不满足 'static
thread::spawn(move || println!("{:?}", data));      // ⭐ ✔ move 移入所有权
```

**⭐ 作用域线程（scoped threads，Rust 1.63+）：可以安全借用栈上数据**：

```rust
let mut data = vec![1, 2, 3];

thread::scope(|s| {                     // ⭐ 保证所有线程在作用域结束前 join
    s.spawn(|| println!("{:?}", &data));         // ✔ 直接借用，不需要 move + Arc
    s.spawn(|| println!("len = {}", data.len()));
});                                     // ⭐ 到这里所有线程已结束，借用安全
data.push(4);                           // ✔
```

> 需要并行处理一份栈上数据时，`thread::scope` 比 `Arc` 简单得多。

---

### 3. Send 与 Sync（核心考点）

⭐ **两个标记 trait 定义了"什么类型可以跨线程"**：

| Trait | 含义 | 通俗解释 |
| ----- | ---- | -------- |
| ⭐`Send` | 类型的**所有权**可以在线程间**转移** | ⭐ 可以"送给"另一个线程 |
| ⭐`Sync` | 类型可以被多个线程**同时引用**（`&T` 是 `Send`） | ⭐ 可以被多线程"共享读" |

⭐ **关系：`T: Sync` ⟺ `&T: Send`**。

**⭐ 它们是自动 trait（auto trait）**：编译器自动推导 —— 一个类型的所有字段都是 `Send`，它就是 `Send`。绝大多数类型两者都满足，**记住例外即可**：

| 类型 | `Send` | `Sync` | 原因 |
| ---- | :----: | :----: | ---- |
| 大多数类型（`i32`、`String`、`Vec<T>`） | ✔ | ✔ | — |
| ⭐`Rc<T>` | ✘ | ✘ | ⭐ 引用计数**非原子**，并发增减会算错 |
| ⭐`RefCell<T>` / `Cell<T>` | ✔ | ⭐ ✘ | ⭐ 借用计数非原子，多线程同时 `borrow_mut` 会绕过检查 |
| ⭐`Arc<T>` | ✔（需 `T: Send+Sync`） | ✔ | 原子计数 |
| ⭐`Mutex<T>` / `RwLock<T>` | ✔（需 `T: Send`） | ⭐ ✔ | ⭐ **锁把非 `Sync` 的类型变成 `Sync`** |
| 裸指针 `*const T` | ✘ | ✘ | 无法保证安全 |
| `MutexGuard<T>` | ✘ | ✔ | 某些平台要求同线程解锁 |

```rust
let rc = Rc::new(1);
thread::spawn(move || println!("{rc}"));            // ✘ `Rc<i32>` cannot be sent between threads
let arc = Arc::new(1);
thread::spawn(move || println!("{arc}"));           // ✔
```

> ⭐ **`Arc<Mutex<T>>` 是最常见的组合**：`Arc` 让多线程都能拥有，`Mutex` 让内部数据可以安全修改。

---

### 4. 消息传递（Channel）

⭐ **Rust 官方推崇的并发范式：「不要通过共享内存来通信，而要通过通信来共享内存」**（源自 Go）。

```rust
use std::sync::mpsc;                    // ⭐ multi-producer, single-consumer

let (tx, rx) = mpsc::channel();

for i in 0..3 {
    let tx = tx.clone();                // ⭐ 发送端可克隆，多生产者
    thread::spawn(move || tx.send(i).unwrap());
}
drop(tx);                               // ⭐ 关键：丢弃原始 tx，否则 rx 永远收不到结束信号

for received in rx {                    // ⭐ 迭代直到所有发送端关闭
    println!("{received}");
}
```

| API | 说明 |
| --- | ---- |
| `tx.send(v)` | 发送，返回 `Result`（接收端关闭则 `Err`） |
| `rx.recv()` | ⭐ 阻塞接收，所有发送端关闭则 `Err` |
| `rx.try_recv()` | 非阻塞 |
| `rx.recv_timeout(d)` | 带超时 |
| `mpsc::sync_channel(n)` | ⭐ 有界通道，满了会阻塞发送方（背压） |

> 需要**多消费者**（mpmc）或更高性能时用 `crossbeam-channel`；异步场景用 `tokio::sync::mpsc`（第二十三章）。

---

### 5. 共享状态（Mutex / RwLock）

```rust
use std::sync::{Arc, Mutex};

let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);         // ⭐ 每个线程一份 Arc
    handles.push(thread::spawn(move || {
        let mut n = counter.lock().unwrap();    // ⭐ 加锁，返回守卫
        *n += 1;
    }));                                        // ⭐ 守卫在此 drop，自动解锁
}
for h in handles { h.join().unwrap(); }
println!("{}", *counter.lock().unwrap());       // 10
```

**⭐ Rust 的锁把数据"包在里面"**，而不是像 Java 那样锁一个对象再自觉访问数据 —— **不加锁就拿不到数据，从类型上杜绝了忘记加锁**。

**`RwLock`：读多写少时用**：

```rust
use std::sync::RwLock;

let cfg = Arc::new(RwLock::new(Config::default()));
{
    let r = cfg.read().unwrap();        // ⭐ 多个读可以并发
    println!("{}", r.host);
}
{
    let mut w = cfg.write().unwrap();   // ⭐ 写独占
    w.host = "new".into();
}
```

| 对比 | `Mutex` | `RwLock` |
| ---- | ------- | -------- |
| 并发读 | ✘ 互斥 | ⭐ ✔ 多读并发 |
| 开销 | 小 | 较大 |
| 适用 | ⭐ 读写都频繁 / 临界区很短 | ⭐ 读远多于写（配置、缓存） |

> ⚠ **锁中毒（poisoning）**：持锁线程 panic 后，锁进入"中毒"状态，之后的 `lock()` 返回 `Err`。这就是到处要写 `.unwrap()` 的原因。想避免可以用 `parking_lot::Mutex`（更快、无中毒、`lock()` 直接返回守卫）。

---

### 6. 原子类型

⭐ **简单计数、标志位用原子类型，比锁快得多**：

```rust
use std::sync::atomic::{AtomicU64, AtomicBool, Ordering};

static COUNTER: AtomicU64 = AtomicU64::new(0);      // ⭐ 可以直接做 static

COUNTER.fetch_add(1, Ordering::Relaxed);
COUNTER.load(Ordering::Relaxed);
flag.store(true, Ordering::Release);
flag.compare_exchange(old, new, Ordering::AcqRel, Ordering::Acquire);   // CAS
```

**内存序（Ordering）**：

| 值 | 说明 |
| -- | ---- |
| ⭐`Relaxed` | 只保证原子性，⭐ 计数器用它就够 |
| `Acquire` / `Release` | 读端/写端配对使用，建立 happens-before 关系 |
| `AcqRel` | 读改写操作同时需要两者 |
| ⭐`SeqCst` | 全序，最强也最慢，⭐ **不确定就用它** |

> 内存序是深水区。工程建议：**纯计数器用 `Relaxed`，其他情况用 `SeqCst`**，需要极致性能再研究 `Acquire`/`Release`。

---

### 7. 死锁与常见并发问题

⭐ **Rust 不能防止死锁**，以下四种情况照样卡死：

**① `Mutex` 不可重入 —— 同一线程二次加锁直接死锁**

```rust
let m = Mutex::new(0);
let a = m.lock().unwrap();
let b = m.lock().unwrap();              // ⭐ ✘ 死锁！Rust 的 Mutex 不可重入
```

**② 加锁顺序不一致**

```rust
// 线程 1：先锁 A 再锁 B      线程 2：先锁 B 再锁 A     ⭐ 经典死锁
```

> ⭐ **规范：全局约定一个固定的加锁顺序**（比如按变量地址或业务层级排序）。

**③ 守卫存活过久**

```rust
let guard = data.lock().unwrap();
expensive_io().await;                   // ⚠ 持锁做 IO，其他线程全部阻塞
db_query();
```

```rust
// ✔ 缩小临界区
let value = { data.lock().unwrap().clone() };       // ⭐ 取出就释放
expensive_io(value);
```

**④ `let _ = lock()` 立即释放（第一章的坑）**

```rust
let _ = m.lock().unwrap();              // ✘ 锁当场释放，等于没锁
let _guard = m.lock().unwrap();         // ✔
```

**其他并发问题**：**竞态条件**（逻辑层面的时序依赖，如"先检查后使用"）、**活锁**、**假共享**（不同线程频繁写同一缓存行的相邻数据，性能骤降）—— 这些 Rust 都管不了。

---

### 8. 线程池与并行计算

**⭐ 数据并行用 `rayon`，一行代码搞定**：

```rust
use rayon::prelude::*;

let sum: u64 = data.par_iter().map(|x| expensive(x)).sum();     // ⭐ iter → par_iter
data.par_sort();
results.par_iter_mut().for_each(|x| *x = process(x));
```

`rayon` 内部是工作窃取（work-stealing）线程池，自动按 CPU 核数调度，**API 与标准迭代器完全一致**。

**其他同步原语**：

```rust
use std::sync::{Barrier, Condvar, OnceLock};

static CONFIG: OnceLock<Config> = OnceLock::new();      // ⭐ 全局只初始化一次
CONFIG.get_or_init(|| load_config());
```

---

### 9. 高频踩坑点（重点）

**① `Rc` 跨线程 → 改 `Arc`**

报错 `` `Rc<T>` cannot be sent between threads safely `` 时，换成 `Arc` 即可。

**② `RefCell` 跨线程 → 改 `Mutex`**

```rust
Arc<RefCell<T>>                         // ✘ RefCell 不是 Sync
Arc<Mutex<T>>                           // ✔
```

**③ 忘记 `drop(tx)` 导致接收端永远阻塞**

```rust
let (tx, rx) = mpsc::channel();
for i in 0..3 { let tx = tx.clone(); thread::spawn(move || tx.send(i)); }
// ⭐ 必须 drop(tx)，否则原始 tx 还活着，rx 的循环永不结束
drop(tx);
for r in rx { }
```

**④ 主线程退出，子线程被杀**

```rust
thread::spawn(|| { /* 长任务 */ });
// main 结束 → 进程退出 → 子线程直接被终止
handle.join().unwrap();                 // ✔ 必须 join
```

**⑤ 持锁跨 `.await`（异步场景）**

```rust
let guard = data.lock().unwrap();
some_async().await;                     // ⚠ 标准库锁跨 await 极易死锁
```

> 异步代码里要么在 `.await` 前释放锁，要么用 `tokio::sync::Mutex`（第二十三章）。

**⑥ 锁粒度过粗**

```rust
Mutex<HashMap<K, V>>                    // ⚠ 整个 map 一把锁，并发度低
DashMap<K, V>                           // ⭐ 分段锁并发 Map
// 或者按 key 分片：Vec<Mutex<HashMap<K,V>>>
```

**⑦ 用共享状态做本该用通道的事**

大量 `Arc<Mutex<>>` 说明设计偏向共享内存。**优先考虑通道 + 单一所有者（actor 模式）**：把状态交给一个线程独占，其他线程通过消息与它交互，既没有锁也没有死锁。

**⑧ `unwrap()` 处理锁中毒**

`lock().unwrap()` 在锁中毒时会 panic 并扩散。生产代码可以显式处理，或直接用 `parking_lot`。

---

### 10. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 优先消息传递，其次共享状态 | 通道 + actor 模式没有死锁风险 |
| 共享只读用 `Arc<T>` | 不需要锁 |
| 共享可变用 `Arc<Mutex<T>>` | 读多写少才用 `RwLock` |
| 简单计数/标志用原子类型 | 比锁快一个数量级 |
| ⭐ 临界区最小化 | ⭐ 绝不持锁做 IO、网络、日志 |
| 全局统一加锁顺序 | 防止死锁 |
| 锁守卫绑定具名变量 | 绝不写 `let _ = lock()` |
| 高并发 Map 用 `DashMap` 或分片 | 避免一把大锁 |
| CPU 密集型批处理用 `rayon` | `par_iter()` 一行并行 |
| 全局单次初始化用 `OnceLock`/`LazyLock` | 不用 `static mut` |
| 追求性能可换 `parking_lot` | 更快、无中毒、体积更小 |
| 并发代码必须有压力测试 | 死锁和竞态只在高并发下暴露 |

---

### 11. 面试重点

**Q1：Rust 是怎么做到"无畏并发"的？**

靠**所有权 + 借用规则 + `Send`/`Sync`** 三者配合。数据竞争需要"多线程访问 + 至少一个写 + 无同步"三个条件同时成立，而借用规则保证"存在可变引用时不能有任何其他引用"，直接让前两条无法共存；`Send`/`Sync` 则在类型层面约束了什么能跨线程转移、什么能跨线程共享。所以**数据竞争在 Rust 里是编译错误**。⚠ 但 Rust **不能防止死锁、竞态条件和活锁** —— 这是常见的追问点。

**Q2：`Send` 和 `Sync` 是什么？**

都是**自动推导的标记 trait**（auto trait），没有任何方法。`Send` 表示类型的**所有权可以安全地转移到另一个线程**；`Sync` 表示类型可以被**多个线程同时引用**，等价定义是 `T: Sync ⟺ &T: Send`。编译器会根据字段自动推导，绝大多数类型两者都满足。

**Q3：哪些类型不是 `Send` 或 `Sync`？为什么？**

⭐ **`Rc<T>` 既不是 `Send` 也不是 `Sync`** —— 引用计数是**非原子**的，多线程并发增减会导致计数错误，进而 double free 或内存泄漏；对应的线程安全版本是 `Arc<T>`。⭐ **`RefCell<T>`/`Cell<T>` 是 `Send` 但不是 `Sync`** —— 借用计数同样非原子，多个线程同时 `borrow_mut()` 会绕过运行时检查造成数据竞争；多线程要共享可变数据得用 `Mutex`/`RwLock`。另外裸指针两者都不是。

**Q4：`Mutex<T>` 为什么能让非 `Sync` 的类型变成 `Sync`？**

因为 `Mutex` 保证同一时刻**只有一个线程能拿到内部数据的引用**，从而人为地满足了 `Sync` 的要求。所以 `Mutex<T>: Sync` 的条件只是 `T: Send`（能被移交给持锁的那个线程），不要求 `T: Sync`。这也是 `Arc<Mutex<T>>` 成为标准共享可变组合的原因。

**Q5：Rust 的 `Mutex` 和 Java 的 `synchronized` 有什么区别？**

① ⭐ **Rust 的锁把数据"包在里面"**（`Mutex<T>`），不 `lock()` 就拿不到数据，**类型系统强制你加锁**；Java 是锁一个对象，访问数据靠程序员自觉，忘记加锁编译器不管。② Rust 的锁通过 **RAII 守卫**自动释放（守卫 drop 时解锁），不需要 `finally`。③ ⭐ **Rust 的 `Mutex` 不可重入**，同一线程二次 `lock()` 会死锁，而 Java 的 `synchronized` 是可重入的。④ Rust 有**锁中毒**机制：持锁线程 panic 后 `lock()` 返回 `Err`。

**Q6：消息传递和共享内存怎么选？**

Rust 官方推崇"**不要通过共享内存来通信，而要通过通信来共享内存**"。**通道（channel）+ 单一所有者（actor 模式）**没有锁、没有死锁风险、逻辑更清晰，适合任务分发、流水线、事件驱动；**共享状态（`Arc<Mutex<T>>`）**在需要多线程频繁读写同一份数据、且通道的复制/传递开销不可接受时使用（如大型缓存、连接池）。实践中如果代码里 `Arc<Mutex<>>` 满天飞，通常说明该改成 actor 模式了。

**Q7：怎么避免死锁？**

① **全局统一加锁顺序**（所有线程按同一顺序获取多把锁）；② **最小化临界区**，绝不持锁做 IO、网络请求、调用外部回调；③ 注意 **`Mutex` 不可重入**，同一线程不要二次加锁；④ 守卫必须绑定具名变量（`let _guard = ...`，不能写 `let _ = ...`）；⑤ 异步代码不要跨 `.await` 持有标准库锁；⑥ 能用通道就不用锁。必要时用带超时的 `try_lock_for`（`parking_lot` 提供）来暴露问题。

**Q8：`Mutex` 和 `RwLock` 怎么选？原子类型呢？**

`Mutex` 无论读写都互斥，开销小，适合临界区很短或读写都频繁的场景；`RwLock` 允许**多个读并发**但写独占，内部记账更复杂、开销更大，只有在**读远多于写**（配置、路由表、缓存）时才划算。如果共享的只是一个计数器或布尔标志，应该直接用 **`AtomicU64`/`AtomicBool`** —— 无锁，比任何锁都快一个数量级。

---

## 二十三、异步编程（async/await 与 Tokio）

> ⭐ 后端岗位的核心技能。Rust 的异步是**零成本抽象 + 无 GC** 的高并发方案，但心智负担明显高于同步代码。

### 1. 异步 vs 多线程

| 对比项 | 多线程 | ⭐ 异步（async） |
| ------ | ------ | --------------- |
| 并发单位 | OS 线程（栈默认 2~8 MB） | ⭐ 任务/Future（几十字节到几 KB） |
| 数量级 | 几千个就吃不消 | ⭐ **几十万个轻松** |
| 切换成本 | 内核态上下文切换 | ⭐ 用户态状态机跳转，极低 |
| 适合场景 | ⭐ **CPU 密集** | ⭐ **IO 密集、高并发连接** |
| 心智负担 | 低 | ⭐ 高（染色、Send 约束、取消语义） |

⭐ **一句话选型：等待 IO 的用异步，占用 CPU 的用线程（或 `rayon`）。** Web 服务、数据库客户端、消息队列消费者几乎全是 IO 密集，所以 Rust 后端基本等于 Tokio 生态。

---

### 2. async/await 基础

```rust
async fn fetch(url: &str) -> Result<String> {       // ⭐ 返回 impl Future<Output = Result<String>>
    let resp = reqwest::get(url).await?;            // ⭐ .await 是后缀语法，可链式
    let body = resp.text().await?;
    Ok(body)
}

let fut = fetch("https://x.com");                   // ⭐ 什么都没发生！Future 是惰性的
let body = fut.await?;                              // ⭐ 到这里才真正执行
```

**⭐ 三个必须理解的点**：

```rust
// ① Future 惰性：不 await 就不执行（和迭代器一样）
fetch(url);                             // ⚠ warning: unused implementer of `Future`

// ② async 块
let fut = async { 1 + 1 };
let fut = async move { data.len() };    // ⭐ move 捕获所有权

// ③ ⭐ 函数染色：async 只能在 async 上下文里 await
fn sync_fn() { fetch(url).await; }      // ✘ `await` only allowed inside `async`
```

> ⭐ **"函数染色"（function coloring）**是异步最大的心智负担：一个函数变成 `async`，所有调用它的函数都得跟着变 `async`，一直传染到 `main`。

---

### 3. Future 与状态机原理

```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

enum Poll<T> { Ready(T), Pending }
```

⭐ **`async fn` 会被编译器改写成一个状态机**：每个 `.await` 是一个状态切换点。

```text
async fn f() {
    let a = step1().await;      ← 状态 0：调用 step1，未完成则返回 Pending
    let b = step2(a).await;     ← 状态 1：从这里恢复
    done(b);                    ← 状态 2：完成
}

运行时不断 poll：
  Pending → 挂起，注册唤醒器（Waker），线程去干别的
  IO 就绪 → Waker 唤醒任务 → 再次 poll → 从上次的状态继续
```

⭐ **这就是异步"零成本"的原因**：没有额外线程、没有栈切换，只是一个枚举状态机 + 一次函数调用。

**⭐ `Pin` 为什么存在**：状态机里可能保存跨 `.await` 的**引用**（比如 `let x = ...; foo(&x).await;`），这使 Future 成为**自引用结构**（第九章说过 Rust 不支持自引用）。一旦 Future 被移动，内部引用就会失效。`Pin<&mut T>` 的作用就是**保证被轮询的 Future 不会再被移动**。

> 日常业务开发几乎不用手写 `Pin`，只需要知道：**遇到 `Pin` 相关报错，多半是异步递归或手动实现 `Future`**，加 `Box::pin` 通常能解决。

---

### 4. 运行时：Tokio

⭐ **Rust 标准库没有异步运行时** —— 语言只提供 `async/await` 语法和 `Future` trait，**执行器（executor）、IO 事件循环、定时器全部由第三方提供**。事实标准是 **Tokio**。

```toml
tokio = { version = "1", features = ["rt-multi-thread", "macros", "time", "net"] }
```

```rust
#[tokio::main]                          // ⭐ 宏展开成：创建运行时 + block_on(async main)
async fn main() {
    let body = fetch(url).await.unwrap();
}

// 等价于
fn main() {
    tokio::runtime::Builder::new_multi_thread()
        .worker_threads(4)
        .enable_all()
        .build().unwrap()
        .block_on(async { fetch(url).await });
}

#[tokio::test]                          // ⭐ 异步测试
async fn test_fetch() { }
```

**⭐ 任务（task）= 异步世界的"线程"**：

```rust
let handle = tokio::spawn(async {       // ⭐ 交给运行时并发执行，立即返回
    heavy_io().await
});
let result = handle.await?;             // ⭐ 等待结果（注意是 .await 不是 join()）
```

`tokio::spawn` 要求 Future 满足 **`Send + 'static`**（会被调度到任意工作线程）。

---

### 5. 并发组合：join / select / 限流

**⭐ 顺序 await ≠ 并发**，这是新人最常见的性能问题：

```rust
let a = fetch(u1).await?;               // ⚠ 串行：总耗时 = t1 + t2 + t3
let b = fetch(u2).await?;
let c = fetch(u3).await?;

// ⭐ ✔ 并发：总耗时 = max(t1, t2, t3)
let (a, b, c) = tokio::try_join!(fetch(u1), fetch(u2), fetch(u3))?;
let (a, b) = tokio::join!(f1, f2);                      // 不带错误短路的版本
```

**批量并发**：

```rust
use futures::future::join_all;
let results = join_all(urls.iter().map(|u| fetch(u))).await;

// ⭐ 带并发上限的批处理（防止打爆下游）
use futures::stream::{self, StreamExt};
let results: Vec<_> = stream::iter(urls)
    .map(|u| fetch(u))
    .buffer_unordered(10)               // ⭐ 最多 10 个并发
    .collect().await;
```

**`select!`：谁先完成用谁**（超时、取消的基础）：

```rust
tokio::select! {
    r = fetch(url) => println!("{r:?}"),
    _ = tokio::time::sleep(Duration::from_secs(3)) => println!("超时"),
    _ = shutdown.recv() => println!("收到关闭信号"),
}

// ⭐ 超时的简写
let r = tokio::time::timeout(Duration::from_secs(3), fetch(url)).await??;
```

> ⚠ **`select!` 会 drop 掉未完成的分支** —— 如果那个 Future 已经消费了部分数据（如从 channel 读了一半），数据就丢了。这叫**取消安全性（cancellation safety）**，用 `select!` 时必须确认各分支是否 cancel-safe。

---

### 6. 异步同步原语

⭐ **`tokio::sync` 下的版本在等待时会让出线程，而不是阻塞它**：

| 原语 | 说明 |
| ---- | ---- |
| ⭐`mpsc::channel(n)` | 多生产单消费，⭐ 有界（自带背压） |
| ⭐`oneshot::channel()` | 一次性，用于返回单个结果 |
| `broadcast::channel(n)` | 多生产多消费，每个消费者都收到全部消息 |
| ⭐`watch::channel(v)` | 只关心最新值（配置热更新、关闭信号） |
| ⭐`Mutex` / `RwLock` | 异步锁，`lock().await` |
| ⭐`Semaphore` | ⭐ 限流、限并发的标准手段 |
| `Notify` | 异步条件变量 |

```rust
// ⭐ 用信号量限制并发
let sem = Arc::new(Semaphore::new(10));
for url in urls {
    let permit = sem.clone().acquire_owned().await?;
    tokio::spawn(async move {
        fetch(&url).await;
        drop(permit);                   // ⭐ 释放名额
    });
}
```

**⭐ 标准库 `Mutex` vs `tokio::sync::Mutex` 怎么选**（高频面试题）：

| 场景 | 用哪个 |
| ---- | ------ |
| ⭐ 临界区**不跨** `.await` | ⭐ **`std::sync::Mutex`**（更快，无异步开销） |
| ⭐ 临界区**需要跨** `.await` | ⭐ `tokio::sync::Mutex` |

```rust
// ✔ 推荐：缩小临界区，用标准库锁
let value = { state.lock().unwrap().counter };      // ⭐ 守卫在块结束时释放
do_async(value).await;

// ✘ 危险：标准库锁跨 await，守卫非 Send，且会阻塞整个工作线程
let guard = state.lock().unwrap();
do_async().await;
```

---

### 7. 阻塞与异步的边界（重点）

⭐ **在异步任务里调用阻塞代码，是 Rust 异步最常见、也最严重的性能事故**：一个工作线程被阻塞，它上面排队的**成百上千个任务全部卡死**。

```rust
// ✘ 全都是错误示范
std::thread::sleep(Duration::from_secs(1));         // 阻塞线程
std::fs::read_to_string("f.txt")?;                  // 同步文件 IO
expensive_cpu_computation();                        // 长时间 CPU 计算
reqwest::blocking::get(url)?;                       // 同步 HTTP 客户端
diesel::query(...)                                  // 同步数据库驱动
```

**正确做法**：

```rust
tokio::time::sleep(d).await;                        // ⭐ 异步 sleep
tokio::fs::read_to_string("f.txt").await?;          // ⭐ 异步文件 IO

// ⭐ CPU 密集 / 无法避免的阻塞调用 → 丢到专用阻塞线程池
let result = tokio::task::spawn_blocking(move || {
    expensive_cpu_computation()
}).await?;

// CPU 密集的数据并行，结合 rayon
let result = tokio::task::spawn_blocking(|| data.par_iter().map(f).sum()).await?;
```

> ⭐ **判据：任何可能超过 10~100 微秒还不让出的操作，都应该 `spawn_blocking`。**

---

### 8. 异步 trait 与 Send 约束

**异步 trait**（第十五章）：

```rust
trait Service {
    async fn call(&self) -> Result<()>;             // ⭐ Rust 1.75+ 原生支持
}

#[async_trait::async_trait]                         // ⭐ 需要 dyn Service 时用宏
trait Service { async fn call(&self) -> Result<()>; }
```

**⭐ `Send` 约束的经典报错**：`tokio::spawn` 要求 Future 是 `Send`，如果**跨 `.await` 持有了非 `Send` 的东西**就会失败：

```rust
tokio::spawn(async {
    let rc = Rc::new(1);                // Rc 不是 Send
    do_async().await;                   // ✘ future is not `Send`
    println!("{rc}");                   // ⭐ 因为 rc 跨越了 await 点
});

// ✔ 让非 Send 的值在 await 之前结束生命周期
tokio::spawn(async {
    { let rc = Rc::new(1); println!("{rc}"); }      // ⭐ 作用域内用完
    do_async().await;
});
```

> 报错 `` future cannot be sent between threads safely `` 时，找**跨越 `.await` 的非 `Send` 变量**（`Rc`、`RefCell` 的守卫、`std::sync::MutexGuard`）。

**异步递归需要装箱**：

```rust
fn walk(n: Node) -> BoxFuture<'static, ()> {        // ⭐ Future 大小无法确定
    Box::pin(async move { for c in n.children { walk(c).await; } })
}
```

---

### 9. 高频踩坑点（重点）

**① 忘记 `.await`**

```rust
fetch(url);                             // ⚠ 什么都不会发生
fetch(url).await?;                      // ✔
```

**② 阻塞异步运行时**（见第 7 节，最严重）

**③ 顺序 await 导致串行**（见第 5 节）

**④ 持锁跨 `.await`**

```rust
let guard = mutex.lock().unwrap();
async_op().await;                       // ✘ 阻塞工作线程 + 守卫非 Send
```

**⑤ 任务泄漏与失控 spawn**

```rust
for req in requests { tokio::spawn(handle(req)); }  // ⚠ 无上限，可能 OOM
// ✔ 用 Semaphore 限并发，或 buffer_unordered 限流
```

⭐ `tokio::spawn` 返回的 `JoinHandle` **被 drop 后任务仍继续运行**（与 `std::thread` 的 `JoinHandle` 语义不同），需要优雅关闭时应保存 handle 或用 `JoinSet` / `CancellationToken`。

**⑥ 在异步上下文里 `block_on`**

```rust
async fn f() { runtime.block_on(g()); } // ✘ panic: Cannot start a runtime from within a runtime
```

**⑦ `select!` 的取消安全性**

```rust
tokio::select! {
    v = rx.recv() => { }                // ⭐ recv 是 cancel-safe 的
    _ = other => { }                    // ⚠ 但有些操作被取消会丢数据
}
```

**⑧ 大量小任务的调度开销**

任务不是免费的，几十万个空转任务同样会拖垮调度器。批处理比逐条 spawn 更高效。

**⑨ 异步析构不存在**

Rust 没有 `async Drop`，清理逻辑里不能 `.await`。需要异步清理时得显式调用 `shutdown().await`。

---

### 10. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ IO 密集用异步，CPU 密集用 `spawn_blocking`/`rayon` | 不要混淆两者 |
| ⭐ 绝不在异步中调用阻塞 API | `std::thread::sleep`、同步 IO、重计算 |
| 多个独立异步操作用 `join!` | 不要顺序 `.await` |
| 临界区不跨 `.await` 就用 `std::sync::Mutex` | 更快；跨 `await` 才用 `tokio::sync::Mutex` |
| 并发量必须有上限 | `Semaphore` 或 `buffer_unordered` |
| 通道用有界的 | 自带背压，防止内存爆掉 |
| 所有外部调用加 `timeout` | 没有超时的网络请求迟早拖垮服务 |
| 实现优雅关闭 | `watch` / `CancellationToken` 广播关闭信号 |
| 全链路加 `tracing` + `#[instrument]` | 异步调用栈难以调试，日志是主要手段 |
| 只在 `main` 附近使用运行时 | 库不要自带运行时，交给调用方 |
| 依赖统一在 Tokio 生态内选 | 混用不同运行时会出问题 |
| 压测验证并发行为 | 异步 bug 只在高负载下显现 |

---

### 11. 面试重点

**Q1：异步和多线程有什么区别？怎么选？**

多线程的并发单位是 OS 线程，每个线程有独立的栈（2~8 MB）和内核态上下文切换成本，几千个就吃不消；异步的并发单位是编译期生成的**状态机（Future）**，只占几十字节到几 KB，切换只是用户态的函数调用，**几十万并发任务也没问题**。选型：**IO 密集（网络、数据库、文件）用异步；CPU 密集用线程或 `rayon`**。代价是异步有"函数染色"、`Send` 约束、取消语义等额外心智负担。

**Q2：`async fn` 编译后变成了什么？**

变成一个**返回 `impl Future` 的普通函数**，函数体被编译器改写成**状态机**（一个枚举，每个 `.await` 点对应一个状态）。运行时通过反复调用 `poll()` 驱动它：返回 `Poll::Pending` 就挂起并注册 `Waker`，等 IO 就绪后被唤醒，再次 `poll` 时从上次的状态继续。整个过程**没有额外线程、没有栈切换**，所以是零成本抽象。

**Q3：为什么 Rust 的 Future 是惰性的？**

`async fn` 调用后只是**构造出一个状态机对象**，不执行任何代码，必须被 `.await`（或交给运行时 `spawn`）才会被 `poll` 驱动。好处是可以自由组合（`join!`、`select!`、`timeout`）、可以在真正执行前取消、也不会有意外的副作用。这与 JavaScript 的 Promise 不同——Promise 一创建就开始执行。

**Q4：Rust 为什么没有内置异步运行时？**

语言层只提供 `async/await` 语法和 `Future` trait 这个**接口**，把执行器、IO 事件循环（epoll/kqueue/IOCP）、定时器留给生态实现。原因是不同场景需求差异极大：服务器要多线程工作窃取调度，嵌入式要单线程无分配，WASM 要对接浏览器事件循环——内置一种会限制其他场景。代价是必须依赖第三方，且**不同运行时之间不通用**。目前事实标准是 **Tokio**。

**Q5：`Pin` 是干什么的？**

`async fn` 生成的状态机可能保存**跨 `.await` 的引用**（例如 `let x = ...; f(&x).await`），这使 Future 成为**自引用结构**。而 Rust 中移动是按位复制，一旦 Future 被移动，内部指向自身字段的指针就会失效。`Pin<&mut T>` 的作用就是在类型层面**保证被轮询的 Future 不会再被移动**。业务开发基本不用手写，只在异步递归（需要 `Box::pin`）或手动实现 `Future` 时才会遇到。

**Q6：在异步代码里调用阻塞函数会怎样？**

⭐ **会阻塞整个工作线程**，导致排在这个线程上的**所有其他任务都无法执行**——这是 Rust 异步最常见的性能事故，表现为 QPS 骤降、延迟飙升，且很难定位。典型元凶：`std::thread::sleep`、同步文件 IO、同步 HTTP/数据库客户端、长时间 CPU 计算。正确做法是用异步版本（`tokio::time::sleep`、`tokio::fs`），无法避免时用 **`tokio::task::spawn_blocking`** 丢到专用的阻塞线程池。判据是：**任何可能超过 10~100 微秒还不让出的操作**。

**Q7：`std::sync::Mutex` 和 `tokio::sync::Mutex` 怎么选？**

关键看**临界区是否跨越 `.await`**：不跨越就用**标准库的**（更快，没有异步调度开销），并且要把守卫限制在一个小作用域里；跨越 `.await` 才用 `tokio::sync::Mutex`——它在等待时会让出线程而不是阻塞，且守卫是 `Send` 的。标准库的 `MutexGuard` 不是 `Send`，跨 `.await` 持有会导致整个 Future 不满足 `tokio::spawn` 的 `Send` 约束而编译失败（这其实是编译器在帮你）。

**Q8：怎么让多个异步操作并发执行？**

顺序 `.await` 是**串行**的（总耗时是各段之和）。要并发：少量固定的用 **`tokio::join!` / `try_join!`**（总耗时取最大值，后者遇错短路）；数量不定的用 **`futures::future::join_all`**；需要限制并发数防止打爆下游时用 **`buffer_unordered(n)`** 或 **`Semaphore`**；需要"谁先完成用谁"（超时、取消）用 **`select!`**，但要注意未完成分支会被 drop，需确认**取消安全性**。

---

## 二十四、Unsafe Rust 与 FFI

> ⭐ 业务开发几乎不需要写 `unsafe`。但面试常问它的**边界**——什么是 UB、`unsafe` 到底做了什么、怎么安全封装。

### 1. unsafe 是什么，不是什么

⭐ **最大的误解：`unsafe` 不会关闭借用检查器。**

```rust
unsafe {
    let mut s = String::from("hi");
    let r1 = &mut s;
    let r2 = &mut s;                    // ✘ 照样编译错误！借用检查依然生效
}
```

⭐ **`unsafe` 只做一件事：解锁 5 种编译器无法验证安全性的能力**，并把"保证不出错"的责任从编译器转移给程序员。

| 能力 | 说明 |
| ---- | ---- |
| ⭐ ① 解引用裸指针 | `*const T` / `*mut T` |
| ⭐ ② 调用 `unsafe` 函数 | ⭐ 包括所有 FFI 外部函数 |
| ③ 访问或修改可变静态变量 | `static mut`（⚠ 已强烈不推荐） |
| ④ 实现 `unsafe` trait | ⭐ 手动 `unsafe impl Send/Sync` |
| ⑤ 访问 `union` 的字段 | 主要用于 FFI |

> ⭐ **`unsafe` 的正确心态**：它是一份**契约声明** —— "我已经人工验证过这段代码满足所有安全前提"。写 `unsafe` 不是绕过编译器，而是承担编译器的职责。

---

### 2. 裸指针

```rust
let mut n = 5;
let p1 = &n as *const i32;              // ⭐ 创建裸指针是安全的
let p2 = &mut n as *mut i32;

unsafe { println!("{}", *p1); }         // ⭐ 解引用才需要 unsafe
```

| 特性 | 引用 `&T` | ⭐ 裸指针 `*const T` |
| ---- | --------- | ------------------- |
| 可以为空 | ✘ | ⭐ ✔ |
| 可以悬垂 | ✘ | ⭐ ✔ |
| 有生命周期 | ✔ | ⭐ ✘ |
| 保证对齐/有效 | ✔ | ⭐ ✘ |
| 可同时存在多个可变指针 | ✘ | ⭐ ✔（但同时用就是 UB） |

---

### 3. 未定义行为（UB）清单

⭐ **UB 意味着编译器可以做任何事** —— 程序可能崩溃、可能给出错误结果、也可能"看起来正常"直到某次升级编译器后爆炸。面试常问"哪些操作是 UB"：

| UB | 说明 |
| -- | ---- |
| ⭐ 解引用空指针 / 悬垂指针 / 未对齐指针 | — |
| ⭐ 数据竞争 | 多线程无同步地并发读写 |
| ⭐ 违反别名规则 | 存在 `&mut T` 的同时通过其他指针访问同一数据 |
| ⭐ 读未初始化内存 | 要用 `MaybeUninit<T>` |
| 越界访问 | 手动指针算术越界 |
| ⭐ 构造非法值 | `bool` 不是 0/1、`char` 超出范围、`enum` 判别值非法、`str` 不是合法 UTF-8 |
| ⭐ panic 跨越 FFI 边界 | 见第 6 节 |
| 破坏 `Pin` 的约定 | 移动了已固定的对象 |

```rust
let s: &str = unsafe { std::str::from_utf8_unchecked(&[0xFF]) };    // ⭐ UB！
let b: bool = unsafe { std::mem::transmute(2u8) };                  // ⭐ UB！
```

---

### 4. 安全抽象：unsafe 的正确用法

⭐ **`unsafe` 应该被封装在安全的 API 内部**，对外暴露的接口保证任何调用方式都不会出问题 —— 标准库就是这么做的（`Vec`、`RefCell`、`split_at_mut` 内部全是 `unsafe`）。

```rust
use std::slice;

pub fn split_at_mut(s: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = s.len();
    let ptr = s.as_mut_ptr();
    assert!(mid <= len);                            // ⭐ 先做运行时校验

    unsafe {
        // SAFETY: mid <= len 已断言，两个切片的范围不重叠，
        //         因此不存在两个可变引用指向同一元素。
        (slice::from_raw_parts_mut(ptr, mid),
         slice::from_raw_parts_mut(ptr.add(mid), len - mid))
    }
}
```

**⭐ 三条铁律**：

1. **`unsafe` 块尽可能小** —— 只包住真正需要的那几行
2. **每个 `unsafe` 块必须写 `// SAFETY:` 注释**，说明为什么它是安全的
3. **对外 API 必须是安全的**：任何输入都不能导致 UB（做不到就把函数本身标成 `unsafe fn` 并写 `# Safety` 文档）

---

### 5. FFI：调用 C 代码

```rust
use std::os::raw::c_int;

unsafe extern "C" {                     // ⭐ Rust 2024 起 extern 块要标 unsafe
    fn abs(input: c_int) -> c_int;
}

fn main() {
    unsafe { println!("{}", abs(-3)); } // ⭐ 所有外部函数调用都是 unsafe
}
```

**⭐ 字符串与结构体的跨语言传递**：

```rust
use std::ffi::{CString, CStr, c_char};

// Rust → C：必须转成以 \0 结尾的 CString
let s = CString::new("hello").unwrap();
unsafe { c_func(s.as_ptr()); }          // ⭐ 注意 s 必须活到调用结束

// C → Rust
let rust_str = unsafe { CStr::from_ptr(ptr).to_str()? };

// ⭐ 结构体必须标 repr(C)，否则字段顺序不保证（第十一章）
#[repr(C)]
pub struct Point { x: i32, y: i32 }
```

**链接原生库**：

```rust
#[link(name = "mylib")]
unsafe extern "C" { fn my_func() -> c_int; }
```

> ⭐ 实际项目用 **`bindgen`** 从 C 头文件自动生成 Rust 绑定，不要手写；常用 C 库多半已有现成的 `-sys` crate（如 `openssl-sys`）。

---

### 6. 导出给 C 调用

```rust
#[unsafe(no_mangle)]                    // ⭐ Rust 2024 写法；旧版是 #[no_mangle]
pub extern "C" fn rust_add(a: i32, b: i32) -> i32 {
    a + b
}
```

**⭐ 两个必须处理的问题**：

```rust
// ① panic 不能跨越 FFI 边界（UB）
#[unsafe(no_mangle)]
pub extern "C" fn rust_api() -> i32 {
    std::panic::catch_unwind(|| {       // ⭐ 必须捕获
        risky_operation()
    }).unwrap_or(-1)
}

// ② ⭐ 内存所有权：谁分配谁释放
#[unsafe(no_mangle)]
pub extern "C" fn make_string() -> *mut c_char {
    CString::new("hi").unwrap().into_raw()          // ⭐ 交出所有权
}

#[unsafe(no_mangle)]
pub extern "C" fn free_string(p: *mut c_char) {
    if p.is_null() { return; }
    unsafe { drop(CString::from_raw(p)); }          // ⭐ 必须由 Rust 侧释放
}
```

> ⭐ **绝不能让 C 用 `free()` 释放 Rust 分配的内存**（分配器不同），反之亦然。生成 C 头文件用 **`cbindgen`**。

---

### 7. 验证工具

| 工具 | 作用 |
| ---- | ---- |
| ⭐**Miri** | ⭐ 解释执行并检测 UB（越界、未初始化、别名违规），`unsafe` 代码必跑 |
| `cargo careful` | 加强版 debug 运行 |
| AddressSanitizer / Valgrind | 检测内存错误 |
| `clippy::undocumented_unsafe_blocks` | 强制要求 `SAFETY` 注释 |

```bash
cargo +nightly miri test                # ⭐ 用 Miri 跑测试
```

---

### 8. 高频踩坑点

**① 以为 `unsafe` 能绕过借用检查** —— 不能，见第 1 节。

**② `CString` 临时值被提前释放（经典 bug）**

```rust
let ptr = CString::new("hi").unwrap().as_ptr();     // ✘ 临时 CString 当场 drop
unsafe { c_func(ptr) };                             //    ptr 已悬垂 → UB

let s = CString::new("hi").unwrap();                // ✔ 绑定到变量
unsafe { c_func(s.as_ptr()) };
```

**③ 忘记 `#[repr(C)]`**

Rust 默认布局会重排字段（第十一章），与 C 结构体不匹配，读出来全是乱码。

**④ 乱标 `unsafe impl Send/Sync`**

```rust
unsafe impl Send for MyType {}          // ⚠ 必须真正保证线程安全，否则是数据竞争 UB
```

**⑤ `transmute` 滥用**

```rust
let x: f32 = unsafe { std::mem::transmute(1u32) };  // ⚠ 极易 UB
let x = f32::from_bits(1u32);                       // ✔ 用安全 API
```

> `transmute` 是最危险的函数之一，几乎总有安全替代（`as`、`from_bits`、`bytemuck` crate）。

**⑥ 未初始化内存**

```rust
let x: i32 = unsafe { std::mem::zeroed() };         // ⚠ 对含引用/枚举的类型是 UB
let mut buf = MaybeUninit::<[u8; 1024]>::uninit();  // ✔ 正确做法
```

**⑦ `static mut`**（第一章）：Rust 2024 起对它取引用是硬错误，改用 `OnceLock`/原子类型/`Mutex`。

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 业务代码不写 `unsafe` | 需要时先确认没有安全替代方案 |
| `unsafe` 块最小化 | 只包必要的几行 |
| ⭐ 每个 `unsafe` 必须有 `// SAFETY:` 注释 | 说明满足了哪些前提 |
| 对外 API 必须安全 | 做不到就标 `unsafe fn` 并写 `# Safety` 文档 |
| `unsafe` 代码必须跑 Miri | 加进 CI |
| FFI 结构体必须 `#[repr(C)]` | — |
| FFI 边界必须 `catch_unwind` | panic 跨边界是 UB |
| 跨语言内存"谁分配谁释放" | 提供配套的 `free_xxx` 函数 |
| C 绑定用 `bindgen` 生成 | 不手写；优先找现成 `-sys` crate |
| `unsafe` 代码集中在少数模块 | 便于审计，用 `#![forbid(unsafe_code)]` 保护其余部分 |
| 提高 `unsafe` 代码的测试覆盖率 | 边界条件必须覆盖 |

---

### 10. 面试重点

**Q1：`unsafe` 到底做了什么？会关闭借用检查吗？**

⭐ **不会**。借用检查、类型检查在 `unsafe` 块里照常生效。`unsafe` 只解锁 5 种编译器无法验证安全性的能力：① 解引用裸指针；② 调用 `unsafe` 函数（含所有 FFI）；③ 访问/修改 `static mut`；④ 实现 `unsafe` trait（如手动 `impl Send`/`Sync`）；⑤ 访问 `union` 字段。它的本质是**契约声明**——程序员向编译器保证"我已人工验证这里满足安全前提"，责任从编译器转移到人。

**Q2：Rust 里哪些行为是未定义行为（UB）？**

解引用空指针/悬垂指针/未对齐指针；数据竞争；违反别名规则（存在 `&mut T` 时通过其他路径访问同一数据）；读取未初始化内存；越界访问；**构造非法值**（`bool` 不是 0/1、`char` 超范围、非法的 `enum` 判别值、非 UTF-8 的 `str`）；panic 跨越 FFI 边界；破坏 `Pin` 约定。UB 的可怕之处在于编译器可以做任何事，程序可能"看起来正常"直到某次编译器升级后突然崩溃。

**Q3：`unsafe` 的正确使用方式是什么？**

**把 `unsafe` 封装在安全的抽象内部**——对外暴露的 API 必须保证任何调用方式都不会导致 UB。标准库就是范例：`Vec`、`RefCell`、`split_at_mut` 内部都是 `unsafe`，但对外完全安全。三条实践铁律：① `unsafe` 块尽可能小；② 每个块写 `// SAFETY:` 注释说明前提；③ 无法保证输入安全时，把函数本身标成 `unsafe fn` 并在文档写 `# Safety` 段。此外 `unsafe` 代码必须用 **Miri** 验证并加进 CI。

**Q4：裸指针和引用有什么区别？**

裸指针（`*const T`/`*mut T`）**可以为空、可以悬垂、没有生命周期、不保证对齐和有效性，也不受借用规则约束**（可以同时存在多个可变裸指针）；引用则由编译器保证始终有效、对齐、且遵守借用规则。注意：**创建裸指针是安全的，解引用才需要 `unsafe`**。

**Q5：FFI 调用 C 需要注意什么？**

① 结构体必须加 **`#[repr(C)]`**，否则 Rust 默认布局会重排字段；② 字符串要转成 `CString`（以 `\0` 结尾），且**必须保证 `CString` 活到调用结束**——`CString::new(..).as_ptr()` 这种写法会让临时值当场析构导致悬垂；③ **panic 不能跨越 FFI 边界**（UB），导出给 C 的函数要用 `catch_unwind` 包住；④ **内存所有权遵循"谁分配谁释放"**，Rust 分配的内存必须由 Rust 侧提供 `free_xxx` 函数释放；⑤ 绑定用 `bindgen` 生成，反向导出头文件用 `cbindgen`。

**Q6：什么时候真的需要 `unsafe`？**

四类场景：① **FFI**——调用 C/C++ 库或被其他语言调用；② **实现底层数据结构**——链表、自定义分配器、无锁队列等借用检查无法表达的结构；③ **性能极致优化**——已确认边界检查是瓶颈时用 `get_unchecked` 消除（必须先有 benchmark 证据）；④ **与硬件/操作系统交互**——嵌入式寄存器操作、系统调用。业务开发中这几类都很少见，绝大多数项目可以直接 `#![forbid(unsafe_code)]`。

---

## 二十五、内存布局与性能优化

> ⭐ 面试问性能时，**能说清"为什么快"比背优化技巧更重要**。核心是三点：栈优于堆、避免分配、数据布局友好。

### 1. 常见类型的大小

```rust
std::mem::size_of::<T>();               // 类型占用字节数
std::mem::align_of::<T>();              // 对齐要求
```

| 类型 | 大小（64 位） | 说明 |
| ---- | ------------- | ---- |
| `i32` / `u32` / `f32` | 4 | — |
| `i64` / `f64` / `usize` | 8 | — |
| `char` | ⭐ 4 | Unicode 标量值 |
| `bool` | 1 | — |
| ⭐`&T` / `Box<T>` / `Rc<T>` | ⭐ 8 | 瘦指针 |
| ⭐`&str` / `&[T]` | ⭐ **16** | ⭐ 胖指针（ptr + len） |
| ⭐`&dyn Trait` / `Box<dyn Trait>` | ⭐ **16** | ⭐ 胖指针（ptr + vtable） |
| ⭐`String` / `Vec<T>` | ⭐ **24** | ptr + len + cap |
| `Box<[T]>` | 16 | ptr + len（无 cap） |
| ⭐`Option<Box<T>>` / `Option<&T>` | ⭐ **8** | ⭐ 空位优化，不额外占空间 |
| `Option<i32>` | 8 | 4 数据 + 4 tag/padding |
| `()` / 单元结构体 | ⭐ 0 | 零大小类型（ZST） |

**⭐ 结构体会自动重排字段以减少填充**（第十一章）：

```rust
struct A { a: u8, b: u32, c: u8 }
size_of::<A>();                         // ⭐ 8（Rust 自动重排）

#[repr(C)]
struct B { a: u8, b: u32, c: u8 }
size_of::<B>();                         // ⭐ 12（保持顺序，产生 6 字节填充）
```

**⭐ 枚举大小 = tag + 最大变体**，所以大变体要装箱（第十二章）：

```rust
enum E1 { A, B([u8; 1024]) }            // ⭐ 1032 字节，即使是 A 也占这么多
enum E2 { A, B(Box<[u8; 1024]>) }       // ⭐ 16 字节
```

---

### 2. 栈与堆的性能差距

| 操作 | 栈 | 堆 |
| ---- | -- | -- |
| 分配 | ⭐ 移动栈指针，几乎免费 | ⭐ 调用分配器，可能加锁、查找空闲块 |
| 释放 | 函数返回自动完成 | 调用 `free`，可能触发合并 |
| 访问 | ⭐ 缓存友好（连续） | 指针跳转，可能 cache miss |

⭐ **堆分配的成本大约是栈的几十到上百倍**。所以 Rust 性能优化的第一原则是：**减少堆分配次数**。

**常见的隐藏分配**：

```rust
s.to_string();  format!(...);  vec![...];  x.clone();
Box::new(x);  collect::<Vec<_>>();  String::from(...);
```

---

### 3. 减少分配的常用手段（重点）

**① 预分配容量**

```rust
let mut v = Vec::new();
for i in 0..10_000 { v.push(i); }               // ⚠ 约 14 次重分配 + 拷贝
let mut v = Vec::with_capacity(10_000);         // ⭐ ✔ 一次到位
let mut s = String::with_capacity(1024);
```

**② 复用缓冲区**

```rust
// ✘ 每次循环都分配
for line in lines { let mut buf = String::new(); process(line, &mut buf); }

// ⭐ ✔ 循环外分配一次，内部 clear 复用
let mut buf = String::with_capacity(256);
for line in lines {
    buf.clear();                                // ⭐ 保留容量，只重置长度
    process(line, &mut buf);
}
```

**③ 用借用代替所有权**

```rust
fn f(s: String) { }                     // ⚠ 调用方被迫 clone / to_string
fn f(s: &str) { }                       // ⭐ ✔ 零拷贝
fn f(v: &[T]) { }                       // ⭐ ✔ 而非 Vec<T>
```

**④ 避免中间集合**

```rust
let a: Vec<_> = v.iter().map(f).collect();      // ⚠ 一次分配
let b: Vec<_> = a.iter().filter(g).collect();   // ⚠ 又一次

let b: Vec<_> = v.iter().map(f).filter(g).collect();     // ⭐ ✔ 一次遍历一次分配
```

**⑤ 小对象用栈上容器**

```rust
// smallvec crate：元素少时放栈上，超过阈值才转堆
let mut v: SmallVec<[u32; 8]> = SmallVec::new();
```

**⑥ 用 `Cow` 避免"可能不需要的"分配**（第六章）

```rust
fn normalize(s: &str) -> Cow<'_, str> {
    if s.contains(' ') { Cow::Owned(s.replace(' ', "_")) } else { Cow::Borrowed(s) }
}
```

**⑦ 共享只读数据用 `Arc`**

```rust
let cfg = config.clone();               // ⚠ 深拷贝整个配置
let cfg = Arc::clone(&config);          // ⭐ ✔ 只增加计数
```

---

### 4. 数据布局与缓存

⭐ **现代 CPU 的瓶颈往往是内存访问而不是计算**。一次 L1 缓存命中约 1ns，主存访问约 100ns，差 100 倍。

**⭐ AoS vs SoA**：只访问部分字段时，结构体数组（AoS）会把用不到的字段也拉进缓存：

```rust
// AoS（Array of Structs）：常规写法
struct Particle { x: f32, y: f32, z: f32, name: String }
let ps: Vec<Particle>;
for p in &ps { p.x += 1.0; }            // ⚠ 每次都把 name 等无关数据拉进缓存

// ⭐ SoA（Struct of Arrays）：只在热点循环需要时才这么做
struct Particles { xs: Vec<f32>, ys: Vec<f32>, names: Vec<String> }
for x in &mut ps.xs { *x += 1.0; }      // ⭐ 连续内存，可向量化（SIMD）
```

**其他布局要点**：

- ⭐ `Vec<Vec<T>>` 内存不连续 → 性能敏感处用一维 `Vec` 模拟二维（第七章）
- ⭐ `LinkedList` 节点分散 → 几乎总是比 `Vec` 慢
- ⭐ **假共享（false sharing）**：不同线程频繁写入同一缓存行（64 字节）内的相邻数据，会导致缓存行反复失效。解法是按缓存行对齐 `#[repr(align(64))]`

---

### 5. 编译优化

```toml
[profile.release]
opt-level = 3               # ⭐ 默认；"s"/"z" 为体积优先
lto = "fat"                 # ⭐ 链接时优化，跨 crate 内联，通常 5~20% 提升
codegen-units = 1           # ⭐ 牺牲编译并行度换优化质量
panic = "abort"             # ⭐ 去掉栈展开代码，体积更小、稍快
strip = true                # 去符号表
```

```bash
RUSTFLAGS="-C target-cpu=native" cargo build --release   # ⭐ 启用本机 CPU 指令集（AVX 等）
```

**更换分配器**（多线程高分配场景收益明显）：

```rust
#[global_allocator]
static ALLOC: mimalloc::MiMalloc = mimalloc::MiMalloc;      // 或 jemallocator
```

**内联控制**：

```rust
#[inline]                   // 建议内联（跨 crate 内联必须标）
#[inline(always)]           // ⚠ 强制，滥用会导致代码膨胀
#[inline(never)]            // 便于 profiling
```

---

### 6. 性能分析方法

⭐ **铁律：先测量，再优化。** 凭直觉优化几乎总是白费力气。

| 工具 | 用途 |
| ---- | ---- |
| ⭐`criterion` | ⭐ 微基准测试（统计学意义上的对比） |
| ⭐`cargo flamegraph` | ⭐ 火焰图，定位热点函数 |
| `perf` / `Instruments` / VTune | 系统级 profiling |
| `hyperfine` | 命令行整体耗时对比 |
| `dhat` / `heaptrack` | ⭐ 堆分配分析（找出分配次数最多的路径） |
| `cargo bloat` | 分析二进制体积占用 |
| `cargo asm` | 查看生成的汇编 |

**标准流程**：

```text
1. 建立基准（criterion / 压测）      ← 没有基准就没有优化
2. 火焰图定位热点                     ← 通常 90% 时间在 10% 代码
3. 只优化热点                         ← 冷代码优化毫无意义
4. 重新测量验证                       ← 确认真的变快了
```

> ⚠ **必须用 `--release` 测量**（第二十章）：debug 模式性能可能差 10~100 倍且相对开销失真。

---

### 7. 零成本抽象的边界

⭐ **哪些是真正零成本，哪些不是** —— 这是面试的深度问题：

| 抽象 | 是否零成本 |
| ---- | ---------- |
| ⭐ 泛型 / `impl Trait` | ⭐ ✔ 单态化，编译期展开 |
| ⭐ 迭代器链 | ⭐ ✔ 内联融合成单循环，还省掉边界检查 |
| ⭐`Option<&T>` / `Option<Box<T>>` | ⭐ ✔ 空位优化，与裸指针同大小 |
| `Result` / `?` | ⭐ ✔ 编译成普通的 `match` 和分支 |
| 所有权 / 借用检查 | ⭐ ✔ 纯编译期，无运行时代码 |
| `newtype` / `#[repr(transparent)]` | ✔ 布局完全相同 |
| ⭐`dyn Trait` 动态分发 | ⭐ ✘ 虚表跳转 + **无法内联** |
| ⭐`Rc` / `Arc` | ✘ 引用计数（`Arc` 还是原子操作） |
| ⭐`RefCell` | ✘ 运行时借用计数 |
| ⭐`Mutex` / 通道 | ✘ 同步开销 |
| ⭐ 索引访问 `v[i]` | ✘ 边界检查（迭代器可消除） |
| async 状态机 | 大体零成本，但**大 Future 会占用大量栈/堆空间** |

---

### 8. 高频踩坑点

**① 用 debug 模式下结论** —— 最常见的错误，见第二十章。

**② 热路径上的 `format!` / `to_string()`**

```rust
for item in items { log(&format!("处理 {}", item.id)); }    // ⚠ 每次都分配
for item in items { info!(id = item.id, "处理"); }          // ⭐ ✔ 结构化日志，级别未开启则零开销
```

**③ 无谓的 `clone()`**

```rust
process(data.clone());                  // ⚠ 先问：真的需要所有权吗？
process(&data);                         // ⭐ ✔
```

**④ 大结构体按值传递**

```rust
fn f(cfg: Config) { }                   // ⚠ 整体拷贝到栈
fn f(cfg: &Config) { }                  // ⭐ ✔ 只传 8 字节
```

**⑤ `HashMap` 默认 hasher 偏慢**

内部可信数据换 `ahash`/`FxHashMap`（第十七章），对外接口保留默认以防 HashDoS。

**⑥ 热路径上的动态分发**

```rust
Vec<Box<dyn Handler>>                   // ⚠ 每次调用一次虚表跳转，无法内联
fn process(h: impl Handler)             // ⭐ ✔ 编译期确定就用泛型
```

**⑦ 过度优化冷代码**

启动逻辑、配置解析、错误处理路径优化得再好也没有收益，反而牺牲可读性。

**⑧ 大 Future / 大枚举变体**

异步函数里的大局部变量会撑大整个状态机，`tokio::spawn` 时要堆分配。大变量应该 `Box` 起来。

---

### 9. 企业开发规范

| 规范 | 说明 |
| ---- | ---- |
| ⭐ 先测量再优化 | 没有火焰图/基准就不要动手 |
| 只优化热点 | 90% 时间在 10% 代码 |
| 性能测试一律 `--release` | debug 数据无意义 |
| 优先减少堆分配 | 预分配、复用缓冲、借用代替所有权 |
| 参数用 `&str` / `&[T]` | 让调用方免于分配 |
| 已知规模先 `with_capacity` | `Vec`/`String`/`HashMap` 都适用 |
| 共享只读数据用 `Arc` | 而非 `clone` 整份 |
| 避免中间 `collect()` | 迭代器链一次完成 |
| release 开 `lto` + `codegen-units = 1` | 通常有 5~20% 提升 |
| 多线程高分配场景换 `mimalloc`/`jemalloc` | — |
| 建立性能回归测试 | CI 对比基准，防止劣化 |
| 可读性优先，除非有数据支撑 | 不为想象中的性能牺牲代码质量 |

---

### 10. 面试重点

**Q1：Rust 为什么快？**

四点：① **无 GC**——内存管理在编译期完成，没有垃圾回收停顿和后台开销；② **零成本抽象**——泛型单态化、迭代器内联融合，高层抽象编译后与手写底层代码等价；③ **默认栈分配、精确控制内存布局**——不像 Java 那样对象默认在堆上且带对象头；④ **LLVM 优化 + 更强的别名信息**——借用规则保证 `&mut T` 是独占的，编译器能做比 C 更激进的优化。

**Q2：`&str`、`String`、`&dyn Trait` 各占多少字节？**

`&str` 是**胖指针**，16 字节（数据指针 + 长度）；`String` 24 字节（指针 + 长度 + 容量）；`&dyn Trait` 和 `Box<dyn Trait>` 也是**胖指针**，16 字节（数据指针 + 虚表指针）；普通 `&T`、`Box<T>` 是瘦指针 8 字节。特别地，`Option<Box<T>>`、`Option<&T>` 因为**空位优化**仍然只有 8 字节。

**Q3：怎么优化 Rust 程序的性能？**

**流程**：先用 `criterion` 建基准、用 `cargo flamegraph` 定位热点，只优化那 10% 的热点代码，改完重新测量验证。**常见手段**：减少堆分配（`with_capacity` 预分配、循环外复用缓冲区、参数用 `&str`/`&[T]`、避免中间 `collect`、用 `Cow`/`Arc`）；优化数据布局（避免 `Vec<Vec<T>>`、热点循环考虑 SoA）；用迭代器代替下标（消除边界检查）；热路径避免动态分发；编译层面开 `lto`、`codegen-units = 1`、`target-cpu=native`，必要时换 `mimalloc` 分配器。

**Q4：哪些 Rust 抽象是零成本的，哪些不是？**

**零成本**：泛型/`impl Trait`（单态化）、迭代器链（内联融合且消除边界检查）、`Option<&T>`/`Option<Box<T>>`（空位优化）、`Result` 和 `?`（编译成普通分支）、所有权和借用检查（纯编译期）。**有成本**：`dyn Trait` 动态分发（虚表跳转 + **无法内联**，这是主要代价）、`Rc`/`Arc`（引用计数，`Arc` 还是原子操作）、`RefCell`（运行时借用计数）、`Mutex`/通道（同步开销）、`v[i]` 索引（边界检查）。

**Q5：结构体的内存布局是怎样的？怎么优化？**

默认 `repr(Rust)` 下编译器会**自动重排字段**以最小化对齐填充，所以 `{u8, u32, u8}` 只占 8 字节而非 12（加 `#[repr(C)]` 保持顺序则是 12）。优化要点：① 枚举大小由**最大变体**决定，大变体要用 `Box` 装箱；② `Option<Box<T>>` 等有**空位优化**，不额外占空间；③ FFI 必须 `#[repr(C)]`；④ 多线程频繁写入的数据注意**假共享**，必要时用 `#[repr(align(64))]` 按缓存行对齐。

**Q6：为什么说"用迭代器比用下标快"？**

用 `v[i]` 时编译器必须插入**边界检查**（越界要 panic）；用迭代器时，迭代器的实现保证了不会越界，编译器能证明这一点并把检查**完全消除**。同时多级适配器会被内联并**融合成单次循环**，不产生中间集合。所以迭代器写法通常与手写 C 循环性能相当，甚至更快。

**Q7：Rust 和 Java 的性能差异主要来自哪里？**

① **GC**——Java 有垃圾回收的 CPU 开销和 STW 停顿，尾延迟不可预测；Rust 完全没有。② **内存布局**——Java 对象默认在堆上、带对象头、`List<Integer>` 要装箱；Rust 的 `Vec<i32>` 是紧凑的连续内存，缓存友好。③ **泛型**——Java 类型擦除 + 装箱，Rust 单态化零开销。④ **抽象成本**——Java 的接口调用默认虚分发（虽有 JIT 去虚化），Rust 默认静态分发。反过来 Java 的优势是 JIT 能做运行时 profile 引导优化、开发效率高、生态成熟。

---

## 二十六、常见设计模式与工程实践

> ⭐ Rust 里很多 GoF 模式**要么被语言特性内置了，要么需要换一种表达方式**。这一章也是全文的收尾与实战串联。

### 1. 经典模式在 Rust 中的对应

| GoF 模式 | ⭐ Rust 中的做法 |
| -------- | --------------- |
| 建造者（Builder） | ⭐ 原样保留，弥补没有默认参数/重载（第十一章） |
| 策略（Strategy） | ⭐ trait + 泛型（静态）或 `Box<dyn Trait>`（动态）；简单场景直接传闭包 |
| 工厂（Factory） | ⭐ 关联函数 `new()` / `from()`；多态工厂返回 `Box<dyn Trait>` |
| 单例（Singleton） | ⭐`OnceLock` / `LazyLock`（**绝不用 `static mut`**） |
| 观察者（Observer） | ⭐ 通道（`mpsc`/`broadcast`）或 `Vec<Box<dyn Fn>>` |
| 迭代器（Iterator） | ⭐ 语言内置 `Iterator` trait |
| 装饰器（Decorator） | ⭐ 泛型包装 + `Deref`，或中间件（tower 的 `Layer`） |
| 适配器（Adapter） | ⭐`From`/`Into` + newtype |
| 状态（State） | ⭐ 枚举 + `match`，或 **typestate**（见下） |
| 命令（Command） | ⭐ 枚举消息 + channel |
| 模板方法 | ⭐ trait 默认方法 |
| ⭐ RAII / 资源管理 | ⭐ `Drop` trait 内置，不需要 `try-with-resources` |

**⭐ 三个 Rust 特有的模式**：

**① Newtype**——零成本的类型安全（第二章、第十五章）：

```rust
struct UserId(u64);
struct OrderId(u64);
fn find(id: UserId) { }
find(OrderId(1));                       // ⭐ ✘ 编译期就拦住了参数传反
```

**② Typestate**——⭐ **用类型表达状态机，非法状态迁移根本编译不过**：

```rust
struct Open;  struct Closed;
struct Door<S> { _state: PhantomData<S> }

impl Door<Closed> {
    fn open(self) -> Door<Open> { Door { _state: PhantomData } }
}
impl Door<Open> {
    fn close(self) -> Door<Closed> { Door { _state: PhantomData } }
}

let d = Door::<Closed>::new().open();
d.open();                               // ⭐ ✘ 编译错误：Door<Open> 没有 open 方法
```

> 实际应用：HTTP 请求构建器（未设置 URL 就不能 `send()`）、数据库事务（已提交的事务不能再写）、连接状态机。**把运行时错误变成编译错误**是 Rust 最有价值的能力。

**③ Extension trait**——给外部类型加方法（绕过孤儿规则）：

```rust
trait VecExt<T> { fn second(&self) -> Option<&T>; }
impl<T> VecExt<T> for Vec<T> {
    fn second(&self) -> Option<&T> { self.get(1) }
}
vec![1, 2, 3].second();                 // ⭐ 给标准库类型扩展能力
```

---

### 2. API 设计惯例（命名规范）

⭐ **遵循标准库惯例，调用方一眼就知道语义**（Rust API Guidelines）：

| 前缀/后缀 | 语义 | 示例 |
| --------- | ---- | ---- |
| ⭐`as_xxx` | ⭐ 借用转换，**零开销** | `as_str()`、`as_ref()` |
| ⭐`to_xxx` | ⭐ 克隆转换，**有分配开销** | `to_string()`、`to_owned()` |
| ⭐`into_xxx` | ⭐ **消耗所有权**的转换 | `into_bytes()`、`into_iter()` |
| ⭐`try_xxx` | 可能失败，返回 `Result` | `try_into()`、`try_lock()` |
| ⭐`xxx_mut` | 返回可变引用 | `get_mut()`、`iter_mut()` |
| ⭐`is_xxx` / `has_xxx` | 返回 `bool` | `is_empty()`、`is_some()` |
| `new` / `with_xxx` | 构造 | `new()`、`with_capacity()` |
| `iter` / `iter_mut` / `into_iter` | 三种迭代 | — |
| `_unchecked` | ⭐ 跳过检查的 `unsafe` 版本 | `get_unchecked()` |

**其他 API 准则**：

```rust
fn f(s: &str) { }                       // ⭐ 参数用 &str / &[T]，不用 &String / &Vec<T>
fn f(s: impl Into<String>) { }          // ⭐ 需要所有权时提升易用性
fn f(p: impl AsRef<Path>) { }           // ⭐ 路径参数的标准写法

fn find(id: u64) -> Option<User>;       // ⭐ "可能没有"用 Option
fn load(id: u64) -> Result<User, E>;    // ⭐ "可能失败"用 Result
fn all() -> Vec<User>;                  // ⭐ 空集合而非 Option<Vec>

#[derive(Debug, Clone)]                 // ⭐ 公开类型至少 Debug
#[non_exhaustive]                       // ⭐ 公开枚举便于将来扩展
pub enum Error { }
```

---

### 3. 项目分层与依赖注入

**⭐ 典型的后端服务分层**（workspace，第十九章）：

```text
crates/
├── domain/     纯业务模型与 trait 定义，不依赖任何框架
├── infra/      数据库、缓存、外部 API 的具体实现
├── service/    业务逻辑编排
└── api/        HTTP 层（binary）
              ⭐ 依赖方向：api → service → domain ← infra
```

**⭐ 依赖注入的两种做法**：

```rust
// ① 泛型注入（静态分发，零开销，编译期确定）—— 推荐
pub struct UserService<R: UserRepo> { repo: R }

impl<R: UserRepo> UserService<R> {
    pub fn new(repo: R) -> Self { Self { repo } }
}

// ② trait 对象注入（运行时可替换，适合插件/配置驱动）
pub struct UserService { repo: Arc<dyn UserRepo + Send + Sync> }
```

**可测试性**：把外部依赖抽象成 trait，测试时注入假实现：

```rust
#[cfg_attr(test, mockall::automock)]        // ⭐ 自动生成 mock
pub trait UserRepo {
    fn find(&self, id: u64) -> Result<Option<User>>;
}

#[test]
fn test_service() {
    let mut repo = MockUserRepo::new();
    repo.expect_find().returning(|_| Ok(Some(User::default())));
    let svc = UserService::new(repo);
    assert!(svc.get(1).is_ok());
}
```

---

### 4. 常见反模式（重点）

| ⚠ 反模式 | 问题 | ⭐ 正确做法 |
| -------- | ---- | ---------- |
| ⭐ 到处 `clone()` | 用性能换编译通过 | 先想清所有权，参数改 `&T` |
| ⭐ 到处 `unwrap()` | 生产环境 panic | `?` 传播、`expect("原因")` |
| ⭐`Rc<RefCell<T>>` 满天飞 | 运行时 panic + 丧失编译期保证 | 重新设计所有权；图/树用 arena + 索引 |
| ⭐`Arc<Mutex<T>>` 满天飞 | 锁竞争、死锁风险 | 改用通道 + actor 模式 |
| `Result<T, String>` | 无法分类处理错误 | `thiserror` 定义错误枚举 |
| 结构体存 `&'a T` | 生命周期传染整个调用链 | 业务类型用 `String`/`Arc` |
| 过度泛型 | 签名不可读、编译爆炸 | 泛型参数控制在 2~3 个 |
| 过早抽象 | trait 满天飞却只有一个实现 | 先写具体实现，出现第二个再抽象 |
| 在异步里调阻塞代码 | ⭐ 整个工作线程卡死 | `spawn_blocking` |
| 用 `unsafe` 绕过借用检查 | UB 风险 | 换设计，或用标准库的安全抽象 |
| 模仿 Java 写 Rust | getter/setter 满天飞、继承式设计 | 用枚举 + `match`、组合 + trait |

---

### 5. 一个典型 Web 服务骨架

```rust
// main.rs
#[tokio::main]
async fn main() -> anyhow::Result<()> {
    tracing_subscriber::fmt().with_env_filter("info").json().init();     // ⭐ 结构化日志

    let cfg = Config::from_env()?;                                       // ⭐ 配置
    let pool = PgPoolOptions::new().max_connections(20)
        .connect(&cfg.database_url).await?;                              // ⭐ 连接池

    let state = AppState { pool, cfg: Arc::new(cfg) };                    // ⭐ Arc 共享只读

    let app = Router::new()
        .route("/users/{id}", get(get_user))
        .layer(TimeoutLayer::new(Duration::from_secs(5)))                // ⭐ 超时
        .layer(TraceLayer::new_for_http())
        .with_state(state);

    let listener = TcpListener::bind("0.0.0.0:3000").await?;
    axum::serve(listener, app)
        .with_graceful_shutdown(shutdown_signal())                       // ⭐ 优雅关闭
        .await?;
    Ok(())
}

// handler：错误统一转成 HTTP 响应
async fn get_user(State(s): State<AppState>, Path(id): Path<i64>)
    -> Result<Json<User>, AppError>
{
    let user = sqlx::query_as!(User, "SELECT * FROM users WHERE id = $1", id)
        .fetch_optional(&s.pool).await?
        .ok_or(AppError::NotFound)?;                                     // ⭐ 无 unwrap
    Ok(Json(user))
}

impl IntoResponse for AppError {                                         // ⭐ 错误→响应
    fn into_response(self) -> Response {
        let (code, msg) = match self {
            AppError::NotFound => (StatusCode::NOT_FOUND, "未找到"),
            AppError::Db(e) => {
                error!(?e, "数据库错误");                                 // ⭐ 细节只进日志
                (StatusCode::INTERNAL_SERVER_ERROR, "服务异常")           // ⭐ 不泄露给客户端
            }
        };
        (code, Json(json!({ "message": msg }))).into_response()
    }
}
```

---

### 6. 全文核心规范汇总

| 维度 | ⭐ 核心原则 |
| ---- | ---------- |
| 所有权 | 参数默认 `&T`/`&str`/`&[T]`；`clone()` 要有理由 |
| 错误 | 库用 `thiserror`，应用用 `anyhow`；业务代码禁止 `unwrap` |
| 类型 | 用类型表达约束（newtype、枚举、typestate），把运行时错误变编译错误 |
| 集合 | 更新 Map 用 `entry`；已知规模先 `with_capacity` |
| 迭代 | 优先迭代器而非下标；避免中间 `collect` |
| 抽象 | 默认静态分发，需要异构集合才用 `dyn` |
| 并发 | 优先通道，其次 `Arc<Mutex>`；临界区最小化，绝不持锁做 IO |
| 异步 | IO 密集用异步，CPU 密集用 `spawn_blocking`；并发必须有上限和超时 |
| 工程 | workspace 分层；CI 跑 `fmt` + `clippy -D warnings` + `test` |
| 性能 | 先测量再优化；只优化热点；一律 `--release` 测 |
| unsafe | 业务代码 `#![forbid(unsafe_code)]` |

---

### 7. 面试重点

**Q1：Rust 中如何实现设计模式？和 Java 有什么不同？**

很多 GoF 模式在 Rust 里**被语言内置或简化了**：迭代器模式是 `Iterator` trait；RAII/资源管理由 `Drop` 自动完成，不需要 `try-with-resources`；模板方法就是 trait 默认方法；策略模式直接传闭包或泛型参数；单例用 `OnceLock`。而**因为没有继承**，所有基于继承的模式都改用**组合 + trait**表达。此外 Rust 有几个特有模式：**newtype**（零成本类型安全）、**typestate**（用类型参数表达状态机，非法迁移编译不过）、**extension trait**（给外部类型加方法）。

**Q2：什么是 typestate 模式？**

用**泛型类型参数表示对象所处的状态**，不同状态的 `impl` 块提供不同方法，从而让非法的状态迁移**根本无法编译**。例如 `Door<Closed>` 只有 `open()`、`Door<Open>` 只有 `close()`，对已打开的门再调 `open()` 是编译错误。实际用于构建器（必填字段没设置就不能 `build()`）、数据库事务（已提交不能再写）、连接状态机。它体现了 Rust 的核心价值主张：**把运行时错误提前成编译错误**。

**Q3：Rust 项目怎么做依赖注入和单元测试？**

把外部依赖（数据库、HTTP 客户端、消息队列）抽象成 **trait**，业务代码依赖 trait 而非具体实现。注入有两种方式：**泛型参数**（`struct Service<R: Repo>`，静态分发、零开销、编译期确定）和 **trait 对象**（`Arc<dyn Repo + Send + Sync>`，运行时可替换、适合配置驱动）。测试时用 `mockall` 自动生成 mock 实现注入。单元测试写在源文件内的 `#[cfg(test)] mod tests`，集成测试放 `tests/`。

**Q4：Rust 项目中有哪些常见的反模式？**

① **到处 `clone()`** 绕过借用检查——把编译问题变成性能问题；② **到处 `unwrap()`**——生产环境直接 panic；③ **`Rc<RefCell<T>>` / `Arc<Mutex<T>>` 满天飞**——通常说明所有权设计有问题，前者应重新设计或用 arena+索引，后者应改 actor 模式；④ **用 `String` 当错误类型**——调用方无法分类处理；⑤ **业务结构体存 `&'a T`**——生命周期传染整个调用链；⑥ **过早抽象**——只有一个实现却先定义 trait；⑦ **异步里调阻塞代码**——卡死整个工作线程；⑧ **按 Java 习惯写 Rust**——getter/setter 满天飞、试图模拟继承。

**Q5：你会怎么设计一个 Rust Web 服务？**

技术栈：`tokio` + `axum` + `sqlx` + `serde` + `tracing` + `thiserror`/`anyhow`。结构上用 **workspace 分层**（domain / infra / service / api），依赖方向单向。要点：① 共享只读状态用 `Arc`，连接池自带并发管理；② 错误定义成枚举并实现 `IntoResponse` 统一转 HTTP 响应，**内部细节只进日志、不返回给客户端**；③ 全链路 `tracing` + `#[instrument]`；④ 所有外部调用加 **超时**，并发有 **上限**（`Semaphore`）；⑤ 实现**优雅关闭**；⑥ 业务代码零 `unwrap`；⑦ CI 跑 `fmt` + `clippy -D warnings` + `test`。

**Q6：从 Java 转 Rust 最需要转变的思维是什么？**

① **从"对象引用"到"所有权"**——不再是"谁都能持有引用"，而是要想清楚数据归谁、谁借用多久；② **从继承到组合 + trait**——没有类层次，用枚举表达封闭集合、用 trait 表达能力；③ **从异常到返回值**——错误是 `Result`，用 `?` 传播，不存在"任意位置抛出"；④ **从 null 到 `Option`**——编译器强制处理空值；⑤ **让编译器成为盟友而非对手**——报错通常意味着设计问题，用 `clone`/`unsafe` 硬闯是走错方向；⑥ **用类型表达约束**——newtype、枚举、typestate 把大量运行时校验提前到编译期。

---

## 结语

全文 26 章的主线其实只有一句话：

> ⭐ **Rust 把「正确性」从运行时移到了编译期。**

所有权和借用消灭了内存错误与数据竞争，`Option`/`Result` 消灭了空指针和被忽略的异常，`match` 的穷尽性保证了分支不遗漏，`Send`/`Sync` 保证了并发安全，newtype 和 typestate 让业务约束也由编译器守护——代价是前期要多想一层，收益是**编译通过的代码，运行时几乎不会以这些方式出错**。

面试准备建议按这个优先级：

1. **必答**：所有权三原则、借用规则、`Option`/`Result` 与 `?`、`String` vs `&str`、`Vec`/`HashMap`
2. **高频**：生命周期与省略规则、trait 与静态/动态分发、`Send`/`Sync`、`Rc`/`Arc`/`RefCell`、迭代器惰性与零成本
3. **进阶**：`Pin` 与 async 状态机、对象安全、单态化与代码膨胀、UB 与 `unsafe` 边界、性能分析方法
4. **加分**：空位优化、NLL、typestate、`thiserror`/`anyhow` 选型、`spawn_blocking` 的判据

工程上最值钱的三条习惯：**参数默认借用、业务代码零 `unwrap`、遇到编译错误先反思设计而不是 `clone`**。
