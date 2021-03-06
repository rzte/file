### 杂

Android5.0版本会强制开启SELinux（Android上又称SEAndroid）。

目录 | 主要功能
-- | --
/system/app | 系统自带的应用程序存放，Root权限才可更改(删除预装应用可从这里删)
/data/app   | 用户程序安装的目录，有删除权限，安装时把apk文件复制到此目录
/data/data  | 存放应用程序的数据
/data/dalvik-cache | 将apk中的dex文件安装到dalvik-cache目录下

`getevent -l`读取信号，实现按键监控（需root权限）。`sendevent`模拟发送事件。

安装过程

复制APK安装包到`/data/app`下，解压并扫描安装包，把dex文件（Dalvik字节码）保存到dalvik-cache目录，并在`/data/data`下创建对应的应用数据目录


**读取log**

在app中声明权限：**READ_LOGS**

```java
public void run(){
	Process mProc = null;
	BufferrdReader br = null;
	String line = null;
	try{
		mProc = Runtime.getRuntime().exec(new String[]{"logcat", "ActivityManager:D"}); // 监听tag为ActivityManager的debug信息
		br = new BufferedReader(new InputStreamReader(mProc.getInputStream()));
		while((line=br.readLine()) != null){
			// 过滤password
			if(line.indexOf("password") > 0){
				Looper.prepare(); // 使用looper显示Toast
				Toast.makeText(this, "password：" + line, Toast.LENGTH_SHORT).show();
				Looper.loop();
			}
		}
	}
}
```

### Android系统

#### Android系统的层级

Android整体架构组件主要分为5层，包括应用层（Application）、应用框架层（Application Framework）、核心库与运行环境层（Libraries and Runtime）、Linux内核层（Linux Kernel层）。

- 应用层
	Android应用程序主要是用户界面（User Interface），通常以java编写，其中还可以包含各种资源文件（放在res目录中）。Java程序及相关资源经过编译后，将生成一个APK（Android Package）。

- 框架层
	Android的应用程序框架为应用程序层的开发者提供API，它实际上是一个应用程序的框架。上层应用程序是以Java构建的，因此，本层次包含了UI程序中需要的各个控件，如Views（视图组件）包括list、grids、text boxes、buttons、webview等。一个Android应用程序可以利用应用程序框架中的以下几个部分：
	- Activity（活动）
	- Broadcast Intent Receiver（广播意图接收者）
	- Service（服务）
	- Content Provider（内容提供者）

- 核心库与运行环境层
	本层次对应嵌入式系统，相当于中间件层次。Android的本层次分成两个部分，一个是各种库，另一个是Android运行环境。本层的内容大多是C++实现的。各种库包括：
	- C库
		C语言的标准库，这也是系统中最底层的库，C库通过Linux系统调用来实现
	- 多媒体框架（MediaFramework）
		这部分是Android多媒体的核心部分，基于PacketVideo（PV）的OpenCore，从功能上分为两大部分，一部分是音频、视频的回放（PlayBack），另一部分是音频视频的记录（Recorder）
	- SGL
		2D图像引擎
	- SSL
		即Secure Socket Layer，位于TCP/IP协议与各种应用层协议之间，为数据通信提供安全支持
	- OpenGL ES 1.0
		提供了3D的支持
	- 界面管理工具（Surface Management）
		提供了管理显示子系统等功能
	- SQLite
		一个通用的嵌入式数据库
	- WebKit
		网络浏览器的核心
	- FreeType
		位图与矢量字体的功能

- Linux内核层
	Android使用Linux2.6为操作系统，Android对操作系统的使用包括核心和驱动程序两部分，Android的Linux核心为标准的Linux2.6内核，Android更多的是需要一些与移动设备相关的驱动程序：
	- 显示驱动
		常基于Linux的帧缓冲驱动
	- Flash内存驱动
	- 照相机驱动
		常基于Linux的v41驱动
	- 音频驱动
		常基于ALSA（高级Linux声音体系 Advanced Linux Sound Architecture）驱动
	- WIFI驱动
		基于IEEE802.11标准的驱动程序
	- 键盘驱动
	- 蓝牙驱动
		Binder IPC驱动：Android的一个特殊的驱动程序，具有单独的设备节点，提供进程间通信功能
	- Power Management（能源管理）

#### Android系统的分区

