# Go 指针应用实践

![在这里插入图片描述](https://img-blog.csdnimg.cn/e023bbf5ac824698b0d9cd7a2c795831.png)



## 1. 前言
指针概念在 Go 语言中被拆分为两个核心概念：  

 - 类型指针，允许对这个指针类型的数据进行修改。传递数据使用指针，而无须拷贝 数据。类型指针不能进行偏移和运算。

 

 - 切片，由指向起始元素的原始指针、元素数量和容量组成。

 受益于这样的约束和拆分， Go 语言的指针类型变量拥有指针的高效访问，但又不会 发生指针偏移，从而避免非法修改关键性数据问题。同时， 垃圾回收也比较容易对不会发 生偏移的指针进行检索和回收。 切片比原始指针具备更强大的特性， 更为安全。切片发生越界时，运行时会报出岩机，并打出堆枝，而原始指针只会崩溃。 要明白指针，需要知道几个概念：`指针地址、指针类型和指针取值`。
**每个变量在运行时都拥有一个地址，这个地址代表变量在内存中的位置。 Go 语言中 使用“＆” 操作符放在变量前面对变量进行“取地址”操作。** 
格式如下：
 ptr : = &v
  II v 的类型为 T 其中 v代表被取地址的变量，被取地址的V使用 阳变量进行接收，p仕的类型就为川T”， 称做 T 的指针类型。 “＊”代表指针

我们都知道，变量是一种使用方便的占位符，用于引用计算机内存地址。

Golang 支持指针类型 T，指针的指针 **T，以及包含包名前缀的 .T。

 - 默认值 nil，没有 NULL 常量。
 - 操作符 "&" （取地址符） 取变量地址
 - "*" （取值符）透过指针访问目标对象。
 - 不支持指针运算，不支持 "->" 运算符，直接用 "." 访问目标成员。

变量在内存中地址：

```go
package main
import "fmt"
func main(){
    var a int =10
    fmt.Printf("变量的地址: %x\n",&a)}
执行以上代码输出结果为：

变量的地址: c420012058
```

## 2. 什么是指针
一个指针变量可以指向任何一个值的内存地址，它指向那个值的内存地址。
类似于变量和常量，在使用指针前你需要声明指针。
指针声明格式如下：

```go
var name *类型
```
指针声明：

```go
package main
var ip *int     /* 指向整型*///声明一个int值得指针变量
var fp *float32 /* 指向浮点型 */
var sp *string  /* 指向字符串类型 */
func main(){}
```

## 3. 如何使用指针
指针使用流程：
定义指针变量。
为指针变量赋值。
访问指针变量中指向地址的值。
在指针类型前面加上 * 号（前缀）来获取指针所指向的内容。

```go
$ cat pointer.go
package main
import "fmt"
func main(){
    var a int =20/* 声明实际变量 */
    var ip *int    /* 声明指针变量 */
    ip =&a /* 指针变量的存储地址 */
    fmt.Printf("a 变量的地址是: %x\n",&a)/* 指针变量的存储地址 */
    fmt.Printf("ip 变量的存储地址: %x\n", ip)/* 使用指针访问值 */
    fmt.Printf("*ip 变量的值: %d\n",*ip)}
[root@localhost go]# go run pointer.go
a 变量的地址是: c000082010
ip 变量的存储地址: c000082010
*ip 变量的值: 20
```
直接用指针访问目标对象成员：

```go
package main
import ("fmt")
func main(){
    type data struct{ a int }
    var d = data{1234}
    var p *data
    p =&d
    fmt.Printf("%p, %v\n", p, p.a)// 直接用指针访问目标对象成员，无须转换。}
输出结果:

0xc420012058,1234
```
不能对指针做加减法等运算。

```go
package main
func main(){
    x :=1234
    p :=&x
    p++}
输出结果：

./main.go:6:3: invalid operation: p++(non-numeric type *int)
```
可以在 `unsafe.Pointer` 和任意类型指针间进 转换。

```go
cat pointer2.go
package main
import (
      "fmt" 
      "unsafe")
func main(){
    x :=0x12345678
    p := unsafe.Pointer(&x)// *int -> Pointer
    n :=(*[4]byte)(p)// Pointer -> *[4]byte
    for i :=0; i <len(n); i++{
        fmt.Printf("%X ", n[i])}
}
[root@localhost pointer]#  go run pointer2.go
78 56 34 12
```
将 Pointer 转换成 uintptr，可变相实现指针运算。

```go
$ cat pointer3.go
package main
import (
     "fmt"
     "unsafe"
)
func main(){
    d := struct {
        s string
        x int
    }{"abc",100}
    p :=uintptr(unsafe.Pointer(&d))   // *struct -> Pointer -> uintptr
    p += unsafe.Offsetof(d.x)         // uintptr + offset
    p2 := unsafe.Pointer(p)           // uintptr -> Pointer
    px :=(*int)(p2) 
    d.x = 200
    fmt.Printf("%#v\n", px)
    fmt.Printf("%#v\n", d)
}
[root@localhost pointer]#  go run pointer3.go
(*int)(0xc00006af30)
struct { s string; x int }{s:"abc", x:200}
```
注意:GC 把 uintptr 当成普通整数对象，它无法阻止 “关联” 对象被回收。

## 4. Go 空指针（nil）
当一个指针被定义后没有分配到任何变量时，它的值为 nil。

nil 指针也称为空指针。

nil在概念上和其它语言的null、None、nil、NULL一样，都指代零值或空值。

一个指针变量通常缩写为 ptr。

```go
$ cat pointer4.go
package main
import "fmt"
func main(){
    var ptr *int
    fmt.Printf("ptr 的值为 : %x\n", ptr)}
[root@localhost pointer]#  go run pointer4.go
ptr 的值为 : 0
```
空指针判断：

```go
$ cat pointer5.go
package main
import ("fmt")
func main(){
    var ptr1 *int
    var i int =1
    ptr2 :=&i
    if ptr1 == nil { 
        fmt.Println("prt1 是空指针")
    }
    if ptr2 != nil { 
        fmt.Println("prt2 不是空指针")
    }
}
[root@localhost pointer]# go run pointer5.go
prt1 是空指针
prt2 不是空指针
```

## 5. Go 指针数组
定义了长度为 3 的整型数组：
```go
package main
import "fmt"
const MAX int =3
func main(){
    a :=[MAX]int{10,100,200}
    var i int
    for i =0; i < MAX; i++{
        fmt.Printf("a[%d] = %d\n", i, a[i])}}
 
[root@localhost pointer]# go run pointer6.go
a[0] = 10
a[1] = 100
a[2] = 200

```
有一种情况，我们可能需要保存数组，这样我们就需要使用到指针。

以下声明了整型指针数组：

```go
var ptr [MAX]*int;
```

ptr 为整型指针数组。因此每个元素都指向了一个值。以下实例的三个整数将存储在指针数组中：

```go
$ cat pointer7.go
package main
import "fmt"
const MAX int =3
func main(){
    a :=[]int{9,99,999}
    var i int
    var ptr [MAX]*int
    for i =0; i < MAX; i++{
        ptr[i]=&a[i]/* 整数地址赋值给指针数组 */
    }
    for i =0; i < MAX; i++{
        fmt.Printf("a[%d] = %d\n", i,*ptr[i])
    }
}
[root@localhost pointer]# go run pointer7.go
a[0] = 9
a[1] = 99
a[2] = 999
```
参考：
- [http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.5.md](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.5.md)
