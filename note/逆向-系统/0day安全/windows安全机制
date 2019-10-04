# 内存保护

- 使用GS编译技术，在函数返回地址之前加入了Security Cookie，在函数返回前首先检测Security Cookie是否被覆盖，从而把针对操作系统的栈溢出利用变得非常困难

- 增加了对 S.E.H 的安全校验机制，能够有效的挫败绝大多数通过改写 S.E.H 而劫持进程的攻击

- 堆中加入了 Heap Cookie、Safe Unlinking 等一系列的安全机制，为原本就困难重重的堆溢出增加了更多的限制

- DEP（Data Execution Protection，数据执行保护）将数据部分标识为不可执行，组织了堆、栈和数据节中攻击代码的执行

- ASLR（Address space layout randomization，加载地址随机）技术通过对系统关键地址的随机化，使得经典堆溢出手段失效

- SEHOP(Structured Exception Handler Overwrite Protection, S.E.H 覆盖保护)，作为对安全 S.E.H 机制的补充，SEHOP将 S.E.H 的保护提升到系统级别，使得 S.E.H 的保护机制更有效

# GS

## GS原理

Visual Studio 2003及以后版本的Visual Studio默认启用了这个编译选项

![GS](/images/Sat-Aug-17-10:56:39-2019_127770.png "GS")

GS编译选项为每个函数调用增加了一些额外的数据和操作，用来检测栈溢出

- 在所有函数调用发生时，向栈帧内压入一个额外的随机DWORD，这个随机数被称作“canary”，如果使用IDA反汇编的话，可看到IDA会将这个随机数标记为“Security Cookie”

- Security Cookie位于ebp之前，系统还将在 .data 的内存区域放置一个 Security Cookie的副本

![GS保护下的内存布局](/images/Sat-Aug-17-13:23:40-2019_383917.png "GS保护下的内存布局")

- 当栈中发生溢出时，Security Cookie将被首先淹没，之后才是EBP和返回地址

- 在函数返回之前，系统将执行一个额外的安全验证操作，被称作Security check

- 在Security Check过程中，系统将比较栈帧中原先存放的Security Cookie和 .data 中副本的值，如果两者不吻合，说明栈帧中的Security Cookie已被破坏，即栈发生了溢出

- 当检测到栈中发生溢出时，系统将进入异常处理流程，函数不会被正常返回，ret指令也不会执行

![GS保护机制](/images/Sat-Aug-17-13:30:22-2019_666224.png "GS保护机制")

但是额外的数据和操作会导致系统性能下降，为了将性能的影响讲到最低，编译器在编译程序时并不是对所有函数都应用GS，以下情况不会应用GS（新版本vs有变）

- 函数不包含缓冲区

- 函数被定义为具有变量参数列表

- 函数使用无保护的关键字印记

- 函数在第一个语句中包含内嵌汇编代码

- 缓冲区不是8字节类型而且大小不大于4个字节

从Visual Studio 2005 SP1开始引入了一个新的安全标识：

```c
#pragma strict_gs_check
```

通过添加 `#pragma strict_gs_check(on)` 可对任意类型的函数添加Security Cookie，如下所示：

```c
#include "stdafx.h"
#include "string.h"

#pragma strict_gs_check(on)		// 为下面的函数强制启用GS

int vulfunction(char* str){
	char arry[4];
	strcpy(arry, str);
	return 1;
}

int main(){
	char * str = "yeah, i have GS protection";
	vulfunction(str);
	return 0;
}
```

除了在返回地址前添加 Security COokie外，在Visual Studio 2005及后续版本还使用了变量重排技术，在编译时根据局部变量的类型的类型对变量在栈帧中的位置进行调整，及那个字符串变量移动到栈帧的高地址，使其无法通过溢出覆盖到其他的局部变量。

## GS实现细节

- 系统以 .data 节的第一个双字作为 Cookie 的种子，或称原始 Cookie（所有函数的Cookie都用这个DWORD生成）
- 在程序每次运行时 Cookie 的种子都不同，因此种子有很强的随机性
- 在栈帧初始化以后，系统用ESP异或种子，作为当前函数的Cookie，以此作为不哦她那个函数之间的区别，并增加Cookie的随机性
- 在函数返回前，用ESP还原出（异或）Cookie的种子

