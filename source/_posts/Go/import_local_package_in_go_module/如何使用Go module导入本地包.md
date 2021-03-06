---
title: 如何使用go module导入本地包
tags:
  - go module
  - 导入本地包
categories: 开发环境准备
abbrlink: go_import_local
date: 2020-02-27 00:00:00
toc: true
---

`go module`是`Go1.11`版本之后官方推出的版本管理工具，并且从`Go1.13`版本开始，`go module`将是`Go`语言默认的依赖管理工具。到`Go1.14`版本推出之后`Go modules` 功能已经被正式推荐在生产环境下使用了。<!--more-->

本文详细介绍一下如何使用`go module`导入本地包。

## 前提

假设我们现在有`moduledemo`和`mypackage`两个包，其中`moduledemo`包中会导入`mypackage`包并使用它的`New`方法。

`mypackage/mypackage.go`内容如下：

```go
package mypackage

import "fmt"

func New(){
	fmt.Println("mypackage.New")
}
```

我们现在分两种情况讨论：

## 在同一个项目下

**注意**：在一个项目（project）下我们是可以定义多个包（package）的。

### 目录结构

现在的情况是，我们在`moduledemo/main.go`中调用了`mypackage`这个包。

```bash
moduledemo
├── go.mod
├── main.go
└── mypackage
    └── mypackage.go
```

### 导入包

这个时候，我们需要在`moduledemo/go.mod`中按如下定义：

```go
module moduledemo

go 1.16
```

然后在`moduledemo/main.go`中按如下方式导入`mypackage`

```go
package main

import (
	"fmt"
	"moduledemo/mypackage"  // 导入同一项目下的mypackage包
)
func main() {
	mypackage.New()
	fmt.Println("main")
}
```

### 举个例子

举一反三，假设我们现在有文件目录结构如下：

```bash
└── bubble
    ├── dao
    │   └── mysql.go
    ├── go.mod
    └── main.go
```

其中`bubble/go.mod`内容如下：

```go
module github.com/imw7/bubble

go 1.16
```

`bubble/dao/mysql.go`内容如下：

```go
package dao

import "fmt"

func New(){
	fmt.Println("mypackage.New")
}
```

`bubble/main.go`内容如下：

```go
package main

import (
	"fmt"
	"github.com/imw7/bubble/dao"
)
func main() {
	dao.New()
	fmt.Println("main")
}
```

## 不在同一个项目下

### 目录结构

```bash
├── moduledemo
│   ├── go.mod
│   └── main.go
└── mypackage
    ├── go.mod
    └── mypackage.go
```

### 导入包

这个时候，`mypackage`也需要进行`module`初始化，即拥有一个属于自己的`go.mod`文件，内容如下：

```go
module mypackage

go 1.16
```

然后我们在`moduledemo/main.go`中按如下方式导入：

```go
import (
	"fmt"
	"mypackage"
)
func main() {
	mypackage.New()
	fmt.Println("main")
}
```

因为这两个包不在同一个项目路径下，你想要导入本地包，并且这些包也没有发布到远程的`Github`或其他代码仓库地址。这个时候我们就需要在`go.mod`文件中使用`replace`指令。

在调用方也就是`moduledemo/go.mod`中按如下方式指定使用相对路径来寻找`mypackage`这个包。

```go
module moduledemo

go 1.16


require "mypackage" v0.0.0
replace "mypackage" => "../mypackage"
```

### 举个例子

最后我们再举个例子巩固下上面的内容。

我们现在有文件目录结构如下：

```bash
├── p1
│   ├── go.mod
│   └── main.go
└── p2
    ├── go.mod
    └── p2.go
```

`p1/main.go`中想要导入`p2.go`中定义的函数。

`p2/go.mod`内容如下：

```go
module imw7.com/eric/p2

go 1.16
```

`p1/main.go`中按如下方式导入

```go
import (
	"fmt"
	"imw7.com/eric/p2"
)
func main() {
	p2.New()
	fmt.Println("main")
}
```

因为我并没有把`imw7.com/eric/p2`这个包上传到`imw7.com`这个网站，我们只是想导入本地的包，这个时候就需要用到`replace`这个指令了。

`p1/go.mod`内容如下：

```go
module github.com/imw7/p1

go 1.16


require "imw7.com/eric/p2" v0.0.0
replace "imw7.com/eric/p2" => "../p2"
```

此时，我们就可以正常编译`p1`这个项目了。

被导入的包需要在`go.mod`文件里面详细写清楚`moudle`的路径，如`github.com/eric/p1`，不能只写`p1`。