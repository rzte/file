### 工具

- `auditd`
	主要用于查看系统改动的信息，如系统密码修改，用户的新建，主要用于保障系统的安全

- `nm`
	用于列出目标文件中的符号。可以用它来查找程序中各个函数的地址

- `readelf`
	用来显示一个或多个elf文件的格式信息

- `ld`
	链接器：`ld -m elf_i386 shell.o -o shell`

- `nasm`
	汇编器：`nasm -f elf shell.asm -o shell.o`

- `ndisasm`
	反汇编，将机器码转成汇编语言

	```bash
	rz@kali:/tmp$ xxd shell.sh
	00000000: 6a68 682f 2f2f 7368 2f62 696e 89e3 6801  jhh///sh/bin..h.
	00000010: 0101 0181 3424 7269 0101 31c9 516a 0459  ....4$ri..1.Qj.Y
	00000020: 01e1 5189 e131 d26a 0b58 cd80            ..Q..1.j.X..
	rz@kali:/tmp$ ndisasm -b32 shell.sh
	00000000  6A68              push byte +0x68
	00000002  682F2F2F73        push dword 0x732f2f2f
	00000007  682F62696E        push dword 0x6e69622f
	0000000C  89E3              mov ebx,esp
	0000000E  6801010101        push dword 0x1010101
	00000013  81342472690101    xor dword [esp],0x1016972
	0000001A  31C9              xor ecx,ecx
	0000001C  51                push ecx
	0000001D  6A04              push byte +0x4
	0000001F  59                pop ecx
	00000020  01E1              add ecx,esp
	00000022  51                push ecx
	00000023  89E1              mov ecx,esp
	00000025  31D2              xor edx,edx
	00000027  6A0B              push byte +0xb
	00000029  58                pop eax
	0000002A  CD80              int 0x80
	```

- `objdump`
	反汇编目标文件

- `strace`
	跟踪系统调用

### 转储内存

`ulimit -c unlimited`，转储的内核文件可以是任意大小

### 参数传递规则

不同于32位的参数传递，64位一般的函数调用规则如下：

```c
func(1, 2, 3, 4, 5, 6, 7, 8); // 8个参数

// 参数传递如下：
|           0x00001195      6a08           push 8
|           0x00001197      6a07           push 7
|           0x00001199      41b906000000   mov r9d, 6
|           0x0000119f      41b805000000   mov r8d, 5
|           0x000011a5      b904000000     mov ecx, 4
|           0x000011aa      ba03000000     mov edx, 3
|           0x000011af      be02000000     mov esi, 2
|           0x000011b4      bf01000000     mov edi, 1
|           0x000011b9      e877ffffff     call sym.fun
```

可以看到仍然是从右向左依次赋值的。前6个参数赋值给寄存器，之后的参数同32位一样入栈

相当于`func($rdi, $rsi, $rdx, $rcx, $r8, $r9, ...)`

### 动态链接库

**LD_PRELOAD**是个环境变量，用于动态库的加载，其优先级最高。可以用它来进行一些hook操作。

```c
// test.c
// gcc test.c -o test
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int i = 0;
    for (; i < 5; ++i) {
        char *c = (char*)malloc(sizeof(char)); // 正常会malloc成功
        if (NULL == c)
            printf("malloc fails\n");
        else
            printf("malloc ok\n");
    }
    return 0;
}
```

设置自定义的`malloc`

```c
// preload.c
// $ gcc preload.c -shared -fpic -o libpreload.so
#include <stdio.h>
#include <stdlib.h>

void* malloc(size_t size)
{
    printf("%s size: %lu\n", __func__, size);
    return NULL;
}
```

设置`LD_PRELOAD`环境变量：`export LD_PRELOAD=/<path>/libpreload.so`，此时在执行test则malloc返回NULL。

### linux缓冲区溢出保护机制

- 地址随机化
	```c
	rz$ sysctl kernel.randomize_va_space 
	kernel.randomize_va_space = 2
	
	// 关闭内存地址随机化
	rz$ sudo sysctl -w kernel.randomize_va_space=0
	```

