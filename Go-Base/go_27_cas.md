# Go CAS操作

go中的Cas操作与java中类似，都是借用了CPU提供的原子性指令来实现。CAS操作修改共享变量时候不需要对共享变量加锁，而是通过类似乐观锁的方式进行检查，本质还是不断的占用CPU 资源换取加锁带来的开销（比如上下文切换开销）。下面一个例子使用CAS来实现计数器

```go
package main
import (
    "fmt"
    "sync"
    "sync/atomic"
)

var (
    counter int32          //计数器
    wg      sync.WaitGroup //信号量
)

func main() {

    threadNum := 5

    //1. 五个信号量
    wg.Add(threadNum)

    //2.开启5个线程
    for i := 0; i < threadNum; i++ {
        go incCounter(i)
    }

    //3.等待子线程结束
    wg.Wait()
    fmt.Println(counter)
}

func incCounter(index int) {
    defer wg.Done()

    spinNum := 0
    for {
        //2.1原子操作
        old := counter
        ok := atomic.CompareAndSwapInt32(&counter, old, old+1)
        if ok {
            break
        } else {
            spinNum++
        }
    }

    fmt.Printf("thread,%d,spinnum,%d\n",index,spinNum)

}
```

```go
[root@localhost cas]# go run test1.go 
thread,4,spinnum,0
thread,0,spinnum,0
thread,1,spinnum,0
thread,2,spinnum,0
thread,3,spinnum,0
5
```

 - 如上代码main线程首先创建了5个信号量，然后开启五个线程执行incCounter方法 incCounter内部执行代码2.1
   使用cas操作递增counter的值，
 - atomic.CompareAndSwapInt32具有三个参数，第一个是变量的地址，第二个是变量当前值，第三个是要修改变量为多少，该函数如果发现传递的old值等于当前变量的值，则使用第三个变量替换变量的值并返回true，否则返回false。
 - 这里之所以使用无限循环是因为在高并发下每个线程执行CAS并不是每次都成功，失败了的线程需要重写获取变量当前的值，然后重新执行CAS操作。读者可以把线程数改为10000或者更多会发现输出thread,5329,spinnum,1其中1说明该线程尝试了两个CAS操作，第二次才成功。
