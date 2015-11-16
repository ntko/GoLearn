###Go相关资源
* [build-web-application-with-golang ](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/preface.md)  
* [golang blog](http://blog.golang.org) 

# 目录

- [1.命令速查](#1命令速查)
- 2.入门准备
	- GOPATH设置
	- 文档查看
- 3.程序基础
	- 程序入口
		- main函数和init函数
		- import
	- 变量定义
	- 常量定义示范
	- 内置常规数据类型
		- bool
		- 数值类型
		- 字符串
		- 数组
	- 内置高级数据类型
		- 基础:方法集
		- 错误类型
		- Struct类型
		- Pointer类型
		- Function类型
		- Interface类型
		- Channel类型
		- slice类型
		- map类型 
	- 程序语句
		- if goto for switch
	- 函数
		- 变参
		- 传值与传指针
		- defer
		- 函数作为值、类型(函数指针)
		- Panic和Recover
	- Method方法成员
		- method可以被继承和重写
- 4.一般语言技巧
	- 零值
	- 语言隐含规则
	- 可赋值操作
	- 错误类型的使用
	- 分组声明
	- iota枚举
	- make、new操作
- 5.高级技巧
	- 类型断言及类型分支(Type assertions及Type switches)
		- 类型断言
		- 类型分支
	- 反射(reflect)
	- Go语言并发
		- Channel用于同步
		- Buffered Channels
		- Range和Close
		- select语句
		- runtime goroutine



#1.命令速查

  * go install pachage  //编译并安装包或者应用
  * go get github.com/astaxie/beedb  //获取远程包
  * go build pachage //编译包
  * go clean //清除编译后中间文件
  * go fmt //实际调用了 gofmt 格式化代码
  * go test //执行这个命令，会自动读取源码目录下面名为 *_test.go 的文件，生成并运行测试用的可执行文件
  * go tool fix .  用来修复以前老版本的代码到新版本，例如go1之前老版本的代码转化到go1,例如API的变化
  * go tool vet directory|files  用来分析当前目录的代码是否都是正确的代码,例如是不是调用fmt.Printf里面的参数不正确，例如函数里面提前return了然后出现了无用代码之类的。
  * go generate //这个命令是从Go1.4开始才设计的，用于在编译前自动化生成某类代码
  * go version 查看go当前的版本
  * go env 查看当前go的环境变量
  * go list 列出当前全部安装的package
  * go run 编译并运行Go程序
  * godoc 文档查看




#2.入门准备

##GOPATH设置

go 命令依赖一个重要的环境变量：$GOPATH.GOPATH允许多个目录，当有多个目录时，请注意分隔符，多个目录的时候Windows是分号，Linux系统是冒号，当有多个GOPATH时，默认会将go get的内容放在第一个目录下。

$GOPATH 目录约定有三个子目录：  

* src 存放源代码（比如：.go .c .h .s等）
* pkg 编译后生成的平台相关文件（比如：.a）
* bin 编译后生成的可执行文件

GOPATH下的src目录就是接下来开发的主要目录，所有源码都放在这个目录下面，一般做法是一个目录一个项目，例如: $GOPATH/src/mymath 表示mymath这个应用包或者可执行应用，这个根据package是main还是其他来决定，main的话就是可执行应用，其他的话就是应用包。

##文档查看

如何查看相应package的文档呢？ 例如builtin包，那么执行 godoc builtin  如果是http包，那么执行 godoc net/http  查看某一个包里面的函数，那么执行 godoc fmt Printf  也可以查看相应的代码，执行 godoc -src fmt Printf 

通过命令在命令行执行 godoc -http=:端口号 比如 godoc -http=:8080 。然后在浏览器中打开 127.0.0.1:8080 ，你将会看到一个golang.org的本地copy版本，通过它你可以查询pkg文档等其它内容。如果你设置了GOPATH，在pkg分类下，不但会列出标准包的文档，还会列出你本地 GOPATH 中所有项目的相关文档。


#3.程序基础

##程序入口

#### `main`函数和`init`函数

Go里面有两个保留的函数：`init`函数（能够应用于所有的`package`）和`main`函数（只能应用于`package main`）。这两个函数在定义时不能有任何的参数和返回值。

Go程序会自动调用`init()`和`main()`，所以你不需要在任何地方调用这两个函数。每个`package`中的`init`函数都是可选的，但`package main`就必须包含一个`main`函数。

**GO程序的入口就是`main.main()`,也就是main pachage的main()函数。每一个可独立运行的Go程序，必定包含一个`package main`，在这个`main`包中必定包含一个入口函数`main`.除了`main`包之外，其它的包最后都会生成`*.a`文件（也就是包文件）并放置在`$GOPATH/pkg/$GOOS_$GOARCH`中。**

>Go是天生支持UTF-8的，任何字符都可以直接输出，你甚至可以用UTF-8中的任何字符作为标识符。

程序的初始化和执行都起始于`main`包。如果`main`包还导入了其它的包，那么就会在编译时将它们依次导入。有时一个包会被多个包同时导入，那么它只会被导入一次.当一个包被导入时，如果该包还导入了其它的包，那么会先将其它包导入进来，然后再对这些包中的包级常量和变量进行初始化，接着执行`init`函数（如果有的话），依次类推。等所有被导入的包都加载完毕了，就会开始对`main`包中的包级常量和变量进行初始化，然后执行`main`包中的`init`函数（如果存在的话），最后执行`main`函数。下图详细地解释了整个执行过程：

#### import

import这个命令用来导入包文件，而我们经常看到的方式参考如下：

	import(
	    "fmt"
	)

然后我们代码里面可以通过如下的方式调用

	fmt.Println("hello world")

上面这个fmt是Go语言的标准库，其实是去`GOROOT`环境变量指定目录下去加载该模块，当然Go的import还支持如下两种方式来加载自己写的模块：

1. 相对路径

	import “./model” //当前文件同一目录的model目录，但是不建议这种方式来import

2. 绝对路径

	import “shorturl/model” //加载gopath/src/shorturl/model模块
	
	
上面展示了一些import常用的几种方式，但是还有一些特殊的import，让很多新手很费解，下面我们来一一讲解一下到底是怎么一回事
	
	
1. 点操作
	
	我们有时候会看到如下的方式导入包
	
		import(
		    . "fmt"
		)
	
	这个点操作的含义就是这个包导入之后在你调用这个包的函数时，你可以省略前缀的包名，也就是前面你调用的fmt.Println("hello world")可以省略的写成Println("hello world")

2. 别名操作

	别名操作顾名思义我们可以把包命名成另一个我们用起来容易记忆的名字
	
		import(
		    f "fmt"
		)
		
	别名操作的话调用包函数时前缀变成了我们的前缀，即f.Println("hello world")

3. _操作

	这个操作经常是让很多人费解的一个操作符，请看下面这个import
	
		import (
		    "database/sql"
		    _ "github.com/ziutek/mymysql/godrv"
		)
		
	_操作其实是引入该包，而不直接使用包里面的函数，而是调用了该包里面的init函数。


##变量定义

>**Go的变量定义,类型type在变量名称之后。**

* 定义三个类型都是“type”的变量    

	var vname1, vname2, vname3 type

* 定义三个类型都是"type"的变量,并且分别初始化为相应的值  

	var vname1, vname2, vname3 type= v1, v2, v3

* 省略type, Go自动根据类型初始化

	var vname1, vname2, vname3 = v1, v2, v3

* 省略var和type, Go自动根据类型初始化

	vname1, vname2, vname3 := v1, v2, v3

`:=`这个符号直接取代了`var`和`type`,这种形式叫做简短声明。限制是只能用在函数内部；在函数外部使用则会无法编译通过，所以一般用`var`方式来定义全局变量。

* `_`（下划线）特殊变量

`_`（下划线）是个特殊的变量名，任何赋予它的值都会被丢弃。在这个例子中，我们将值`35`赋予`b`，并同时丢弃`34`：

	_, b := 34, 35

Go对于已声明但未使用的变量会在编译阶段报错，比如下面的代码就会产生错误：声明了`i`但未使用。

	package main

	func main() {
		var i int
	}

##常量定义示范

	const a = 2 + 3.0          // a == 5.0   (untyped floating-point constant)
	const b = 15 / 4           // b == 3     (untyped integer constant)
	const c = 15 / 4.0         // c == 3.75  (untyped floating-point constant)
	const Θ float64 = 3/2      // Θ == 1.0   (type float64, 3/2 is integer division)
	const Π float64 = 3/2.     // Π == 1.5   (type float64, 3/2. is float division)
	const d = 1 << 3.0         // d == 8     (untyped integer constant)
	const e = 1.0 << 3         // e == 8     (untyped integer constant)
	const f = int32(1) << 33   // illegal    (constant 8589934592 overflows int32)
	const g = float64(2) >> 1  // illegal    (float64(2) is a typed floating-point constant)
	const h = "foo" > "bar"    // h == true  (untyped boolean constant)
	const j = true             // j == true  (untyped boolean constant)
	const k = 'w' + 1          // k == 'x'   (untyped rune constant)
	const l = "hi"             // l == "hi"  (untyped string constant)
	const m = string(k)        // m == "x"   (type string)
	const Σ = 1 - 0.707i       //            (untyped complex constant)
	const Δ = Σ + 2.0e-4       //            (untyped complex constant)
	const Φ = iota*1i - 1/1i   //            (untyped complex constant)

##内置常规数据类型

#### bool

在Go中，布尔值的类型为`bool`，值是`true`或`false`，默认为`false`。

	//示例代码
	var enabled, disabled = true, false  // 忽略类型的声明
	func test() {
		var available bool  // 一般声明
		valid := false      // 简短声明
		available = true    // 赋值操作
	}

#### 数值类型

整数类型有无符号和带符号两种。Go同时支持`int`和`uint`，这两种类型的长度相同，但具体长度取决于不同编译器的实现。Go里面也有直接定义好位数的类型：`rune`, `int8`, `int16`, `int32`, `int64`和`byte`, `uint8`, `uint16`, `uint32`, `uint64`。其中`rune`是`int32`的别称，`byte`是`uint8`的别称。

>需要注意的一点是，这些类型的变量之间不允许互相赋值或操作，不然会在编译时引起编译器报错。
>
>如下的代码会产生错误：invalid operation: a + b (mismatched types int8 and int32)
>
	var a int8
	var b int32
	c:=a + b
>
>另外，尽管int的长度是32 bit, 但int 与 int32并不可以互用。

浮点数的类型有`float32`和`float64`两种（没有`float`类型），默认是`float64`。

Go还支持复数。它的默认类型是`complex128`（64位实数+64位虚数）。如果需要小一些的，也有`complex64`(32位实数+32位虚数)。复数的形式为`RE + IMi`，其中`RE`是实数部分，`IM`是虚数部分，而最后的`i`是虚数单位。下面是一个使用复数的例子：

	var c complex64 = 5+5i
	//output: (5+5i)
	fmt.Printf("Value is: %v", c)


#### 字符串

Go中的字符串都是采用`UTF-8`字符集编码。字符串是用一对双引号（`""`）或反引号（`` ` `` `` ` ``）括起来定义，它的类型是`string`。

	//示例代码
	var frenchHello string  // 声明变量为字符串的一般方法
	var emptyString string = ""  // 声明了一个字符串变量，初始化为空字符串
	func test() {
		no, yes, maybe := "no", "yes", "maybe"  // 简短声明，同时声明多个变量
		japaneseHello := "Konichiwa"  // 同上
		frenchHello = "Bonjour"  // 常规赋值
	}

Go中可以使用`+`操作符来连接两个字符串：

	s := "hello,"
	m := " world"
	a := s + m
	fmt.Printf("%s\n", a)

可以通过`` ` ``来声明一个多行的字符串:

	m := `hello
		world`

`` ` `` 括起的字符串为Raw字符串，即字符串在代码中的形式就是打印时的形式，它没有字符转义，换行也将原样输出。例如本例中会输出：
	
    hello
		world

#### 数组
`array`就是数组，它的定义方式如下：

	var arr [n]type

`n`表示数组的长度，`type`表示存储元素的类型：

	var arr [10]int  // 声明了一个int类型的数组
	arr[0] = 42      // 数组下标是从0开始的

由于长度也是数组类型的一部分，因此`[3]int`与`[4]int`是不同的类型，数组也就不能改变长度。数组之间的赋值是值的赋值，而不是它的指针。如果要使用指针，那么就需要用到`slice`类型了。

数组可以使用另一种`:=`来声明

	a := [3]int{1, 2, 3} // 声明了一个长度为3的int数组

	b := [10]int{1, 2, 3} // 声明了一个长度为10的int数组，其中前三个元素初始化为1、2、3，其它默认为0

	c := [...]int{4, 5, 6} // 可以省略长度而采用`...`的方式，Go会自动根据元素个数来计算长度

Go支持嵌套数组：

	// 声明了一个二维数组，该数组以两个数组作为元素，其中每个数组中又有4个int类型的元素
	doubleArray := [2][4]int{[4]int{1, 2, 3, 4}, [4]int{5, 6, 7, 8}}

	// 上面的声明可以简化，直接忽略内部的类型
	easyArray := [2][4]int{{1, 2, 3, 4}, {5, 6, 7, 8}}

##内置高级数据类型

#### 基础:方法集

> **方法集是一个定义而不是类型.方法集,或者叫做成员方法,是一个类型的函数集合.需要注意的是,T的方法集,就是带有接收器T类型的方法,而其对应的指针类型\*T的方法集,则包括了接收器T和\*T的方法.**

#### 错误类型
Go内置有一个`error`类型，专门用来处理错误信息，Go的`package`里面还专门有一个包`errors`来处理错误：

	err := errors.New("emit macho dwarf: elf header corrupted")
	if err != nil {
		fmt.Print(err)
	}

#### Struct类型

直接看示例:

	// An empty struct.
	struct {}
	
	// A struct with some fields.
	struct {
		x, y int
		u float32
		_ float32  // padding,_用于对齐
		A *[]int
		F func()
	}

	// 匿名成员字段的struct
	struct {
		T1        // field name is T1
		*T2       // field name is T2
		P.T3      // field name is T3
		*P.T4     // field name is T4
		x, y int  // field names are x and y
	}

>**注意以下是非法定义,因为成员名称重复**

	struct {
		T     // conflicts with anonymous field *T and *P.T
		*T    // conflicts with anonymous field T and *P.T
		*P.T  // conflicts with anonymous field T and *T
	}

>**假设f是strcut x中的一个匿名成员n的字段或者方法,如果x.f是一个能标识f的合法的选择器,那么f称为"被提升"."被提升"的字段和struct x的普通字段一样,只是他们不能被用作strcut的构造语法的字段名.**

给定strcut S,以及类型T,S中根据如下规则包含被提升的方法:

* 如果S包含了匿名字段T,那么, S和\*S的方法集中,都包含了带有 T类型接收器的被提升的方法.[注:接收器类似于类成员方法],即,都包含了T的成员方法.但是,\*S的方法集中,则只包含了带有\*T类型接收器的被提升的方法. 
* 如果S包含了匿名字段\*T,那么, S和\*S的方法集中,则都包含了T和\*T类型接收器的被提升方法.也即包含了T和\*T的所有成员方法. 

strcut的字段定义中,可以跟随一个可选的字符串tag标签.这个标签蒋称为相关字段定义的所有字段的属性.tag被用于反射接口,参与struct类型标识.其它时候将被忽略.

	struct {
		a,b  uint64 "tag 1"
		c uint64 "field 2"
	}

>**实例1:**

	type User struct {
	    Name   string "user name" //这引号里面的就是tag
	    Passwd string "user passsword"
	}
	func main() {
	    user := &User{"chronos", "pass"}
	    s := reflect.TypeOf(user).Elem() //通过反射获取type定义
	    for i := 0; i < s.NumField(); i++ {
	        fmt.Println(s.Field(i).Tag) //将tag输出出来
	    }
	}

>**实例2:**

	//Golang.org中reflect的示例代码
	
	package main
	 
	import (
	    "fmt"
	    "reflect"
	)
	 
	func main() {
	    type S struct {
	        F string `species:"gopher" color:"blue"`
	    }
	 
	    s := S{}
	    st := reflect.TypeOf(s)
	    field := st.Field(0)
	    fmt.Println(field.Tag.Get("color"), field.Tag.Get("species"))
	 
	}


#### Pointer类型

一个示例就够了:

	*Point
	*[4]int

#### Function类型

>**(注:相当于C中的函数指针)**

function类型标识了具有相同参数和返回值的函数集合.未初始化的function类型的变量值为nil.

同样上示例即可:

	func()
	func(x int) int
	func(a, _ int, z float32) bool
	func(a, b int, z float32) (bool)
	func(prefix string, values ...int)
	func(a, b int, z float64, opt ...interface{}) (success bool)
	func(int, int, float64) (float64, *[]int)
	func(n int) func(p *T)

#### Interface类型

interface类型定义了一组接口方法.interface类型的变量,可以存储任何实现了该组接口方法的类型.且该类型就叫做实现了该接口.未初始化的Interface类型的变量值为nil.

>**Go语言取消了C++中相对复杂的接口定义及实现.直接使用如下原则:**
>
>* **实现方法即实现接口**
>* **接口可嵌入另外一个接口,即可标识接口继承了**

代码实例:

	type Locker interface {
		Lock()
		Unlock()
	}
	type ReadWriter interface {
		Read(b Buffer) bool
		Write(b Buffer) bool
	}
	
	type File interface {
		ReadWriter  // same as adding the methods of ReadWriter
		Locker      // same as adding the methods of Locker
		Close()
	}
	
	type LockedFile interface {
		Locker
		File        // 错误: Lock, Unlock not unique
		Lock()      // 错误: Lock not unique
	}

#### Channel类型

参考后面[Go语言并发部分.Channel用于同步](#Channel用于同步)

#### slice类型

`slice`是数组的引用类型。`slice`总是指向一个底层`array`，`slice`的声明也可以像`array`一样，只是不需要长度。

	var fslice []int //定义slice
	slice := []byte {'a', 'b', 'c', 'd'} //定义并初始化

`slice`可以从一个数组或一个已经存在的`slice`中再次声明。如果`slice`通过`array[i:j]`来获取的，其中`i`是数组的开始位置，`j`是结束位置，但不包含`array[j]`，它的长度是`j-i`。它的容量capacity是`len(array)-i`,相当于`array[i:j:len(array)]`

	// 声明一个含有10个元素元素类型为byte的数组
	var ar = [10]byte {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}

	// 声明含有byte的slice
	var a []byte

	// a指向数组的第3个元素开始，并到第五个元素结束，
	a = ar[2:5]

	//这个的容量就是`7-2`，即5。这样这个产生的新的slice就没办法访问最后的三个元素。
	slice := ar[2:4:7]
	


如果slice是这样的形式`array[:i:j]`，即第一个参数为空，默认值就是0。

>**注意`slice`和数组在声明时的区别：声明数组时，方括号内写明了数组的长度或使用`...`自动计算长度，而声明`slice`时，方括号内没有任何字符。**

slice有一些简便的操作

 - `slice`的默认开始位置是0，`ar[:n]`等价于`ar[0:n]`
 - `slice`的第二个序列默认是数组的长度，`ar[n:]`等价于`ar[n:len(ar)]`
 - 如果从一个数组里面直接获取`slice`，可以这样`ar[:]`，因为默认第一个序列是0，第二个是数组的长度，即等价于`ar[0:len(ar)]`

>**因为`slice`是引用类型，所以当引用改变其中元素的值时，其它的所有引用都会改变该值。**

从概念上面来说`slice`像一个结构体，这个结构体包含了三个元素：

- 一个指针，指向数组中`slice`指定的开始位置
- 长度，即`slice`的长度
- 最大长度(容量)，也就是`slice`开始位置到数组的最后位置的长度

	
对于`slice`有几个有用的内置函数：

- `len`    获取`slice`的长度
- `cap`    获取`slice`的最大容量
- `append` 向`slice`里面追加一个或者多个元素，然后返回一个和`slice`一样类型的`slice`
- `copy`   函数`copy`从源`slice`的`src`中复制元素到目标`dst`，并且返回复制的元素的个数

>**注意：`append`函数会改变`slice`所引用的数组的内容，从而影响到引用同一数组的其它`slice`。
但当`slice`中没有剩余空间（即`(cap-len) == 0`）时，此时将动态分配新的数组空间。返回的`slice`数组指针将指向这个空间，而原数组的内容将保持不变；其它引用此数组的`slice`则不受影响。**

#### map类型

`map`的格式为`map[keyType]valueType`

我们看下面的代码，`map`的读取和设置通过`key`来操作，而`map`的`key`可以是`int`，也可以是`string`及所有完全定义了`==`与`!=`操作的类型。

	// 声明一个key是字符串，值为int的字典,这种方式的声明需要在使用之前使用make初始化
	var numbers map[string]int

	// 另一种map的声明方式
	numbers := make(map[string]int)
	numbers["one"] = 1  //赋值

	// 带有值得初始化方式
	rating := map[string]float32{"C":5, "Go":4.5, "Python":4.5, "C++":2 }

	// map有两个返回值，第二个返回值，如果不存在key，那么ok为false，如果存在ok为true
	csharpRating, ok := rating["C#"]
	if ok {
		fmt.Println("C# is in the map and its rating is ", csharpRating)
	} else {
		fmt.Println("We have no rating associated with C# in the map")
	}

	delete(rating, "C")  // 删除key为C的元素


>**注意: `map`也是一种引用类型.`map`和其他基本型别不同，它不是thread-safe，在多个go-routine存取时，必须使用mutex lock机制。**

##程序语句

###流程控制

####if

* Go里面if条件判断语句中不需要括号
* Go的if还允许在条件判断语句里面声明一个变量，这个变量的作用域只能在该条件逻辑块内.

		// 计算获取值x,然后根据x返回的大小，判断是否大于10。
		if x := computedValue(); x > 10 {
		    fmt.Println("x is greater than 10")
		} else {
		    fmt.Println("x is less than 10")
		}


#### goto

用`goto`跳转到必须在当前函数内定义的标签,且标签名大小写敏感。

#### for
Go里面最强大的一个控制逻辑就是`for`，它即可以用来循环读取数据，又可以当作`while`来控制逻辑，还能迭代操作。它的语法如下：(**注意同样不需要括号**)

	for expression1; expression2; expression3 {
		//...
	}

`expression1`、`expression2`和`expression3`都是表达式，其中`expression1`和`expression3`是变量声明或者函数调用返回值之类的，`expression2`是用来条件判断，`expression1`在循环开始之前调用，`expression3`在每轮循环结束之时调用。

* 基本形态: 

		for index:=0; index < 10 ; index++ {
			sum += index
		}

>**有些时候需要进行多个赋值操作，由于Go里面没有`,`操作符，那么可以使用平行赋值`i, j = i+1, j-1`**

* 忽略`expression1`和`expression3`及`;`,等同于`while`的功能:

		sum := 1
		for sum < 1000 {
			sum += sum
		}

>在循环里面有两个关键操作`break`和`continue`	,`break`操作是跳出当前循环，`continue`是跳过本次循环。当嵌套过深的时候，`break`和`continue`都可以配合标签使用，即跳转至标签所指定的位置,用来跳到多重循环中的外层循环.

* `for`配合`range`可以用于读取`slice`和`map`的数据：

		for k,v:=range map {
			fmt.Println("map's key:",k)
			fmt.Println("map's val:",v)
		}

* 由于 Go 支持 “多值返回”, 而对于“声明而未被调用”的变量, 编译器会报错, 在这种情况下, 可以使用`_`来丢弃不需要的返回值
例如

		for _, v := range map{
			fmt.Println("map's val:", v)
		}


#### switch

`switch`的语法如下:

	switch sExpr {
	case expr1:
		some instructions
	case expr2:
		some other instructions
	case expr3:
		some other instructions
	default:
		other code
	}

`sExpr`和`expr1`、`expr2`、`expr3`的类型必须一致。表达式不必是常量或整数，执行的过程从上至下，直到找到匹配项；如果`switch`没有表达式，它会匹配`true`。

* 我们可以把很多值聚合在了一个`case`里面，比如,`case 2, 3, 4:`

* Go里面`switch`默认相当于每个`case`最后带有`break`，匹配成功后不会自动向下执行其他case，而是跳出整个`switch`, 但是可以使用`fallthrough`强制执行后面的case代码。

## 函数

函数是Go里面的核心设计，它通过关键字`func`来声明，它的格式如下：

	func funcName(input1 type1, input2 type2) (output1 type1, output2 type2) {
		//这里是处理逻辑代码
		//返回多个值
		return value1, value2
	}



- 关键字`func`用来声明一个函数`funcName`
- **函数可以返回多个值**
- 上面返回值声明了两个变量`output1`和`output2`，如果你不想声明也可以，直接就两个类型
- 如果只有一个返回值且不声明返回值变量，那么你可以省略 包括返回值 的括号
- 如果没有返回值，那么就直接省略最后的返回信息

下面我们来看一个实际应用函数的例子（用来计算Max值）

	// 返回a、b中最大值.
	func max(a, b int) int {
		if a > b {
			return a
		}
		return b
	}

	//返回 A+B 和 A*B
	func SumAndProduct(A, B int) (int, int) {
		return A+B, A*B
	}

上面这个里面我们可以看到`max`函数有两个参数，它们的类型都是`int`，那么第一个变量的类型可以省略（即 a,b int,而非 a int, b int)，同理多于2个同类型的变量或者返回值。同时我们注意到它的返回值就是一个类型，这个就是省略写法。

而SumAndProduct直接返回了两个参数.

#### 变参

	func myfunc(arg ...int) {}
`arg ...int`告诉Go这个函数接受不定数量的参数。注意，这些参数的类型全部是`int`。在函数体中，变量`arg`是一个`int`的`slice`：

	for _, n := range arg {
		fmt.Printf("And the number is: %d\n", n)
	}

#### 传值与传指针

Go语言中`channel`，`slice`，`map`这三种类型的实现机制类似指针，所以可以直接传递，而不用取地址后传递指针。（注：若函数需改变`slice`的长度，则仍需要取地址传递指针）

#### defer

Go语言中有种不错的设计，即延迟（defer）语句，你可以在函数中添加多个defer语句。当函数执行到最后时，这些defer语句会按照逆序执行,即采用后进先出模式，最后该函数返回。特别是当你在进行一些打开资源的操作时，遇到错误需要提前返回，在返回前你需要关闭相应的资源，不然很容易造成资源泄露等问题。示例：

	func ReadWrite() bool {
		file.Open("file")
		defer file.Close()
		if failureX {
			return false
		}
		if failureY {
			return false
		}
		return true
	}

#### 函数作为值、类型(函数指针)

在Go中函数也是一种变量，我们可以通过`type`来定义它，它的类型就是所有拥有相同的参数，相同的返回值的一种类型

	type typeName func(input1 inputType1 , input2 inputType2 [, ...]) (result1 resultType1 [, ...])

函数作为类型到底有什么好处呢？那就是可以把这个类型的函数当做值来传递，请看下面的例子

	package main
	import "fmt"

	type testInt func(int) bool // 声明了一个函数类型

	func isOdd(integer int) bool {
		if integer%2 == 0 {
			return false
		}
		return true
	}

	func isEven(integer int) bool {
		if integer%2 == 0 {
			return true
		}
		return false
	}

	// 声明的函数类型在这个地方当做了一个参数

	func filter(slice []int, f testInt) []int {
		var result []int
		for _, value := range slice {
			if f(value) {
				result = append(result, value)
			}
		}
		return result
	}

	func main(){
		slice := []int {1, 2, 3, 4, 5, 7}
		fmt.Println("slice = ", slice)
		odd := filter(slice, isOdd)    // 函数当做值来传递了
		fmt.Println("Odd elements of slice are: ", odd)
		even := filter(slice, isEven)  // 函数当做值来传递了
		fmt.Println("Even elements of slice are: ", even)
	}

函数当做值和类型在我们写一些通用接口的时候非常有用。

#### Panic和Recover

Go没有异常机制，它不能抛出异常，而是使用了`panic`和`recover`机制。

Panic
>是一个内建函数，可以中断原有的控制流程，进入一个令人恐慌的流程中。当函数`F`调用`panic`，函数F的执行被中断，但是`F`中的延迟函数会正常执行，然后F返回到调用它的地方。在调用的地方，`F`的行为就像调用了`panic`。这一过程继续向上，直到发生`panic`的`goroutine`中所有调用的函数返回，此时程序退出。恐慌可以直接调用`panic`产生。也可以由运行时错误产生，例如访问越界的数组。

Recover
>是一个内建的函数，可以让进入令人恐慌的流程中的`goroutine`恢复过来。`recover`仅在延迟函数中有效。在正常的执行过程中，调用`recover`会返回`nil`，并且没有其它任何效果。如果当前的`goroutine`陷入恐慌，调用`recover`可以捕获到`panic`的输入值，并且恢复正常的执行。

下面这个函数演示了如何在过程中使用`panic`

	var user = os.Getenv("USER")

	func init() {
		if user == "" {
			panic("no value for $USER")
		}
	}

下面这个函数检查作为其参数的函数在执行时是否会产生`panic`：

	func throwsPanic(f func()) (b bool) {
		defer func() {
			if x := recover(); x != nil {
				b = true
			}
		}()
		f() //执行函数f，如果f中出现了panic，那么就可以恢复回来
		return
	}

##Method方法成员

method的语法如下：

	func (r ReceiverType) funcName(parameters) (results)

用Rob Pike的话来说就是：
>"A method is a function with an implicit first argument, called a receiver."
>
>**即:Method就是一个函数,它的第一个参数是隐含的,就是接收器.**


另外：
>如果一个method的receiver是*T,你可以在一个T类型的实例变量V上面调用这个method，而不需要&V去调用这个method

类似的
>如果一个method的receiver是T，你可以在一个*T类型的变量P上面调用这个method，而不需要 *P去调用这个method

所以，你不用担心你是调用的指针的method还是不是指针的method，Go知道你要做的一切。

#### method可以被继承和重写

	package main
	import "fmt"

	type Human struct {
		name string
		age int
		phone string
	}

	type Student struct {
		Human //匿名字段
		school string
	}

	type Employee struct {
		Human //匿名字段
		company string
	}

	//在human上面定义了一个method
	func (h *Human) SayHi() {
		fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
	}

	func main() {
		mark := Student{Human{"Mark", 25, "222-222-YYYY"}, "MIT"}
		sam := Employee{Human{"Sam", 45, "111-888-XXXX"}, "Golang Inc"}

		mark.SayHi()
		sam.SayHi()
	}

上面的例子中，如果Employee想要实现自己的SayHi,在Employee上面定义一个method，重写匿名字段的方法即可。


#4.一般语言技巧

## 零值
关于“零值”，所指并非是空值，而是一种“变量未填充前”的默认值，通常为0。
此处罗列 部分类型 的 “零值”

	int     0
	int8    0
	int32   0
	int64   0
	uint    0x0
	rune    0 //rune的实际类型是 int32
	byte    0x0 // byte的实际类型是 uint8
	float32 0 //长度为 4 byte
	float64 0 //长度为 8 byte
	bool    false
	string  ""

## 语言隐含规则

Go语言有一些默认的行为：    

- 大写字母开头的变量是可导出的，也就是其它包可以读取的，是公用变量；小写字母开头的就是不可导出的，是私有变量。    
- 大写字母开头的函数也是一样，相当于`class`中的带`public`关键词的公有函数；小写字母开头的就是有`private`关键词的私有函数。

## 可赋值操作

Assignability

A value x is assignable to a variable of type T ("x is assignable to T") in any of these cases: 

* x's type is identical to T. 
* x's type V and T have identical underlying types and at least one of V or T is not a named type. 
* T is an interface type and x implements T. 
* x is a bidirectional channel value, T is a channel type, x's type V and T have identical element types, and at least one of V or T is not a named type. 
* x is the predeclared identifier nil and T is a pointer, function, slice, map, channel, or interface type. 
* x is an untyped constant representable by a value of type T. 

## 错误类型的使用

此处暂缺

## 分组声明

在Go语言中，同时声明多个常量、变量，或者导入多个包时，可采用分组的方式进行声明。

例如下面的代码：

	import "fmt"
	import "os"

	const i = 100
	const pi = 3.1415
	const prefix = "Go_"

	var i int
	var pi float32
	var prefix string

可以分组写成如下形式：

	import(
		"fmt"
		"os"
	)

	const(
		i = 100
		pi = 3.1415
		prefix = "Go_"
	)

	var(
		i int
		pi float32
		prefix string
	)

## iota枚举

Go里面的关键字`iota`，可以在声明`enum`时采用，它默认开始值是0，每调用一次加1：

	const(
		x = iota  // x == 0
		y = iota  // y == 1
		z = iota  // z == 2
		w  // 常量声明省略值时，默认和之前一个值的字面相同。这里隐式地说w = iota，因此w == 3。其实上面y和z可同样不用"= iota"
	)

	const v = iota // 每遇到一个const关键字，iota就会重置，此时v == 0
	
	const ( 
	  e, f, g = iota, iota, iota //e=0,f=0,g=0 iota在同一行值相同
	)

>除非被显式设置为其它值或`iota`，每个`const`分组的第一个常量被默认设置为它的0值，第二及后续的常量被默认设置为它前面那个常量的值，如果前面那个常量的值是`iota`，则它也被设置为`iota`。

## make、new操作

`make`用于内建类型（`map`、`slice` 和`channel`）的内存分配。`new`用于各种类型的内存分配。

内建函数`new`本质上说跟其它语言中的同名函数功能一样：`new(T)`分配了零值填充的`T`类型的内存空间，并且返回其地址，即一个`*T`类型的值。用Go的术语说，它返回了一个指针，指向新分配的类型`T`的零值。有一点非常重要：

>`new`返回指针。

内建函数`make(T, args)`与`new(T)`有着不同的功能，make只能创建`slice`、`map`和`channel`，并且返回一个有初始值(非零)的`T`类型，而不是`*T`。本质来讲，导致这三个类型有所不同的原因是指向数据结构的引用在使用前必须被初始化。例如，一个`slice`，是一个包含指向数据（内部`array`）的指针、长度和容量的三项描述符；在这些项目被初始化之前，`slice`为`nil`。对于`slice`、`map`和`channel`来说，`make`初始化了内部的数据结构，填充适当的值。

>`make`返回初始化后的（非零）值。

#5.高级技巧

## 类型断言及类型分支(Type assertions及Type switches)

#### 类型断言

对于interface类型变量x,以及类型T:

	x.(T)

断言x不是nil,并且x中存储的值是类型T.更准确的说:

* 如果T不是一个interface类型,x.(T)断言动态类型x,和类型T等同.这时,T必须实现x.否则,断言不成立,因为x无法存储一个T类型的值.

* 反之,如果T是一个interface类型,x.(T)则断言,x必须实现了接口T.

如果断言成立,该表达式的值是类型为T的x的值,否则,**会发生运行时错误.**

	var x interface{} = 7  // x has dynamic type int and value 7
	i := x.(int)           // i has type int and value 7
	
	type I interface { m() }
	var y I
	s := y.(string)        // illegal: string does not implement I (missing method m)
	r := y.(io.Reader)     // r has type io.Reader and y must implement both I and io.Reader

在赋值和初始化有个特殊形态的断言:

	v, ok = x.(T)
	v, ok := x.(T)
	var v, ok = x.(T)

这时蒋产生一个附加的bool值.该布尔值标识断言是否成立.如果成立,v保存了T类型的x的值,如果不成立,v保存了T类型的0值.**此时将不会有运行时错误**.


#### 类型分支

x是一个interface{}类型, 如下类型分支代码: 

	switch i := x.(type) {
	case nil:
		printString("x is nil")                // type of i is type of x (interface{})
	case int:
		printInt(i)                            // type of i is int
	case float64:
		printFloat64(i)                        // type of i is float64
	case func(int) float64:
		printFunction(i)                       // type of i is func(int) float64
	case bool, string:
		printString("type is bool or string")  // type of i is type of x (interface{})
	default:
		printString("don't know the type")     // type of i is type of x (interface{})
	}


也可以被这样重写: 
	
	v := x  // x is evaluated exactly once
	if v == nil {
		i := v                                 // type of i is type of x (interface{})
		printString("x is nil")
	} else if i, isInt := v.(int); isInt {
		printInt(i)                            // type of i is int
	} else if i, isFloat64 := v.(float64); isFloat64 {
		printFloat64(i)                        // type of i is float64
	} else if i, isFunc := v.(func(int) float64); isFunc {
		printFunction(i)                       // type of i is func(int) float64
	} else {
		_, isBool := v.(bool)
		_, isString := v.(string)
		if isBool || isString {
			i := v                         // type of i is type of x (interface{})
			printString("type is bool or string")
		} else {
			i := v                         // type of i is type of x (interface{})
			printString("don't know the type")
		}
	}


The type switch guard may be preceded by a simple statement, which executes before the guard is evaluated. 

**The "fallthrough" statement is not permitted in a type switch.** 

## 反射(reflect)

直接看示例:

	var reflectTest float64 = 3.4
	pv := reflect.ValueOf(&reflectTest)
	
	fmt.Println("type:",pv.Type())
	fmt.Println("kind is float64:",pv.Kind() == reflect.Float64)
	fmt.Println("Value:",pv)
	
	rv := pv.Elem()
	rv.SetFloat(7.1)

	fmt.Printf("reflectTest=%f\n",reflectTest)
	
	vv := reflect.ValueOf(reflectTest)
	fmt.Println("type:",vv.Type())
	fmt.Println("kind is float64:",vv.Kind() == reflect.Float64)
	fmt.Println("Value:",vv.Float())

以上代码输出:

	type: *float64
	kind is float64: false
	Value: 0xc082008d50
	reflectTest=7.100000
	type: float64
	kind is float64: true
	Value: 7.1

##Go语言并发

####goroutine

goroutine是Go并行设计的核心。goroutine是通过Go的runtime管理的一个线程管理器。goroutine通过`go`关键字实现，其实就是一个普通的函数。

	go hello(a, b, c)

代码示例:

	package main

	import (
		"fmt"
		"runtime"
	)

	func say(s string) {
		for i := 0; i < 3; i++ {
			runtime.Gosched()
			fmt.Println(s)
		}
	}

	func main() {
		go say("world") //开一个新的Goroutines执行
		say("hello") //当前Goroutines执行
	}

	// 以上程序执行后将输出：
	// hello
	// world
	// hello
	// world
	// hello

> **注意:**尽管上面的多个goroutine运行在同一个进程里面，可以共享内存数据，不过设计上我们要遵循：**不要通过共享来通信，而要通过通信来共享。**

> **runtime.Gosched()表示让CPU把时间片让给别人,下次某个时候继续恢复执行该goroutine。**

>这里有一篇Rob介绍的关于并发和并行的文章：http://concur.rspace.googlecode.com/hg/talk/concur.html#landing-slide

####Channel用于同步

channel通过发送和接收某种类型的值进行通讯的方式,提供了同步执行的机制.未初始化的channel类型的变量值为nil.可以被定义为双向和单向,且限定可以通过它发送或者接收的数据类型.可以与Unix shell 中的双向管道做类比。

	chan T          // 双向发送接收类型T的数据
	chan<- float64  // 只能用来发送float64s
	<-chan int      // 只能用来接收int

>**注意，必须使用make 创建channel：**

	ci := make(chan int)
	cs := make(chan string)
	cf := make(chan interface{})

channel通过操作符`<-`来接收和发送数据

	ch <- v    // 发送v到channel ch.
	v := <-ch  // 从ch中接收数据，并赋值给v

初始化可以用make,可选容量100:

	make(chan int, 100)


默认情况下，channel接收和发送数据都是阻塞的，除非另一端已经准备好，也就是如果读取（value := <-ch）它将会被阻塞，直到有数据接收。其次，任何发送（ch<-5）将会被阻塞，直到数据被读出。无缓冲channel是在多个goroutine之间同步很棒的工具。

#### Buffered Channels

上面我们介绍了默认的非缓存类型的channel，不过Go也允许指定channel的缓冲大小，`ch:= make(chan bool, 4)`，创建了可以存储4个元素的bool 型channel。在这个channel 中，前4个元素可以无阻塞的写入。当写入第5个元素时，代码将会阻塞，直到其他goroutine从channel 中读取一些元素，腾出空间。

	ch := make(chan type, value)

	value == 0 ! 无缓冲（阻塞）
	value > 0 ! 缓冲（非阻塞，直到value 个元素）


#### Range和Close

可以通过range，像操作slice或者map一样操作缓存类型的channel，请看下面的例子

	package main

	import (
		"fmt"
	)

	func fibonacci(n int, c chan int) {
		x, y := 1, 1
		for i := 0; i < n; i++ {
			c <- x
			x, y = y, x + y
		}
		close(c)
	}

	func main() {
		c := make(chan int, 10)
		go fibonacci(cap(c), c)
		for i := range c {
			fmt.Println(i)
		}
	}

>**`for i := range c`**能够不断的读取channel里面的数据，直到该channel被显式的关闭。上面代码我们看到可以显式的关闭channel，生产者通过内置函数`close`关闭channel。关闭channel之后就无法再发送任何数据了，在消费方可以通过语法`v, ok := <-ch`测试channel是否被关闭。如果ok返回false，那么说明channel已经没有任何数据并且已经被关闭。

>**记住应该在生产者的地方关闭channel，而不是消费的地方去关闭它，这样容易引起panic**

>**另外记住一点的就是channel不像文件之类的，不需要经常去关闭，只有当确实没有任何数据要发送，或者想显式的结束range循环之类才去close**


####select语句

如果存在多个channel的时候，我们Go里面提供的关键字`select`，可以监听channel上的数据流动。`select`默认是阻塞的，只有当监听的channel中有发送或接收可以进行时才会运行，当多个channel都准备好的时候，select是随机的选择一个执行的。

有时候会出现goroutine阻塞的情况，为了避免整个程序进入阻塞,我们可以利用select来设置超时，通过如下的方式实现：

	cccc := make(chan int)
	oooo := make(chan bool)
	go func() {
		for {
			select {
			case v := <-cccc: //如果没有下面的匿名标号_:代码,永远不会触发
				println("v=", v, " received.")
			case <-time.After(3 * time.Second):
				println("timeout")
				oooo <- true
				break
			}
		}
	}()
	
	_:
	for loopChan := 0; loopChan < 10; loopChan++ {
		cccc <- loopChan
	}
	
	<-oooo


在`select`里面还有default语法，`select`其实就是类似switch的功能，default就是当监听的channel都没有准备好的时候，默认执行的（select不再阻塞等待channel）。

	select {
	case i := <-c:
		// use i
	default:
		// 当c阻塞的时候执行这里
	}

#### runtime goroutine
runtime包中有几个处理goroutine的函数：

- Goexit

	退出当前执行的goroutine，但是defer函数还会继续调用
	
- Gosched

	让出当前goroutine的执行权限，调度器安排其他等待的任务运行，并在下次某个时候从该位置恢复执行。

- NumCPU

	返回 CPU 核数量
	
- NumGoroutine

	返回正在执行和排队的任务总数
	
- GOMAXPROCS

	用来设置可以并行计算的CPU核数的最大值，并返回之前的值。