## golang基础知识汇总

### 1 约定 

- package写法

```sh
package packname // 包名称
// 每个go最先执行的是init，而不是main。init可以没有，但main必须有。
func init() () {

}
```

- 导入外部库

```sh
import ("fmt";"net")
import(
    "fmt"          // fmt是标准库，去 "$GOROOT" 下查找
    "net"
    "app/selfwritelib"  // 非标准库，去 "$GOPATH/src/app/selfwritelib" 查找
)
```

- 在go中，首写字母为大写的函数或变量，是要被导出的。如

```sh
func WriteIObuffer() error {
    // ... ...
}
var IoNumber int
```

- go需要显示的类型转换，不支持隐式转换。

```sh
var i int = 5
var f float64 = float64(i)

// interface 的强制转换
type T struct{
    K string
    V string
}

func f(i interface{}) {
    i.(T).K //转换
    i.(T).V
}
```

- go有指针，但没有指针运算

```sh
type Vertex struct {
    x int
    y int
}

func f() {
    v := Vertex{1, 2}
    p = &Vertex{} //默认初始化为{0,0}，p为 *Vertex指针
    p = &v
    p++ //error，不支持，不能通过++,--改变指针的指向
}
```

- defer，defer是一个栈，先进后出。

```sh
//defer语句注册一个函数，当defer所在的函数返回时，调用defer注册的函数。
func f() (result int) {
    defer func() {
        result++
    }()

    return 1
}

func main() {
    fmt.Println(f()) // 打印的值为2，因为先返回1，后调用defer.
}
```

### 2 变量

函数外的每个语句，必须以关键字`var/func/type`等开始。

简洁赋值语句`:=`只能出现在函数内部，不能出现在函数外。

### 3 数组array

长度是数组的一部分，[3]int和[4]int是不同的数组。
可以使用下标对数组进行操作。

```sh
a := [3]int{1, 2, 3}  // 一维数组
b := [2][4]int{[4]int{1,2,3,4}, [4]int{5,6,7,8}} // 二维数组
c := [2][4]int{{1,2,3,4},{5,6,7,8}} // 二维数组

package main
import "fmt"
import "reflect"

func main() {
    m := [...]interface{} {  // m为数组，若声明为 m := []interface{}，则m为切片
        [3]int{1, 2, 3}, // 数组，长度为3
        [...]int{1, 2, 3}, // 数组，长度由编译器自己算
        []int{1, 2, 3}, //　切片
        string("aaaaa"),
        string("cccccccc"),
    }

    for _, v := range m {
        fmt.Printf("%v \n", v)
    }

    for i, v := range m {
        rv := reflect.ValueOf(v)
        fmt.Println(i, rv.Kind())
    }
}

// 结果
[1 2 3]
[1 2 3]
[1 2 3]
aaaaa
cccccccc
0 array
1 array
2 slice
3 string
4 string
```

### 4 切片slice

slice有cap和len两个属性

#### 4.1 创建

- 通过声明创建
`p := []int{2,3,4,5,6} // []T是一个元素类型为T的slice`  
- 通过make创建
`a := make([]int, 0, 5) //int类型的切片，len(a)=0, cap(a)=5`  
- 通过赋值创建
```sh
s := [4]int{0, 1, 2, 3}   //　s为数组
t := s[1:3] // 1, 2　t为slice
m := s[1:]  // 1, 2, 3
n := s[:3]  // 0, 1, 2
```

#### 4.2 加入元素&遍历
使用**append**向切片中加入元素，若空间不够，会自动开辟新的空间，返回值指向新的切片。
append的原型为：`append(s[ ]T, vs ...T) [ ]T`，
使用`for range`进行遍历。

```sh
func f() {
    a := make([]int, 1, 2) // len = 1, cap = 2
    a = append(a, 5)　　// 若使用 a[2] = 5，程序会崩溃
    a = append(a, 6)
    a = append(a, 7)
    //使用range进行遍历
    for i, v := range a { 
        fmt.Printf("i[%d] v[%d] ", i, v)
    }
}
```

#### 4.3 拷贝和赋值

