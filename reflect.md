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


### 未知原有类型【遍历探测其Filed】

+ 对于不知道其具体类型的情况，需要遍历探测其Field来得知，如下：
```go
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
		field := getType.Field(i)
		value := getVal.Field(i).Interface()
		fmt.Printf("%s: %v = %v\n", field.Name, field.Type, value) //
	}

	// 获取方法
	// 1. 先获取interface的reflect.Type,然后通过.NumMethod进行遍历
	for i := 0; i < getType.NumMethod(); i++ {
		m := getType.Method(i)
		fmt.Printf("%s: %v\n", m.Name, m.Type)
	}
}
```
