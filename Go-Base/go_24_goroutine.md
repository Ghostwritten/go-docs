#  Go 了解 goroutine 运用

![](https://img-blog.csdnimg.cn/43ea5b2f387d4a7e95261d9c30abe69e.png)

## 1. goroutine 原理
在 Go 语言中，通过协程和管道实现了 `Communicating Sequential Processes, CSP` 模型，两者承担了通信和同步中的重要角色。

## 2. CSP并发模型
`CSP`模型是上个世纪七十年代提出的，用于描述两个独立的并发实体通过共享的通讯 channel(管道)进行通信的并发模型。 CSP中channel是第一类对象，它不关注发送消息的实体，而关注与发送消息时使用的channel。

`Golang` 就是借用CSP模型的一些概念为之实现并发进行理论支持，其实从实际上出发，go语言并没有，完全实现了CSP模型的所有理论，仅仅是借用了 `process`和`channel`这两个概念。process是在go语言上的表现就是 `goroutine` 是实际并发执行的实体，每个实体之间是通过channel通讯来实现数据共享。

Golang中使用 CSP中 `channel` 这个概念。channel 是被单独创建并且可以在进程之间传递，它的通信模式类似于 boss-worker 模式的，一个实体通过将消息发送到channel 中，然后又监听这个 channel 的实体处理，两个实体之间是匿名的，这个就实现实体中间的解耦，其中 channel 是同步的一个消息被发送到 channel 中，最终是一定要被另外的实体消费掉的，在实现原理上其实是一个阻塞的消息队列。

`Goroutine` 是实际并发执行的实体，它底层是使用协程(coroutine)实现并发，coroutine是一种运行在用户态的用户线程，类似于 greenthread，go底层选择使用coroutine的出发点是因为，它具有以下特点：

 1. 用户空间 避免了内核态和用户态的切换导致的成本
 2. 可以由语言和框架层进行调度
 3. 更小的栈空间允许创建大量的实例

## 3. Goroutine 调度器

golang使用goroutine做为最小的执行单位，但是这个执行单位还是在用户空间，实际上最后被处理器执行的还是内核中的线程。
**用户线程和内核线程的调度方法有：**
多个用户线程对应一个内核线程
![](https://img-blog.csdnimg.cn/20200307144705223.png)
一个用户线程对应一个内核线程
![](https://img-blog.csdnimg.cn/20200307144732806.png)
用户线程和内核线程是多对多的对应关系
![](https://img-blog.csdnimg.cn/20200307144821540.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
golang 通过为goroutine提供语言层面的调度器，来实现了高效率的`M:N`线程对应关系
![](https://img-blog.csdnimg.cn/20200307145532594.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
支撑整个调度器的主要有4个重要结构，分别是`M、G、P、Sched`，前三个定义在`runtime.h`中，Sched定义在`proc.c`中
 - M：是内核线程
 - P : 是调度协调，用于协调M和G的执行，内核线程只有拿到了P才能对goroutine继续调度执行，一般都是通过限定P的个数来控制golang的并发度，P的数量可以通过GOMAXPROCS()来设置，它其实也就代表了真正的并发度，即有多少个goroutine可以同时运行。
图中灰色的那些goroutine并没有运行，而是出于ready的就绪态，正在等待被调度。P维护着这个队列（称之为runqueue），
Go语言里，启动一个goroutine很容易：go function 就行，所以每有一个go语句被执行，runqueue队列就在其末尾加入一个
goroutine，在下一个调度点，就从runqueue中取出（如何决定取哪个goroutine？）一个goroutine执行。
 - G : 是待执行的goroutine，包含这个goroutine的栈空间
 - Gn : 灰色背景的Gn 是已经挂起的goroutine，它们被添加到了执行队列中，然后需要等待网络IO的goroutine，当P通过 epoll查询到特定的fd的时候，会重新调度起对应的，正在挂起的goroutine。
 
 Golang为了调度的公平性，在调度器加入了steal working 算法 ，在一个P自己的执行队列，处理完之后，它会先到全局的执行队列中偷G进行处理，如果没有的话，再会到其他P的执行队列中抢G来进行处理。

当一个OS线程M0陷入阻塞时（如下图)，P转而在运行M1，图中的M1可能是正被创建，或者从线程缓存中取出。
![](https://img-blog.csdnimg.cn/202003071521378.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
 当MO返回时，它必须尝试取得一个P来运行goroutine，一般情况下，它会从其他的OS线程那里拿一个P过来，
如果没有拿到的话，它就把goroutine放在一个global runqueue里，然后自己睡眠（放入线程缓存里）。所有的P也会周期性的检查global runqueue并运行其中的goroutine，否则global runqueue上的goroutine永远无法执行。

另一种情况是P所分配的任务G很快就执行完了（分配不均），这就导致了这个处理器P很忙，但是其他的P还有任务，此时如果global runqueue没有任务G了，那么P不得不从其他的P里拿一些G来执行。一般来说，如果P从其他的P那里要拿任务的话，一般就拿run queue的一半，这就确保了每个OS线程都能充分的使用，如下图：
![](https://img-blog.csdnimg.cn/20200307152311236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
参考地址：[http://morsmachine.dk/go-scheduler](http://morsmachine.dk/go-scheduler)


## 4. 方法
设置goroutine运行的CPU数量，最新版本的go已经默认已经设置了。

```go
num := runtime.NumCPU()    //获取主机的逻辑CPU个数
runtime.GOMAXPROCS(num)    //设置可同时执行的最大CPU数
```

```go
$ cat go1.go 
package main

import (
    "fmt"
    "time"
)

func cal(a int , b int)  {
    c := a+b
    fmt.Printf("%d + %d = %d\n",a,b,c)
}

func main() {
    for i :=0 ; i<10 ;i++{
        go cal(i,i+1)
    }
    time.Sleep(time.Second * 2)
} 
[root@localhost goroutine]# go run go1.go 
0 + 1 = 1
1 + 2 = 3
2 + 3 = 5
3 + 4 = 7
4 + 5 = 9
8 + 9 = 17
9 + 10 = 19
5 + 6 = 11
6 + 7 = 13
7 + 8 = 15
```

## 5. goroutine异常捕捉
当启动多个goroutine时，如果其中一个goroutine异常了，并且我们并没有对进行异常处理，那么整个程序都会终止，所以我们在编写程序时候最好每个goroutine所运行的函数都做异常处理，异常处理采用`recover`

```go
package main

import (
    "fmt"
    "time"
)

func addele(a []int ,i int)  {
    defer func() {    //匿名函数捕获错误
        err := recover()
        if err != nil {
            fmt.Println("add ele fail")
        }
    }()
   a[i]=i
   fmt.Println(a)
}

func main() {
    Arry := make([]int,4)
    for i :=0 ; i<10 ;i++{
        go addele(Arry,i)
    }
    time.Sleep(time.Second * 2)
}
//结果
add ele fail
[0 0 0 0]
[0 1 0 0]
[0 1 2 0]
[0 1 2 3]
add ele fail
add ele fail
add ele fail
add ele fail
add ele fail
```

## 6. 同步的goroutine(?)

由于goroutine是异步执行的，那很有可能出现主程序退出时还有goroutine没有执行完，此时goroutine也会跟着退出。此时如果想等到所有goroutine任务执行完毕才退出，go提供了`sync`包和`channel`来解决同步问题，当然如果你能预测每个goroutine执行的时间，你还可以通过`time.Sleep`方式等待所有的groutine执行完成以后在退出程序(如上面的列子)。

示例一：使用sync包同步goroutine
sync大致实现方式
`WaitGroup` 等待一组`goroutinue`执行完毕. 主程序调用 `Add` 添加等待的goroutinue数量. 每个goroutinue在执行结束时调用 `Done` ，此时等待队列数量减1.，主程序通过`Wait阻塞`，直到等待队列为0.

```go
package main

import (
    "fmt"
    "sync"
)

func cal(a int , b int ,n *sync.WaitGroup)  {
    c := a+b
    fmt.Printf("%d + %d = %d\n",a,b,c)
    defer n.Done() //goroutinue完成后, WaitGroup的计数-1

}

func main() {
    var go_sync sync.WaitGroup //声明一个WaitGroup变量
    for i :=0 ; i<10 ;i++{
        go_sync.Add(1) // WaitGroup的计数加1
        go cal(i,i+1,&go_sync)  
    }
    go_sync.Wait()  //等待所有goroutine执行完毕
}
//结果
9 + 10 = 19
0 + 1 = 1
1 + 2 = 3
2 + 3 = 5
3 + 4 = 7
4 + 5 = 9
5 + 6 = 11
6 + 7 = 13
7 + 8 = 15
8 + 9 = 17
```

示例二：通过channel实现goroutine之间的同步。

实现方式：通过channel能在多个groutine之间通讯，当一个goroutine完成时候向channel发送退出信号,等所有goroutine退出时候，利用for循环channe去channel中的信号，若取不到数据会阻塞原理，等待所有goroutine执行完毕，使用该方法有个前提是你已经知道了你启动了多少个goroutine。

```go
package main

import (
    "fmt"
    "time"
)

func cal(a int , b int ,Exitchan chan bool)  {
    c := a+b
    fmt.Printf("%d + %d = %d\n",a,b,c)
    time.Sleep(time.Second*2)
    Exitchan <- true
}

func main() {

    Exitchan := make(chan bool,10)  //声明并分配管道内存
    for i :=0 ; i<10 ;i++{
        go cal(i,i+1,Exitchan)
    }
    for j :=0; j<10; j++{   
         <- Exitchan  //取信号数据，如果取不到则会阻塞
    }
    close(Exitchan) // 关闭管道
}
```

## 7. goroutine之间的通讯(?)

goroutine本质上是协程，可以理解为不受内核调度，而受go调度器管理的线程。goroutine之间可以通过channel进行通信或者说是数据共享，当然你也可以使用全局变量来进行数据共享。

示例：使用channel模拟消费者和生产者模式

```go
package main

import (
    "fmt"
    "sync"
)

func Productor(mychan chan int,data int,wait *sync.WaitGroup)  {
    mychan <- data
    fmt.Println("product data：",data)
    wait.Done()
}
func Consumer(mychan chan int,wait *sync.WaitGroup)  {
     a := <- mychan
    fmt.Println("consumer data：",a)
     wait.Done()
}
func main() {

    datachan := make(chan int, 100)   //通讯数据管道
    var wg sync.WaitGroup

    for i := 0; i < 10; i++ {
        go Productor(datachan, i,&wg) //生产数据
        wg.Add(1)
    }
    for j := 0; j < 10; j++ {
        go Consumer(datachan,&wg)  //消费数据
        wg.Add(1)
    }
    wg.Wait()
}
//结果
consumer data： 4
product data： 5
product data： 6
product data： 7
product data： 8
product data： 9
consumer data： 1
consumer data： 5
consumer data： 6
consumer data： 7
consumer data： 8
consumer data： 9
product data： 2
consumer data： 2
product data： 3
consumer data： 3
product data： 4
consumer data： 0
product data： 0
product data： 1
```
参考：
- [https://www.cnblogs.com/wdliu/p/9272220.html](https://www.cnblogs.com/wdliu/p/9272220.html)
