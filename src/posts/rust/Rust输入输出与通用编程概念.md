---
date: 2022-12-2
category:
  - 后端
tag:
  - Rust
archive: true
---



# Rust输入输出与通用编程概念

### 输入输出与猜数字

1.输入输出

```rust
// use std::io; //如果prelude（序曲/预导入）没有，需要显式导入

fn main() {
    println!("猜数！"); /* !是一个宏，不是一个函数 */

    println!("猜一个数");

    // let mut foo = 1;
    // let bar = foo;

    // foo = 2; //cannot assign twice to immutable variable `foo`

    /* Rust认为，应该显式地区分可变于不可变，如果我们要声明一个可变的变量，则需要使用mut关键字 */

    //声明可变变量guess，将guess的值绑定到了一个空的字符串实例上
    let mut guess = String::new();
    /* String是类型，::两个冒号是该类型的一个关联函数（pub struct String）
    类似于一个静态方法，new()是创建实例的一个惯用函数名*/

    std::io::stdin().read_line(&mut guess).expect("无法读取行");
    /*传入的是一个引用，所以使用&(涉及所有权)
    可以直接在使用std库函数时使用std::来引用io中stdin的函数
    read_line用来读取用户输入的内容，并将其放入字符串中，因为我们输入的值是改变的，所以我们需要让其为mut可变的，并且
    我们使用了取地址符号&来表示我们引用这个值
    */
    // io::Result OK,Err

    println!("你猜测的数是:{}", guess);
}
```



cargo是rust的构建系统和软件包管理器。

cargo.toml中添加

```toml
[dependencies]
rand = "^0.8.5"
```



出现：`Blocking waiting for file lock on package cache`

换源在用户文件夹下的`.cargo`文件夹中添加config文件，换源：

```
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
# 如果所处的环境中不允许使用 git 协议，可以把上面的地址改为
# registry = "https://mirrors.ustc.edu.cn/crates.io-index"
#[http]
#check-revoke = false
```



如果还不行，删除`.package-cache`文件。

重新执行`cargo build`下载依赖。



```rust
use rand::Rng; //trait
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("猜数字游戏！");
    //i32,u32,i64 下面使用了guess与其比较，所以是u32，默认是i32。
    let secret_number = rand::thread_rng().gen_range(1..101);
    println!("神秘数字是：{}", secret_number);
    println!("猜一个数");
    let mut guess = String::new();
    io::stdin().read_line(&mut guess).expect("无法读取行");
    println!("你猜测的数是:{}", guess);

    // shadow
    let guess: u32 = guess.trim().parse().expect("Please type a number!"); //Result<F, F::Err>

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too Small"), //arm
        Ordering::Greater => println!("Too Big"),
        Ordering::Equal => println!("You Win!"),
    }
}
```





多次猜测：

```rust
use rand::Rng; //trait
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("猜数字游戏！");
    //i32,u32,i64 下面使用了guess与其比较，所以是u32，默认是i32。
    let secret_number = rand::thread_rng().gen_range(1..101);
    // println!("神秘数字是：{}", secret_number);

    loop {
        println!("猜一个数");
        let mut guess = String::new();
        io::stdin().read_line(&mut guess).expect("无法读取行");
        println!("你猜测的数是:{}", guess);

        // shadow
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too Small"), //arm
            Ordering::Greater => println!("Too Big"),
            Ordering::Equal => {
                println!("You Win!");
                break;
            }
        }
    }
}
```



# 三、通用编程概念

## 1.变量与可变性

### let变量

我们在声明变量时使用`let`关键字，默认情况下，变量是不可变的（immutable）。

let与const不同。let 与 const最大的区别在于初始化，let的初始化在程序中是不确定的，且声明与赋值可以分开。另外，const的初始化无类型推导，强迫开发者手动加类型注释。而且const后面必须是常量，初始化时的值是永远确定的。

如果我们要使变量为可变变量，需要在`let`后跟`mut`关键字。

```rust
fn main() {
    let x = 5;
    println!("The value of x is {}", x);

    // x = 6; //cannot assign twice to immutable variable `x`
    let mut y = 5;
    println!("The value of y is {}", y);
    y = 6;
    println!("The value of y is {}", y);
}
```

### const常量

常量在绑定值以后也是不可变的的，但与不可变变量有区别：

 - 不可以使用mut，常量永远都是不可变的。
- 声明常量使用const关键字，必须标注类型。
- 常量可以在任何作用域内进行声明，包括全局作用域。
- 常量只能绑定到常量表达式，无法绑定到函数的调用结果或只能在运行时才能计算数的值。

需要注意的是：在程序运行期间，常量在其声明的作用域内一直有效。

命名规范：Rust中常量使用全大写字母，每个单词之间使用下划线分开：`MAX_POINTS`

```rust
const MAX_POINTS: u32 = 100_000;//可在全局作用域中声明

fn main() {
    const MIN_POINTS: u32 = 10_000;//可使用下划线来分割，增强可读性
    println!("The value of MAX_POINTS is {}", MAX_POINTS);
    println!("The value of MIN_POINTS is {}", MIN_POINTS);
}
```



