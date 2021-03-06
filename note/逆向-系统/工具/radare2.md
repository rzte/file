### 文件

- r2/radare2
	- 主程序
- rabin2
	- 分析文件（导入、导出、字符串...）
	- 这个工具是用来获取文件信息的，我们可以获取二进制文件的架构、平台、区块、导入导出等信息
	- 可以用`rabin2 -I ./crackme.exe`来获取二进制文件信息
- rax2
	- 数据格式转换
- radiff2
	- 查找不同
- rahash2
	- 从文件块或整个文件创建哈希
	- 这个工具可以让我们快速对文件整体，对部分区块、字符串等执行加密解密和hash等各种操作。比如我们可以在下载一些文件时需要快速比较文件的hash，我们可以用这个工具快速生成各种hash
- rasm2
	- 汇编指令帮助
	- 这个工具可用作汇编反汇编用
- rarun2
	- 用于在不同环境中运行程序的启动器，具有不同的参数，权限，目录和覆盖的默认文件描述符
	- 可用于
		- 破解小程序
		- 模糊测试
		- 测试组件

---
### 简要使用

- 使用`a`命令可以分析程序，`af`分析函数，`aaaa`分析全部，`afl`列出所有函数
- 分析完毕后可通过`vv`命令来粗略浏览一下函数列表
- `pdf`查看汇编
- 除了汇编外，还实现了类似IDA`F5`的功能，可以用`pdc`来查看类C语法的代码
- `VV`来到图形界面，`hjkl`移动图形，输入`$`可以得到更友好的界面
- `s`跳转到一个地址或者标志处
- `/ hello`查找`hello`字符串

---
### 杂

- `px 200 @ esp` ; show 200 hex bytes at esp
- `pc > file.c` ; dump buffer as C byte array to file
- `wx 90 @@ sym.*` ; write a nop on every symbol
- `pd 2000 | grep eax` ; grep cpcode using`eax`register
- `x 20 && s + 3 && x40` ; multiple commands in a single line
- `pd 20~call[0]` ; 查找出20行反汇编中包含call的行的第一列
- `? 0x80480000` ; 打印出`0x80480000`的十进制，八进制，二进制，ascii等信息
- `flag`
	> 设置标记
	> `f`列出所有标记
	> `f flag_name @ offset` 在`offset`处设置标识为`flag_name`
	> `s flag_name` 移动到设置的标记处
	> `f-flag_name` 移除`flag_name`标记
- 打印字符串
	```c
	[0x00000620]> S	//查看段Section
	[00:00] * pa=0x00000000 r-x va=0x00000000 sz=0x0e10 vsz=0x0e10 LOAD0
	[00:01] . pa=0x00000ea8 rw- va=0x00001ea8 sz=0x0160 vsz=0x0164 LOAD1
	[0x00000620]> s 0x00 // 跳转至LOAD0开头
	[0x00000000]> pds 0x0e10 // 打印字符串（psb pds等）
	...
	```
- 比较
	```c
	[0x000008a5]> s 0 // 跳转至0x00处
	[0x00000000]> cf a.out // 与a.out比较
	Compare 256/256 equal bytes (100%) // 相同
	[0x00000000]> s - // 返回
	[0x000008a5]> ps // 这里是字符串 Usage:...
	Usage: %s <password>

	[0x000008a5]> c Usage // 与Usage比较
	Compare 5/5 equal bytes (100%) // 相同
	[0x000008a5]> c usage // 与usage比较
	Compare 4/5 equal bytes (0%)
	0x000008a5 (byte=01)   55 'U'  ->  75 'u'
	```
- 配置
	```c
	[0x000008a5]> ev search // 查找所有与search相关的配置
	search.align = 0 ; Only catch aligned search hits
	search.chunk = 0 ; Chunk size for /+ (default size is asm.bits/8
	search.contiguous = true ; Accept contiguous/adjacent search hits
	```

---
### rabin2

- 文件标识
	`rabin -I /bin/ls`
- 入口点
	`rabin2 -e /bin/ls`
- 导入
	`rabin2 -i /bin/ls`
- 导出（Symbols）
	`rabin2 -s /bin/ls`
- Libraries
	`rabin2 -I /bin/ls`
- 字符串
	`rabin2 -z /bin/ls`
- 程序段
	`rabin2 -Svv /bin/ls`
	`rabin2 -Sr /bin/ls`

---
### asm/dasm

- 简单使用
	```c
	rz$ rasm2 'mov eax, 33'
	b821000000
	rz$ rasm2 -d b821000000 -s att // AT&T语法
	movl $0x21, %eax
	```
- thmub示例
	thmub为16位的arm
	```c
	rz$ rasm2 -a arm -b 16 "adds r0, r2, #1"
	501c
	rz$ rasm2 -a arm -b 16 "adds r0, r2, #2"
	901c
	rz$ rasm2 -a arm -b 16 -d 901c
	adds r0, r2, 2
	```

---
### ragg2/ragg2-cc

- radare自己实现的编译器

---
### 参考

> [radare2](https://www.radare.org)
> [radare2介绍及简单使用](https://cloud.tencent.com/developer/article/1073910)
> [先知社区 Radare2使用全解](https://xz.aliyun.com/t/1514)












