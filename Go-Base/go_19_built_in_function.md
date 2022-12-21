#  Go 内置函数

![](https://img-blog.csdnimg.cn/3ecf3a90188f478c8ba13d7d1a6d0dbf.png)


## 1. close

close用于关闭一个channel，使用close函数要注意以下几点：

 - 关闭一个只接受的channel会导致错误
 - 在一个已经关闭的channel上发送数据会导致panic
 - 关闭一个nil channel会导致panic
 - 在一个channel关闭之后，如果channel已经没有剩余数据等待接受了，这时候如果继续接收，会返回一个channel对应数据类型的nil
   value，如果接收的时候使用多返回值，第二个参数表示一个channel是否已经关闭。

```go
package main
import "fmt"
func main() {
    ch := make(chan int, 5)

    for i := 0; i < 5; i++ {
        ch <- i
    }

    close(ch) // 关闭ch
    for i := 0; i < 10; i++ {
        e, ok := <-ch
        fmt.Printf("%v, %v\n", e, ok)

        if !ok {
            break
        }
    }
}
输出：
0, true
1, true
2, true
3, true
4, true
0, false
```

在close之后， 还可以读取， 不过在读取完之后， 再检测ok, 就是false了
## 2. len 和 cap

len和cap都接收多种类型的参数，返回值是int类型，具体接收哪些类型的参数
创建方式make([]type, len, cap)，其中，type表示数组元素类型，`len`表示长度，`cap`表示容量
以及返回的值的含义见下表

```bash
len(s)	
string	        字符串的字节长度
[n]T *[n]T  	数组的长度（==n）
[]T	            slice的长度
map[K][T]	map的长度，即有多少个key-value对
chan T	        在channel里面有多少个等待接收的元素

cap(s)	
[n]T *[n]T	    数组长度（==n）
[]T	            slice的capacity（预分配空间）
chan T	        channel的buffer的长度
```

len，cap的返回值满足如下条件：
**0<=len(s)<=cap(s)
slice,map,channel的nil值的len为0
slice,channel的nil值的cap为0**

##  3. new 与 make

```go
$ cat mem.go
package main
import (
 "fmt"
)
func main() {
 var i *int
 *i=10
 fmt.Println(*i)
}
[root@localhost func]# go run mem.go
panic: runtime error: invalid memory address or nil pointer dereference
```
从这个提示中可以看出，对于引用类型的变量，我们`不光要声明`它，还要为它`分配内容空间`，否则我们的值放在哪里去呢？这就是上面错误提示的原因
对于值类型的声明不需要，是因为已经默认帮我们分配好了
分配内存，Go提供了两种方式，分别是`new和make`

#### 3.1 new
new函数的参数是一个类型，返回一个指向该类型的指针，并且进行0值初始化
```go
func new(Type) *Type
```
**它只接受一个参数，这个参数是一个类型，分配好内存后，返回一个指向该类型内存地址的指针。同时请注意它同时把分配的内存置为零，也就是类型的零值**。那么上面的函数可以改写成

```go
$ cat new.go
package main
import "fmt"
func main() {
 var i *int
 i=new(int)
 *i=10
 fmt.Println(*i)
}
[root@localhost func]# go run new.go 
10
```

类似new（）的功能：

```go
func newInt() *int {
  var i int
  return &i
}
someInt := newInt()
```

m>=n，且n和m必须是整型且不能为负数。

#### 3.2 make

make也根据不同参数类型和参数个数具有不同的含义，它的定义比 new 多了一个参数，返回值也不同：

```go
func make(Type, size ...IntegerType) Type
```
内建函数 make 用来为 `slice，map 或 chan` 类型分配内存和初始化一个对象(注意：只能用在这三种类型上)，跟 new 类似，第一个参数也是一个类型而不是一个值，跟 new 不同的是，make 返回类型的引用而不是指针，而返回值也依赖于具体传入的类型。

 - Slice: 第二个参数 size 指定了它的长度，它的容量和长度相同。 你可以传入第三个参数来指定不同的容量值，但必须不能比长度值小。
   比如 make([]int, 0, 10)
 - Map: 根据 size 大小来初始化分配内存，不过分配后的 map 长度为 0，如果 size被忽略了，那么会在初始化分配内存时分配一个小尺寸的内存
 - Channel: 管道缓冲区依据缓冲区容量被初始化。如果容量为 0 或者忽略容量，管道是没有缓冲区的

```bash
make(T, n)	    slice	     创建一个T类型的slice且长度为n
make(T, n, m)	slice	     创建一个T类型的slice且长度为n，capacity位m
make(T)	        map	         创建一个T类型的map
make(T, n)	    map	         创建一个T类型的map，且预分配n个空间
make(T)	        channel	     创一个channel
make(T, n)	    channel	     创建一个拥有n长度的buffer的channel
c := make(chan int)      如果不指定容量，默认通道的容量是0，这种通道也成为非缓冲通道
c := make(chan string, 10)
```
slice数组切片实例：
```go
$ cat make1.go
package main
import "fmt"
func main() {
a := make([]int, 2)
b := make([]int, 2, 10)
fmt.Println(a, b)
fmt.Println(len(a), len(b))
}
[root@localhost func]#  go run make1.go
[0 0] [0 0]
2 2
```
map字典初始化实例1：

```go
package main
import (
    "fmt"
    "reflect"
)
func main() {
    var yinzhengjie map[string]string //先声明一个字典（map）名字叫做yinzhengjie。其key所对应的数据类型是“string”[字符串]，value所对应的数据类型也是“string”。
    fmt.Printf("判断yinzhengjie字典是否为空:【%v】\n",yinzhengjie == nil)  //声明的字典，默认为空，需要用make进行初始化操作(map是引用类型，未初始化的是指向nil，初始化了以后应该就有自己的内存空间了，所以不是nil。)所以返回值为空。
    fmt.Printf("第一次查看yinzhengjie字典的值：【%v】\n",yinzhengjie)
    yinzhengjie = make(map[string]string) //再使用make函数进行初始化创建一个非nil的map，nil map不能赋值,如果直接赋值会报错：“panic: assignment to entry in nil map”
    fmt.Printf("再次判断yinzhengjie字典是否为空：【%v】\n",yinzhengjie == nil) //你就把它理解为一个指针，没初始化就是nil，make之后分配内存了,一旦分配了内存地址就不为空了
    fmt.Printf("第二次查看yinzhengjie字典的值：【%v】\n",yinzhengjie)
    yinzhengjie["name"] = "尹正杰"
    fmt.Printf("yinzhengjie字典的类型为：【%v】\n",reflect.TypeOf(yinzhengjie))
    fmt.Printf("第三次查看yinzhengjie字典的值：【%v】\n",yinzhengjie)
}

#以上代码执行结果如下：
判断yinzhengjie字典是否为空:【true】
第一次查看yinzhengjie字典的值：【map[]】
再次判断yinzhengjie字典是否为空：【false】
第二次查看yinzhengjie字典的值：【map[]】
yinzhengjie字典的类型为：【map[string]string】
第三次查看yinzhengjie字典的值：【map[name:尹正杰]】
```
map字典直接初始化实例2：

```go
package main
import "fmt"
func main() {
    yinzhengjie := map[string]int{
        "尹正杰":18,
        "饼干":20,
    }
    fmt.Println(yinzhengjie)
}

#以上代码执行结果如下：
map[尹正杰:18 饼干:20]
```
channel初始化实例1：

```go
$ cat ch01.go
package main
import "fmt"
channel初始化


func main() {
c := make(chan int)
defer close(c)
go func() { c <- 3 + 4 }()
i := <-c
fmt.Println(i)
}
[root@localhost chan]#  go run ch01.go
7
```
channel初始化实例2：
```go
cat ch12.go 
package main
import (
	"fmt"
	"time"
)
func worker(done chan bool) {
	time.Sleep(time.Second)
	// 通知任务已完成
	done <- true
}
func main() {
	done := make(chan bool, 1)
	go worker(done)
	// 等待任务完成
        fmt.Println(<-done)
}
[root@localhost chan]#  go run ch12.go
true
```

参考链接：
[https://www.cnblogs.com/yinzhengjie/p/7689996.html](https://www.cnblogs.com/yinzhengjie/p/7689996.html)


## 4. append

```bash
append(s S, x …T) S
```

Go语言的内建函数 append() 可以为切片动态添加元素，代码如下所示：

```go
var a []int
a = append(a, 1) // 追加1个元素
a = append(a, 1, 2, 3) // 追加多个元素, 手写解包方式
a = append(a, []int{1,2,3}...) // 追加一个切片, 切片需要解包
```

不过需要注意的是，在使用 append() 函数为切片动态添加元素时，如果空间不足以容纳足够多的元素，切片就会进行“扩容”，此时新切片的长度会发生改变。

切片在扩容时，容量的扩展规律是按容量的 2 倍数进行扩充，例如 1、2、4、8、16……，代码如下：

```go
var numbers []int
for i := 0; i < 10; i++ {
    numbers = append(numbers, i)
    fmt.Printf("len: %d  cap: %d pointer: %p\n", len(numbers), cap(numbers), numbers)
}
代码输出如下：
len: 1  cap: 1 pointer: 0xc0420080e8
len: 2  cap: 2 pointer: 0xc042008150
len: 3  cap: 4 pointer: 0xc04200e320
len: 4  cap: 4 pointer: 0xc04200e320
len: 5  cap: 8 pointer: 0xc04200c200
len: 6  cap: 8 pointer: 0xc04200c200
len: 7  cap: 8 pointer: 0xc04200c200
len: 8  cap: 8 pointer: 0xc04200c200
len: 9  cap: 16 pointer: 0xc042074000
len: 10  cap: 16 pointer: 0xc042074000
```

代码说明如下：
第 1 行，声明一个整型切片。
第 4 行，循环向 numbers 切片中添加 10 个数。
第 5 行，打印输出切片的长度、容量和指针变化，使用函数 len() 查看切片拥有的元素个数，使用函数 cap() 查看切片的容量情况。

**开头添加元素**：

```go
var a = []int{1,2,3}
a = append([]int{0}, a...) // 在开头添加1个元素
a = append([]int{-3,-2,-1}, a...) // 在开头添加1个切片
```

在切片开头添加元素一般都会导致内存的重新分配，而且会导致已有元素全部被复制 1 次，因此，从切片的开头添加元素的性能要比从尾部追加元素的性能差很多。
**中间插入元素**
因为 append 函数返回新切片的特性，所以切片也支持链式操作，我们可以将多个 append 操作组合起来，实现在切片中间插入元素：
```go
var a []int
a = append(a[:i], append([]int{x}, a[i:]...)...) // 在第i个位置插入x
a = append(a[:i], append([]int{1,2,3}, a[i:]...)...) // 在第i个位置插入切片
```

每个添加操作中的第二个 append 调用都会创建一个临时切片，并将 a[i:] 的内容复制到新创建的切片中，然后将临时创建的切片再追加到 a[:i] 中。

```bash
s = append(s, a)
```
**append覆盖**
实例1:
```go
package main

import "fmt"

func main(){
  s := []int{5}
    
  s = append(s,7)
  fmt.Println("cap(s) =", cap(s), "ptr(s) =", &s[0])
    
  s = append(s,9)
  fmt.Println("cap(s) =", cap(s), "ptr(s) =", &s[0])
    
  x := append(s, 11)
  fmt.Println("cap(s) =", cap(s), "ptr(s) =", &s[0], "ptr(x) =", &x[0])
    
  y := append(s, 12)
  fmt.Println("cap(s) =", cap(s), "ptr(s) =", &s[0], "ptr(y) =", &y[0])
}
输出:
cap(s) = 2 ptr(s) = 0x10328008
cap(s) = 4 ptr(s) = 0x103280f0
cap(s) = 4 ptr(s) = 0x103280f0 ptr(x) = 0x103280f0
cap(s) = 4 ptr(s) = 0x103280f0 ptr(y) = 0x103280f0
```
**文字说明:**
创建s时，cap(s) == 1，内存中数据[5]
append(s, 7) 时，按Slice扩容机制，cap(s)翻倍 == 2，内存中数据[5,7]
append(s, 9) 时，按Slice扩容机制，cap(s)再翻倍 == 4，内存中数据[5,7,9]，但是实际内存块容量4
x := append(s, 11) 时，容量足够不需要扩容，内存中数据[5,7,9,11]
y := append(s, 12) 时，容量足够不需要扩容，内存中数据[5,7,9,12]
5中的append是在s的基础上加入元素12,实际上就是从s切片的末尾开始写入,所以覆盖掉了11
扩展链接：
[https://learnku.com/articles/38316](https://learnku.com/articles/38316)

## 5. copy
Go内建函数copy:

```go
func copy(dst, src []Type) int
```
**用于将源slice的数据（第二个参数），复制到目标slice（第一个参数）。
返回值为拷贝了的数据个数，是len(dst)和len(src)中的最小值。**

```go
package main
import (
"fmt"
)
func main() {
var a = []int{0, 1, 2, 3, 4, 5, 6, 7}
var s = make([]int, 6)
//源长度为8，目标为6，只会复制前6个
n1 := copy(s, a)
fmt.Println("s - ", s)
fmt.Println("n1 - ", n1)
//源长为7，目标为6，复制索引1到6
n2 := copy(s, a[1:])
fmt.Println("s - ", s)
fmt.Println("n2 - ", n2)
//源长为8-5=3，只会复制3个值，目标中的后三个值不会变
n3 := copy(s, a[5:])
fmt.Println("s - ", s)
fmt.Println("n3 - ", n3)
//将源中的索引5,6,7复制到目标中的索引3,4,5
n4 := copy(s[3:], a[5:])
fmt.Println("s - ", s)
fmt.Println("n4 - ", n4)
}
执行结果：
s - [0 1 2 3 4 5]
n1 - 6
s - [1 2 3 4 5 6]
n2 - 6
s - [5 6 7 4 5 6]
n3 - 3
s - [5 6 7 5 6 7]
n4 - 3
```
参考链接：
[https://www.cnblogs.com/baiyuxiong/p/4770978.html](https://www.cnblogs.com/baiyuxiong/p/4770978.html)
## 6. delete

删除一个map指定key的元素

```go
delete(m, k)
```
实例1：
```go
$ cat del1.go
package main
import "fmt"
func main() {
    m := map[string]int{
        "a": 1,
        "b": 2,
        "c": 3,
    }

    fmt.Println("Deleting values")
    name, ok := m["a"]
    fmt.Println(name,ok)

    delete(m,"a")
    name,ok = m["a"]
    fmt.Println(name,ok)
}

[root@localhost func]#  go run del1.go
Deleting values
1 true
0 false
```

## 7. panic 与 recover
panic内置函数停止当前goroutine的正常执行，当函数F调用panic时，函数F的正常执行被立即停止，然后运行所有在F函数中的defer函数，然后F返回到调用他的函数对于调用者G，F函数的行为就像panic一样，终止G的执行并运行G中所defer函数，此过程会一直继续执行到goroutine所有的函数。panic可以通过内置的recover来捕获。
```go
panic(interface{})
```
1.defer 表达式的函数如果定义在 panic 后面，该函数在 panic 后就无法被执行到
在defer前panic

```go
$ cat panic1.go 
package main
import "fmt"
func main() {
    panic("a")
    defer func() {
        fmt.Println("b")
    }()
}
[root@localhost func]# go run panic1.go   //结果，b没有被打印出来
panic: a

goroutine 1 [running]:
main.main()
	/root/go/func/panic1.go:4 +0x39
exit status 2
```
而在defer后panic

```go
$ cat panic2.go 
package main
import "fmt"

func main() {
    defer func() {
        fmt.Println("b")
    }()
    panic("a")
}
[root@localhost func]# go run panic2.go   ////结果，b被打印出来
b
panic: a

goroutine 1 [running]:
main.main()
	/root/go/func/panic2.go:8 +0x5f
exit status 2
```
2.F中出现panic时，F函数会立刻终止，不会执行F函数内panic后面的内容，但不会立刻return，而是调用F的defer，如果F的defer中有recover捕获，则F在执行完defer后正常返回，调用函数F的函数G继续正常执行

```go
$ cat panic3.go
package main
import "fmt"

func F() {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("捕获异常:", err)
        }
        fmt.Println("b")
    }()
    panic("a")
}

func main() {
    defer func() {
        fmt.Println("c")
    }()
    F()
    fmt.Println("继续执行")
}

[root@localhost func]#  go run panic3.go
捕获异常: a
b
继续执行
c
```
3、如果F的defer中无recover捕获，则将panic抛到G中，G函数会立刻终止，不会执行G函数内后面的内容，但不会立刻return，而调用G的defer...以此类推

```go
$ cat panic4.go
package main
import "fmt"

func F() {
    defer func() {
        fmt.Println("b")
    }()
    panic("a")
}

func main() {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("捕获异常:", err)
        }
        fmt.Println("c")
    }()
    F()
    fmt.Println("继续执行")
}

[root@localhost func]# go run panic4.go
b
捕获异常: a
c
```
4、如果一直没有recover，抛出的panic到当前goroutine最上层函数时，程序直接异常终止

```go
$ cat panic5.go
package main
import "fmt"

func F() {
    defer func() {
        fmt.Println("b")
    }()
    panic("a")
}

func main() {
    defer func() {
        fmt.Println("c")
    }()
    F()
    fmt.Println("继续执行")
}

[root@localhost func]#  go run panic5.go
b
c
panic: a

goroutine 1 [running]:
main.F()
	/root/go/func/panic5.go:8 +0x5f
main.main()
	/root/go/func/panic5.go:15 +0x5a
exit status 2
```
5、recover都是在当前的goroutine里进行捕获的，这就是说，对于创建goroutine的外层函数，如果goroutine内部发生panic并且内部没有用recover，外层函数是无法用recover来捕获的，这样会造成程序崩溃

```go
$ cat panic6.go
package main
import (
      "fmt"
      "time"
)

func F() {
    defer func() {
        fmt.Println("b")
    }()
    //goroutine内部抛出panic
    panic("a")
}

func main() {
    defer func() {
        //goroutine外进行recover
        if err := recover(); err != nil {
            fmt.Println("捕获异常:", err)
        }
        fmt.Println("c")
    }()
    //创建goroutine调用F函数
    go F()
    time.Sleep(time.Second)
}

[root@localhost func]# go run panic6.go
b
panic: a

goroutine 18 [running]:
main.F()
	/root/go/func/panic6.go:12 +0x5f
created by main.main
	/root/go/func/panic6.go:24 +0x5b
exit status 2
```
6.recover返回的是interface{}类型而不是go中的 error 类型，如果外层函数需要调用err.Error()，会编译错误，也可能会在执行时panic

```go
$ cat  panic7.go
package main
import (
      "fmt"
      "time"
)

func F() {
    defer func() {
        fmt.Println("b")
    }()
    panic("a")
}

func main() {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("捕获异常:", err.Error())
        }
        fmt.Println("c")
    }()
    F()
    time.Sleep(time.Second)
}

[root@localhost func]# vim panic7.go
[root@localhost func]# go run panic7.go
# command-line-arguments
./panic7.go:17:45: err.Error undefined (type interface {} is interface with no methods)
```
正确实例：

```go
$ cat panic7.go
package main
import (
      "fmt"
      "time"
)

func F() {
    defer func() {
        fmt.Println("b")
    }()
    panic("a")
}

func main() {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("捕获异常:", fmt.Errorf("%v", err).Error())
        }
        fmt.Println("c")
    }()
    F()
    time.Sleep(time.Second)
}

[root@localhost func]# go run panic7.go
b
捕获异常: a
c
```



参考：
- [https://blog.csdn.net/sydnash/article/details/53516524](https://blog.csdn.net/sydnash/article/details/53516524)
