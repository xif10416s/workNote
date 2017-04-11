#基础
1.  scala 没有重载的操作
2.  '+ , - , * ,' and / can be used in method names
    1 + 2  ==> Int object 1 调用了 +这个方法 参数 是 2  
    (1).+(2)
3.   赋值操作被转换成调用update方法
    greetStrings(0) = "Hello"  == >greetStrings.update(0, "Hello")
4.   数组操作
    -   4.1 ==> 1 :: twoThree 在list前面加元素
    -   ::  是right 操作 ==> twoThree.::(1)
5.  函数写法:
    -   def add(b: Byte): Unit = sum += b  
    -   简写为def add(b: Byte) { sum += b },因为只有赋值操作 1  
6. Singleton objects
    -   6.1  当一个单例对象名字和类名相同时,称为类的 companion object
    -   6.2 可以相互访问私有变量
    -   6.3  singleton object 持有所有静态方法,并且是类的第一个对象, 不能接受参数
7. 符号意义
    -   7.1 =>
        转换  >> var increase = (x: Int) => x + 1
    -   7.2 _
        占位符,需要填入内容
        多个_的时候,顺序对应参数的位置
        代表这个参数列表  someNumbers.foreach(println _)  
        def sum(a: Int, b: Int, c: Int) = a + b + c    ==> val a = sum _
8.  闭包
    (x: Int) => x + 1  不是
1.  scala 可以 在 子类 把父类定义的方法 变为 变量
    -   Java’s four namespaces are fields, methods, types, and packages
    -   Scala’s two namespaces are:
        -   values (fields, methods, packages, and singleton objects)
        -    types (class and trait names)
    因为scala的变量和方法在一个namespace,所以无参数的方法可以被override称为val变量
2.  scala =>trait 不能定义带参数的构造函数
    -   trait NoPoint(x: Int, y: Int) // Does not compile

1.  推荐使用类型匹配 match 代替isInstanceOf 类型判断 和 asInstanceOf 类型转换
```
    def generalSize(x: Any) = x match {
        case s: String => s.length
        case m: Map[_, _] => m.size
        case _ => -1
    }
```
    - sion expr has type String, say, you write:
    expr.isInstanceOf[String]
    To cast the same expression to type String, you use:
      expr.asInstanceOf[String]
      

1. 范型匹配
```
    def isIntIntMap(x: Any) = x match {
           case m: Map[Int, Int] => true
           case _ => false
         }
    ／／＝＝＝＝没有类型保存，所以不能匹配范型＝＝＝＝
    isIntIntMap(Map("abc" -> "abc"))
    res20: Boolean = true
```

3. 数组到范型可以，数组有特殊处理，类型随值一起保存
```
def isStringArray(x: Any) = x match {
                 case a: Array[String] => "yes"
                 case _ => "no"
 }
```

##  模式匹配:
```
selector match {
  pattern1 => <body1>
  pattern2 => <body2>
  ...
}
```
###pattern总结起来大约以下几类：
*   Wildcard patterns // _ 统配
*   Constant patterns // 常量
*   Variable patterns // 变量
*   Constructor patterns // 构造函数
*   Sequence patterns // 比如List(,). 如果需要匹配剩余的话使用List(0,_*)
*   Tuple patterns // (a,b,c)
*   Typed patterns // 使用类型匹配 case a:Map[,]
*   asInstanceOf[]
*   isInstanceOf[]
*   note(dirlt):这里需要注意容器类型擦除.Array例外因为这个是java内置类型

###sealed关键字
*   sealed class必须和subclass在同一个文件内。
*   其修饰的trait，class只能在当前文件里面被继承
*   用sealed修饰这样做的目的是告诉scala编译器在检查模式匹配的时候，让scala知道这些case的所有情况，scala就能够在编译的时候进行检查，看你写的代码是否有没有漏掉什么没case到，减少编程的错误。

##List 类型:
*   list type 是covariant.协变的,如果S是T的子类,List[S] 也是 list[T]的子类
*   empty list 是 List[Nothing]的类型,Nothing是scala继承的最底层类,
*   val xs: List[String] = List() // 空list 是任何类型的子类
*   Nil是空list
*   List("apples", "oranges", "pears") <==>"apples" :: ("oranges" :: ("pears" :: Nil))
*   :: 是一个case class    x :: xs  ==> ::(x,xs) ==> xs.::(x)

##Scala的成员变量
*   var hour = 12
*   get 方法 hour
*   set 方法 hour_=
----------------
```
class Time {
    var hour = 12
    var minute = 0
}
```
-----------------
```
class Time {
    private[this] var h = 12
    private[this] var m = 0
    def hour: Int = h
    def hour_=(x: Int) { h = x }
    def minute: Int = m
    def minute_=(x: Int) { m = x }
}
```

