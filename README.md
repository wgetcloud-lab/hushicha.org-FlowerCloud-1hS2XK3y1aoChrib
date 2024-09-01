
## 编译工具链


* 我们写程序的时候用的都是集成开发环境 (IDE: Integrated Development Environment)，集成开发环境可以极大地方便我们程序员编写程序，但是配置起来也相对麻烦。在 Linux 环境下，我们用的是编译工具链，又叫软件开发工具包(SDK:Software Development Kit)。Linux 环境下常见的编译工具链有：GCC 和 Clang，我们使用的是 GCC。


## 编译


### 准备工作


* 查看当前系统是否安装gcc，g\+\+，gdb。
`gcc --version` `g++ --version` `gdb --version`
![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901095825479-438852729.png)
* 未安装可通过命令安装。
`sudo apt update`
`sudo apt install gcc g++ gdb`


### 生成可执行程序/编译过程


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901095148592-381283817.png)



```
Copy

|  | gcc -E hello.c -o hello.i # -E激活预处理，生成预处理后的文件 |
| --- | --- |
|  | gcc -S hello.i -o hello.s # —S激活预处理和编译，生成汇编代码 |
|  | gcc -c hello.s -o hello.o # -c激活预处理、编译和汇编，生成目标文件 |
|  | gcc hello.o -o hello # 执行所有阶段，生成可执行程序 |
|  |  |
|  | gcc -c hello.c # 生成目标文件，gcc会根据文件名hello.c生成hello.o |
|  | gcc hello.o -o hello # 生成可执行程序hello，这里我们需要指定可执行程序的名称，否则会默认生成a.out |
|  | gcc hello.c -o hello # 编译链接，生成可执行程序hello |


```

![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901100707185-518848811.png)
![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901100719770-628278978.png)


### gcc与g\+\+区别


* `gcc` 和 `g++` 都是`GNU(组织)`的一个编译器
* **误区一**：`gcc` 只能编译 c 代码，g\+\+ 只能编译 c\+\+ 代码
	+ 后缀为 `.c` 的，`gcc` 把它当作是 C 程序，而 `g++` 当作是 `c++` 程序
	+ 后缀为 `.cpp` 的，两者都会认为是 `C++` 程序，`C++` 的语法规则更加严谨一些
	+ 编译阶段，`g++` 会调用 `gcc`，对于 `C++` 代码，两者是等价的，但是因为 `gcc` 命令不能自动和 `C++` 程序使用的库联接，所以通常用 `g++` 来完成链接，为了统一起见，干脆编译/链接统统用 `g++` 了，这就给人一种错觉，好像 `cpp` 程序只能用 `g++` 似的
* **误区二**：`gcc` 不会定义 `__cplusplus` 宏，而 `g++` 会
	+ 实际上，这个宏只是标志着编译器将会把代码按 C 还是 C\+\+ 语法来解释
	+ 如上所述，如果后缀为 `.c`，并且采用 `gcc` 编译器，则该宏就是未定义的，否则，就是已定义
* **误区三**：编译只能用 `gcc`，链接只能用 `g++`
	+ 严格来说，这句话不算错误，但是它混淆了概念，应该这样说：编译可以用 `gcc/g++`，而链接可以用 `g++` 或者 `gcc -lstdc++`
	+ `gcc` 命令不能自动和C\+\+程序使用的库联接，所以通常使用 `g++` 来完成链接。但在编译阶段，`g++` 会自动调用 `gcc`，二者等价


## 条件编译


### 预处理指令



```
Copy

|  | 1) #if [#elif] [#else] #endif |
| --- | --- |
|  | 2) #ifdef [#elif] [#else] #endif |
|  | 3) #ifndef [#elif] [#else] #endif |


```

### 指令格式


#### 1\. \#if 指令的格式



```
Copy

|  | #if 常量表达式 |
| --- | --- |
|  | ... |
|  | #endif |


```

