#如何写Go代码

* 介绍
* 代码组织结构
  * 综述 
  * 工作空间
  * GOPATH环境变量
  * 导入路径
  * 你的第一个程序
  * 你的第一个库
  * 包名
* 测试
* 远程包
* 下一步
* 获得帮助

## > 介绍

这个文档阐述了如何开发一个简单的Go包，并介绍了go tool，
标准方式获取，构建，安装Go包和命令
go tool需要按指定的方式来组织你的代码。请仔细阅读文档，
它讲解了最简单的使用Go installation建造和运行的方式。

一个类似的屏幕录像讲解[screencast](https://www.youtube.com/watch?v=XCsL89YtqCs)

## > 代码组织结构
### 综述
* Go程序员一般都把所有的Go代码放在一个工作空间下
* 工作空间包含了很多版本控制库（例如，由Git管理）
* 每一个库都包含了一个或多个包
* 每个包由一个目录下的一个或多个Go源代码文件
* 到包目录的路径决定了他的import路径
注意这些不同于其他编程环境的，其他编程环境的每一个
项目都有一个独立的工作空间并且紧密的和版本控制库绑定在一起

## > 工作空间
工作空间就是一个有三个目录在顶层的目录层级
* src 包含了Go源码文件
* pkg 包含了包对象
* bin 包含了可执行命令（文件）

go tool构建源码包并且把二进制文件结果安装到pkg目录和bin目录

src子目录通常包含多个版本控制库（诸如Git或者Mercurial）用来跟踪用户的
一个或多个源码包

为了给你一个对现实中工作空间直观的认识，这里有一个例子
```
bin/
    hello                          # command executable
    outyet                         # command executable
pkg/
    linux_amd64/
        github.com/golang/example/
            stringutil.a           # package object
src/
    github.com/golang/example/
        .git/                      # Git repository metadata
	hello/
	    hello.go               # command source
	outyet/
	    main.go                # command source
	    main_test.go           # test source
	stringutil/
	    reverse.go             # package source
	    reverse_test.go        # test source
    golang.org/x/image/
        .git/                      # Git repository metadata
	bmp/
	    reader.go              # package source
	    writer.go              # package source
    ... (many more repositories and packages omitted) ...
   
```

这棵树展示了一个包含两个版本库的工作空间（example和image）。 example库
包含了两个命令（hello和outyet），image库包含了bmp包和一些其他的。

一个典型的工作空间包含很多源码库，这些源码库包含了很多的包和命令。大多数
Go程序员都把他们所有的源代码和依赖放到一个工作空间中。
命令和库文件来源于不同的源码和包的构建。我们稍后讨论他们的不同处。

## > GOPATH 环境变量
GOPATH 环境变量指定了工作空间的位置。他是你做Go开发时唯一要设置的环境变量。
开始使用Go时，相应地创建一个工作空间目录并设置GOPATH。你的工作空间可以是任何
你喜欢的位置，但是我们在这篇文档中使用$HOME/work。注意的是这个位置不能和你的Go
 安装路径一样。(另外一个通用设置是设置GOPATH=$HOME)

```
$ mkdir $HOME/work
$ export GOPATH=$HOME/work
```
方便起见，把工作空间的子目录bin放到PATH中

```
$ export PATH=$PATH:$GOPATH/bin
```

想学更多的关于GOPATH环境变量设置的知识，请看 [go help gopath](https://golang.org/cmd/go/#hdr-GOPATH_environment_variable)

##导入路径
一个导入路径是唯一标识一个包的字符串。一个包的导入路径对应于它在工作区内或远程存储库中的位置（下面说明）。

标准库中的包给短的进口路径如“fmt”和“net/ HTTP”。对于您自己的包，您必须选择一个不与未来添加到标准库
或其他外部库的基础路径进行冲突的基本路径。

如果你把你的代码保存在某个源代码库中的某个地方，那么你应该使用该源库的根作为你的基本路径。例如，
如果你在github.com/user有Github帐号，这应该是你的基本路径。

注意，在你构建之前，你不需要将你的代码发布到一个远程存储库。如果你有一天会发布它，这是一个很好的组织你的代码的习惯，
在实践中，你可以选择任何任意的路径名称，只要它是唯一的标准库和更大的去生态系统。
我们将使用github.com/user作为我们的基本路径。在您的工作区中创建一个目录，以保持源代码：

```
$ mkdir -p $GOPATH/src/github.com/user
```
## 你的第一个程序

为了编译并运行一个简单的程序，第一步选择一个包路径（我将用 github.com/user/hello) 并且在
工作空间中创建一个相应的包目录

```
$ mkdir $GOPATH/src/github.com/user/hello
```

下一步，在此目录下，创建一个叫“hello.go”的文件，包含以下代码

```
package main

import "fmt"

func main() {
	fmt.Printf("Hello, world.\n")
}
```

现在你可以使用go tool构建并安装这个程序了。

```
$ go install github.com/user/hello
```
记住你可以在你的系统里的任何地方运行这个命令。go tool通过GOPATH指定的工作空间里的 github.com/user/hello包来
查找源代码。
如果你在当前包目录下运行go install你也可以忽略包路径
```
$ cd $GOPATH/src/github.com/user/hello
$ go install
```
这个命令构建了hello命令，创建了一个可执行二进制文件。然后把这个二进制文件安装到工作空间中
的bin目录下叫做hello（在windows中叫hello.exe）。在这个例子中路径是$GOPATH/bin/hello, 就是
$home/work/bin/hello.

Go只在发生错误的时候打印信息，所以如果这些命令没有打印输出那么他们就是执行成功了。
你现在可以通过他的完整路径在命令行运行这个程序了
```
$ $GOPATH/bin/hello
Hello, world.
```
或者，如果你已经添加$GOPATH/bin到你的PATH路径中,那么只需打出二进制名称
```
$ hello
Hello, world.
```
如果你是用代码管理软件，那么现在是时候初始化一个库，增加一些文件，并提交第一个变化。
当然，这一步是可选的，你不一定需要使用代码管理软件来写Go代码
```
$ cd $GOPATH/src/github.com/user/hello
$ git init
Initialized empty Git repository in /home/user/work/src/github.com/user/hello/.git/
$ git add hello.go
$ git commit -m "initial commit"
[master (root-commit) 0b4507d] initial commit
 1 file changed, 1 insertion(+)
  create mode 100644 hello.go
```
推送代码到远程库就留个读者作为练习来完成

## 你的第一个库
让我们来写一个库，并在hello程序中使用它。
好了， 第一步是选择一个包路径（我们使用github.com/user/stringutil)并且创建包目录
```
$ mkdir $GOPATH/src/github.com/user/stringutil
```
下一步，在目录中创建一个叫做reverse.go的文件，包含以下内容
```
// Package stringutil contains utility functions for working with strings.
package stringutil

// Reverse returns its argument string reversed rune-wise left to right.
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```
现在，使用go来编译这个包来测试一下
```
$ go build github.com/user/stringutil
```
或者，如果你正在源码目录下工作，只需如下
```
$ go build
```
这不会创建任何输出文件。想要输出文件，你必须使用go install，这个命令会把包对象放到工作空间的pkg目录

