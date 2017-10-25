#Buffer
buffer是一个类似于Array的对象,主要用于操作字节.buffer是一个典型的js和c++结合的模块,它将性能相关的部分用c++实现,将非性能的相关部分用js实现.buffer所占用的内存不是通过v8分配的,属于堆外内存.
##Buffer对象
buffer对象类似于数组,它的元素师16进制的两位数,即0到255的数值. 不同编码的字符串所占用的元素个数各不相同,如中文在UTF-8编码下占用3个元素,字母占一个. 
buffer同Array类似,可以访问length,可以通过下标访问元素,构造对象new Buffer(100) (括号里面指定buffer的长度)
##buffer内存分配
buffer对象的内存分配不是在v8的堆内存中,而是在ndoe的c++层实现申请.  
node采用slab分配机制(一种动态内存管理机制)来申请内存,slab有如下三种状态:

- full: 完全分配状态
- partial: 部分分配状态
- empty: 完全没有被分配状态