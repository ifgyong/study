# 消息转发

1. 首先查找系统的`cache`中是否有该函数，没有的话则在`class`和`superclass`中查找是否有该方法。
2. 没有的话则进入动态解析，在这个函数中可以利用`runtime`添加未实现的方法。添加之后，系统又会重新回到步骤1
3. 快速转发给`- (id)forwardingTargetForSelector:(SEL)aSelector`的返回值
4. 慢速转发给`- (void)forwardInvocation:(NSInvocation *)anInvocation`,然后手动调用`[anInvocation invoke]`.


### 1. 动态解析
在`+[resolveInstanceMethod:]`中根据`sel`来动态添加方法。
```objc
+(BOOL)resolveInstanceMethod:(SEL)sel{
	NSLog(@"%s",__func__);

	if ([NSStringFromSelector(sel) isEqualToString: @"test"]) {
		Method m = class_getInstanceMethod(self.class, @selector(t));
		class_addMethod(self,
							sel,
							method_getImplementation(m),
							method_getTypeEncoding(m));
		return  YES;
	}
	return [super resolveInstanceMethod:sel];
}
```

### 2. 快速转发
转发给已经实现该`sel`的`instance`或者`class`.

```objc
- (id)forwardingTargetForSelector:(SEL)aSelector{
	NSLog(@"%s",__func__);

	if (aSelector==@selector(test)) {
		return [ViewController alloc];
	}
	return [super forwardingTargetForSelector:aSelector];
}
```

### 3. 慢速转发
慢速转发会调用`-[methodSignatureForSelector:]`获取函数签名，然后用`NSInvocation`手动调用。
```objc
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector{
	NSLog(@"%s",__func__);

	if (aSelector == @selector(test)) {
		NSMethodSignature *sign=[NSMethodSignature signatureWithObjCTypes:"v16@0:8"];
		return  sign;
	}
	return [super methodSignatureForSelector:aSelector];
}

- (void)forwardInvocation:(NSInvocation *)anInvocation{
	anInvocation.selector=@selector(t);
	[anInvocation invoke];
}
```


### 4. `NSTimer`前引用`self`，使用`NSProxy`解决相互强引用

```objc
@implementation Proxy
- (instancetype)initWithTarget:(id)target {
	_target = target;
	return self;
}

//类方法
+ (instancetype)proxyWithTarget:(id)target {
	return [[self alloc] initWithTarget:target];
}

#pragma mark - over write

//重写NSProxy如下两个方法，在处理消息转发时，将消息转发给真正的Target处理
- (void)forwardInvocation:(NSInvocation *)invocation {
	[invocation invokeWithTarget:self.target];
}

//
- (NSMethodSignature *)methodSignatureForSelector:(SEL)selector {
	return [_target methodSignatureForSelector:selector];
}
- (void)dealloc
{
	NSLog(@"%s",__func__);
}


// 使用例子:
-(void)setTimer{
	Proxy *xy= [Proxy proxyWithTarget:self];

	timer=[NSTimer scheduledTimerWithTimeInterval:0.2
										   target:xy
										 selector:@selector(p)
										 userInfo:nil
										  repeats:YES];
}
```