确认stringutil包正确构建后，修改你的hello.go源文件（他在$GOPATH/src/github.com/user/hello)如下
```
package main

import (
	"fmt"

	"github.com/user/stringutil"
)

func main() {
	fmt.Printf(stringutil.Reverse("!oG ,olleH"))
}
```
无论go tool何时安装一个包或者二进制文件，他也会安装他所有的依赖。所以，当你安装hello程序时，
```
$ go install github.com/user/hello
```
stringutil包也被自动的安装。
运行新的版本，你将会看到一个新的反转的信息
```
$ hello
Hello, Go!
```
完成以上步骤后，你的工作空间应该像这样
```
bin/
    hello                 # command executable
pkg/
    linux_amd64/          # this will reflect your OS and architecture
        github.com/user/
            stringutil.a  # package object
src/
    github.com/user/
        hello/
            hello.go      # command source
        stringutil/
            reverse.go    # package source
```
注意go install把stringutil.a对象放在pkg/linux_amd64目录中映射源文件的目录下。这是因为未来go tool的调用能够
找到这个包对象并且避免了重编译这个包的必要。这个linux_amd64部分有助于交叉编译，并且反映了操作系统和你的系统架构。

Go 命令是以静态链接方式执行的，运行Go程序时包对象不需要存在。
## 包名
Go源代码中的第一个语句必须是
```
package name
```
这里*name*是包的默认名称，用于（其他包）导入（所有在这个包里的文件必须使用相同的*name*）

