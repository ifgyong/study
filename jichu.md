# 基础常见题目

### 1. `assgin` `copy` `retain` `strong` 区别

 1. `assgin` 对基础数据类型 （`NSInteger`）和`C`数据类型（`int`,` float`,` double`, `char`,等)
 2. `copy`用于`String`，`block`
 3. `retain` 用于`NSObject`和其他子类。
 4. `readonly`只生成`getter`方法
 5. `readwrite`生成`setter和getter`方法。

 > `assign：`默认类型,`setter`方法直接赋值，而不进行`retain`操作
`retain：setter`方法对参数进行`release`旧值，再`retain`新值。

`copy：setter`方法进行`Copy`操作，与`retain`一样
 
 
### 2. `#include`与`#import`的区别、`#import`与`@class` 的区别

 `#include`和`#import`都可以引入头文件，但是`#import`只会引入一次。`#import`虽然只会引入一次，但是还会导致相互引入的问题。`@class`可以在头文件引入类，类可以是不存在的，可以在头文件`@class`引入类，在`.m`用`#import`引入类实现从而解决循环引用的问题。
 
### 3. 用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题

我们知道`copy`关键词是复制一份内存地址，用新的指针指向。是属于深拷贝，而用`strong`关键字是通过一个指针指向对象的内存地址，通过内存地址访问对象，是属于浅拷贝。对于`strong`声明的字符串 数组字典其他地方可以通过地址直接修改对象

### 4. 这个写法会出什么问题： @property (copy) NSMutableArray *array

通过`copy`修饰的`NSMutableArry`对象变成了`NSArray`，这样在编译类型是`NSMutableArray`，在运行时是`NSArray`，这样我们在编译时候调用`NSMutableArray`方法是不报错的，但是在运行时候类型不一样从而导致崩溃发生。

### 5. 如何为 `Class `定义一个对外只读对内可读写的属性

我们可以在`.h`文件声明属性用`readonly`关键词，在`.m`声明同样的属性用`readwrite`默认的关键词。


```objc
/// .h
@interface ClassA : NSObject
@property (nonatomic, assign, readonly) int number;
@end

/// .m
@interface ClassA ()
@property (nonatomic, assign) int number;
@end

@implementation ClassA
@end
```

### 6. block和代理的区别，哪个更好？

`block`使用起来更加简单，比如访问作用域的变量还有代码逻辑的连贯性。代理更能表现是面向接口，能更好的描述功能和作用。如果是对外面向的接口推荐使用代理，日常开发页面之间的传值可以使用`block`.并且`block`和代理之间，`block`的执行会比较快一些。

### 7. 在block内如何修改block外部变量？

通过`block`内部访问外部的变量，`block`会自动`copy`到自己的闭包空间。也就是复制了一份变量在自己的作用域内部，所以是无法修改外部的变量。通过`__block`关键词修饰之后，通过复制变量的引用地址来修改和访问外部变量的。

### 8. 类别和类扩展的区别

类别可以添加属性 添加的方法不实现会报警告。可以添加实例变量，但是只能自己类访问。扩展可以添加方法，不实现不会报警告。不能添加实例变量，属性可以通过关联对象实现

### 9. 分类的作用？分类和继承的区别？

分类可以将一个臃肿的类分散到多个文件中，可以将功能按照模块划分，多人多人开发一个类。分类可以给系统类添加方法，还可以拦截系统方法。继承拥有对类完全控制权，分类只对于引入文件有效。分类可以在不获取原来代码基础上增加方法，不可以删除方法，也不可以新增属性，分类具有更高的优先级。继承可以添加和删除方法，也可以新增属性。

### 10. 重写一个类的方法用继承好还是分类好? 为什么?

分类只对于本次有效，推荐用分类。

### 11. 若一个类有实例变量`NSString *_foo`，调用`setValue:forKey:`时，可以以`foo`还是`_foo`作为`key`

这个其实是考察我们`KVC`的基本原理，都是可以的.