初始化：

```asm
mov     eax, ___security_cookie
xor     eax, ebp
mov     [ebp+var_4], eax
```

返回时：

```asm
mov     ecx, [ebp+var_4]
xor     ecx, ebp
call    j_@__security_check_cookie@4 ; __security_check_cookie(x)
```

在微软出版的 Writing Secure Code一书中，作者给出了微软内部对GS为产品提供的安全保护的看法：

- 修改栈帧中函数返回地址的经典攻击将被GS机制有效遏制
- 基于改写函数指针的攻击，如 C++ 虚函数的攻击，GS机制仍然很难防御
- 针对异常处理机制的攻击，GS很难防御
- GS是对栈帧的保护机制，因此很难防御堆溢出的攻击

# SafeSEH

在 Windows XP SP2 及后续版本的操作系统中，微软引入了著名的 S.E.H 校验机制 SafeSEH

## SafeSEH原理

在程序调用异常处理函数前，要对调用的异常处理函数进行一系列的有效性校验，当发现异常处理函数不可靠时将终结异常处理函数的调用。SafeSEH实现需要编译器与操作系统更多双重支持

### 编译器

通过启用 /SafeSEH 链接选项可以让编译好后的程序具备SafeSEH功能，该选项在 VS2003 及后续版本中是默认启用的。启用该链接选项后，编译器在编译程序时将程序所有的异常处理函数地址提取出来，编入一张S.E.H表，并将这张表放在程序的映像里，当程序调用异常处理函数时会将函数地址与安全S.E.H表进行匹配，检查调用的异常处理函数是否位于安全的 S.E.H 表中

![SafeSEH](/images/Sat-Aug-17-15:26:07-2019_162589.png "SafeSEH")

...


# DEP

## DEP机制的保护原理

溢出的根源在于现代计算机没有对数据和代码明确区分，DEP（数据执行保护，Data Execution Prevention)就是用来弥补计算机对数据和代码混淆这一缺陷的

DEP的基本原理是将数据所在内存页标识为不可执行，当试图在数据页面上执行指令时，CPU会抛出异常，而不是执行恶意指令

DEP的主要作用是阻止数据页（如默认的堆页、各种堆栈页以及内存池页）执行代码。Windows XP SP2开始提供堆这种技术的支持，根据实现机制不同分为：软件DEP和硬件DEP

前面介绍的 SafeSEH 其实就是软件DEP，这种机制与CPU无关，windows利用软件模拟实现DEP，堆操作系统提供一定的保护。

硬件 DEP 才是真正意义上的DEP，硬件DEP需要CPU的支持，AMD和Intel都对此做了设计，AMD称为 No-Execute page-Protection（NX），Intel称之为 Execute Disable Bit(XD)，两者功能及工作原理在本质上是相同的

操作系统通过内存页的 NX/XD 属性标记，来指明不能从该内存执行代码。为了实现这个功能，需要在内存的页面表（Page Table）中加入一个特殊的标识位（NX/XD）来标识是否允许在该页上执行指令。标识位为0，标识允许这个页面执行指令，为1表示该页面不允许执行指令

![DEP](/images/Sat-Aug-17-17:02:11-2019_111982.png "DEP")

同样的，DEP有着自身的局限性

- 硬件DEP需要CPU的支持，但并不是所有的CPU都提供了硬件DEP的支持，一些比较老的CPU上DEP是无法发挥作用的

- 由于兼容性原因，windows不能对所有进程开启DEP保护，否则可能会出现异常。例如一些第三方的插件DLL，由于无法确认其是否支持DEP，对设计这些DLL的程序不敢贸然开启DEP保护。再有就是ATL7.1或者以前版本的程序需要在数据页面上产生可以执行的代码，这种情况就不能开启DEP保护，否则程序会出现异常

- `/NXCOMPAT`编译选项，或者是 `IMAGE_DLLCHARACTERISTICS_NX_COMPAT` 的设置，只会对 Windows Vista以上的西游有效。在以前的系统中，如Windows XP SP3等，这个设置会被忽略

