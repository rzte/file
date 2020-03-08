### APK动态分析

#### [AndBug](https://github.com/swdunlop/AndBug)

AndBug可用来在不需要源代码的情况下调试android程序，依赖于python

- 配置
	```bash
	$ git clone https://github.com/swdunlop/AndBug.git
	$ export PYTHONPATH=`pwd`/lib
	$ make
	```
- 运行
	在adb连接android目标机后，可根据包名或进程号开启AndBug调试
	```bash
	$ adb shell pm list packages | grep example # 列出目标包名
	package:com.example.rz.myapplication
	$ ./andbug shell -p com.example.rz.myapplication # 开始调试
	```

### Hook技术

#### Hook原理

Hook技术无论对安全软件还是恶意软件都是一项十分关键的技术，其本质就是劫持函数调用。但是用户处于Linux用户态，每个进程都有自己的进程空间，所以必须先注入到所要Hook的进程空间，修改器内存中的进程代码，替换其过程表中的符号地址。

在Android中一般是通过**ptrace**函数附加进程，然后向远程进程注入so库，从而达到监控以及远程进程关键函数挂钩。**Hook技术的难点不在于Hook技术，而是如何找到函数的入口点、替换函数，这就涉及到了理解函数的连接与加载机制。**

就Android的开发来说，Android本身提供了两种开发方式，基于Android SDK的Java语言开发，基于Android NDK的native C/C++语言开发。所以我们在讨论Hook的时候必须在两个层面上来讨论。**对于Native层来说Hook的难点在于理解ELF文件与学习ELF文件。对于Java层来说，Hook就需要了解虚拟机的特性与Java反射的使用。**

#### Hook工作流程

Hook的原理就是改变目标函数的指向，可以将问题细分为两个，一个是**如何注入代码**，另一个是**如何注入动态链接库**

注入代码需要解决两个问题：

- 需要注入的代码要存放到哪里

- 如何注入代码

注入动态链接库也需要解决两个问题：

- 我们不能只在自己的进程载入动态链接库，如何使进程附着上目标进程

- 如何让目标进程调用我们的动态链接库函数

对于**进程的附着**，Android的内核中有一个函数叫**ptrace**，能动态地attach（跟踪一个目标进程）、detach（结束跟踪一个目标进程）、peektext（获取内存字节）、poketext（向内存中写入地址）等，可满足我们的需求。

Android中另一个内核函数**dlopen**，能够以指定模式打开指定的动态链接库文件。对于程序的指向流程，我们可以调用ptrace让PC指向LR堆栈。最后调用，对目标进程调用dlopen则能将我们希望注入的动态库注入到目标进程中。

对于**代码的注入**（Hook API），我们可以使用**mmap**函数分配一段临时的内存来完成代码的存放。对于目标进程中的mmap函数地址的寻找与Hook API函数地址的寻找都需要通过目标进程的虚拟地址空间解析与ELF文件解析来完成。具体算法如下：

- 通过读取`/proc/<PID>/maps`文件找到链接库的基地址
- 读取动态库，解析ELF文件，找到符号（需要对ELF文件格式的深入理解）
- 计算目标函数的绝对地址

目标进程函数绝对地址=函数地址+动态库基地址

上面说了这么多，向目标进程注入代码总结后的步骤分为以下几步：

1. 用`ptrace`函数attach上目标进程
2. 发现装载共享库so函数
3. 让目标进程的执行流程跳转到注入的代码执行
4. 使用`ptrace`函数的detach释放目标进程

![Ptrace hook](/images/Wed-Aug-22-07:55:10-2018_837938.png "Ptrace hook")

```c
int ptrace(
    int request, // 请求ptrace执行的操作。决定了ptrace会执行什么操作，常见的有跟踪指定的进程（PTRACE_ATTACH）、结束跟踪指定进程（PTRACE_DETACH）等
    int pid, // 目标进程的pid
    int addr, // 目标进程的地址值
    int data // 作用的数据
);
```

#### Hook的种类

- Java层 API hook
	通过Android平台的虚拟机注入与java反射的方式来改变Android虚拟机调用函数的方式（ClassLoader），从而达到Java函数重定向的目的。因为是根据Java的反射机制来实现的，所以会有一些局限性，比如无法反射调用native函数，基本类型的静态常量无法反射修改等。
- Native层so库hook
	主要针对NDK开发出来的so库文件的函数重定向。也包括对Android操作系统底层的Linux函数重定向。如使用so文件（ELF格式文件）中的**全局偏移表GOT**或**符号表SYM**进行修改从而达到函数重定向。一般称为**GOT Hook**和**SYM Hook**。针对其中的inline函数（内联函数）的Hook成为**inline Hook**
- 全局hook
	在Android中应用程序进程都是由Zygote进程孵化出来的，而Zygote进程是由Init进程启动的。Zygote进程在启动时会创建一个Dalvik虚拟机实例，每当它孵化一个新的应用程序进程时搜会将这个Dalvik虚拟机实例复制到新的应用程序进程里去。从而使每一个应用程序进程都由一个独立的Dalvik虚拟机实例。如果对Zygote进程Hook，则能达到对系统上所有应用程序进程Hook，也就是一个全局Hook

	对应的`app_process`是zygote进程启动一个应用程序的入口，常见的Hook框架Xposed、Cydiasubstrate就是通过替换`app_process`来完成全局Hook的

#### Hook常用工具

- [Xposed](http://repo.xposed.info/)
	Hook Java函数
- [Cydiasubstrate](http://www.cydiasubstrate.com/)
	Android、IOS，Cydia Substrate是一个代码修改平台，可修改任何主进程的代码，不管是Java还是C/C++（native）
- [ADBI](https://github.com/crmulliner/adbi)
- [DDI](https://github.com/crmulliner/ddi)
	Android动态二进制指令包，命令行工具

#### Hook检测

对于应用程序自身的检测，只需要读取对应进程的虚拟地址空间目录`/proc/<PID>/maps`文件，判断当前进程空间中载入的代码库文件是否存在自己的白名单中即可判断自身程序是否被hook。不过对于zygote进程来说如果没有root权限则无法判断释放被hook

- Java层的Hook检测
	/proc/<PID>/maps文件中可以查看进程的虚拟机地址空间是如何使用的
	
	查看地址空间中对应的dex文件有哪些是非系统应用提供的，即过滤出`/data@app`（系统应用是`/system@app`)
	```c
	cat /proc/<PID>/maps | grep /data/dalvik-cache/data@app
	```
- ative层Hook检测
	与之前一样，native的hook检测也是查看maps
	
	查看<PID>进程的虚拟地址空间加载了哪些第三方库文件，可以过滤`/data/data/`
	```c
	cat /proc/<PID>/maps | grep /data/data/
	```

#### Hook修复

因为所有的第三方库都是通过dlopen后注入的方式附加到应用程序进程中的，很容易想到用dlclose将其中的第三方函数挨个卸载关闭。可以扫描`/proc/<PID>/maps`下所有so，将不正常的都卸载关闭即可。

不过这样做仍可能会残留一些没被卸载，dlclose用于关闭指定句柄的动态链接库，只有当此动态链接库的使用计数为0时，才会真正被系统卸载。也就是说，在我们要手动卸载动态链接库之前如果系统已经保持对其的引用的话，是无法卸载的。

Hook框架的动态库什么时候被加载的不能得知，对本包的动态链接库卸载也要实时监测去卸载。而且就算卸载也不能完全地保证系统没有对相关函数的引用，达到卸载干净的目的。所以，对于Hook后的应用程序修复暂时是无法解决的。










