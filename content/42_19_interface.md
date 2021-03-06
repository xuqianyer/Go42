# <center>《Go语言四十二章经》第十九章 接口</center>

作者：李骁

## 19.1 接口是什么

Go 语言里有非常灵活的 接口 概念，通过它可以实现很多面向对象的特性。
接口定义了一组方法（方法集），但是这些方法不包含（实现）代码：它们没有被实现（它们是抽象的）。接口里也不能包含变量。

通过如下格式定义接口：

```Go
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
```
上面的 Namer 是一个 接口类型

（按照约定，只包含一个方法的）接口的名字由方法名加 [e]r 后缀组成，例如 Printer、Reader、Writer、Logger、Converter 等等。还有一些不常用的方式（当后缀 er 不合适时），比如 Recoverable，此时接口名以 able 结尾，或者以 I 开头。
Go 语言中的接口都很简短，通常它们会包含 0 个、最多 3 个方法。

>注意：
>类型不需要显式地声明它实现了某个接口，接口被隐式地实现，隐式接口解藕了实现接口的包和定义接口的包：互不依赖。

多个类型可以实现同一个接口，一个类型可以实现多个接口，实现了某个接口的类型，还可以有其它的方法。

接口类型是由一组方法定义的集合。接口类型的值可以存放实现这些方法的任何值。

类型（比如结构体）实现接口方法集中的所有方法，一定是接口方法集中所有方法。那么接口类型的值其实也可以存放该结构体的值。

如：

```Go
package main

import (
	"fmt"
)

type A struct {
	Face int
}

type B interface {
	f()
}

func (a A) f() {
	fmt.Println("hi ", a.Face)
}

func main() {
	var s A = A{Face: 9}
	s.f()

	var b B = A{Face: 9}  //接口类型可接受结构体的值，因为结构体实现了接口
	b.f()
}
```
即使接口在类型之后才定义，二者处于不同的包中，被单独编译：只要类型实现了接口中的方法，它就实现了此接口。

所有这些特性使得接口具有很大的灵活性。

接口变量里包含了接收器实例的值和指向对应方法表的指针，也就是说接口实例上可以调用该实例方法，它使此方法更具有一般性。

>注意：接口中的方法必须要全部实现，才能实现接口。

## 19.2 接口嵌套

一个接口可以包含一个或多个其他的接口，这相当于直接将这些内嵌接口的方法列举在外层接口中一样。但是在接口内不能内嵌结构体，编译会出错。

比如接口 File 包含了 ReadWrite 和 Lock 的所有方法，它还额外有一个 Close() 方法。和结构体内嵌基本差不多。

```Go
type ReadWrite interface {
    Read(b Buffer) bool
    Write(b Buffer) bool
}

type Lock interface {
    Lock()
    Unlock()
}

type File interface {
    ReadWrite
    Lock
    Close()
}
```
 
## 19.3 类型断言

如何检测和转换接口变量的类型呢？这是类型断言（Type Assertion）就用上了。
一个接口类型的变量 varI 中可以包含任何类型的值，必须有一种方式来检测它的 动态 类型，即运行时在变量中存储的值的实际类型。在执行过程中动态类型可能会有所不同，但是它总是可以分配给接口变量本身的类型。通常我们可以使用 类型断言 来测试在某个时刻接口变量 varI 是否包含类型 T 的值：

```Go
v := varI.(T)       // unchecked type assertion
```
**varI 必须是一个接口变量**，否则编译器会报错：invalid type assertion: varI.(T) (non-interface type (type of varI) on left) 。

Go 语言中的所有程序都实现了interface{}的接口，这意味着，所有的类型如string, int, int64甚至是自定义的struct类型都就此拥有了interface{}的接口，这种做法和java中的Object类型比较类似。那么在一个数据通过func funcName(interface{})的方式传进来的时候，也就意味着这个参数被自动的转为interface{}的类型。

```Go
func funcName(a interface{}) string {
     return string(a)
}
```
类型断言可能是无效的，虽然编译器会尽力检查转换是否有效，但是它不可能预见所有的可能性。如果转换在程序运行时失败会导致错误发生。更安全的方式是使用以下形式来进行类型断言：

```Go
if v, ok := varI.(T); ok {  // 类型断言检查
    Process(v)
    return
}
```
如果转换合法，v 是 varI 转换到类型 T 的值，ok 会是 true；否则 v 是类型 T 的零值，ok 是 false，也没有运行时错误发生。

* **类型判断：type-switch**

接口变量的类型也可以使用一种特殊形式的 switch 来检测：type-switch

