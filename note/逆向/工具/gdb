# gdb-peda

[https://github.com/longld/peda](https://github.com/longld/peda)

## Installation

```bash
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit
echo "DONE! debug your program with gdb and enjoy"
```

# GDB 常用调试命令

设置Inter格式的汇编可在`~/.gdbinit`文件中添加`set disassembly-flavor intel`或者在gdb运行时执行此命令。

查看（设置）当前gdb支持的架构，如果没有支持的则需要从tool-chain中获取(见[stackoverflow](https://stackoverflow.com/questions/24669864/gdb-remote-debug-cant-stop-the-thread# "stackoverflow"))

```c
(gdb) set architecture
Requires an argument. Valid arguments are i386, i386:x86-64, i386:x64-32, i8086, i386:intel, i386:x86-64:intel, i386:x64-32:intel, i386:nacl, i386:x86-64:nacl, i386:x64-32:nacl, auto
```

## 信号

- 捕获信号（handle）
	```bash
	gdb-peda$ handle SIGALRM nostop nopass print # 闹钟信号（alarm），不暂停程序、打印、不让程序接收到
	Signal        Stop	Print	Pass to program	Description
	SIGALRM       No	Yes	No		Alarm clock
	```
- 发送信号（signal）
	```bash
	gdb-peda$ signal SIGALRM # 向程序发送闹钟信号
	Continuing with signal SIGALRM.

	Program terminated with signal SIGALRM, Alarm clock.
	The program no longer exists.
	Warning: not running or target is remote
	```
- 查看信号（info signals）
	```bash
	gdb-peda$ info signals
	Signal        Stop	Print	Pass to program	Description

	SIGHUP        Yes	Yes	Yes		Hangup
	SIGINT        Yes	Yes	No		Interrupt
	...
	```
---
## 内存相关

**gcore**： Generate a core dump of a running program with process ID pid。gcore是linux的命令，但是可以在gdb里直接输入`gcore`来dump调试程序的内存

dump -- Dump target code/data to a local file
dump binary -- Write target code/data to a raw binary file
dump binary memory -- Write contents of memory to a raw binary file
dump binary value -- Write the value of an expression to a raw binary file
dump ihex -- Write target code/data to an intel hex file
dump ihex memory -- Write contents of memory to an ihex file
dump ihex value -- Write the value of an expression to an ihex file
dump memory -- Write contents of memory to a raw binary file
dump srec -- Write target code/data to an srec file
dump srec memory -- Write contents of memory to an srec file
dump srec value -- Write the value of an expression to an srec file
dump tekhex -- Write target code/data to a tekhex file
dump tekhex memory -- Write contents of memory to a tekhex file
dump tekhex value -- Write the value of an expression to a tekhex file
dump value -- Write the value of an expression to a raw binary file
dump verilog -- Write target code/data to a verilog hex file
dump verilog memory -- Write contents of memory to a verilog hex file
dump verilog value -- Write the value of an expression to a verilog hex file

---

## 源码

在调试过程中查看源代码是必须的。list (缩写 l) 支持多种方式查看源码。

(gdb) l # 显示源代码
```
3    int test(int a, int b)
4    {
5        int c = a + b;
6        return c;
7    }
8
9    int main(int argc, char* argv[])
10    {
11        int a = 0x1000;
```

(gdb) l # 继续显示
```
12        int b = 0x2000;
13        int c = test(a, b);
14        printf("%d/n", c);
15
16        printf("Hello, World!/n");
17        return 0;
18    }
```

(gdb) l 3, 10 # 显示特定范围的源代码
```
3    int test(int a, int b)
4    {
5        int c = a + b;
6        return c;
7    }
8
9    int main(int argc, char* argv[])
10    {
```

(gdb) l main # 显示特定函数源代码
```
5        int c = a + b;
6        return c;
7    }
8
9    int main(int argc, char* argv[])
10    {
11        int a = 0x1000;
12        int b = 0x2000;
13        int c = test(a, b);
14        printf("%d/n", c);
```

可以用如下命令修改源代码显示行数。
```
(gdb) set listsize 50
```

## 断点

可以使用函数名或者源代码行号设置断点。

```
(gdb) b main # 设置函数断点

Breakpoint 1 at 0x804841b: file hello.c, line 11.
(gdb) b 13 # 设置源代码行断点

Breakpoint 2 at 0x8048429: file hello.c, line 13.
(gdb) b # 将下一行设置为断点 (循环、递归等调试很有用)

Breakpoint 5 at 0x8048422: file hello.c, line 12.
(gdb) tbreak main # 设置临时断点 (中断后失效)

Breakpoint 1 at 0x804841b: file hello.c, line 11.
(gdb) info breakpoints # 查看所有断点

Num     Type           Disp Enb Address    What
2       breakpoint     keep y   0x0804841b in main at hello.c:11
3       breakpoint     keep y   0x080483fa in test at hello.c:5
(gdb) d 3 # delete: 删除断点 (还可以用范围 "d 1-3"，无参数时删除全部断点)

(gdb) disable 2 # 禁用断点 (还可以用范围 "disable 1-3")

(gdb) enable 2 # 启用断点 (还可以用范围 "enable 1-3")

(gdb) ignore 2 1 # 忽略 2 号中断 1 次
```

当然少不了条件式中断。
```
(gdb) b test if a == 10

Breakpoint 4 at 0x80483fa: file hello.c, line 5.
(gdb) info breakpoints

Num     Type           Disp Enb Address    What
4       breakpoint     keep y   0x080483fa in test at hello.c:5
        stop only if a == 10
```

可以用 condition 修改条件，注意表达式不包含 "if"。
```
(gdb) condition 4 a == 30
(gdb) info breakpoints

Num     Type           Disp Enb Address    What
2       breakpoint     keep y   0x0804841b in main at hello.c:11
        ignore next 1 hits
4       breakpoint     keep y   0x080483fa in test at hello.c:5
        stop only if a == 30
```

## 执行

通常情况下，我们会先设置 main 入口断点。
```
(gdb) b main

Breakpoint 1 at 0x804841b: file hello.c, line 11.
(gdb) r # 开始执行 (Run)

Starting program: /home/yuhen/Learn.c/hello
Breakpoint 1, main () at hello.c:11
11              int a = 0x1000;
(gdb) n # 单步执行 (不跟踪到函数内部, Step Over)

12              int b = 0x2000;
(gdb) n

13              int c = test(a, b);
(gdb) s # 单步执行 (跟踪到函数内部, Step In)

test (a=4096, b=8192) at hello.c:5
5               int c = a + b;
(gdb) finish # 继续执行直到当前函数结束 (Step Out)

Run till exit from #0  test (a=4096, b=8192) at hello.c:5
0x0804843b in main () at hello.c:13
13              int c = test(a, b);
Value returned is $1 = 12288
(gdb) c # Continue: 继续执行，直到下一个断点。

Continuing.
12288
Hello, World!

Program exited normally.
```

## 堆栈

查看调用堆栈(call stack)无疑是调试过程中非常重要的事情。
```
(gdb) where # 查看调用堆栈 (相同作用的命令还有 info s 和 bt)

#0  test (a=4096, b=8192) at hello.c:5
#1  0x0804843b in main () at hello.c:13
(gdb) frame # 查看当前堆栈帧，还可显示当前代码

#0  test (a=4096, b=8192) at hello.c:5
5               int c = a + b;
(gdb) info frame # 获取当前堆栈帧更详细的信息

Stack level 0, frame at 0xbfad3290:
 eip = 0x80483fa in test (hello.c:5); saved eip 0x804843b
 called by frame at 0xbfad32c0
 source language c.
 Arglist at 0xbfad3288, args: a=4096, b=8192
 Locals at 0xbfad3288, Previous frame's sp is 0xbfad3290
 Saved registers:
  ebp at 0xbfad3288, eip at 0xbfad328c

可以用 frame 修改当前堆栈帧，然后查看其详细信息。

(gdb) frame 1

#1  0x0804843b in main () at hello.c:13
13              int c = test(a, b);
(gdb) info frame

Stack level 1, frame at 0xbfad32c0:
 eip = 0x804843b in main (hello.c:13); saved eip 0xb7e59775
 caller of frame at 0xbfad3290
 source language c.
 Arglist at 0xbfad32b8, args:
 Locals at 0xbfad32b8, Previous frame's sp at 0xbfad32b4
 Saved registers:
  ebp at 0xbfad32b8, eip at 0xbfad32bc
```

## 变量和参数

```
(gdb) info locals # 显示局部变量

c = 0
(gdb) info args # 显示函数参数(自变量)

a = 4096
b = 8192

我们同样可以切换 frame，然后查看不同堆栈帧的信息。

(gdb) p a # print 命令可显示局部变量和参数值

$2 = 4096
(gdb) p/x a # 十六进制输出 (d: 十进制; u: 十进制无符号; x: 十六进制; o: 八进制; t: 二进制; c: 字符)

$10 = 0x1000
(gdb) p a + b # 还可以进行表达式计算

$5 = 12288

set variable 可用来修改变量值。

(gdb) set variable a=100

(gdb) info args
a = 100
b = 8192
```

## 内存及寄存器

x 命令可以显示指定地址的内存数据。

格式: x/nfu [address]
n: 显示内存单位(组或者行)。
f: 格式 (除了 print 格式外，还有 字符串s 和 汇编 i)。
u: 内存单位 (b: 1字节; h: 2字节; w: 4字节; g: 8字节)。
```
(gdb) x/8w 0x0804843b # 按四字节(w)显示 8 组内存数据

0x804843b <main+49>:    0x8bf04589      0x4489f045      0x04c70424      0x04853024
0x804844b <main+65>:    0xfecbe808      0x04c7ffff      0x04853424      0xfecfe808
(gdb) x/8i 0x0804843b # 显示 8 行汇编指令

0x804843b <main+49>:    mov    DWORD PTR [ebp-0x10],eax
0x804843e <main+52>:    mov    eax,DWORD PTR [ebp-0x10]
0x8048441 <main+55>:    mov    DWORD PTR [esp+0x4],eax
0x8048445 <main+59>:    mov    DWORD PTR [esp],0x8048530
0x804844c <main+66>:    call   0x804831c <printf@plt>
0x8048451 <main+71>:    mov    DWORD PTR [esp],0x8048534
0x8048458 <main+78>:    call   0x804832c <puts@plt>
0x804845d <main+83>:    mov    eax,0x0
(gdb) x/s 0x08048530 # 显示字符串

0x8048530:       "%d/n"

除了通过 "info frame" 查看寄存器值外，还可以用如下指令。

(gdb) info registers # 显示所有寄存器数据

eax            0x1000   4096
ecx            0xbfad32d0       -1079168304
edx            0x1      1
ebx            0xb7fa1ff4       -1208344588
esp            0xbfad3278       0xbfad3278
ebp            0xbfad3288       0xbfad3288
esi            0x8048480        134513792
edi            0x8048340        134513472
eip            0x80483fa        0x80483fa <test+6>
eflags         0x286    [ PF SF IF ]
cs             0x73     115
ss             0x7b     123
ds             0x7b     123
es             0x7b     123
fs             0x0      0
gs             0x33     51
(gdb) p $eax # 显示单个寄存器数据

$11 = 4096
```

## 反汇编

```
我对 AT&T 汇编不是很熟悉，还是设置成 intel 格式的好。

(gdb) set disassembly-flavor intel # 设置反汇编格式
(gdb) disass main # 反汇编函数

Dump of assembler code for function main:
0x0804840a <main+0>:    lea    ecx,[esp+0x4]
0x0804840e <main+4>:    and    esp,0xfffffff0
0x08048411 <main+7>:    push   DWORD PTR [ecx-0x4]
0x08048414 <main+10>:   push   ebp
0x08048415 <main+11>:   mov    ebp,esp
0x08048417 <main+13>:   push   ecx
0x08048418 <main+14>:   sub    esp,0x24
...
0x08048458 <main+78>:   call   0x804832c <puts@plt>
0x0804845d <main+83>:   mov    eax,0x0
0x08048462 <main+88>:   add    esp,0x24
0x08048465 <main+91>:   pop    ecx
0x08048466 <main+92>:   pop    ebp
0x08048467 <main+93>:   lea    esp,[ecx-0x4]
0x0804846a <main+96>:   ret
End of assembler dump.
```

可以用 `b *address` 设置汇编断点，然后用 "si" 和 "ni" 进行汇编级单步执行，这对于分析指针和寻址非常有用。

## 进程

查看进程相关信息，尤其是 maps 内存数据是非常有用的。

```
(gdb) help info proc stat

Show /proc process information about any running process.
Specify any process id, or use the program being debugged by default.
Specify any of the following keywords for detailed info:

  mappings -- list of mapped memory regions.
  stat     -- list a bunch of random process info.
  status   -- list a different bunch of random process info.
  all      -- list all available /proc info.
(gdb) info proc mappings # 相当于 cat /proc/{pid}/maps

process 22561
cmdline = '/home/yuhen/Learn.c/hello'
cwd = '/home/yuhen/Learn.c'
exe = '/home/yuhen/Learn.c/hello'
Mapped address spaces:

        Start Addr   End Addr       Size     Offset objfile
         0x8048000  0x8049000     0x1000          0       /home/yuhen/Learn.c/hello
         0x8049000  0x804a000     0x1000          0       /home/yuhen/Learn.c/hello
         0x804a000  0x804b000     0x1000     0x1000       /home/yuhen/Learn.c/hello
         0x8a33000  0x8a54000    0x21000  0x8a33000           [heap]
        0xb7565000 0xb7f67000   0xa02000 0xb7565000
        0xb7f67000 0xb80c3000   0x15c000          0      /lib/tls/i686/cmov/libc-2.9.so
        0xb80c3000 0xb80c4000     0x1000   0x15c000      /lib/tls/i686/cmov/libc-2.9.so
        0xb80c4000 0xb80c6000     0x2000   0x15c000      /lib/tls/i686/cmov/libc-2.9.so
        0xb80c6000 0xb80c7000     0x1000   0x15e000      /lib/tls/i686/cmov/libc-2.9.so
        0xb80c7000 0xb80ca000     0x3000 0xb80c7000
        0xb80d7000 0xb80d9000     0x2000 0xb80d7000
        0xb80d9000 0xb80da000     0x1000 0xb80d9000           [vdso]
        0xb80da000 0xb80f6000    0x1c000          0      /lib/ld-2.9.so
        0xb80f6000 0xb80f7000     0x1000    0x1b000      /lib/ld-2.9.so
        0xb80f7000 0xb80f8000     0x1000    0x1c000      /lib/ld-2.9.so
        0xbfee2000 0xbfef7000    0x15000 0xbffeb000           [stack]
```

## 线程

可以在 pthread_create 处设置断点，当线程创建时会生成提示信息。
```
(gdb) c

Continuing.
[New Thread 0xb7e78b70 (LWP 2933)]
(gdb) info threads # 查看所有线程列表

* 2 Thread 0xb7e78b70 (LWP 2933)  test (arg=0x804b008) at main.c:24
  1 Thread 0xb7e796c0 (LWP 2932)  0xb7fe2430 in __kernel_vsyscall ()
(gdb) where # 显示当前线程调用堆栈

#0  test (arg=0x804b008) at main.c:24
#1  0xb7fc580e in start_thread (arg=0xb7e78b70) at pthread_create.c:300
#2  0xb7f478de in clone () at ../sysdeps/unix/sysv/linux/i386/clone.S:130
(gdb) thread 1 # 切换线程

[Switching to thread 1 (Thread 0xb7e796c0 (LWP 2932))]#0  0xb7fe2430 in __kernel_vsyscall ()
(gdb) where # 查看切换后线程调用堆栈

#0  0xb7fe2430 in __kernel_vsyscall ()
#1  0xb7fc694d in pthread_join (threadid=3085405040, thread_return=0xbffff744) at pthread_join.c:89
#2  0x08048828 in main (argc=1, argv=0xbffff804) at main.c:36
```

## 其他

调试**子进程**
```
(gdb) set follow-fork-mode child
```

临时进入 Shell 执行命令，Exit 返回。
```
(gdb) shell
```

调试时直接调用函数。
```
(gdb) call test("abc")
```

使用 "--tui" 参数，可以在终端窗口上部显示一个源代码查看窗。
```
$ gdb --tui hello
```

查看命令帮助。
```
(gdb) help b
```

> [GDB调试演示过程](https://blog.csdn.net/cout_sev/article/details/38424993 "GDB调试演示过程")

---

---

(gdb) finish：退出函数

(gdb) set args 参数:指定运行时的参数

(gdb) show args：查看设置好的参数

(gdb) show paths:查看程序运行路径；

set environment varname [=value] 设置环境变量。如：set env USER=hchen；

show environment [varname] 查看环境变量；

(gdb)info program： 来查看程序的是否在运行，进程号，被暂停的原因。

(gdb)clear 行号n：清除第n行的断点

(gdb)delete 断点号n：删除第n个断点

(gdb)disable 断点号n：暂停第n个断点

(gdb)enable 断点号n：开启第n个断点

(gdb)step：单步调试如果有函数调用，则进入函数；与命令n不同，n是不进入调用的函数的

step：简记为 s ，单步跟踪程序，当遇到函数调用时，则进入此函数体（一般只进入用户自定义函数）。

next：简记为 n，单步跟踪程序，当遇到函数调用时，也不进入此函数体；此命令同 step 的主要区别是，step 遇到用户自定义的函数，将步进到函数中去运行，而 next 则直接调用函数，不会进入到函数体内。

until：当你厌倦了在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体。

finish： 运行程序，直到当前函数完成返回，并打印函数返回时的堆栈地址和返回值及参数值等信息。

stepi或nexti：单步跟踪一些机器指令。

**display 表达式：在单步运行时将非常有用，使用display命令设置一个表达式后，它将在每次单步进行指令后，紧接着输出被设置的表达式及值。如： display a**

watch 表达式：设置一个监视点，一旦被监视的“表达式”的值改变，gdb将强行终止正在被调试的程序。如： watch a

kill：将强行终止当前正在调试的程序

layout：用于分割窗口，可以一边查看代码，一边测试：

layout src：显示源代码窗口

layout asm：显示反汇编窗口

layout regs：显示源代码/反汇编和CPU寄存器窗口

layout split：显示源代码和反汇编窗口