- 在早期，操作系统对控制DEP状态的API的调用没有任何限制，所有进程都可以调用这些API函数，这就埋下了很大的安全隐患

## 绕过

### Ret2Libc

DEP不允许我们直接到非可执行页执行指令，我们可以在其他可执行的位置找到符合我们要求的指令，让这条指令来替我们工作

- ret2libc实战之利用 ZwSetInformationProcess
- ret2libc实战之利用 VirtualProtect
- ret2libc实战之利用 VirtualAlloc
- 利用可执行内存挑战 DEP

# ASLR

## 内存随机化保护机制的原理

纵观前面介绍的所有漏洞利用方法都有着一个共同的特征：需要一个明确的跳转地址。无论是 JMP ESP等通用跳板指令还是 Ret2Libc 使用的各指令，都要先确定这条指令的入口点。ASLR（Address Space Layout Randomization）技术就是通过加载程序的时候不再使用固定的基址加载，从而干扰shellcode定位的一种保护机制

ASLR的概念在XP时代就已经提出来了，不过XP上的ASLR功能有限，只是对PEB和TEB进行了简单的随机化处理，而对于模块的加载基址没有进行随机化处理，知道Windows Vista出现后，ASLR才真正开始发挥作用

与 SafeSEH 类似，ASLR的实现也需要程序自身的支持和操作系统的双重支持，其中程序支持不是必须的。它包含了 **映像随机化、堆栈随机化、PEB和TEB随机化**

支持ASLR的程序在他的PE头中会设置 IMAGE_DLL_CHARACTERISTICS_DYNAMIC_BASE 标识来说明其支持ASLR，微软从 Visual Studio 205 SP1开始加入了 /dynamicbase 链接选项来帮助我们完成这个任务

![ASLR](/images/Sat-Aug-17-18:32:27-2019_559821.png "ASLR")

### 映像随机化

映像随机化是指PE文件映射到内存时，对其加载的虚拟地址进行随机化处理，这个地址是在系统启动时确定的，系统重启后这个地址会变化。可通过设置注册表：`\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\MoveImages` 的键值来设定映像随机化的工作模式（0：禁用，-1：强制使用，其他值：正常工作模式）

### 堆栈随机化

这项措施在程序运行时随机的选择堆栈的基址，与映像基址随机化不同的是堆栈基址不是在系统启动的时候确定的，而是在打开程序时确定的

### PEB与TEB随机化

PEB和TEB随机化在 Windows XP SP2中就已经引入了，微软在 XP SP2之后不再使用固定的PEB基址 0x7FFDF0000 和 TEB基址 0x7FFDE000，而是使用具有一定随机性的基址，这就增加了攻击PEB中的函数指针的难度

获取当前进程的TEB和PEB的方式很简单，TEB存放在 FS:0 和 FS:[0x18] 处，PEB存放在 TEB偏移 0x30 的位置，如下：

```c
unsigned int teb;
unsigned int peb;

__asm {
	mov eax, FS:[0x18]
	mov teb, eax
	mov eax, dword ptr [eax+0x30]
	mov peb, eax
}
```

## 绕过

### 利用部分覆盖进行定位内存地址

映像随机化只是对映像加载基址的前2个字节做随机化处理，这样做的后果是什么呢？试想一下，无论是函数的返回地址还是异常处理函数的指针，或者虚函数表指针都是要放到堆栈中的，虽然保存的这地址是经过随机处理后的地址，但页仅仅是前两个字节的随机化。借鉴“off by one”的思想，只需要覆盖这个地址的最后一两个字节，就可以在一定范围内控制程序了

## Heap Spray定位内存地址

申请大量内存，在这些内存中放置0x90和shellcode，最后让控制程序转入申请的内存的0x90中即可

# SEHOP

## 原理

随着针对 S.E.H 的攻击日益增多，微软推出了一种新的 S.E.H 保护机制 SEHOP（Structured Exception Handling Overwrite Protection），这是一种比SafeSEH更严厉的保护机制。目前Windows Vista SP1、Windows 7、Windows Server2008、Windows 2008 R2均支持 SEHOP

