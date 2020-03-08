# 快捷键

## 基础操作

- 跳转操作
	- 跳转到之前的位置（热键：Esc）
	- 跳转到之后的位置（热键：Ctrl+Enter）

## 搜索操作

- 文本搜索
  - Search -> Text（热键：ALT+T）
  - 选择Find all accurences查找所有结果，IDA将在一个新的窗口中显示搜索结果
- 二进制搜索
  - Search -> Sequence of Bytes（热键：ALT+B）
  - 搜索特定二进制内容，如已知的字节序列
  - 如要搜索字符串，可用引号将要搜索的字符串引起来
  - 使用 Unicode Strings 选项可以搜索你所搜索的字符串的 Unicode版本
  - 如果没有选中 Case-sensitive 选项，则不区分大小写，在进行十六进制搜索时可能会出现问题（例如搜索0x41也会匹配0x61，他们都是字符`a`）

## 反汇编操作

- 命名
  - 右键要修改名称的位置，选择 Rename 选项（热键：N）
- 注释
  - 常规注释
    - 位于汇编代码行的尾部，默认显示蓝色
    - 右键反汇编窗口的右边缘，选择 Enter comment（热键：冒号':'）
  - 可重复注释
    - 一旦输入，将自动出现在反汇编窗口中的许多位置。默认是蓝色，不过其回显为灰色
    - 右键，选择 Enter repeatable comment （热键：分号';'）
  - 在前注释、在后注释
    - 出现在反汇编行之前或之后的注释，他们是IDA中仅有的不以分号为前缀的注释
  - 函数注释
    - 运行你为函数的反汇编代码清单顶部显示的注释分组
- 数据代码转换
  - Edit -> Undefine（热键：U），可取消函数、代码或数据的定义
  - Edit -> Code（热键：C），反汇编所有字节，直到遇到一个已定义的项目或非法指令
  - Edit -> Data（热键：D），将指令转换为数据
- 字符串
  - Option -> Ascii string style，编辑字符串风格（默认是C风格，以0结尾的字符串）
  - Edit -> Strings，将数据转换为字符串

# 加载 dump binary

有时可能需要从内存中dump出来代码段进行分析

## 直接拖入，选择模式

![加载模式](/images/Mon-Mar--4-10:53:53-2019_796430.png "加载模式")

## 设置基址

Edit -> Segments -> Rebase the whole program

![设置基址](/images/Mon-Mar--4-10:56:03-2019_125768.png "设置基址")

## 设置编译选项

此时仍然无法正常用f5反编译，需要设置一下这个程序的编译选项。否则可能会提示："The target compiler was not specified in Options, Compiler. The decompiler set it based on the input file format"

Option -> Compiler（下面是gcc的示例）

![编译选项](/images/Mon-Mar--4-11:00:02-2019_534533.png)

## 结构体

添加结构体：

Structures界面，按 insert 创建结构体

![添加结构体](/images/Sun-Sep-22-16:56:28-2019_933660.jpg "添加结构体")

在变量处， alt+q 选择要使用的结构体

![选择结构体](/images/Sun-Sep-22-16:58:48-2019_686756.jpg "选择结构体")

