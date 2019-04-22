## 写在前面  

在编译frp项目的时候，它依赖的golang相关库在下载时被墙了，导致编译中止。根据以往的经验，尝试手动把相关依赖`git clone`到`GOPATH`后继续编译，但是仍然失败。

打开源码目录，发现一个`go.mod`的文件，查了下资料才明白原来这是Go1.11的一个新特性——`Go Modules`，也就是**Go**的包依赖管理。  

## 快速开始  

在`GOPATH`目录外创建一个新目录：  

```  shell
$ mkdir -p /tmp/scratchpad/hello
$ cd /tmp/scratchpad/hello
```

初始化一个新模块：  

```  shell
$ go mod init github.com/you/hello
go: creating new go.mod: module github.com/you/hello
```

写段测试代码：  

```  shell
$ cat <<EOF > hello.go
package main

import (
    "fmt"
    "rsc.io/quote"
)

func main() {
    fmt.Println(quote.Hello())
}
EOF
```

编译：  

```  shell
$ go build
go: golang.org/x/text@v0.0.0-20170915032832-14c0d48ead0c: unrecognized import path "golang.org/x/text" (https fetch: Get https://golang.org/x/text?go-get=1: dial tcp 216.239.37.1:443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has
failed to respond.)
go: error loading module requirements
```

不妙，被墙了。幸好`Go Modules`支持`replace`命令，提供导入另一个路径，可以是位于VCS (GitHub或其他地方)中的另一个模块，或者是本地的绝对、相对路径。  

编辑`go.mod`文件，用github仓库代替golang：  

```  shell
cat go.mod
module github.com/kysonlok/hello

go 1.12

// Add replace directive in here
replace (
        golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c => github.com/golang/text v0.0.0-20170915032832-14c0d48ead0c
)
```

重新编译并运行：  

```   shell
$ go build
$ .\hello.exe
你好，世界。
```

`go.mod`文件会自动被更新，而且包含了依赖的明确依赖信息：  

```   shell
$ cat go.mod
module github.com/kysonlok/hello

go 1.12

replace golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c => github.com/golang/text v0.0.0-20170915032832-14c0d48ead0c

require rsc.io/quote v1.5.2
```

有了`Go Modules`以后，工作流应该变成这样：  

- 在.go文件用import导入依赖  

- 执行标准命令`go build`或者`go test`，将会自动更新`go.mod`，并且下载依赖  

- 当需要指定依赖版本的时候，可以执行诸如这样的命令`go get foo@v1.2.3`, `go get foo@master`, `go get foo@e3702bed2`，或者直接编辑`go.mod`文件  

简单介绍我们可能常会用到的功能：  

- `go list -m all` — 查看构建中直接或者间接依赖的packages的当前版本  
- `go list -u -m all` — 查看直接或者间接依赖的packages的最新版本  
- `go get -u` or `go get -u=patch` — 更新直接或者间接依赖的pakages到最新版本  
- `go build ./...` or `go test ./...` — 在模块的跟目录执行时，构建并且测试模块中的所有pakages  
- `go mod tidy` — 从`go.mod`中删除任何不再需要的依赖项  
- `replace` directive or `gohack` — 用另外一个版本代替自动推导的版本，可以是源于不同的repo  
- `go mod vendor` — 创建`vendor`目录，并且把直接或者间接依赖的packages复制到该目录下    

好了，先Mark到这里。  

## See also  

[Go Modules](https://github.com/golang/go/wiki/Modules)