程序中的各 S.E.H 函数是以单链表的形式存放于栈中的，而这个链表的末端是程序的默认异常处理，它负责处理前面 S.E.H 函数都不能处理的异常，一个经典的 S.E.H 链如下：

![S.E.H链](/images/Sat-Aug-17-23:20:43-2019_412550.png "S.E.H链")

SEHOP的核心任务就是检查这条 S.E.H 链的完整性，在程序转入异常处理前 SEHOP 会检查 S.E.H 链上最后一个异常处理函数是否为系统固定的终极异常处理函数。如果是，则说明该条 S.E.H 链没有被破坏，程序可执行当前的异常处理函数；否侧说明该 S.E.H 链被破坏，可能发生了 S.E.H 覆盖攻击，程序将不会执行当前的异常处理函数

攻击时，将S.E.H结构中的异常处理函数地址覆盖为跳板指令地址，跳板指令根据实际情况进行选择。当程序出现异常时，系统会从 S.E.H 链中取出异常处理函数来处理异常，异常处理函数的指针已被覆盖，程序的流程就会被劫持，在经过一系列跳转后转入shellcode执行

由于覆盖异常处理函数指针时同时覆盖了指向下一异常处理结构的指针，这样 S.E.H 链就会被破坏，从而被 SEHOP 机制检测出

作为对 SafeSEH 强有力的补充，SEHOP检查是在 SafeSEH 的RtlIsValidHandler函数校验前进行的，也就是说利用攻击加载模块之外的地址、堆地址和未启用SafeSEH模块的方法都行不通了，必须要考虑其他处理。理论上还有三条路可以走，分别是：

- 不去攻击 S.E.H，而是攻击函数返回地址或虚函数等
- 利用未启用 SEHOP 的模块
- 伪造 S.E.H 链

出于兼容性考虑，操作系统会根据PE头中的 MajorLinkerVersion 和 MinorLinkerVersion 两个选项来判断是否为程序禁用 SEHOP。例如我们可以将这两个选项分别设置为：0x53 和 0x52来模拟经过 Armadilo 加壳的程序，从而达到禁用 SEHOP 的目的

## 伪造 S.E.H 链表

SEHOP的原理就是检测 S.E.H 链中最后一个异常处理函数指针是否指向了一个固定的终极处理函数，理论上我们溢出时只要伪造一个结构就可以绕过SEHOP了

实际上，真实情况下要实现 S.E.H 的伪造是非常困难的。在系统的ASLR启用时，每次系统重启后，FinalExceptionHandler指向的地址都会变动，大大加大了伪造的难度

# 堆保护

## 堆保护机制的原理

除了上面的一些安全措施外，微软在堆中也增加了一些安全校验操作

- PEB random：微软在 Windows XP SP2之后不再使用固定的PEB机制 0x7ffdf000，而是使用具有一定随机性的 PEB 基址，这点上面已经提到了
- Safe Unlink：微软改写了操作双向链表的代码，在卸载 free list 中的堆块时更加小心
- heap cookie: 与栈中的 security cookie 类似，微软在堆中也引入了cookie，用于检测堆溢出的发生。cookie被布置在堆首部分原堆块的segment table位置，占一个字节大小
	![heap cookie](/images/Sun-Aug-18-00:04:51-2019_259067.png "heap cookie")
- 元数据加密：微软在 Windows Vista 及后续版本的操作系统中开始使用该安全措施。块首中的一些重要数据在保存时会与一个4字节的随机数进行异或运算，在使用这些数据时，需要再来一次异或来运行或还原，这样就不能直接破坏该数据了，以达到保护堆的目的





# 文献

> [Stack Guand:Automatic Adaptive Detection and Prevention of Buffer-Overflow Attacks](https://www.usenix.org/legacy/publications/library/proceedings/sec98/full_papers/cowan/cowan.pdf)
>
> [Defeating the Stack Based Buffer Overflow Prevention Mechanism of Microsoft Windows 2003 Server](http://www.cs.utexas.edu/~shmat/courses/cs378_fall07/litchfield.pdf)
