### 12. `isMemberOfClass` 和 `isKindOfClass` 联系与区别

`isMemberOfClass`只能判断是否是当前的类，`isKindOfClass`可以用来判断是否是当前类或者子类。

### 13. 一个objc对象的isa的指针指向什么？有什么作用？

`isa`其实是一个结构体,`isa`对象指向当前类 当前类指向父类,父类指向元类也就是`NSObject`，元类指向自己。作用是方便找到对应所在的方法。

### 14. 请描述一个你所遇到`retain cycle`例子

这个第一次解除`Block`时候应该经常用到，在`Block`内部使用`self`调用属性或者方法。
### 15.请谈谈内存的使用和优化的注意事项
我们知道创建对象 加载图片 加载文件 创建线程都需要消耗内存，特别是加载图片和加载文件会大量消耗内存。我们还知道在大量循环中创建临时变量就直线性的内存暴涨。通过上面我们可以才去进一步的优化内存，对于创建对象，因为单利会驻留在内存里面，不要创建消耗内存大的对象作为单利。加载图片imageName:会把图片驻留在内存里面做缓存提高下一次访问速度，我们可以把大图片用imageWithContentsOfFile的方法进行加载。尽可能减少多线程创建和数量，使用NSCache作为缓存机制替换我们常用的数组和字段作为缓存手段。在大量循环中用外部变量防止多次创建变量或者在循环内部使用runloop.如果是列表形式，做到数据重用。尽可能少的创建和消耗内存

### 16. runtime 如何实现 weak 属性
`weak`属性其实是系统维护的一个`Hash`表，系统会把`weak`声明的属性的内存地址作为`key`存放在`hash`表里面。当这个属性引用技术为`0`走`dealloc`方法时候，通过对应内存地址作为`key`再从`hash`表移出出去。

### 17. `runtime`如何通过`selector`找到对应的`IMP`地址？

每个类里面都有一个`method_list`数组存储着这个类所有的方法和实现，通过`selector`对应的方法名称找到对应的方法和实现。

### 18. `+(void)load; +(void)initialize；`有什么用处？


`load`放在会在第一次加载类方法会调用，调用顺序是先父类，然后子类，然后分类。`initialize`会在第一次调用类方法或者实例方法时候被调用。

### 19. 如何访问并修改一个类的私有属性？

可以通过`KVC`或者通过`ivar`进行访问.

### 20. Objective-C 如何对已有的方法，添加自己的功能代码以实现类似记录日志这样的功能？


我们可以利用分类，通过交换方法实现。可以在方法之前也可以在方法后面。

```objc
@interface ClassA : NSObject
@end

@implementation ClassA

- (void)printMyName {
    NSLog(@"josercc");
}

@end

@interface ClassA (Print)
@end

@implementation ClassA (Print)

+ (void)load {
    Method method1 = class_getClassMethod(self, @selector(printMyName));
    Method method2 = class_getClassMethod(self, @selector(printMyName1));
    method_exchangeImplementations(method1, method2);
}

- (void)printMyName1 {
    NSLog(@"Hello");
    [self printMyName1];
}

@end
```

### 21. 请说明并比较以下关键词：strong, weak, assign, copy
- `strong` 会持有对象，会使对象引用计数加1
- `weak` 不会持有对象 只是存储了对象的内存地址
- `assgin` 用于声明基本类型 被`assgin`声明的会存放在栈区
- `copy` 会复制一份内存地址,一般用于`NSString NSArray NSDictionary`

### 22. 请说明并比较以下关键词：`__weak，__block`

- `__weak`用来修饰实例变量
- `__block`用来修饰可以在`block`内部修改外部变量

### 23. 请说明并比较以下关键词：atomatic, nonatomic

- `atomatic` 是相对线程安全的因为会自动在`set`和`get`方法进行加锁,但是会额外的消耗性能
- `nonatomic` 是线程不安全的性能好