分区是逻辑层存储单元用来区分设备内部的永久性存储结构。不同厂商和平台之间一般有不同的分区布局。但是有几个分区是在所有Android设备中最常见的，及**Boot**、**Data**、**Recovery**和**Cache**分区。通常NAND闪存的设备都具备以下分区布局：

- Boot Loader分区
	中文名称“系统加载器”，相当于电脑的BIOS，在手机进入系统之前初始化软硬件环境、加载硬件设备，最终让手机启动成功。各大厂商为了保障手机有稳定的运行环境，自家的系统价值，用户的使用安全等都会给BootLoader进行加密。加密后的BootLoader仅能引导给定的固件，第三方固件将不与识别。

- Boot分区
	存储着Android的Boot镜像，包含Linux kernel（zImage）与initrd等文件

- Splash分区
	主要是存储系统启动后第一屏显示的内容，一般都是一些公司的Logo或者动画，存储在BootLoader中

- Radio分区
	基带所在分区，存储着一些与通信质量相关的Linux驱动，如电话、GPS、蓝牙、WIFI驱动等。常见的驱动是可以打包存在于Linux内核的Boot分区的，但为了提升设备的通信质量所有单独开辟了Radio分区

- Recovery分区
	存储着一个mini型的Android Boot镜像文件，主要作用是来做故障维修和系统恢复（有点类似WinPE）

- System分区
	存储着Android系统的镜像文件，镜像文件中包含着Android的Framework、Libraries、Binaries和一些预装应用。系统挂载后即/system目录

- User Data分区
	也称数据分区，它是设备的内部存储分区，比如应用产生的图片、声音等文件。挂载在/data目录下

- Cache分区
	存储而告终实用文件，如恢复日志和OTA下载的更新包。在应用程序安装在SD卡上时，它可能包含Dalvik缓存文件夹，其中存储着Dalvik虚拟机的缓存文件

---
### Root之后

#### 删除预装

Android系统的捆绑应用软件基本安装在`/system/app`文件夹下，删除下面对应的apk文件即可卸载。因为`/system`是系统的目录，所有需要对目录操作需要Root权限。

有些恶意的手机ROM会有更恶心的方法来留住预装应用，比如修改ROM的逻辑，让系统在开机时检测以下自己的预装是否完整然后重新安装。当然，这种一般会在一个隐蔽的目录中存储着一份它们的安装包apk文件。这类预装应用（开机静默安装）常用的方式就是修改`init.rc`。添加一个开机执行脚本，在脚本中调用一个Service使用`pm install`命令批量安装应用。要删除这种应用的话，可以修改启动逻辑也可以全盘扫描apk文件进行删除（具体还要具体分析）

#### 键盘监控

键盘监控包括物理按键与如那键盘的监控，通常监控的事件有：点击、长按、滑动等。这些事件在Android上表现出来的是一系列KeyEvent

为了实现键盘监控，重新开发一个输入法是不现实的，一般的操作是在系统输入法机制中添加接口回调：

```java
public boolean onKeyDown(int keyCode, KeyEvent event);
```

不过**事件的回调机制是在其沙箱中运行的，其他应用中无法拿到当前应用事件回调**，这里可以使用`getevent`和`setevent`来对按键监控和模拟按键发送

`getevent`用来获取`/dev/input/event*`设备汇报的事件。这个命令还会输出所有event设备的基本信息，包括触屏、按键、耳机插入等。

```c
root@android:/data/app # getevent -l
...
/dev/input/event7: EV_KEY       BTN_TOUCH            DOWN
/dev/input/event7: EV_ABS       ABS_MT_PRESSURE      00000001
/dev/input/event7: EV_ABS       ABS_MT_POSITION_X    00000294
/dev/input/event7: EV_ABS       ABS_MT_POSITION_Y    00000511
/dev/input/event7: EV_SYN       SYN_MT_REPORT        00000000
/dev/input/event7: EV_SYN       SYN_REPORT           00000000
/dev/input/event7: EV_SYN       SYN_MT_REPORT        00000000
/dev/input/event7: EV_SYN       SYN_REPORT           00000000
```

#### 短信拦截与静默发送

