---
layout: post
title: "JNI基础介绍"
date: 2015-09-18 14:57:30 +0800
comments: true
categories: Android JNI
---

这里简单介绍一下JNI的语法

 <!-- more -->

#### 初识第一面

##### HelloJNI.java

{% codeblock lang:java %}

public class HelloJNI {
   static {
      System.loadLibrary("hello"); // Load native library at runtime
                                   // hello.dll (Windows) or libhello.so (Unixes)
   }
 
   // Declare a native method sayHello() that receives nothing and returns void
   private native void sayHello();
 
   // Test Driver
   public static void main(String[] args) {
      new HelloJNI().sayHello();  // invoke the native method
   }
}

{% endcodeblock %}

##### HelloJNI.h/.c

{% codeblock lang:c %}

JNIEXPORT void JNICALL Java_HelloJNI_sayHello(JNIEnv *env, jobject thisObj) {
   printf("Hello World!\n");
   return;
}

{% endcodeblock %}

需要注意的有：

* 函数命名约定 `Java_{package_and_classname}_{function_name}(JNI arguments)`
* JNIEnv*: 对于JNI环境的引用, 通过这个入口可以进行jni函数
* jobject: 对于this对象的引用

#### JNI基础

JNI在native系统中定义了下面这些JNI数据类型，来对应Java的类型。

* Java基本类型：jint，jbyte，jshort，jlong，jfloat，jdouble，jchar，jboolean对应Java的int，byte，short，long，float，double，char，boolean
* Java应用类型：jobject对应java.lang.Object。同时还有一下子类型
 * jclass -> java.lang.Class
 * jstring -> java.lang.String
 * jthrowable -> java.lang.Throwable
 * jarray -> Java Array。也就是jintArray, jbyteArray, jshortArray, jlongArray, jfloatArray, jdoubleArray, jcharArray, jbooleanArray, jobjectArray.

由于native的函数都是接受JNI类型参数，返回JNI类型参数，所以一般情况下我们需要这样做：

* 把JNI的参数转换或者拷贝成C可以操作的类型，如jintArray到int[]
* 使用C类型参数完成计算
* 把结果转换或者拷贝成JNI类型并返回

JNI环境提供了大量的这些转换工作的工具可以帮助大家完成

#### 在Java和Nativie之间传递参数

##### 基础类型

{% codeblock lang:c %}

	// In "win\jni_mh.h" - machine header which is machine dependent
	typedef long            jint;
	typedef __int64         jlong;
	typedef signed char     jbyte;
 
	// In "jni.h"
	typedef unsigned char   jboolean;
	typedef unsigned short  jchar;
	typedef short           jshort;
	typedef float           jfloat;
	typedef double          jdouble;
	typedef jint            jsize;

{% endcodeblock %}

##### String

JNI环境提供了函数可以进行很方便的转换

* jstring -> char* : `const char* GetStringUTFChars(JNIEnv*, jstring, jboolean*)`.
* char* -> jstring : `jstring NewStringUTF(JNIEnv*, char*)`.

例如：

{% codeblock lang:c %}

JNIEXPORT jstring JNICALL Java_TestJNIString_sayHello(JNIEnv *env, jobject thisObj, jstring inJNIStr) {
   // Step 1: Convert the JNI String (jstring) into C-String (char*)
   const char *inCStr = (*env)->GetStringUTFChars(env, inJNIStr, NULL);
   if (NULL == inCSt) return NULL;
 
   // Step 2: Perform its intended operations
   printf("In C, the received string is: %s\n", inCStr);
   (*env)->ReleaseStringUTFChars(env, inJNIStr, inCStr);  // release resources
 
   // Prompt user for a C-string
   char outCStr[128];
   printf("Enter a String: ");
   scanf("%s", outCStr);    // not more than 127 characters
 
   // Step 3: Convert the C-string (char*) into JNI String (jstring) and return
   return (*env)->NewStringUTF(env, outCStr);
}

{% endcodeblock %}

##### primitive数组