### Shadowing（隐藏）

我们可以重复声明一个属性来对之前声明的变量进行隐藏，在后续的代码中的这个变量名代表的就是新的变量：

```rust
fn main() {
    let x = 5;
    let x = x + 1;
    let x = x * 2;
    println!("The value of x is {}", x) //The value of x is 12
}
```

shadow和把变量标记为mut是不一样的：

- 如果不使用let关键字，那么重新给非mut的变量赋值将会导致编译时错误。

- 而使用let声明的同名新变量，也是不可变的。

- 使用let声明的同名新变量，它的类型可以与之前不同：

```rust
fn main() {
    let spaces = "    ";
    let spaces = spaces.len();
    println!("{}", spaces) //4
}
```

如果我们使用mut则会报错，因为类型不同：

```rust
fn main() {
    let mut spaces = "    ";
    spaces = spaces.len(); //expected `&str`, found `usize`
}
```

shadowing的设计得到的好处是非常明显的，比如我们先得到了一个字符串`spaces_str`，我们将其转换为数字`spaces_num`，但从表达的意图来说，这个后缀又是多余的，所以使用shadowing并不是坏处。



## 2.数据类型

Rust是静态编译语言，在编译时必须知道所有变量的类型。

- 基于使用的值，编译器通常可以推断出它具体的类型。
- 但如果可能的类型比较多（例如把String转为整数parse方法），就必须添加类型的标注，否则编译会报错：

```rust
fn main() {
    //如果不写u32类型，会报错：type annotations needed，因为编译器不知道该转为那种数字类型
    let guess: u32 = "42".parse().expect("Not a number");
    println!("{}", guess);
}
```

### 标量类型

一个标量类型代表一个单个的值

rust有4种主要的标量类型：

- 整数类型
- 浮点类型
- 布尔类型
- 字符类型

#### 整数类型

整数类型没有小数部分，u32就是一个无符号的整数类型，占据32位的空间

- 无符号整数类型以u开头
- 有符号的整数类型以i开头

