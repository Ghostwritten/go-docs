#  Go 了解 channel 管道

![](https://img-blog.csdnimg.cn/3eb4c5532fa94d98b9376928c17a6140.png)



## 1.channel概念
协程是并发编程的基础，而管道（channel）则是并发中协程之间沟通的桥梁，很多时候我们启动一个协程去执行完一个操作，执行操作之后我们需要返回结果，或者多个协程之间需要相互协作

```bash
a. 类似unix中管道（pipe）
b. 先进先出
c. 线程安全，多个goroutine同时访问，不需要加锁
d. channel是有类型的，一个整数的channel只能存放整数
```

## 2.channel语法

### 2.1 var 变量名 chan 类型

```go
package main
var ch0 chan int
var ch1 chan string
var ch2 chan map[string]string
type stu struct{}
var ch3 chan stu
var ch4 chan *stu
func main(){}
```
###  2.2 channel方向<-
```bash
ch <- v    // 发送值v到Channel ch中
v := <-ch  // 从Channel ch中接收数据，并将数据赋值给v
```
Channel类型的定义格式如下：

```go
ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .
```

它包括三种类型的定义。可选的<-代表channel的方向。如果没有指定方向，那么Channel就是双向的，既可以接收数据，也可以发送数据。

```go
chan T          // 可以接收和发送类型为 T 的数据
chan<- float64  // 只可以用来发送 float64 类型的数据
<-chan int      // 只可以用来接收 int 类型的数据
```

<-总是优先和最左边的类型结合。(The <- operator associates with the leftmost chan possible)

```go
chan<- chan int    // 等价 chan<- (chan int)
chan<- <-chan int  // 等价 chan<- (<-chan int)
<-chan <-chan int  // 等价 <-chan (<-chan int)
chan (<-chan int)
```
### 2.3 send发送（先进）
send语句用来往Channel中发送数据， 如`ch <- 3`。
它的定义如下:

```go
SendStmt = Channel "<-" Expression .
Channel  = Expression .
```

在通讯(communication)开始前channel和expression必选先求值出来(evaluated)，比如下面的(3+4)先计算出7然后再发送给channel。

```go
c := make(chan int)
defer close(c)
go func() { c <- 3 + 4 }()
i := <-c
fmt.Println(i)
```

send被执行前(proceed)通讯(communication)一直被阻塞着。如前所言，无缓存的channel只有在receiver准备好后send才被执行。如果有缓存，并且缓存未满，则send会被执行
往一个已经被close的channel中继续发送数据会导致run-time panic。
往nil channel中发送数据会一致被阻塞着。

### 2.4 receive 接受（先出）
`<-ch`用来从channel ch中接收数据，这个表达式会一直被block,直到有数据可以接收。
从一个`nil channel`中接收数据会一直被`block`。

从一个被close的channel中接收数据不会被阻塞，而是立即返回，接收完已发送的数据后会返回元素类型的零值(zero value)。

如前所述，你可以使用一个额外的返回参数来检查channel是否关闭。

```go
x, ok := <-ch
x, ok = <-ch
var x, ok = <-ch
```

如果OK 是false，表明接收的x是产生的零值，这个channel被关闭了或者为空。

## 3. make进行初始化

```go
package main
import ("fmt")
var ch0 chan int =make(chan int)
var ch1 chan int =make(chan int,10)
func main(){
    var ch2 chan string
    ch2 =make(chan string)
    var ch3 chan string
    ch3 =make(chan string,1)
    ch4 :=make(chan float32)
    ch5 :=make(chan float64,2)
    fmt.Printf("无缓冲 全局变量 chan ch0 : %v\n", ch0)
    fmt.Printf("有缓冲 全局变量 chan ch1 : %v\n", ch1)
    fmt.Printf("无缓冲 局部变量 chan ch2 : %v\n", ch2)
    fmt.Printf("有缓冲 局部变量 chan ch3 : %v\n", ch3)
    fmt.Printf("无缓冲 局部变量 chan ch4 : %v\n", ch4)
    fmt.Printf("有缓冲 局部变量 chan ch5 : %v\n", ch5)}
输出结果：

无缓冲 全局变量 chan ch0 :0xc420070060
有缓冲 全局变量 chan ch1 :0xc42001c0b0
无缓冲 局部变量 chan ch2 :0xc4200700c0
有缓冲 局部变量 chan ch3 :0xc420054060
无缓冲 局部变量 chan ch4 :0xc420070120
有缓冲 局部变量 chan ch5 :0xc420050070
```
通过缓存的使用，可以尽量避免阻塞，提供应用的性能。
`容量(capacity)`代表`Channel`容纳的最多的元素的数量，**代表Channel的缓存的大小**。
如果没有设置容量，或者容量设置为0, 说明Channel没有缓存，只有`sender和receiver`都准备好了后它们的通讯(communication)才会发生(Blocking)。如果设置了缓存，就有可能不发生阻塞， 只有`buffer`满了后 send才会阻塞， 而只有缓存空了后receive才会阻塞。一个nil channel不会通信。

可以通过内建的`close`方法可以关闭Channel。

你可以在多个`goroutine`从/往 一个channel 中 receive/send 数据, 不必考虑额外的同步措施。

Channel可以作为一个`先入先出`(FIFO)的队列，接收的数据和发送的数据的顺序是一致的。
无缓存的与有缓存channel有着重大差别，那就是一个是同步的 一个是非同步的。
比如
c1:=make(chan int) 无缓存
c2:=make(chan int,1) 有缓存
c1<-1
`无缓存`： 不仅仅是向 c1 通道放 1，而是一直要等有别的携程 <-c1 接手了这个参数，那么c1<-1才会继续下去，要不然就一直阻塞着。
`有缓存： c2<-1 则不会阻塞，因为缓存大小是1(其实是缓冲大小为0)，只有当放第二个值的时候，第一个还没被人拿走，这时候才会阻塞。
缓存区是内部属性，并非类型构成要素。


## 4. 不同类型`channel`写入、读取

```go
package main
import ("fmt")
type Stu struct {
    name string
}
func main(){
    var intChan chan int        //int类型
    intChan =make(chan int,10)
    intChan <-10
    a :=<-intChan
    fmt.Printf("int 类型 chan : %v\n", a)
    var mapChan chan map[string]string         //map类型
    mapChan =make(chan map[string]string,10)
    m :=make(map[string]string,16)
    m["stu01"]="001"
    m["stu02"]="002"
    m["stu03"]="003"
    mapChan <- m
    b :=<-mapChan
    fmt.Printf("map 类型 chan : %v\n", b)
    var stuChan chan Stu                  //结构体
    stuChan =make(chan Stu,10)
    stu := Stu{
        name:"Murphy",}
    stuChan <- stu
    tempStu :=<-stuChan
    fmt.Printf("struct 类型 chan : %v\n", tempStu)
    var stuChanId chan *Stu                //结构体内存地址值
    stuChanId =make(chan *Stu,10)
    stuId :=&Stu{
        name:"Murphy",}
    stuChanId <- stuId
    tempStuId :=<-stuChanId
    fmt.Printf("*struct 类型 chan : %v\n", tempStuId)
    fmt.Printf("*struct 类型 chan 取值 : %v\n",*(tempStuId))
    var StuInterChain chan interface{}                   //接口
    StuInterChain =make(chan interface{},10)
    stuInit := Stu{
        name:"Murphy",}//存
    StuInterChain <-&stuInit
    //取
    mFetchStu :=<-StuInterChain
    fmt.Printf("interface 类型 chan : %v\n", mFetchStu)//转
    var mStuConvert *Stu
    mStuConvert, ok := mFetchStu.(*Stu)if!ok {
        fmt.Println("cannot convert")return}
    fmt.Printf("interface chan转 *struct chan : %v\n", mStuConvert)
    fmt.Printf("interface chan转 *struct chan 取值 : %v\n",*(mStuConvert))}
输出结果：

int 类型 chan :10
map 类型 chan : map[stu02:002 stu03:003 stu01:001]
struct 类型 chan :{Murphy}*struct 类型 chan :&{Murphy}*struct 类型 chan 取值 :{Murphy}
interface 类型 chan :&{Murphy}
interface chan转 *struct chan :&{Murphy}
interface chan转 *struct chan 取值 :{Murphy}
```
## 5. 判断channel
它可以用来检查Channel是否已经被关闭了。
```go
v, ok := <-ch
```

## 6. channel 写入、读取

```go
cat ch1.go 
package main
import ("fmt")
func main(){
    ch :=make(chan int,11)//写入chan
    ch <-99
    for i :=0; i <10; i++{
        ch <- i
    }
    fmt.Printf("writed chan ch : %v\n", ch)//读取chan
    first_chan, ok :=<-ch
    if ok {
        fmt.Printf("first chan is %v\n", first_chan)}
    ch <-10
    for value := range ch {
        fmt.Println(value)
        if value ==10{
        close(ch)//break 
        }
    }
    fmt.Println("after range or close ch!")
}
[root@localhost chan]# go run ch1.go 
writed chan ch : 0xc000078000
first chan is 99
0
1
2
3
4
5
6
7
8
9
10
after range or close ch!
```
## 7. close()处理chan
**使用内置函数close进行关闭，chan关闭之后，for range遍历chan中已经存在的元素后结束
使用内置函数close进行关闭，chan关闭之后，没有使用for range的写法，需要使用，v, ok := <- ch进行判断chan是否关闭**


```go
$ cat ch2.go
package main

import "fmt"

func main(){

 var ch chan int
 ch =make(chan int,5)
 for i :=0; i <5; i++{
     ch <- i
 }
 close(ch)
 for{
     var b int
     b, ok :=<-ch
     if ok ==false{
         fmt.Println("chan is close")
         break
     }
 fmt.Println(b)}
}
[root@localhost chan]# go run ch2.go 
0
1
2
3
4
chan is close
```

**如果将close(ch)注释掉，意思是不关闭管道，那么会出现dead lock死锁**

**因为存入管道5个数字，然后无限取数据，会出现死锁。**


```go
$ cat  ch3.go 
package main
import "fmt"
func main(){
    var ch chan int
    ch =make(chan int,5)
    for i :=0; i <5; i++{
        ch <- i
    }
    //close(ch)
    for{
        var b int
        b, ok :=<-ch
        if ok ==false{
            fmt.Println("chan is close")
            break
        }
        fmt.Println(b)
    }
}
[root@localhost chan]# go run ch3.go 
0
1
2
3
4
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
	/root/go/chan/ch3.go:12 +0xfc
exit status 2
```

向 `closed channel` **发送数据**引发 panic 错误，接收立即返回零值。而 `nil channel`， 无论收发都会被阻塞。

```go
package main
func main(){
    ch :=make(chan int,1)
    close(ch)
    ch <-2}
输出结果：

panic: send on closed channel
goroutine 1[running]:
main.main()/Users/***/Desktop/go/src/main.go:6+0x63
exit status 2
```

## 8. range 遍历 chan


```go
$ cat ch4.go 
package main
import "fmt"
func main(){
    var ch chan int
    ch =make(chan int,10)
    for i :=0; i <10; i++{
        ch <- i
    }
    close(ch)
    for v := range ch {
        fmt.Println(v)
    }
}
[root@localhost chan]#  go run ch4.go 
0
1
2
3
4
5
6
7
8
9
```

同样如果将`close(ch)`注释掉，意思是不关闭管道，那么会出现`dead lock`死锁
除用 `range` 外，还可用 `ok-idiom` 模式判断 channel 是否关闭。
```go
$ cat  ch5.go
package main
import "fmt"
func main(){
    var ch chan int
    ch =make(chan int,10)
   for i :=0; i <10; i++{
        ch <- i
    }
    close(ch)
    for {
       if d, ok :=<-ch; ok {
          fmt.Println(d)
       } else{
          break
        }
    }
}
[root@localhost chan]# go run ch5.go
0
1
2
3
4
5
6
7
8
9
```




## 9. 内置函数len()、cap()处理chan

 `len` 返回未被读取的缓冲元素数量，cap 返回缓冲区大小。


```go
$ cat ch6.go
package main
import "fmt"
func main(){
    ch1 :=make(chan int)
    ch2 :=make(chan int,3)
    ch2 <-1
    fmt.Println(len(ch1),cap(ch1))
    fmt.Println(len(ch2),cap(ch2))}
[root@localhost chan]# go run ch6.go
0 0
1 3
```

## 10. select处理chan

`select` 语句类似于 `switch` 语句，但是select会随机执行一个可运行的case。如果没有case可运行，它将阻塞，直到有case可运行。
select语句选择一组可能的send操作和receive操作去处理。它类似switch,但是只是用来处理通讯(communication)操作。
它的case可以是send语句，也可以是receive语句，亦或者default。

receive语句可以将值赋值给一个或者两个变量。它必须是一个receive操作。

最多允许有一个default case,它可以放在case列表的任何位置，尽管我们大部分会将它放在最后。

```go
$ cat ch7-1.go
package main
import ("fmt")
func main(){
    ch1 :=make(chan int,1)
    ch1 <-1
    ch2 :=make(chan int,1)
    ch2 <-2
    select {             //随机读数
    case k1 :=<-ch1:
        fmt.Println(k1)
    case k2 :=<-ch2:
        fmt.Println(k2)
    default:
        fmt.Println("chan")}}

[root@localhost chan]# go run ch7.go 
2
[root@localhost chan]# go run ch7.go 
1
```

```go
$ cat ch7-2.go 
package main
import "fmt"
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}
func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
[root@localhost chan]# go run ch10.go
0
1
1
2
3
5
8
13
21
34
quit
```

## 11. timeout处理chan
select有很重要的一个应用就是超时处理。 因为上面我们提到，如果没有case需要处理，select语句就会一直阻塞着。这时候我们可能就需要一个超时操作，用来处理超时的情况。
下面这个例子我们会在2秒后往channel c1中发送一个数据，但是select设置为1秒超时,因此我们会打印出timeout 1,而不是result 1。

```go
$ cat ch11.go
package main
import "time"
import "fmt"
func main() {
    c1 := make(chan string, 1)
    go func() {
        time.Sleep(time.Second * 2)
        c1 <- "result 1"
    }()
    select {
    case res := <-c1:
        fmt.Println(res)
    case <-time.After(time.Second * 1):
        fmt.Println("timeout 1")
    }
}
[root@localhost chan]# go run ch11.go
timeout 1
[root@localhost chan]# go run ch11.go
timeout 1
```
其实它利用的是time.After方法，它返回一个类型为<-chan Time的单向的channel，在指定的时间发送一个当前时间给返回的channel中。
## 12. chan的只读和只写

### 12.1 只读chan的声明

```go
var 变量名 <-chan 类型
```

```go
package main
var ch0 <-chan int
var ch1 <-chan string
var ch2 <-chan map[string]string
type stu struct{}
var ch3 <-chan stu
var ch4 <-chan *stu
func main(){}
```

### 12.2 只写chan的声明

```go
var 变量名 chan<- 类型
```

```go
package main
var ch0 chan<- int
var ch1 chan<- string
var ch2 chan<- map[string]string
type stu struct{}
var ch3 chan<- stu
var ch4 chan<-*stu
func main(){}
```

## 13. channel单向

可以将 channel 隐式转换为单向队列，只收或只发。

```go
package main
import ("fmt")
func main(){
    c :=make(chan int,3)
    var send chan<- int = c // send-only
    var recv <-chan int = c // receive-only
    send <-1// <-send               // Error: receive from send-only type chan<- int
    val, ok :=<-recv
    if ok {
        fmt.Println(val)}// recv <- 2           // Error: send to receive-only type <-chan int}
输出结果：

1
```

**不能将单向 channel 转换为普通 channel。**

```go
package main
func main(){
    c :=make(chan int,3)
    var send chan<- int = c // send-only
    var recv <-chan int = c // receive-only
    ch1 :=(chan int)(send)// Error: cannot convert type chan<- int to type chan int
    ch2 :=(chan int)(recv)// Error: cannot convert type <-chan int to type chan int}
输出结果：

./main.go:8:19: cannot convert send (type chan<- int) to type chan int
./main.go:9:19: cannot convert recv (type <-chan int) to type chan int
```

## 14. channel传参

channel 是第一类对象，可传参 (内部实现为指针) 或者作为结构成员。

```go
$ cat ch8.go
package main
import "fmt"
type Request struct {
    data []int
    ret  chan int
}
func NewRequest(data ...int)*Request {
     return&Request{data,make(chan int,1)}
     }

func Process(req *Request){
    x :=0
    for _, i := range req.data {
        x += i
    }
    req.ret <- x
}
func main(){
    req :=NewRequest(10,20,30)
    Process(req)
    fmt.Println(<-req.ret)}
[root@localhost chan]#  go run ch8.go 
60
```
参考：
- [http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.4.4.md](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.4.4.md)
- [https://colobu.com/2016/04/14/Golang-Channels/](https://colobu.com/2016/04/14/Golang-Channels/)
