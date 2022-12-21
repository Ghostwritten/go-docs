#  Go 了解接口运用

![](https://img-blog.csdnimg.cn/49a6fc6b6494477c8506974329728f12.png)

## 1. 接口定义

Go 语言中的接口很特别，而且提供了难以置信的一系列灵活性和抽象性。指定一个特定类型的值和指针表现为特定的方式。从语言角度看，接口是一种类型，它指定一个方法集，所有方法为接口类型就被认为是该接口。

interface类型可以定义一组方法，但是这些不需要实现。并且interface不能包含任何变量。

interface类型默认是一个指针

 **interface语法：**

```go
type example interface{
        Method1(参数列表) 返回值列表
        Method2(参数列表) 返回值列表
        …
}
var a example
a.Method1()
```

**实际定义一个接口：**

```go
type Shape interface {
    Area() float32
}
```
上面的代码定义了接口类型 `Shape`，接口中包含了一个不带参数、返回值为 `float32`  的方法 `Area()`。任何实现了方法 Area() 的类型 T，我们就说它实现了接口 Shape。
**inter1.go**
```go
type Shape interface {
	Area() float32
}

func main() {
	var s Shape
	fmt.Println("value of s is", s)
	fmt.Printf("type of s is %T\n", s)
}
```

输出：

```go
value of s is <nil>
type of s is <nil>
```

上面的代码，由于接口是一种类型，所以可以创建 Shape 类型的变量 s，你是不是很疑惑 s 的类型为什么是 nil？让我们来看下一节！

## 2. 接口类型值
**第一种解释**
**变量的类型在声明时指定、且不能改变，称为静态类型。接口类型的静态类型就是接口本身。接口没有静态值，它指向的是动态值。接口类型的变量存的是实现接口的类型的值。该值就是接口的动态值，实现接口的类型就是接口的动态类型**。

**第二种解释
Golang 接口执行机制 ：接口对象由接口表 (interface table) 指针和数据指针组成。**

 **inter2.go**
```go
type Iname interface {
	Mname()
}

type St1 struct {}
func (St1) Mname() {}
type St2 struct {}
func (St2) Mname() {}

func main() {
	var i Iname = St1{}
	fmt.Printf("type is %T\n",i)
	fmt.Printf("value is %v\n",i)
	i = St2{}
	fmt.Printf("type is %T\n",i)
	fmt.Printf("value is %v\n",i)
}
```

复制代码输出：

```go
type is main.St1
value is {}
type is main.St2
value is {}
```

变量 i 的静态类型是 Iname，是不能改变的。动态类型却是不固定的，第一次分配之后，i 的动态类型是 St1，第二次分配之后，i 的动态类型是 St2，动态值都是空结构体。
有时候，接口的动态类型又称为具体类型，当我们访问接口类型的时候，返回的是底层动态值的类型。

只能修改方法指针改变对象值

```go
package main
import "fmt"
type User struct {
    id   int
    name string
}
func main() {
    u := User{1, "Tom"}
    var i interface{} = u
    u.id = 2
    u.name = "Jack"        //使用T修改值
    fmt.Printf("%v\n", u)
    fmt.Printf("%v\n", i.(User))
}
```
输出结果:

```go
{2 Jack}
{1 Tom}
```

接口转型返回临时对象，只有使用指针才能修改其状态。

```go
package main
import "fmt"
type User struct {
    id   int
    name string
}
func main() {
    u := User{1, "Tom"}
    var vi, pi interface{} = u, &u
    // vi.(User).name = "Jack"
    // Error: cannot assign to vi.(User).name
    pi.(*User).name = "Jack"        //使用*T修改值
    fmt.Printf("%v\n", vi.(User))
    fmt.Printf("%v\n", pi.(*User))
}
```

输出结果:

```go
{1 Tom}
&{1 Jack}
```

## 3. nil 接口值
**inter3.go**
```go
type Iname interface {
	Mname()
}
type St struct {}
func (St) Mname() {}
func main() {
	var t *St
	if t == nil {
		fmt.Println("t is nil")
	} else {
		fmt.Println("t is not nil")
	}
	var i Iname = t
	fmt.Printf("%T\n", i)
	if i == nil {
		fmt.Println("i is nil")
	} else {
		fmt.Println("i is not nil")
	}
	fmt.Printf("i is nil pointer:%v",i == (*St)(nil))
}
```

输出：

```go
t is nil
*main.St
i is not nil
i is nil pointer:true
```

是不是很惊讶，我们分配给变量 i 的值明明是 nil，然而 i 却不是 nil。 来看下怎么回事！
动态类型在上面已经讲过，动态值是实际分配的值。记住一点：当且仅当动态值和动态类型都为 nil 时，接口类型值才为 nil。上面的代码，给变量 i  赋值之后，i 的动态值是 nil，但是动态类型却是 *St， i 是一个 nill 指针，所以相等条件不成立。
**看下 Go 语言规范：**

```go
 var x interface{} = struct()
 var x interface{}  // x is nil and has static type interface{}
 var v *T           // v has value nil, static type *T
 x = 42             // x has value 42 and dynamic type int
 x = v              // x has value (*T)(nil) and dynamic type *T
```

通过这一节学习，相信你已经很清楚为什么上一节的 Shape 类型的变量的 s 输出的类型是 nil，**因为 var s Shape 声明时，s 的动态类型是 nil**。

## 4. 实现接口
### 4.1 单接口单方法
inter4.go  
```go
type Shape interface {
	Area() float32
}

type Rect struct {
	width  float32
	height float32
}

func (r Rect) Area() float32 {
	return r.width * r.height
}

func main() {
	var s Shape
	s = Rect{5.0, 4.0}
	r := Rect{5.0, 4.0}
	fmt.Printf("type of s is %T\n", s)
	fmt.Printf("value of s is %v\n", s)
	fmt.Println("area of rectange s", s.Area())
	fmt.Println("s == r is", s == r)
}
```
输出：

```go

type of s is main.Rect
value of s is {5 4}
area of rectange s 20
s == r is true
```
创建了接口 Shape、结构体 Rect 以及方法 Area()。由于 Rect 实现了接口定义的所有方法，虽然只有一个，所以说 Rect 实现了接口 Shape。
在主函数里，创建了接口类型的变量 s ，值为 nil，并用 Rect 类型的结构体初始化，因为 Rect 结构体实现了接口，所以这是有效的。赋值之后，s 的动态类型变成了 Rect，动态值就是结构体的值 {5.0,4.0}。
可以直接使用 . 语法调用 Area() 方法，因为 s 的具体类型是 Rect，而 Rect 实现了 Area() 方法。
###  4.2 单节口多方法
**inter4.go**
```go
package main
import (
    "fmt"
)
type I interface {
    Method1() int
    Method2() string
}
type S struct {
    name string
    age  int
}
func (s S) Method1() int {
    return s.age
}
func (s S) Method2() string {
    return s.name
}
func main() {
    var user I = S{"Murphy", 28}
    age := user.Method1()
    name := user.Method2()
    fmt.Println(age, name)
}
```

输出结果：

```go
28 Murphy
```
### 4.3 多接口多方法
**inter5-1.go**
```go
type Shape interface {
	Area() float32
}

type Object interface {
	Perimeter() float32
}

type Circle struct {
	radius float32
}

func (c Circle) Area() float32 {
	return math.Pi * (c.radius * c.radius)
}

func (c Circle) Perimeter() float32 {
	return 2 * math.Pi * c.radius
}

func main() {
	c := Circle{3}
	var s Shape = c
	var p Object = c
	fmt.Println("area: ", s.Area())
	fmt.Println("perimeter: ", p.Perimeter())
}
输出：
area:  28.274334
perimeter:  18.849556
```

### 4.4 多接口多方法（嵌套接口）
**inter5-2.go**
```go
type Math interface {
	Shape
	Object
}
type Shape interface {
	Area() float32
}

type Object interface {
	Perimeter() float32
}

type Circle struct {
	radius float32
}

func (c Circle) Area() float32 {
	return math.Pi * (c.radius * c.radius)
}

func (c Circle) Perimeter() float32 {
	return 2 * math.Pi * c.radius
}

func main() {

	c := Circle{3}
	var m Math = c
	fmt.Printf("%T\n", m )
	fmt.Println("area: ", m.Area())
	fmt.Println("perimeter: ", m.Perimeter())
}
```
输出：

```go
main.Circle
area:  28.274334
perimeter:  18.849556
```



### 4.5 空接口
一个不包含任何方法的接口，称之为空接口，形如：interface{}。**因为空接口不包含任何方法，所以任何类型都默认实现了空接口。**
举个例子，**fmt 包中的 Println() 函数，可以接收多种类型的值，比如：int、string、array等。为什么，因为它的形参就是接口类型，可以接收任意类型的值。**
**inter6-1.go**  
第一种

```go
package main
import "fmt"
func Print(v interface{}) {
    fmt.Printf("%T: %v\n", v, v)
}
func main() {
    Print(1)
    Print("Hello, World!")
}
```
输出：

```go
int: 1
string: Hello, World!
```

**inter6-2.go**  
第二种
```go
func Println(a ...interface{}) (n int, err error) {}
```

```go
type MyString string
type Rect struct {
	width  float32
	height float32
}
func explain(i interface{}) {
	fmt.Printf("type of s is %T\n", i)
	fmt.Printf("value of s is %v\n\n", i)
}
func main() {
	ms := MyString("Seekload")
	r := Rect{5.0, 4.0}
	explain(ms)
	explain(r)
}
```
输出：

```go
type of s is main.MyString
value of s is Seekload

type of s is main.Rect
value of s is {5 4}
```
上面的代码，创建了自定义的字符串类型 MyString 、结构体 Rect 和 explain() 函数。explain() 函数的形参是空接口，所以可以接收任意类型的值。

## 5. 指针接收者和值接收者实现接口区别
**inter7.go**
```go
type Shape interface {
	Area() float32
}

type Circle struct {
	radius float32
}

type Square struct {
	side float32
}

func (c Circle) Area() float32 {
	return math.Pi * (c.radius * c.radius)
}

func (s *Square) Area() float32 {
	return s.side * s.side
}

func main() {
	var s Shape
	c1 := Circle{3}
	s = c1
	fmt.Printf("%v\n",s.Area())

	c2 := Circle{4}
	s = &c2
	fmt.Printf("%v\n",s.Area())

	c3 := Square{3}
	//s = c3
	s = &c3
	fmt.Printf("%v\n",s.Area())

}
```
输出：

```go
28.274334
50.265484
9
```
上面的代码，结构体 Circle 通过值接收者实现了接口 Shape。我们在方法那篇文章中已经讨论过了，值接收者的方法可以使用值或者指针调用，所以上面的 c1 和 c2 的调用方式是合法的。
结构体 Square 通过指针接收者实现了接口 Shape。如果将上方注释部分打开的话，编译就会出错：

```go
cannot use c3 (type Square) as type Shape in assignment:
Square does not implement Shape (Area method has pointer receiver)
```

从报错提示信息可以清楚看出，此时我们尝试将值类型 c3 分配给 s，但 c3 并没有实现接口 Shape。这可能会令我们有点惊讶，因为在方法中，我们可以直接通过值类型或者指针类型调用指针接收者方法。
**记住一点**：**对于指针接受者的方法，用一个指针或者一个可取得地址的值来调用都是合法的。但接口存储的具体值是不可寻址的，对于编译器无法自动获取 c3 的地址，于是程序报错。**

## 6. 匿名接口
*匿名接口可用作变量类型，或结构成员
**inter8.go**
```go
package main
import "fmt"
type Tester struct {
    s interface {
        String() string
    }
}
type User struct {
    id   int
    name string
}
func (self *User) String() string {
    return fmt.Sprintf("user %d, %s", self.id, self.name)
}
func main() {
    t := Tester{&User{1, "Tom"}}
    fmt.Println(t.s.String())
}
```
输出结果:

```go
user 1, Tom
```

## 7. 类型断言
一个 interface 被多种类型实现时，有时候我们需要区分 interface 的变量究竟存储哪种类型的值，**go 可以使用 comma, ok 的形式做区分 value, ok := em.(T)：em 是 interface 类型的变量，T代表要断言的类型，value 是 interface 变量存储的值，ok 是 bool 类型表示是否为该断言的类型 T。**

Go语言中使用接口断言（type assertions）将接口转换成另外一个接口，也可以将接口转换为另外的类型。接口的转换在开发中非常常见，使用也非常频繁

```go
if t, ok := i.(*S); ok {
    fmt.Println("s implements I", t)
}
```

ok 是 true 表明 i 存储的是 *S 类型的值，false 则不是，这种区分能力叫 Type assertions (类型断言)。

如果需要区分多种类型，可以使用 switch 断言，更简单直接，这种断言方式只能在 switch 语句中使用。
**inter9-1.go**

```go
switch t := i.(type) {
case *S:
    fmt.Println("i store *S", t)
case *R:
    fmt.Println("i store *R", t)
}
```

## 8. 类型转换
**inter10-1.go**
```go
var s int
var x interface

x = s
y , ok := x.(int)  //将interface 转为int,ok可省略 但是省略以后转换失败会报错，true转换成功，false转换失败, 并采用默认值
```
**inter10-2.go**
```go
package main

import "fmt"

func main()  {
    var x interface{}

    s := "WD"
    x = s
    y,ok := x.(int)
    z,ok1 := x.(string)
    fmt.Println(y,ok)
    fmt.Println(z,ok1)
}
//0 false
//WD true
```
 **inter10-3.go**
```go
package main

import "fmt"

type Student struct {
    Name string
}

func TestType(items ...interface{}) {
    for k, v := range items {
        switch v.(type) {
        case string:
        fmt.Printf("type is string, %d[%v]\n", k, v)
        case bool:
        fmt.Printf("type is bool, %d[%v]\n", k, v)
        case int:
        fmt.Printf("type is int, %d[%v]\n", k, v)
        case float32, float64:
        fmt.Printf("type is float, %d[%v]\n", k, v)
        case Student:
        fmt.Printf("type is Student, %d[%v]\n", k, v)
        case *Student:
        fmt.Printf("type is Student, %d[%p]\n", k, v)
        }
}
}

func main() {
 var stu Student
TestType("WD", 100, stu,3.3)
}
//type is string, 0[WD]
//type is int, 1[100]
//type is Student, 2[{}]
//type is float, 3[3.3]
```
参考：
- [http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch6.md](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch6.md)
- [https://juejin.im/post/5c9d5631f265da60d82dde9c](https://juejin.im/post/5c9d5631f265da60d82dde9c)
