### [cmd 模块](https://wiki.python.org/moin/CmdModule)

cmd模块与`OptParse`不同之处在于cmd模块可以在程序内实现交互式shell

这个模块定义了一个`Cmd`类，在使用时继承它即可

#### return

大多数情况不需要返回值，如果需要退出解释器循环时可返回`True`

---
### action

```python
from cmd import Cmd

class Cli(Cmd, object):
    def do_hello(self, s):
        print("hello~ {}".format(s))
	
	def do_exit(self, s):
		print("exit...")
		return True	# 返回True即可退出循环

if __name__ == '__main__':
	cli = Cli()
	cli.cmdloop()	# 开启交互式
```
如上，如果要响应`hello`命令，只需要定义`do_hello`即可。该方法只有一个额外的参数，这个参数对应`hello`之后全部的字符串。

### help

```python
def help_hello(self):
	print("hello~~~~")
```
```python
(Cmd) help hello
hello~~~~
```
定义`help_xxx`可响应`xxx`命令的帮助


### 空行

默认情况下，输入空行将重复上一个命令。可以通过覆盖`emptyline`方法来更改这个行为
```python
def emptyline(self):
    pass
```

### prompt

可以通过修改`self.prompt`来修改交互式提示符，默认是`(Cmd)`。
如果需要子模块可以如下设置：

```python
    def __init__(self):
        self.prompt = 'cli> '	# 设置默认的交互式提示符
		self.intro = 'this is intro...'	# 进入交互式shell前输出的字符串，也可以用`preloop`方法输出
        Cmd.__init__(self)

    def do_test(self, s):
        self.prompt = self.prompt[:-2] + ": test> "	# 修改提示符
```
```bash
cli> test
cli: test> hello tom
hello~ tom
cli: test>
```

### preloop postloop

在交互开始时，会调用`preloop`方法，并在结束时调用`postloop`方法，可以对其进行重新定义

```python
    def preloop(self):
        print 'Hello'
        super(Cli,self).preloop()
    def postloop(self):
        print 'Goodbye'
        super(Cli,self).postloop()
```

### command processing

处理命令时，会调用如下几个方法：

- `precmd`
	命令line解析之前被调用，**返回将用作onecmd方法的参数字符串**
- `onecmd`
	读取输入，并进行处理。**获取`precmd`的返回值**，并**返回一个布尔值**（True将停止解释器）。其工作方法：提取命令，找到对应的`do_xxx`方法并调用它
- `postcmd`
	命令line解析之后被调用。这个方法有两个参数：**`onecmd`方法的返回值和`precmd`返回的字符串

















