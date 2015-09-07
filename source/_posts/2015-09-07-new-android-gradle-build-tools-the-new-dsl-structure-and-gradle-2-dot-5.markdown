---
layout: post
title: "新Android Gradle Build Tools: Gradle 2.5"
date: 2015-09-07 16:22:45 +0800
comments: true
categories: Android
---

Android Studio 1.3已经除了稳定版了。新特性包括了完全NDK支持，而且一个主要的更改是DSL(Domain-Specific Language)的变化。

翻译自[inthechessefactory](http://inthecheesefactory.com/blog/new-gradle-build-tools-with-gradle-2.5/en)

<!--more-->

####什么是Android Gradle Build Tools

在把每个module的build.gradle文件传递给Gradle之前，Android Gradle Build Tools 用来提前处理下这些文件。

Gradle Build Tools的版本是在project的build.gradle里指定的，类似：

	dependencies {
		classpath 'com.android.tools.build:gradle:1.2.3'
	}

Gradle Build Tools版本和Gradle版本对应关系如下：

Android Gradle Plugin | Gradle
:---------------------|:------------------
1.0.0-1.1.3           |2.2.1-2.3
1.2+                  |2..2.1+

####The new Android Gradle Build Tools

使用新的Gradle Build Tools的话，只需要换掉build tools的version

	dependencies {
		classpath 'com.android.tools.build:gradle-experimental:0.1.0'
	}

不过只有gradle2.5才能匹配使用，所以需要设置gradle-wrapper.properties

	distributionUrl=https\://services.gradle.org/distributions/gradle-2.5-bin.zip

然后像下面这样编辑模块的build.gradle

	apply plugin: 'com.android.model.application'
	
	model {
        android {
			compileSdkVersion = 22
			buildToolsVersion = "23.0.0 rc3"
	
		    defaultConfig.with {
				applicationId = "com.inthecheesefactory.hellojni25"
				minSdkVersion.apiLevel = 15
				targetSdkVersion.apiLevel = 22
				versionCode = 1
				versionName = "1.0"
			}
		}

		android.buildTypes {
			release {
				isMinifyEnabled = false
				proguardFiles += file('proguard-rules.pro')
			}
		}
	}
	
	dependencies {
		compile fileTree(dir: 'libs', include: ['*.jar'])
		compile 'com.android.support:appcompat-v7:22.2.0'
	}

仔细看就会发现，plugin不再是`com.android.application`，而是`com.android.model.application`。  
`+=`被引入表示在一个collection中增加一些元素。

####支持 NDK

修改项目的local.properites文件

	ndk.dir=PATH_TO_NDK_ROOT

，或者直接使用android studio下载ndk

然后在java的package里创建`HelloJni.java`文件

{% codeblock lang:java %}

public class HelloJni {
    public native String stringFromJNI();
}

{% endcodeblock %}

在src/main目录下创建jni文件夹，然后创建hello-jni.c文件

{% codeblock lang:c %}

#include <string.h>
#include <jni.h>
 
jstring
Java_com_inthecheesefactory_hellojni25_HelloJni_stringFromJNI( JNIEnv* env,
                                                  jobject thiz )
{
#if defined(__arm__)
  #if defined(__ARM_ARCH_7A__)
    #if defined(__ARM_NEON__)
      #if defined(__ARM_PCS_VFP)
        #define ABI "armeabi-v7a/NEON (hard-float)"
      #else
        #define ABI "armeabi-v7a/NEON"
      #endif
    #else
      #if defined(__ARM_PCS_VFP)
        #define ABI "armeabi-v7a (hard-float)"
      #else
        #define ABI "armeabi-v7a"
      #endif
    #endif
  #else
   #define ABI "armeabi"
  #endif
#elif defined(__i386__)
   #define ABI "x86"
#elif defined(__x86_64__)
   #define ABI "x86_64"
#elif defined(__mips64)  /* mips64el-* toolchain defines __mips__ too */
   #define ABI "mips64"
#elif defined(__mips__)
   #define ABI "mips"
#elif defined(__aarch64__)
   #define ABI "arm64-v8a"
#else
   #define ABI "unknown"
#endif
 
    return (*env)->NewStringUTF(env, "Hello from JNI !!  Compiled with ABI " ABI ".");
}

{% endcodeblock %}

请记住让`com_inthecheesefactory_hellojni25`和HelloJni.java的包名是一致的。makefile不再需要了。

现在在`MainActivity.java`里测试一下

{% codeblock lang:java %}
public class MainActivity extends AppCompatActivity {
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
 
	    Toast.makeText(MainActivity.this,
                    new HelloJni().stringFromJNI(),
                    Toast.LENGTH_LONG).show();
	} 
    ...
 
    static {
        System.loadLibrary("hello-jni");
    }
}

{% endcodeblock %}


[more video](https://www.youtube.com/watch?v=SeKXi-viRrk)
