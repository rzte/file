### 0x01 IDA调试dex

- **解压apk，将classes.dex拖进IDA**

- **加载完毕后，点击`Debugger -> Debugger setup`，选中`suspend on process entry point`（进程入口挂起）。点击`set specific options`，设置好adb、package name、Activity**

	![Debugger setup](/images/Tue-Aug-21-07:36:18-2018_459297.png "Debugger setup")

- **在IDA点击`Debugger -> Process Options`，设置端口**

	![Process Option](/images/Tue-Aug-21-07:39:22-2018_602949.png "Process Option")

- **F9或点击开始按钮即可调试，此时可开始调试**
	![debug](/images/Tue-Aug-21-07:45:57-2018_766053.png "debug")

### 0x02 IDA调试so

可用IDA自带的**dbgsrv/android_server**对app进行调试

```java
λ adb push android_server64 /data/local/tmp/android_server // 我用的一加五，aarch64，所以用的android_server64，下面的ida也用的64位
λ adb shell chmod 777 /data/local/tmp/android_server
λ adb shell
$ su
# /data/local/tmp/android_server
IDA Android 32-bit remote debug server(ST) v1.19. Hex-Rays (c) 2004-2015
Listening on port #23946...

λ adb forward tcp:23946 tcp:23946 // 端口转发
```

- **启动IDA，打开`debugger -> attach -> remote Armlinux/android debugger`，填写Hostname、Port**
- **选择进程，程序断下来**

	> 为什么会断在libc.so中
	> android系统中libc是c层中最基本的函数库，libc中封装了io、文件、socket等基本系统调用。所有上层的调用都需要经过libc封装层。所以libc.so是最基本的，所以会断在这里，而且我们还需要知道一些常用的系统so,比如linker;这个linker是用于加载so文件的模块，如何在.init_array处下断点;还有一个就是libdvm.so文件，他包含了DVM中所有的底层加载dex的一些方法

- **定位函数**

	`Ctrl+s`打开segment窗口，选择so文件（可ctrl+f搜索），记下so文件的起始地址。用另一个IDA分析so文件，找到对应的函数偏移地址。计算出绝对地址，按下**快捷键G**输入绝对地址即可跳转到正确的函数处

	> 窗口里有多个so文件怎么办？应该选择哪一个？
	> 其实这里并不是多个so文件，而是so文件对应的不同Segement信息被映射到内存中，包括代码段，数据段等，很明显，代码段会有执行权限的特点，所以我们选择带有X属性的so文件，即是我们想要调试的so文件。
	![segment](/images/Tue-Aug-21-08:41:24-2018_538730.png "segment")

	如下，已成功来到要调试的函数处。之后可下断点用f7f8调试

	![ok](/images/Tue-Aug-21-08:44:50-2018_498018.png)


### 0x03 使用ida在程序开始位置断下来调试so

- ida加载so，下好断点

- 启动android_server并端口转发
	```
	adb shell am start -D -n com.demo.rz.vv/.MainActivity
	adb shell /data/local/tmp/android_server
	adb forward tcp:23946 tcp:23946
	```

- 调试启动app并设置转发
	![调试启动app](/images/Tue-Sep--4-07:27:05-2018_500449.png "调试启动app")
	```c
	C:\Users\15748>adb shell ps | findstr vv
	u0_a53    1429  53    684160 17280 ffffffff b6f64798 t com.demo.rz.vv

	C:\Users\15748>adb forward tcp:7788 jdwp:1429
	```

- 设置ida的debugger、`Process Option`、`Debugger Option`，并附加刚才启动的进程
	![debugger option](/images/Tue-Sep--4-07:40:10-2018_864476.png "debugger option") ![debugger option](/images/Tue-Sep--4-07:43:13-2018_412617.png "debugger option")

- jdb附加(需要debugable)
	```
	jdb -connect com.sun.jdi.SocketAttach：port=7878,hostname=127.0.0.1
	```

### 0x04 gdb

- **将ndk中自带的 gdbserver push 到/data/local/tmp/下（选对cpu架构）**
- **选定进程，用gdbserver附加此进程（root）**

	```c
	OnePlus5：/ # ip addr show | grep inet // 查看ip
		...
		inet 192.168.1.100/24 brd 192.168.1.255 scope global wlan0
		...
	OnePlus5：/data/local/tmp # ps | grep example // 查找要附加的进程
	u0_a29       11341   750 4383068  56256 SyS_epoll_wait 78dc3323ac S com.example.rz.myapplication
	OnePlus5：/data/local/tmp # ./gdbserver :7878 --attach 11341
	```
- **启用gdb（最好用ndk自带的那个gdb，普通的gdb可能没有对应的架构）**

	```c
	(gdb) set architecture // 可看到ndk自带的gdb支持很多架构
	Requires an argument. Valid arguments are i386, i386：x86-64, i386：x64-32, i8086, i386：intel, i386：x86-64：intel, i386：x64-32：intel, i386：nacl, i386：x86-64：nacl, i386：x64-32：nacl, arm, armv2, armv2a, armv3, armv3m, armv4, armv4t, armv5, armv5t, armv5te, xscale, ep9312, iwmmxt, iwmmxt2, aarch64, aarch64：ilp32, auto.

	(gdb) target remote 192.168.1.100:7878 // 连接
	...
	...

	(gdb) info functions Java_.*_example // 可以用正则来查找jni
	All functions matching regular expression "Java_.*_example":
	Non-debugging symbols:
	0x000000783fa68fa0  Java_com_example_rz_myapplication_MainActivity_stringFromJNI
	0x000000783fa690f0  Java_com_example_rz_myapplication_MainActivity_loginCheck
	```

- **当然，也可以用ida来连接gdbserver**，不过感觉不太好用

### 0x05 jeb调试smali

- 连接adb，保证`adb devices`可以显示处目标设备。
- 把目标app拖入jeb中，点击Debugger->start，会让选择进程，选中目标进程即可开始调试。
- `ctrl+b`下断点

---
### 0x06 参考

> [Android逆向学习汇总](http://www.tasfa.cn/index.php/2017/08/09/android_conclusion/#more-593)
> [Android安全技术揭秘与防范](https://book.douban.com/subject/26641765/)
> [教我兄弟学Android逆向](https://www.52pojie.cn/thread-742703-1-1.html)




