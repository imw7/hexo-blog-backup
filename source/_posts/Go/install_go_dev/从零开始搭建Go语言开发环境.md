---
title: 安装Go语言及搭建Go语言开发环境
tags: 跨平台编译
categories: 开发环境准备
abbrlink: go_install_dev
date: 2021-03-20 00:00:00
toc: true
---

最新1.16版本，一步一步，从零搭建Go语言开发环境。<!--more-->

**注意：**Go语言1.14版本之后推荐使用go modules管理以来，也不再需要把代码写在GOPATH目录下了。

## 下载

### 下载地址

Go官网下载地址：https://golang.org/dl/

Go官方镜像站（推荐）：https://golang.google.cn/dl/

### 版本的选择

Windows平台和Mac平台推荐下载可执行文件版，Linux平台下载压缩文件版。

**下图中的版本号可能并不是最新的，但总体来说安装教程是类似的。Go语言更新迭代比较快，推荐使用较新版本，体验最新特性。**

![download1](https://imw7.github.io/images/Go/install_go_dev/download1.png)

## 安装

### Windows安装

此安装实例以 `64位Win10`系统安装 `Go1.16可执行文件版本`为例。

将上一步选好的安装包下载到本地。

![download2](https://imw7.github.io/images/Go/install_go_dev/download2.png)

双击下载好的文件，然后按照下图的步骤安装即可。

![install01](https://imw7.github.io/images/Go/install_go_dev/install01.png)

![install02](https://imw7.github.io/images/Go/install_go_dev/install02.png)

![install03](https://imw7.github.io/images/Go/install_go_dev/install03.png)

![install04](https://imw7.github.io/images/Go/install_go_dev/install04.png)

![install05](https://imw7.github.io/images/Go/install_go_dev/install05.png)

### Linux下安装

如果不是要在`Linux`平台敲`Go`代码就不需要在`Linux`平台安装`Go`，我们开发机上写好的`Go`代码只需要跨平台编译（详见文章末尾的跨平台编译）好之后就可以拷贝到`Linux`服务器上运行了，这也是`Go`程序跨平台易部署的优势。

我们在版本选择页面选择并下载好`go1.16.linux-amd64.tar.gz`文件：

```bash
wget https://dl.google.com/go/go1.16.linux-amd64.tar.gz
```

将下载好的文件解压到`/usr/local`目录下：

```bash
tar -zxvf go1.16.linux-amd64.tar.gz -C /usr/local  # 解压
```

如果提示没有权限，加上`sudo`以`root`用户的身份再运行。执行完就可以在`/usr/local/`下看到`go`目录了。

配置环境变量： `Linux`下有两个文件可以配置环境变量，其中`/etc/profile`是对所有用户生效的；`$HOME/.profile`是对当前用户生效的，根据自己的情况自行选择一个文件打开，添加如下两行代码，保存退出。

```bash
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
```

修改`/etc/profile`后要重启生效，修改`$HOME/.profile`后使用`source`命令加载`$HOME/.profile`文件即可生效。 检查：

```bash
~ go version
go version go1.16 linux/amd64
```

### Mac下安装

下载可执行文件版，直接点击**下一步**安装即可，默认会将`Go`安装到`/usr/local/go`目录下。

![Mac安装Go](https://imw7.github.io/images/Go/install_go_dev/mac_install_go.png)

### 检查

上一步安装过程执行完毕后，可以打开终端窗口，输入`go version`命令，查看安装的`Go`版本。

![install06](https://imw7.github.io/images/Go/install_go_dev/install06.png)

## GOROOT和GOPATH

`GOROOT`和`GOPATH`都是环境变量，其中`GOROOT`是我们安装go开发包的路径，而从`Go 1.8`版本开始，`Go`开发包在安装完成后会为`GOPATH`设置一个默认目录，参见下表。

**GOPATH在不同操作系统平台上的默认值**

|  平台   |   GOPATH默认值   |        举例        |
| :-----: | :--------------: | :----------------: |
| Windows | %USERPROFILE%/go | C:\Users\用户名\go |
|  Unix   |     $HOME/go     |  /home/用户名/go   |

可以通过以下方法更改默认的`GOPATH`目录：

![env01](https://imw7.github.io/images/Go/install_go_dev/env01.png)

![env02](https://imw7.github.io/images/Go/install_go_dev/env02.png)

![env03](https://imw7.github.io/images/Go/install_go_dev/env03.png)

![env04](https://imw7.github.io/images/Go/install_go_dev/env04.png)

我们只需要记住默认的`GOPATH`路径在哪里就可以了，并且默认情况下 `GOROOT`下的`bin`目录及`GOPATH`下的`bin`目录都已经添加到环境变量中了，我们也不需要额外配置了。自定义只需将 `GOROOT`下的`bin`目录及`GOPATH`下的`bin`目录添加到系统变量的`Path`下即可。

![env05](https://imw7.github.io/images/Go/install_go_dev/env05.png)

### GOPROXY

`Go1.14`版本之后，都推荐使用`go mod`模式来管理依赖环境了，也不再强制我们把代码必须写在`GOPATH`下面的`src`目录了，你可以在你电脑的任意位置编写`Go`代码。（网上有些教程适用于1.11版本之前。）

默认`GOPROXY`配置是：`GOPROXY=https://proxy.golang.org,direct`，由于国内访问不到`https://proxy.golang.org`，所以我们需要换一个`PROXY`，这里推荐使用`https://goproxy.io`或`https://goproxy.cn`。

可以执行下面的命令修改`GOPROXY`：

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

## Go开发编辑器

`Go`采用的是`UTF-8`编码的文本文件存放源代码，理论上使用任何一款文本编辑器都可以做`Go`语言开发，这里推荐使用`VS Code`和`Goland`。 `VS Code`是微软开源的编辑器，而`Goland`是`JetBrains`出品的付费`IDE`。

我们这里使用`VS Code` 加插件做为`Go`语言的开发工具。

### VS Code介绍

`VS Code`全称`Visual Studio Code`，是微软公司开源的一款**免费**现代化轻量级代码编辑器，支持几乎所有主流的开发语言的语法高亮、智能代码补全、自定义热键、括号匹配、代码片段、代码对比 Diff、GIT 等特性，支持插件扩展，支持 `Win`、`Mac` 以及 `Linux`平台。

虽然不如某些`IDE`功能强大，但是它添加`Go`扩展插件后已经足够胜任我们日常的`Go`开发。

### 下载与安装

`VS Code`官方下载地址：https://code.visualstudio.com/Download

三大主流平台都支持，请根据自己的电脑平台选择对应的安装包。![vscode_home](https://imw7.github.io/images/Go/install_go_dev/vscode_home.png)双击下载好的安装文件，双击安装即可。

### 配置

#### 安装中文简体插件

点击左侧菜单栏最后一项`管理扩展`，在`搜索框`中输入`chinese` ，选中结果列表第一项，点击`install`安装。

安装完毕后右下角会提示`重启VS Code`，重启之后你的`VS Code`就显示中文啦！

![安装简体中文插件](https://imw7.github.io/images/Go/install_go_dev/vscode1.gif)`VSCode`主界面介绍：

![vscode_menu](https://imw7.github.io/images/Go/install_go_dev/vscode_menu1.png)

#### 安装Go扩展

现在我们要为我们的`VS Code`编辑器安装`Go`扩展插件，让它支持`Go`语言开发。

![安装go扩展图](https://imw7.github.io/images/Go/install_go_dev/vscode_plugin1.png)

## 第一个Go程序

### Hello World

现在我们来创建第一个`Go`项目——`hello`。在我们桌面创建一个`hello`目录。

在该目录中创建一个`main.go`文件：

```go
package main  // 声明 main 包，表明当前是一个可执行程序

import "fmt"  // 导入内置 fmt 包

func main(){  // main函数，是程序执行的入口
	fmt.Println("Hello World!")  // 在终端打印 Hello World!
}
```

### go build

`go build`表示将源代码编译成可执行文件。

在`hello`目录下执行：

```bash
go build
```

或者在其他目录执行以下命令：

```bash
go build hello
```

`Go`编译器会去 `GOPATH`的`src`目录下查找你要编译的`hello`项目

编译得到的可执行文件会保存在执行编译命令的当前目录下，如果是`windows`平台会在当前目录下找到`hello.exe`可执行文件。

可在终端直接执行该`hello.exe`文件：

```bash
c:\desktop\hello>hello.exe
Hello World!
```

我们还可以使用`-o`参数来指定编译后得到的可执行文件的名字。

```bash
go build -o helloworld.exe
```

### Windows下VSCode切换cmd.exe作为默认终端

如果你打开`VS Code`的终端界面出现如下图场景（注意观察红框圈中部分），那么你的`VS Code`此时正使用`powershell`作为默认终端：![vscode shell配置1](https://imw7.github.io/images/Go/install_go_dev/vscode_shell1.png)十分推荐你按照下面的步骤，选择`cmd.exe`作为默认的终端工具：![vscode shell配置2](https://imw7.github.io/images/Go/install_go_dev/vscode_shell2.png)此时，`VS Code`正上方中间位置会弹出如下界面，参照下图挪动鼠标使光标选中后缀为`cmd.exe`的那一个，然后点击鼠标左键。

最后**重启`VS Code`中已经打开的终端**或者**直接重启`VS Code`**就可以了。![vscode shell配置3](https://imw7.github.io/images/Go/install_go_dev/vscode_shell3.png)如果没有出现下拉三角，也没有关系，按下`Ctrl+Shift+P`，`VS Code`正上方会出现一个框，你按照下图输入`shell`，然后点击指定选项即可出现上面的界面了。![vscode shell配置4](https://imw7.github.io/images/Go/install_go_dev/vscode_shell4.png)

*补充说明：由于`VS Code`对`go mod`模式的支持暂时还不够完善，建议大家使用`Goland`编辑器。*

### go install

`go install`表示安装的意思，它先编译源代码得到可执行文件，然后将可执行文件移动到`GOPATH`的`bin`目录下。因为我们的环境变量中配置了`GOPATH`下的`bin`目录，所以我们就可以在任意地方直接执行可执行文件了。

### 跨平台编译

默认我们`go build`的可执行文件都是当前操作系统可执行的文件，如果我想在`windows`下编译一个`linux`下可执行文件，那需要怎么做呢？

只需要指定目标操作系统的平台和处理器架构即可：

```bash
SET CGO_ENABLED=0  // 禁用CGO
SET GOOS=linux  // 目标平台是linux
SET GOARCH=amd64  // 目标处理器架构是amd64
```

***使用了`cgo`的代码是不支持跨平台编译的***

然后再执行`go build`命令，得到的就是能够在`Linux`平台运行的可执行文件了。

`Mac `下编译 `Linux` 和` Windows`平台 `64位` 可执行程序：

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build
```

`Linux` 下编译 `Mac` 和 `Windows` 平台`64位`可执行程序：

```bash
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build
```

`Windows`下编译`Mac`平台`64位`可执行程序：

```bash
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build
```

好了，完成。人生苦短，let’s Go。

