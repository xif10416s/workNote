# spark version <2.0
##DataFrames
*   分布式数据集合
*   有列名称组织而成
*   对应于关系数据库的table 和 R/Python的dataframe

##Datasets--特殊编码序列化,可以不需要反序列化操作数据
*   强类型的数据集合,映射到relational schema
*   dataframe 是 类型为Row 的 dataset
*   可以并行函数转换处理 functional transformations (map, flatMap, filter, etc.).
*   DataSet的操作分为transformations 和 actions , Transformations 产生新的 DataSet , actions 执行计算返回结果
*   dataset的核心内容是encoder
*   Dataset用一个逻辑计划描述数据如何处理计算,action调用时,优化逻辑计划,生成物理计划并行高效执行
*   Encoder--把对象映射成spark内部类型==>case class Person(name:String,age:int)  ==> binary 结构  
*   降低内存,优化数据处理,可以操作序列化 的数据使用运行时生成的自定义bytecode序列化反序列化,加速,减少网络传输
*   与RDD相似，但是使用特殊序列化方式处理网络传输，encoders和标准序列化方式都是负责把对象转换成字节数组，但是encoders是动态生成code,使用特殊的格式，允许spark操作如 filtering,sorting 等转换操作时候不需要反序列化


