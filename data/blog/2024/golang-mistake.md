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

在这里，**两个参数**3是切片的**长度**，6是切片的**容量**，灰色的部分是内存已经分配了，但是没有被使用的部分

![image-20240802210709732](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240802210709732.png)

我们要读取切片内容只能在len的范围内，如果读s[4]（即使此时还在分配内存的范围内），那就会报`index out of range`的err

如果当我们通过append新增元素超过cap时，Go就会创建一个新的数组，同时复制所有当前元素过去，新的数组的长度是原来的**两倍**

![image-20240802210945668](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240802210945668.png)

> 在Go中，切片小于等于1024时，每次扩容增加一倍，超过1024以后，每次扩容增加25%

而原来的数组，会在GC后被释放

### 切割append问题

```go
	s1 := make([]int, 3, 6) // [0 ,0, 0]
	s2 := s1[1:3] // [0, 0]
	s1[1] = 1   // [0, 1, 0]
	fmt.Print(s2) // [1, 0]

	s2 = append(s2, 2)
	fmt.Print(s1) // [0, 1, 0]
	fmt.Print(s2) // [1, 0, 2]

	s1 = append(s1, 4)
	fmt.Print(s1) // [0, 1, 0, 4]
	fmt.Print(s2) // [1, 0, 4] 这个有没有超出预期?
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

为了避免上面的不可预知问题，有**两种方式**来解决

1. **使用copy来实现切片间的复制**， 两者就没有引用上的关联，自然就没有上面的问题，但是它也有隐藏的问题，比如切片非常大时，复制就非常消耗性能， 还有一个问题就是copy只能从头开始复制，不能从中间复制
2. 使用另一种**切片语法**`s[low：high：max]`，max的意义是生成的切片的容量等于max-low，在这个例子里

```
	s1 := make([]int, 3, 6) // [0 ,0, 0]
	s2 := s1[1:3:3] // [0, 0] 限定了s2的cap是2 (3-1)
	s1[1] = 1   // [0, 1, 0]
	fmt.Print(s2) // [1, 0]

	s2 = append(s2, 2) // 此时s2已经和s1脱钩，因为s2已经超过他原本的cap
	fmt.Print(s1) // [0, 1, 0]
	fmt.Print(s2) // [1, 0, 2]

	s1 = append(s1, 4)
	fmt.Print(s1) // [0, 1, 0, 4]
	fmt.Print(s2) // [1, 0, 2]  符合预期
```



## # 21 低效的切片初始化

```go
func convertEmptySlice(foos []Foo) []Bar {
	bars := make([]Bar, 0)

	for _, foo := range foos {
		bars = append(bars, fooToBar(foo))
	}
	return bars
}
```

这里例子初始化切片传了一个0值的长度（长度这个参数是**必传**的）且没有传容量。然后，使用append添加Bar元素。起初，bars是空的，因此添加第一个元素会分配一个大小为1的底层数组。每当底层数组被充满时，Go都会创建一个2倍于当前容量的新数组

在我们添加第3个元素、第5个元素、第9个元素时等，这个创建数据的逻辑都会重复。假设输入切片有1000个元素，该算法需要分配10个底层数组，并将1000多个元素从一个数组复制到另一个。这导致GC需要付出很多额外的工作来清理所有这些临时的底层数组。

解决这个问题，我们必须通过指定合适大小的长度或者容量来初始化

```go
func convertGivenCapacity(foos []Foo) []Bar {
	n := len(foos)
	bars := make([]Bar, 0, n)

	for _, foo := range foos {
		bars = append(bars, fooToBar(foo))
	}
	return bars
}
```

如果我们不指定长度，指定容量。在内部Go预先分配了一个由n个元素组成的数组。因此，添加n个元素都会使用同一个底层数组，从而大幅减少分配数组的数量。

```go
func convertGivenLength(foos []Foo) []Bar {
	n := len(foos)
	bars := make([]Bar, n)

	for i, foo := range foos {
		bars[i] = fooToBar(foo)
	}
	return bars
}
```

又或者不指定容量，只指定长度。 但这里要注意的是添加元素只能用下标来改变而不能用append，除非容量已经充满

经过测试100w量级的数据，后面两种的速度是不指定版本的3倍以上



## # 22 空切片与nil切片

- 空切片：长度为0，值不为nil
- nil切片：类型是切片，长度为0，值为nil

```go
func main() {
	var s []string
  log(1, s) // 1: empty=true nil=true

	s = []string(nil)
	log(2, s) // 2: empty=true nil=true

	s = []string{}
	log(3, s) // 3: empty=true nil=false

	s = make([]string, 0)
	log(4, s) // 4: empty=true nil=false
}

func log(i int, s []string) {
	fmt.Printf("%d: empty=%t\tnil=%t\n", i, len(s) == 0, s == nil)
}
```

两者类型相同（后续能做的操作也相同比如append），主要区别是是否有资源分配，显然nil切片是**不需要**资源分配的

```go
var s []string  // 使用最广
s = []string(nil)  // 几乎用不到，只要是语法中的快捷方式
s = []string{}   // 用来初始化包含初始元素的切片
s = make([]string, 0) // 创建一个已知长度的切片
```

还要注意在encoding/json这种包的编码问题， 如果是nil切片，结果是null，而如果是空切片，结果是`[]`

最后如何判断一个切片是否为空，最好的办法不是判断切片是不是nil，而是去判断`len(s)`因为不管是nil切片还是空切片，len结果都是0，这样就不需要去做主动区分了



## #24 无法正确复制切片

如上提到，复制切片最好是使用copy，但copy可能也会有些常见错误

```go
func bad() {
	src := []int{0, 1, 2}
	var dst []int
	copy(dst, src)
	fmt.Println(dst) // []
}

func correct() {
	src := []int{0, 1, 2}
	dst := make([]int, len(src))
	copy(dst, src)
	fmt.Println(dst) // [0 ,1, 2]
}
```

失败的原因是copy只会复制源和目标中长度的**最小值**，所以本例中最小值是nil切片的长度0，所以就没复制成功。（换句话说copy不会帮助我们扩展切片的长度）

如果不使用copy，还有一种更快捷的方式

```go
src := []int{0, 1, 2}
dst := append([]int(nil), src...)
```

