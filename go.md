#### 字符串切片

```
数组是属于值类型传递
对字符串切片后进行操作不会影响原字符串

```



实例化结构体

```go
    //命名字段初始化 不用按照位置顺序
	person := Person{
        Name:    "Alice",
        Country: "USA",
        Age:     25,
    }
```





#### 并发编程

```go
//通道（channel）  作为函数参数传递的时候 默认就是引用类型不用再去申明为c * chan inta
//如果未指定方向那么就是双向通道
ch <- v    // 把 v 发送到通道 ch
v := <-ch  // 从 ch 接收数据
           // 并把值赋给 v

//声明通道
ch := make(chan int)
//默认情况下，通道是不带缓冲区的。发送端发送数据，同时必须有接收端相应的接收数据。
//意思就是有发送端就要有接收端

//设置缓冲区  不设置默认为0没有缓冲会一直阻塞知道读取
ch := make(chan int , 2)


```





#### 管道

```go
// 定义声明管道
var intChan chan int
//make初始化管道
intChan = make(chan in ,100)



//声明只写 
var intChan2 chan <- int 
//声明只读
var intChan2 <- chan int 








```



### 创建项目

```
go mod init 项目名
```



环境安装

```
#安装goctl
go install github.com/zeromicro/go-zero/toos/goctl@latest
#安装protoc
goctl env check --install --verbose --force

#下载
go get -u github.com/zeromicro/go-zero@latest
```



proto生成代码

```
#proto生成  (.bat)  从执行目录开始写
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative ./person/person.proto

#多个的话就
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative ./person/person.proto
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative ./person2/person2.proto
```



```
goctl rpc protoc user.proto --go_out=types --go-grpc_out=types --zrpc_out=.



```



```
最后会生成一个user.go
运行go run user.go即可启动这个服务

注册信息在etc的yaml文件中例如ip地址 端口 key 返回的逻辑处理在internal/logic/ 下的代码文件函数中书写

```







### proto文件书写内容

```
syntax = "proto3";

package 其他proto引用这个proto时候的名字;

option go_package="mod包名开始写_同mod包名/proto所在的文件夹的名字;生成代码引用的名字"
//别名如果不写 默认是当前文件夹的名字
//go-zero的时候别写别名  前面的名字就是./proto的名字(不要加后缀以及其他的前缀 暂时不知道为什么
```



### 生成api

```
goctl api go -api user.api -dir .
```



### etcd

```
#注意安装后了之后要启动etcd 例如配置好环境变量后 随便一个cmd窗口输入etcd 这样将服务跑起来之后才能正常使用

//设置货更新值
etcdctl put name 张三

//获取值
etcdctl get name

//只获取value
etcdctl get name --print-value-only

//获取name前缀的键值对
etcdctl get --prefix name

//可结合使用例如 etcdctl get --prefix --print-value-only name   顺序无关

//删除键值对
etcdctl del name

//监听键的变化
etcdctl watch name
```







### gin

```

```



### go-zero

#### 环境安装

安装goctl 和 protoc    和 goland 安装goctl插件

```
go install github.com/zeromicro/go-zero/tools/goctl@latest
goctl env check --install --verbose --force
go get -u github.com/zeromicro/go-zero@latest
```

#### 快速创建

快速创建api

```
go 
```

快速创建rpc

```
goctl rpc new rpc的名字（一般取服务名字+rpc 例如user_rpc）
```



#### proto生成代码



注意所有的生成的无论是代码还是json都是例如TempaleTest 大驼峰命名法(会自动把下划线转为这个还有就是首字母大写)



生成grpc

```
				这个是文件路径 注意从运行的路径开始写而不是go.mod开始写
goctl rpc protoc user.proto --go_out=types --go-grpc_out-types --zrpc_out=.
```



生成api

```go

type LoginRequest{
    Username string `json:"username"`
}

type LoginResponse{
    Code int `json:"code"`
}

service users{
    @handler login //视图函数
    post /api/users/login (LoginRequest) returns (LoginResponse)

    @handler xxxx
    get /api/v1 returns ()
}
```

生成代码

```
goctl api go -api user.api -dir .
```

读取配置文件

```go
var configFile = flag.String("f", "etc/alipay_rpc.yaml", "the config file")//全局声明(也是说在package的主函数的main外面声明)

/*这个flag.String 指针地址存放的是 etc/alipay_rpc.yaml
func String(name string, value string, usage string) *string {
	return CommandLine.String(name, value, usage)
}

*/
func main() {
	flag.Parse()
	var c config.Config
	conf.MustLoad(*configFile, &c)
	fmt.Println(c.Name)
	fmt.Println(c.ListenOn)
	fmt.Println(c.AliPayConfig.AppId)
	fmt.Println(c.AliPayConfig.PrivateKey)
	fmt.Println(c.AliPayConfig.PublicKey)
}

```







### go相关技巧代码

#### go实现py装饰器



```go
package main

import (
	"fmt"
	"reflect"
)

type MyFunc interface{}

func middleware(f MyFunc) MyFunc {
	return func(args ...interface{}) {
		fmt.Println("在函数执行前做一些事情")

		funcValue := reflect.ValueOf(f)
		funcArgs := make([]reflect.Value, len(args))
		for i, arg := range args {
			funcArgs[i] = reflect.ValueOf(arg)
		}
		funcValue.Call(funcArgs)

		fmt.Println("在函数执行后做一些事情")
	}
}

func myFunction(s string) {
	fmt.Println("这是我的函数，参数为:", s)
}

func myFunction2(a int, b int) {
	fmt.Println("这是我的另一个函数，参数为:", a, b)
}

func main() {
	decoratedFunction := middleware(myFunction).(func(...interface{}))
	decoratedFunction("Hello, World!")

	decoratedFunction2 := middleware(myFunction2).(func(...interface{}))
	decoratedFunction2(1, 2)
}

```



