## 介绍
这是一个go语言的手册，想要了解更多的文档或信息可以访问[golang.org](https://golang.org/)。

Go是一种通用的系统编程语言，它是强类型，有牢记回收机制并且原生的支持并发编程。使用Go开发的程序通过package的概念来高效的管理依赖。而传统的方式是通过在编译时添加到二进制文件的链接。

Go的语法简洁且有规则，这让集成开发环境的开发工具可以很容易的分析代码。

## 标记

语法采用巴斯科范式。

```
Production  = production_name "=" [ Expression ] "." .
Expression  = Alternative { "|" Alternative } .
Alternative = Term { Term } .
Term        = production_name | token [ "…" token ] | Group | Option | Repetition .
Group       = "(" Expression ")" .
Option      = "[" Expression "]" .
Repetition  = "{" Expression "}" .
```

Productions are expressions constructed from terms and the following operators, in increasing precedence:

```
|   alternation
()  grouping
[]  option (0 or 1 times)
{}  repetition (0 to n times)
```

Lower-case production names are used to identify lexical tokens. Non-terminals are in CamelCase. Lexical tokens are enclosed in double quotes "" or back quotes ``.

The form a … b represents the set of characters from a through b as alternatives. The horizontal ellipsis … is also used elsewhere in the spec to informally denote various enumerations or code snippets that are not further specified. The character … (as opposed to the three characters ...) is not a token of the Go language.

## 源码描述

源码统一使用`UTF-8`编码。

## 词汇部分

#### 注释

注释将会作为程序的文档使用。有两种表达方式：

* 单行注释在`//`处开始一直到行的末尾结束。
* 通用的注释以`/*`开头，一直到`*/`处。

注释不能在字面量和rune的值中。通用注释会被识别为空格，而其他注释都会被识别为换行符。

## 常量

常量分为：布尔型，rune型，整型，浮点型，复数型，字符串型。其中rune,整型，浮点型，复数型统称为数字常量。

一个常量可以代表rune，整型，浮点型，虚数型，字面值，常数的标识符，常量表达式，一个结果为常量类型类型转换，和一些内置的函数返回值(`unsafe.Sizeof`,`cap`,`len`,`real`,`imag`)，布尔常量的值代表的是预定义的`true`或`false`，预定义的标识符`iota`可以表示一个整型常量。

一般情况下复数常量是常量表达式的一种形式。将在本节讨论。

数字常量可以是任意精度的确定值并且能保证不会溢出。没有常量可以表示非0，无穷大和非数字值。

常量可以是指定类型或者是无类型的。字面常量，true，false，iota，和只包含无类型常量的常量表达式是无类型的。

常量也可以是指定类型的，或者从其他变量初始化值和表达式中继承类型。如果常量的值和他的类型不能匹配，那么将会产生一个错误。举个例子：3.0可以用在整型常量和其他浮点数常量上。而2147483648.0 (equal to 1<<31) 可以是float32,float64,或者uint32类型，但是不能是int32或者string类型。

一个无类型的常量会在程序的上下文中被转换成程序所需要的类型。

## 变量

变量是一个用来储存指定值的地方。根据不同的变量类型，可以储存不同的值。

一个变量声明，或函数的参数和返回值，函数声明的签名函数的值都保存在已经被命名的变量中。调用内置的new或者为符合值分配一个地址来保存一个变变量在运行的时候。结构化的变量，例如数组，切片，结构体类型都有很多元素和字段可以单独被分配地址。没个元素的行为都和变量相似。

静态类型的变量就是在声明时就确定的类型。通过new或者类型初始化。变量的也有一个不重复的动态接口类型。动态类型在程序的执行过程中会动态的改变，但是接口变量的值总是储存在静态类型的变量中。

一个变量将会在表达式中取回最近一次的赋值结果。如果变量没有被赋值，那么他的值时该类型的零值。

## 类型

一个类型代表一些值和操作这些值的方法的集合。一个类型可以用一个类型名称进行表示。

布尔类型，数字类型和字符串类型已经被预定义。其他已经被定义的类型将会在类型预定义章节说明。复合类型(数组，结构体，指针，函数，接口，切片，map，channel类型)可以使用他们的类型字面值。

每个类型T都有下游类型。如果T是预定义类型或者类型字面值。那么下游类型就是他自己。否则，T的下游类型将会是类型声明中的说到。

```go
type (
	A1 = string
	A2 = A1
)

type (
	B1 string
	B2 B1
	B3 []B1
	B4 B3
)

```

string，A1，A2，B1，B2的下游类型是string，[]B1，B3，B4的下游类型是[]B1。

#### 方法集合


一个类型可以有一个方法集。 一个接口的方法集就是他的接口。在T类型上定义的方法都需要接受一个类型为T的参数。定义在T的指针类型上的方法可以接收*T或者T，所以*T将会包括所有的T方法。在方法集中，任何一个方法都需要有一个非空唯一的方法名。

一个类型的方法集是所有这个类型实现的接口和所有接收者为该类型的可以被调用的方法。

#### 布尔类型

一个布尔类型相当于被预定于的true和false代表的真实值。预定义的布尔类型是bool。

#### 数字类型

一个数字类型相当于整型和浮点型的所有值的集合。预定义的数字类型包括：
```
uint8       the set of all unsigned  8-bit integers (0 to 255)
uint16      the set of all unsigned 16-bit integers (0 to 65535)
uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

int8        the set of all signed  8-bit integers (-128 to 127)
int16       the set of all signed 16-bit integers (-32768 to 32767)
int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
int64       the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

float32     the set of all IEEE-754 32-bit floating-point numbers
float64     the set of all IEEE-754 64-bit floating-point numbers

complex64   the set of all complex numbers with float32 real and imaginary parts
complex128  the set of all complex numbers with float64 real and imaginary parts

byte        alias for uint8
rune        alias for int32
```

n位整数是n

这里还有几个指定实现的类型：

```
uint     either 32 or 64 bits
int      same size as uint
uintptr  an unsigned integer large enough to store the uninterpreted bits of a pointer value
```
为了避免移植性问题，所有的苏子类型都是明确的，除了被uint8的别名byte和uint32的别名rune。当在表达式中使用不同的数字类型需要进行强制转换。例如：int32和int不是相同的类型即使他们在指定的平台上是相等的。

#### 字符串类型

一个字符串类型相当于字符串的所有值。一个字符串的值时一个字符序列。字符串时不能被改变的。只要字符串被创建，就无法更改其中的内容。预定义的字符串类型是`string`。

字符串的长度可以用过内置的`len`函数获取。当字符串是常量的时候，那么在编译时他的长度也将会是一个常量。一个字符串的byte集合可以通过数字下标范围0～len(s)-1。不能对字符串中的byte元素进行取址操作。

#### 数组类型

一个数字是一定数量的单一类型元素的集合。原属的格式叫做数组的长度，并且他不能为负数。

```
ArrayType   = "[" ArrayLength "]" ElementType .
ArrayLength = Expression .
ElementType = Type .
```

长度是数组类型的一部分。他肯定是一个类型为int的非负数。可以用过内置函数`len`来获取数组的长度。元素可以通过下标0～`len(a)-1`访问。数组一般都是1维，不过也可以是多维。

```
[32]byte
[2*N] struct { x, y int32 }
[1000]*float64
[3][5]int
[2][2][2]float64  // same as [2]([2]([2]float64))
```

#### 切片类型

切片是对底层数组一个连续片段的描述。并提供对数组中元素的访问。一个切片类型代表所有数组的切片。没有被初始化的切片用nil表示。
```
SliceType = "[" "]" ElementType .
```

和数组一样，切片的可以使用index是有一个长度的，切片的长度可以通过内置的`len`函数获取。和数组不同的是他的长度在运行时是可以更改的。我们可以通过下标0～`len(s)-1`来访问切片内的元素。

切片只要被初始化那么救火和底层的数组联系起来并持有数组中的元素。所以切片和底层的数组还有其他指向该数组的切片共享相同的储存空间。和数组不同的是，不同的数组总是有着不同的存储空间。

切片可以扩大长度，切片的`capacity`等于切片现在的长度加上数组中切片还没使用的长度。一个切片可以通过从原始切片中切出一个达到器容量的切片。切片的容量可用通过内置的`cap()`函数来获取。可以通过函数`make`来创建一个T类型的新切片。创建切片需要指定长度，也可选的指定容量。通过make创建的切片会分配新的切片引用的隐藏的底层数组。

```
make([]T, length, capacity)
```

由于make的作用就是创建新的数组并使用切片引用他，所以下面的写法是等价的：
```
make([]int, 50, 100)
new([100]int)[0:50]
```

和数组一样，切片总是一个维度，并且可以复合成多维。数组中的数组都必须是相同的长度，但是切片中的切片长度是动态变化的，切片中的切片需要单独初始化。

#### 结构体类型

结构体是一被命名的元素的序列，叫做字段。每个字段都有一个名字和一个类型，字段的名字可以是显式或者隐式的。在结构体中非空字段必须是唯一的。

```
StructType    = "struct" "{" { FieldDecl ";" } "}" .
FieldDecl     = (IdentifierList Type | EmbeddedField) [ Tag ] .
EmbeddedField = [ "*" ] TypeName .
Tag           = string_lit .
```

```
// An empty struct.
struct {}

// A struct with 6 fields.
struct {
	x, y int
	u float32
	_ float32  // padding
	A *[]int
	F func()
}

```

一个只有类型没有字段名的字段叫做嵌入字段，一个嵌入字段必须指定一个类型名。或者一个非接口类型的指针。并且T本身不能为指针类型。这时类型名和作为字段的名字

```
// A struct with four embedded fields of types T1, *T2, P.T3 and *P.T4
struct {
	T1        // field name is T1
	*T2       // field name is T2
	P.T3      // field name is T3
	*P.T4     // field name is T4
	x, y int  // field names are x and y
}
```

下面的声明时错误的因为字段名字必须时唯一的。
```
struct {
	T     // conflicts with embedded field *T and *P.T
	*T    // conflicts with embedded field T and *P.T
	*P.T  // conflicts with embedded field T and *T
}
```

一个嵌入字段的方法`f`会晋升为结构x的方法`f`。他是一个合法的选择器。 


给定一个结构体S和一个类型T，晋升的方法包括的方法集按下面的规则：

* 如果S包括前乳字段T，则S和*S的方法集包括T晋升的方法，*S包括*T的方法。
* 如果S包括字段*T。那么S和*S均包含T和*T的所有方法集。


声明字段时可以给该字段添加一个字符串的tag。这个tag将会成为他对应的字段的一个属性。一个空的tag和缺省的tag时一样的。tag的值可以被反射的接口访问到，并且参加类型结构体的类型定义，也可以被忽略。

```
struct {
	x, y float64 ""  // an empty tag string is like an absent tag
	name string  "any string is permitted as a tag"
	_    [4]byte "ceci n'est pas un champ de structure"
}

// A struct corresponding to a TimeStamp protocol buffer.
// The tag strings define the protocol buffer field numbers;
// they follow the convention outlined by the reflect package.
struct {
	microsec  uint64 `protobuf:"1"`
	serverIP6 uint64 `protobuf:"2"`
}
```

#### 指针类型

一个指针类型代表所有给定类型的指针变量。叫做类型的指针。没有被初始化的指针为nil。

```
PointerType = "*" BaseType .
BaseType    = Type .
```

```
*Point
*[4]int
```

函数类型可以代表所有具有相同参数和返回值的函数。没有被初始化的函数类型变量为nil。

```
FunctionType   = "func" Signature .
Signature      = Parameters [ Result ] .
Result         = Parameters | Type .
Parameters     = "(" [ ParameterList [ "," ] ] ")" .
ParameterList  = ParameterDecl { "," ParameterDecl } .
ParameterDecl  = [ IdentifierList ] [ "..." ] Type .
```

在参数列表或者返回值的列表中他们必须全部被命名或者全部为缺省。如果不是缺省，那么每个名字将会代表一个字段，这些名字不能重复。如果是缺省正太，每个类型代表一个该类型的元素。参数列表和返回值列表一般都是需要加括号的除非只有一个未命名的返回值他可以不使用括号。

函数的最后一个参数可以添加前缀...A。这个方式叫做可变参数。你可以传入0个或多个参数。

```
func()
func(x int) int
func(a, _ int, z float32) bool
func(a, b int, z float32) (bool)
func(prefix string, values ...int)
func(a, b int, z float64, opt ...interface{}) (success bool)
func(int, int, float64) (float64, *[]int)
func(n int) func(p *T)
```

#### 接口类型

一个接口指定一个方法集。一个接口变量可以储存任何方法集是该接口超集的类型。我们可以说该类型实现了这个接口。没有被初始化的接口类型值为nil。

```
InterfaceType      = "interface" "{" { MethodSpec ";" } "}" .
MethodSpec         = MethodName Signature | InterfaceTypeName .
MethodName         = identifier .
InterfaceTypeName  = TypeName .
```

接口中的方法集不能有空的或者重复的方法名。

```
// A simple File interface
interface {
	Read(b Buffer) bool
	Write(b Buffer) bool
	Close()
}
```

不止一个类型可以实现接口，例如：类型`S1`和类型`S2`都有以下方法集：

```
func (p T) Read(b Buffer) bool { return … }
func (p T) Write(b Buffer) bool { return … }
func (p T) Close() { … }
```

这里的类型T可以代表`S1`也可以代表`S2`.所以我们可以说`S1`和`S2`都实现了接口`File`。不用管他俩个类型是否有其他的方法。

一个类型实现了所有有他的方法集的子集构成的接口。所以也实现了很多不同的接口。例如所有的类型都实现了空接口：

```go
interface{}
```

相似的，下面这个接口使用type定义成一个名为`Locker`的接口：
```go
type Locker interface {
	Lock()
	Unlock()
}
```

如果`S1`和`S2`也实现了它：
```go
func (p T) Lock() { … }
func (p T) Unlock() { … }
```

于是他们就实现了两个接口`Locker`和`File`。

一个接口T可以使用另一个接口E来指定方法。这种方式叫做将接口E嵌入进接口T。他把E中所有的方法包括导出和未导出的方法全部添加进接口T。

```go
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
	File        // illegal: Lock, Unlock not unique
	Lock()      // illegal: Lock not unique
}
```

接口T不能嵌入自己或者那些已经嵌入他的接口。

```go
// illegal: Bad cannot embed itself
type Bad interface {
	Bad
}

// illegal: Bad1 cannot embed itself using Bad2
type Bad1 interface {
	Bad2
}
type Bad2 interface {
	Bad1
}
```

#### Map类型

一个map是一种一一种类型的非重复值作为键的一种类型的无序集合。

```
MapType     = "map" "[" KeyType "]" ElementType .
KeyType     = Type .
```

map的键类型必须能使用比较运算符`==`和`!=`进行比较。因此他的键类型不能是函数，map，或者切片。如果键的类型是一个接口，那么比较运算符必须能比较他的动态值。如果不能将会抛出一个运行时错误。

```go
map[string]int
map[*T]struct{ x, y float64 }
map[string]interface{}
```

map中元素的个数叫做他的长度。对于一个map`m`。他的长度可以通过内置函数`len`获得，并且他的长度可以在运行时被改变。元素在运行时可以被添加和取回，并且可以通过内置的`delete`方法移除元素。

想要初始化一个新的空的map可以使用内置函数`make`。他能指定map的类型和预留的空间。

初始化种的预留空间并不是他的长度，map可以通过添加一定数量的原属来增大自己的尺寸。

#### Channel类型

channel提供一种并发的手段去发送和接收指定类型的值。没有被初始化的channel是nil。

```
ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .
```

操作符`<-`指定channel的方向是发送还是接收。如果没有指定方向，channel就是双向的。一个频道的操作符可以限制他的方向。

```go
chan T          // can be used to send and receive values of type T
chan<- float64  // can only be used to send float64s
<-chan int      // can only be used to receive ints
```

可以通过内置的`make`函数来初始化channel。可以指定channel的类型和容量。

```go
make(chan int, 100)
```

容量是设置了最大能缓存的数量。如果没有设置容量，channel就是没有还蠢的。并且只有当发送者和接收者都准备好了才能开始传送。不过带缓存的buffer在缓存没有满的时候可以发送成功数据，当缓存不为空的时候可以接收到数据，一个nil的channel永远都不会准备好。

channel可以通过内置函数`close`进行关闭操作。在接收多将会有2个值来提示接收者是否有数据被发送。

一个单通道可以用来发送和接收数据。也可以用内置函数`cap`来获取他的容量。
使用内置函数`len`来检查还有多少元素没有被同步。channel模拟了一个fifo队列。举个例子，一个goruntine发送数据，第二个goruntine接收他们，他们被接收的顺序和被发送的顺序是相同的。