* jintArray -> jint[], `jint* GetIntArrayElements(JNIEnv *env, jintArray a, jboolean *iscopy)`.
* jint[] -> jintArray, 首先分配内存`jintArray NewIntArray(JNIEnv *env, jsize len)`, 然后填充`void SetIntArrayRegion(JNIEnv *env, jintArray a, jsize start, jsize len, const jint *buf)`

{% codeblock lang:c %}

JNIEXPORT jdoubleArray JNICALL Java_TestJNIPrimitiveArray_sumAndAverage
          (JNIEnv *env, jobject thisObj, jintArray inJNIArray) {
   // Step 1: Convert the incoming JNI jintarray to C's jint[]
   jint *inCArray = (*env)->GetIntArrayElements(env, inJNIArray, NULL);
   if (NULL == inCArray) return NULL;
   jsize length = (*env)->GetArrayLength(env, inJNIArray);
 
   // Step 2: Perform its intended operations
   jint sum = 0;
   int i;
   for (i = 0; i < length; i++) {
      sum += inCArray[i];
   }
   jdouble average = (jdouble)sum / length;
   (*env)->ReleaseIntArrayElements(env, inJNIArray, inCArray, 0); // release resources
 
   jdouble outCArray[] = {sum, average};
 
   // Step 3: Convert the C's Native jdouble[] to JNI jdoublearray, and return
   jdoubleArray outJNIArray = (*env)->NewDoubleArray(env, 2);  // allocate
   if (NULL == outJNIArray) return NULL;
   (*env)->SetDoubleArrayRegion(env, outJNIArray, 0 , 2, outCArray);  // copy
   return outJNIArray;
}

{% endcodeblock %}

##### 访问对象变量并且调用Java方法

*访问实例对象*

* GetObjectID
* GetFieldID
* GetInt
* SetFieldID

{% codeblock lang:c %}

JNIEXPORT void JNICALL Java_TestJNIInstanceVariable_modifyInstanceVariable
          (JNIEnv *env, jobject thisObj) {
   // Get a reference to this object's class
   jclass thisClass = (*env)->GetObjectClass(env, thisObj);
 
   // int
   // Get the Field ID of the instance variables "number"
   jfieldID fidNumber = (*env)->GetFieldID(env, thisClass, "number", "I");
   if (NULL == fidNumber) return;
 
   // Get the int given the Field ID
   jint number = (*env)->GetIntField(env, thisObj, fidNumber);
   printf("In C, the int is %d\n", number);
 
   // Change the variable
   number = 99;
   (*env)->SetIntField(env, thisObj, fidNumber, number);
 
   // Get the Field ID of the instance variables "message"
   jfieldID fidMessage = (*env)->GetFieldID(env, thisClass, "message", "Ljava/lang/String;");
   if (NULL == fidMessage) return;
 
   // String
   // Get the object given the Field ID
   jstring message = (*env)->GetObjectField(env, thisObj, fidMessage);
 
   // Create a C-string with the JNI String
   const char *cStr = (*env)->GetStringUTFChars(env, message, NULL);
   if (NULL == cStr) return;
 
   printf("In C, the string is %s\n", cStr);
   (*env)->ReleaseStringUTFChars(env, message, cStr);
 
   // Create a new C-string and assign to the JNI string
   message = (*env)->NewStringUTF(env, "Hello from C");
   if (NULL == message) return;
 
   // modify the instance variables
   (*env)->SetObjectField(env, thisObj, fidMessage, message);
}

{% endcodeblock %}

*访问静态变量*

* GetStaticFieldID
* GetStatic<type>Field
* SetStatic<type>Field

{% codeblock lang:c %}

JNIEXPORT void JNICALL Java_TestJNIStaticVariable_modifyStaticVariable
          (JNIEnv *env, jobject thisObj) {
   // Get a reference to this object's class
   jclass cls = (*env)->GetObjectClass(env, thisObj);
 
   // Read the int static variable and modify its value
   jfieldID fidNumber = (*env)->GetStaticFieldID(env, cls, "number", "D");
   if (NULL == fidNumber) return;
   jdouble number = (*env)->GetStaticDoubleField(env, cls, fidNumber);
   printf("In C, the double is %f\n", number);
   number = 77.88;
   (*env)->SetStaticDoubleField(env, cls, fidNumber, number);
}

