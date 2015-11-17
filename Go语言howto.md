
## Twelve Go Best Practices

[http://talks.golang.org/2013/bestpractices.slide](http://talks.golang.org/2013/bestpractices.slide)

## 修改字符串

在Go中字符串是不可变的，例如下面的代码编译时会报错：cannot assign to s[0]

	var s string = "hello"
	s[0] = 'c'

但如果真的想要修改怎么办呢？下面的代码可以实现：

	s := "hello"
	c := []byte(s)  // 将字符串 s 转换为 []byte 类型
	c[0] = 'c'
	s2 := string(c)  // 再转换回 string 类型
	fmt.Printf("%s\n", s2)

修改字符串也可写为：

	s := "hello"
	s = "c" + s[1:] // 字符串虽不能更改，但可进行切片操作
	fmt.Printf("%s\n", s)
# 坑,大坑!!!
## Go FAQ：Why is my nil error value not equal to nil? 
	type MyError string
	
	func (e *MyError) Error() string {
	    return string(*e)
	}
	
	var ErrBad = MyError("ErrBad")
	
	func bad() bool {
	    return false
	}
	
	func returnsError() error {
	    var p *MyError = nil
	    if bad() {
	        p = &ErrBad
	    }
	    return p // Will always return a non-nil error.
	}
	
	func main() {
	    err := returnsError()
	    if err != nil {
	        fmt.Println("return non-nil error")
	        return
	    }
	    fmt.Println("return nil")
	}

上面的输出结果是”return non-nil error”，也就是说returnsError返回后，err != nil。err是一个interface类型变量，其underlying有两部分组成：类型和值。只有这两部分都为nil时，err才为nil。但returnsError返回时将一个值为nil，但类型为*MyError的变量赋值为err，这样err就不为nil。解决方法： 
func returnsError() error {
    var p *MyError = nil
    if bad() {
        p = &ErrBad
    }
    return nil
}

## switch err.(type)的匹配次序 

试想一下下面代码的输出结果： 
	type MyError string
	
	func (e MyError) Error() string {
	    return string(e)
	}
	
	func Foo() error {
	    return MyError("foo error")
	}
	
	func main() {
	    err := Foo()
	    switch e := err.(type) {
	    default:
	        fmt.Println("default")
	    case error:
	        fmt.Println("found an error:", e)
	    case MyError:
	        fmt.Println("found MyError:", e)
	    }
	    return
	
	}
	
	你可能会以为会输出：”found MyError: foo error”，但实际输出却是：”found an error: foo error”，也就是说e先匹配到了error！如果我们调换一下次序呢： 
	... ...
	func main() {
	    err := Foo()
	    switch e := err.(type) {
	    default:
	        fmt.Println("default")
	    case MyError:
	        fmt.Println("found MyError:", e)
	    case error:
	        fmt.Println("found an error:", e)
	    }
	    return
	}

这回输出结果变成了：“found MyError: foo error”。 

也许你会认为这不全是错误处理的坑，和switch case的匹配顺序有关，但不可否认的是有些人会这么去写代码，一旦这么写，坑就踩到了。因此对于通过switch case来判定error type的情况，将error这个“通用”类型放在后面或去掉。
