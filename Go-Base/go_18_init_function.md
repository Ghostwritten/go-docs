# Go init 函数

## 1. 介绍
golang中有个神奇的函数init,该函数会在所有程序执行开始前被调用，每个包可以包含多个init函数，所有被编辑器识别到的init函数都会在main函数执行前被调用。通常被用来注册一个程序需要使用的依赖，如mysql注册，配置文件加载等。
## 2. 作用

 - 初始化不能采用初始化表达式初始化的变量。
 - 程序运行前的注册。
 - 实现sync.Once功能。
 - 其他

## 3. 特点

 - init函数先于main函数自动执行，不能被其他函数调用；
 - init函数没有输入参数、返回值；
 - 每个包可以有多个init函数；
 - 包的每个源文件也可以有多个init函数，这点比较特殊；
 - 同一个包的init执行顺序，golang没有明确定义，编程时要注意程序不要依赖这个执行顺序。
 - 不同包的init函数按照包导入的依赖关系决定执行顺序。

## 4. 测试
golang程序初始化先于main函数执行，由runtime进行初始化，初始化顺序如下：

初始化导入的包（包的初始化顺序并不是按导入顺序（“从上到下”）执行的，runtime需要解析包依赖关系，没有依赖的包最先初始化，与变量初始化依赖关系类似，参见[golang变量的初始化](https://mp.weixin.qq.com/s/PGDzMaYznZVuDiO6V-zYDw)）；
初始化包作用域的变量（该作用域的变量的初始化也并非按照“从上到下、从左到右”的顺序，runtime解析变量依赖关系，没有依赖的变量最先初始化，参见[golang变量的初始化](https://mp.weixin.qq.com/s/PGDzMaYznZVuDiO6V-zYDw)）；
执行包的init函数；

```go
package main                                                                                                                     

import (
   "fmt"              
)

var T int64 = a()

func init() {
   fmt.Println("init in main.go ")
}

func a() int64 {
   fmt.Println("calling a()")
   return 2
}
func main() {                  
   fmt.Println("calling main")     
}
```

```go
[root@localhost init]# go run init2.go
calling a()
init in main.go 
calling main
```
**初始化顺序：变量初始化->init()->main()**

示例2：
pack.go
```go
package pack                                                                                                                     

import (
   "fmt"
   "test_util"
)

var Pack int = 6               

func init() {
   a := test_util.Util        
   fmt.Println("init pack ", a)    
} 
```
test_util.go
```go
test_util.go

package test_util                                                                                                                

import "fmt"

var Util int = 5

func init() {
   fmt.Println("init test_util")
}  
```
main.go

```go
package main                                                                                                                     

import (
   "fmt"
   "pack"
   "test_util"                
)

func main() {                  
   fmt.Println(pack.Pack)     
   fmt.Println(test_util.Util)
}
```
输出：

```go
init test_util
init pack  5
6
5
```
由于pack包的初始化依赖test_util，因此运行时先初始化test_util再初始化pack包；
示例3：
sandbox.go

```go
package main
import "fmt"
var _ int64 = s()
func init() {
   fmt.Println("init in sandbox.go")
}
func s() int64 {
   fmt.Println("calling s() in sandbox.go")
   return 1
}
func main() {
   fmt.Println("main")
}
```

a.go

```go
package main
import "fmt"
var _ int64 = a()
func init() {
   fmt.Println("init in a.go")
}
func a() int64 {
   fmt.Println("calling a() in a.go")
   return 2
}
```

z.go

```go
package main
import "fmt"
var _ int64 = z()
func init() {
   fmt.Println("init in z.go")
}
func z() int64 {
   fmt.Println("calling z() in z.go")
   return 3
}
```

```go
[root@localhost test2]# go run sandbox.go a.go z.go 
calling s() in sandbox.go
calling a() in a.go
calling z() in z.go
init in sandbox.go
init in a.go
init in z.go
main
```
同一个包不同源文件的init函数执行顺序，golang spec没做说明，以上述程序输出来看，执行顺序是源文件名称的字典序。
示例4：

```go
package main
import "fmt"
func init() {
   fmt.Println("init")
}
func main() {
   init()
}
```

**init函数不可以被调用，上面代码会提示：undefined: init**



示例5：

```go
package main
import "fmt"
func init() {
   fmt.Println("init 1")
}
func init() {
   fmt.Println("init 2")
}
func main() {
   fmt.Println("main")
}
```

输出：

```go
init 1
init 2
main
```

**init函数比较特殊，可以在包里被多次定义。**



示例6：

```go
var initArg [20]int
func init() {
   initArg[0] = 10
   for i := 1; i < len(initArg); i++ {
       initArg[i] = initArg[i-1] * 2
   }
}
```

**init函数的主要用途：初始化不能使用初始化表达式初始化的变量**

**示例7：**
```go
import _ "net/http/pprof"
```
golang对没有使用的导入包会编译报错，但是有时我们只想调用该包的init函数，不使用包导出的变量或者方法，这时就采用上面的导入方案。

执行上述导入后，init函数会启动一个异步协程采集该进程实例的资源占用情况，并以http服务接口方式提供给用户查询。


参考：
- [https://zhuanlan.zhihu.com/p/34211611](https://zhuanlan.zhihu.com/p/34211611)
