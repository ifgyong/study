# LLVM
## clang 
使用一个简单例子
### 1. 预处理阶段
将文件`main.m`预处理输出到`main2.m`.


```
clang	-E	main.m>>main2.m
```

### 2. 编译阶段

### 2.1 语法分析
将代码分析
`clang	-fmodules	-fsyntax-only -Xclang -dump-tokens main.m`

```
int main(int argc, const c'		Loc=<main.m:9:1>
int 'int'	 [StartOfLine]	Loc=<main.m:13:1>
identifier 'test'	 [LeadingSpace]	Loc=<main.m:13:5>
l_paren '('		Loc=<main.m:13:9>
int 'int'		Loc=<main.m:13:10>
identifier 'a'	 [LeadingSpace]	Loc=<main.m:13:14>
comma ','		Loc=<main.m:13:15>
int 'int'		Loc=<main.m:13:16>
identifier 'b'	 [LeadingSpace]	Loc=<main.m:13:20>
r_paren ')'		Loc=<main.m:13:21>
l_brace '{'		Loc=<main.m:13:22>
return 'return'	 [StartOfLine] [LeadingSpace]	Loc=<main.m:14:5>
identifier 'a'	 [LeadingSpace]	Loc=<main.m:14:12>
plus '+'	 [LeadingSpace]	Loc=<main.m:14:14>
identifier 'b'	 [LeadingSpace]	Loc=<main.m:14:16>
plus '+'	 [LeadingSpace]	Loc=<main.m:14:18>
```

### 2.2 词法分析

`clang -fmodules -fsyntax-only -Xclang -ast-dump  main.m`


```
TranslationUnitDecl 0x7f8ce601c608 <<invalid sloc>> <invalid sloc> <undeserialized declarations>
|-TypedefDecl 0x7f8ce601cea0 <<invalid sloc>> <invalid sloc> implicit __int128_t '__int128'
| `-BuiltinType 0x7f8ce601cba0 '__int128'
|-TypedefDecl 0x7f8ce601cf10 <<invalid sloc>> <invalid sloc> implicit __uint128_t 'unsigned __int128'
| `-BuiltinType 0x7f8ce601cbc0 'unsigned __int128'
|-TypedefDecl 0x7f8ce601cfb0 <<invalid sloc>> <invalid sloc> implicit SEL 'SEL *'
| `-PointerType 0x7f8ce601cf70 'SEL *'
|   `-BuiltinType 0x7f8ce601ce00 'SEL'
|-TypedefDecl 0x7f8ce601d098 <<invalid sloc>> <invalid sloc> implicit id 'id'
| `-ObjCObjectPointerType 0x7f8ce601d040 'id'
|   `-ObjCObjectType 0x7f8ce601d010 'id'
|-TypedefDecl 0x7f8ce601d178 <<invalid sloc>> <invalid sloc> implicit Class 'Class'
| `-ObjCObjectPointerType 0x7f8ce601d120 'Class'
|   `-ObjCObjectType 0x7f8ce601d0f0 'Class'
|-ObjCInterfaceDecl 0x7f8ce601d1d0 <<invalid sloc>> <invalid sloc> implicit Protocol
|-TypedefDecl 0x7f8ce601d568 <<invalid sloc>> <invalid sloc> implicit __NSConstantString 'struct __NSConstantString_tag'
| `-RecordType 0x7f8ce601d340 'struct __NSConstantString_tag'
|   `-Record 0x7f8ce601d2a0 '__NSConstantString_tag'
|-TypedefDecl 0x7f8ce701ce00 <<invalid sloc>> <invalid sloc> implicit __builtin_ms_va_list 'char *'
| `-PointerType 0x7f8ce601d5c0 'char *'
|   `-BuiltinType 0x7f8ce601c6a0 'char'
|-TypedefDecl 0x7f8ce701d108 <<invalid sloc>> <invalid sloc> implicit __builtin_va_list 'struct __va_list_tag [1]'
| `-ConstantArrayType 0x7f8ce701d0b0 'struct __va_list_tag [1]' 1
|   `-RecordType 0x7f8ce701cef0 'struct __va_list_tag'
|     `-Record 0x7f8ce701ce58 '__va_list_tag'
|-ImportDecl 0x7f8ce701d930 <main.m:9:1> col:1 implicit Darwin.C.stdio
|-FunctionDecl 0x7f8ce701db10 <line:13:1, line:15:1> line:13:5 test 'int (int, int)'
| |-ParmVarDecl 0x7f8ce701d988 <col:10, col:14> col:14 used a 'int'
| |-ParmVarDecl 0x7f8ce701da08 <col:16, col:20> col:20 used b 'int'
| `-CompoundStmt 0x7f8ce701dd00 <col:22, line:15:1>
|   `-ReturnStmt 0x7f8ce701dcf0 <line:14:5, col:20>
|     `-BinaryOperator 0x7f8ce701dcd0 <col:12, col:20> 'int' '+'
|       |-BinaryOperator 0x7f8ce701dc90 <col:12, col:16> 'int' '+'
|       | |-ImplicitCastExpr 0x7f8ce701dc60 <col:12> 'int' <LValueToRValue>
|       | | `-DeclRefExpr 0x7f8ce701dc20 <col:12> 'int' lvalue ParmVar 0x7f8ce701d988 'a' 'int'
|       | `-ImplicitCastExpr 0x7f8ce701dc78 <col:16> 'int' <LValueToRValue>
|       |   `-DeclRefExpr 0x7f8ce701dc40 <col:16> 'int' lvalue ParmVar 0x7f8ce701da08 'b' 'int'
|       `-IntegerLiteral 0x7f8ce701dcb0 <col:20> 'int' 3
`-FunctionDecl 0x7f8ce910edb0 <line:18:1, line:22:1> line:18:5 main 'int (int, const char **)'
  |-ParmVarDecl 0x7f8ce701dd30 <col:10, col:14> col:14 argc 'int'
  |-ParmVarDecl 0x7f8ce910ec60 <col:20, col:38> col:33 argv 'const char **':'const char **'
  `-CompoundStmt 0x7f8ce910f690 <col:41, line:22:1>
    |-DeclStmt 0x7f8ce910f418 <line:19:5, col:20>
    | |-VarDecl 0x7f8ce910eeb8 <col:5, col:13> col:9 used a 'int' cinit
    | | `-IntegerLiteral 0x7f8ce910ef20 <col:13> 'int' 10
    | `-VarDecl 0x7f8ce910ef58 <col:5, col:18> col:16 used b 'int' cinit
    |   `-IntegerLiteral 0x7f8ce910efc0 <col:18> 'int' 11
    |-CallExpr 0x7f8ce910f600 <line:20:5, col:22> 'int'
    | |-ImplicitCastExpr 0x7f8ce910f5e8 <col:5> 'int (*)(const char *, ...)' <FunctionToPointerDecay>
    | | `-DeclRefExpr 0x7f8ce910f430 <col:5> 'int (const char *, ...)' Function 0x7f8ce910efe8 'printf' 'int (const char *, ...)'
    | |-ImplicitCastExpr 0x7f8ce910f648 <col:12> 'const char *' <NoOp>
    | | `-ImplicitCastExpr 0x7f8ce910f630 <col:12> 'char *' <ArrayToPointerDecay>
    | |   `-StringLiteral 0x7f8ce910f488 <col:12> 'char [3]' lvalue "%d"
    | `-BinaryOperator 0x7f8ce910f588 <col:17, line:10:11> 'int' '+'
    |   |-BinaryOperator 0x7f8ce910f548 <line:20:17, col:19> 'int' '+'
    |   | |-ImplicitCastExpr 0x7f8ce910f518 <col:17> 'int' <LValueToRValue>
    |   | | `-DeclRefExpr 0x7f8ce910f4a8 <col:17> 'int' lvalue Var 0x7f8ce910eeb8 'a' 'int'
    |   | `-ImplicitCastExpr 0x7f8ce910f530 <col:19> 'int' <LValueToRValue>
    |   |   `-DeclRefExpr 0x7f8ce910f4e0 <col:19> 'int' lvalue Var 0x7f8ce910ef58 'b' 'int'
    |   `-IntegerLiteral 0x7f8ce910f568 <line:10:11> 'int' 30
    `-ReturnStmt 0x7f8ce910f680 <line:21:5, col:12>
      `-IntegerLiteral 0x7f8ce910f660 <col:12> 'int' 0
