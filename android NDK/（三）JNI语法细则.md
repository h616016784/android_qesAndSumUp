
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