![image-20221202194746413](https://qiankun825.oss-cn-hangzhou.aliyuncs.com/img/image-20221202194746413.png)



有符号的类型有正有负，无符号的类型是非负数。

范围（设位数为n）：

![image-20221202195104165](https://qiankun825.oss-cn-hangzhou.aliyuncs.com/img/image-20221202195104165.png)



#### isize和usize类型

由arch，计算机架构决定，如果是64位，就是64位的。

使用场景主要是对某种集合进行索引操作。



#### 整数字面值

除byte类型外，所以数值类型字面值都允许使用类型后缀，如57u8。

不清楚用什么类型就用默认类型，整数是i32。



#### 整数的溢出

u8范围为0-255，如果设为256：

- 调试模式下编译会被检查出整数溢出，发生溢出，程序会panic（只会在程序运行时才回抛出来的异常）
- --release下编译，不会检查，可能会导致溢出，如果溢出会发生“环绕”：256为0，257为1。但程序不会panic



#### 浮点类型

f32，32位，单精度。f64（默认），64位，双精度。



#### 数值操作

加减乘除余等

```rust
fn main() {
    let sum = 5 + 10; //i32
    let differnce = 95.5 - 4.3; //f64
    let product = 4 * 30; //i32
    let quotient = 56.7 / 32.2; //f64
    let reminder = 54 % 5; //i32
    let yu = 1.0 % 0.3; //f64 Rust支持对浮点类型的取余（两边都是同类型浮点数才行）
}
```



#### 布尔

true/false，一个字节大写，类型为bool



#### 字符

- char类型被用来描述最基础的单个字符。

- 字符类型的字面值使用单引号
- 占用4字节大写
- 是Unicode标量值

rust中的字符可能与一般理解的字符不同。

```rust
fn main() {
    let x = 'x'; //char
    let y: char = 'β'; //char
    let z = '😁'; //char
}
```



### 复合类型

- 复合类型可以将多个值放在一个类型里。

- rust提供了两种基础的复合类型：元组（Tuple）、数组

#### Tuple

可将多个类型的多个值放在一个类型里，Tuple的长度固定，一旦声明无法改变。

创建Tuple：在小括号中将值用逗号分开，每个位置都对应一个类型，Tuple中各元素的类型不必相同：

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    println!("{},{},{}", tup.0, tup.1, tup.2); //500,6.4,1
}
```



- **获取tuple的元素值——使用模式匹配来解构**

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    let (x, y, z) = tup;
    println!("{},{},{}", x, y, z); //500,6.4,1
}
```



- **访问tuple的元素，使用点标记法：`tup.0`**



#### 数组

- 数组也可以将多个值放在一个类型里。
- 数组中每个元素的类型必须相同。
- 数组的长度也固定

```rust
fn main() {
    let arr = [1, 2, 3]; //[i32; 3]
    println!("{},{},{}", arr[0], arr[1], arr[2]); //1,2,3
}
```

##### 使用场景

如果想让数据存放在stack（栈）上而不是heap（堆）上，或者想保证有固定数量的元素，这时使用数组更有好处。

另外，数组没有Vector（标准库提供）灵活，并且Vector用得更多。

- vector长度可变
- 如果不确定用哪个，就用vector

##### 数组类型

```rust
let arr:[i32; 3] = [1, 2, 3];
```

- 如果数组的每个元素值都相同，那么可以在中括号中指定初始值，然后是`;`，最后是数组长度：

```rust
let arr = [5; 3];
println!("{},{},{}", arr[0], arr[1], arr[2]); //5,5,5
```



##### 访问数组元素

使用索引`arr[0]`，`arr[1]`。

如果访问的索引超出了数组的范围，那么：

- 编译会通过（不一定）
- 运行会报错（panic），rust不允许其继续访问相应地址的内存。





### 3.函数

#### 定义

- 声明函数使用`fn`关键字
- 针对函数和变量名，rust使用`snake_case`命名规范：所有字母都是小写，单词间使用下划线分开。

```rust
fn main() {
    println!("hello main");
    another_function();
}

fn another_function() {
    println!("Another func");
}
```

#### 参数

parameters，arguments（形参，实参）

函数签名中必须指明所有参数的类型

```rust
fn main() {
    //arguments
    another_function(1, 2.2); //a=1，b=2.2
}

fn another_function(a: i32, b: f64) {//parameter
    println!("a={}，b={}", a, b);
}
```



#### 语句与表达式

函数的定义也是语句

语句不返回值，所以我们不能使用let将一个语句赋值给一个变量：

```rust
let x = (let y = 6);//error

let y = 6;//6是一个表达式，值为6，作为语句的一部分。
```

调用宏如`println!("");`也是一个表达式

下面的块表达式的值为4，如果在`x+3`后加`;`，则变成一个语句，返回一个空的tuple

```rust
fn main() {
    let x = 5;

    let y = {
        let x = 1;
        x + 3
    };

    println!("y={}", y); //y=4
}
```



#### 函数的返回值

- 在`->`符号后边声明函数返回值的类型，但是不可以以返回值命名
- rust中，返回值是函数体中最后表达式的值
- 若想提前返回，需使用return，并指定一个值。

```rust
fn main() {
    let x = cc();
    println!("x={}", x); //x=5

    let y = dd(2);
    println!("y={}", y); //y=3
}

fn cc() -> i128 {
    5
}

fn dd(x: i128) -> i128 {
    x + 1
}
```





### 4.注释

单行`//`，多行`/* */`或`//`。

另外，rust中还有一种特殊的注释：**“文档注释”**



### 5.控制流

#### if表达式

- 条件必须为bool类型，不像js这样会进行转换。

- if表达式中，与条件相关联的代码块叫分支（arm）
- 可选的，在后边可以加上一个else表达式

```rust
fn main() {
    let number = 3;
    if number < 5 {
        println!("true");
    } else {
        println!("false");
    }
}
```



#### else if

```rust
fn main() {
    let number = 6;
    if number % 4 == 0 {
        println!("divisible by 4");
    } else if number % 3 == 0 {
        println!("divisible by 3");
    } else if number % 2 == 0 {
        println!("divisible by 2");
    } else {
        println!("false");
    }
    //divisible by 3 按顺序执行
}
```



如果用了多于一个else if，最好使用match来重构代码

```rust
fn main() {
    judge(); //divisible by 2
}

fn judge() {
    let number = 6;
    for ele in [2, 4, 6] {
        let print = match number % ele {
            0 => ele.to_string(),
            _ => "false".to_string(),
        };
        return println!("divisible by {}", print);
    }
}
```





#### 在let语句中使用if

因为if是一个表达式，所以可以将他放在let语句中等号的后面

```rust
fn main() {
    let condition = true;
    let numebr = if condition { 5 } else { 6 };
    println!("number={}", numebr); //number=5
}
//如果将6改为“6”，会报错。if和else的类型不兼容。无法在编译时确定number的类型，rust要求ifelse表达式中可能成为结果的分支返回的类型一致。
```



#### 循环

loop，for，while

##### loop

反复循环，直到喊停。

可以使用break关键字。

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("result is {}", result); //result is 20
}
```



##### while条件循环

每次执行循环体之前都判断一次条件：

```rust
fn main() {
    let mut counter = 3;

    while counter != 0 {
        println!("{}!", counter);
        counter = counter - 1;
    }
    println!("LIFTOFF!");
}
```



##### for循环遍历集合

可以使用while或loop遍历集合，但是易错且低效：

第一是index的范围容易搞错，二是低效，每次都会进行判断

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;
	
    while index < 5 {
        println!("{}!", a[index]);

        index = index + 1;
    }
}
```

所以我们使用for循环来遍历：

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    for ele in a.iter() {
        println!("{}!", ele);
    }
}
```

由于for的安全，简洁性，所以在rust中用得最多。



##### Range

- 标准库提供
- 指定一个开始数字和一个结束数字，range可以生成他们之间的数字（不包括结束）
- rev方法可以反转Range

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF");
}
// 3!
// 2!
// 1!
// LIFTOFF
```

