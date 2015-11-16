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