##Scala函数的泛型特性——逆变与协变
```
trait Function1[-T, +U] {
  def apply(x: T): U
}
```
###“+”表示协变，而“-”表示逆变。
*   C[+T]：如果A是B的子类，那么C[A]是C[B]的子类。<==协变--作为返回值--返回该类或者子类
*   C[-T]：如果A是B的子类，那么C[B]是C[A]的子类。 <==逆变 --作为参数--输入该类或者父类
*   C[T]：无论A和B是什么关系，C[A]和C[B]没有从属关系。
Scala规定，协变类型只能作为方法的返回类型，而逆变类型只能作为方法的参数类型。

###协变
*   class A; class B; class C extends B;
*   val t2 = (p:A)=>new C  //定义A=>C类型的函数
*   val t3:A=>B = t2 //可以把 A=>C类型的函数赋值给 A=>B类型的

总结: 返回子类的变化(A=>C) 可以 转换(赋值) 为  转换父类的变化(A=>B)

###逆变
*   val f1 = (x:B)=>new A  //定义B=>A类型的函数
*   val f2 : C =>A = f1   //把B=>A类型的函数赋值给 C=>A 类型的

总结:输入参数父类B转换A的类型 可以 赋值 输入参数子类C转换A的类型

###类型边界
*   上边界   T <: Animal  => Animal的子类

##类型比较运算符
>    =:= (type equality) 
>    typeOf[List[java.lang.String]] =:= typeOf[List[Predef.String]]


##scala抽象成员
*   type
*   metohod
*   val
*   var
```
trait Abstract {
    type T
    def transform(x: T): T
    val initial: T
    var current: T
}

class Concrete extends Abstract {
    type T = String
    def transform(x: String) = x + x
    val initial = "hi"
    var current = initial
}
```

###初始化抽象vals
*   变量赋值在传入构造器之前
*   定义在子类的val变量,在父类初始化之后赋值

###未初始化字段出错解决
*    预定义字段==>在父类初始化之前初始化字段
*    Pre-initialized fields in an anonymous class expression
```
new {
    val numerArg = 1 * x
    val denomArg = 2 * x
} with RationalTrait

Pre-initialized fields in an object definition

object twoThirds extends {
    val numerArg = 2
    val denomArg = 3
} with RationalTrait
```
*   Lazy vals ==>使用的时候初始化
```
object Demo {
lazy val x = { println("initializing x"); "done" }
}
```


##Traits
*   基础代码重用单元
*   包含方法和字段
*   混合多个被class使用
*   拓展简单接口为复杂接口
*   相当于工具类
*   用 with 连接, 可以多个

###使用抽象类还是Traits
*   行为不会被重用,使用具体类
*   在多个不相关的类重用,使用trait, trait类似接口可以多继承
*   需要继承java代码,使用抽象类
*   关注效率的,使用类,java runtimes 调用类方法比接口方法快,Traits 需要编译成 interface 额外开销


##implicits
*   允许编译器自动选择处理类型不匹配的调用
*   必须定义 implicits 编译器才会使用
*   variable , function, object definition
*   需要import 指定的implicits,再import的范围有效
*   理解为自动调用的转换方法
*   只调用一次类型转换
*   只有在类型不匹配的时候起作用
*   命名的时候表示这是一个类型转换

###调用场景
*    赋值到一个预期的类型 val i:Int = 3.5   <== implict def doubleToInt(x:double) = x.toInt
*    接受的参数类型和方法的类型不匹配的情况
*    implicit parameters
*    自动添加缺失的参数列表,完成调用
*    在方法定义的参数列表的最后或者定义类的时候
*    def greet(name:String)(implicit prompt : PreferredPrompt) { prompt.XXX}
*    -Xprint:typer 

##for 语法
*   for (seq ) yield expr
*   seq ==> 一组,generators,definitions,filters,分号分割
*   expr 返回 一个数组
*   for (p <- persions;n = p.name ; if(n startsWith "To")) yield
*   所有for语句都可以转换为高级方法 map , flatMap,withFilter

##java集合类型转换
>import conversions.javaConversions._


## 偏函数
*   Created by xifei on 16-9-7.
*   偏函数是只对函数定义域的一个子集进行定义的函数。 scala中用scala.PartialFunction[-T, +S]类来表示
*   函数　y = f(x) ,每个ｘ都有对应的ｙ值
*   偏函数　ｘ不一定有对应的ｙ值
*   akka部分应用函数, 是指一个函数有N个参数, 而我们为其提供少于N个参数, 那就得到了一个部分应用函数.






