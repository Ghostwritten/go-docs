#  Go 快速入门

![在这里插入图片描述](https://img-blog.csdnimg.cn/92d770fef292450ca52337824584e9aa.png)


## 1. go 背景
Go语言由来自Google公司的Robert Griesemer，Rob Pike和Ken Thompson三位大牛于2007年9月开始设计和实现，然后于2009年的11月对外正式发布。
## 2. go 特点 
- 并发与协程

- 基于消息传递的通信方式

- 丰富实用的内置数据类型

- 函数多返回值

- defer机制

- 反射(reflect)

- 高性能HTTP Server

- 工程管理

- 编程规范
- 
## 3. go 安装
###  3.1 windows go
- 下载 [go 1.19.4](https://go.dev/dl/go1.19.4.windows-amd64.msi)

1. 点击安装

2. 配置环境变量
我的电脑右击---> 高级系统配置-----> 环境变量配置

   - 配置 `GOROOT`为安装目录位置，我的是`D:\install\go1.19.4`
   - 配置 命令路径 `PATH` ：`D:\install\go1.19.4\bin`


### 3.2 UNIX/Linux/Mac OS X, 和 FreeBSD 安装
以下介绍了在UNIX/Linux/Mac OS X, 和 FreeBSD系统下使用源码安装方法：
- 官网地址：[https://studygolang.com/dl](https://studygolang.com/dl)
- [Centos 8.2 配置 go 项目开发环境](https://blog.csdn.net/xixihahalelehehe/article/details/127652839)

```bash
$ wget https://studygolang.com/dl/golang/go1.13.8.linux-amd64.tar.gz
$ tar -C /usr/local -xzf go1.4.linux-amd64.tar.gz
```

3、将 `/usr/local/go/bin` 目录添加至PATH环境变量：

```bash
$ vim /etc/profile
#根目录
export GOROOT=/usr/local/go
#bin目录
export GOBIN=$GOROOT/bin
#工作目录
export GOPATH=/usr/local/gopath
export PATH=$PATH:$GOPATH:$GOBIN:$GOROOT
```

保存并重启

```bash
$ . /etc/profile
$ go env
```



## 4. go 学习网站
web应用开发
- [build-web-application-with-golang](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/preface.md)
- [go语言圣经](https://books.studygolang.com/gopl-zh/index.html)

## 5. go 开始
hello world

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, 世界")
}
```
运行.go文件
```bash
go run helloworld.go
Hello, 世界
```
命令生成一个名为helloworld的可执行的二进制文件

```bash
go build helloworld.go
```

```bash
$ ./helloworld
Hello, 世界
```
获取/编译/安装
```bash
go get gopl.io/ch1/helloworld
```