当预处理器遇到 \#if 指令时，会计算后面常量表达式的值。如果表达式的值为 0，则\#if 与 \#endif 之间的代码会在预处理阶段删除；否则，\#if 与 \#endif 之间的代码会被保留，交由编译器处理。
\#if 指令常用于调试程序，如下所示：



```
Copy

|  | #define DEBUG 1 |
| --- | --- |
|  | ... |
|  | #if DEBUG |
|  | printf("i = %d\n", i); |
|  | printf("j = %d\n", j); |
|  | #endif |


```

#### 2\. defined运算符


是预处理器的一个运算符，它后面接标识符。如果标识符是一个定义过的宏则值为 1，否则值为 0。defined 运算符常和 \#if 指令一起使用，比如：



```
Copy

|  | #if defined(DEBUG) |
| --- | --- |
|  | ... |
|  | #endif |


```

仅当 DEBUG 被定义成宏时，\#if 和 \#endif 之间的代码会保留到程序中。defined 后面的括号不是必须的，因此可以写成这样：
`#if defined DEBUG`
defined 运算符仅检测 DEBUG 是否有被定义成宏，所以我们不需要给 DEBUG 赋值：
`#define DEBUG`


#### 3\. \#ifdef 的格式



```
Copy

|  | #ifdef 标识符 |
| --- | --- |
|  | ... |
|  | #endif |


```

当标识符有被定义成宏时，保留 \#ifdef 与 \#endif 之间的代码；否则，在预处理阶段删除 \#ifdef 与 \#endif 之间的代码。等价于：



```
Copy

|  | #if defined(标识符) |
| --- | --- |
|  | ... |
|  | #endif |


```

#### 4\. \#ifndef 的格式



```
Copy

|  | #ifndef 标识符 |
| --- | --- |
|  | ... |
|  | #endif |


```

它的作用恰恰与 \#ifdef 相反：当标识符没有被定义成宏时，保留 \#ifndef 与 \#endif之间的代码。


### 作用


#### 1\. 编写可移植的程序


下面的例子会根据 WIN32、MAC\_OS 或 LINUX 是否被定义为宏，而将对应的代码包含到程序中：



```
Copy

|  | #if defined(WIN32) |
| --- | --- |
|  | ... |
|  | #elif defined(MAC_OS) |
|  | ... |
|  | #elif defined(LINUX) |
|  | ... |
|  | #endif |


```

我们可以在程序的开头，定义这三个宏中的一个，从而选择一个特定的操作系统


#### 2\. 为宏提供默认定义


我们可以检测一个宏是否被定义了，如果没有，则提供一个默认的定义：



```
Copy

|  | #ifndef BUFFER_SIZE |
| --- | --- |
|  | #define BUFFER_SIZE 1024 |
|  | #endif |


```

#### 3\. 避免头文件重复包含


多次包含同一个头文件，可能会导致编译错误(比如，头文件中包含类型的定义)。因此，我们应该避免重复包含头文件。使用 \#ifndef 和 \#define 可以轻松实现这一点：



```
Copy

|  | #ifndef __WD_FOO_H |
| --- | --- |
|  | #define __WD_FOO_H |
|  | typedef struct { |
|  | int id; |
|  | char name[25]; |
|  | char gender; |
|  | int chinese; |
|  | int math; |
|  | int english; |
|  | } Student; |
|  | #endif |


```

#### 4\. 临时屏蔽包含注释的代码


我们不能用 /*...*/ "注释掉" 已经包含 /*...*/注释的代码，即**不能嵌套多行注释**。但是我们可以用 \#if 指令来实现：



```
Copy

|  | #if 0 |
| --- | --- |
|  | 包含/*...*/注释的代码 |
|  | #endif |


```

注：这种屏蔽方式，我们称之为**"条件屏蔽"**。


## 库的链接


### 库


