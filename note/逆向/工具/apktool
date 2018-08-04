### [ApkTool](https://ibotpeaches.github.io/Apktool/documentation/ "ApkTool")

apk文件是一个包含资源和java字节码的zip文件，但是只是简单的解压这样的apk，将会留下`classes.dex`和`resources.arsc`之类的文件。并且像`AndroidManifest.xml`这种文件无法正常被查看。这时就可以用到apktool了。

其简单用法如下：

```c
sage: apktool
 -advance,--advanced   prints advance information.
 -version,--version    prints the version then exits
usage: apktool if|install-framework [options] <framework.apk>
 -p,--frame-path <dir>   Stores framework files into <dir>.
 -t,--tag <tag>          Tag frameworks using <tag>.
usage: apktool d[ecode] [options] <file_apk>
 -f,--force              Force delete destination directory.
 -o,--output <dir>       The name of folder that gets written. Default is apk.out
 -p,--frame-path <dir>   Uses framework files located in <dir>.
 -r,--no-res             Do not decode resources.
 -s,--no-src             Do not decode sources.
 -t,--frame-tag <tag>    Uses framework files tagged by <tag>.
usage: apktool b[uild] [options] <app_path>
 -f,--force-all          Skip changes detection and build all files.
 -o,--output <dir>       The name of apk that gets written. Default is dist/name.apk
 -p,--frame-path <dir>   Uses framework files located in <dir>.
```

---

- Decode

	 解码可以用`d`或`decode`，如下：
	
	```c
	$ apktool d foo.jar			// decodes foo.jar to foo.jar.out folder

	$ apktool decode foo.jar	// decodes foo.jar to foo.jar.out folder

	$ apktool d bar.apk			// decodes bar.apk to bar folder

	$ apktool decode bar.apk	// decodes bar.apk to bar folder

	$ apktool d bar.apk -o baz	// decodes bar.apk to baz folder
	```
- Building
	
	构建可以用`b`或者`build`，如下：
	
	```c
	$ apktool b foo.jar.out				// builds foo.jar.out folder into foo.jar.out/dist/foo.jar file

	$ apktool build foo.jar.out			// builds foo.jar.out folder into foo.jar.out/dist/foo.jar file

	$ apktool b bar						// builds bar folder into bar/dist/bar.apk file

	$ apktool b .						// builds current directory into ./dist

	$ apktool b bar -o new_bar.apk		// builds bar folder into new_bar.apk
	```

### 签名

Android系统要求每一个Android应用程序要经过数字签名后才能够安装到系统中，所以重新构造后的apk需要进行签名后才可正常使用。

给apk签名一共要用到三个工具，分别是**keytool**、**jarsigner**和**zipalign**

- keytool

	生成数字证书（即秘钥 .keystore），在`jdk1.6/bin/keytool`处
	
	```c
	$ keytool -genkey -alias alias.keystore -keyalg RSA -validity 40000 -keystore demo.keystore
	-alias alias.keystore 表示证书的别名位alias.keystore
	-keyalg RSA 表示生成秘钥文件采用的算法是RSA
	-validity 40000 表示该数字证书的有效期为40000天
	```

- jarsigner

	使用数字证书给apk文件签名，在`jdk1.6/bin/jarsigner`处

	```c
	jarsigner [ options ] jar-file alias
	
	jarsigner -verbose -keystore demo.keystore -signedjar signedxxx.apk xxx.apk alias.keystore
	-keystore demo.keystore 表示签名使用的数字证书所在的位置，没写路径表示在当前目录下
	-signedjar signedxxx.apk 签名后的apk为signedxxx.apk
	xxx.apk 对xxx.apk进行签名
	alias.keystore 证书别名
	```

- zipalign（非必须）

	对签名后的apk进行优化，提高与Android系统交互的效率，在`/usr/lib/android-sdk/build-tools/debian/zipalign`处（windows也类似）
	
	```c
	NAME
		   zip对齐
	SYNOPSIS
		   对齐 infile.zip 并且保存至 outfile.zip:
			   zipalign [-f] [-p] [-v] [-z] align infile.zip outfile.zip
		   确认 existing.zip 的对齐方式：
			   zipalign -c [-v] align existing.zip
	```

从上面可以看到，签名需要的两个工具都是jdk自带的，这就意味着生成数字证书和文件签名不是android的专利，从`jarsigner`也能看出这个主要是给jar文件签名的