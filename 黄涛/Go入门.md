## 包（Packages）

Go程序由包构成，并从`main`包开始运行。示例：

```go
package main

import (
    "fmt"
	"math/rand"
)

func main() {
	fmt.Println("got a random number", rand.Intn(10))
}
```

编译后运行结果：

```shell
got a random number 1
```

本程序通过导入路径`"fmt"`和`"math/rand"`，来使用这两个包。按照约定，包名与导入路径的最后一个元素相同。例如，`"math/rand"`包中的源码均以`package rand`语句开始。

## 导入（Imports）

示例：

```go
package main

import (
    "fmt"
	"math"
)

func main() {
    fmt.Printf("Now you have %g problems.\n", math.Sqrt(7))
}
```

编译后运行结果：

```shell
Now you have 2.6457513110645907 problems.
```

这段代码使用圆括号组合了导入，也就是分组导入。当然你也可以编写多个导入语句：

```go
import "fmt"
import "math"
```

但是，推荐使用分组导入的方式。

## 导出名（Exported names）

示例代码：

```go
package main

import (
    "fmt"
	"math"
)

func main() {
    fmt.Println(math.pi)
}
```

`go build`编译报错：

```shell
.\hello.go:9:17: cannot refer to unexported name math.pi
.\hello.go:9:17: undefined: math.pi
```

这是因为：在Go中，导出名以大写字母开头。比如，`Pizza`是一个已导出名，`Pi`也是，它导出自`math`包。上述代码中，`pi`并未以大写字母开头，因此是未导出的，所以报错。要修复该问题，只需将`math.pi`重命名为`math.Pi`即可。

## 函数（Functions）

示例代码：

```go
package main

import "fmt"

func add(x int, y int) int {
    return x + y
}

func main() {
    fmt.Println(add(42, 13))
}
```

编译运行结果：

```shell
55
```

函数可以接收零个或多个参数。在本例中，`add`函数接收两个`int`类型的参数，并且类型跟在变量的后面。

当两个或连续多个参数的类型相同时，可以只声明最后一个参数的类型：

```go
x, y int
```

注意，如果`add`函数不声明返回值类型为`int`，将会报错：

```shell
.\hello.go:6:5: too many arguments to return
        have (int)
        want ()
.\hello.go:10:20: add(42, 13) used as value
```

## 函数返回值

示例代码：

```go
package main

import "fmt"

func swap(x, y string) (string, string) {
    return y, x
}

func main() {
    a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```

编译运行结果：

```shell
world hello
```

函数可以返回任意数量的返回值，这里的`swap`函数返回了两个字符串。

### 命名返回值

示例代码

```go
package main

import "fmt"

func split(sum int) (x, y int) {
    x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
    fmt.Println(split(17))
}
```

编译运行结果

```shell
7 10
```

函数可以返回值可以被命名，它们会被视作定义在函数顶部的变量。没有参数的`return`语句返回已命名的返回值，也就是直接返回。直接返回影响代码的可读性，建议只在简短的函数中使用。

## 变量

Go使用`var`声明变量，类型跟在变量名后面。可以同时声明多个：

```go
var a, b, c int
```

声明变量时可以赋初始值，并省略类型，变量获得初始值的类型：

```go
package main

import "fmt"

var a = 1
var b = "hello"
var c bool = true

var x, y = 2, 3


func main() {
    fmt.Println(a, b, c, x ,y)
}
```

编译运行结果：

```shell
1 hello true 2 3
```

### 短变量声明

`var`声明可以在函数内部使用，也可以使用短赋值语句`:=`代替。`:=`仅能用在函数内部，因为函数外部的语句必须以关键字开始（`var`, `func`等）

```go
package main

import "fmt"

func main() {
    var a, b int = 1, 2
	k := 3
	c, python, java := true, "fantastic", "what"
	
	fmt.Println(a, b, k, c, python, java)
}
```

编译运行结果：

```shell
1 2 3 true fantastic what
```

## 基本类型

Go的基本类型：

 ```shell
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
 ```

`int`, `unit`, `uintptr`在32位机器上通常是32位宽，64位机器上是64位宽。

变量声明可以和导入一样，使用分组的方式：

```go
package main

import (
    "fmt"
	"math/cmplx"
)

var (
    ToBe	bool		= false
	MaxInt	uint64		= 1<<64 - 1
	z		complex128	= cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}
```

编译运行结果：

```shell
Type: bool Value: false
Type: uint64 Value: 18446744073709551615
Type: complex128 Value: (2+3i)
```

## 零值

只声明未赋值的变量获得零值：

* `0`: 数值类型
* `false`: 布尔类型
* `""`：字符串

示例：

```go
package main

import "fmt"

func main() {
	var i int
	var f float64
	var b bool
	var s string
	fmt.Printf("%v %v %v %q\n", i, f, b, s)
}
```

编译运行结果

```shell
0 0 false ""
```

## 类型转化

