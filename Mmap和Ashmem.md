Mmap在安卓源码库中就是对应`MappedByteBuffer.java`，C层的代码对应的是`MappedByteBuffer.c`

Ashmem在安卓源码库中就是对应`MemoryFile.java`（或者它引用的具体实现类`SharedMemory.java`），C层的代码对应的是`android_os_SharedMemory.cpp`

具体参考：[AnroidShareMemory](https://github.com/jakkypan/AnroidShareMemory)