Go 的惯例是包名就是最导入路径的最后一个项。导入包"crypto/rot13"应该被命名为rot13
可执行命令也必须总是使用包名
没有要求包的名字在多个包链接到一个二进制文件时必须是唯一的，只是导入的包路径（他们的完整文件名）必须是唯一的。

查看[Effective Go](https://golang.org/doc/effective_go.html#names)了解更多的关于go命名的

## 测试
Go 自带一个轻量级的测试框架，他由go 测试命令和测试包组成。
你可以通过以_test.go结尾的文件名来创建一个测试文件，并且包含了以TestXXX命名的方法带有（t *testing)的参数。
测试框架会运行这样的方法。如果方法调用失败，例如Error或者Fail，测试被认为是失败。

给stringutil包增加一个测试，创建一个测试文件 $GOPATH/src/github.com/user/stringutil/reverse_test.go 包含
以下代码
```
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```
然后使用go test运行测试
```
$ go test github.com/user/stringutil
ok github.com/user/stirngutil 0.165s
```
和以往一样，如果你正在包目录下运行go tool，你可以忽略包路径：
```
$ go test
ok github.com/user/stringutil 0.165s
```
运行 go help test 并查看[testing package documentation](https://golang.org/pkg/testing/)获取更多信息

## 远程包
一个导入路径可以描述如何利用版本控制软件获取包的源码，例如git或者Mercurial。 Go tool使用这个属性来自动
从远程库中获取包。例如，本文中描述的例子也在git库中保存着一份，
这个git库是Git Hub [github.com/golang/example](https://github.com/golang/example)
如果把这个库的URL宝航在包导入路径中，go就会自动去获取，构建并安装。
```
$ go get github.com/golang/example/hello
$ $GOPATH/bin/hello
Hello, Go examples!
```
如果指定的包不在当前的工作空间中，go get将会把他放到GOPATH指定的第一个工作空间中。(如果包已经存在，
go get会跳过远程获取并且像go install一样执行)

经过以上的go get命令后，工作空间的目录应该看起来如下所示：
```
bin/
    hello                           # command executable
pkg/
    linux_amd64/
        github.com/golang/example/
            stringutil.a            # package object
        github.com/user/
            stringutil.a            # package object
src/
    github.com/golang/example/
	.git/                       # Git repository metadata
        hello/
            hello.go                # command source
        stringutil/
            reverse.go              # package source
            reverse_test.go         # test source
    github.com/user/
        hello/
            hello.go                # command source
        stringutil/
            reverse.go              # package source
            reverse_test.go         # test source
```
托管在GitHub上的命令hello和他依赖的stringutil包在同一个资源库。hello.go中导入的文件使用同一个导入路径惯例，
所以go get命令也可以定位并安装依赖包。

```
import "github.com/golang/example/stringutil"
```
这种惯例是创建一个可用Go包给别人使用的最简单的方式。
[Go Wiki](https://golang.org/wiki/Projects) 和 [godoc.org](https://godoc.org/) 提供了一系列的外部Go项目
更多的用go tool使用远程库的信息，请看 [go help importpath](https://golang.org/cmd/go/#hdr-Remote_import_paths)

## 下一步？
订阅 [golan-announce](https://groups.google.com/group/golang-announce) 邮件列表，当新的Go稳定版发布时会通知
参看[Effective Go](https://golang.org/doc/effective_go.html)帮助编写干净，通畅的go代码
学习[A Tour of Go](https://tour.golang.org/)了解更多的语言特性
访问[documentation page](https://golang.org/doc/#articles)可找到一些关于go语言和库，工具的深度好文
##帮助
想要实时帮助，请在[Freenode](http://freenode.net/)网上实时聊天服务器的#go-nuts频道找乐于助人的gophers
官方的讨论Go语言的邮件列表是[Go Nuts](https://groups.google.com/group/golang-nuts)
报告bug在此[Go issue tracker](https://golang.org/issue)
