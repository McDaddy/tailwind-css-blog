---
title: GoLang - 《100个GO语言典型错误》笔记
date: 2024-05-28
tags:
 - GoLang
lastmod: 2024-05-28
draft: false
summary: '《100个GO语言典型错误》笔记'
---



##  \# 1 意想不到的变量隐藏

下面这段代码中（可以正常编译），在if/else之后，最后打印出来的client是nil

```go
func listing1() error {
	var client *http.Client
	if tracing {
		client, err := createClientWithTracing()
		if err != nil {
			return err
		}
		log.Println(client)
	} else {
		client, err := createDefaultClient()
		if err != nil {
			return err
		}
		log.Println(client)
	}

  log.Println(client)  // nil
	return nil
}
```

明明前面的逻辑是有赋值的，但为什么最后是nil呢？

If/else中的`:=` 定义的是内部的client，和外部的client无关

在Go中，**一个块中声明的变量名可以在它内部的块里面被重新声明**，这被称为*变量隐藏*。内外虽然是同一个名字的变量，但是**完全不相干**，代码极端点，可以两者不同类型，也是不会报错的

```go
func listing1() error {
	var client *http.Client
	if true {
		client := "ssss"
		fmt.Println(client)
	} 
	fmt.Print(client)
	return nil
}
```

怎么解决？

方法一： 定义一个临时变量，然后把它赋值给外部的这个变量

```go
func listing2() error {
	var client *http.Client
	if true {
		c, err := createClientWithTracing()
		if err != nil {
			return err
		}
		client = c
	}

	_ = client
	return nil
}
```

方法二：用`=`替换`:=`，这样就不会有声明，而仅仅是赋值，这样就不会有变量隐藏的问题了

```go
func listing3() error {
	var client *http.Client
	var err error
	if true {
		client, err = createClientWithTracing()
		if err != nil {
			return err
		}
	}

	_ = client
	return nil
}
```



## # 3 滥用init函数

init函数是用于初始化应用程序状态的函数，它没有入参，也没有出参。它是在初始化包的时候被执行的，且不能被显式调用

它可能会带来几个问题

1. 一个包可以定义多个init函数，执行的顺序是基于源文件字母顺序的，a.go的执行会早于b.go（两者同包），如有相互依赖是有风险的
2. init的错误管理是有缺陷的，因为它不能返回任何东西，所以也就不能返回err，唯一handle错误的方式就是panic，如果是在一个lib里面写init，那它的调用方很有可能因为这个包而发生意料外的painc
3. 不方便测试，如果写单测，有些初始化逻辑是不需要的，但是就无法跳过了
4. 如果init函数要给什么变量赋值，那只能赋值给全局变量，但事实上全局变量太容易被篡改，所以不建议这么做

所以总之，我们还是尽量避免使用init函数



## # 8 any意味着nothing

没有指定任何方法的接口类型称为**空接口**，即`interface{}`

Go 1.18开始，加入了any类型，也就是空接口的别名，所有interce{} 都可以被any替换

一旦使用any，变量就可以被赋各种类型的值，且不会编译报错。但这样也失去了Go静态语言的优势，我们应该能不用就不用

只有少数情况是可以使用的，比如Marshal方法，它无法预知入参会是什么类型的结构体，所以就是any



## # 10 没有意识到类型嵌入可能存在的问题

如下代码，Bar在Foo中是一个嵌入的类型，我们可以从foo实例里面**直接获得**Baz变量

```go
type Foo struct {
	Bar
}

type Bar struct {
	Baz int
}

func fooBar() {
	foo := Foo{}
	foo.Baz = 42
	foo.Bar.Baz = 42 // 两者指向同一个变量
}
```

但这样可能会有些隐藏的问题

```go
type InMem struct {
	sync.Mutex
	m map[string]int
}

func New() *InMem {
	return &InMem{m: make(map[string]int)}
}

func (i *InMem) Get(key string) (int, bool) {
	i.Lock()
	v, contains := i.m[key]
	i.Unlock()
	return v, contains
}
```