```Go
switch t := areaIntf.(type) {
case *Square:
    fmt.Printf("Type Square %T with value %v\n", t, t)
case *Circle:
    fmt.Printf("Type Circle %T with value %v\n", t, t)
case nil:
    fmt.Printf("nil value: nothing to check?\n")
default:
    fmt.Printf("Unexpected type %T\n", t)
}
```
可以用 type-switch 进行运行时类型分析，但是在 type-switch 不允许有 fallthrough 。

在处理来自于外部的、类型未知的数据时，比如解析诸如 Json 或 XML 编码的数据，类型测试和转换会非常有用。

* **测试一个值是否实现了某个接口**

我们想测试它是否实现了 Stringer 接口，可以这样做：

```Go
type Stringer interface {
    String() string
}

if sv, ok := v.(Stringer); ok {
    fmt.Printf("v implements String(): %s\n", sv.String()) 
}
```
Print 函数就是如此检测类型是否可以打印自身的。

接口是一种契约，实现类型必须满足它，它描述了类型的行为，规定类型可以做什么。接口彻底将类型能做什么，以及如何做分离开来，使得相同接口的变量在不同的时刻表现出不同的行为，这就是多态的本质。

**编写参数是接口变量的函数，这使得它们更具有一般性。**

**使用接口使代码更具有普适性。**

标准库里到处都使用了这个原则，如果对接口概念没有良好的把握，是不可能理解它是如何构建的。

## 19.4 接口与动态类型

**Go 的动态类型**

在经典的面向对象语言（像 C++，Java 和 C#）中数据和方法被封装为 类 的概念：类包含它们两者，并且不能剥离。

Go 没有类：数据（结构体或更一般的类型）和方法是一种松耦合的正交关系。

Go 中的接口跟 Java/C# 类似：都是必须提供一个指定方法集的实现。但是更加灵活通用：任何提供了接口方法实现代码的类型都隐式地实现了该接口，而不用显式地声明。

和其它语言相比，Go 是唯一结合了接口值，静态类型检查（是否该类型实现了某个接口），运行时动态转换的语言，并且不需要显式地声明类型是否满足某个接口。该特性允许我们在不改变已有的代码的情况下定义和使用新接口。

接收一个（或多个）接口类型作为参数的函数，其实参可以是任何实现了该接口的类型。 实现了某个接口的类型可以被传给任何以此接口为参数的函数 。

类似于 Python 和 Ruby 这类动态语言中的 动态类型（duck typing）；这意味着对象可以根据提供的方法被处理（例如，作为参数传递给函数），而忽略它们的实际类型：它们能做什么比它们是什么更重要。

**动态方法调用**

像 Python，Ruby 这类语言，动态类型是延迟绑定的（在运行时进行）：方法只是用参数和变量简单地调用，然后在运行时才解析（它们很可能有像 responds_to 这样的方法来检查对象是否可以响应某个方法，但是这也意味着更大的编码量和更多的测试工作）

Go 的实现与此相反，通常需要编译器静态检查的支持：当变量被赋值给一个接口类型的变量时，编译器会检查其是否实现了该接口的所有函数。如果方法调用作用于像 interface{} 这样的“泛型”上，你可以通过类型断言来检查变量是否实现了相应接口。

因此 Go 提供了动态语言的优点，却没有其他动态语言在运行时可能发生错误的缺点。

对于动态语言非常重要的单元测试来说，这样即可以减少单元测试的部分需求，又可以发挥相当大的作用。

Go 的接口提高了代码的分离度，改善了代码的复用性，使得代码开发过程中的设计模式更容易实现。用 Go 接口还能实现 依赖注入模式。

## 19.5 接口的提取

提取接口，是非常有用的设计模式，可以减少需要的类型和方法数量，而且不需要像传统的基于类的面向对象语言那样维护整个的类层次结构。

Go 接口可以让开发者找出自己写的程序中的类型。假设有一些拥有共同行为的对象，并且开发者想要抽象出这些行为，这时就可以创建一个接口来使用

所以你不用提前设计出所有的接口；整个设计可以持续演进，而不用废弃之前的决定。类型要实现某个接口，它本身不用改变，你只需要在这个类型上实现新的方法。

显式地指明类型实现了某个接口

如果你希望满足某个接口的类型显式地声明它们实现了这个接口，你可以向接口的方法集中添加一个具有描述性名字的方法。

大部分代码并不使用这样的约束，因为它限制了接口的实用性。

但是有些时候，这样的约束在大量相似的接口中被用来解决歧义。

## 19.6 接口的继承

当一个类型包含（内嵌）另一个类型（实现了一个或多个接口）的指针时，这个类型就可以使用（另一个类型）所有的接口方法。

类型可以通过继承多个接口来提供像 多重继承 一样的特性：

```Go
type ReaderWriter struct {
    *io.Reader
    *io.Writer
}
```
上面概述的原理被应用于整个 Go 包，多态用得越多，代码就相对越少。这被认为是 Go 编程中的重要的最佳实践。