- 可执行程序的屏蔽保护机制
	> 对于`Federal`系统，默认会执行可执行程序的屏蔽保护机制，该机制**不允许执行存储在栈中的代码`，`Ubuntu`系统中默认没有采用这种机制
	```c
	rz$ sudo sysctl -w kernel.exec-shield=0
	```
- gcc编译器gs验证码机制

	> gcc编译器专门为防止缓冲区溢出而采取的保护措施，具体方法是gcc首先在缓冲区被写入之前在buff的结束地址之后返回改地址之前放入**随机的gs验证码**，并在缓冲区写入操作结束时校验该值。通常缓冲区会从低地址到高地址覆写内存，所以要覆写返回地址就会覆写gs验证码，被识别产生溢出

	```c
	// 关闭gcc编译器gs验证码机制
	$ gcc auto_overflow.c -fno-stack-protector -o auth_overflow

	// -fstack-protector：
	// 启用堆栈保护，不过只为局部变量中含有 char 数组的函数插入保护代码。
	// -fstack-protector-all：
	// 启用堆栈保护，为所有函数插入保护代码。
	// -fno-stack-protector：
	// 禁用堆栈保护。
	```
- ld链接器堆栈段不可执行机制
	> ld链接器在链接程序时，如果所有的`.o`文件的堆栈段都标记不可执行，那么整个库的堆栈段才会被标记为不可执行
	> 相反，只要有一个`.o`文件的堆栈段被标记为可执行，那么整个库的堆栈段被标记为可执行。检查堆栈段可执行性的方法是：
	- 如果检查ELF库：`readelf -lW a.out | grep GNU_STACK`查看是否有E标记
	- 如果检查生成的`.o`文件：`scanelf -e a.o`查看是否有X标记

	```c
	// 关闭ld链接器不可执行机制的方法是在gcc编译时采用`-z execstack`选项
	$ gcc -fno-stack-protector -z execstack a.c -o a.out
	```

### Linux保护机制

```
rz@kali:/opt/pwn/pwn/0x09/insomnihack CTF 2016-microwave$ checksec microwave
[*] '/opt/pwn/pwn/0x09/insomnihack CTF 2016-microwave/microwave'
    Arch:     amd64-64-little
    RELRO:    Full RELRO				// 设置符号重定向表格为只读或在程序启动时就解析并绑定所有动态符号，从而减少对got的攻击
    Stack:    Canary found				// 栈保护，启用后在函数开始时会往栈里插入些gs信息，并在函数返回时验证gs信息是否被修改，可保护栈溢出（__readfsqword, call    ___stack_chk_fail）
    NX:       NX enabled				// 堆栈代码执行保护
    PIE:      PIE enabled				// 地址随机化
    FORTIFY:  Enabled					// 对一些函数做一些增强保护，比如 printf的第一个参数禁用 %n %8$x等
```

#### KASLR

ASLR（用户空间地址随机化）

KSLR（内核地址随机化）

#### RELRO

RELRO是重定位表只读（Reloction Read Only）的缩写，重定位表即经常提到的got表和plt表。一般来说got表中的数据会在使用时才进行修改，所以got表是可写的，但是这样给字符串漏洞带来一个非常方便的利用方式，即修改got表。比如把got表中的puts改为system的地址，这样在执行put时实际执行的是sytem。而`Full RELRO`意味着该程序的重定位表项全部只读，无论是.got还是.got.plt都无法修改。这样对于一些劫持got表的行为都会被阻止。

### 特殊变量

#### IFS

IFS用来定义分隔符，比如在用`for`遍历时，想要以行为单位，需要定义IFS为换行符

```
IFS='\n' 		# 以字符`\n`为分隔符
IFS=$"\n"		# 与上相同
IFS="\n"		# 与上相同？
IFS=$'\n'		# 这次是真正的换行符
```

### Linux下32位程序执行流程

[![32位ELF执行流程](http://dbp-consulting.com/tutorials/debugging/images/callgraph.png "32位ELF执行流程")](http://dbp-consulting.com/tutorials/debugging/linuxProgramStartup.html "32位ELF执行流程")










