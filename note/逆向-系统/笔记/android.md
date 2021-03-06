### webview

- 开启javascript

	```
	webview.getSettings().setJavaScriptEnabled(true);
	```

- java中调用js

	```
	webview.loadurl("javascript:func();");   // 需要开启JavaScript
	```
- js中调用java接口
	```java
	webview.addJavaScriptInterface(new Func(), "android");    // javascript中android即可代表Func对象
	...

	class Func{
		void func1(){}
		void fun2(){}
	}
	```

### 非root设备使用gdbserver进行native调试

没有root的设备，在使用gdbserver调试app会遇到权限问题。（emulator没问题）

Android提供了一个**run-as**命令来暂时切换用户，但这个命令有限制，必须是app打开了**debuggable**才行，否则会提示`Package xx is not debuggalbe`错误

```c
<application android:debuggable="true" ...
```

**开启`debuggable`的作用主要有两个:**

- 可以查看该应用的数据（/data/data/com.xxx.xxx/里的数据）
- 可以使用gdb_server进行native的debug

gdb_server在ndk包下：`android-ndk-r16b/prebuilt/android-arm/gdbserver`

```c
adb push gdbserver /data/local/tmp/
adb shell chmod 755 /data/local/tmp/gdbserver
```

### 查看所有已安装的apk包

- adb shell ls -l /data/data/（需root权限才可查看）
- adb shell pm list package（无需root权限）

```c
adb shell
$ ls /data/data/com.xxx/
ls: /data/data/com.xxx/: Permission denied

$ run-as com.xxx
$ ls /data/data/com.xxx/
cache databases lib ...
```

### 调试模式启动

```c
adb shell am start -D -n com.exammple.vv/.MainActivity
```

### 查看顶部的activity

```c
adb shell dumpsys activity top
```

### 分析Trace文件

用DDMS`Start Method Profiling`抓取到的trace文件可用**dmtracedump**转换为图片进行分析其调用关系

```bash
$ sudo apt install dmtracedump // 安装 dmtracedump
$ sudo apt install graphviz // graphviz用来绘制DOT语言脚本描述的图形
$ dmtracedump -g out.png ddms736688747470615083.trace
```

---
### IDA分析so

**使用IDA load file功能，导入JNI.h解析【JNI函数】**

使IDA导入C/C++头文件，添加头文件中的结构体，使此结构体中的函数替换反汇编中的偏移，使文件的可读性更好

1. 修改ndk包中的jni.h文件（最好是备份一下）
	- 注释`#include <stdarg.h>`和`#include <stdint.h>`
	- 将1100行左右的`#define JNIEXPORT  __attribute__ ((visibility ("default")))`改为`#define JNIEXPORT`
2. 点击IDAPro菜单选项“File -> Load file -> Parse c header file”，选择第一步修改好的jni.h文件
3. 在IDAPro的 Structures 界面，按 insert 按键，选择 “Add standard structure”，添加`JNIInvokeInterface`和`JNINativeInterface`。
	![Add standard stucture](/images/Mon-Aug-13-08:46:39-2018_223224.png "Add standard stucture")
4. 此时按F5查看反汇编发现得到的代码仍然没有转化为函数调用。选中目标函数的第一个参数，右键int，选择“Convert to struct\*”，选择`_JNIEnv`
	![Convert to struct](/images/Mon-Aug-13-08:53:09-2018_274850.png "Convert to struct")
5. 此时已发现转化为函数形式
	![over](/images/Mon-Aug-13-08:56:13-2018_641805.png "over")


### 源码

#### 源码

```bash
git clone https://android.google.com/kernel/common.git
git clone https://android.google.com/kernel/goldfish.git # 所有模拟器平台的内核，夜深研究时经常使用的内核源码
git clone https://android.google.com/kernel/x86_64.git  # 针对设备Nexus Player，常用于Intel的Intel x86_64芯片设备的Android内核
git clone https://android.google.com/kernel/exynos.git  # 针对设备Nexus 10，常用于三星的Exynos芯片设备的Android内核
git clone https://android.google.com/kernel/msm.git     # 针对设备ADPI，adp2，Nexus One，Nexus4，Nexus5，Nexus6的内核，常用于高通芯片设备的Android内核
git clone https://android.google.com/kernel/omap.git    # 针对pandaboard和Galaxy Nexus系列的设备内核，常用于TI OMAP芯片设备的Android内核
git clone https://android.google.com/kernel/samsung.git # 三星内核，用于三星的蜂鸟芯片设备的Android内核
git clone https://android.google.com/kernel/tegra.git   # 针对设备Xoom，Nexus7，Nexus9，常用于Nvidia的Tegra芯片设备的Android内核
```

#### 编译内核(Android安全技术防范与揭秘 p292)

```bash
git clone https://android.google.com/kernel/goldfish.git
...# 等待...
cd goldfish
git checkout -t origin/android-goldfish-2.6.29 -b goldfish

# 下载交叉编译器进行交叉编译
git clone https://android.googlesource.com/platform/perbuilt

# 加入环境变量
export PATH=$PATH:`pwd`/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin

export ARCH=arm
export SUBARCH=arm
export CROSS_COMPILE=arm-eabi- # 交叉编译器的前缀
export TARGET_PREBUILT_KERNEL=$your_kernel_path/arch/arm/boot/zImage # 编译后生成image文件的路径
sudo...
```

### 参考

> [非root Android 设备用gdbserver进行native 调试的方法 ](https://www.52pojie.cn/forum.php?mod=viewthread&tid=462161 "非root Android 设备用gdbserver进行native 调试的方法 ")















