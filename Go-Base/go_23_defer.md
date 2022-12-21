#  Go defer 延迟调用

![](https://img-blog.csdnimg.cn/b090c1eb7c0b4e33bcd75952648c3ba5.png)

## 1. 简介
defer关键字的作用是当外围函数返回之后才执行被推迟的函数。在文件输入输出操作中经常可以见到
defer关键字，因为它使您不必记住何时关闭已打开的文件：defer关键字调用文件关闭函数关闭已打开
的文件时，可以紧靠着文件打开函数之后

## 2. 原则
defer函数在外围函数返回之后，以后进先出(LIFO)的原则执行。简单点
说，在一个外围函数中有3个defer函数： f1() 最先出现，然后 f2() ，最后 f3() ，当外围函数执行返回之后， f3() 最先被执行，接着是 f2() ，最后是 f1()
## 3. defer 特性

 - 关键字 defer 用于注册延迟调用。
 - 这些调用直到 return 前才被执。因此，可以用来做资源清理。
 - 多个defer语句，按先进后出的方式执行。
 - defer语句中的变量，在defer声明时就决定了。

## 4. defer 用途

 - 关闭文件句柄
 - 锁资源释放
 - 数据库连接释放

go语言 defer
go 语言的defer功能强大，对于资源管理非常方便，但是如果没用好，也会有陷阱。
defer 是先进后出
这个很自然,后面的语句会依赖前面的资源，因此如果先前面的资源先释放了，后面的语句就没法执行了

```go
$ cat defer1.go 
package main
import "fmt"
func main(){
    var whatever [5]struct{} 
    for i := range whatever {
        defer fmt.Println(i)}
}
[root@localhost defer]# go run defer1.go 
4
3
2
1
0
```
defer 碰上闭包

```go
$ cat defer2.go 
package main
import "fmt"
func main(){
    var whatever [5]struct{}
    for i := range whatever {
        defer func(){ fmt.Println(i)}()}}
[root@localhost defer]# go run defer2.go 
4
4
4
4
4
```
其实go说的很清楚,我们一起来看看go spec如何说的

```bash
Each time a “defer” statement executes, the function value and parameters to the call are evaluated as usualand saved anew but the actual function is not invoked.
```

也就是说函数正常执行,由于闭包用到的变量 i 在执行的时候已经变成4,所以输出全都是4.

defer f.Close

这个大家用的都很频繁,但是go语言编程举了一个可能一不小心会犯错的例子.

```go
$ cat defer3.go
package main
import "fmt"
type Test struct {
    name string
}
func (t *Test)Close(){
    fmt.Println(t.name," closed")
}
func main(){
    ts :=[]Test{{"a"},{"b"},{"c"}}
    for _, t := range ts {
        defer t.Close()
    }
}
[root@localhost defer]# go run defer3.go 
c  closed
c  closed
c  closed
```
这个输出并不会像我们预计的输出c b a,而是输出c c c

可是按照前面的go spec中的说明,应该输出c b a才对啊.

那我们换一种方式来调用一下.

```go
$ cat defer4.go
package main
import "fmt"
type Test struct {
    name string
}
func (t *Test)Close(){
    fmt.Println(t.name," closed")
}
func Close(t Test){
    t.Close()
}
func main(){
    ts :=[]Test{{"a"},{"b"},{"c"}}
    for _, t := range ts {
            defer Close(t)
    }
}
[root@localhost defer]# go run defer4.go 
c  closed
b  closed
a  closed
```
这个时候输出的就是c b a
当然,如果你不想多写一个函数,也很简单,可以像下面这样,同样会输出c b a
看似多此一举的声明

```go
$ cat defer5.go
package main
import "fmt"
type Test struct {
    name string
}
func (t *Test)Close(){
    fmt.Println(t.name," closed")
}
func main(){
    ts :=[]Test{{"a"},{"b"},{"c"}}
    for _, t := range ts {
        t2 := t
        defer t2.Close()
    }
}
[root@localhost defer]# go run defer5.go 
c  closed
b  closed
a  closed
```
通过以上例子，结合

Each time a “defer” statement executes, the function value and parameters to the call are evaluated as usualand saved anew but the actual function is not invoked.

这句话。可以得出下面的结论：

defer后面的语句在执行的时候，函数调用的参数会被保存起来，但是不执行。也就是复制了一份。但是并没有说struct这里的this指针如何处理，通过这个例子可以看出go语言并没有把这个明确写出来的this指针当作参数来看待。

多个 defer 注册，按 FILO 次序执行 ( 先进后出 )。哪怕函数或某个延迟调用发生错误，这些调用依旧会被执行。

```go
$ cat defer6.go
package main
func test(x int){
    defer println("a")
    defer println("b")
    defer func(){
        println(100/ x)// div0 异常未被捕获，逐步往外传递，最终终止进程。
    }()
    defer println("c")
}
func main(){
    test(0)
}
[root@localhost defer]# go run defer6.go 
c
b
a
panic: runtime error: integer divide by zero

goroutine 1 [running]:
main.test.func1(0x0)
	/root/go/defer/defer6.go:6 +0x6f
main.test(0x0)
	/root/go/defer/defer6.go:9 +0x140
main.main()
	/root/go/defer/defer6.go:11 +0x2a
exit status 2
```

*延迟调用参数在注册时求值或复制，可用指针或闭包 “延迟” 读取。(**未通过**)

```go
package main
func test() func() {
    x, y :=10,20
    defer func(i int){println("defer:", i, y)// y 闭包引用}(x)// x 被复制
    x +=10
    y +=100println("x =", x,"y =", y)
}
func main(){
    test()
}
```

输出结果:

```go
x =20 y =120
defer:10120
```

*滥用 defer 可能会导致性能问题，尤其是在一个 “大循环” 里。

```go
$ cat defer8.go 
package main
import (
"fmt"
"sync"
"time"
)
var lock sync.Mutex
func test(){
    lock.Lock()
    lock.Unlock()
}
func testdefer(){
    lock.Lock()
    defer lock.Unlock()
}
func main(){
    func(){
        t1 := time.Now()
        for i :=0; i <10000; i++{test()}
        elapsed := time.Since(t1)
        fmt.Println("test elapsed: ", elapsed)}()
        func(){
        t1 := time.Now()
        for i :=0; i <10000; i++{testdefer()}
        elapsed := time.Since(t1)
        fmt.Println("testdefer elapsed: ", elapsed)}()
}
[root@localhost defer]# go run defer8.go 
test elapsed:  238.665µs
testdefer elapsed:  428.412µs
```
## 5. defer 陷阱
defer 与 closure

```go
$ cat defer10.go 
package main
import (
    "errors"
    "fmt"
)
func foo(a, b int)(i int, err error){
    defer fmt.Printf("first defer err %v\n", err)
    defer func(err error){ 
    fmt.Printf("second defer err %v\n", err)
    }(err)
    defer func(){ 
    fmt.Printf("third defer err %v\n", err)}()
    if b ==0{
        err = errors.New("divided by zero!")
    return
    }
    i = a / b
    return}
func main(){
     foo(2,0)
}
[root@localhost defer]# go run defer10.go 
third defer err divided by zero!
second defer err <nil>
first defer err <nil>
```
解释：如果 defer 后面跟的不是一个 closure 最后执行的时候我们得到的并不是最新的值。

```go
$ cat defer11.go
package main
import "fmt"
func foo()(i int){
    i =0
    defer func(){
        fmt.Println(i)}()
        return 2
}
func main(){foo()}

[root@localhost defer]# go run defer11.go 
2
```
解释：在有具名返回值的函数中（这里具名返回值为 i），执行 return 2 的时候实际上已经将 i 的值重新赋值为 2。所以defer closure 输出结果为 2 而不是 1。

## 6. defer nil 函数

```go
$ cat  defer12.go
package main
import ("fmt")
func test(){
    var run func()= nil
    defer run()
    fmt.Println("runs")}
func main(){
    defer func(){if err :=recover(); err != nil {
            fmt.Println(err)}}()
    test()
}
[root@localhost defer]#  go run defer12.go
runs
runtime error: invalid memory address or nil pointer dereference
```
解释：名为 test 的函数一直运行至结束，然后 defer 函数会被执行且会因为值为 nil 而产生 panic 异常。然而值得注意的是，run() 的声明是没有问题，因为在test函数运行完成后它才会被调用。


## 7. 错误的位置使用 defer(未通过验证)

当 http.Get 失败时会抛出异常。

```go
package main
import "net/http"
func do() error {
    res, err := http.Get("http://www.google.com")
    defer res.Body.Close()if err != nil {return err
    }// ..code...return nil
}
func main(){do()}
```
输出结果：

```go
panic: runtime error: invalid memory address or nil pointer dereference
```

因为在这里我们并没有检查我们的请求是否成功执行，当它失败的时候，我们访问了 Body 中的空变量 res ，因此会抛出异常
解决方案

总是在一次成功的资源分配下面使用 defer ，对于这种情况来说意味着：当且仅当 `http.Get` 成功执行时才使用 defer。

参考：
- [http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch4.6.md](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch4.6.md)
