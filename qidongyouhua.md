# 启动

## 1. 启动顺序
<details>
  <summary>点击查看详细内容</summary>
  - 1. `framework initializers`
  - 2. image `+load`
  - 3. `c/c++ __attribute__`
  - 4. 所有 `initalizers`

> `+load` 函数父类先于子类，类先于类别。
>   可以将`+load`函数的实现在`initalizers`+`dispatch_once`来保证只执行一次。减少`pre-main`启动时间
  
![](media/16123179478932.jpg)

</details>


### 2. 火焰图

火焰图swift插件

https://github.com/lennet/FlameGraph


>使用这个插件，然后配合instruments的App Launch，选择主线程，deep copy，并执行命令即可生成火焰图。
> 首先安装插件`Mint`，使用`Mint`安装`FlameGraph`，然后生成火焰图。


#### 1. 手动复制函数结构

![-w1358](media/16137229080195.jpg)


#### 2. 命令
`swift run FlameGraph ./a.pdf`

#### 3 优化结果

|耗时|优化阶段|
|---|---|
|2.65s|优化前|
|1.275s|优化后|

采用的是 `Instruments App Launch`统计的耗时，均测试了**10**次，平均数即为表格数据，**提高50%**。

根据`Instruments App Launch`，可以查看每个函数耗时，根据函数耗时来做优化。
`Instruments App Launch`其实已经统计到了 主页的`viewDidAppear:`，这个数字和APP绘画出来第一帧，会有所偏大。

> 优化函数在读取文件、图片、初始化、解析数据、读取写入持久化数据、等耗时操作，延迟或者在子线程异步执行。

#### 4. 二进制重排
二进制插桩重排，经过测试，冷启动`page fault` 次数平均在`1000-3000`次，耗时平均是`100`us，左右，也就是优化过最多可以节省`1500*0.1us=150ms`,也就是在`0.1s`左右。完全在冷启动正常波动范围，效果不是很明显。

#### [查看二进制重排步骤](./erjinzhichazhuang.md)




## 总结：
1. main之前动态库比较少，则基本无优化内容
2. 多线程、延迟初始化。（效果比较明显）
3. UI用代码不用xib。
4. 