{% endcodeblock %}

*访问方法和静态方法*

* GetMethodID
* Call<type>Method
* GetStaticMethodID
* CallStatic<type>Method

{% codeblock lang:c %}

JNIEXPORT void JNICALL Java_TestJNICallBackMethod_nativeMethod
          (JNIEnv *env, jobject thisObj) {
 
   // Get a class reference for this object
   jclass thisClass = (*env)->GetObjectClass(env, thisObj);
 
   // Get the Method ID for method "callback", which takes no arg and return void
   jmethodID midCallBack = (*env)->GetMethodID(env, thisClass, "callback", "()V");
   if (NULL == midCallBack) return;
   printf("In C, call back Java's callback()\n");
   // Call back the method (which returns void), baed on the Method ID
   (*env)->CallVoidMethod(env, thisObj, midCallBack);
 
   jmethodID midCallBackStr = (*env)->GetMethodID(env, thisClass,
                               "callback", "(Ljava/lang/String;)V");
   if (NULL == midCallBackStr) return;
   printf("In C, call back Java's called(String)\n");
   jstring message = (*env)->NewStringUTF(env, "Hello from C");
   (*env)->CallVoidMethod(env, thisObj, midCallBackStr, message);
 
   jmethodID midCallBackAverage = (*env)->GetMethodID(env, thisClass,
                                  "callbackAverage", "(II)D");
   if (NULL == midCallBackAverage) return;
   jdouble average = (*env)->CallDoubleMethod(env, thisObj, midCallBackAverage, 2, 3);
   printf("In C, the average is %f\n", average);
 
   jmethodID midCallBackStatic = (*env)->GetStaticMethodID(env, thisClass,
                                 "callbackStatic", "()Ljava/lang/String;");
   if (NULL == midCallBackStatic) return;
   jstring resultJNIStr = (*env)->CallStaticObjectMethod(env, thisClass, midCallBackStatic);
   const char *resultCStr = (*env)->GetStringUTFChars(env, resultJNIStr, NULL);
   if (NULL == resultCStr) return;
   printf("In C, the returned string is %s\n", resultCStr);
   (*env)->ReleaseStringUTFChars(env, resultJNIStr, resultCStr);
}

{% endcodeblock %}

##### 创建java对象和对象数组

*创建java对象*

* FindClass
* NewObject
* AllocObject

{% codeblock lang:c %}

JNIEXPORT jobject JNICALL Java_TestJNIConstructor_getIntegerObject
          (JNIEnv *env, jobject thisObj, jint number) {
   // Get a class reference for java.lang.Integer
   jclass cls = (*env)->FindClass(env, "java/lang/Integer");
 
   // Get the Method ID of the constructor which takes an int
   jmethodID midInit = (*env)->GetMethodID(env, cls, "<init>", "(I)V");
   if (NULL == midInit) return NULL;
   // Call back constructor to allocate a new instance, with an int argument
   jobject newObj = (*env)->NewObject(env, cls, midInit, number);
 
   // Try runnning the toString() on this newly create object
   jmethodID midToString = (*env)->GetMethodID(env, cls, "toString", "()Ljava/lang/String;");
   if (NULL == midToString) return NULL;
   jstring resultStr = (*env)->CallObjectMethod(env, newObj, midToString);
   const char *resultCStr = (*env)->GetStringUTFChars(env, resultStr, NULL);
   printf("In C: the number is %s\n", resultCStr);
 
   return newObj;
}

{% endcodeblock %}

*创建java数组*

* NewObjectArray
* GetObjectArrayElement
* setObjectArrayElement

{% codeblock lang:c %}

