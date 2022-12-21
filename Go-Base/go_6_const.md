#  Go const 定义常量

![](https://img-blog.csdnimg.cn/5469b9d199be45c7954fff3a3a471a0e.png)


## 1. 前言
常量的定义与变量类似，只不过使用 `const` 关键字，表示只读不能修改。

```go
const World = “世界”
```

常量值必须是编译期可确定的字符、字符串、布尔或数字类型的值。

常量不能使用 “:=” 语法定义。**加粗样式**

数值常量

数值常量是高精度的值 。

一个未指定类型的常量由上下文来决定其类型。

（int 可以存放最大64位的整数，根据平台不同有时会更少。）

## 2. 常量初始化

```go
package main
const x, y int = 1, 2     // 多常量初始化
const s = "Hello, World!" // 类型推断
const ( //常量组
    a, b      = 10, 100
    c    bool = false
)
func main() {
    const str = "xxx" // 未使用的局部常量不会引发编译错误。
}
```

**在常量组中，如不提供类型和初始化值，那么视作与上一个常量相同。**

```go
package main
import (
    "fmt"
)
const (
    s = "abc"
    x // x = "abc"
)
func main() {
    fmt.Println(s)
    fmt.Println(x)
}
```

输出结果：

```go
abc
abc
```

常量值还可以是 `len、cap、unsafe.Sizeof` 等编译期可确定结果的函数返回值。

```go
const (
    a = "abc"
    b = len(a)
    c = unsafe.Sizeof(b)
)
```

如果常量类型足以存储初始化值，那么不会引发溢出错误。

```go
package main
import (
    "fmt"
    "unsafe"
)
const (
    a = "hello world"
    b = len(a)
    c = unsafe.Sizeof(b)
)
func main() {
    fmt.Println(a, b, c)
}
```

输出结果：

```go
hello world 11 8
```

## 3. 枚举

iota 可以被用作枚举值：
iota，特殊常量，可以认为是一个可以被编译器修改的常量。
在每一个const关键字出现时，被重置为0，然后再下一个const出现之前，每出现一次iota，其所代表的数字会自动增加1。

**关键字 iota 定义常量组中从 0 开始按行计数的自增枚举值。**

```go
package main
import (
    "fmt"
)
const (
    Sunday = iota
    Monday //通常省略后续行表达式
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
)
func main() {
    fmt.Println(Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday)
}
输出结果：

0 1 2 3 4 5 6
```

```go
package main
import (
    "fmt"
)
const (
    _        = iota // iota = 0
    KB int64 = 1 << (10 * iota)
    MB       // iota=1
    GB       // 与 KB 表达式相同，但 iota = 2
    TB
)
func main() {
    fmt.Println(KB, MB, GB, TB)
}
输出结果：

1024 1048576 1073741824 1099511627776
```

在同一常量组中，可以提供多个 iota，它们各自增长。

```go
package main
import (
    "fmt"
)
const (
    A, B = iota, iota << 10 //0,0<<10
    C, D                    // 1, 1 << 10
)
func main() {
    fmt.Println(A, B, C, D)
}
输出结果：

0 0 1 1024
```

**如果 iota 自增被打断，须显式恢复。**

```go
package main
import (
    "fmt"
)
const (
    A = iota //0
    B        // 1
    C = "c"  //c
    D        // c，与上  相同。
    E = iota // 4，显式恢复。注意计数包含了 C、D 两 。
    F        // 5
)
func main() {
    fmt.Println(A, B, C, D, E, F)
}
输出结果：

0 1 c c 4 5
```

可通过自定义类型来实现枚举类型限制。

```go
package main
type Color int
const (
    Black Color = iota
    Red
    Blue
)
func test(c Color) {}
func main() {
    c := Black
    test(c)
    // x := 1
    // test(x)
    // ./main.go:18:6: cannot use x (type int) as type Color in argument to test
    test(1) // 常量会被编译器自动转换。
}
```
参考：
- [http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.2.md](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.2.md)
