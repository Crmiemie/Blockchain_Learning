 <img src="/Users/yangchaoran/Library/Application Support/typora-user-images/截屏2023-08-31 09.48.04.png" alt="截屏2023-08-31 09.48.04" style="zoom:60%;" />

| **报告名称**：   | 区块链复习笔记   |
| ---------------- | ---------------- |
| **学生姓名：**   | 杨超然           |
| **学生学号：**   | 202106060220     |
| **专业班级：**   | 信管2101班       |
| **学**  **院：** | 工商管理学院     |
| **指导老师：**   | 周中定           |
| **日  期：**     | 2023-2024 Autumn |

------

# Go语言结构

基础组成部分：包声明、引入包、函数、变量、语句&表达式、注释

```go
package main /* 定义包名 */

import "fmt" /* 告诉编译器程序需要引入fmt包，fmt 包实现了格式化 IO（输入/输出）的函数 */

func main() { /* 这个括号不能放在单独一行，否则会报错 */
   /* 这是我的第一个简单的程序 */
   fmt.Println("Hello, World!")
}
```

执行go程序的两种方式：

```bash
$ go run hello.go

$ go build hello.go # 生成go的二进制文件
$ ./hello
```

# Go 语言基础语法

## 标记

Go程序的标记可以为：**关键字，标识符，常量，字符串，符号**

```go
以下语句由6个标记组成：fmt.Println("Hello, World!")
分别为：
1. fmt
2. .
3. Println
4. (
5. "Hello, World!"
6. )
```

## 标识符

就是一个或是多个字母(A~Z和a~z)数字(0~9)、下划线_组成的序列，但是**第一个字符必须是字母或下划线**而不能是数字

以下是无效的标识符：

- 1ab（以数字开头）
- case（Go 语言的关键字）
- a+b（运算符是不允许的）

## 关键字

下面列举了 Go 代码中会使用到的 25 个关键字或保留字：

| break    | default     | func   | interface | select |
| -------- | ----------- | ------ | --------- | ------ |
| case     | defer       | go     | map       | struct |
| chan     | else        | goto   | package   | switch |
| const    | fallthrough | if     | range     | type   |
| continue | for         | import | return    | var    |

除了以上介绍的这些关键字，Go 语言还有 36 个预定义标识符：

| append | bool    | byte    | cap     | close  | complex | complex64 | complex128 | uint16  |
| ------ | ------- | ------- | ------- | ------ | ------- | --------- | ---------- | ------- |
| copy   | false   | float32 | float64 | imag   | int     | int8      | int16      | uint32  |
| int32  | int64   | iota    | len     | make   | new     | nil       | panic      | uint64  |
| print  | println | real    | recover | string | true    | uint      | uint8      | uintptr |

程序一般由关键字、常量、变量、运算符、类型和函数组成。

程序中可能会使用到这些分隔符：括号 ()，中括号 [] 和大括号 {}。

程序中可能会使用到这些标点符号：**.**、**,**、**;**、**:** 和 **…**。

## 空格

Go 语言中变量的声明必须使用空格隔开：

```go
var x int
const Pi float64 = 3.14159265358979323846
```

在变量与运算符间加入空格，程序看起来更加美观：

```go
fruit = apples + oranges; 
```

在关键字和表达式之间要使用空格：

```go
if x > 0 {
    // do something
}
```

在函数调用时，函数名和左边等号之间要使用空格，参数之间也要使用空格：

```go
result := add(2, 3)
```

## 格式化字符串

用**fmt.Sprintf** 或 **fmt.Printf** 格式化字符串并赋值给新串：

- **Sprintf** 根据格式化参数生成格式化的字符串并返回该字符串。（即得到一个新字符串，还需要使用fmt.Println()输出结果）
- **Printf** 根据格式化参数生成格式化的字符串并写入标准输出。（直接输出结果）

# Go语言数据类型

分为布尔型、数字类型、字符串类型、派生类型

