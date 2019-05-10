
# 切片

- `make([]int,l,c)` ，`l`为长度，`c`为容量，不传`c`则容量等于长度
- 底层还是数组，通过`len()`获取长度，`cap()`获取容量
- `append`之后返回的是一个新的切片
- 扩容：
	- `capacity`小于1000时，两倍扩容
	- `capacity`大于1000时，增长因子为`1.25`，`25%`扩容
- 赋值：将一个切片赋值给另一个切片可指定索引
	- 第一个索引：指定切片的头部
	- 第二个索引：指定切片长度的尾部
	- 第三个索引：限制切片的容量

参考下面代码：
  

    a := []int{1, 2, 3, 4, 5}
    b := a[1:]
    c := a[:4]
    d := a[1:4]
    e := a[2:3:4]
    fmt.Println("a", len(a), cap(a))
    fmt.Println("b", len(b), cap(b))
    fmt.Println("c", len(c), cap(c))
    fmt.Println("d", len(d), cap(d))
    fmt.Println("e", len(e), cap(e))
    
    //打印结果
    a 5 5
    b 4 4
    c 4 5
    d 3 4
    e 1 2

- `for-range`返回的是每个元素的副本，而不是引用
- 切片在函数件传递还是以值传递的方式传递，由于切片的尺寸很小，在函数间复制和传递切片的成本也很低。在64位结构的机器上，一个切片需要24个字节，指针字段8字节，长度和容量分别需要8字节，由于与切片关联的数据包含在底层数组里面，不属于切片本身，所以将切片复制给人以数组时对底层数组大小都不会有影响。

