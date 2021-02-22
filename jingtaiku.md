# 静态库与动态库

1. 静态链接，只会将用到的.o文件链接到可执行文件中，按需链接。
2. 静态库在打包的时候已经打包到可执行文件，
3. 动态库是在执行可执行文件进行递归链接的，不会占用可执行文件体积
4. 静态库会有冗余，动态库的，相同的东西都被打包在可执行文件中，不会冗余，体积更小。**不一定更小，动态库vmess size 最小32k，静态库无此限制。**


静态库加载更快原因：
1. 每加载一个单独的动态库，都要走一遍加载主app的类似流程，递归加载，符号查找，bind，rebase
2. 静态链接需要的external符号可以更少，原来动态库导出的只给主映像用的符号都可以变成internal