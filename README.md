# How to Write Go Code

https://golang.org/doc/code

https://xie.infoq.cn/article/11356ca6abc04c656954a89d0

## 代码组织
**包:**

**模块：** 一起发布相关联的 go 包的集合。名为 go.mod 的文件声明了这个模块的路径：模块内所有包的导入路径前缀。
每个模块的路径不仅充当了其他包的导入路径前缀，而且还表明了 go command 下载该模块的地方

**库:** 一个库中包含一个或多个模块。。Go 库通常只包含一个位于根目录的模块。

## 第一个程序
```shell
$ mkdir hello # Alternatively, clone it if it already exists in version control.
$ cd hello
$ go mod init example/user/hello
go: creating new go.mod: module example/user/hello
$ cat go.mod
module example/user/hello

go 1.16
$
```
go 源文件中的第一个声明必须是`package name`,可执行命令必须总是使用`package main`
创建一个文件`hello.go`
```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world.")
}
```
### 安装
```shell
go install example/user/hello
```
## Importing packages from your module

编写一个 `morestrings` 包并从 `hello` 程序中使用它。
给这个包创建一个目录叫`$HOME/hello/morestrings`，并且在里面新建个文件`reverse.go` 包含下面的内容
```go
// Package morestrings implements additional functions to manipulate UTF-8
// encoded strings, beyond what is provided in the standard "strings" package.
package morestrings

// ReverseRunes returns its argument string reversed rune-wise left to right.
func ReverseRunes(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```
编译
```shell
cd <GOPATH>/hello/morestrings
go build
```
从 `hello.go` 中引用 `morestrings`
```go
package main
import (
	"fmt"
	"example/user/hello/morestrings"
)

func main() {
	fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
}
```
再次安装 hello
```shell
go install example/user/hello
```
Running
```shell
$ hello
Hello, Go!
```
## 从远程模块引入包
导入路径可以描述如何用版本控制工具获取包的源码。
修改 `hello.go`
```go
package main

import (
	"fmt"

	"example/user/hello/morestrings"
	"github.com/google/go-cmp/cmp"
)

func main() {
	fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
	fmt.Println(cmp.Diff("Hello World", "Hello Go"))
}
```
记录依赖版本在`go.mod`
```shell
go mod tidy
```
清除模块缓存
```shell
go clean -modcache
```
## Testing

创建文件`<GOPATH>/hello/morestrings/reverse_test.go` 包含如下代码。
```go
package morestrings

import "testing"

func TestReverseRunes(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := ReverseRunes(c.in)
		if got != c.want {
			t.Errorf("ReverseRunes(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```
run test
```shell
$ cd <GOPATH>/hello/morestrings
$ go test
PASS
ok  	example/user/hello/morestrings 0.165s
$
```