### 24. 下面代码中有什么bug?

```objc
- (void)viewDidLoad {
  UILabel *alertLabel = [[UILabel alloc] initWithFrame:CGRectMake(100,100,100,100)];
  alertLabel.text = @"Wait 4 seconds...";
  [self.view addSubview:alertLabel];

  NSOperationQueue *backgroundQueue = [[NSOperationQueue alloc] init];
  [backgroundQueue addOperationWithBlock:^{
    [NSThread sleepUntilDate:[NSDate dateWithTimeIntervalSinceNow:4]];
    alertLabel.text = @"Ready to go!”
  }];
}
```

> 我们是在子线程操作的UI应该在主线程操作UI不会更新(`在Xcode11`测试依然会更新，只是会报不能在子线程更新UI的警告 `-[UILabel setText:] must be used from main thread only`)。
> 

### 25. Object-C有多继承吗？没有的话用什么代替？cocoa 中所有的类都是NSObject 的子类

`OC`是没有多继承的，可以通过协议进行代替。

### 26. Object-C有私有方法吗？私有变量呢？

`OC`并不存在严格来说的私有方法和私有变量。对于用`@public`修饰的属性和方法在`.h`都可以被外部调用。在`.m`实现的方法和实例变量只能本类调用就称作私有的。`OC`是一个有运行时的特性，所以我们可以获取方法列表找到私有方法通过消息转发调用。对于私有变量我们可以通过`KVC`,或者`Ivar`进行访问。所以`OC`不存在严格意义上面的私有方法和私有变量

### 27. 关键字const什么含义
对于`const`来说 声明在谁前面就意味着谁不能修改，如果`const`在第一位则向右移一位。比如`int ` `const a`等效果于`const int a`都是让整形的`a`不可变。声明`const`可以让编译时候代码更紧凑，让系统修改变量时候随时产生报错，防止`BUG`的产生。

### 28. tableView的重用机制？

`UITableView`有两个实例变量，`visiableCells`保存当前界面显示的`Cell`的数组，`reusableTableCells`保存在可以重用的`Cell`的字典。当第一次加载界面，`reusableTableCells`没有任何重用的`Cell`就会走`UITableViewCell`的初始化方法进行创建，之后添加到`visiableCells`数组里面。当界面滚动时候，原本展示的`Cell`消失在屏幕之后。对应的`Cell`对象就会添加到`reusableTableCells`里面，即而从`visiableCells`数组里面进行移出。当一个新的`cell`将展示时候从`reusableTableCells`查看是否存在重用的，如果存在就展示重用的，如果没有就重新走创建的流程。

### 29. ViewController的didReceiveMemoryWarning是在什么时候调用的？默认的操作是什么？

当手机内存运行不足时候会调用`ViewController`的`didReceiveMemoryWarning`方法，默认时尝试释放`ViewController`所拥有的`View`。我们也可以重写做其他释放内存的任务。

### 30. delegate和notification区别，分别在什么情况下使用？

`delegate`是一对一的关系，`notification`是一对多的关系。`delegate`通畅用于开放接口用于其他模块的调用，`notification`可以做到模块分离，通畅用于模块之间的通信。

### 31. id、nil代表什么？
`id`代表在`OC`里面的对象的指针，`nil`代表是指针指向一个空的对象。

### 32. timer的间隔周期准吗？为什么？怎样实现一个精准的timer?
`NSTimer`因为是添加到`runloop`中的走的是默认的`mode`，当滑动表格的时候就会切换对应`mode`停止`timer`。`runloop`因为优先在触发时候执行输入源，才会执行`runloop`中的任务，虽有会有50-100毫秒的误差。

我们想做一个精准的`timer`就需要创建一个新的`runloop`来不受其他输入源的影响。

添加Timer到当前的`runloop`设置为`NSRunLoopCommonModes`

