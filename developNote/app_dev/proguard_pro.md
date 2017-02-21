# android混淆语法和规则
## Android Studio 中使用混淆
```java
buildTypes {
    release {
        minifyEnabled true
        zipAlignEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}

```
	发布一款应用除了设**minifyEnabled**为ture，你也应该设置**zipAlignEnabled**为true，像Google Play强制要求开发者上传的应用必须是经过zipAlign的，zipAlign可以让安装包中的资源按4字节对齐，这样可以减少应用在运行时的内存消耗。

## 混淆的作用
- 压缩
	默认开启，用以减小应用体积，移除未被使用的类和成员，并且会在优化动作执行之后再次执行（因为优化后可能会再次暴露一些未被使用的类和成员）。
    ```
    -dontshrink 关闭压缩
    ```
- 优化
	默认开启，在字节码级别执行优化，让应用运行的更快。
	```
    -dontoptimize  关闭优化
	-optimizationpasses n 表示proguard对代码进行迭代优化的次数，Android一般为5
    ```
- 混淆
	默认开启，增大反编译难度，类和类成员会被随机命名，除非用keep保护。
	```
    -dontobfuscate 关闭混淆
    ```
	混淆后默认会在工程目录app/build/outputs/mapping/release下生成一个mapping.txt文件，这就是混淆规则，我们可以根据这个文件把混淆后的代码反推回源本的代码

## 基本规则
- 保持类名
    ```
    # 只是保持该包下的类名，而子包下的类名还是会被混淆 具体方法和变量命名还是变
    -keep class com.joker.test.*

    #把本包和所含子包下的类名都保持 具体方法和变量命名还是变
    -keep class com.joker.test.**

    ```
	如果既想保持类名，又希望里面的内容不被混淆，则
    ```
    -keep class cn.hadcn.test.* {*;}
    ```
- java规则　
    此基础上，我们也可以使用Java的基本规则来保护特定类不被混淆，比如我们可以用extend，implement等这些Java规则。如下例子就避免所有继承Activity的类被混淆
    ```
    -keep public class * extends android.app.Activity
    ```

    如果我们要保留一个类中的内部类不被混淆则需要用$符号，如下例子表示保持ScriptFragment内部类JavaScriptInterface中的所有public内容不被混淆。
    ```
    -keepclassmembers class cc.ninty.chat.ui.fragment.ScriptFragment$JavaScriptInterface {
       public *;
    }
```

- 再者，如果一个类中你不希望保持全部内容不被混淆，而只是希望保护类下的特定内容，就可以使用
    ```
    <init>;     //匹配所有构造器
    <fields>;   //匹配所有域
    <methods>;  //匹配所有方法
    ```
	你还可以在<fields>或<methods>前面加上private 、public、native等来进一步指定不被混淆的内容，如
    One类下的所有public方法不被混淆
    ```
    -keep class cn.hadcn.test.One {
        public <methods>;
    }
    ```
	

- 更多配置
	有时候你是不是还想着，我不需要保持类名，我只需要把该类下的特定方法保持不被混淆就好，那你就不能用keep方法了，keep方法会保持类名，而需要用keepclassmembers ，如此类名就不会被保持，为了便于对这些规则进行理解，官网给出了以下表格:
| 保留 | 防止被移除或者被重命名 | 防止被重命名 |
|--------|--------|--------|
|   类和类成员     |  -keep      |    -keepnames    |
|   仅类成员     |  -keepclassmembers      |    -keepclassmembersnames    |
|   如果拥有某成员，保留类和类成员     |  -keepclasseswithmembers      |    -keepclasseswithmembernames    |

### 实际开发的混淆规则
- jni方法不可混淆，因为这个方法需要和native方法保持一致
	```
    # For native methods, see http://proguard.sourceforge.net/manual/examples.html#native
	-keepclasseswithmembernames class * {
    native <methods>;
	}
    ```
    
- AndroidMainfest中的类不混淆，所以四大组件和Application的子类和Framework层下所有的类都不能进行混淆
```
-keep public class * extends android.app.Fragment
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class com.android.vending.licensing.ILicensingService
```
```
 # keep setters in Views so that animations can still work.
    # see http://proguard.sourceforge.net/manual/examples.html#beans
    -keepclassmembers public class * extends android.view.View {
       void set*(***);
       *** get*();
    }

    # We want to keep methods in Activity that could be used in the XML attribute onClick
    -keepclassmembers class * extends android.app.Activity {
       public void *(android.view.View);
    }
```
- 与服务端交互时，使用GSON、fastjson等框架解析服务端数据时，所写的JSON对象类不混淆，否则无法将JSON解析成对应的对象；

- 使用第三方开源库或者引用其他第三方的SDK包时，如果有特别要求，也需要在混淆文件中加入对应的混淆规则；
 
- Parcelable的子类和Creator静态成员变量不混淆，否则会产生Android.os.BadParcelableException异常；
```
-keep class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator *;
}
```

- app module 依赖其它的module也不能混淆
```
-keep class com.submodule.packagename.**{*;}
```
- 使用enum类型时需要注意避免以下两个方法混淆，因为enum类的特殊性，以下两个方法会被反射调用，见第二条规则。
```
# For enumeration classes,
# see http://proguard.sourceforge.net/manual/examples.html#enumerations
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
```

- R文件
```
-keepclassmembers class **.R$* {
    public static <fields>;
}
```

- 有用到WebView的JS调用也需要保证写的接口方法不混淆，原因和第一条一样；

## 保持给定的特定属性
```
# -keepattributes *Annotation*,Signature
# 保护给定的可选属性
-keepattributes *Annotation*,LineNumberTable,SourceFile,Signature
# *Annotation* 注释
# EnclosingMethod 不明
# Signature 签名
# Deprecated 过期的类或方法
# InnerClasses 内部类是否混淆
# SourceFile 文件名
# LineNumberTable 行号
```
