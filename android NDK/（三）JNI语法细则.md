
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
  
