# KVC&KVO
### 1.KVC&KVO 知识图
<details>
  <summary>点击查看详细内容</summary>
![查看知识图](./img/kvo.png)
</details>

## 2.KVC

```
  // 4:KVC - 访问非对象属性
    ThreeFloats floats = {1., 2., 3.};
    NSValue *value  = [NSValue valueWithBytes:&floats objCType:@encode(ThreeFloats)];
    [person setValue:value forKey:@"threeFloats"];
    NSValue *reslut = [person valueForKey:@"threeFloats"];
    NSLog(@"%@",reslut);
    
    ThreeFloats th;
    [reslut getValue:&th] ;
    NSLog(@"%f - %f - %f",th.x,th.y,th.z);
    
    // 5:KVC - 层层访问
    LGStudent *student = [[LGStudent alloc] init];
    student.subject    = @"iOS";
    person.student     = student;
    [person setValue:@"大师班" forKeyPath:@"student.subject"];
    NSLog(@"%@",[person valueForKeyPath:@"student.subject"]);
```
## 函数调用优先级顺序
### 原理 setXXX
- 1. `setName:`
- 2. `_setName:`
- 3. `setIsName:`
![](media/16121704612214.jpg)


### 原理 getXXX
- 1. `getName:`
- 2. `_name:`
- 3. `isName:`
- 4. `_isName:`

`成员变量的查找顺序`
![](media/16121706654167.jpg)

## 3. KVO
[apple 文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)

### 1.`[obj addObserver:forKeyPath:options:context:]`中的`context`有什么用？
<details>
  <summary>点击查看详细内容</summary>
利用`context`来区分不同的对象相同的`keypath`的值,平时穿NULL，多个对象被观察，使用指针来区分即可。

```
	static NSString *KPersonKey=@"KPersonKey";
	static void* personKey = &KPersonKey;
    [self.person addObserver:self forKeyPath:@"nickName" options:(NSKeyValueObservingOptionNew) context:&personKey];
```

</details>


### 2. 需要在dealloc中移除观察者吗？
需要，如果被观察的是单例，容易发送崩溃，野指针。

### 3. 通过automaticallyNotifiesObserversForKey：来改变是否自动观察
`YES: `自动观察
`NO：` 需要手动`willChangeForKey：`来触发。
```
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key{
	return  NO;
}
```