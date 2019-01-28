
Android下JNI开发
=======
这里假设已对C++语法有了一定的了解。

# 1、JNI协议
以下列出了JNI别名、java、C++类型的对应表    

![JNI协议]( https://github.com/h616016784/android_qesAndSumUp/raw/master/pic/jni.png )

# 2、android JNI目录下的c代码结构
```c++
 #include<stdio.h>
  #include<stdlib.h>
//必须引入jni.h的头文件，因为后面声明的方法中，会引用到jni.h中的类型；
#include<jni.h>
/**
 * Java_com_itheima_jnihelloworld_MainActivity_helloFromC
 * Java_类名(完整的类名,需要把.替换成_)_方法名
 *
 * JNIEnv* env：JNINativeInterface**(JNINativeInterface结构体的二级指针)
 *   (**env).GetVersion 调用结构体方法
 *   (*env).等价于env->所以(**env).等价于(*env)->
 * jobject obj：java中调用此方法的对对象。当前是：MainActivity.this对象
 */
jstring Java_com_itheima_jnihelloworld_MainActivity_helloFromC(JNIEnv* env,jobject obj){
	//  jstring     (*NewStringUTF)(JNIEnv*, const char*);
	char* text = "hello from C!!!";
	//(**env).NewStringUTF(env,text);
	return (*env)->NewStringUTF(env,text);
}

```

# 3、java与c之间的数据传递
 ## 3.1、在c语言中实现字符串运算
 在cpp文件中的主要代码如下：
 ``` C
JNIEXPORT jstring JNICALL Java_com_hhl_jnirundemo_JniPassParams_sayHelloInC
        (JNIEnv *env, jobject obj, jstring jstr){
    //把Java中的String转换成C语言中的char*

    char* cStr = _JString2CStr(env,jstr);
    char* text = " add zhangsan";
    strcat(cStr,text);
    // jstring     (*NewStringUTF)(JNIEnv*, const char*);

    return env->NewStringUTF(cStr);

}
/**
 * 把一个jstring转换成一个c语言的char* 类型.
 */
char* _JString2CStr(JNIEnv* env, jstring jstr) {
    char* rtn ;
    jclass clsstring = env->FindClass( "java/lang/String");
    jstring strencode = env->NewStringUTF("GB2312");
    jmethodID mid = env->GetMethodID( clsstring, "getBytes", "(Ljava/lang/String;)[B");
    jbyteArray barr = (jbyteArray)(*env).CallObjectMethod( jstr, mid, strencode); // String .getByte("GB2312");
    jsize alen = env->GetArrayLength( barr);
    jbyte* ba = env->GetByteArrayElements(barr, JNI_FALSE);
    if(alen > 0) {
        rtn = (char*)malloc(alen+1); //"\0"
        memcpy(rtn, ba, alen);
        rtn[alen]=0;
    }
    env->ReleaseByteArrayElements( barr, ba,0);
    return rtn;
}
 ```
 ## 3.2、 在c代码中实现数组运算
 ``` c++
 JNIEXPORT jintArray JNICALL Java_com_itheima_datapassdemo_JNI_arrElementsIncrease
  (JNIEnv *env, jobject obj , jintArray jArray){
	//目标：把java传过来的数组，给每一个元素加上10；

	//得到数组的长度
	int length = (*env)->GetArrayLength(env,jArray);

	//遍历赋值
	//数组的地址和数组的首地址一样，得到首地址
	// jint* (*GetIntArrayElements)(JNIEnv*, jintArray, jboolean*);
	int*  cArray = (*env)->GetIntArrayElements(env,jArray,JNI_FALSE);
	int i=0;
	for(i; i < length; i++){
		//把每一个元素取出来，在原有的基础上加上10
		//取值
		*(cArray + i) +=10;//当这个循环执行完毕后，Java中的数组就改变了
	}

	return jArray;
}

 ```
## 3.3、 在c代码中整数预算
``` c++
JNIEXPORT jint JNICALL Java_com_hhl_jnirundemo_JniPassParams_add
        (JNIEnv *env, jobject obj, jint x, jint y){
//在C中做加法运算，并把结果返回给Java显示

    int ruslt = x + y;
    printf("ruslt==%d\n",ruslt);
    return x + y;

}
```

# 4、综合案例使用
简单的登录验证和密码加密算法，只是抛砖引玉，可以知道如果只是单纯的用自己的代码，那么如果用ndk-build的话，每一个cpp文件都要在android.mk中配置，否则就会爆出undefined reference to的错误
android.mk文件内容：
```android.mk
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := JNISample
LOCAL_SRC_FILES := com_hhl_jnirundemo_JniPassword.cpp md5/md5.cpp
include $(BUILD_SHARED_LIBRARY)
```
jni目录下有md5文件，md5文件内有md5.h md5.cpp文件，加密算法在md5.cpp的文件内。

# 5、C语言回调java方法
其中前提要理解java反射的原理
## 5.1、调用java空方法
java类中的空方法及调用空方法的本地方法 
```java
public void helloFromJava(){
}
/**
 * 调用C语言代码，C代码回调helloFromJava方法
 */
public native void callbackHelloFromJava();

```
C语言中具体实现
```c++
/**
* 回调Java中JNI类中的HelloFromJava方法
*/
JNIEXPORT void JNICALL Java_com_hhl_jnirundemo_JniUtils_callbackHelloFromJava
        (JNIEnv *env, jclass) {

        //用反射
        //1.首先得到字节码 jclass  (*FindClass)(JNIEnv*, const char*);第一个参数是要反射类的那个类的完整名称；但是需要把.改成/
        jclass clazz = env->FindClass("com/hhl/jnirundemo/JniUtils");
        //2.得到方法jmethodID   (*GetMethodID)(JNIEnv*, jclass, const char*, const char*);
        //倒数第二个方法：要回调的方法名；最后一个参数是方法签名
        jmethodID methodID =env->GetMethodID(clazz,"helloFromJava","()V");
        //3.Utils实例对象jobject   (*AllocObject)(JNIEnv*, jclass);
        jobject  object =env->AllocObject(clazz);
        //4.执行方法
        //void   (*CallVoidMethod)(JNIEnv*, jobject, jmethodID, ...);
        env->CallVoidMethod(object,methodID);//调用Java中的JNI中的HelloFromJava()方法了
}
```
## 5.2、调用java中的带两个int参数的方法
java类中的空方法及调用空方法的本地方法 
```java
    // C调用java中的带两个int参数的方法
    public int javaAdd(int x, int y) {
        System.out.println("java中add：" + (x+y));
        return x + y;
    }
    /**
     * 调用C语言代码，C代码回调add方法
     */
    public native void callbackAdd();

```
C语言中具体实现
```c++
/**
* C回调Java中JNI.java中的add方法
*/
JNIEXPORT void JNICALL Java_com_hhl_jnirundemo_JniUtils_callbackAdd
        (JNIEnv *env, jclass) {

        //用反射
        //1.首先得到字节码 jclass  (*FindClass)(JNIEnv*, const char*);第一个参数是要反射类的那个类的完整名称；但是需要把.改成/
        jclass clazz = env->FindClass("com/hhl/jnirundemo/JniUtils");

        //2.得到方法jmethodID   (*GetMethodID)(JNIEnv*, jclass, const char*, const char*);
        //倒数第二个方法：要回调的方法名；最后一个参数是方法签名
        jmethodID methodID = env->GetMethodID(clazz,"javaAdd","(II)I");
        //3.Utils实例对象jobject   (*AllocObject)(JNIEnv*, jclass);
        jobject cObj = env->AllocObject(clazz);
        //4.执行方法
        //   jint     (*CallIntMethod)(JNIEnv*, jobject, jmethodID, ...);
        int x = 10;
        int y = 20;
        int cInt= env->CallIntMethod(cObj,methodID,x,y);//调用了Java中JNI.java中的add()方法

}
```
## 5.3、调用java中参数为string的方法

java类中的空方法及调用空方法的本地方法 
```java
/**
 * 调用C语言代码，C代码回调printString方法
 */
public native void callbackPrintString();

    // C调用java中参数为string的方法
    public void printString(String s) {
        System.out.println("java中输入的：" + s);
    }

```
C语言中具体实现
```c++
/**
* 调用Java中的JNI类的PrintString方法
*/
JNIEXPORT void JNICALL Java_com_hhl_jnirundemo_JniUtils_callbackPrintString
        (JNIEnv *env, jclass) {
        //1.得到对应类字节码
        // jclass (*FindClass)(JNIEnv*, const char*);//第一个参数是被调用对象的全类名；但是得把.换成/
        jclass  clazz= env->FindClass("com/hhl/jnirundemo/JniUtils");
        //2.得到对应的方法
        // jmethodID   (*GetMethodID)(JNIEnv*, jclass, const char*, const char*);
        //倒数第二个参数：方法名;最后一个参数是方法签名
        jmethodID  methodID =env->GetMethodID(clazz,"printString","(Ljava/lang/String;)V");

        //3.得JNI的对象，设执行方法的对象jobject  (*AllocObject)(JNIEnv*, jclass);
        jobject cObject= env->AllocObject(clazz);
        //4.执行方法  jint   (*CallIntMethod)(JNIEnv*, jobject, jmethodID, ...);
        //java String 和 C中的jstring;
        //创建jstring:jstring (*NewStringUTF)(JNIEnv*, const char*);
        char* text = " I am from C Data !!!";
        jstring cString =env->NewStringUTF(text);
        env->CallVoidMethod(cObject,methodID,cString);//C代码调用Java中JNI类的printString
}
```


