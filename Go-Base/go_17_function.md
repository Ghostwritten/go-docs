#  Go 了解函数运用

![](https://img-blog.csdnimg.cn/2d42a75886ed4967ba2f4a75e5560d49.png)


## 1. 函数特点

```bash
• 无需声明原型。
• 支持不定 变参。
• 支持多返回值。
• 支持命名返回参数。 
• 支持匿名函数和闭包。
• 函数也是一种类型，一个函数可以赋值给变量。
• 不支持 嵌套 (nested) 一个包不能有两个名字一样的函数。
• 不支持 重载 (overload) 
• 不支持 默认参数 (default parameter)。
```

## 2. 函数参数
### 2.1 参数传递
函数定义时指出，函数定义时有参数，该变量可称为函数的形参。形参就像定义在函数体内的局部变量。

但当调用函数，传递过来的变量就是函数的实参，函数可以通过两种方式来传递参数：

**值传递**：指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。

```go
  func swap(x, y int) int {......}
```

**引用传递：**是指在调用函数时将实际参数的地址传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。

**arg1.go**
```go
package main
import ("fmt")/* 定义相互交换值的函数 */
func swap(x, y *int){
    var temp int    
    temp = *x /* 保存 x 的值 */
    *x = *y   /* 将 y 值赋给 x */
    *y = temp /* 将 temp 值赋给 y*/
    }
func main(){
    var a, b int =1,2
    /*
        调用 swap() 函数
        &a 指向 a 指针，a 变量的地址
        &b 指向 b 指针，b 变量的地址
    */
    swap(&a,&b)
    fmt.Println(a, b)
}
```

输出结果：

```bash
21
```

在默认情况下，Go 语言使用的是值传递，即在调用过程中不会影响到实际参数。

**注意1**：无论是值传递，还是引用传递，传递给函数的都是变量的副本，不过，值传递是值的拷贝。引用传递是地址的拷贝，一般来说，地址拷贝更为高效。而值拷贝取决于拷贝的对象大小，对象越大，则性能越低。

**注意2**：map、slice、chan、指针、interface默认以引用的方式传递。

### 2.2 不定参数传值
就是函数的参数不是固定的，后面的类型是固定的。（可变参数）

Golang `可变参数本质上就是 slice`。只能有一个，且必须是最后一个。

在参数赋值时可以不用用一个一个的赋值，可以直接传递一个数组或者切片，特别注意的是在参数后加上“…”即可。

 

```go
 func myfunc(args ...int){//0个或多个参数}
 func add(a int, args…int) int {//1个或多个参数}
 func add(a int, b int, args…int) int {//2个或多个参数}
```

**注意**：其中args是一个slice，我们可以通过arg[index]依次访问所有参数,通过len(arg)来判断传递参数的个数.

**任意类型的不定参数：**

就是函数的参数和每个参数的类型都不是固定的。

用interface{}传递任意类型数据是Go语言的惯例用法，而且interface{}是类型安全的。

```go
  func myfunc(args ...interface{}){}
```

代码：
**arg2.go**
```go
package main
import ("fmt")
func test(s string, n ...int) string {
    var x int
    for _, i := range n {
        x += i
    }
    return fmt.Sprintf(s, x)
}
func main(){
    println(test("sum: %d",1,2,3))
}
```

输出结果：

```bash
sum:6
```

使用 `slice` 对象做变参时，必须展开。（slice...）
**arg3.go**
```go
package main
import ("fmt")
func test(s string, n ...int) string {
    var x int
    for _, i := range n {
        x += i
    }
    return fmt.Sprintf(s, x)
}
func main(){
    s :=[]int{1,2,3}
    res :=test("sum: %d", s...)// slice... 展开slice
    println(res)
}
```
## 3. 函数返回值
命名返回值
`"_"`标识符，用来忽略函数的某个返回值

Go 的返回值可以被命名，并且就像在函数体开头声明的变量那样使用。

返回值的名称应当具有一定的意义，可以作为文档使用。

没有参数的 return 语句返回各个返回变量的当前值。这种用法被称作“裸”返回。

直接返回语句仅应当用在像下面这样的短函数中。在长的函数中它们会影响代码的可读性。
**return1.go**
```go
package main
import ("fmt")
func add(a, b int)(c int){
    c = a + b
    return
}
func calc(a, b int)(sum int, avg int){
    sum = a + b
    avg =(a + b)/2
    return
}
func main(){
    var a, b int =1,2
    c :=add(a, b)
    sum, avg :=calc(a, b)
    fmt.Println(a, b, c, sum, avg)
}
```

输出结果：

```bash
12331
```

Golang返回值不能用容器对象接收多返回值。只能用多个变量，或 “_” 忽略。
**return2.go**
```go
package main
func test()(int, int){
    return1,2
}
func main(){
    // s := make([]int, 2)
    // s = test()   // Error: multiple-value test() in single-value context
    x, _ :=test()println(x)
}
```

输出结果：

```bash
1
```

多返回值可直接作为其他函数调用实参。
**return3.go**
```go
package main
func test()(int, int){
    return1,2
}
func add(x, y int) int {
    return x + y
}
func sum(n ...int) int {
    var x int
    for _, i := range n {
        x += i
    }
    return x
}
func main(){
    println(add(test()))
    println(sum(test()))
}
```

输出结果：

```go
3
3
```

命名返回参数可看做与形参类似的局部变量，最后由 `return 隐式返回`。
**return4.go**
```go
package main
func add(x, y int)(z int){
    z = x + y
    return}
func main(){
    println(add(1,2))
}
```

输出结果：

```go
3
```

命名返回参数可被同名局部变量遮蔽，此时`需要显式返回`。
**return5.go**
```go
func add(x, y int)(z int){
    {// 不能在一个级别，引发 "z redeclared in this block" 错误。
            var z = x + y
            // return   // Error: z is shadowed during returnreturn z // 必须显式返回。
    }
}
```

命名返回参数允许 defer 延迟调用通过闭包读取和修改。
**return6.go**
```go
package main
func add(x, y int)(z int){
    defer func(){
        z +=100}()
    z = x + y
    return
}
func main(){
    println(add(1,2))
}
```

输出结果：

```bash
103
```

显式 return 返回前，会先修改命名返回参数。
**return6.go**
```go
package main
func add(x, y int)(z int){
    defer func(){println(z)// 输出: 203}()
    z = x + y
    return z +200// 执行顺序: (z = z + 200) -> (call defer) -> (return)
}
func main(){
    println(add(1,2))// 输出: 203
}
```

输出结果：

```bash
203
203
```

## 4. 匿名函数

匿名函数是指不需要定义函数名的一种函数实现方式。1958年LISP首先采用匿名函数。

在Go里面，函数可以像普通变量一样被传递或使用，Go语言支持随时在代码里定义匿名函数。

**匿名函数由一个不带函数名的函数声明和函数体组成。匿名函数的优越性在于可以直接使用函数内的变量，不必申明。**
**anon1.go**
```go
package main
import ("fmt""math")
func main(){
    getSqrt :=func(a float64) float64 {
        return math.Sqrt(a)
    }
    fmt.Println(getSqrt(4))
}
```

输出结果：

```go
2
```

上面先定义了一个名为getSqrt 的变量，初始化该变量时和之前的变量初始化有些不同，使用了func，func是定义函数的，可是这个函数和上面说的函数最大不同就是没有函数名，也就是匿名函数。这里将一个函数当做一个变量一样的操作。

**Golang匿名函数可赋值给变量，做为结构字段，或者在 channel 里传送**。
**anon2.go**

```go
package main
func main() {
    // --- function variable ---
    fn := func() { println("Hello, World!") }
    fn()
    // --- function collection ---
    fns := [](func(x int) int){
        func(x int) int { return x + 1 },
        func(x int) int { return x + 2 },
    }
    println(fns[0](100))
    // --- function as field ---
    d := struct {
        fn func() string
    }{
        fn: func() string { return "Hello, World!" },
    }
    println(d.fn())
    // --- channel of function ---
    fc := make(chan func() string, 2)
    fc <- func() string { return "Hello, World!" }
    println((<-fc)())
}
```

输出结果：

```go
Hello, World!101
Hello, World!
Hello, World!
```

## 5. 函数闭包
闭包的应该都听过，但到底什么是闭包呢？

闭包是由函数及其相关引用环境组合而成的实体(即：闭包=函数+引用环境)。

“官方”的解释是：所谓“闭包”，指的是一个拥有许多变量和绑定了这些变量的环境的表达式（通常是一个函数），因而这些变量也是该表达式的一部分。

维基百科讲，闭包（Closure），`是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外`。所以，有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的实体。闭包在运行时可以有多个实例，不同的引用环境和相同的函数组合可以产生不同的实例。

看着上面的描述，会发现`闭包和匿名函数似乎有些像`。可是可能还是有些云里雾里的。因为跳过闭包的创建过程直接理解闭包的定义是非常困难的。目前在JavaScript、Go、PHP、Scala、Scheme、Common Lisp、Smalltalk、Groovy、Ruby、 Python、Lua、objective c、Swift 以及Java8以上等语言中都能找到对闭包不同程度的支持。通过支持闭包的语法可以发现一个特点，他们都有垃圾回收(GC)机制。

javascript应该是普及度比较高的编程语言了，通过这个来举例应该好理解写。看下面的代码，只要关注script里方法的定义和调用就可以了。

```go
<!DOCTYPE html>
<html lang="zh">
<head>
    <title></title>
</head>
<body> 
</body>
</html>
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.2.6/jquery.min.js" type="text/javascript"></script>
<script>
function a(){
    var i=0;
    function b(){
        console.log(++i);
        document.write("<h1>"+i+"</h1>");
    }
    return b;
}
$(function(){
    var c=a();
    c();
    c();
    c();
    //a(); //不会有信息输出
    document.write("<h1>=============</h1>");
    var c2=a();
    c2();
    c2();
});
</script>
```

这段代码有两个特点：

 - 函数b嵌套在函数a内部
 - 函数a返回函数b

这样在`执行完var c=a()后，变量c实际上是指向了函数b()，再执行函数c()后就会显示i的值，第一次为1，第二次为2，第三次为3`，以此类推。

其实，这段代码就创建了一个闭包。`因为函数a()外的变量c引用了函数a()内的函数b()`，就是说：

**当函数a()的内部函数b()被函数a()外的一个变量引用的时候，就创建了一个闭包。**

在上面的例子中，`由于闭包的存在使得函数a()返回后，a中的i始终存在，这样每次执行c()，i都是自加1后的值。`

**从上面可以看出闭包的作用就是在a()执行完并返回后，闭包使得Javascript的垃圾回收机制GC不会收回a()所占用的资源，因为a()的内部函数b()的执行需要依赖a()中的变量i。**

在给定函数被多次调用的过程中，这些**私有变量能够保持其持久性**。变量的作用域仅限于包含它们的函数，因此无法从其它程序代码部分进行访问。不过，变量的生存期是可以很长，在一次函数调用期间所创建所生成的值在下次函数调用时仍然存在。**正因为这一特点，闭包可以用来完成信息隐藏，并进而应用于需要状态表达的某些编程范型中。**

下面来想象另一种情况，如果a()返回的不是函数b()，情况就完全不同了。因为a()执行完后，b()没有被返回给a()的外界，只是被a()所引用，而此时a()也只会被b()引 用，因此函数a()和b()互相引用但又不被外界打扰（被外界引用），函数a和b就会被GC回收。所以直接调用a();是页面并没有信息输出。

下面来说闭包的另一要素引用环境。c()跟c2()引用的是不同的环境，在调用i++时修改的不是同一个i，因此两次的输出都是1。函数a()每进入一次，就形成了一个新的环境，对应的闭包中，函数都是同一个函数，环境却是引用不同的环境。这和c()和c()的调用顺序都是无关的。

Go的闭包
Go语言是支持闭包的，这里只是简单地讲一下在Go语言中闭包是如何实现的。

下面我来将之前的JavaScript的闭包例子用Go来实现。
**bibao1.go**
```go
package main
import (
    "fmt"
)
func a() func() int {
    i := 0
    b := func() int {
        i++
        fmt.Println(i)
        return i
    }
    return b
}
func main() {
    c := a()
    c()
    c()
    c()
    a() //不会输出i
}
```

输出结果：

```go
1
2
3
```

可以发现，输出和之前的JavaScript的代码是一致的。具体的原因和上面的也是一样的，这说明Go语言是支持闭包的。

`闭包复制的是原对象指针`，这就很容易解释`延迟引用现象`。
**bibao2.go**
```go
package main
import "fmt"
func test() func() {
    x := 100
    fmt.Printf("x (%p) = %d\n", &x, x)
    return func() {
        fmt.Printf("x (%p) = %d\n", &x, x)
    }
}
func main() {
    f := test()
    f()
}
```

输出:

```go
x (0xc42007c008)=100
x (0xc42007c008)=100
```

在汇编层 ，test 实际返回的是 `FuncVal 对象`，其中包含了`匿名函数地址、闭包对象指针`。当调 匿名函数时，只需以某个寄存器传递该对象即可。

FuncVal { func_address, closure_var_pointer ...}

## 6. 递归函数

**递归，就是在运行的过程中调用自己。**

一个函数调用自己，就叫做递归函数。

构成递归需具备的条件：

子问题须与原始问题为同样的事，且更为简单。
不能无限制地调用本身，须有个出口，化简为非递归状况处理。
数字阶乘

一个正整数的阶乘（factorial）是所有小于及等于该数的正整数的积，并且0的阶乘为1。自然数n的阶乘写作n!。1808年，基斯顿·卡曼引进这个表示法。

```go
package main
import "fmt"
func factorial(i int) int {
    if i <= 1 {
        return 1
    }
    return i * factorial(i-1)
}
func main() {
    var i int = 7
    fmt.Printf("Factorial of %d is %d\n", i, factorial(i))
}
```

输出结果：

```bash
Factorial of 7 is 5040
```

斐波那契数列(Fibonacci)

这个数列从第3项开始，每一项都等于前两项之和。

```go
package main
import "fmt"
func fibonaci(i int) int {
    if i == 0 {
        return 0
    }
    if i == 1 {
        return 1
    }
    return fibonaci(i-1) + fibonaci(i-2)
}
func main() {
    var i int
    for i = 0; i < 10; i++ {
        fmt.Printf("%d\n", fibonaci(i))
    }
}
输出结果：

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
```

参考：
- [http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch4.5.md](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch4.5.md)