使用**copy**命令进行拷贝
```sh
make([]string, 0, 10) // string类型的切片，长度0，容量10.
copy(dst, src)
var a = [...]int{0,1,2,3,4,5,6,7} // int类型的数组
var s = make([]int, 6)  // int类型的slice
var b = make([]byte, 5) // byte类型的slice

var b[]byte // byte类型切片
b = append(b, "hi"...) //append可能会重新分配底层数组来容纳新的单元

//添加b
a = append(a, b)
//复制
b = make([]T, len(a))
copy(b,a)
//删除 [i:j]
a = append(a[:i], a[j:]...)
//删除第i个元素
a = append(a[:i], a[i+1:]...)
//扩展j个空元素
a = append(a, make([]T, j)...)
//插入j个空元素
a = append(a[:i], append(make([]T, j), a[i:]...)...)
//插入元素x
a = append(a[:i], append([]T{x}, a[i:]...)...)
//插入切片b
a = append(a[:i], append(b, a[i:]...)...)
//弹出最后一个元素
x, a = a[len(a)-1], a[:len(a)-1]
//压入x
a = append(a, x)
```

#### 4.4 切片和数组的不同

数组在编译的时候，就确定了长度，且长度不能改变。

slice实际上是对源数组的引用，内部也是用数组实现的，是个长度可变的数组。

```sh
a := [...]int{1, 2, 3, 4}  // 数组，长度由编译器来计算
sa := a[0:]    //  切片
sa[2] = 10    //  数组a的值，也会跟着变
sa = append(sa, 100) // 数组a的值，也会跟着变

sb := a[0:]
sb = append(sb, 101, 102, 103, 104)
// slice在引用的时候，可能会发生意外。即当slice的长度增大时（append），
// 可能会重新分配内存，这时sb的地址和a的地址不同。

m := []interface{} { // 声明一个slice
	[3]int{1, 2, 3}  // int型数组 
	[3]string{"aa", "bb", "cc"} // string类型数组
	string("efef") // string
}
```

数组在作为参数时，是值传递（一份拷贝，而不是引用传递）。slice作为参数时，是引用传递。


### 5 map

- map使用前，必须使用make来创建
- 可以使用下标法进行赋值
- 可以直接判断元素是否存在
- 使用**for range**迭代时，可以安全的删除和插入map

```sh
type Ver struct {
    x, y int
}

var m1 map[string]Ver //声明一个map，key为string类型，value为Ver类型
func f() {
    m1 = make(map[string]Ver) //map使用前，必须使用make而不是new来创建
    m1["dzh"] = Ver{103, 601516} //赋值方法1
    var m2 = map[string]Ver { //赋值方法2
        "Bell": Ver{3,5},
        "Dzh": Ver{6,4},
    }

    m3 := map[string]Ver { //赋值方法3
        "Bell": Ver{3,5},
        "Dzh": Ver{6,4},
    }

    m3["ccav"] = Ver{3, 5} //添加元素
    delete(m3, "Dzh") //删除元素
    if v, ok := m3["Dzh"]; ok { //判断元素是否存在
        fmt.Println(v)
    }
}

m := map[int] string {1:"One", 2:"Two" }
for k, v := range m {
    v // **v只是一个拷贝，不能通过v来改变m中的值**
    m[3] = "Three"
    delete(m, 1)
}
```


### 6 函数 & 闭包

#### 6.1 基本函数和闭包

```sh
// 函数
func main() {
    p1 := func(x, y int) float64 { //p1为函数
        return math.Sqrt(x*x+y*y)
    }

    fmt.Println(p1(3,4)) //调用p1函数
}
//-------------------------------------------------------------------
// 闭包
// func(a, b int) float64是一个函数的声明
func adder(m float64) func(a, b int) float64 {
    sum := 0.0
    return func(a, b int) float64 {
        sum += (m * 10) + math.Sqrt(float64(a*a+b*b))
        return sum
    }
}

func main() {
    pos20 := adder(20) // 返回函数
    fmt.Println(pos20(3, 4))

    pos30 := adder(30)  // 返回函数
    fmt.Println(pos30(6,8))
}
//---------------------------------------------------------------------

// 用闭包演示 fibonacci
package main
import "fmt"
// fibonacci 函数会返回一个返回 int 的函数。
func fibonacci() func() int {
    a := 0
    b := 0
    return func() int {
        if a == 0 && b == 0 {
            a = 1
            b = 0
            return 0
        }

        c := a+b
        a = b
        b = c
        return c
    }
}

func main() {
    f := fibonacci()
    for i := 0; i < 10; i++ {
        fmt.Printf("%d ", f()) // 0 1 1 2 3 5 8 13 21 34
    }

    fmt.Printf("%d ", f()) // 55
}
```

