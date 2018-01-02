# Android JNI

1. ###JNI函数注册方式
	两种: 动态注册和静态注册

	通过注册才能使Java能够找到native定义的函数在c++ 库中的实现

2. ###静态注册
	1. 编写加载动态库和定义Native函数的java类
	2. 通过javac生成该类的class文件,
	   使用javah 生成对应的.h头文件

	3. 将文件放在项目的java同级目录JNI中
	4. 在JNI目录下编写c++代码来实现.h中定义的函数, 并在该目录下编写Android.mk(ndk根据mk文件定义的编译规则生成so库)和Application.mk(主要来定义适应的平台, x86, arm等)

		##### Android.mk 如下

		```bash
			   LOCAL_PATH := $(call my-dir)
			   include $(CLEAR_VARS)

			   LOCAL_MODULE := HelloJni
			   LOCAL_SRC_FILES := HelloJni.cpp

			   include $(BUILD_SHARED_LIBRARY)
		```
		##### Application.mk 如下
		```bash
			   #支持标准C++的一些特性
			   APP_STL := gunstl_static
			   APP_CPPFLAGS := -frtti -fexceptions

			   #支持的CPU架构
			   APP_ABI := armeabi-v7a
			   #Android版本
			   APP_PLATFORM := android-22
		```

	5. 在jni目录下, 调用ndk-build生成so库

	在AndroidStuido 下默认会将libs目录下的so库拷贝到jniLibs目录下

3. #### 动态注册
	使用jni函数RegisterNatives来注册函数

	在System.loadLibrary的时候, 会在C/C++文件中回调一个名为JNI_OnLoad的函数, 在该函数中调用 RegisterNatives函数将c++函数和java函数关联起来

	```java
		// native 方法实现

		jint get_random_num(){
			return rand();
		}
	```
	// 将要注册函数的列表, 放在一个JNINativeMethod类型的数组中

	JNINativeMetho 有三个参数

		1. java代码中定义的native函数名
		2. 签名()
		3. c/c++中对应函数的函数名(地址)

	  ```java
				static JNINativeMethod getMethods[] = {
					{"getRandomNum", "()I", (void*) get_random_num}
				}
				// 这个函数是有JNI在加载时调用
				JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {

					// 在虚拟机中获取JNIEnv , jni的环境指针

					JNIEnv env* = NULL;
					vm->getEnv((void**) &env, JNI_VERSION_1_8);


					// java中的类
					const char* className = "com/example/myjni/jniTest"

					// 通过名字来定位java的class对象
					jclass clazz;
					clazz = env->FindClass(className);

					// 注册方法列表
					env->RegisterNatives(clazz, getMethods, methodsNum);

				}
      ```

#### 参数对应表
		签名符号	JNI	java
		V	void	void
		Z	jboolean	boolean
		I	jint	int
		J	jlong	long
		D	jdouble	double
		F	jfloat	float
		B	jbyte	byte
		C	jchar	char
		S	jshort	short
		[Z	jbooleanArray	boolean[]
		[I	jintArray	int[]
		[J	jlongArray	long[]
		[D	jdoubleArray	double[]
		[F	jfloatArray	float[]
		[B	jbyteArray	byte[]
		[C	jcharArray	char[]
		[S	jshortArray	short[]
		L完整包名加类名;	jobject	class

		例如 String getString(int a ,long[] b) 函数的签名:
		 (I[J)Ljava/lang/String
