# 优势
熟悉底层原理，了解flutter，了解一般数据结构

### 1. 内存优化 
1. 图片下采样
2. 离屏渲染
3. kvo 页面统计加载时间
4. LRU算法

### 2. 启动优化
1. 二进制插桩
2. 业务接口合并，重复接口改成内存数据服务，用到的地方还需要读即可。
3. load函数迁移
4. 废弃class清理

### 3. 崩溃处理
1. 让`APP`不立即次崩溃，可以手机崩溃堆栈发送到服务器

```objc
CFRunLoopRef runloopRef=CFRunLoopGetCurrent();
	
while (!dismiss) {
	for (NSString * item in (__bridge NSArray *)CFRunLoopCopyAllModes(runloopRef) ) {
		CFRunLoopRunInMode((CFStringRef)item, 0.01, false);
	}
}
```
### 4. runtime的实际应用

1. 利用关联对象存储时间，然后在`+load`调换`hitTest`函数，根据点击的时间间隔来判断是否响应该次点击。
2. `jsonModel`利用`runtime`和`kvc`来实现的
3. `KVO`其实内部就`runtime`动态生成了`NSKVONotifying_XXX`类，重写了获取`class`的方法。
4. 做了一个动态库来记录`method_exchange`的历史记录,需要将该动态库调整到第一个才行。

### 5. 读写锁 与缓存
1. `momeryCache`利用自旋锁/`os_unfair_lock`，性能更好
2. `diskCache`利用`mutex_lock`互斥锁就能满足需求
3. 读写常用文件需要家读写锁来防止资源竞争
4. 读写锁是**多读单写，读写互斥**。
5. [FYML 记录方法交换历史](https://github.com/ifgyong/FYMSL)
6. [fishhook原理](https://www.jianshu.com/p/d4dd4eb27b50)/[冬瓜fishhook原理](https://www.desgard.com/iOS-Source-Probe/C/fishhook/%E5%B7%A7%E7%94%A8%E7%AC%A6%E5%8F%B7%E8%A1%A8%20-%20%E6%8E%A2%E6%B1%82%20fishhook%20%E5%8E%9F%E7%90%86%EF%BC%88%E4%B8%80%EF%BC%89.html)
7. `os_un_fair_lock`互斥锁，`NSLock`是互斥锁


![-w466](media/16173486146063.jpg)

> 首先遍历`load_com`，找到`link_edit`基地址/`systab`和`dysytab`确认2个`section`的偏移量，，然后遍历一次找到`symtab`和`dysytab`中的符号，将符号指向需要替代的地址。

> 首先是`image`已经读入系统，然后修改`image`中的符号表和动态符号表中的函数地址，指向需要的函数即可。
> 
> 修改之前调用还是直接调用系统函数，修改之后调用自己的函数。因为符号表在`__DATA`段，修改不限制。
> 
> 可以根据自定义函数的地址和**section**范围来判断是否被**中间人**`hook`了。

1. 为什么分类的方法可以覆盖类的方法？

```objc
 if (mlist) {
    if (mcount == ATTACH_BUFSIZ) {
		// attachCategories 追加分类方法 然后调用attachLists
		//将后边追加的mcount个数据，挪到了数组前边。
        prepareMethodLists(cls, mlists, mcount, NO, fromBundle, __func__);
        rwe->methods.attachLists(mlists, mcount);
        mcount = 0;
    }
    mlists[ATTACH_BUFSIZ - ++mcount] = mlist;
    fromBundle |= entry.hi->isBundle();
}
```
![](media/16176929282297.jpg)


### 6. flutter
[携程酒店flutter性能优化](https://www.toutiao.com/article/7147521191882981920/?app=news_article&timestamp=1667270519&use_new_style=1&req_id=202211011041580102100430420523BFC4&group_id=7147521191882981920&share_token=3987DD47-5236-4A52-8A35-A582D445EF55&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_ios&utm_campaign=client_share&wxshare_count=1&source=m_redirect&wid=1667270560382)
 1. 局部刷新
 2. 复杂的UI利用分帧刷新处理，首先用一个Container占位，等下一帧开始渲染时候再进行刷新
 3. 负责图像加入RepaintBoundary，引擎认为足够复杂则会缓存下来
 4. 列表页包含的数据直接url带入详情页，降低服务端压力
 5. 内存泄露 利用devtool查看页面实例对象，和native交互的时候不使用feature，使用
 ```
 void callNative () {
Future future = FlutterBridge. callNative ("method");
    _streamSubscription?.cancel();
    _streamSubscription = future?.asStream()?.listen((value)
{
        do something;
});
}

void onPageDestroy() {
    _ streamSubscription?.cancel();
}
 ```
 6. 页面数据的预加载，提高用户体验 
 
**一些观察者模式中的订阅者在页面退出时没有取消订阅**

### 7. flutter [做了一个动画库 包含了30多个动画](https://github.com/ifgyong/flutter_easyHub),还有[QQ气泡动画](https://github.com/ifgyong/flutter_qq_bubble)
### 8. py做的自动化数据校验功能，生成各种尺寸Icon脚本
