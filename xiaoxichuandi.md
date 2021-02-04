# 消息传递

### 1. `runtime` 查找方法流程图
<details>
- 1. 123
- 2. 3
- 4. 234
- 5. 124


![大师班第六天课程笔记](media/%E5%A4%A7%E5%B8%88%E7%8F%AD%E7%AC%AC%E5%85%AD%E5%A4%A9%E8%AF%BE%E7%A8%8B%E7%AC%94%E8%AE%B0.png)

</details>

#### 2. 方法查找以及拦截流程
- 1.`objc_msgSend`发送消息，会首先在`cache`中查找，查找不到则去方法列表(顺序是`cache->class_rw_t->supclass cache ->superclass class_rw_t ->动态解析`)
- 2.第二步是动态解析，能在`resolveInstanceMethod`或`+ (BOOL)resolveClassMethod:(SEL)sel`来来拦截，可以给`class`新增实现函数，达到不崩溃目的
- 3.第三步是消息转发，转发第一步可以在`+ (id)forwardingTargetForSelector:(SEL)aSelector`或`- (id)forwardingTargetForSelector:(SEL)aSelector`拦截类或实例方法，能将对象方法转发给其他对象，也能将对象方法转发给类方法，也可以将类方法转发给实例方法
- 4.第三步消息转发的第二步可以在`+ (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`或`- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`实现拦截类和实例方法并返回函数签名
- 5.第三步消息转发的第三步可以`+ (void)forwardInvocation:(NSInvocation *)anInvocation`或`- (void)forwardInvocation:(NSInvocation *)anInvocation`实现类方法和实例方法的调用和获取返回值

<details>
  <summary>点击查看消息转发流程详细内容</summary>
  
![](./media/16122550523975.jpg)

</details>


### 3. 消息转发
[消息转发](https://juejin.cn/post/6844903892765900807)

<details>
![消息转发机制](media/%E6%B6%88%E6%81%AF%E8%BD%AC%E5%8F%91%E6%9C%BA%E5%88%B6.png)


</details>


## 4. 