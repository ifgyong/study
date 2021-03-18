# 消息传递题

### 1. 题目1

```objc
@interface FYPerson : NSObject
-(void)hello;
@end
@implementation FYPerson
@end

// 类别
@interface FYPerson (FY)
-(void)readOnce;
+(void)read;
@end

@implementation FYPerson (FY)
-(void)readOnce{
	NSLog(@"%s",__func__);

}
+(void)read{
	NSLog(@"%s",__func__);
}
@end


FYPerson * p=[[FYPerson alloc]init];
[p performSelector:@selector(readOnce)];
[FYPerson performSelector:@selector(read)];
```

#### 1.1 会崩溃吗？为什么？

> 不会，`[p performSelector:@selector(readOnce)]`会执行`FYPerson(FY)`分类中的函数，`[FYPerson performSelector:@selector(read)]`会执行`FYPerson(FY)`的函数`read`.因为`FYPerson`中未定义，但是分类中实现了。