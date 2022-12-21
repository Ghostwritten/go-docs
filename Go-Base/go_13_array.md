#  Go array 数组

![](https://img-blog.csdnimg.cn/b9d24137765a450b9ac5486fb8e18897.png)


@[toc]

----
## 1. 数组特点

Golang Array和以往认知的数组有很大不同。

 - 数组：是同一种数据类型的固定长度的序列。
 - 数组定义：var a [len]int，比如：var a [5]int，数组长度必须是常量，且是类型的组成部分。一旦定义，长度不能变。
 - 长度是数组类型的一部分，因此，var a[5] int和var a[10]int是不同的类型。
 - 数组可以通过下标进行访问，下标是从0开始，最后一个元素下标是：len-1

```bash
 for i := 0; i < len(a); i++ {
 }
 for index, v := range a {
 }
```

 - 访问越界，如果下标在数组合法范围之外，则触发访问越界，会panic
 - 数组是值类型，赋值和传参会复制整个数组，而不是指针。因此改变副本的值，不会改变本身的值。
 - 支持 “==”、”!=” 操作符，因为内存总是被初始化过的。
 - 指针数组 [n]*T，数组指针 [n]T。

## 2. 数组初始化

### 2.1 一维数组
   

```bash
 全局：
    var arr0 [5]int = [5]int{1, 2, 3}
    var arr1 = [5]int{1, 2, 3, 4, 5}
    var arr2 = [...]int{1, 2, 3, 4, 5, 6}
    var str = [5]string{3: "hello world", 4: "tom"}
    局部：
    a := [3]int{1, 2}           // 未初始化元素值为 0。
    b := [...]int{1, 2, 3, 4}   // 通过初始化值确定数组长度。
    c := [5]int{2: 100, 4: 200} // 使用引号初始化元素。
    d := [...]struct {
        name string
        age  uint8
    }{
        {"user1", 10}, // 可省略元素类型。
        {"user2", 20}, // 别忘了最后一行的逗号。
    }
```

代码：

```bash
package main
import (
    "fmt"
)
var arr0 [5]int = [5]int{1, 2, 3}
var arr1 = [5]int{1, 2, 3, 4, 5}
var arr2 = [...]int{1, 2, 3, 4, 5, 6}
var str = [5]string{3: "hello world", 4: "tom"}
func main() {
    a := [3]int{1, 2}           // 未初始化元素值为 0。
    b := [...]int{1, 2, 3, 4}   // 通过初始化值确定数组长度。
    c := [5]int{2: 100, 4: 200} // 使用引号初始化元素。
    d := [...]struct {
        name string
        age  uint8
    }{
        {"user1", 10}, // 可省略元素类型。
        {"user2", 20}, // 别忘了最后一行的逗号。
    }
    fmt.Println(arr0, arr1, arr2, str)
    fmt.Println(a, b, c, d)
}
输出结果：

[1 2 3 0 0] [1 2 3 4 5] [1 2 3 4 5 6] [ hello world tom]

[1 2 0] [1 2 3 4] [0 0 100 0 200] [{user1 10} {user2 20}]
```

### 2.2 多维数组

```bash
全局
var arr0 [5][3]int
var arr1 [2][3]int = [...][3]int{{1, 2, 3}, {7, 8, 9}}
局部：
a := [2][3]int{{1, 2, 3}, {4, 5, 6}}
b := [...][2]int{{1, 1}, {2, 2}, {3, 3}} // 第 2 纬度不能用 "..."。
代码：

package main
import (
    "fmt"
)
var arr0 [5][3]int
var arr1 [2][3]int = [...][3]int{{1, 2, 3}, {7, 8, 9}}
func main() {
    a := [2][3]int{{1, 2, 3}, {4, 5, 6}}
    b := [...][2]int{{1, 1}, {2, 2}, {3, 3}} // 第 2 纬度不能用 "..."。
    fmt.Println(arr0, arr1)
    fmt.Println(a, b)
}
输出结果：

[[0 0 0] [0 0 0] [0 0 0] [0 0 0] [0 0 0]] [[1 2 3] [7 8 9]]

[[1 2 3] [4 5 6]] [[1 1] [2 2] [3 3]]
```

值拷贝行为会造成性能问题，通常会建议使用 `slice`，或数组指针。

```bash
package main
import (
    "fmt"
)
func test(x [2]int) {
    fmt.Printf("x: %p\n", &x)
    x[1] = 1000
}
func main() {
    a := [2]int{}
    fmt.Printf("a: %p\n", &a)
    test(a)
    fmt.Println(a)
}
输出结果:

a: 0xc42007c010

x: 0xc42007c030

[0 0]
```

内置函数 len 和 cap 都返回数组长度 (元素数量)。

```bash
package main
func main() {
    a := [2]int{}
    println(len(a), cap(a)) 
}
输出结果：

2 2
```

### 2.3 多维数组遍历

```bash
package main
import (
    "fmt"
)
func main() {
    var f [2][3]int = [...][3]int{{1, 2, 3}, {7, 8, 9}}
    for k1, v1 := range f {
        for k2, v2 := range v1 {
            fmt.Printf("(%d,%d)=%d ", k1, k2, v2)
        }
        fmt.Println()
    }
}
输出结果：

(0,0)=1 (0,1)=2 (0,2)=3

(1,0)=7 (1,1)=8 (1,2)=9
```