* 库文件是计算机上的一类文件，可以简单的把库文件看成一种代码仓库，它提供给使用者一些**可以直接拿来用的变量、函数或类**
* 库是特殊的一种程序，编写库的程序和编写一般的程序区别不大，只是**库不能单独运行**
* 库文件有两种，`静态库`和`动态库（共享库）`。区别是：
	+ **静态库**在程序的链接阶段被复制到了程序中
	+ **动态库**在链接阶段没有被复制到程序中，而是程序在运行时由系统动态加载到内存中供程序调用
* 库的好处：**代码保密** 和**方便部署和分发**


### 静态库的制作


* 规则


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901105759437-1763235358.png)


* 示例：有如下图所示文件(其中每个分文件用于实现四则运算)，将其打包为**静态库**



add.c源代码

```
Copy

|  | #include |
| --- | --- |
|  |  |
|  | int add(int a, int b) |
|  | { |
|  | return a+b; |
|  | } |


```



sub.c源代码

```
Copy

|  | #include |
| --- | --- |
|  |  |
|  | int sub(int a, int b) |
|  | { |
|  | return a-b; |
|  | } |


```



mul.c源代码

```
Copy

|  | #include |
| --- | --- |
|  |  |
|  | int mul(int a, int b) |
|  | { |
|  | return a*b; |
|  | } |


```



div.c源代码

```
Copy

|  | #include |
| --- | --- |
|  |  |
|  | double div(int a, int b) |
|  | { |
|  | return (double)a/b; |
|  | } |


```



head.h头文件

```
Copy

|  | #ifndef _HEAD_H |
| --- | --- |
|  | #define _HEAD_H |
|  | // 加法 |
|  | int add(int a, int b); |
|  | // 减法 |
|  | int sub(int a, int b); |
|  | // 乘法 |
|  | int mul(int a, int b); |
|  | // 除法 |
|  | double div(int a, int b); |
|  | #endif |


```



main.c源文件

```
Copy

|  | #include |
| --- | --- |
|  | #include "head.h" |
|  |  |
|  | int main() |
|  | { |
|  | int a = 20; |
|  | int b = 12; |
|  | printf("a = %d, b = %d\n", a, b); |
|  | printf("a + b = %d\n", add(a, b)); |
|  | printf("a - b = %d\n", subtract(a, b)); |
|  | printf("a * b = %d\n", multiply(a, b)); |
|  | printf("a / b = %f\n", divide(a, b)); |
|  | return 0; |
|  | } |


```


* 查看目录结构 `tree`
![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901113107569-1465571478.png)


1. 生成`.o`文件：`gcc -c 文件名`
![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901113418586-387175285.png)
2. 将`.o`文件打包：`ar rcs libxxx.a xx1.o xx2.o`
![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901113934885-723884812.png)


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901113947289-549207663.png)


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901114108563-344108367.png)


### 静态库的使用


* 需要提供**静态库文件和相应的头文件**
* 编译运行：`gcc main.c -o app -I ./include -l calc -L ./lib`


	+ `-I ./include`：指定头文件目录，如果不指定，出现编译错误
	
	
	![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901114330570-1955493340.png)
	+ `-l calc`：指定静态库名称，如果不指定，出现链接错误
	
	
	![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901114743479-1816950112.png)
	+ `-L ./lib`：指定静态库位置，如果不指定，出现链接错误
	
	
	![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901114959585-8511043.png)
	+ **正确执行**（成功生成`app`可执行文件）
	
	
	![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901115055039-1744201433.png)
	+ 测试程序
	
	
	![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901115130052-71157156.png)


### 动态库的制作


* 规则
![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901115154144-1433881910.png)
* 示例：有如下图所示文件(其中每个分文件用于实现四则运算)，将其打包为**动态库**


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901115247882-9080633.png)


	1. 生成`.o`文件：`gcc -c -fpic 文件名`
	
	
	![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901115647106-315676715.png)
	2. 将`.o`文件打包：`gcc -shared xx1.o xx2.o -o libxxx.so`
	
	
	![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901115642521-899126535.png)