```


> 语法分析只是将完整的句子断开，不管对不对，词法分析负责是否对错。
> 

如果有头文件，需要导入skd才可以。

> clang	-isysroot	/Applications/Xcode.app/Contents/Developer/Platforms/
iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator12.2.sdk(自己sdkpath) 	-fmodules	-fsyntax-only -Xclang -ast-dump main.m

## 3 生成中间代码 IR

做完上边的步骤可以生成中间代码`IR`了，代码生成器会将词法树自顶向下遍历逐步翻译成`LLVM IR`，通过以下命令生成`.ll`文本文件，查看IR代码。

```
clang	-S	-fobjc-arc	-emit-llvm main.m
```

## 4 IR优化
LLVM的优化级别分别是`-O0 -O1 -O2 -O3 -Os(大写字母O)`

```
clang	-Os	-S	-fobjc-arc	-emit-llvm main.m -o main.ll
```

## bitCode
Xcode7 以后开启bitCode 苹果作进一步优化，生成.bc中间代码，我们通过优化后的IR代码生成.bc代码。

```
clang	-emit-llvm	-c	main.ll -o main.bc
```

## 生成汇编代码

最终通过`.ll`或者`.bc`生成汇编代码。


```
clang	-S	-fobjc-arc	main.bc	-o main.s
clang	-S	-fobjc-arc	main.ll	-o main.s

```

生成了汇编代码也可以进行优化

```
clang	-Os	-S	-fobjc-arc	main.m -o main.s
```

## 生成目标文件

目标文件的生成，是汇编机器已汇编代码作为输入，将汇编代码转换成机器代码，最终输出目标文件(`.o`文件)

```
clang	-fmodules	-c	main.s	-o main.o
```

通过nm命令，查看main.o中的符号。

```
$xcrun	nm	-nm	main.o
                 (undefined) external _printf
0000000000000000 (__TEXT,__text) external _test
000000000000000a (__TEXT,__text) external _main
```

`printf`是一个`undefined external`的。

`undefined` 表示在当前文件暂时找不到符号`_printf`


## 生成可执行文件

连接器把编译产生的.o文件和.dylib .a 文件，生成一个mach-o文件。

```
clang main.o -o main
```

查看链接之后的符号

```
                 (undefined) external _printf (from libSystem)
                 (undefined) external dyld_stub_binder (from libSystem)
0000000100000000 (__TEXT,__text) [referenced dynamically] external __mh_execute_header
0000000100003f6d (__TEXT,__text) external _test
0000000100003f77 (__TEXT,__text) external _main
0000000100008008 (__DATA,__data) non-external __dyld_private

```

### [查看 LLVM pdf]([LLVM-课件-](media/LLVM-%E8%AF%BE%E4%BB%B6-.pdf))