#### 6.2 不定参数

**arg ...int**，表示函数的参数个数不定;且 ...int这种参数只能作为函数的最后一个

```sh
func myfun(s string, arg ...int) {
    for i, n := range arg {
        fmt.Printf("第[%d]个参数[%d]", i, arg[n])
    }
}
```

#### 6.3 参数传递

函数参数传递，都是以拷贝的形式进行的。若想在函数中改变值，需要使用指针。

**Go语言中string，slice，map这三种类型的实现机制类似指针，所以可以直接传递，而不用取地址后传递指针**。（注：若函数需改变slice的长度，则仍需要取地址传递指针）

```sh
func add_1(a *int) int {
    *a = *a +1
    return *a
}
```

### 7 方法

golang没有类，但可以在struct的基础上定义方法。

```sh
type Ver struct {
    x, y int
}

// 用拷贝,不改变值
func (v Ver) Abs() float64 {
    return math.Sqrt(float64(v.x*v.x + v.y*v.y))
}
// 用指针,改变值
func (v *Ver) Setx(a int) {
    v.x = a // 改变x的值
}

func main() {
    v := &Ver{6,8} //指针
    fmt.Println(v.Abs()) // 10
}
```

### 8 interface

```sh
type ITEM struct {
    k int64
    v string
}

// interface类型转换
func achange(v interface{}) {
    k := v.(ITEM).k
    v := v.(ITEM).v
}
```

### 9 goroutine
- 非抢占式的，需要自己让出yield
- go程调度发生在操作系统调用时。当发生系统调用时，会使正在执行的go程让出CPU给其他go程
- go程相互通信，靠的是channel。channel类型的运算有三种：**send/receive/select**

```sh
var chin chan int //声明一个chin，用于收发int型
chin<-0 //把0发送给chin
<-chin //从chin中读取
chin = make(chan int) //必须make后才能进行操作
chin = make(chan int, 1024) //不标明个数(1024)，默认为0
```

- select语句的每个case分支，只能是发送和接收语句

### 10 channel

- 收&发

```sh
// 将 v 送入 channel ch
ch <- v
// 从 ch 接收，并且赋值给 v
v := <-ch

//只能从里面读
func sendchnn(ch <- chan interface{}){
    t := <- ch
}
// 只能往里面写
func writechnn(ch chan <- interface{}) {
    c := string("write")
    ch <- c
}
//可读写
func readwritechnn(ch chan interface{}) {
    c := string("write")
    ch <- c //写
    r := <- ch //读
}
```

- 和map与slice一样，channel使用前必须创建
`ch := make(chan int)`

- 接收者可以通过赋值语句的第二个参数来测试channel是否被关闭

```sh
if v, ok := <- ch; ok == false {  //判断ok的返回值
    fmt.Println("被关闭了")
}
```

- 只有发送者才可以关闭channel，接收者不能关闭channel。向一个已经关闭的channel发送数据时，会panic。
- 当channle关闭后，下面的for循环会被退出。

```sh
for i := range ch { //循环会不停的从ch中读取数据，直到ch被关闭

}
```

- 当调用close(ch)时，所有处于<-ch的routine都能够收到消息，可以利用该属性让所有的routine退出。



### 11 select

- select会被阻塞，直到case分支中有条件满足。
```sh
func f() {
    for {
        select { //select会被阻塞，直到case中的条件满足
            case x <- ch1: //从ch1通道中读取数据
                fmt.Println(x)
            case y <- ch2: //从ch2通道中读取数据
                fmt.Println(y)
        }
    }
}
```

- 当select中的其他条件分支都没有准备好的时候，default分支会被执行。
```sh
func f() {
    for {
        select {
            case x <- ch1:
                fmt.Println(x)
            case y <- ch2:
                fmt.Println(y)
            default:
                time.Sleep(time.Millisecond) //当其他分支没有被触发时，总被满足
        }
    }
}
```