### 动态库的使用


* 需要提供**动态库文件和相应的头文件**
* 定位动态库（**原因见工作原理\-\>如何定位共享库文件**，其中路径为动态库所在位置）


	+ 方法一：修改环境变量，**当前终端生效**，退出当前终端失效
	
	
	
	```
	Copy
	
	|  | export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/u/Desktop/Linux/calc/lib |
	| --- | --- |
	
	
	```
	+ 方法二：修改环境变量，用户级别永久配置
	
	
	
	```
	Copy
	
	|  | # 修改~/.bashrc |
	| --- | --- |
	|  | vim ~/.bashrc |
	|  |  |
	|  | # 在~/.bashrc中添加下行，保存退出 |
	|  | export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/u/Desktop/Linux/calc/lib |
	|  |  |
	|  | # 使修改生效 |
	|  | source ~/.bashrc |
	
	
	```
	+ 方法三：修改环境变量，系统级别永久配置
	
	
	
	```
	Copy
	
	|  | # 修改/etc/profile |
	| --- | --- |
	|  | sudo vim /etc/profile |
	|  |  |
	|  | # 在~/.bashrc中添加下行，保存退出 |
	|  | export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/u/Desktop/Linux/calc/lib |
	|  |  |
	|  | # 使修改生效 |
	|  | source /etc/profile |
	
	
	```
	+ 方法四：修改`/etc/ld.so.cache文件列表`
	
	
	
	```
	Copy
	
	|  | # 修改/etc/ld.so.conf |
	| --- | --- |
	|  | sudo vim /etc/ld.so.conf |
	|  |  |
	|  | # 在/etc/ld.so.conf中添加下行，保存退出 |
	|  | /home/u/Desktop/Linux/calc/lib |
	|  |  |
	|  | # 更新配置 |
	|  | sudo ldconfig |
	
	
	```
* 有如下结构文件，其中`main.c`测试文件


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901120334264-1270400348.png)
* 配置环境变量
![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901120645328-1325320544.png)
* 编译运行：`gcc main.c -o app -I ./include -l calc -L ./lib`


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901120831564-1290364630.png)
* 测试程序


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901120848130-1547571108.png)
* 如果不将动态库文件绝对路径加入环境变量，则会出现以下错误


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901120307507-1816061591.png)


### 工作原理


* 静态库：`GCC` 进行链接时，会把静态库中代码打包到可执行程序中
* 动态库：`GCC` 进行链接时，动态库的代码不会被打包到可执行程序中
* 程序启动之后，动态库会被动态加载到内存中，通过 `ldd （list dynamic dependencies）`命令检查动态库依赖关系


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901120931284-615879337.png)
* 如何定位共享库文件呢？


	+ 当系统加载可执行代码时候，能够知道其所依赖的库的名字，但是还需要知道**绝对路径**。此时就需要系统的动态载入器来获取该绝对路径
	+ 对于`elf格式`的可执行程序，是由`ld-linux.so`来完成的，它先后搜索`elf文件`的 `DT_RPATH`段 \=\> `环境变量LD_LIBRARY_PATH` \=\> `/etc/ld.so.cache文件列表` \=\> `/lib/`，`usr/lib`目录找到库文件后将其载入内存


### 静态库和动态库的对比


#### 程序编译成可执行程序的过程


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901121142445-1486488581.png)


#### 静态库制作过程


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901121149917-1575345620.png)


#### 动态库制作过程


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901121157521-1566388612.png)


#### 静态库的优缺点


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901121455559-1513124605.png)


#### 动态库的优缺点


![image](https://img2024.cnblogs.com/blog/3400871/202409/3400871-20240901121458146-1485224075.png)


 本博客参考[蓝猫机场](https://fenfang.org)。转载请注明出处！
