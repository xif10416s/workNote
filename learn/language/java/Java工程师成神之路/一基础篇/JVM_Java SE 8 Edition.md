#The Java® Virtual Machine Specification
##Java SE 8 Edition
###Reference Types and Values
*   三种Reference类型：
    -   class 类型 - 值 指向动态创建的类实例
    -   array 类型 - 值 指向动态创建的数组
    -   interface 类型 -值 指向实现接口的类实例
*   null reference



###Heap 堆内存
*   jvm的堆内存被所有线程共享
*   堆内存是运行时数据内存区域，存放类的实例，和数组
*   数组或者实例无法在栈里面存储，因为frame创建后大小固定
*   jvm启动时，堆内存被创建
*   给堆使用的内存不需要连续
*   当对象太多，堆内存管理系统无法有效管理时，OutOfMemoryError


###Method Area 方法区
*   方法区被所有jvm线程共享
*   编译后的代码存储区域，每个class 结构
    -   类加载引用
    -   运行时常量Run time constant poll
        +   数值常量
        +   字段引用
        +   方法引用
        +   属性
    -   字段
        +   每个字段
            *   名称
            *   类型
            *   限定符
            *   属性
    -   方法数据
        +   每个方法
            *   名称
            *   返回类型
            *   参数类型
            *   限定符
            *   属性
    -   方法代码
        +   每个方法
            *   字节码
            *   操作栈大小
            *   本地变量大小
            *   本地变量table
            *   异常table
                -   每个异常处理
                    +   开始点
                    +   结束点
                    +   pc 代码执行点
                    +   被捕捉异常类的常量池索引
    -   方法和构造器的代码
*   在jvm启动时被创建
*   所有线程共享方法区，必须线程安全
*   可以固定，也可以动态扩展
*   OutOfMemoryError

###Run-Time Constant Pool 运行时常量池
*   在一个class文件中代表每个类或者每个接口运行时的常量表
*   每个常量池在jvm方法区分配
*   OutOfMemoryError，创建一个类或者一个接口构造常量池要求内存不够时


###Frames
*   用来存储数据和部分结果，方法返回值，发送异常，常量池引用
*   每次当方法调用的时候新的frame被创建
*   当方法调用结束的时候frame被销毁
*   线程的栈分配frame
*   结构：{Frame [ReturnValue] [LocalVariables[][][][]...] [OperandStack [][][]...] [ConstPoolRef] }

###reference
*   http://blog.jamesdbloom.com/JVMInternals.html
*   http://www.cnblogs.com/caca/p/jvm_stack_frame.html

###local variables
*   每个frame包含一组变量称为本地变量
*   frame的本地变量数组的长度在编译时被确定，代表的类或者接口中与方法相关的frame的二进制代码提供
*   一个本地变量持有类型：
    -   boolean
    -   byte
    -   char
    -   short
    -   int
    -   float
    -   reference
    -   returnAddress
*   一对变量可以持有long,double
*   本地变量由索引标识，从0开始
*   jvm使用本地变量在方法调用时传递参数
    -   类方法调用，传递连续的索引从0开始的本地变量数组
    -   实例的方法调用，索引为0的本地变量为当前方法调用的对象（也就是this),其他的参数从1开始


###Operand Stacks 操作栈
*   每个frame包含一个后进先出的栈称为操作栈
*   操作栈用来执行二进制代码指令，作用类似于本地cpu执行指令时用的寄存器
*   绝大部分时间都用在把jvm 二进制代码 对操作栈的压入，弹出，执行操作
*   指令操作二进制代码在本地数组变量和操作栈之间频繁操作
*   frame的操作栈的最大深度在编译时方法关联的frame的代码确定
*   frame被创建时，操作栈是空的
*   jvm提供说明，如何从本地变量或者字段中加载常量或者值到操作栈中
*   其他说明如，从栈中获取内容，操作内容，把结果放回操作栈
*   操作栈也被用来准备要传递给方法的参数，和接受方法的返回结果
*   instruction，说明如何操作栈，指令
    -   iadd,把两个int相加
        +   在之前的操作把两个值放在栈的顶端
        +   pop出这两个值
        +   相加
        +   把结果push会操作栈


###Dynamic Linking--方法符号到方法实际代码的映射
*   每个frame包含一个运行时常量池的引用，当前方法动态链接到方法代码
*   class文件中的方法的代码，方法的调用和变量的访问通过一些符号，动态链接就是将这些符号映射到实际代码
*   class编译后，所有变量和方法的引用被存储在常量池称为符号引用
*   符号引用是逻辑引用，不是指向实际内存地址