JNIEXPORT jobjectArray JNICALL Java_TestJNIObjectArray_sumAndAverage
          (JNIEnv *env, jobject thisObj, jobjectArray inJNIArray) {
   // Get a class reference for java.lang.Integer
   jclass classInteger = (*env)->FindClass(env, "java/lang/Integer");
   // Use Integer.intValue() to retrieve the int
   jmethodID midIntValue = (*env)->GetMethodID(env, classInteger, "intValue", "()I");
   if (NULL == midIntValue) return NULL;
 
   // Get the value of each Integer object in the array
   jsize length = (*env)->GetArrayLength(env, inJNIArray);
   jint sum = 0;
   int i;
   for (i = 0; i < length; i++) {
      jobject objInteger = (*env)->GetObjectArrayElement(env, inJNIArray, i);
      if (NULL == objInteger) return NULL;
      jint value = (*env)->CallIntMethod(env, objInteger, midIntValue);
      sum += value;
   }
   double average = (double)sum / length;
   printf("In C, the sum is %d\n", sum);
   printf("In C, the average is %f\n", average);
 
   // Get a class reference for java.lang.Double
   jclass classDouble = (*env)->FindClass(env, "java/lang/Double");
 
   // Allocate a jobjectArray of 2 java.lang.Double
   jobjectArray outJNIArray = (*env)->NewObjectArray(env, 2, classDouble, NULL);
 
   // Construct 2 Double objects by calling the constructor
   jmethodID midDoubleInit = (*env)->GetMethodID(env, classDouble, "<init>", "(D)V");
   if (NULL == midDoubleInit) return NULL;
   jobject objSum = (*env)->NewObject(env, classDouble, midDoubleInit, (double)sum);
   jobject objAve = (*env)->NewObject(env, classDouble, midDoubleInit, average);
   // Set to the jobjectArray
   (*env)->SetObjectArrayElement(env, outJNIArray, 0, objSum);
   (*env)->SetObjectArrayElement(env, outJNIArray, 1, objAve);
 
   return outJNIArray;
}

{% endcodeblock %}

#####本地和全局变量

任何类似FindClass(), GetMethodID(), GetFieldID()返回的引用是一个本地引用。

想要使用全局引用的话，需要使用`NewGlobalRef()`，`DeleteGlobalRef()`

{% codeblock lang:c %}

// Global Reference to the Java class "java.lang.Integer"
static jclass classInteger;
static jmethodID midIntegerInit;
 
jobject getInteger(JNIEnv *env, jobject thisObj, jint number) {
 
   // Get a class reference for java.lang.Integer if missing
   if (NULL == classInteger) {
      printf("Find java.lang.Integer\n");
	  jclass classIntegerLocal = (*env)->FindClass(env, "java/lang/Integer");
      // Create a global reference from the local reference
      classInteger = (*env)->NewGlobalRef(env, classIntegerLocal);
      // No longer need the local reference, free it!
      //(*env)->DeleteLocalRef(env, classIntegerLocal);
   }

   if (NULL == classInteger) return NULL;
 
   // Get the Method ID of the Integer's constructor if missing
   if (NULL == midIntegerInit) {
      printf("Get Method ID for java.lang.Integer's constructor\n");
      midIntegerInit = (*env)->GetMethodID(env, classInteger, "<init>", "(I)V");
   }
   if (NULL == midIntegerInit) return NULL;
 
   // Call back constructor to allocate a new instance, with an int argument
   jobject newObj = (*env)->NewObject(env, classInteger, midIntegerInit, number);
   printf("In C, constructed java.lang.Integer with number %d\n", number);
   return newObj;
}
 
JNIEXPORT jobject JNICALL Java_TestJNIReference_getIntegerObject
          (JNIEnv *env, jobject thisObj, jint number) {
   return getInteger(env, thisObj, number);
}
 
JNIEXPORT jobject JNICALL Java_TestJNIReference_anotherGetIntegerObject
          (JNIEnv *env, jobject thisObj, jint number) {
   return getInteger(env, thisObj, number);
}

{% endcodeblock %}

