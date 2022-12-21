#  Go 流程控制

![](https://img-blog.csdnimg.cn/e5b2a7fc8f6f480793a607e6d4756603.png)


Go中流程控制分三大类：`条件判断，循环控制和无条件跳转`。

## 1. if

if也许是各种编程语言中最常见的了，它的语法概括起来就是：如果满足条件就做某事，否则做另一件事。

Go里面if条件判断语句中不需要括号，如下代码所示

```bash
if x > 10 {
	fmt.Println("x is greater than 10")
} else {
	fmt.Println("x is less than 10")
}
```

Go的if还有一个强大的地方就是条件判断语句里面允许声明一个变量，这个变量的作用域只能在该条件逻辑块内，其他地方就不起作用了，如下所示

```bash
// 计算获取值x,然后根据x返回的大小，判断是否大于10。
if x := computedValue(); x > 10 {
	fmt.Println("x is greater than 10")
} else {
	fmt.Println("x is less than 10")
}
```

//这个地方如果这样调用就编译出错了，因为x是条件里面的变量
fmt.Println(x)
多个条件的时候如下所示：

```bash
if integer == 3 {
	fmt.Println("The integer is equal to 3")
} else if integer < 3 {
	fmt.Println("The integer is less than 3")
} else {
	fmt.Println("The integer is greater than 3")
}
```

## 2.goto

Go有goto语句——请明智地使用它。用goto跳转到必须在当前函数内定义的标签。例如假设这样一个循环：

```bash
func myFunc() {
	i := 0
Here:   //这行的第一个词，以冒号结束作为标签
	println(i)
	i++
	goto Here   //跳转到Here去
}
```

**标签名是大小写敏感的。**

## 3.for

Go里面最强大的一个控制逻辑就是for，它既可以用来循环读取数据，又可以当作while来控制逻辑，还能迭代操作。它的语法如下：

```bash
for expression1; expression2; expression3 {
	//...
}
```

expression1、expression2和expression3都是表达式，其中expression1和expression3是变量声明或者函数调用返回值之类的，expression2是用来条件判断，expression1在循环开始之前调用，expression3在每轮循环结束之时调用。

一个例子比上面讲那么多更有用，那么我们看看下面的例子吧：

 for1.go
```bash
package main

import "fmt"

func main(){
	sum := 0;
	for index:=0; index < 10 ; index++ {
		sum += index
	}
	fmt.Println("sum is equal to ", sum)
}
// 输出：sum is equal to 45
```

有些时候需要进行多个赋值操作，由于Go里面没有,操作符，那么可以使用平行赋值`i, j = i+1, j-1`



```bash
sum := 1
for ; sum < 1000;  {
	sum += sum
}
```

其中;也可以省略，那么就变成如下的代码了，是不是似曾相识？对，这就是while的功能。

```bash
sum := 1
for sum < 1000 {
	sum += sum
}
```

无限循环

```bash
for {
    // ...
}
```

这就变成一个无限循环，尽管如此，还可以用其他方式终止循环, 如一条`break`或`return`语句。
在循环里面有两个关键操作`break`和continue ,break操作是跳出当前循环，continue是跳过本次循环。当嵌套过深的时候，break可以配合标签使用，即跳转至标签所指定的位置，详细参考如下例子：

```bash
for index := 10; index>0; index-- {
	if index == 5{
		break // 或者continue
	}
	fmt.Println(index)
}
// break打印出来10、9、8、7、6
// continue打印出来10、9、8、7、6、4、3、2、1
```

break和continue还可以跟着标号，用来跳到多重循环中的外层循环

for配合`range`可以用于读取slice和map的数据：

```bash
for k,v:=range map {
	fmt.Println("map's key:",k)
	fmt.Println("map's val:",v)
}
```

由于 Go 支持 “多值返回”, 而对于“声明而未被调用”的变量, 编译器会报错, 在这种情况下, 可以使用_来丢弃不需要的返回值 例如

```bash
for _, v := range map{
	fmt.Println("map's val:", v)
}
```
### 3.1 for 与 range 合用
 for2.go

```bash
package main
import "fmt"
func main() {
   var b int = 15
   var a int
   numbers := [6]int{1, 2, 3, 5}
   /* for 循环 */
   for a := 0; a < 10; a++ {
      fmt.Printf("a 的值为: %d\n", a)
   }
   for a < b {
      a++
      fmt.Printf("a 的值为: %d\n", a)
      }
   for i,x:= range numbers {
      fmt.Printf("第 %d 位 x 的值 = %d\n", i,x)
   }   
}
```

```bash
[root@localhost for]# go run for1.go 
a 的值为: 0
a 的值为: 1
a 的值为: 2
a 的值为: 3
a 的值为: 4
a 的值为: 5
a 的值为: 6
a 的值为: 7
a 的值为: 8
a 的值为: 9
a 的值为: 1
a 的值为: 2
a 的值为: 3
a 的值为: 4
a 的值为: 5
a 的值为: 6
a 的值为: 7
a 的值为: 8
a 的值为: 9
a 的值为: 10
a 的值为: 11
a 的值为: 12
a 的值为: 13
a 的值为: 14
a 的值为: 15
第 0 位 x 的值 = 1
第 1 位 x 的值 = 2
第 2 位 x 的值 = 3
第 3 位 x 的值 = 5
第 4 位 x 的值 = 0
第 5 位 x 的值 = 0
```

### 3.2 利用函数作为条件判断
 for3.go


```bash
package main
func length(s string) int {
    println("call length.")
    return len(s)
}
func main() {
    s := "abcd"
    for i, n := 0, length(s); i < n; i++ {     // 避免多次调用 length 函数。
        println(i, s[i])
    } 
}
```
输出:

```bash
call length.
0 97
1 98
2 99
3 100
```

## 4. switch

有些时候你需要写很多的if-else来实现一些逻辑处理，这个时候代码看上去就很丑很冗长，而且也不易于以后的维护，这个时候switch就能很好的解决这个问题。
它的语法如下：

```bash
switch sExpr {
case expr1:
	some instructions
case expr2:
	some other instructions
case expr3:
	some other instructions
default:
	other code
}
```

sExpr和expr1、expr2、expr3的类型必须一致。Go的switch非常灵活，表达式不必是常量或整数，执行的过程从上至下，直到找到匹配项；而如果switch没有表达式，它会匹配true。
sw1.go
```bash
cat sw1.go
package main
import "fmt"
func main() {
i := 10
switch  {                              //第一种表达
case i == 1:
        fmt.Println("i is equal to 1")
case i == 2:
        fmt.Println("i is equal to 2")
case i == 10:
        fmt.Println("i is equal to 10")
default:
        fmt.Println("All I know is that i is an integer")
}

switch i {                              //第二种表达
case 1:
	fmt.Println("i is equal to 1")
case 2, 3, 4:
	fmt.Println("i is equal to 2, 3 or 4")
case 10:
	fmt.Println("i is equal to 10")
default:
	fmt.Println("All I know is that i is an integer")
}

switch i := 2; {                       //第三种表达
case i == 1:
        fmt.Println("i is equal to 1")
case i == 2:
        fmt.Println("i is equal to 2")
case i == 10:
        fmt.Println("i is equal to 10")
default:
        fmt.Println("All I know is that i is an integer")
}


}
[root@localhost switch]# go run sw1.go 
i is equal to 10
i is equal to 10
i is equal to 2


```

在第5行中，我们把很多值聚合在了一个case里面，同时，Go里面switch默认相当于每个case最后带有break，匹配成功后不会自动向下执行其他case，而是跳出整个switch, 但是可以使用**`fallthrough`**强制执行后面的case代码。
 sw2.go
```bash
integer := 6
switch integer {
case 4:
	fmt.Println("The integer was <= 4")
	fallthrough
case 5:
	fmt.Println("The integer was <= 5")
	fallthrough
case 6:
	fmt.Println("The integer was <= 6")
	fallthrough
case 7:
	fmt.Println("The integer was <= 7")
	fallthrough
case 8:
	fmt.Println("The integer was <= 8")
	fallthrough
default:
	fmt.Println("default case")
}
上面的程序将输出

The integer was <= 6
The integer was <= 7
The integer was <= 8
default case
```
变量 expr1 可以是任何类型，而expr1 和 exprl2 则可以是同类型的任意值。类型不被局限于常量或整数，但必须是相同的类型；或者最终结果为相同类型的表达式。

实例:
 sw3.go
```bash
package main
import "fmt"
func main() {
   /* 定义局部变量 */
   var grade string = "B"
   var marks int = 90
   switch marks {
      case 90: grade = "A"
      case 80: grade = "B"
      case 50,60,70 : grade = "C"
      default: grade = "D"  
   }
   switch {
      case grade == "A" :
         fmt.Printf("优秀!\n" )     
      case grade == "B", grade == "C" :
         fmt.Printf("良好\n" )      
      case grade == "D" :
         fmt.Printf("及格\n" )      
      case grade == "F":
         fmt.Printf("不及格\n" )
      default:
         fmt.Printf("差\n" )
   }
   fmt.Printf("你的等级是 %s\n", grade )
}
```

以上代码执行结果为：

```bash
优秀!

你的等级是 A
```


witch 语句还可以被用于 type-switch 来判断某个 interface 变量中实际存储的变量类型。

Type Switch 语法格式如下：

```bash
switch x.(type){
    case type:
       statement(s)      
    case type:
       statement(s)
    /* 你可以定义任意个数的case */
    default: /* 可选 */
       statement(s)
}
```
sw4.go
```bash
package main
import "fmt"
func main() {
    var x interface{}
    //写法一：
    switch i := x.(type) { // 带初始化语句
        case nil:
            fmt.Printf(" x 的类型 :%T\r\n", i)
        case int:
            fmt.Printf("x 是 int 型")
        case float64:
            fmt.Printf("x 是 float64 型")
        case func(int) float64:
            fmt.Printf("x 是 func(int) 型")
        case bool, string:
            fmt.Printf("x 是 bool 或 string 型")
        default:
            fmt.Printf("未知型")
    }
    //写法二
    var j = 0
    switch j {
        case 0:
        case 1:
            fmt.Println("1")
        case 2:
            fmt.Println("2")
        default:
            fmt.Println("def")
    }
    //写法三
    var k = 0
    switch k {
        case 0:
            println("fallthrough")
            fallthrough
            /*
                Go的switch非常灵活，表达式不必是常量或整数，执行的过程从上至下，直到找到匹配项；
                而如果switch没有表达式，它会匹配true。
                Go里面switch默认相当于每个case最后带有break，
                匹配成功后不会自动向下执行其他case，而是跳出整个switch,
                但是可以使用fallthrough强制执行后面的case代码。
            */
        case 1:
            fmt.Println("1")
        case 2:
            fmt.Println("2")
        default:
            fmt.Println("def")
    }
    //写法三
    var m = 0
    switch m {
        case 0, 1:
            fmt.Println("1")
        case 2:
            fmt.Println("2")
        default:
            fmt.Println("def")
    }
    //写法四
    var n = 0
    switch { //省略条件表达式，可当 if...else if...else
        case n > 0 && n < 10:
            fmt.Println("i > 0 and i < 10")
        case n > 10 && n < 20:
            fmt.Println("i > 10 and i < 20")
        default:
            fmt.Println("def")
    }
}
```

以上代码执行结果为：

```bash
x 的类型 :

fallthrough

1

1

def
```

## 5.select
select 语句类似于 switch 语句，但是select会随机执行一个可运行的case。如果没有case可运行，它将阻塞，直到有case可运行。

select 是Go中的一个控制结构，类似于用于通信的switch语句。每个case必须是一个通信操作，要么是发送要么是接收。

select 随机执行一个可运行的`case`。如果没有case可运行，它将阻塞，直到有case可运行。一个默认的子句应该总是可运行的。

Go 编程语言中 select 语句的语法如下：

```bash
select {
    case communication clause  :
       statement(s);      
    case communication clause  :
       statement(s);
    /* 你可以定义任意数量的 case */
    default : /* 可选 */
       statement(s);
}
```

以下描述了 select 语句的语法：

每个case都必须是一个通信
所有channel表达式都会被求值
所有被发送的表达式都会被求值
如果任意某个通信可以进行，它就执行；其他被忽略。
如果有多个case都可以运行，Select会随机公平地选出一个执行。其他不会执行。
否则：
如果有default子句，则执行该语句。
如果没有default字句，select将阻塞，直到某个通信可以运行；Go不会重新对channel或值进行求值。
实例：
select1.go
```bash
package main
import "fmt"
func main() {
   var c1, c2, c3 chan int
   var i1, i2 int
   select {
      case i1 = <-c1:
         fmt.Printf("received ", i1, " from c1\n")
      case c2 <- i2:
         fmt.Printf("sent ", i2, " to c2\n")
      case i3, ok := (<-c3):  // same as: i3, ok := <-c3
         if ok {
            fmt.Printf("received ", i3, " from c3\n")
         } else {
            fmt.Printf("c3 is closed\n")
         }
      default:
         fmt.Printf("no communication\n")
   }    
}
```

以上代码执行结果为：

```bash
no communication
```

select可以监听`channel`的数据流动 select的用法与switch语法非常类似，由select开始的一个新的选择块，每个选择条件由case语句来描述

与switch语句可以选择任何使用相等比较的条件相比，select由比较多的限制，其中最大的一条限制就是每个case语句里必须是一个IO操作。

```bash
select { //不停的在这里检测
    case <-chanl : //检测有没有数据可以读
    //如果chanl成功读取到数据，则进行该case处理语句
    case chan2 <- 1 : //检测有没有可以写
    //如果成功向chan2写入数据，则进行该case处理语句
    //假如没有default，那么在以上两个条件都不成立的情况下，就会在此阻塞//一般default会不写在里面，select中的default子句总是可运行的，因为会很消耗CPU资源
    default:
    //如果以上都没有符合条件，那么则进行default处理流程
}
```

在一个select语句中，Go会按顺序从头到尾评估每一个发送和接收的语句。

如果其中的任意一个语句可以继续执行（即没有被阻塞），那么就从那些可以执行的语句中任意选择一条来使用。

如果没有任意一条语句可以执行（即所有的通道都被阻塞），那么有两种可能的情况：

如果给出了default语句，那么就会执行default的流程，同时程序的执行会从select语句后的语句中恢复。

如果没有default语句，那么select语句将被阻塞，直到至少有一个case可以进行下去。

Golang select的使用及典型用法

### 5.1 select基本使用
select是Go中的一个控制结构，类似于switch语句，用于处理异步IO操作。select会监听case语句中channel的读写操作，当case中channel读写操作为非阻塞状态（即能读写）时，将会触发相应的动作。

select中的case语句必须是一个channel操作

select中的default子句总是可运行的。

如果有多个case都可以运行，select会随机公平地选出一个执行，其他不会执行。

如果没有可运行的case语句，且有default语句，那么就会执行default的动作。

如果没有可运行的case语句，且没有default语句，select将阻塞，直到某个case通信可以运行。

例如：
select2.go
```bash
package main
import "fmt"
func main() {
   var c1, c2, c3 chan int
   var i1, i2 int
   select {
      case i1 = <-c1:
         fmt.Printf("received ", i1, " from c1\n")
      case c2 <- i2:
         fmt.Printf("sent ", i2, " to c2\n")
      case i3, ok := (<-c3):  // same as: i3, ok := <-c3
         if ok {
            fmt.Printf("received ", i3, " from c3\n")
         } else {
            fmt.Printf("c3 is closed\n")
         }
      default:
         fmt.Printf("no communication\n")
   }    
}
//输出：
no communication
```

### 5.2 典型用法
**超时判断**
//比如在下面的场景中，使用全局resChan来接受response，如果时间超过3S,resChan中还没有数据返回，则第二条case将执行

select3.go
```bash
var resChan = make(chan int)
// do request
func test() {
    select {
    case data := <-resChan:
        doData(data)
    case <-time.After(time.Second * 3):
        fmt.Println("request time out")
    }
}
func doData(data int) {
    //...
}
```

退出
//主线程（协程）中如下：

select4.go
```bash
var shouldQuit=make(chan struct{})
fun main(){
    {
        //loop
    }
    //...out of the loop
    select {
        case <-c.shouldQuit:
            cleanUp()
            return
        default:
        }
    //...
}
```

//再另外一个协程中，如果运行遇到非法操作或不可处理的错误，就向shouldQuit发送数据通知程序停止运行

```bash
close(shouldQuit)
```

判断channel是否阻塞
//在某些情况下是存在不希望channel缓存满了的需求的，可以用如下方法判断

select5.go
```bash
ch := make (chan int, 5)
//...
data：=0
select {
    case ch <- data:
    default:
    //做相应操作，比如丢弃data。视需求而定
}
```


参考：
- [https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.3.md](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.3.md)
- [http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch3.md?wd=goto](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch3.md?wd=goto)