![](https://segmentfault.com/img/bVbnCu0)

# 映射
- 键可以是任何值值，这个值的类型可以是内置的类型，也可以是结构类型，只要这个值可以使用 `==` 运算符做比较，切片、函数以及包含切片的结构类型由于具有引用意义，所以不能作为映射的键。
- 映射的值可以为任何值
- 赋值是通过指定适当的键并给这个键赋一个值来完成：`colors["red"] = "#da1337"`
- 未初始化的映射称为nil映射，不能够进行赋值
- 判断是否存在某个键
	- 同时获得值以及一个键是否存在的标记
	- 只返回键的值，通过判断键的值是否为零值判断是否存在
	 
示例：	
	
	1、
	value,exists := colors["blue"]
	if exists{
		//	
	}
	2、
	value := colors["blue"]	
	if velue != ""{
		//
	}
- 通过键来索引映射时，即便这个键不存在的时候也总会返回一个值
- 可通过`for-range`来迭代映射，`for key,value := range colors{}`
- 可通过内置的`delete`函数将一个键值对从映射中删除，`delete(colors,"orange")`
- 在函数中传递映射时，当这个映射被修改，所有对这个映射的引用都会察觉到这个修改，这个特性于切片类似，保证可以用很小的成本来复制映射。

# 方法
方法能给类型添加行为，实际也是函数，只不过在func和方法名增加了一个参数

	func (u user) notify{
		//
	}

关键字func和函数名之间的参数被称作接受者，将函数与接收者的类型绑在一起。

go中有两种类型的接收者：值接收者和指针接收者。

	
	package main
	
	import (
		"fmt"
	)
	
	type User struct {
		name string
		age  int
	}
	
	//值接收者声明方法，调用时使用这个值的副本来执行
	func (u User) method1() {
		u.name = "method1_name"
	}
	//指针接收者声明方式，调用时会共享接收者所指向的值
	func (u *User) method2() {
		u.name = "method2_name"
	}
	
	func main() {
		
		user1 := User{"Dack1", 1}
		user1.method1()
		fmt.Println(user1) 
	
		user2 := &User{"Dack2", 2}
		//go为了支持这种方法调用，调整了指针的值，
		//执行的背后操作 (*user2).method1()
		user2.method1()		
		fmt.Println(user2) 
			
		user3 := User{"Dack3", 3}
		user3.method2()
		//执行的背后操作 (&user3).method2()
		fmt.Println(user3) 
	
		user4 := &User{"Dack4", 4}
		user4.method2()
		fmt.Println(user4) 
	}

执行结果：
	
	{Dack1 1}
	{method2_name 2}
	&{Dack3 3}
	&{method2_name 4}

# 并发
Go语言的并发指的是能让某个函数独立于其他函数运行的能力，当一个函数创建为goroutine时，Go会将其视为一个独立的工作单元，这个单元会被调度到可用的逻辑处理器上执行，逻辑处理器管理所有goroutine并分配执行时间。

Go语言的并发同步模型来自一个叫作通信顺序进程(Communicating Sequential Process, CSP)的范型。CSP是一个消息传递模型，通过在goroutine之间来传递消息，而不是对数据进行加锁来实现同步访问。

goroutine之间同步和传递数据的数据类型是通道(channel)。


### 调度模型

操作系统会在物理处理器上调度线程来运行，而Go语言的运行时会在逻辑处理器上调度goroutine来运行。每个逻辑处理器都会分别绑定到单个操作系统线程。在1.5版本上，**Go语言的运行时默认会为每个可用的物理处理器分配一个逻辑处理器**。在1.5版本之前，默认给整个应用程序只分配一个逻辑处理器。

创建一个goroutine并准备运行，这个goroutine就会被放到调度器的**全局运行队列**中，之后调度器将这些队列中的goroutine分配给一个逻辑处理器，并放到这个**逻辑处理器对应的本地运行队列**中。

![](https://segmentfault.com/img/bVbocFP)


#### IO阻塞情况
在运行的goroutine阻塞时，如打开一个文件，线程和goroutine会从逻辑处理器上分离，该线程会继续阻塞，等待系统调用的放回。与此同时，调度器会**创建一个新线程并将其绑定到该逻辑处理器上**，调度器与从本地运行队中选择另一个goroutine来运行，但被阻塞的系统调用执行完成并返回，对应的goroutine会放回本地运行队列，而之前的线程会保存好，以便之后可以继续使用。

#### 网络IO模型
当goroutine遇到网络I/O调用时，goroutine会和逻辑处理器分离，并移动到集成了**网络轮询器的运行时**，当轮询器执行某个网络操作以及就绪，对应的goroutine就会重新分配到逻辑处理器上来完成操作。

调度器可以创建的逻辑处理器没有数量限制，但是语言运行时默认最多10000个线程。这个值可通过runtime/debug包的SetMaxThreads修改。

### 并行
当有多个逻辑处理器时，调度器会将goroutine平等分配到每个逻辑处理器上，这会让goroutine在不同的线程上运行。不过想要真正的实现并行效果，用户需要让自己的程序运行在有多个物理处理器的机器上

![](https://segmentfault.com/img/bVbocOJ)
 



# 枚举
> 不得不说iota真的是个很神奇的东西...下面是对go枚举的理解

- 不同的const定义块互不干扰
- 所有注释行和空行全部忽略
- 没有表达式的常量定义复用上一行的表达式
- 从第一行开始，iota从0逐行加一



# 内存管理

	
	//活着就是克制


# database/sql

QueryRow执行一次查询，并期望返回最多一行结果（即Row）。QueryRow总是返回非nil的值，直到返回值的Scan方法被调用时，才会返回被延迟的错误。（如：未找到结果）



















# 数组和切片
### 数组
数组时值传递

### 切片
切片是引用传递

### 值传递跟指针传递
- 值传递会将数组的复制一遍，如果数组非常大，那么将消耗非常大的内存
- 指针传递只需要传递8个字节。
- 指针传递也有弊端，多个变量指向同一个指针，其他一个指向改变了，其他的都会改变


# 并发
goroutine是GO并行设计的核心，goroutine说到底其实就是协程，十几个goroutine可能体现在底层就是五六个线程，Go语言内部帮你实现了这些goroutine之间的内存共享。

goroutine是通过Go的runtime管理的一个线程管理器，goroutine通过go关键字实现了，其实就是一个普通的函数

	

	func say(s string){
		for i := 0;i<5;i++{
			runtime.Gosched()	//让cpu把时间片让给别人，下次某个时候继续恢复该goroutine
			fmt.Println(s)	
		}
	}
	
	func main(){
		go say("world")	//开一个新的Goroutine执行
		say("hello")	//当前的Goroutine执行
	}



在Go 1.5之前调度器仅使用单线程，也就是说只实现了并发，想要发挥多核处理器的秉性，需要在我们的程序中显示调用runtime.GOMAXPROCS(n)告诉调度器同时使用多个线程。

### channels
    
> 类似于Java的BlockingQueue

goroutine运行在相同的地址空间，因此访问共享内存必须做好同步，Go提供了一个很好的通信机制channel，channel可以与unix shell中的双向管道做类比：可以通过它发送或者接受值。这些值只能是特定的类型，channel类型。

定义一个人channal时，也需要定义发送到channel的值的类型。

注意：必须使用make创建channel：


	ci := make(chan int)
	cf := make(chan interface{})


channel通过操作符`<-`来接受和发送数据



	ch <- v //发送v到channel ch
	v := <- ch //从ch中接收数据，并赋值给v





	func sum(a []int,c chan int){
		total := 0
		for _,v : range a {
			total += v
		}	
		c <- total  //send total to c
	}

	func main(){
		a := []int{1,2,8,6,3,4,1}
	
		c := make(chan int)
		go sum(a[:len(a)/2],c)
		go sum(a[len(a)/2:],c)
		x,y := <- c,<- c	//receive from c
		fmt.Println(x,y,x+y)
	}



默认情况下，channel接收和发送数据都是阻塞的，除非另一端已经准备好，这样就使得Goroutines同步变得更加简单，而不需要显示的lock，所谓阻塞，也就是要读取(value <- ch)它将会被阻塞，知道有数据接收。其次，任何发送(ch <- 5)将会被阻塞，知道数据被读出。无缓冲channel是在多个goroutine之间同步很棒的工具。

### Buffered Channels

上面我们介绍了默认的非缓存类型的channel，不过Go也允许指定channel的缓冲大小，很简单，就是channel可以存储多少元素。ch:= make(chan bool, 4)，创建了可以存储4个元素的bool 型channel。在这个channel 中，前4个元素可以无阻塞的写入。当写入第5个元素时，代码将会阻塞，直到其他goroutine从channel 中读取一些元素，腾出空间。


ch := make(chan type, value)


当 value = 0 时，channel 是无缓冲阻塞读写的，当value > 0 时，channel 有缓冲、是非阻塞的，直到写满 value 个元素才阻塞写入。

我们看一下下面这个例子，你可以在自己本机测试一下，修改相应的value值
		
	package main
	
	import "fmt"
	
	func main() {
		c := make(chan int, 2)//修改2为1就报错，修改2为3可以正常运行
		c <- 1
		c <- 2
		fmt.Println(<-c)
		fmt.Println(<-c)
	}
    //修改为1报如下的错误:
    //fatal error: all goroutines are asleep - deadlock!

### Range和Close
在Go中也可以通过Range来操作Channel
	func fibonacci(n int, c chan int) {
		x, y := 1, 1
		for i := 0; i < n; i++ {
			c <- x
			x, y = y, x + y
		}
		close(c)
	}
	func main(){
		c := make(chan int ,10)
		go fibonacci(cap(c),c)
		for i := range c{
			fmt.Println(i)
		}	
	}

for-range能够不断的读取channel里面的数据，直到该channel被显示的关闭。关闭channel之后就无法发送数据了，可通过v,ok := <- ch测试channel是否被关闭，如果ok返回false，说明channel已经没有数据并且被关闭。

### Select
当存在多个channel的时候，Go提供了关键字select，通过select可以监听多个channel上的数据流动。

select默认时阻塞的，只有监听的channel可以进行时才会运行，当多个channel都准备好的时候，随机选择一个执行。

	func fibonacci(c,quit chan int){
		x ,y := 1, 1
		for{
			select{
				case c <- x:
					x, y = y, x + y
				case <-quit:
					fmt.Println("quit")
					return
			}
		}
	}
在select里面还有default语法，select其实就是类似switch的功能，default就是当监听的channel都没有准备好的时候，默认执行的（select不在阻塞等待channel)。

	select{
		case i := <-c :
			// use i
		default:
			//当c阻塞的时候执行这里
	}

### 超时
可通过select来设置超时避免整个程序进入阻塞

	select{
		case v := <- c:
			//
		case <- time.After(t * time.Second):
			println("timeout")
			//
			break
	}


[https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.7.md](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.7.md "build-web-application-with-golang/zh/02.7.md")

# struct

go中struct结构默认字段都会有零值，故不能用nil来判断struct是否为空，可通过额外的字段来判断struct是否填充或为空
	
	type Demo struct{
		ready bool
	
		name string
		//其他字段
	}
在初始化的时候必须将ready设置为true

	var d Demo
	if !d.ready{
		//do stuff
	}




# Web工作方式
- Go通过`ListenAndServer`来建立web服务器，底层是初始化一个`server`对象，然后调用`net.Listen("tcp",addr)`来监听端口。
- 调用`srv.server(net.Listener)`函数来处理接收客户端请求。函数里面为一个`for{}`，首先通过`accept`接受请求，接着创建一个Conn，最后单独开一个goroutine取执行：`go c.server()`。
- 用户的每一次请求都是一个新的goroutine去执行。
- conn通过解析request`c.readRequest()`获取相应的`handler := c.server.Handler`，它本质是一个路由器，通过它来匹配url跳到对应的handle函数。
- 可通过`http.HandleFunc("/",sayhelloName)来注册请求的路由规则。

# OS获取环境变量
os.getenv()获取环境变量获取不到最新设置的环境变量，最新设置的需要重新启动电脑获取


# 基本类型
> 这两天在搞反射，看到Go的基础数据类型那么多，int,int32,int64都有，而且运算过程中还需要转换，所以抽空看了些博客以及官方文档。

- int跟uint
	- 有符号：int8,int16,int32,int64
	- 无符号：unit8,unit16,unit32,uint64
	- int和unit取决于操作系统，在32位系统就是32字节，在64位系统就是64字节
	- int跟int32不是相同的类型，虽然在特定的场景下它们大小相同，但是在运算过程中需要转换
	- byte是unit8的别名，rune是int32的别名
- 浮点类型为float32和float64
	- 浮点类型在运算过程中可能存在精度丢失的情况
- string
	- 字符串是不可变的，一旦创建，就不能改变字符串的内容
	- 可以使用内置函数len来发现s的长度，如果字符串为常量，则length是编译时常量。
	- 字符串的字节可以通过索引来获取，但是取元素的地址是非法的，即&s[i]是无效的。



# 反射
> 反射在计算机中是程序检查自己结构的一种能力，尤其是通过类型，它是元数据编程的一种方式，也是混乱的重要来源
> 
> 每个语言的反射模型都不同（很多语言根本不支持它）


### Type And  interface
因为反射是建立在类型系统上，让我们先回顾一下Go中的类型。

Go是静态类型语言，每个变量都有一个静态类型，即在编译时就已知被固定上一种类型：`int, float32, &MyType, []byte`


	type MyInt int
	
	var i int
	var j MyInt

变量i和j具有不同的静态类型，虽然他们具有相同的底层类型，但如果没有转换，则无法将他们分配给彼此。

interface可以存储任何具体的值，interface包括：

- 变量包括（type,value）两部分，这也是为什么nil != nil的原因
- type包括static type和concrete type，static type是编辑时就看到的类型，而concrete type是runtime看到的类型

反射就是建立在类型之上的，Golang的指定类型的变量的类型是静态的（也就是指定int、string这些变量，它的type是static type），在创建变量的时候就已经确定，反射主要于Golang的interface有关（它的类型是concrete type），只有interface类型才有反射之说。


### API
以下是一些API

	reflect:
		TypeOf(interface{}) Type : 返回接口中保存值的类型，i为nil值返回nil
		ValueOf(interface{}) Value : 返回一个初始化为i接口保管的具体值的Value，但i为nil时返回Value零值
		New(Type) Value：返回一个指向类型为Type的新申请的零值的指针。
	
	Type:
		Kind()：返回该接口的具体类型
		Name()：返回类型名
		Elem()：返回该类型的元素类型，如果Kind不是Array,Chan,Map,Slice,Ptr会panic
	
	Value:
		Append(s Value,x ...Value) Value: s需为切片类型的Value值，x需为s的元素类型的Value值，将x复制给s并且返回s
		Type()：返回v持有的类型的Type表示
		Elem() Value：返回v持有的接口或者指针保管值的Value封装，如果v的Kind不是interface或者Ptr将会panic
	    Kind()：同上一致
		CanSet()：判断v持有的值是否能更改，只有当Value持有值为Ptr并且为共有类型时，它才可以被修改。
		Set(x Value)：将v的持有值修改为x的持有值
		SetInt(x Int64)
		SetString(s string)
		....

更多的可参考官方文档：[https://go-zh.org/pkg/reflect/#Value.Convert](https://go-zh.org/pkg/reflect/#Value.Convert "https://go-zh.org/pkg/reflect/#Value.Convert")

反射讲得比较好的一篇文章：[https://juejin.im/post/5a75a4fb5188257a82110544](https://juejin.im/post/5a75a4fb5188257a82110544 "Golang的反射reflect深入理解和示例")

# Go运行时
尽管Go编译器产生的是本地可执行代码，这些代码仍旧运行在Go的runtime（这部分的代码可以在runtime包中找到）当中，这个runtime类似虚拟机，它负责管理包括内存分配、垃圾回收、栈处理、goroutine、channel、slice、map和reflection等等。


