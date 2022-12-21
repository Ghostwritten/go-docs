#  Go map容器引用
![](https://img-blog.csdnimg.cn/4b5aca71eb07400ba48d3331cf256576.png)


Golang Map：引用类型，哈希表。一堆键值对的未排序集合。

键必须是支持相等运算符 (“==”、”!=”) 类型， 如 number、string、 pointer、array、struct，以及对应的 interface。值可以是任意类型，没有限制。

## 1. map声明

声明map的语法如下

```go
var map变量名 map[key] value
```

其中：key为键类型，value为值类型
例如：value不仅可以是标注数据类型，也可以是自定义数据类型

```go
package main
type personInfo struct {
    ID      string
    Name    string
    Address string
}
var m1 map[string]int
var m2 map[string]personInfo
func main(){}
```

## 2. map初始化

### 2.1 直接初始化（创建）

```go
package main
import ("fmt")
var m1 map[string]float32 = map[string]float32{"C":5,"Go":4.5,"Python":4.5,"C++":2}
func main(){
    m2 := map[string]float32{"C":5,"Go":4.5,"Python":4.5,"C++":2}
    m3 := map[int]struct {
        name string
        age  int
    }{1:{"user1",10},// 可省略元素类型。2:{"user2",20},}
    fmt.Printf("全局变量 map m1 : %v\n", m1)
    fmt.Printf("局部变量 map m2 : %v\n", m2)
    fmt.Printf("局部变量 map m3 : %v\n", m3)}
输出结果：

全局变量 map m1 : map[Python:4.5 C++:2 C:5 Go:4.5]
局部变量 map m2 : map[C++:2 C:5 Go:4.5 Python:4.5]
局部变量 map m3 : map[2:{user2 20}1:{user1 10}]
```

注意：由m1,m2可以看出map是键值对的无序集合。

### 2.2 通过make初始化（创建）

Go语言提供的内置函数make()可以用于灵活地创建map。

预先给 make 函数一个合理元素数量参数，有助于提升性能。因为事先申请一大块内存，可避免后续操作时频繁扩张。

```go
package main
import ("fmt")
func main(){// 创建了一个键类型为string,值类型为int的map
    m1 :=make(map[string]int)// 也可以选择是否在创建时指定该map的初始存储能力，如创建了一个初始存储能力为5的map
    m2 :=make(map[string]int,5)
    m1["a"]=1
    m2["b"]=2
    fmt.Printf("局部变量 map m1 : %v\n", m1)
    fmt.Printf("局部变量 map m2 : %v\n", m2)}
输出结果：

局部变量 map m1 : map[a:1]
局部变量 map m2 : map[b:2]
```

## 3. map操作
插入、更新、查找、删除、判断是否存在、求长度

```go
package main
import ("fmt")
func main(){
    m := map[string]string{"key0":"value0","key1":"value1"}
    fmt.Printf("map m : %v\n", m)//map插入
    m["key2"]="value2"
    fmt.Printf("inserted map m : %v\n", m)//map修改
    m["key0"]="hello world!"
    fmt.Printf("updated map m : %v\n", m)//map查找
    val, ok := m["key0"]if ok {
        fmt.Printf("map's key0 is %v\n", val)}// 长度：获取键值对数量。
    len :=len(m)
    fmt.Printf("map's len is %v\n", len)// cap 无效，error// cap := cap(m)    //invalid argument m (type map[string]string) for cap// fmt.Printf("map's cap is %v\n", cap)// 判断 key 是否存在。if val, ok = m["key"];!ok {
        fmt.Println("map's key is not existence")}// 删除，如果 key 不存在，不会出错。if val, ok = m["key1"]; ok {delete(m,"key1")
        fmt.Printf("deleted key1 map m : %v\n", m)}}
输出结果：

map m : map[key0:value0 key1:value1]
inserted map m : map[key0:value0 key1:value1 key2:value2]
updated map m : map[key0:hello world! key1:value1 key2:value2]
map's key0 is hello world!
map's len is 3
map's key is not existence
deleted key1 map m : map[key0:hello world! key2:value2]
```

## 4. map遍历
不能保证迭代返回次序，通常是随机结果，具体和版本实现有关。
package main

```go
import ("fmt")
func main(){
    m :=make(map[int]int)
    for i :=0; i <10; i++{
        m[i]= i
    }
    fmt.Println(m)
    fmt.Println(m)
    for j :=0; j <2; j++{
        fmt.Println("---------------------")
        for k, v := range m {
            fmt.Printf("key -> value : %v -> %v\n", k, v)
        }
     }
 }

输出结果：
map[0:0 1:1 2:2 3:3 4:4 5:5 6:6 7:7 8:8 9:9]
map[0:0 1:1 2:2 3:3 4:4 5:5 6:6 7:7 8:8 9:9]
---------------------
key -> value : 2 -> 2
key -> value : 3 -> 3
key -> value : 4 -> 4
key -> value : 7 -> 7
key -> value : 0 -> 0
key -> value : 1 -> 1
key -> value : 5 -> 5
key -> value : 6 -> 6
key -> value : 8 -> 8
key -> value : 9 -> 9
---------------------
key -> value : 4 -> 4
key -> value : 7 -> 7
key -> value : 2 -> 2
key -> value : 3 -> 3
key -> value : 5 -> 5
key -> value : 6 -> 6
key -> value : 8 -> 8
key -> value : 9 -> 9
key -> value : 0 -> 0
key -> value : 1 -> 1
```
## 5. slice与map操作（slice of map）

```go
package main
import ("fmt")
func main(){
    items :=make([]map[int]int,5)
    for i :=0; i <5; i++{
        items[i]=make(map[int]int)
    }
    items[0][0]=0
    items[1][2]=3
    fmt.Println(items)}
[root@localhost map]# go run map2.go 
[map[0:0] map[2:3] map[] map[] map[]]
```

## 6. map排序

```go
cat map3.go 
package main
import ("fmt";"sort")
func main(){
    m := map[string]string{"q":"q","w":"w","e":"e","r":"r","t":"t","y":"y"}
    var slice []string
    for k, _ := range m {
        slice =append(slice, k)}
    fmt.Printf("clise string is : %v\n", slice)
    sort.Strings(slice[:])
    fmt.Printf("sorted slice string is : %v\n", slice)
    for _, v := range slice {
        fmt.Println(m[v])
    }
}
[root@localhost map]# go run map3.go 
clise string is : [w e r t y q]
sorted slice string is : [e q r t w y]
e
q
r
t
w
y
```
## 7. map反转
cat map4.go 

```go
package main
import ("fmt")
func main(){
    m := map[int]string{1:"x",2:"w",3:"e",4:"r",5:"t",6:"y"}
    fmt.Println(m)
    m_rev :=make(map[string]int)
    for k, v := range m {
        m_rev[v]= k
    }
    fmt.Println(m_rev)
}
[root@localhost map]# go run map4.go 
map[1:x 2:w 3:e 4:r 5:t 6:y]
map[e:3 r:4 t:5 w:2 x:1 y:6]
```
## 8. 容器和结构体（map and struct）
cat map5.go 

```go
package main
import "fmt"
func main(){
    type user struct{ name string }
    m := map[int]user{1:{"user1"},}
    fmt.Println(m)
    u := m[1]
    u.name ="Tom"
    m[1]= u 
    fmt.Println(m)
    m2 := map[int]*user{1:&user{"user1"},}
    m2[1].name ="Jack"
    fmt.Println(m2)
}
[root@localhost map]# go run map5.go 
map[1:{user1}]
map[1:{Tom}]
map[1:0xc000010230]
```
可以在迭代时安全删除键值。但如果期间有新增操作，那么就不知道会有什么意外了
cat map6.go 

```go
package main
import "fmt"
func main(){
        for i :=0; i <5; i++{
        m := map[int]string{0:"a",1:"a",2:"a",3:"a",4:"a",5:"a",6:"a",7:"a",8:"a",9:"a",}
        for k := range m {
            m[k+k]="x"
            delete(m, k)
        }
        fmt.Println(m)}
}
[root@localhost map]# go run map6.go 
map[2:x 6:x 8:x 10:x 12:x 16:x 18:x 28:x]
map[4:x 10:x 12:x 16:x 28:x 36:x]
map[4:x 12:x 14:x 16:x 18:x 20:x]
map[2:x 8:x 10:x 12:x 18:x 28:x 32:x]
map[8:x 10:x 12:x 14:x 18:x 32:x]
```

参考：
- [http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.4.3.md](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.4.3.md)
