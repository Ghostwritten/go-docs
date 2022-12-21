#  Go slice 切片引用类型


![](https://img-blog.csdnimg.cn/c4a6516bc8e54b16b92fad32a30d6b37.png)


## 1. 前言 
Golang的引用类型包括 `slice、map` 和 `channel`。它们有复杂的内部结构，除了申请内存外，还需要初始化相关属性。

内置函数 `new` 计算类型大小，为其分配零值内存，返回指针。而 `make` 会被编译器翻译成具体的创建函数，由其分配内存和初始化成员结构，返回对象而非指针。

```bash
package main
func main() {
    a := []int{0, 0, 0} // 提供初始化表达式。
    a[1] = 10
    b := make([]int, 3) // make slice
    b[1] = 10
    c := new([]int)
    c[1] = 10 // ./main.go:11:3: invalid operation: c[1] (type *[]int does not support indexing)
}
```

变量存储的是一个地址，这个地址存储最终的值。内存通常在堆上分配。通过GC回收。

获取指针类型所指向的值，使用：” * “ 取值符号 。比如：var p int, 使用*p获取p指向的值

**`指针、slice、map、chan`**等都是**引用类型。**

## 2. new和make的区别

 - make 用来创建map、slice、channel
 - new 用来创建值类型

**new 和 make 均是用于分配内存：**

**new 用于值类型和用户定义的类型**，如自定义结构，make 用于内置引用类型（切片、map 和管道）。它们的用法就像是函数，但是将类型作为参数：new(type)、make(type)。new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针。它也可以被用于基本类型：v := new(int)。

make(T) 返回类型 T 的初始化之后的值，因此它比 new 进行更多的工作。new() 是一个函数，不要忘记它的括号。


## 3 切片特点

```bash
1. 切片：切片是数组的一个引用，因此切片是引用类型。但自身是结构体，值拷贝传递。
2. 切片的长度可以改变，因此，切片是一个可变的数组。
3. 切片遍历方式和数组一样，可以用len()求长度。表示可用元素数量，读写操作不能超过该限制。 
4. cap可以求出slice最大扩张容量，不能超出数组限制。0<=len(slice)<=len(array)，其中array是slice引用的数组。
5. 切片的定义：var 变量名 []类型，比如 var str []string  var arr []int。
6. 如果 slice == nil，那么 len、cap 结果都等于 0。
```

## 4. [:6:8] 两个冒号的理解

 - 常规slice , data[6:8]，从第6位到第8位（返回6， 7），长度len为2， 最大可扩充长度cap为4（6-9）
 - 另一种写法： data[:6:8] 每个数字前都有个冒号， slice内容为data从0到第6位，长度len为6，最大扩充项cap设置为8

a[x:y:z] 切片内容 [x:y] 切片长度: y-x 切片容量:z-x

```bash
package main
import ("fmt")
func main(){
    slice :=[]int{0,1,2,3,4,5,6,7,8,9}
    d1 := slice[6:8]
    fmt.Println(d1,len(d1),cap(d1))
    d2 := slice[:6:8]
    fmt.Println(d2,len(d2),cap(d2))}
```

输出：

```bash
[6 7] 2 4
[0 1 2 3 4 5] 6 8
```

##  5. 切片初始化

```bash
全局：
var arr =[...]int{0,1,2,3,4,5,6,7,8,9}
var slice0 []int = arr[start:end] 
var slice1 []int = arr[:end]        
var slice2 []int = arr[start:]        
var slice3 []int = arr[:] 
var slice4 = arr[:len(arr)-1]//去掉切片的最后一个元素
局部：
arr2 :=[...]int{9,8,7,6,5,4,3,2,1,0}
slice5 := arr[start:end]
slice6 := arr[:end]        
slice7 := arr[start:]     
slice8 := arr[:]  
slice9 := arr[:len(arr)-1]//去掉切片的最后一个元素
```
slice1.go

```bash
package main
import ("fmt")
var arr =[...]int{0,1,2,3,4,5,6,7,8,9}
var slice0 []int = arr[2:8]
var slice1 []int = arr[0:6]//可以简写为 var slice []int = arr[:end]
var slice2 []int = arr[5:10]//可以简写为 var slice[]int = arr[start:]
var slice3 []int = arr[0:len(arr)]//var slice []int = arr[:]
var slice4 = arr[:len(arr)-1]//去掉切片的最后一个元素
func main(){
    fmt.Printf("全局变量：arr %v\n", arr)
    fmt.Printf("全局变量：slice0 %v\n", slice0)
    fmt.Printf("全局变量：slice1 %v\n", slice1)
    fmt.Printf("全局变量：slice2 %v\n", slice2)
    fmt.Printf("全局变量：slice3 %v\n", slice3)
    fmt.Printf("全局变量：slice4 %v\n", slice4)
    fmt.Printf("-----------------------------------\n")
    arr2 :=[...]int{9,8,7,6,5,4,3,2,1,0}
    slice5 := arr[2:8]
    slice6 := arr[0:6]//可以简写为 slice := arr[:end]
    slice7 := arr[5:10]//可以简写为 slice := arr[start:]
    slice8 := arr[0:len(arr)]//slice := arr[:]
    slice9 := arr[:len(arr)-1]//去掉切片的最后一个元素
    fmt.Printf("局部变量： arr2 %v\n", arr2)
    fmt.Printf("局部变量： slice5 %v\n", slice5)
    fmt.Printf("局部变量： slice6 %v\n", slice6)
    fmt.Printf("局部变量： slice7 %v\n", slice7)
    fmt.Printf("局部变量： slice8 %v\n", slice8)
    fmt.Printf("局部变量： slice9 %v\n", slice9)}
输出结果：

全局变量：arr [0123456789]
全局变量：slice0 [234567]
全局变量：slice1 [012345]
全局变量：slice2 [56789]
全局变量：slice3 [0123456789]
全局变量：slice4 [012345678]-----------------------------------
局部变量： arr2 [9876543210]
局部变量： slice5 [234567]
局部变量： slice6 [012345]
局部变量： slice7 [56789]
局部变量： slice8 [0123456789]
局部变量： slice9 [012345678]
```
## 6. 读写操作
实际目标是底层数组，只需注意索引号的差别。

```bash
package main
import ("fmt")
func main(){
    data :=[...]int{0,1,2,3,4,5}
    s := data[2:4]
    s[0]+=100
    s[1]+=200
    fmt.Println(s)
    fmt.Println(data)}
输出:

[102203][0110220345]
```

## 7. 通过make来创建切片
语法：

```bash
func make([]T, len, cap) []T
```
其中T代表要创建的切片的元素类型。 make函数采用类型，长度和可选容量。 调用时，make会分配一个数组并返回一个引用该数组的切片

```bash
var slice []type =make([]type, len)
slice  :=make([]type, len)
slice  :=make([]type, len, cap)
```

```bash
var s []byte
s = make([]byte, 5, 5)
// s == []byte{0, 0, 0, 0, 0}
```

省略capacity参数时，默认为指定的长度。 这是相同代码的更简洁版本：

```bash
s := make([]byte, 5)
```

可以使用内置的len和cap函数检查切片的长度和容量。

```bash
len(s) == 5
cap(s) == 5
```

![](https://img-blog.csdnimg.cn/20200305155541904.png)
slice2.go

```bash
package main
import ("fmt")
var slice0 []int =make([]int,10)
var slice1 =make([]int,10)
var slice2 =make([]int,10,10)
func main(){
    fmt.Printf("make全局slice0 ：%v\n", slice0)
    fmt.Printf("make全局slice1 ：%v\n", slice1)
    fmt.Printf("make全局slice2 ：%v\n", slice2)
    fmt.Println("--------------------------------------")
    slice3 :=make([]int,10)
    slice4 :=make([]int,10)
    slice5 :=make([]int,10,10)
    fmt.Printf("make局部slice3 ：%v\n", slice3)
    fmt.Printf("make局部slice4 ：%v\n", slice4)
    fmt.Printf("make局部slice5 ：%v\n", slice5)}
输出结果：

make全局slice0 ：[0000000000]
make全局slice1 ：[0000000000]
make全局slice2 ：[0000000000]--------------------------------------
make局部slice3 ：[0000000000]
make局部slice4 ：[0000000000]
make局部slice5 ：[0000000000]
```


**可直接创建 slice 对象，自动分配底层数组。**

```bash
package main
import "fmt"
func main(){
    s1 :=[]int{0,1,2,3,8:100}// 通过初始化表达式构造，可使用索引号。
    fmt.Println(s1,len(s1),cap(s1))
    s2 :=make([]int,6,8)// 使用 make 创建，指定 len 和 cap 值。
    fmt.Println(s2,len(s2),cap(s2))
    s3 :=make([]int,6)// 省略 cap，相当于 cap = len。
    fmt.Println(s3,len(s3),cap(s3))}
输出结果:

[0 1 2 3 0 0 0 0 100] 9 9
[0 0 0 0 0 0] 6 8
[0 0 0 0 0 0] 6 6

```
**用指针直接访问底层数组，退化成普通数组操作**

```bash
package main
import "fmt"
func main(){
    s :=[]int{0,1,2,3}
    p := &s[2]      // *int, 获取底层数组元素指针。*p +=100
    fmt.Println(s,p)
}
```
输出
```bash
[root@localhost slice]# go run slice2.go
[0 1 2 3] 0xc000014190
```

 **[][]T，是指元素类型为 []T 。**

```bash
package main
import ("fmt")
func main(){
    data :=[][]int{[]int{1,2,3},[]int{100,200},[]int{11,22,33,44},}
    fmt.Println(data)}
输出结果：

[[123][100200][11223344]]
```
## 8. 直接修改 struct array/slice 成员

```bash
package main
import ("fmt")
func main(){
    d :=[5]struct {
        x int
    }{}
    s := d[:]
    d[1].x =10
    s[2].x =20
    fmt.Println(d)
    fmt.Printf("%p, %p\n",&d,&d[0])}
输出结果:

[{0}{10}{20}{0}{0}]0xc4200160f0,0xc4200160f0
```
## 9. append内置函数操作切片
### 9.1 切片追加

```bash
package main
import ("fmt")
func main(){
    var a =[]int{1,2,3}
    fmt.Printf("slice a : %v\n", a)
    var b =[]int{4,5,6}
    fmt.Printf("slice b : %v\n", b)
    c :=append(a, b...)
    fmt.Printf("slice c : %v\n", c)
    d :=append(c,7)
    fmt.Printf("slice d : %v\n", d)
    e :=append(d,8,9,10)
    fmt.Printf("slice e : %v\n", e)}
输出结果：

slice a :[123]
slice b :[456]
slice c :[123456]
slice d :[1234567]
slice e :[12345678910]
```
### 9.2  向 slice 尾部添加数据，返回新的 slice 对象

```bash
package main
import ("fmt")
func main(){
    s1 :=make([]int,0,5)
    fmt.Printf("%p\n",&s1)
    s2 :=append(s1,1)
    fmt.Printf("%p\n",&s2)
    fmt.Println(s1, s2)}
输出结果：

0xc42000a0600xc42000a080[][1]
```
**超出原 slice.cap 限制，就会重新分配底层数组，即便原数组并未填满**

```bash
package main
import ("fmt")
func main(){
    data :=[...]int{0,1,2,3,4,10:0}
    s := data[:2:3]
    s =append(s,100,200)// 一次 append 两个值，超出 s.cap 限制。
    fmt.Println(s, data)// 重新分配底层数组，与原数组无关。
    fmt.Println(&s[0],&data[0])// 比对底层数组起始指针。}
输出结果:

[01100200][01234000000]0xc4200160f00xc420070060
```
从输出结果可以看出，append 后的 s 重新分配了底层数组，并复制数据。如果只追加一个值，则不会超过 s.cap 限制，也就不会重新分配。

通常以 2 倍容量重新分配底层数组。在大批量添加数据时，建议一次性分配足够大的空间，以减少内存分配和数据复制开销。或初始化足够长的 len 属性，改用索引号进行操作。及时释放不再使用的 slice 对象，避免持有过期数组，造成 GC 无法回收。

### 9.3 slice中cap重新分配规律

```bash
package main
import ("fmt")
func main(){
    s :=make([]int,0,1)
    c :=cap(s)
    for i :=0; i <50; i++{
        s =append(s, i)
        if n :=cap(s); n > c {
            fmt.Printf("cap: %d -> %d\n", c, n)
            c = n
        }
    }
}
```
## 10. copy切片拷贝

```bash
package main
import ("fmt")
func main(){
    s1 :=[]int{1,2,3,4,5}
    fmt.Printf("slice s1 : %v\n", s1)
    s2 :=make([]int,10)
    fmt.Printf("slice s2 : %v\n", s2)
    copy(s2, s1)
    fmt.Printf("copied slice s1 : %v\n", s1)
    fmt.Printf("copied slice s2 : %v\n", s2)
    s3 :=[]int{1,2,3}
    fmt.Printf("slice s3 : %v\n", s3)
    s3 =append(s3, s2...)
    fmt.Printf("appended slice s3 : %v\n", s3)
    s3 =append(s3,4,5,6)
    fmt.Printf("last slice s3 : %v\n", s3)}
输出结果：

slice s1 :[12345]
slice s2 :[0000000000]
copied slice s1 :[12345]
copied slice s2 :[1234500000]
slice s3 :[123]
appended slice s3 :[1231234500000]
last slice s3 :[1231234500000456]
```
 **两个 slice 可指向同一底层数组，允许元素区间重叠**。
函数 copy 在两个 slice 间复制数据，复制长度以 len 小的为准

```bash
package main
import ("fmt")
func main(){
    data :=[...]int{0,1,2,3,4,5,6,7,8,9}
    fmt.Println("array data : ", data)
    s1 := data[8:]
    s2 := data[:5]
    fmt.Printf("slice s1 : %v\n", s1)
    fmt.Printf("slice s2 : %v\n", s2)copy(s2, s1)
    fmt.Printf("copied slice s1 : %v\n", s1)
    fmt.Printf("copied slice s2 : %v\n", s2)
    fmt.Println("last array data : ", data)}

输出结果:

array data :[0123456789]
slice s1 :[89]
slice s2 :[01234]
copied slice s1 :[89]
copied slice s2 :[89234]
last array data :[8923456789]
```
应及时将所需数据 copy 到较小的 slice，以便释放超大号底层数组内存。

## 11. slice遍历

```bash
package main
import ("fmt")
func main(){
    data :=[...]int{0,1,2,3,4,5,6,7,8,9}
    slice := data[:]
    for index, value := range slice {
        fmt.Printf("inde : %v , value : %v\n", index, value)}}
输出结果：

inde :0, value :0
inde :1, value :1
inde :2, value :2
inde :3, value :3
inde :4, value :4
inde :5, value :5
inde :6, value :6
inde :7, value :7
inde :8, value :8
inde :9, value :9
```
### 12. 切片resize（调整大小）

```bash
package main
import ("fmt")
func main(){
    var a =[]int{1,3,4,5}
    fmt.Printf("slice a : %v , len(a) : %v\n", a,len(a))
    b := a[1:2]
    fmt.Printf("slice b : %v , len(b) : %v\n", b,len(b))
    c := b[0:3]
    fmt.Printf("slice c : %v , len(c) : %v\n", c,len(c))}
输出结果：

slice a :[1345],len(a):4
slice b :[3],len(b):1
slice c :[345],len(c):3
```
## 13. 字符串和切片（string and slice）
string底层就是一个byte的数组，因此，也可以进行切片操作。

```bash
package main
import ("fmt")
func main(){
    str :="hello world"
    s1 := str[0:5]
    fmt.Println(s1)
    s2 := str[6:]
    fmt.Println(s2)}
输出结果：

hello
world
```
## 14. 修改字符串
string本身是不可变的，因此要改变string中字符。需要如下操作：

英文字符串：

```bash
package main
import ("fmt")
func main(){
    str :="Hello world"
    s :=[]byte(str)//中文字符需要用[]rune(str)
    s[6]='G'
    s = s[:8]
    s =append(s,'!')
    str =string(s)
    fmt.Println(str)}
输出结果：

Hello Go!
```
含有中文字符串：

```bash
package main
import ("fmt")
func main(){
    str :="你好，世界！hello world！"
    s :=[]rune(str) 
    s[3]='够'
    s[4]='浪'
    s[12]='g'
    s = s[:14]
    str =string(s)
    fmt.Println(str)}
输出结果：

你好，够浪！hello go
```
参考：
- [http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.4.2.md](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.4.2.md)
