## `scoped_utf_chars.h` 和 `scoped_string_chars.h`

具体地址：

[scoped_string_chars.h](libnativehelper/header_only_include/nativehelper/scoped_string_chars.h)

[scoped_utf_chars.h](libnativehelper/header_only_include/nativehelper/scoped_utf_chars.h)

一般情况下，我们写jni的时候如果传入了string，则需要调用`GetStringUTFChars()`来转换成`char *`，并且在使用完后需要主动调用`ReleaseStringUTFChars()`来释放，否则会有内存泄漏。这样句有点麻烦，并且容易出错。

## `scoped_local_ref.h`

具体地址:

[scoped_local_ref.h](libnativehelper/header_only_include/nativehelper/scoped_local_ref.h)

当localref的声明期走完会主动去释放。原理跟上面的差不多，都是根据析构函数去释放。

如下的示例：

```c
ScopedLocalRef<jbyteArray> byteArray(env, env->NewByteArray(derLen));
if (byteArray.get() == nullptr) {
    JNI_TRACE("ASN1ToByteArray(%p) => creating byte array failed", obj);
    return nullptr;
}

ScopedLocalRef<jclass> rlimit_class(env, env->FindClass("android/system/StructRlimit"));
jmethodID ctor = env->GetMethodID(rlimit_class.get(), "<init>", "(JJ)V");
if (ctor == NULL) {
    return NULL;
}
```

## `scoped_local_frame.h`

具体地址：

[scoped_local_frame.h](libnativehelper/header_only_include/nativehelper/scoped_local_frame.h)

有必要知道当前有多少本地引用被使用，因为许多函数都返回本地引用，JNI需要设置本地引用的最大值。同时，如果创建了大对象的引用，就有耗尽可用存储器的风险。本地引用的一般函数有：

```c
jint EnsureLocalCapacity(jint capacity);
jint PushLocalFrame(jint capacity);       
jobject PopLocalFrame(jobject result);
```

JNI的规范指出，JVM要确保每个Native方法至少可以创建16个局部引用，经验表明，16个局部引用已经足够平常的使用了。但是如果要与JVM的中对象进行复杂的交互计算，就需要创建更多的局部引用了，这时就需要使用`EnsureLocalCapacity()`来确保可以创建指定数量的局部引用，如果创建成功返回0。

`PushLocalFrame()`与`PopLocalFrame()`是两个配套使用的函数对，它们可以为局部引用创建一个指定数量内嵌的空间，在这个函数对之间的局部引用都会在这个空间内，直到释放后，所有的局部引用都会被释放掉，不用再担心每一个局部引用的释放问题了。


## jni的引用类型

有3种引用类型：

* Local References
* Global References
* Weak Global References

[JNI引用类型](https://www.cnblogs.com/chenxibobo/p/6895484.html)


