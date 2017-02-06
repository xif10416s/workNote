#MemoryManager
*   execution memory
    -    memoryFraction*safetyFraction=0.2*0.8=0.16   分配20% 最大使用 16%
    -    用于shuffles,joins,sorts,aggregations 操作的buffer
*    storeage memory
    -    memoryFraction*safetyFraction=0.6*0.9=0.54  分配60% 最大使用 54%
    -    用于cache RDD,boardcast,task result 缓存 = 0.8
    -    unroll memory
        -    反序列化block = 0.2
*   system memory
    -    memoryFraction=0.2 分配20%
    -    存储运行中产生的对象
*   UnifiedMemoryManager
*   StaticMemoryManager