### 0x01 build.gradle

```xml
dependencies {
    compileOnly 'de.robv.android.xposed:api:53'
    compileOnly 'de.robv.android.xposed:api:53:sources'
}
```

### 0x02 AndroidManifest.xml

```xml
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        <!-- 添加以下节点 -->
        <meta-data
            android:name="xposedmodule"
            android:value="true"/>
        <meta-data
            android:name="xposeddescription"
            android:value="xposed example"/> <!-- 描述 -->
        <meta-data
            android:name="xposedminversion"
            android:value="53"/> <!-- xposed版本(之前build.gradle中配置的那个) -->
    </application>
```

### 0x03 IXposedHookLoadPackage

新建一个java类，实现IXposedHookLoadPackage接口

```java
package com.xposed.rz.xposeddemo;

import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.callbacks.XC_LoadPackage;

/**
 * 实现 IXposedHookLoadPackage
 */
public class Tutorial implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
        XposedBridge.log("Loaded app: "); // log会写到 tag 为 Xposed 的logcat中，具体位置：/data/data/de.robv.android.xposed.installer/log/debug.log
        XposedBridge.log(lpparam.packageName);
        XposedBridge.log(lpparam.processName);
        XposedBridge.log(lpparam.appInfo.className);
    }
}
```

### 0x04 assets/xposed_init

为了确认入口点，还需要在`assets`下新建一个`xposed_init`，每行包含一个全限定类名。

```txt
com.xposed.rz.xposeddemo.Tutorial
```

### 0x05 example

```java
package com.xposed.rz.xposeddemo;

import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;

/**
 * 实现 IXposedHookLoadPackage
 */
public class Tutorial implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
        if(!lpparam.packageName.equals("com.example.rz.myapplication")){
            return;
        }

        XposedBridge.log("we are in com.example.rz.myapplication");
        XposedBridge.log("start hook");

        XposedHelpers.findAndHookMethod("com.example.rz.myapplication.MainActivity", // 类的全限定名
                lpparam.classLoader, "userCheck", // 方法
                String.class, String.class, // 方法的参数类型
                new XC_MethodHook(){
                    @Override
                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                        XposedBridge.log("===================beforeHookedMethod================");
                        XposedBridge.log(param.method.getName());
                        for(Object obj: param.args){ // 被hook的方法的参数
                            XposedBridge.log(obj.toString());
                        }
                        XposedBridge.log("=====================================================");
                        super.beforeHookedMethod(param);
                    }

                    @Override
                    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                        XposedBridge.log("===================afterHookedMethod================");
                        XposedBridge.log(param.method.getName());
                        XposedBridge.log(param.getResult().toString()); // 执行结果
                        XposedBridge.log("=====================================================");

                        param.setResult(true); // 设置结果为true
                        super.afterHookedMethod(param);
                    }
                });
    }
}
```

---
### 相关资料

> [Development-tutorial](https://github.com/rovo89/XposedBridge/wiki/Development-tutorial)
> [Using-the-Xposed-Framework-API](https://github.com/rovo89/XposedBridge/wiki/Using-the-Xposed-Framework-API)
> [API](https://api.xposed.info/reference/packages.html)


