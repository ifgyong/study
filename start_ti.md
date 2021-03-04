### 1. load和initialize区别
1. `load`在`main`之前调用，先调用`类`然后调用`Category`，先调用`父`后`子`。
2. `initialize`在`main`之后，子类没实现会重复调用父类，先父后子类，先类后`Category`。
3. 类与类之前的先后顺序按照编译顺序 工程文件` Build Phases->Compile Sources`的顺序进行，`Category`也是如此。
4. `class` 和`Category`都会执行`load`函数，但是`initialize`函数是`Category`会覆盖`class`中已经实现的的函数，而且当子类没实现则会调用父类2次所以需要`if[self class]==DemoClass]`来判断是否是当前类的调用，子类实现则调用父类函数，然后调用子类函数。

### 2. 启动前方法加载顺序
>
1. All initializers in any framework you link to
2. All +load methods in your image.
3. All C++ static initializers and C/C++ __attribute__(constructor) functions in your image
4. All initializers in frameworks that link to you

翻译：

1. 动态库的初始化包含`+load`然后执行，`c++`函数
2. 自己写的`+load`
3. 自己写的`c++`函数

