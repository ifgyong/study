# 自动释放池 
1. 临时变量什么时候释放？
2. 自动释放池原理？
3. 自动释放池能否嵌套使用？

> 1. 在产生很多临时变量的时候使用，出了释放池范围则进行释放。
> 2. 自动释放池是一个双向链表，里边有`child`和`parent`指针，自动分页，4k一页，当第一页满的话，则`next`指向第二页。
> 3. 可以嵌套使用。
> 4. `#define POOL_BOUNDARY nil` 是哨兵,当新建`page`的时候第一次插入，作用是`release`的时候，倒序销毁到哨兵，这个`page`就为空了。
> 

[iOS 内存管理（四）： 自动释放池详解 ](https://juejin.cn/post/7011446159885467684)

```
@implementation SSLPerson

- (void)dealloc
{
    NSLog(@"%s", __func__);
}
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    @autoreleasepool {
        SSLPerson *person = [[[SSLPerson alloc] init] autorelease];
    }
    NSLog(@"%s", __func__);
}

- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    NSLog(@"%s", __func__);
}

- (void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
    NSLog(@"%s", __func__);
}
@end

运行结果：
-[SSLPerson dealloc]
-[ViewController viewDidLoad]
-[ViewController viewWillAppear:]
-[ViewController viewDidAppear:]
```

可以看到，手动添加到指定的`@autoreleasepool`中的`autorelease`对象，在`@autoreleasepool`大括号结束时就会释放了，不受`RunLoop`控制.

释放在`viewdidLoad` 之前，就是 结构体出栈销毁。



