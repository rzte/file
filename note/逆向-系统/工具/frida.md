# [FRIDA](https://www.frida.re/)

初始化时加载

```
frida -U -l ./xxx.js -f com.app.target --no-pause
```

## 实例

```javascript
// example.js

setImmediate(function(){
        console.log("[*] Starting script");
        Java.perform(function(){
                var Activity = Java.use("com.example.rz.myapplication.MainActivity");
                Activity.userCheck.implementation = function(username, password){
                        console.log("[*] userCheck got called!");
                        console.log("username: " + username);
                        console.log("password: " + password);
                        this.userCheck(username, password);

                        return true;
                }
        });
});
```

```bash
rz$ frida-ps -U | grep example
2585  com.example.rz.myapplication
rz$ frida -U -l example.js com.example.rz.myapplication
...
```

## 需要重载的函数

```javascript
/*
	// java代码，重载了crypto
	public void crypto(String arg1, String arg2);
	public void crypto(int arg1);
*/

Java.perform(function(){
		var Activity = Java.use("com.test.NetCrypto");
		Activity.crypt.overload("java.lang.String", "java.lang.String").implementation = function(arg1, arg2){
		return this.crypt(arg1, arg2);
	}
});
```

## 一些示例

### hook okhttp 的代理

```
const proxy_ip = "192.168.137.1";
const proxy_port = 8080;

if(Java.available){
	Java.perform(function(){
		var ProxyClz = Java.use('java.net.Proxy');
        var InetSocketAddressClz = Java.use("java.net.InetSocketAddress");
        
        const ip = Java.use('java.lang.String').$new(proxy_ip);
        const port = proxy_port;

		// 设置地址
        const addr = InetSocketAddressClz.$new.overload('java.lang.String', 'int').call(
            InetSocketAddressClz, ip, port
        );

		// 从堆中寻找 java.net.Proxy.HTTP
        Java.choose("java.net.Proxy$Type", {
            "onMatch": function(p){
                if(String(p).includes("HTTP")){
                    console.log(String(p));
                    ENUM_HTTP = p;
                }
            },
            "onComplete": function(p){
                if (ENUM_HTTP == null){
                    console.log("[-] not found Proxy.HTTP");
                }
                // 新建代理类
                
                const ProxyType = Java.use('java.net.Proxy$Type');
                const mProxy = ProxyClz.$new.overload('java.net.Proxy$Type', 'java.net.SocketAddress').call(
                    ProxyClz, 
                    ENUM_HTTP,
                    addr
                );
                console.log(mProxy);

			  // hook okhttp3.OkHttpClient.proxy()，使其返回我们指定的代理
                Java.use('okhttp3.OkHttpClient').proxy
                    .overload()
                    .implementation = function () {
                        console.log("[OkHttpClient][proxy]");
                        return mProxy;
                    }
            }
        })
    });
}

```



---
> [frida](https://www.frida.re)
> [hacking-android-apps-with-frida](https://www.codemetrix.net/hacking-android-apps-with-frida-1/?spm=a313e.7916648.0.0.3d6e6d11W32IYf)
> [frida releases](https://github.com/frida/frida/releases)
> [Android逆向之hook框架frida篇](https://www.jianshu.com/p/ca8381d3e094)
> [抓包二三事](https://api-caller.com/2019/11/05/capture-note/)
>