[reference link](https://www.runoob.com/go/go-data-types.html)

# Go语言变量

变量是计算机语言中能储存计算结果或能表示值抽象概念，可以通过变量名访问

Go 语言变量名由字母、数字、下划线组成，其中**首个字符不能为数字**。

使用 **var** 关键字进行声明

## 声明方法

第一种，指定变量类型，如果没有初始化，则变量默认为**零值**

```go
var v_name v_type
v_name = value
```

```go
package main
import "fmt"
func main() {

    // 声明一个变量并初始化
    var a = "RUNOOB"
    fmt.Println(a)

    // 没有初始化就为零值
    var b int
    fmt.Println(b)

    // bool 零值为 false
    var c bool
    fmt.Println(c)
}
```

各种类型的零值如下：

数值类型：0 

布尔类型：false 

字符串：“”

以下几种类型为 **nil**：

```go
var a *int
var a []int
var a map[string] int
var a chan int
var a func(string) int
var a error // error 是接口
```

第二种，根据值自行判定变量类型。

```go
var v_name = value
```

第三种，用:=直接赋值（注：**如果变量已经使用 var 声明过了，再使用 := 声明变量，就产生编译错误**）

```go
v_name := value
```

例如，**intVal := 1** 相等于：

```go
var intVal int 
intVal =1 
```

## 多变量声明

```go
//类型相同多个变量, 非全局变量
var vname1, vname2, vname3 type
vname1, vname2, vname3 = v1, v2, v3

var vname1, vname2, vname3 = v1, v2, v3 // 和 python 很像,不需要显示声明类型，自动推断

vname1, vname2, vname3 := v1, v2, v3 // 出现在 := 左侧的变量不应该是已经被声明过的，否则会导致编译错误


// 这种因式分解关键字的写法一般用于声明全局变量
var (
    vname1 v_type1
    vname2 v_type2
)
```

## 注意事项

1. 在相同的代码块中，我们不可以再次对于相同名称的变量使用初始化声明，例如：a := 20 就是不被允许的，编译器会提示错误 no new variables on left side of :=，但是 a = 20 是可以的，因为这是给相同的变量赋予一个新的值。
2. 在定义变量 a 之前使用它，则会得到编译错误 undefined: a
3. 如果你声明了一个局部变量却没有在相同的代码块中使用它，同样会得到编译错误（但是全局变量是允许声明但不使用的）
4. 如果你想要交换两个变量的值，则可以简单地使用 **a, b = b, a**，两个变量的类型必须是相同
5. 空白标识符 _ 也被用于抛弃值，如值 5 在：_, b = 5, 7 中被抛弃



# Go语言常量

常量即不会被修改的量，其类型只可以是**布尔型、数字型（整数型、浮点型和复数）和字符串型**

使用**const**关键字进行声明

你可以省略类型说明符 [type]，因为编译器可以根据变量的值来推断其类型。

- 显式类型定义： `const b string = "abc"`
- 隐式类型定义： `const b = "abc"`

注：常量可以用**len()**, **cap()**, **unsafe.Sizeof()**函数计算表达式的值。常量表达式中，函数**必须是内置函数**，否则编译不过

## iota

iota，特殊常量，可以认为是一个**可以被编译器修改的常量**

iota 在 const关键字出现时将**被重置为 0**(const 内部的第一行之前)，const 中每新增一行常量声明将使 iota 计数一次

第一个 iota 等于 0，每当 iota 在新的一行被使用时，它的值都会自动加 1；所以 a=0, b=1, c=2 可以简写为如下形式：

```go
const (
    a = iota
    b
    c
)
```

iota实例：

```go
package main

import "fmt"

func main() {
    const (
            a = iota   //0
            b          //1
            c          //2
            d = "ha"   //独立值，iota += 1
            e          //"ha"   iota += 1
            f = 100    //iota +=1
            g          //100  iota +=1
            h = iota   //7,恢复计数
            i          //8
    )
    fmt.Println(a,b,c,d,e,f,g,h,i)
}
```

运行结果：

```go
0 1 2 ha ha 100 100 7 8
```

```go
package main

import "fmt"
const (
    i=1<<iota
    j=3<<iota
    k
    l
)

func main() {
    fmt.Println("i=",i)
    fmt.Println("j=",j)
    fmt.Println("k=",k)
    fmt.Println("l=",l)
}
```

运行结果：

```go
i= 1
j= 6
k= 12
l= 24
```

**i=1<<0**, **j=3<<1**（**<<** 表示左移的意思），即：i=1, j=6，**k=3<<2**，**l=3<<3**

注：**<<n==\*(2^n)**。

#  Go语言运算符

## 关系运算符

A=10 B=20

| 运算符 | 描述 | 实例               |
| :----- | :--- | :----------------- |
| +      | 相加 | A + B 输出结果 30  |
| -      | 相减 | A - B 输出结果 -10 |
| *      | 相乘 | A * B 输出结果 200 |
| /      | 相除 | B / A 输出结果 2   |
| %      | 求余 | B % A 输出结果 0   |
| ++     | 自增 | A++ 输出结果 11    |
| --     | 自减 | A-- 输出结果 9     |

## 关系运算符

A=10 B=20

| 运算符 | 描述                                                         | 实例              |
| :----- | :----------------------------------------------------------- | :---------------- |
| ==     | 检查两个值是否相等，如果相等返回 True 否则返回 False。       | (A == B) 为 False |
| !=     | 检查两个值是否不相等，如果不相等返回 True 否则返回 False。   | (A != B) 为 True  |
| >      | 检查左边值是否大于右边值，如果是返回 True 否则返回 False。   | (A > B) 为 False  |
| <      | 检查左边值是否小于右边值，如果是返回 True 否则返回 False。   | (A < B) 为 True   |
| >=     | 检查左边值是否大于等于右边值，如果是返回 True 否则返回 False。 | (A >= B) 为 False |
| <=     | 检查左边值是否小于等于右边值，如果是返回 True 否则返回 False。 | (A <= B) 为 True  |

## 逻辑运算符

A 值为 True，B 值为 False

| 运算符      | 描述                                                         | 实例               |
| :---------- | :----------------------------------------------------------- | :----------------- |
| &&（并）    | 逻辑 AND 运算符。 如果两边的操作数都是 True，则条件 True，否则为 False。 | (A && B) 为 False  |
| \|\|（交）  | 逻辑 OR 运算符。 如果两边的操作数有一个 True，则条件 True，否则为 False。 | (A \|\| B) 为 True |
| !（取相反） | 逻辑 NOT 运算符。 如果条件为 True，则逻辑 NOT 条件 False，否则为 True。 | !(A && B) 为 True  |

## 🌟位运算符

对整数在内存中的二进制位进行操作

| p    | q    | p & q | p \| q | p ^ q |
| :--- | :--- | :---- | :----- | :---- |
| 0    | 0    | 0     | 0      | 0     |
| 0    | 1    | 0     | 1      | 1     |
| 1    | 1    | 1     | 1      | 0     |
| 1    | 0    | 0     | 1      | 1     |

假定 A = 60; B = 13; 其二进制数转换为：

```go
A = 0011 1100

B = 0000 1101

-----------------

A&B = 0000 1100

A|B = 0011 1101

A^B = 0011 0001
```

Go 语言支持的位运算符如下表所示。假定 A 为60，B 为13：

| 运算符 | 描述                                                         | 实例                                   |
| :----- | :----------------------------------------------------------- | :------------------------------------- |
| &      | 按位与运算符"&"是双目运算符。 其功能是参与运算的两数各对应的二进位相与。 | (A & B) 结果为 12, 二进制为 0000 1100  |
| \|     | 按位或运算符"\|"是双目运算符。 其功能是参与运算的两数各对应的二进位相或 | (A \| B) 结果为 61, 二进制为 0011 1101 |
| ^      | 按位异或运算符"^"是双目运算符。 其功能是参与运算的两数各对应的二进位相异或，**当两对应的二进位相异时，结果为1**。 | (A ^ B) 结果为 49, 二进制为 0011 0001  |
| <<     | 左移运算符"<<"是双目运算符。**左移n位就是乘以2的n次方**。 其功能把"<<"左边的运算数的各二进位全部左移若干位，由"<<"右边的数指定移动的位数，高位丢弃，低位补0。 | A << 2 结果为 240 ，二进制为 1111 0000 |
| >>     | 右移运算符">>"是双目运算符。**右移n位就是除以2的n次方**。 其功能是把">>"左边的运算数的各二进位全部右移若干位，">>"右边的数指定移动的位数。 | A >> 2 结果为 15 ，二进制为 0000 1111  |

## 赋值运算符

| 运算符 | 描述                                           | 实例                                  |
| :----- | :--------------------------------------------- | :------------------------------------ |
| =      | 简单的赋值运算符，将一个表达式的值赋给一个左值 | C = A + B 将 A + B 表达式结果赋值给 C |
| +=     | 相加后再赋值                                   | C += A 等于 C = C + A                 |
| -=     | 相减后再赋值                                   | C -= A 等于 C = C - A                 |
| *=     | 相乘后再赋值                                   | C *= A 等于 C = C * A                 |
| /=     | 相除后再赋值                                   | C /= A 等于 C = C / A                 |
| %=     | 求余后再赋值                                   | C %= A 等于 C = C % A                 |
| <<=    | 左移后赋值                                     | C <<= 2 等于 C = C << 2               |
| >>=    | 右移后赋值                                     | C >>= 2 等于 C = C >> 2               |
| &=     | 按位与后赋值                                   | C &= 2 等于 C = C & 2                 |
| ^=     | 按位异或后赋值                                 | C ^= 2 等于 C = C ^ 2                 |
| \|=    | 按位或后赋值                                   | C \|= 2 等于 C = C \| 2               |

## 其他运算符

| 运算符 | 描述             | 实例                       |
| :----- | :--------------- | :------------------------- |
| &      | 返回变量存储地址 | &a; 将给出变量的实际地址。 |
| *      | 指针变量。       | *a; 是一个指针变量         |

## 运算符优先级

| 优先级 | 运算符           |
| :----- | :--------------- |
| 5      | * / % << >> & &^ |
| 4      | + - \| ^         |
| 3      | == != < <= > >=  |
| 2      | &&               |
| 1      | \|\|             |

当然，你可以通过使用括号来临时提升某个表达式的整体运算优先级



# Go语言条件语句

Go 语言提供了以下几种条件判断语句：

| 语句                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [if 语句](https://www.runoob.com/go/go-if-statement.html)    | **if 语句** 由一个布尔表达式后紧跟一个或多个语句组成。       |
| [if...else 语句](https://www.runoob.com/go/go-if-else-statement.html) | **if 语句** 后可以使用可选的 **else 语句**, else 语句中的表达式在布尔表达式为 false 时执行。 |
| [if 嵌套语句](https://www.runoob.com/go/go-nested-if-statements.html) | 你可以在 **if** 或 **else if** 语句中嵌入一个或多个 **if** 或 **else if** 语句。 |
| [switch 语句](https://www.runoob.com/go/go-switch-statement.html) | **switch** 语句用于基于不同条件执行不同动作。                |
| [select 语句](https://www.runoob.com/go/go-select-statement.html) | **select** 语句类似于 **switch** 语句，但是select会随机执行一个可运行的case。如果没有case可运行，它将阻塞，直到有case可运行。 |

## 一些例子

switch 语句还可以被用于 type-switch 来判断某个 interface 变量中实际存储的变量类型

```go
package main

import "fmt"

func main() {
   var x interface{}
     
   switch i := x.(type) {
      case nil:   
         fmt.Printf(" x 的类型 :%T",i)                
      case int:   
         fmt.Printf("x 是 int 型")                       
      case float64:
         fmt.Printf("x 是 float64 型")           
      case func(int) float64:
         fmt.Printf("x 是 func(int) 型")                      
      case bool, string:
         fmt.Printf("x 是 bool 或 string 型" )       
      default:
         fmt.Printf("未知型")     
   }   
}
```

以上代码执行结果为：

```go
x 的类型 :<nil>
```



使用 fallthrough 会强制执行后面的 case 语句，fallthrough 不会判断下一条 case 的表达式结果是否为 true。

```go
package main

import "fmt"

func main() {

    switch {
    case false:
            fmt.Println("1、case 条件语句为 false")
            fallthrough
    case true:
            fmt.Println("2、case 条件语句为 true")
            fallthrough
    case false:
            fmt.Println("3、case 条件语句为 false")
            fallthrough
    case true:
            fmt.Println("4、case 条件语句为 true")
    case false:
            fmt.Println("5、case 条件语句为 false")
            fallthrough
    default:
            fmt.Println("6、默认 case")
    }
}
```

以上代码执行结果为：

```go
2、case 条件语句为 true
3、case 条件语句为 false
4、case 条件语句为 true
```



# Go语言循环语句

Go 语言提供了以下几种类型循环处理语句：

| 循环类型                                                   | 描述                                 |
| :--------------------------------------------------------- | :----------------------------------- |
| [for 循环](https://www.runoob.com/go/go-for-loop.html)     | 重复执行语句块                       |
| [循环嵌套](https://www.runoob.com/go/go-nested-loops.html) | 在 for 循环中嵌套一个或多个 for 循环 |

## 循环控制语句

循环控制语句可以控制循环体内语句的执行过程。

GO 语言支持以下几种循环控制语句：

| 控制语句                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [break 语句](https://www.runoob.com/go/go-break-statement.html) | 经常用于中断当前 for 循环或跳出 switch 语句（跳出循环）      |
| [continue 语句](https://www.runoob.com/go/go-continue-statement.html) | 跳过当前循环的剩余语句，然后继续进行下一轮循环（跳出该次循环，执行下一次循环） |
| [goto 语句](https://www.runoob.com/go/go-goto-statement.html) | 将控制转移到被标记的语句。                                   |



# Go语言函数

Go 语言最少有个 main() 函数

```go
func function_name( [parameter list] ) [return_types] {
   函数体
}
```

函数定义解析：

- func：函数由 func 开始声明
- function_name：函数名称，参数列表和返回值类型构成了函数签名。
- parameter list：参数列表，参数就像一个占位符，**当函数被调用时，你可以将值传递给参数，这个值被称为实际参数**。参数列表指定的是参数类型、顺序、及参数个数。参数是可选的，也就是说函数也可以不包含参数。
- return_types：返回类型，函数返回一列值。return_types 是该列值的数据类型。有些功能不需要返回值，这种情况下 return_types 不是必须的。
- 函数体：函数定义的代码集合。

## 函数调用

调用函数，即向函数传递参数，并返回值

以下例子调用函数返回多个值：

```go
package main

import "fmt"

func swap(x, y string) (string, string) {
   return y, x
}

func main() {
   a, b := swap("Google", "Runoob")
   fmt.Println(a, b)
}
```



## 函数参数

函数如果使用参数，该变量可称为函数的形参。

形参就像定义在函数体内的局部变量。

调用函数，可以通过两种方式来传递参数：

| 传递类型                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [值传递](https://www.runoob.com/go/go-function-call-by-value.html) | 值传递是指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将**不会影响到实际参数**。 |
| [引用传递](https://www.runoob.com/go/go-function-call-by-reference.html) | 引用传递是指在调用函数时将实际参数的地址传递到函数中，那么在函数中对参数所进行的修改，将**影响到实际参数**。 |

默认情况下，Go 语言使用的是值传递，即在调用过程中不会影响到实际参数。



### 值传递例

```go
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var a int = 100
   var b int = 200

   fmt.Printf("交换前 a 的值为 : %d\n", a )
   fmt.Printf("交换前 b 的值为 : %d\n", b )

   /* 通过调用函数来交换值 */
   swap(a, b)

   fmt.Printf("交换后 a 的值 : %d\n", a )
   fmt.Printf("交换后 b 的值 : %d\n", b )
}

/* 定义相互交换值的函数 */
func swap(x, y int) int {
   var temp int

   temp = x /* 保存 x 的值 */
   x = y    /* 将 y 值赋给 x */
   y = temp /* 将 temp 值赋给 y*/

   return temp;
}
```

### 引用传递例

```go
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var a int = 100
   var b int= 200

   fmt.Printf("交换前，a 的值 : %d\n", a )
   fmt.Printf("交换前，b 的值 : %d\n", b )

   /* 调用 swap() 函数
   * &a 指向 a 指针，a 变量的地址
   * &b 指向 b 指针，b 变量的地址
   */
   swap(&a, &b)

   fmt.Printf("交换后，a 的值 : %d\n", a )
   fmt.Printf("交换后，b 的值 : %d\n", b )
}

func swap(x *int, y *int) {
   var temp int
   temp = *x    /* 保存 x 地址上的值 */
   *x = *y      /* 将 y 值赋给 x */
   *y = temp    /* 将 temp 值赋给 y */
}
```



# Go语言变量作用域

作用域为已声明标识符所表示的常量、类型、变量、函数或包在源代码中的**作用范围**

Go 语言中变量可以在三个地方声明：

- **函数内定义**的变量称为**局部变量**
- **函数外定义**的变量称为**全局变量**
- **函数定义中**的变量称为**形式参数**

## 局部变量

局部变量的作用域只在函数体内，参数和返回值变量也是局部变量

## 全局变量

在函数体外声明的变量称之为全局变量，全局变量**可以在整个包甚至外部包（被导出后）使用**。

全局变量**可以在任何函数中使用**

注：Go 语言程序中全局变量与局部变量名称可以相同，但是函数内的**局部变量会被优先考虑**

## 形式参数

形式参数会作为函数的局部变量来使用

```go
package main

import "fmt"

/* 声明全局变量 */
var a int = 20;

func main() {
   /* main 函数中声明局部变量 */
   var a int = 10
   var b int = 20
   var c int = 0

   fmt.Printf("main()函数中 a = %d\n",  a);
   c = sum( a, b);
   fmt.Printf("main()函数中 c = %d\n",  c);
}

/* 函数定义-两数相加 */
func sum(a, b int) int {
   fmt.Printf("sum() 函数中 a = %d\n",  a);
   fmt.Printf("sum() 函数中 b = %d\n",  b);

   return a + b;
}
```

输出结果如下：

```go
main()函数中 a = 10
sum() 函数中 a = 10
sum() 函数中 b = 20
main()函数中 c = 30
```



# Go语言数组

数组是**具有相同唯一类型**的一组已编号且**长度固定的数据项序列**，这种类型可以是任意的原始类型例如整型、字符串或者自定义类型

数组元素可以通过索引（位置）来读取（或者修改），索引从 0 开始，第一个元素索引为 0，第二个索引为 1，以此类推

![截屏2023-11-18 11.31.47](/Users/yangchaoran/Library/Application Support/typora-user-images/截屏2023-11-18 11.31.47.png)

## 数组声明

数组声明需要指定元素类型及元素个数，如以下定义了数组 balance 长度为 10 类型为 float32：

```go
var balance [10]float32
```

## 数组初始化

可以使用初始化列表来初始化数组的元素，如以上代码声明一个大小为 5 的整数数组，并将其中的元素分别初始化为 1、2、3、4 和 5。

```go
var numbers = [5]int{1, 2, 3, 4, 5}
```

以下演示了数组完整操作（声明、赋值、访问）的实例：

```go
package main

import "fmt"

func main() {
   var n [10]int /* n 是一个长度为 10 的数组 */
   var i,j int

   /* 为数组 n 初始化元素 */         
   for i = 0; i < 10; i++ {
      n[i] = i + 100 /* 设置元素为 i + 100 */
   }

   /* 输出每个数组元素的值 */
   for j = 0; j < 10; j++ {
      fmt.Printf("Element[%d] = %d\n", j, n[j] )
   }
}
```

[向函数传递数组](https://www.runoob.com/go/go-passing-arrays-to-functions.html)

# Go指针

变量是一种使用方便的占位符，用于引用计算机内存地址，Go 语言的取地址符是 **&**，放到一个变量前使用就会返回相应变量的内存地址。

## 什么是指针

一个指针变量指向了一个值的内存地址，指针声明格式如下：

```go
var var_name *var-type
```

var-type 为指针类型，var_name 为指针变量名，* 号用于指定变量是作为一个指针。以下是有效的指针声明：

```go
var ip *int        /* 指向整型*/
var fp *float32    /* 指向浮点型 */
```

在**指针类型前面加上 * 号**（前缀）来获取指针所指向的内容

```go
package main

import "fmt"

func main() {
   var a int= 20   /* 声明实际变量 */
   var ip *int        /* 声明指针变量 */

   ip = &a  /* 指针变量的存储地址 */

   fmt.Printf("a 变量的地址是: %x\n", &a  )

   /* 指针变量的存储地址 */
   fmt.Printf("ip 变量储存的指针地址: %x\n", ip )

   /* 使用指针访问值 */
   fmt.Printf("*ip 变量的值: %d\n", *ip )
}
```

## 空指针

一个指针被定义后没有分配到任何变量时，它的值为 nil，也称为空指针。

一个指针变量通常**缩写为 ptr**

空指针判断

```go
if(ptr != nil)     /* ptr 不是空指针 */
if(ptr == nil)    /* ptr 是空指针 */
```

## 指向指针的指针

当定义一个指向指针的指针变量时，第一个指针存放第二个指针的地址，第二个指针存放变量的地址

访问指向指针的指针变量值需要使用两个 * 号，如下所示：

```go
package main

import "fmt"

func main() {

   var a int
   var ptr *int
   var pptr **int

   a = 3000

   /* 指针 ptr 地址 */
   ptr = &a

   /* 指向指针 ptr 地址 */
   pptr = &ptr

   /* 获取 pptr 的值 */
   fmt.Printf("变量 a = %d\n", a )
   fmt.Printf("指针变量 *ptr = %d\n", *ptr )
   fmt.Printf("指向指针的指针变量 **pptr = %d\n", **pptr)
}
```

# Go结构体

数组可以存储同一类型的数据，但在结构体中我们可以为不同项定义不同的数据类型

结构体是由一系列具有相同类型或不同类型的数据构成的数据集合

结构体定义需要**使用 type 和 struct** 语句。struct 语句定义一个新的数据类型，结构体中有一个或多个成员。type 语句设定了结构体的名称

```go
package main

import "fmt"

type Books struct {
   title string
   author string
   subject string
   book_id int
}


func main() {

    // 创建一个新的结构体
    fmt.Println(Books{"Go 语言", "www.runoob.com", "Go 语言教程", 6495407})

    // 也可以使用 key => value 格式
    fmt.Println(Books{title: "Go 语言", author: "www.runoob.com", subject: "Go 语言教程", book_id: 6495407})

    // 忽略的字段为 0 或 空
   fmt.Println(Books{title: "Go 语言", author: "www.runoob.com"})
}
```

如果要访问结构体成员，需要使用点号 **.** 操作符，格式为：结构体.成员名

# Go语言切片

切片是对数组的抽象，与数组相比**切片的长度是不固定的**，**可以追加元素**，在追加时可能使切片的容量增大

## 定义切片

可以声明一个未指定大小的数组来定义切片（切片不需要说明长度）：

```go
var identifier []type
```

或使用 **make()** 函数来创建切片:

```go
var slice1 []type = make([]type, len)

也可以简写为

slice1 := make([]type, len)
```

也可以指定容量，其中 **cap** 为可选参数，len是数组的长度并且也是**切片的初始长度**

```go
make([]T, length, capacity)
```

## len() 和 cap() 函数

切片是可索引的，并且可以由 **len() 方法获取长度**。

切片提供了计算容量的方法 **cap() 可以测量切片最长可以达到多少**。

## 切片截取

通过设置下限及上限来设置截取切片 **[lower-bound:upper-bound]（左闭右开）**

```go
package main

import "fmt"

func main() {
   /* 创建切片 */
   numbers := []int{0,1,2,3,4,5,6,7,8}   
   printSlice(numbers)

   /* 打印原始切片 */
   fmt.Println("numbers ==", numbers)

   /* 打印子切片从索引1(包含) 到索引4(不包含)*/
   fmt.Println("numbers[1:4] ==", numbers[1:4])

   /* 默认下限为 0*/
   fmt.Println("numbers[:3] ==", numbers[:3])

   /* 默认上限为 len(s)*/
   fmt.Println("numbers[4:] ==", numbers[4:])

   numbers1 := make([]int,0,5)
   printSlice(numbers1)

   /* 打印子切片从索引  0(包含) 到索引 2(不包含) */
   number2 := numbers[:2]
   printSlice(number2)

   /* 打印子切片从索引 2(包含) 到索引 5(不包含) */
   number3 := numbers[2:5]
   printSlice(number3)

}

func printSlice(x []int){
   fmt.Printf("len=%d cap=%d slice=%v\n",len(x),cap(x),x)
}
```

```go
len=9 cap=9 slice=[0 1 2 3 4 5 6 7 8]
numbers == [0 1 2 3 4 5 6 7 8]
numbers[1:4] == [1 2 3]
numbers[:3] == [0 1 2]
numbers[4:] == [4 5 6 7 8]
len=0 cap=5 slice=[]
len=2 cap=9 slice=[0 1]
len=3 cap=7 slice=[2 3 4]
```

## append() 和 copy() 函数

如果想增加切片的容量，我们必须创建一个新的更大的切片并把原分片的内容都拷贝过来。

下面的代码描述了从拷贝切片的 copy 方法和向切片追加新元素的 append 方法。

```go
package main

import "fmt"

func main() {
   var numbers []int
   printSlice(numbers)

   /* 允许追加空切片 */
   numbers = append(numbers, 0)
   printSlice(numbers)

   /* 向切片添加一个元素 */
   numbers = append(numbers, 1)
   printSlice(numbers)

   /* 同时添加多个元素 */
   numbers = append(numbers, 2,3,4)
   printSlice(numbers)

   /* 创建切片 numbers1 是之前切片的两倍容量*/
   numbers1 := make([]int, len(numbers), (cap(numbers))*2)

   /* 拷贝 numbers 的内容到 numbers1 */
   copy(numbers1,numbers)
   printSlice(numbers1)   
}

func printSlice(x []int){
   fmt.Printf("len=%d cap=%d slice=%v\n",len(x),cap(x),x)
}
```

```go
len=0 cap=0 slice=[]
len=1 cap=1 slice=[0]
len=2 cap=2 slice=[0 1]
len=5 cap=6 slice=[0 1 2 3 4]
len=5 cap=12 slice=[0 1 2 3 4]
```

# Go 语言范围

 range 关键字用于 for 循环中迭代数组(array)、切片(slice)、通道(channel)或集合(map)的元素。**在数组和切片中它返回元素的索引和索引对应的值，在集合中返回 key-value 对**

# Go 语言Map(集合)

Map 是一种无序的键值对的集合

Map 是一种集合，所以我们可以像迭代数组和切片那样迭代它。不过，Map 是无序的，遍历 Map 时返回的键值对的顺序是不确定的

在获取 Map 的值时，如果键不存在，返回该类型的零值，例如 int 类型的零值是 0，string 类型的零值是 ""

对 Map 的修改会**影响到所有引用它的变量**。

Map 的**容量是指 Map 中可以保存的键值对的数量**，当 Map 中的键值对数量达到容量时，Map 会自动扩容。如果不指定 initialCapacity，Go 语言会根据实际情况选择一个合适的值。

```go
// 创建一个空的 Map
m := make(map[string]int)

// 创建一个初始容量为 10 的 Map
m := make(map[string]int, 10)
```

对于删除集合中的元素，可以用delete() 函数, 参数为 map 和其对应的 key

```go
package main

import "fmt"

func main() {
        /* 创建map */
        countryCapitalMap := map[string]string{"France": "Paris", "Italy": "Rome", "Japan": "Tokyo", "India": "New delhi"}

        fmt.Println("原始地图")

        /* 打印地图 */
        for country := range countryCapitalMap {
                fmt.Println(country, "首都是", countryCapitalMap [ country ])
        }

        /*删除元素*/ delete(countryCapitalMap, "France")
        fmt.Println("法国条目被删除")

        fmt.Println("删除元素后地图")

        /*打印地图*/
        for country := range countryCapitalMap {
                fmt.Println(country, "首都是", countryCapitalMap [ country ])
        }
}
```

# Go 语言递归函数

递归，就是在运行的过程中调用自己

Go 语言支持递归。但我们在使用递归时，开发者需要设置退出条件，否则递归将陷入无限循环中。

递归函数对于解决数学上的问题是非常有用的，就像计算阶乘，生成斐波那契数列等。

go语言实现斐波那契数列

```go
package main

import "fmt"

func fibonacci(n int) int {
  if n < 2 {
   return n
  }
  return fibonacci(n-2) + fibonacci(n-1)
}

func main() {
    var i int
    for i = 0; i < 10; i++ {
       fmt.Printf("%d\t", fibonacci(i))
    }
}
```

# Go 语言类型转换

[各种类型的转换](https://www.runoob.com/go/go-type-casting.html)

## 注意

```go
var str string = "10"
var num int
num, _ = strconv.Atoi(str)
```

**strconv.Atoi()** 函数的作用是将字符串转换为相应的整数值，其将返回两个值，第一个是转换后的整型值，第二个是可能发生的错误（**如果转换失败，则返回一个非nil的错误值**），我们可以使用空白标识符 **_** 来忽略这个错误

而**strconv.Itoa()**则是将证书转换为字符串

**strconv.ParseFloat()**将字符串转换为浮点数

**strconv.FormatFloat()**将浮点数转换为字符串

## 接口类型转换

接口类型转换有两种情况**：类型断言**和**类型转换**。

类型断言用于将接口类型转换为指定类型，类型转换用于将一个接口类型的值转换为另一个接口类型

# Go 语言接口

接口是一种数据类型，它将**所有的具有共性的方法定义在一起**

接口可以让我们**将不同的类型绑定到一组公共的方法**上，从而**实现多态和灵活的设计**。

Go 语言中的接口是**隐式实现**的，也就是说，如果一个类型实现了一个接口定义的所有方法，那么它就自动地实现了该接口。

[实例](https://www.runoob.com/go/go-interfaces.html)

# Go 错误处理

Go 语言通过内置的错误接口提供了非常简单的错误处理机制

函数通常在最后的返回值中返回错误信息。**使用 errors.New 可返回一个错误信息**：

# Go 并发

## 并发的定义

我们只需要通过 go 关键字来开启 goroutine 即可，goroutine 是**轻量级线程**，**goroutine 的调度是由 Golang 运行时进行管理的**

"并发"是指**同时执行多个任务的能力**，并发编程的目标是提高程序的执行效率，**同时处理多个任务，以实现更快的响应和更高的吞吐量**。

在并发编程中，可以使用**多个goroutine（轻量级线程）**来同时执行不同的任务，并且可以**使用channel（通道）来同步和通信这些goroutine之间的数据**

goroutine 语法格式：

```go
go 函数名( 参数列表 )
```

Go 允许使用 go 语句开启一个新的运行期线程， 即 goroutine，以一个不同的、新创建的 goroutine 来执行一个函数。 **同一个程序中的所有 goroutine 共享同一个地址空间**。

## 通道（channel）

通道（channel）是用来**传递数据的一个数据结构**

通道可用于两个 goroutine 之间通过传递一个指定类型的值来同步运行和通讯。操作符 `<-` 用于指定通道的方向，发送或接收。如果未指定方向，则为双向通道

```go
ch <- v    // 把 v 发送到通道 ch
v := <-ch  // 从 ch 接收数据
           // 并把值赋给 v
```

声明一个通道很简单，我们使用chan关键字即可，通道在使用前必须先创建：

```go
ch := make(chan int)
```

以下实例通过两个 goroutine 来计算数字之和，在 goroutine 完成计算后，它会计算两个结果的和：

```go
package main

import "fmt"

func sum(s []int, c chan int) {
        sum := 0
        for _, v := range s {
                sum += v
        }
        c <- sum // 把 sum 发送到通道 c
}

func main() {
        s := []int{7, 2, 8, -9, 4, 0}

        c := make(chan int)
        go sum(s[:len(s)/2], c)
        go sum(s[len(s)/2:], c)
        x, y := <-c, <-c // 从通道 c 中接收

        fmt.Println(x, y, x+y)
}
```

这段代码是一个简单的 Go 程序，它计算一个整数切片的和并将两个部分的结果打印出来。下面是对每一部分的解释：

1. **导入包**:

```go
go复制代码

import "fmt"
```

这行导入了 Go 的 `fmt` 包，用于格式化输出。

2. **函数 sum**:

```go
func sum(s []int, c chan int) {  
    sum := 0  
    for _, v := range s {  
        sum += v  
    }  
    c <- sum // 把 sum 发送到通道 c  
}
```

这是一个并发的函数，它接收一个整数切片和一个通道作为参数。它计算切片的和并将结果发送到提供的通道。

3. **主函数 main**:

```go
func main() {  
    s := []int{7, 2, 8, -9, 4, 0}  
    c := make(chan int)  
    go sum(s[:len(s)/2], c)  
    go sum(s[len(s)/2:], c)  
    x, y := <-c, <-c // 从通道 c 中接收  
    fmt.Println(x, y, x+y)  
}
```

在主函数中：

- 初始化一个整数切片 `s`。
- 创建一个整数通道 `c`。
- 使用 `go` 关键字并发地调用 `sum` 函数两次，分别计算切片的前半部分和后半部分的和，并将结果发送到通道 `c`。
- 使用 `<-` 从通道 `c` 中接收两个值并分别赋值给 `x` 和 `y`。由于 `go` 的并发执行，`x` 和 `y` 会几乎同时接收到两个部分的和。
- 使用 `fmt.Println` 打印 `x`、`y` 和它们的和。

4. **输出**:
   由于切片的元素是随机顺序的，因此每次运行此程序时，输出的结果可能会有所不同。但无论如何，输出应该是两个部分和的总和。例如，可能的输出为：-5 17 12