###Special Methods
*   init,初始化方法，由编译器提供


###Non-Heap Memory
*   Permanent Generation
    -   方法区
    -   interned strings
*   Code cache
    -   jit 编译的本地代码的编译与存储

###Just In Time Compilation
*   Java字节码的解释执行速度没有本地代码的执行速度快
*   为了提高性能，将热点java 字节码（经常调用，循环）编译成本地代码，存储在非堆内存


###对象创建与操作
*   类的实例和数组都是对象，使用不同的指令创建
*   类实例： new
*   数组：newarray,anewarry,multianewarray
*   访问类字段（static fields,类变量）:getstatic,putstatic
*   访问类实例字段（非static字段，实例变量）：getfield，putfield
*   加载数组到操作栈，：baload,caload,saload,iaload,laload,faload
*   存储在操作栈里面的值作为数组，bastore,castore


###指令集合
*   加载和保存指令-本地变量和操作栈之间传递
    -   加载本地变量：从本地变量加载到操作栈，iload,lload,fload,dload,aload
    -   保存本地变量：从操作栈获取值保存到本地变量，istore,lstore,fstore,astore
    -   加载常量：将常量加载到操作栈，bipush,sipush,ldc,
*   计算指令-在操作栈上的两个值的计算，并将计算结果放回操作栈
    -   两种主要的计算指令
        +   整数操作指令
        +   浮点数操作指令
*   类型转换指令-数字类型之间转换
*   条件控制指令
    -   条件分支：ifeq,ifne。。。
    -   符合条件分支：tablesswitch,lookupswitch
    -   非条件分支：got,jsr，ret
*   方法调用和返回指令
    -   invokevirtual（虚拟调用指令）：调用类实例方法，方法引用 
    -   invokeinterface（接口调用）：调用接口方法，查找运行时对象合适的实现方法
    -   invokespecial（特殊调用）：使用对私有方法，超类方法和实例初始化方法的特殊处理来调用实例方法。 
    -   invokestatic（静态调用）：调用类的方法
    -   invokedynamic（动态调用）：检查的主体过程是在运行期而不是编译
        +   每一处含有invokedynamic指令的位置都被称作“动态调用点（Dynamic Call Site）”，这条指令的第一个参数不再是代表方法符号引用的CONSTANT_Methodref_info常量，而是变为JDK 7新加入的CONSTANT_InvokeDynamic_info常量，从这个新常量中可以得到3项信息：引导方法（Bootstrap Method，此方法存放在新增的BootstrapMethods属性中）、方法类型（MethodType）和名称。引导方法是有固定的参数，并且返回值是java.lang.invoke.CallSite对象，这个代表真正要执行的目标方法调用。根据CONSTANT_InvokeDynamic_info常量中提供的信息，虚拟机可以找到并且执行引导方法，从而获得一个CallSite对象，最终调用要执行的目标方法上。
        +    http://www.infoq.com/cn/articles/Invokedynamic-Javas-secret-weapon
*   异常抛出指令
    -   jvm支持方法之间或者方法内一系列指令的同步，通过monitor
    -   方法级别的同步隐式执行
        -   同步方法可以通过运行时常量池的method_info 结构中的ACC_SYNCHRONIZED标记区分
        -   当调用ACC_SYNCHRONIZED标记的方法时，执行的线程进入一个monitor,调用方法，离开monitor,异常抛出是自动离开
    -   方法快同步，jvm提供monitorenter和monitorexit指令


###Thread 线程
*   jvm system threads,后台运行线程
    -   VM Thread ，该线程等待出现需要JVM到达安全点的操作 //TODO
    -   Periodic task thread,负责时间事件，计划执行周期操作
    -   GC threads,垃圾回收线程
    -   compiler threads，运行时编译二进制代码成本地代码
    -   signal dispatcher Thread，接受信号，发送给jvm处理