上面这个例子，InMem这个类型，不想暴露map，同时想实现加锁，就把Mutex直接嵌入到结构体里。 这样看起来非常方便，但是如果外部在使用inMem的实例时，可以直接调用`inMem.Lock()` ，那这样是不是也很离谱？

所以一般情况下，我们只是把嵌入结构作为一种简化的语法糖（foo.Bar.Baz => foo.Baz），但这个的意义其实并不大

此外，如果被嵌入的结构里有了跟嵌入结构体中相同的成员名，那么这个语法糖也无效了

```
type Foo struct {
	Bar
	Baz string
}

type Bar struct {
	Baz int
}

func fooBar() Foo {
	foo := Foo{}
	foo.Baz = "42"
	foo.Bar.Baz = 42
	return foo
}
```



## # 17 使用八进制字面量会带来混淆

下面这段代码，结果并不是110，而是108

```go
sum := 100 + 010
fmt.Println(sum)
```

因为在Go里，0开头的数字指代八进制整数，八进制的10就是十进制的8，此外0o10也是同样的效果

此外

> 二进制用 0b或者0B开头
>
> 十六进制用0x或者0X开头
>
> 我们可以随意在数字的中间加下划线，增加可读性比如 100_00_0_0



## # 18 容易忽视的整数溢出

Go总共有10种整数类型，分别是int8/16/32/64 和 uint8/16/32/64， 外加int和uint

其中int和uint的长度取决于操作系统，在32位系统中就是32位，64位中就是64位

```go
var count int32 = math.MaxInt32
count++
fmt.Print(count)
```

这段代码可以编译和执行，运行时也不会报错，但最终打印的结果是`-2147483648`。这就是整数溢出的结果。在Go中整数溢出是静默的，不会产生panic，所以在做操作时要非常小心



## # 20 不了解切片的长度和容量

首先，在Go里面，为什么不直接称呼切片叫数组？在Go里，数组是指固定长度的数据结构，比如int[5] 就是一个数组，而切片相对于数组是变长的，可以动态增减

```go
s := make([]int, 3, 6)
```

在这里，3是切片的长度，6是切片的容量，灰色的部分是内存已经分配了，但是没有被使用的部分

![image-20240802210709732](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240802210709732.png)

我们要读取切片内容只能在len的范围内，如果读s[4]，那就会报`index out of range`的err

如果当我们通过append新增元素超过cap时，Go就会创建一个新的数组，同时复制所有当前元素过去，新的数组的长度是原来的**两倍**

![image-20240802210945668](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240802210945668.png)

> 在Go中，切片小于等于1024时，每次扩容增加一倍，超过1024以后，每次扩容增加25%

而原来的数组，会在GC后被释放

### 切割问题

```go
	s1 := make([]int, 3, 6) // [0 ,0, 0]
	s2 := s1[1:3] // [0, 0]
	s1[1] = 1
	fmt.Print(s2) // [1, 0]

	s2 = append(s2, 2)
	fmt.Print(s1) // [0, 1, 0]
	fmt.Print(s2) // [1, 0, 2]

	s1 = append(s1, 4)
	fmt.Print(s1) // [0, 1, 0, 4]
	fmt.Print(s2) // [1, 0, 4]
```

运行上面的代码，虽然s2从s1上切出来了，但是它们各自的改动还是在影响着对方，下面简述下原因

![image-20240802212147429](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240802212147429.png)

s2与s1共享一段数组，只是s2从第二位开始，同时容量比s1小1

![image-20240802212308644](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240802212308644.png)

所以s1改动的第二位其实也就是s2的第一位

![image-20240802212358774](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240802212358774.png)

同理，s2添加一位也是在同一个数组上操作， 但是，此时s1是读不到这个2的，因为它的长度还是3

最后s1添加了一个4后，s1的长度来到了4，而这个4也把刚刚s2添加的2给覆盖了

如果此时，我们再往s2里添加3个元素，s2就会扩容，从此s1和s2彻底无关

![image-20240802212747507](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240802212747507.png)

总之，切片的长度是切片中**可用元素的数量**，容量是切片**底层数组中的数量**，如果两者相等，代表底层数组满了，再往里添加元素就会把当前所有元素复制到一个新的数组里，同时切片指向新的数组

为了避免上面的不可预知问题，