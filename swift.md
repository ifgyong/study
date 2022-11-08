# swift 
[2021swift 和oc的不同](https://blog.csdn.net/iOSvv/article/details/122235204)

[swift 中结构体和class的区别](https://www.jianshu.com/p/8015b163baf6)

## 1. class和struct区别
> `class`是引用类型，`struct`是值类型，
> `class`可以继承，`struct`不可继承，可实现协议。
> 一般需要更改之前的值，可以使用`class`，只是展示则使用`struct`
 
## 2. defer什么时间使用
一般用于清理资源和回收资源，解锁。

至少要执行到`defer`这一行，否则不会执行该代码段。多个`deffer`在一个代码段，倒叙执行。

## 3.throws 和 rethrows
- `throws` 抛出异常
- `rethrows` 传递异常，可能闭包有异常，使用`rethrows`来向上抛出异常。

## 4. 简述`associatetype`

`associatedtype`其实是一般用于协议类型的声明，当使用的时候可以指定该类型。

```objc
protocol Container{
    associatedtype ItemType
    mutating func append(_ item:ItemType)
    var count:Int{get}
    subscript(i:Int)->ItemType{get}


// 使用 实现协议
import UIKit
protocol Container{
    associatedtype ItemType
    mutating func append(_ item:ItemType)
    var count:Int{get}
    subscript(i:Int)->ItemType{get}
}
 
struct InStack: Container{
    typealias ItemType = Int
    var count: Int
    var items=[Int]()
    mutating func append(_ item: Int) {
        items.append(item)
    }
    subscript(i: Int) -> Int {
        return items[i]
    }
    //自定义方法
    mutating func pop()->Int{
        return items.removeLast()
}
```

同时也可以添加约束
```objc
//协议可作为它自身的要求出现
protocol SuffixableContainer:Container{
    associatedtype Item:Equatable
    associatedtype Suffix: SuffixableContainer where Suffix.Item == ItemType
    func suffix(_ size:Int)->Suffix
}
```

## 5. dynamic作用
`dynamic`与`@objc`修饰`class`走runtime，可以使用`kvo`.
```objc
class Person :NSObject{
	@objc dynamic var name:String
	var age:Int
	override init() {
		age = 0
		name = ""
	}
}

let p = Person()
override func viewDidLoad() {
	super.viewDidLoad()
	//会kvo 通知更改
	p.addObserver(self, forKeyPath: "name", options: [.new], context: nil)
	// 不会通知
	p.addObserver(self, forKeyPath: "age", options: .new, context: nil)
	p.name = "123"
	p.age = 12
}
```
## 6. lazy作用
懒加载，在使用的时候再去执行与赋值。
>首先是一个Optional，在没有被访问之前默认值是nil，在内存的表现是0x0，在第一次访问的过程中，访问的是getter方法，通过enum值得分支进行一个赋值的操作
>

## 7.swift 闭包详解

## 8. 简述 autoclosure的作用
延迟执行。
- `@escaping`生命周期大于函数，一般作为参数传给函数，却没有立即执行。容易造成循环引用。
- `nonescaping`默认，在函数生命周期内执行。


## 9. 高阶函数
#### 1.`map`
- `map`一般用于遍历将集合或数组的元素转化成另外一个格式

```objc
let prices = [3,4,2,1]
		let strs = prices.map { "\($0)_"}
		// ["3 _", "4 _", "2 _", "1 _"]
		print(strs)
		
		
let values = [4, 6, 9]
let squares = values.map({ $0 * $0 })
print(squares)// [16 36 81]
```
- `flatMap`和`map`类似，多了一个将`opention`解包和过滤`nil`
- 会将二维数组打开变成一个新的一位数组

```objc
let array = ["Apple", "Orange", "Grape", ""]
 
let arr1 = array.map { a -> Int? in
    let length = a.count
    guard length > 0 else { return nil }
    return length
}
print("arr1:\(arr1)")
 
let arr2 = array.flatMap { a-> Int? in
    let length = a.count
    guard length > 0 else { return nil }
    return length
}
print("arr2:\(arr2)")//[]

let array = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
let arr1 = array.map{ $0 }   // [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
let arr2 = array.flatMap{ $0 } // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```


### filter
- `filter`过滤元素

```objc
let prices = [20,30,40]
let result = prices.filter({ $0 > 25 })
print(result)
```
### reduce
- `reduce` 将数组元素合并计算为一个值，并且会接受一个初值，这个初值的类型可以和数组元素类型不同。

```objc
let prices = [20,30,40]
let sum = prices.reduce(0) { $0 + $1 }
print(sum)// 90
```

链式调用

```objc
		let prices = [3,4,2,1]
		let count = prices.map({ $0+1})
		.flatMap({ $0+1})
		.reduce(0) { $0+$1}
		
		print(count)// 18
```