*   每个线程都有的组件：
    -   program counter(PC)
        +   每个线程有自己的pc寄存器
        +   如果当前方法不是naive方法，pc 寄存器包含了当前jvm执行指令的地址
        +   如果当前方法是native方法，pc寄存器的值不会被定义
        +   JVM 使用PC跟踪执行的指令
        +   PC实际上指向方法区的内存地址
    -   stack  栈
        +   每个线程有私有的栈，在创建线程的时候一起被创建
        +   为当前线程执行的每一个方法持有一个frame
        +   每次方法调用时会创建一个新的frame,添加到栈顶，方法执行完成（正常结束，或者异常）被移除
        +   栈的作用
            *   存储本地变量，在方法里面定义的变量（基本类型，或者引用地址）
            *   部分结果
            *   方法调用和返回
        +   jvm的栈内存不需要连续
        +   栈可以是固定大小或者有计算得到动态扩展
        +   每个固定大小的jvm栈的大小在被创建时独立设置
        +   与jvm stack相关的异常
            *   方法计算超过了允许的栈的大小，StackOverflowError
            *   如果栈是动态扩展的，当没有足够的内存用来扩展时，或者没有足够的内存创建一个新线程的栈内存时，OutOfMemoryError
        +   结构：{JVM Stack [Frame][Frame][Frame]... }  
    -   Native Method statcks 本地方法栈
        *   jvm使用c 栈，支持native方法，c语言的方法
        *   StackOverflowError，OutOfMemoryError  

###jvm主内存与工作内存
*   参考：http://blog.csdn.net/sc313121000/article/details/40266531
*   主内存=方法区+堆内存
*   线程工作内存 = 栈内存 + 对主内存变量拷贝的寄存器
*   JVM规范定义了线程对内存间交互操作：
    -    Lock(锁定)：作用于主内存中的变量，把一个变量标识为一条线程独占的状态。
    -    Read(读取)：作用于主内存中的变量，把一个变量的值从主内存传输到线程的工作内存中。
    -    Load(加载)：作用于工作内存中的变量，把read操作从主内存中得到的变量的值放入工作内存的变量副本中。
    -    Use(使用)：作用于工作内存中的变量，把工作内存中一个变量的值传递给执行引擎。
    -    Assign(赋值)：作用于工作内存中的变量，把一个从执行引擎接收到的值赋值给工作内存中的变量。
    -    Store(存储)：作用于工作内存中的变量，把工作内存中的一个变量的值传送到主内存中。
    -    Write(写入)：作用于主内存中的变量，把store操作从工作内存中得到的变量的值放入主内存的变量中。
    -    Unlock(解锁)：作用于主内存中的变量，把一个处于锁定状态的变量释放出来，之后可被其它线程锁定。
  


###Classloader 类加载
*   Jvm启动通过bootstrap classloader加载初始类
*   这个类链接并且初始化，在main函数调用之前
*   依次驱动其他的类和接口的加载、链接和初始化。
    -   loading,加载
        +   通过名称查找代表这个类或者接口类型的class文件，读取到byte数组，
        +   解析byte数组，确认他们所代表的类对象是否有正确的主次版本。
        +   任何类或者结构的直接父类也加载，
    -   linking,连接，一些列步骤
        +   verifying，验证
            *   确认类或者结构结构是否准确
                -   符号表symbol table 一致性和准确性
                -   final 方法 和类 没有被 重写
                -   方法访问控制关键字
                -   方法是否有正确的参数
                -   字节码不正确地操作堆栈
                -   变量在读取之前被初始化
                -   变量的值类型正确
        -   preparing,准备
            +   静态存储的内存分配以及JVM（如方法表）所使用的任何数据结构。
            +   静态字段的创建与初始化默认值
        -   resolving,解析,可选步骤
            +   检查符号引用是否正确
    -   initializaion，初始化
        +   执行类或者接口的一系列初始化方法
*   JVM各种类加载器
    -   bootstrap classloader,
        +   通常由native 代码实现，在 jvm 加载之前初始化
        +   负责加载基本java api，包括rt.jar
        +   只加载boot classpath发现的类，高信任度
    -   extension classloader
        +   加载标准扩展java api，如何 安全扩展
    -   System classloader
        +   默认类加载，从 classpath 加载 程序类
    -   user defined classloaders
        +   用户自定义类加载


##JVM编译
###指令格式
*   <index> <opcode> [ <operand1> [ <operand2>... ]] [<comment>]


###Switch 指令
*   tableswitch
    -   case中的值连续时，编译成tableswitch，解释执行时从table数组根据case值计算下标来取值，从数组中取到的，便是要跳转的行数。
*   lookupswitch
    -   当case中的值不连续时，编译成lookupswitch，解释执行时需要从头到尾遍历找到case对应的代码行。
    -   因为case对应的值不是连续的，如果仍然用表来保存case对应的行号，会浪费大量空间。
    



##JAVA class 格式
*   每个class文件由一系列8位字节流组成







