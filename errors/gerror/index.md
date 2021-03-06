[TOC]

# 错误处理

`gf`框架提供了强大、丰富的错误处理能力，由`gerror`模块实现。

**使用方式**：
```go
import "github.com/gogf/gf/errors/gerror"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/errors/gerror


## 错误堆栈

标准库的`error`错误实现比较简单，无法进行堆栈追溯，对于产生错误时的上层调用者来讲不是很友好，无法获得错误的调用链详细信息。`gerror`支持错误堆栈记录，通过`New/Newf`、`Wrap/Wrapf`均会自动记录当前错误产生时的堆栈信息。

使用示例：
```go
package main

import (
	"fmt"

	"github.com/gogf/gf/errors/gerror"
)

func OpenFile() error {
	return gerror.New("permission denied")
}

func OpenConfig() error {
	return gerror.Wrap(OpenFile(), "configuration file opening failed")
}

func ReadConfig() error {
	return gerror.Wrap(OpenConfig(), "reading configuration failed")
}

func main() {
	fmt.Printf("%+v", ReadConfig())
}
```
执行后，终端输出：
```html
reading configuration failed: configuration file opening failed: permission denied
1. reading configuration failed
    1). main.ReadConfig
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:18
    2). main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:25
2. configuration file opening failed
    1). main.OpenConfig
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:14
    2). main.ReadConfig
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:18
    3). main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:25
3. permission denied
    1). main.OpenFile
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:10
    2). main.OpenConfig
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:14
    3). main.ReadConfig
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:18
    4). main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:25
```
可以看到，调用端可以通过`Wrap`方法将底层的错误信息进行层级叠加，并且包含完整的错误堆栈信息。

## `fmt`格式化

通过以上示例我们可以看到，通过`%+v`的打印格式可以打印出完整的堆栈信息，当然`gerror.Error`对象支持多种fmt格式：

|格式符|输出内容
|---|---
|`%v`, `%s`| 打印所有的层级错误信息，构成完成的字符串返回，多个层级使用`: `拼接。
|`%-v`, `%-s`| 打印当前层级的错误信息，返回字符串。
|`%+s`| 打印完整的堆栈信息列表。
|`%+v`| 打印所有的层级错误信息字符串，以及完整的堆栈信息，等同于`%s\n%+s`。

使用示例：
```go
package main

import (
	"errors"
	"fmt"

	"github.com/gogf/gf/errors/gerror"
)

func main() {
	var err error
	err = errors.New("sql error")
	err = gerror.Wrap(err, "adding failed")
	err = gerror.Wrap(err, "api calling failed")
	fmt.Printf(" %%s: %s\n", err)
	fmt.Printf("%%-s: %-s\n", err)
	fmt.Println("%+s: ")
	fmt.Printf("%+s\n", err)
}
```
执行后，终端输出为：
```html
 %s: api calling failed: adding failed: sql error
%-s: api calling failed
%+s: 
1. api calling failed
   1).  main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/.example/other/test.go:14
2. adding failed
   1).  main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/.example/other/test.go:13
3. sql error
```

## `Stack`方法

```go
func Stack(err error) string
```
通过`Stack`方法我们可以获得`error`对象的完整堆栈信息，返回堆栈列表字符串。
注意参数为标准库`error`类型，当该参数为`gerror`模块生成的`error`时，
或者开发者自定义的`error`对象实现了`gerror.ApiStack`接口时支持打印，否则，返回空字符串。

使用示例：
```go
package main

import (
	"errors"
	"fmt"

	"github.com/gogf/gf/errors/gerror"
)

func main() {
	var err error
	err = errors.New("sql error")
	err = gerror.Wrap(err, "adding failed")
	err = gerror.Wrap(err, "api calling failed")
	fmt.Println(gerror.Stack(err))
}
```
执行后，终端输出：
```html
1. api calling failed
   1).  main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/.example/other/test.go:14
2. adding failed
   1).  main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/.example/other/test.go:13
3. sql error
```

## `Current`方法

```go
func Current(err error) error
```

`Current`方法用于获取当前层级的错误信息，通过`error`接口对象返回。


使用示例：
```go
package main

import (
	"errors"
	"fmt"

	"github.com/gogf/gf/errors/gerror"
)

func main() {
	var err error
	err = errors.New("sql error")
	err = gerror.Wrap(err, "adding failed")
	err = gerror.Wrap(err, "api calling failed")
	fmt.Println(err)
	fmt.Println(gerror.Current(err))
}
```
执行后，终端输出：
```html
api calling failed: adding failed: sql error
api calling failed
```

## `Next`方法

```go
func Next(err error) error
```

`Next`方法用于获取层级错误的下一级错误`error`接口对象。当下一层级不存在时，返回`nil`。


使用示例：
```go
package main

import (
	"errors"
	"fmt"

	"github.com/gogf/gf/errors/gerror"
)

func main() {
	var err error
	err = errors.New("sql error")
	err = gerror.Wrap(err, "adding failed")
	err = gerror.Wrap(err, "api calling failed")

	fmt.Println(err)

	err = gerror.Next(err)
	fmt.Println(err)

	err = gerror.Next(err)
	fmt.Println(err)
}
```
执行后，终端输出：
```html
api calling failed: adding failed: sql error
adding failed: sql error
sql error
```


## `Cause`方法

```go
func Cause(err error) error
```
通过`Cause`方法我们可以获得`error`对象的根错误信息（原始错误）。
注意参数为标准库`error`类型，当该参数为`gerror`模块生成的`error`时，
或者开发者自定义的`error`对象实现了`gerror.ApiCause`接口时支持打印，否则，返回输出的`error`对象。

使用示例：
```go
package main

import (
	"fmt"

	"github.com/gogf/gf/errors/gerror"
)

func OpenFile() error {
	return gerror.New("permission denied")
}

func OpenConfig() error {
	return gerror.Wrap(OpenFile(), "configuration file opening failed")
}

func ReadConfig() error {
	return gerror.Wrap(OpenConfig(), "reading configuration failed")
}

func main() {
	fmt.Println(gerror.Cause(ReadConfig()))
}
```
执行后，终端输出：
```html
permission denied
```



## 日志输出支持

`glog`日志管理模块天然支持对`gerror`错误堆栈打印支持，这种支持不是强耦合性的，而是通过`fmt`格式化打印接口支持的。

使用示例：
```go
package main

import (
	"errors"
	"github.com/gogf/gf/frame/g"

	"github.com/gogf/gf/errors/gerror"
)

func main() {
	var err error
	err = errors.New("sql error")
	err = gerror.Wrap(err, "adding failed")
	err = gerror.Wrap(err, "api calling failed")
	g.Log().Print(err)
}
```
执行后，终端输出：
```html
2020-10-17 15:22:26.793 api calling failed: adding failed: sql error
1. api calling failed
   1).  main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/.example/other/test.go:14
2. adding failed
   1).  main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/.example/other/test.go:13
3. sql error
```