```objc
_timer = [NSTimer scheduledTimerWithTimeInterval:1.0
				      target:self
				    selector:@selector(timerStart)
				    userInfo:nil
				     repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:_timer forMode:NSRunLoopCommonModes];
```

- 在新线程添加`NSTimer`

```objc
dispatch_async(dispatch_queue_create("", 0), ^{
@autoreleasepool {
    self->_timer = [NSTimer scheduledTimerWithTimeInterval:1.0
					      target:self
					    selector:@selector(timerStart)
					    userInfo:nil
					     repeats:YES];
    [[NSRunLoop currentRunLoop] run];
}
});
```
- GCD

```objc
timer = 
dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_main_queue());
dispatch_source_set_timer(timer, DISPATCH_TIME_NOW, 1.0 * NSEC_PER_SEC, 0 * NSEC_PER_SEC);
dispatch_source_set_event_handler(timer, ^{
NSLog(@"timer");
});
dispatch_resume(timer);
```

对比还是第三种方案简单，第二种执行的任务会在子线程，容易写出`BUG`，第一种容易遗漏添加到`runloop`设置对应的`mode`。第三种有现成的代码块，而且还十分的精准。




### 33. 一个 `NSObject` 对象占用多少内存？

系统分配`16`个字节，但是真正占用只有`8`个字节。

### 34 . 方法和选择器有何不同？(`Difference between method and selector`?)

`selector`只是代表方法名称，而`method`包含了名称和实现。

### 35. 你是否接触过OC中的反射机制？简单聊一下概念和使用

反射机制就是通过字符串反射为对应的类，协议，方法。或者将协议，方法或者类反射为字符串。通常用于做模块化跳转，或者用于做`deeplink`等运行时创建类等功能。

### 36. `Objective-C`中的协议默认是`@optional`还是`@require？`在使用协议的时候应当注意哪些问题？

协议默认为`@require`，使用时候要注意用`weak`声明代理，防止循环引用。

### 37 .通知的实现机制？为什么有时候监听的名称和发送通知名称相同却收不到回调。

通知底层核心是存在一个`notication_map`的字典，通知的名称作为`Key`，`Value`为数组结构，数组的每个元素包含了监听的对象，方法名称，还有传递的参数。当发送通知的时候，会在`notication_map`的字典当中找到对应`key`的数组，通过遍历数组，对数组里面所有的监听者发送通知。当发送参数和监听的参数不一样的时候，会被忽略，这就是为什么名字相同却收不到通知。

当接受通知的参数为空可以接受发送通知参数为空或者不为空

可以接受到通知

```objc
/// 当object=nil；则都可以接受到.
/// 当不为空的时候,则key为object过滤Observer。
	NSString *name = @"addNotification";
	NSNotificationCenter * center =  [NSNotificationCenter defaultCenter];
	[center addObserver:self
			   selector:@selector(receData:)
				   name:name
				 object:nil];
	[center postNotificationName:name object:@(1)];
	[center postNotificationName:name object:nil];
	


	NSString *name = @"addNotification";
	NSNotificationCenter * center =  [NSNotificationCenter defaultCenter];
	NSObject *key  = [NSObject new];
	[center addObserver:self
			   selector:@selector(receData:)
				   name:name
				 object:key];
	/// 1. 不接受
	/// 2. 不接受
	/// 3. 接受 当object设置的时候，下次一定要传过来才能接受到通知
	[center postNotificationName:name object:@(1)];
	[center postNotificationName:name object:nil];
	[center postNotificationName:name object:key];
	
```


### 38. `KVO (Key-value observing)`的实现机制

`KVO`系统通过`isa`混淆技术通过创建类的不可见子类，通过重写`set`和`get`方法来实现监听机制的。对于没有调用`set`和`get`方法是调用不了监听机制的，需要手动调用`willChangeValueForKey`和`didChangeValueForKey`触发。

### 39. 能否向已经编译的类添加实例变量

不能，因为编译之后类的内存大小已经固定不能再添加实例变量，但是可以动态的添加实例变量。















 
 