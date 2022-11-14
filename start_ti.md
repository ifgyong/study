### 1. load和initialize区别
1. `load`在`main`之前调用，先调用`类`然后调用`Category`，先调用`父`后`子`。
2. `initialize`在`main`之后，子类没实现会重复调用父类，先父后子类，先类后`Category`。
3. 类与类之前的先后顺序按照编译顺序 工程文件` Build Phases->Compile Sources`的顺序进行，`Category`也是如此。
4. `class` 和`Category`都会执行`load`函数，但是`initialize`函数是`Category`会覆盖`class`中已经实现的的函数，而且当子类没实现则会调用父类2次所以需要`if[self class]==DemoClass]`来判断是否是当前类的调用，子类实现则调用父类函数，然后调用子类函数。

### 2. 启动前方法加载顺序
>
1. `All initializers in any framework you link to`
2. `All +load methods in your image.`
3. `All C++ static initializers and C/C++ __attribute__(constructor) functions in your image`
4. `All initializers in frameworks that link to you`

翻译：

1. 动态库的初始化包含`+load`然后执行，`c++`函数
2. 自己写的`+load`
3. 自己写的`c++`函数



##  启动加载时机和需要处理的事情

`app `启动 内核创建虚拟页，虚拟页通过`mmap()`映射到 物理内存，因为`app` 函数和代码很多没必要一次性读取出来，
则产生了分页的概念。**在Mac OS系统上每页为4KB，在iOS系统上每页为16KB**,则启动的时候需要调用很多函数，每当函数未读取
到虚拟页上，系统内核接管进程，主线程`block`掉，当`block`掉多次则耗时较长，需要将每次读取的数据都是启动需要的，则编译的时候排列顺序设置成需要的`map`
减少了` page fault `次数，提高启动时间。[具体获取顺序步骤](https://www.jianshu.com/p/559f724933ff)

1. 动态库首先加载，一般为系统动态库几百个
3. `rebase `  `app` 启动需要对每个`section`进行 `ASLR` 进行地址布局随机化，增加一个`offset`，之前的指针需要进行`+offset`偏移处理。自己代码偏移量是一样的，其他系统动态库，每个动态库偏移量不一样。Xcode 断点模式下，可以使用 `image list -o `查看所有偏移量。
4. `rebinding` `non lazy` 的数据需要重新绑定到系统的共享内存的实际地址上
5. 依赖库的`+load`和`c++ _attribute__(constructor) funcs`,然后是自己代码的 `+load`和`c++ _attribute__(constructor) funcs`.
6. `C/C++ __attribute__(constructor) functions ` 在main之前执行，可以在这个函数中增加以下代码实现读取`macho`读取`section __DATA `数据.

```
NSArray<NSString *>* CSReadConfiguration(char *sectionName,const struct mach_header *mh) {
    NSMutableArray<NSString*> *annotionDefines = nil;
    unsigned long size = 0;
#ifndef __LP64__
    uintptr_t *memory = (uintptr_t*)getsectiondata(mh, SEG_DATA, sectionName, &size);
#else
    const struct mach_header_64 *mhp64 = (const struct mach_header_64 *)mh;
    uintptr_t *memory = (uintptr_t*)getsectiondata(mhp64, SEG_DATA, sectionName, &size);
#endif
    unsigned long counter = size/sizeof(void*);
    for(int idx = 0; idx < counter; ++idx){
        char *string = (char*)memory[idx];
        NSString *str = [NSString stringWithUTF8String:string];
        if(!str)continue;
        if (!annotionDefines) annotionDefines = [[NSMutableArray alloc] init];
        [annotionDefines addObject:str];
    }
    return annotionDefines;
}

static void dyld_add_image_callback(const struct mach_header *mh, intptr_t vmaddr_slide) {
    [[[CSMonitorCenter sharedInstance] applicationTimeProfiler] beginAnnotationRead];
    Dl_info image_info;
    if (dladdr(mh, &image_info)) {
#if TARGET_OS_SIMULATOR
        const char *simulator_env_lib_name_prefix = "/Users";
        if (strncmp(simulator_env_lib_name_prefix, image_info.dli_fname, strlen(simulator_env_lib_name_prefix)) == 0) {
            NSArray<NSString*> *annotationDefines = CSReadConfiguration(CSAnnotationSectionName, mh);
            [[CSAnnotation sharedInstance] appendAnnotationDefines:annotationDefines];
        }
#else
        char main_lib_name_prefix[] = "/var";
        char private_lib_name_prefix[] = "/private";
        if (strncmp(private_lib_name_prefix, image_info.dli_fname, strlen(private_lib_name_prefix)) == 0 || strncmp(main_lib_name_prefix, image_info.dli_fname, strlen(main_lib_name_prefix)) == 0) {
            NSArray<NSString*> *annotationDefines = CSReadConfiguration(CSAnnotationSectionName, mh);
            [[CSAnnotation sharedInstance] appendAnnotationDefines:annotationDefines];
        }
#endif
    }
    [[[CSMonitorCenter sharedInstance] applicationTimeProfiler] endAnnotationRead];
}

__attribute__((constructor))
void initAnnotationsFunc() {
    _dyld_register_func_for_add_image(dyld_add_image_callback);
}
```
增加`_dyld_register_func_for_add_image` 根据`sectionname` 判断是否需要读取数据，存入的数据是`string`的指针，实际的数据存储在 `__TEXT cstring`中，读取到单利变量中。以供后续
处理，在编译的时候使用`__attribute((used, section("__DATA,"#sectionName""))) = "task:task1"`进行写入到 `section __DATA ` `name`是 `sectionName`。

###### 优点： 业务线解耦
在`will launch `中调用服务 `task` 的初始化和执行协议函数。不重要的`task`再`root vc didappear`之后进行执行。