- 短信拦截
	短信拦截与获取root权限关系并不大，Android在接收到短信时会发送出来一个广播，而短信广播是一个**有序广播**，该有序广播会发送给优先级最高的Receiver，这样就提供给我们一个利用优先级的方式声明一个高优先级的广播接收器去拦截短信。如下，我们声明了一个优先级为9999的短信广播接收器SmsReceiver去拦截系统的短信接收广播

	```xml
	<receiver android:name=".SmsReceiver">
		<intent-filter android:priority="9999">
			<action android:name="android.provider.Telephony.SMS_RECEIVED"/>
		</intent-filter>
	</receiver>
	```
	
	在接收SmsReceiver后做一些拦截操作

	```java
	public class SmsReceiver extends BroadcastReceiver{
		@Override
		public void onReceive(Context context, Intent intent){
			// 判断是否是系统短信
			if (intent.getAction().equals("android.provider.Telephony.SMS_RECEIVED")){
				StringBuffer msgText = new StringBuffer();
				String sender = null;
				Bundle bundle = intent.getExtras();
				if (bundle != null){
					//通过pdus获得接收到的所有短信消息，获取短信内容
					Object[] pdus = (Object[])bundle.get("pdus");
					// 构造短信数组
					SmsMessage[] mges = new SmsMessage[pdus.length];
					for (int i = 0; i < pdus.length; ++i){
						// 获取单挑短信内容，以pdu格式保存，并生成短信对象
						mges[i] = SmsMessage.createFromPdu((byte[])pdus[i]);
					}
					msgText.append("短信来自：").append(mge.getDisplayOriginatingAddress()).append("\n");
					msgText.append("短信内容：").append(mge.getMessageBody());
					
					if ("5555".equals(sender)){
						// 屏蔽手机号为5555的短信
						// 这里还可以实现一些处理，比如把该信息发送到第三人的手机
						abortBroadcast();
					}
				}
				Toast.makeText(context, msgText.toString(), Toast.LENGTH_LONG).show();
			}
		}
	}
	```

- 短信静默发送
	在不通过用户任何许可下在后台静默发送以达到扣费的目的。在Android系统中，希望发送短信的话，一般可通过调用系统的SmsManager中的sendTextMessage方法来完成

#### 电话监控

在android平台上`TelephonyManager`中的方法是针对设备通话的封装，常用的方法有**dial**（拨号界面）、**call**（直接通话）、**endCall**（结束通话）、**answerRingingCall**（接听通话）。其API是不对系统外的应用开放的（注解为@Systemapi)的方法。但是开发者们仍能通过Java中的**反射**方法去调用其接口完成通话直接拨打电话。如下所示，只需要在AndroidManifest.xml中声明CALL_PHONE权限然后调用call方法就能在不经过用户授权的情况下拨打电话：

```java
private void call(String number){
	Class<TelephonyManager> c = TelephoneManager.class;
	Method getITelephonyMethod = null;
	try{
		// setAccessible(true) 跳过java中的private方法的反射权限检查
		// 即我们能够在外部调用private方法而不报错
		getITelephonyMethod = c.getDeclareMethod("getITelephony", (Class[])null);
		getITelephonyMethod.setAccessible(true);
	}catch(Exception e){
		e.printStackTrace();
	}
	
	try{
		// 获取TelephonyManager
		TelephonyManager tManager = (TelephonyManager)getSystemService(Context.TELEPHONY_SERVICE);
		Object iTelephony;
		iTelephony = (Object)getITelephonyMethod.invoke(tManager, (Object[])null);
		
		// 反射call方法
		Method dial = iTelephony.getClass().getDeclareMethod("call", String.class);
		
		// 拨打指定电话
		dial.invoke(iTelephony, number);
	}catch(Exception e){
		//...
	}
}
```

当然在android中不经过用户同意拨打电话虽然很流氓，但是没啥用，恶意软件更多是在窃取用户的通话记录，有些还会在通话时录音。

在Android中进行电话监控只需要在**TelephonyManager**中注册**PhoneStateListener**就能拿到当前通话状态的变化，当然也能做相应的处理。

```java
// 获取TelephonyManager
Telephony telephoneyManager = (TelephonyManager)getSystemService(Context.TELEPHONY_SERVICE);

// 注册监听通话状态
telephoneyManager.listen(new MyPhoneCallListener(), PhoneStateListener.LISTEN_CALL_STATE);

/**
* 监听电话的具体表现
*/
public class MyPhoneCallListener extends PhoneStateListener{
	@Override
	public void onCallStateChanged(int state, String incomingNumber){
		switch(state){
			// 电话通话的状态
			case TelephonyManager.CALL_STATE_OFFHOOK:
				break;
			// 电话响铃的状态
			case TelephonyManager.CALL_STATE_RINGING:
				break;
			// 空闲中
			case TelephonyManager.CALL_STATE_IDLE:
				break;
		}
		super.onCallStateChanaged(state, incomingNumber);
	}
}
```

