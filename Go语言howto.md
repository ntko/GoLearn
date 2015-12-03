
## Twelve Go Best Practices

[http://talks.golang.org/2013/bestpractices.slide](http://talks.golang.org/2013/bestpractices.slide)

## Practical Go: C Bindings
[https://codegangsta.io/blog/2013/07/08/c-bindings-in-go-a-practical-example/](https://codegangsta.io/blog/2013/07/08/c-bindings-in-go-a-practical-example/)

I have been playing with Go lately. The language itself has a lot going for it, one of which is a decent set of interop with existing C code. Today I am going to walk you through a practical example of how this is done by showing some code that I have been working on lately with Go.

### The Simplest Hello World

Assuming you have your Go development environment all set up, go ahead and create a new go file with the following contents:
package main
	
	func main() {
	  println("Hello world")
	}


Running this should obviously print “Hello world”. Let’s move on!

### Using libspotify

Lets start creating some C bindings for libspotify! libspotify is a C api distributed by Spotify that basically allows us to do everything a typical spotify app can do; log in, play, music, manipulate playlists - all available to us with libspotify.

For the sake of keeping this post short and to the point, we will just be creating a sp_session object. This session will lay some groundwork for us to log in and do the fun stuff libspotify allows us to do. Why don’t we dive in:
package main

	type Session struct {
	  session *C.sp_session
	}
	
	func main() {
	  println("Hello world")
	}


Running this will result in an error
> ./bindings.go:4: undefined: C


This is because we actually need to use some special magic that Go provides for us, the C package.
package main

	import "C"
	
	type Session struct {
	  session *C.sp_session
	}
	
	func main() {
	  println("Hello world")
	}


This gives us a different error:

>error: 'sp_session' undeclared (first use in this function)  
>error: (Each undeclared identifier is reported only once


Cool, so it sees the C package, but sp_session doesn’t exist. This is because we need to include libspotify using the #include directive for the C preprocessor. First make sure you have libspotify installed. On a Mac you can run brew install libspotify. Now try to run the following code:
package main

	/*
	#include <libspotify/api.h>
	*/
	import "C"
	
	type Session struct {
	  session *C.sp_session
	}
	
	func main() {
	  println("Hello world")
	}


This should compile and run since sp_session now resides in the global C namespace! This works great for things that exist in the header, but it breaks down when we try to initialize our session:
	package main
	
	/*
	#include <libspotify/api.h>
	*/
	import "C"
	import "unsafe"
	
	type Session struct {
	  session *C.sp_session
	}
	
	func main() {
	  key := "appkey_good"
	  session := Session{}
	  appkey := C.CString(key)
	  appkey_size := len(key)
	
	  var config = C.sp_session_config {
	    api_version:          C.SPOTIFY_API_VERSION,
	    cache_location:       C.CString(".spotify/"),
	    settings_location:    C.CString(".spotify/"),
	    user_agent:           C.CString("spotify for go"),
	    application_key:      unsafe.Pointer(appkey),
	    application_key_size: C.size_t(appkey_size)
	  }
	
	  C.sp_session_create(&config, &session.session)
	}


Will result in the following error:
> Undefined symbols for architecture x86_64:
  "_sp_session_create", referenced from:
      __cgo_90280c6a3021_Cfunc_sp_session_create in bindings.cgo2.o
     (maybe you meant: __cgo_90280c6a3021_Cfunc_sp_session_create)
ld: symbol(s) not found for architecture x86_64
collect2: ld returned 1 exit status


This is because we didn’t link libspotify. This is an easy fix, add this in the comment above import "C":
> cgo LDFLAGS: -lspotify.12


Your code should compile and link fine. You will get a SIGSEGV in your application, this is becuase we specified a bogus app key. We will cover this in detail during Part 2 of C Bindings in Go: A Practical Example.

Until next time!


## json相关

	package main
	
	import (
	    "encoding/json"
	    "fmt"
	)
	
	type Server struct {
	    ServerName string
	    ServerIP   string
	}
	
	type Serverslice struct {
	    Servers []Server
	}
	
	func main() {
	    var s Serverslice
	    s.Servers = append(s.Servers, Server{ServerName: "Shanghai_VPN", ServerIP: "127.0.0.1"})
	    s.Servers = append(s.Servers, Server{ServerName: "Beijing_VPN", ServerIP: "127.0.0.2"})
	    b, err := json.Marshal(s)
	    if err != nil {
	        fmt.Println("json err:", err)
	    }
	    fmt.Println(string(b))
	}

输出如下内容：
	
	 {"Servers":[{"ServerName":"Shanghai_VPN","ServerIP":"127.0.0.1"},{"ServerName":"Beijing_VPN","ServerIP":"127.0.0.2"}]}

我们看到上面的输出字段名的首字母都是大写的，如果你想用小写的首字母怎么办呢？**把结构体的字段名改成首字母小写的是不行的！**

JSON输出的时候必须注意，只有导出的字段才会被输出，如果修改字段名， 必须通过struct tag定义来实现：

	type Server struct {
	    ServerName string `json:"serverName"`
	    ServerIP   string `json:"serverIP"`
	}
	
	type Serverslice struct {
	    Servers []Server `json:"servers"`
	}

通过修改上面的结构体定义，输出的JSON串就和我们最开始定义的JSON串保持一致了。

针对JSON的输出，我们在定义struct tag的时候需要注意的几点是:

* 字段的tag是 "-" ，那么这个字段不会输出到JSON
* tag中带有自定义名称，那么这个自定义名称会出现在JSON的字段名中，例如上面例子中serverName
* tag中如果带有 "omitempty" 选项，那么如果该字段值为空，就不会输出到JSON串中
* 如果字段类型是bool, string, int, int64等，而tag中带有 ",string" 选项，那么这个字段在输出到JSON的时候会把该字段对应的值转换成JSON字符串

举例来说：
	
	type Server struct {
	    // ID 不会导出到JSON中
	    ID int `json:"-"`
	
	    // ServerName 的值会进行二次JSON编码
	    ServerName  string `json:"serverName"`
	    ServerName2 string `json:"serverName2,string"`
	
	    // 如果 ServerIP 为空，则不输出到JSON串中
	    ServerIP   string `json:"serverIP,omitempty"`
	}
	
	s := Server {
	    ID:         3,
	    ServerName:  `Go "1.0" `,
	    ServerName2: `Go "1.0" `,
	    ServerIP:   ``,
	}
	b, _ := json.Marshal(s)
	os.Stdout.Write(b)

会输出以下内容：

	{"serverName":"Go \"1.0\" ","serverName2":"\"Go \\\"1.0\\\" \""}

Marshal函数只有在转换成功的时候才会返回数据，在转换的过程中我们需要注意几点：

* JSON对象只支持string作为key，所以要编码一个map，那么必须是map[string]T这种类型(T是Go语言中任意的类型)
* Channel, complex和function是不能被编码成JSON的
* 嵌套的数据是不能编码的，不然会让JSON编码进入死循环
* 指针在编码的时候会输出指针指向的内容，而空指针会输出null



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

## 不需要web框架,只需要库

### 示例 Gzip所有输出:

	type gzipResponseWriter struct {
	  io.Writer
	  http.ResponseWriter
	}
	
	func (w gzipResponseWriter) Write(b []byte) (int, error) {
	  return w.Writer.Write(b)
	}
	
	func makeGzipHandler(fn http.HandlerFunc) http.HandlerFunc {
	  return func(w http.ResponseWriter, r *http.Request) {
	    if !strings.Contains(r.Header.Get("Accept-Encoding"), "gzip") {
	      fn(w, r)
	      return
	    }
	    w.Header().Set("Content-Encoding", "gzip")
	      gz := gzip.NewWriter(w)
	      defer gz.Close()
	      fn(gzipResponseWriter{Writer: gz, ResponseWriter: w}, r)
	    }
	}

>INTERFACES

	type Handler interface {
	  ServeHTTP(ResponseWriter, *Request)
	}

>Just pass in your own Handler and gzip all requests with Accept-Encoding: gzip You can also do some other useful stuff in there...

>USEFUL STUFF...

	func (s *server) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	  // Log every request...
	  log.Printf("%s %s %s %s", req.Proto, req.Method, req.RemoteAddr, req.URL)
	
	  // Track request latency to StatHat...
	  defer func(t time.Time) {
	    name := fmt.Sprintf("MyServer[%s][%s]", req.Method, req.URL.Path)
	    stathat.PostEZValue(name, "...", time.Since(t).Seconds()*1000)
	  }(time.Now())
	  
	  ... gzip, let the muxer do its thing.
	}

###更多关于GZIP

* `Gzip Performance for Go Webservers`:
	* [https://blog.klauspost.com/gzip-performance-for-go-webservers/](https://blog.klauspost.com/gzip-performance-for-go-webservers/)
	
相关的资源:

* [https://github.com/xi2/httpgzip](https://github.com/xi2/httpgzip)
* [https://github.com/klauspost/compress](https://github.com/klauspost/compress)