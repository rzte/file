### 系统调用

在目录`/usr/include/asm/unistd.h`下可查看系统调用号

```c
# define __NR_execve 11
# define __NR_setuid 23
```

---
### 陷入内核

通过`man syscall`，可以查看不同架构下陷入内核的方法和参数传递的方式

- 进入内核
	```c
       ...
       arm64      svc #0                  x8         x0      -
       blackfin   excpt 0x0             P0         R0      -
       i386         int $0x80              eax       eax     -
       ia64         break 0x100000     r15       r8      r10
	   ...
       x86-64      syscall                 rax        rax     -
       x32           syscall                 rax        rax     -
       xtensa      syscall                  a2         a2      -
	```
- 参数传递
	```c
       arch/ABI      arg1   arg2   arg3  arg4    arg5  arg6  arg7  Notes
       ─────────────────────────────────────────
       ...
       arm64           x0      x1      x2     x3     x4      x5     -
       blackfin        R0      R1      R2     R3    R4      R5     -
       i386              ebx    ecx     edx   esi    edi     ebp   -
       ia64              out0   out1   out2  out3  out4  out5  -
       ...
       x86-64        rdi   rsi   rdx   r10   r8    r9    -
       x32           rdi   rsi   rdx   r10   r8    r9    -
       xtensa        a6    a3    a4    a5    a8    a9    -
	```