---
### APK静态分析

- [AXMLPrint2](http://code.google.com/p/android4me)
	APK中的资源是经过压缩的，用文本工具查看为乱码，其格式为AXML（Android binary XML）。可以用AXMLPrint2转换为可读的xml文件
- [dex2jar](http://code.google.com/p/dex2jar)
	将`.dex`文件转换为jar
- [jd-GUI](http://jd.benow.ca)
	`.class`转java文件
- [APKTool](https://code.google.com/p/android-apktool/)
	反编译apk
	```cmd
	apktool d xxx.apk -o xxx
	apktool b xxx -o xxx.apk
	```

静态分析也就是对APK解压后的几个重要文件，classes.dex文件、AndroidManifest.xml文件、so文件（JNI）、resources.arsc（资源文件）进行分析、篡改、注入等

#### 反编译前结构

使用unzip解压APK文件，其包含的文件文件夹大概如下:

- assets 声音、字体、网页...等资源（可直接查看）
- org/com 第三方库，如org.apache.http等
- lib 应用中使用到的库
	- armeabi .so文件，C/C++代码库文件【重要】
- META-INF APK的签名文件（xxx.RSA、xxx.SF、xxx.MF3个文件）
- res 应用中使用到的资源目录，已编译无法直接阅读
- anim 动画资源animation
- color 颜色资源
	- drawable 可绘制的图片资源
	- drawable-hdpi 图片资源高清
	- drawable-land 图片资源横向
	- drawable-land-hdpi 图片资源横向高清
	- drawable-mdpi 图片资源中等清晰
	- drawable-port 图片资源纵向
	- drawable-port-hdpi 图片资源纵向高清
	- layout 页面布局文件
	- layout-land 页面布局文件横向
	- layout-port 页面布局文件纵向
	- xml 应用属性配置文件
- AndroidManifest.xml 应用的属性定义文件，无法直接阅读【重要】
- classes.dex Java源码编译后的代码文件【重要】
- resources.arsc 编译后的资源文件，如strings.xml【重要】

#### 反编译后的结构

用apktool等反编译工具对apk反编译后的文件结构如下：

- assets 声音、字体、网页...等资源
- lib 应用中使用到的库
	- armeabi .so文件，C/C++代码库文件【JNI部分】【无法阅读】【重要】
- res 应用中使用到的资源目录，目录下的东西可以【直接阅读】
	- anim 动画资源animation
	- color 颜色资源
	- drawable 可绘制的图片资源
	- drawable-hdpi 图片资源高清
	- drawable-land 图片资源横向
	- drawable-land-hdpi 图片资源横向高清
	- drawable-mdpi 图片资源中等清晰
	- drawable-port 图片资源纵向
	- drawable-port-hdpi 图片资源纵向高清
	- layout 页面布局文件【重要】
	- layout-land 页面布局文件横向【重要】
	- layout-port 页面布局文件纵向【重要】
	- values
		- strings.xml 应用中使用到的字符串常量【重要】
		- dimens.xml 间隔（dip、sp）常量
	- xml 应用属性配置文件【重要】
- AndroidManifest.xml 应用的属性定义文件【重要】
- smali Java代码反编译后生成的代码文件【smali语法】【重要】
- apktool.yml apktool反编译的配置文件，用于重新打包

#### DEX文件

Dex文件是Android上的可执行文件，由Java虚拟机JVM编译后再由ANdroid中的虚拟机Dalvik所编译后而成，如下：

![dex文件生成过程](/images/Sun-Aug-12-05:44:41-2018_233751.png "dex文件生成过程")

#### Smali

Smali是对Dalvik虚拟机字节码的一种解释，虽然不受官方标准语言，但所有语句都遵循一定的语法规范。可查看[官方文档](https://code.google.com/p/smali/)

**常用的Smali注入代码**：

- 增加Log信息
	```java
	Log.d("Test", "Hello Smali Inject Log");
	```
	```smali
	const-string v1, "Test"
	const-string v2, "Hello Smali Inject Log"
	invoke-static {v1, v2}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I
	```
- 弹出AlertDialog消息框
	```java
	new AlertDialog.Builder(this)
		.setTitle("Dialog Title") // 弹窗标题
		.setMessage("Hello Smali Inject Message") // 弹窗消息
		.show();
	```
	```smali
	new-instance v1, Landroid/app/AlertDialog$Builder;
	invoke-direct{v1, p0}, Landroid/App/AlertDialog$Builder;-><init>(Landroid/content/Context;)V
	
	# 新建一个string内容为Dialog Title
	const-string v2, "Dialog Title"
	
	# 调用AlertDialog方法Builder中的setTitle设置标题
	invoke-virtual {v1, v2}, Landroid/App/AlertDialog$Builder;->setTitle(Ljava/lang/CharSequence;)Landroid/App/AlertDialog$Builder;
	
	# 调用setMessage设置信息为Hello Smali Inject Messag
	move-result-object v1
	const-string v2, "Hello Smali Inject Message"
	invoke-virtual {v1, v2}, Landroid/App/ALertDialog$Builder;->setMessage(Ljava/lang/CharSequence;)Landroid/App/AlertDialog$Builder;
	
	# 调用show，显示Dialog
	move-result-object v1
	invoke-virtual {v1}, Landroid/app/AlertDialog$Builder;->show()Landroid/app/AlertDialog;
	```
- MethodTracing
	```java
	android.os.Debug.startMethodTracing("123");
	android.os.Debug.stopMethodTracing();
	```
	```smali
	# 新建一个内容为123的string
	const-string v1, "123"
	
	# 调用startMethodTracing方法
	invoke-static {v1}, Landroid/os/Debug;->startMethodTracing(Ljava/lang/String;)V
	
	# 调用stopMethodTracing方法
	invoke-static { }, Landroid/os/Debug;->stopMethodTracing()V
	```

#### AndroidManifest.xml

Application类是全局的基础类，它的生命周期即整个应用的生命周期，是一个单例。在AndroidManifest.xml中声明如下：

```xml
<application
	android:name="com.hello.MyApplication"
	android:icon="@drawable/ic_launcher"
	android:label="@string/app_name">...</application>
```

**特殊的Permission**

- 开机启动权限
	系统启动完后会发送一个全局的广播**Boot Completed**，在AndroidManifest.xml中定义如下：
	```xml
	<user-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
	```
	分析：接收开机广播无非是想启动自身后台服务，如打开拉取后台配置文件服务，打开push服务，弹出通知信息等。

- 短信读、写发送权限
	全局搜索短信操作方法的关键字，“content://sms”、“SMS.CONTENT_URI”等即可定位到关键代码
	```xml
	<user-permission android:name="android.permission.SEND_SMS"/>
	<user-permission android:name="android.permission.WRITE_SMS"/>
	<user-permission android:name="android.permission.READ_SMS"/>
	```
	分析：发送、接收、读取短信的目的无非是“订阅SP服务（现在已经很少了）”、“发送/接收验证码信息”、“拦截短信（常用于安全类应用中）”、“木马/扣费等恶意传播”

- 手机信息读取权限
	手机信息读取权限，表明应用中有一些需要读取设备的相关信息，如GSM手机的IMEI、CDMA手机的序列号、手机号、设备版本网络类型等。
	此类具体服务都封装在一个名为“TelephonyManager”的service里，所以需要定位到此类权限的使用处可以全局搜索“TelephonyManager”、“phone”、“Context.TELEPHONY_SERVICE”等
	```xml
	<user-permission android:name="android.permission.READ_PHONE_STATE"/>
	```
	分析：此类权限看似普通，但是却是很多应用做唯一性判断的关键。因为设备的DeviceId（IMEI或MEID）是唯一的。很多开发者喜欢用此作为用户唯一区分值，如“首次安装用户可抽奖”、“新用户送话费”等。修改了DeviceId的读取判断信息后，用户就可以多次参与了

#### 关键字突破

涉及网络相关的操作可搜索`HttpConnection`、`HttpClient`等网络连接关键字。涉及加密解密相关的，可搜索`MD5`、`BASE64`、`AES`等加密关键字。设计so文件里面的函数的，可搜索`native`关键字

#### 资源索引突破

从资源（图片、布局、字符串、声音等）定位关键代码，主要分以下几步：

1. 反编译
2. 确定关键图片/文字/布局等文件
3. 从关键的资源文件找到R文件的资源索引
4. 根据R文件的索引，找到相关view
5. 修改view的onClick、onLongClick等事件
6. 签名/回编译








