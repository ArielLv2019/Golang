## 编程语言中反射的概念
+ 在计算机科学领域，反射是指一类应用，它们能够自描述和自控制。也就是说，这类应用通过采用某种机制来实现对自己行为的描述（self-representation）和监测（examination），并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义。
+ 每种语言的反射模型都不同，并且有些语言根本不支持反射。Golang语言实现了反射，反射机制就是在运行时动态的调用对象的方法和属性，官方自带的reflect包就是反射相关的，只要包含这个包就可以使用。

+ 多插一句，Golang的gRPC也是通过反射实现的。

## interface 和 反射
+ 在讲反射之前，先来看看Golang关于类型设计的一些原则：
    - 变量包括（type, value）两部分
        - 理解这一点就知道为什么nil != nil了
    - type 包括 static type和concrete type. 简单来说 static type是你在编码是看见的类型(如int、string)，concrete type是runtime系统看见的类型
    - 类型断言能否成功，取决于变量的concrete type，而不是static type. 因此，一个 reader变量如果它的concrete type也实现了write方法的话，它也可以被类型断言为writer.

+ 反射，就是建立在类型之上的，Golang的指定类型的变量的类型是静态的（也就是指定int、string这些的变量，它的type是static type），在创建变量的时候就已经确定，反射主要与Golang的interface类型相关（它的type是concrete type），只有interface类型才有反射一说。
在Golang的实现中，每个interface变量都有一个对应pair，pair中记录了实际变量的值和类型:
```go
(value, type)
```
+ value是实际变量值，type是实际变量的类型。一个interface{}类型的变量包含了2个指针，一个指针指向值的类型（对应concrete type），另一个指针指向实际的值（对应value）。

+ 例如，创建类型为*os.File的变量，然后将其赋给一个接口变量r:
```go
    tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)

	var r io.Reader
	r = tty // r的pair中将记录: (tty, *os.File),这个pair在接口变量的连续赋值过程中是不变的

	// 将接口变量r赋给另一个接口变量w:
	var w io.Writer
	w = r.(io.Writer) //接口变量w的pair与r的pair相同，都是:(tty, *os.File)，即使w是空接口类型，pair也是不变的   
```
+ interface及其pair的存在，是Golang中实现反射的前提，理解了pair，就更容易理解反射。反射就是用来检查存储在接口变量内部（值value；类型concrete type）pair对的一种机制。

## Golang的反射
### reflect的基本功能TypeOf和ValueOf
+ Golang提供了两种类型（或者说两个方法）让我们可以很容易的访问接口变量内容，分别是TypeOf()和ValueOf(),官方解释：
```go
// ValueOf returns a new Value initialized to the concrete value
// stored in the interface i.  ValueOf(nil) returns the zero 
func ValueOf(i interface{}) Value {...}

// 翻译一下：ValueOf用来获取输入参数接口中的数据的值，如果接口为空则返回0

// TypeOf returns the reflection Type that represents the dynamic type of i.
// If i is a nil interface value, TypeOf returns nil.
func TypeOf(i interface{}) Type {...}

// 翻译一下：TypeOf用来动态获取输入参数接口中的值的类型，如果接口为空则返回nil
```
+ reflect.TypeOf(): 获取pair中的type,直接给出想要的type类型，如float64，int，各种pointer，struct等真实的类型
+ reflect.ValueOf(): 获取pair中的value，直接给出想要的value值，如1.2345，或者类似&{1 "Allen.Wu" 25} 这样的结构体struct的值
+ 反射可以将“接口类型变量”转换为“反射类型对象”，返回类型=reflect.Type + reflect.Value
```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var num float64 = 1.2345

	fmt.Println("type: ", reflect.TypeOf(num)) // type: float64
	fmt.Println("value: ", reflect.ValueOf(num)) //value: 1.2345
}
```
### 从reflect.Value中获取接口interface的信息
+ 当执行reflect.ValueOf(interface)后，得到一个类型为"reflect.Value"的变量，可以通过它本身的Interface()方法获得接口变量的真实内容，然后通过类型断言转换为原有的真实类型。
+ 有两种情况： 1）已知原有类型； 2）未知原有类型
#### 1. 已知原有类型【进行“强制转换”】
+ 直接通过Interface方法然后强制转换：
```go
realValue := value.Interface().(已知的类型)
```
+ 示例：
```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var num float64 = 1.2345

	pointer := reflect.ValueOf(&num)
	value := reflect.ValueOf(num)

	// 可以理解为“强制转换”，但是要注意：如果转换的类型不完全符合就直接panic
	// Golang对类型要求非常严格，类型一定要完全符合
	// 如下两个，一个是*float64，一个是float64，如果弄混，则会panic
	convertPointer := pointer.Interface().(*float64)
	convertValue := value.Interface().(float64)

	fmt.Println(convertPointer)
	fmt.Println(convertValue)
}
```
+ Note:
    - 转换的时候，如果转换的类型不完全符合，则直接panic，类型要求非常严格！
    - 转换的时候，要区分是指针还是值
    - 也就是说反射可以将“反射类型对象”再重新转换为“接口类型变量”

#### 2.未知原有类型【遍历探测其Filed】

+ 对于不知道其具体类型的情况，需要遍历探测其Field来得知，如下：
```go
package main

import (
	"fmt"
	"reflect"
)

type User struct {
	Id   int
	Name string
	Age  int
}

func (u User) ReflectCallFunc() {
	fmt.Println("allen.wu reflectCallFunc")
}

func main() {
	user := User{1, "allen.wu", 25}
	DoFiledAndMethod(user)
}

//通过接口获取任意参数，然后一一揭晓
func DoFiledAndMethod(input interface{}) {
	getType := reflect.TypeOf(input)
	fmt.Println("get type: ", getType.Name()) //get type:  User

	getVal := reflect.ValueOf(input)
	fmt.Println("get val: ", getVal) //get val:  {1 allen.wu 25}

	// 获取方法字段
	// 1. 先获取Interface的reflect.Type,然后通过NumField进行遍历
	// 2. 通过reflect.Type的Field()获取其对应的Field
	// 3. 最后通过Field的Interface()得到对应的value
	for i := 0; i < getType.NumField(); i++ {
		field := getType.Field(i) //2.
		value := getVal.Field(i).Interface() //3.
        fmt.Printf("%s: %v = %v\n", field.Name, field.Type, value) 
        // Id: int = 1
        // Name: string = allen.wu
        // Age: int = 25
	}

	// 获取方法
	// 1. 先获取interface的reflect.Type,然后通过.NumMethod进行遍历
	for i := 0; i < getType.NumMethod(); i++ {
		m := getType.Method(i)
		fmt.Printf("%s: %v\n", m.Name, m.Type)  // ReflectCallFunc: func(main.User)
	}
}
```
+ Note: 通过运行结果可以得知获取未知类型的interface的具体变量及其类型的步骤：
    - 1. 先获取interface的reflect.Type,然后通过NumField()进行遍历
    - 2. 再通过reflect.Type的Field(index)获得其Field
    - 3. 最后通过Field的Interface()得到对应的value值

+ 通过运行结果可以得知获取未知类型的interface的所属方法（函数）的步骤：
    - 1. 先获取interface的reflect.Type,然后通过NumMethod()进行遍历
    - 2. 再通过reflect.Method(index)获得其真实的方法（函数）
    - 3. 最后对结果取其Name和Type得知具体的方法名
    - 4. 也就是说：反射可以将“反射类型对象”再重新转换为“接口类型变量”
    - 5. struct或者struct的嵌套都是一样的判断处理方式
