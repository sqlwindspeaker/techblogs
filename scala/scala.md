# Scala

## 资料整理

https://windor.gitbooks.io/beginners-guide-to-scala/content/chp6-error-handling-with-try.html

https://www.yiibai.com/scala

https://blog.csdn.net/lovehuangjiaju/article/list/2



在线运行环境：https://scastie.scala-lang.org/



## 语法基础

1. main 函数的形式：

   ```scala
     def main(args: Array[String]) {
         println("Hello, world!") // prints Hello World
      }
   ```

2. 多行字符串，用三个双引号"""来包围的字符串；

3. Unit类型：类似于void返回值；Any类型：所有类型的超类；AnyRef：所有引用类型（简单理解为变量，因为还有一些值类型，比如2，3，这种）的超类，Null：空指针；Nothing：

4. val 定义常量；var 定义变量；变量名:类型 ＝ 初始值，其中等号和初始值是可选的。如果设置了初始值，可以使用编译器推断变量类型；

5. Scala 的for循环有点厉害：首先是语法

   ```scala
   for( var x <- Range ){
      statement(s);
   }
   ```

然后Range可以用各种序列生成器，比如1 to 10 代表[1, 10]；1 until 10 代表[1, 10)这种；然后更神奇的是可以多个范围：

```scala
for( a <- 1 to 3; b <- 1 to 3){
         println( "Value of a: " + a );
         println( "Value of b: " + b );
      }
```

结果是a和b的所有不同取值的组合；

然后还可以用过滤器：

```scala
 val numList = List(1,2,3,4,5,6,7,8,9,10);

      // for loop execution with multiple filters
      for( a <- numList
           if a != 3; if a < 8 ){
         println( "Value of a: " + a );
      }
```

6. Scala 中的break语句有点小诡异，不能直接像其他语言那样直接break，具体细节再看

7. Scala的函数名，是可以包含符号的，比如＋，＋＋，~，&，-，--，\，/，: 这些符号；函数语法定义为：

   ```scala
   def functionName ([list of parameters]) : [return type] = {
      function body
      return [expr]
   }
   ```

   但是当没有返回值时，也可以简化一些：

   ```scala
    def main(args: Array[String]) {
         println( "Returned Value : " + addInt(5,7) );
      }
   ```

   还有可变参数的函数：

   ```scala
   def printStrings( args:String* ) = {
         var i : Int = 0;
   
         for( arg <- args ){
            println("Arg value[" + i + "] = " + arg );
            i = i + 1;
         }
      }
   ```

8. 字符串插值，有两种插值方式，第一种用s标识，直接把变量插进去；第二种f插值，在$name%s这种，后面加printf的格式符；第三种特殊的插值raw插值器，表示不对字符串做转义替换

9. 鼓励使用val 而不是 var 

10. Scala中，不初始化变量是一个语法错误 

11. Scala和Java 不同的地方是，Scala没有Primitive Type，所有的类型都是class对象，不存在包装类型，编译器会自己做这种包装和解包装的操作（翻译到虚拟机上时） 

12. 运算符也是方法，方法名可以是这些运算符的符号 

13. Scala中的单参数的方法调用方式可以用空格，a method b 相当于a.method(b) 

14. Scala中，如果方法不带参数的话，可以省略括号 

15. companion objects（用来存放Java中类似静态方法和属性的地方), singleton objects, package objects

16. apply 方法，用来实现() 操作符的重载

17. Scala 常用模式：使用companion objects的apply方法，来实现构造对象

18. if/else 在Scala中不是语句，而是表达式，因此是有类型和返回值的，所以可以用 val s = if (x > 0) 1 else -1 这种表达式来代替?:操作符。如果两个分支返回的类型不同的话，则整个表达式的返回类型为各个分支类型的父类；如果有缺失的分支，则该分支返回类型为Unit

19. block 在Scala中不再是语句的集合，而是表达式的集合。其本身也是一个表达式，因此也是有值的，值就是最后一个表达式的值

20. 赋值表达式的值和Java中不太一样，它的返回值是Unit，因此，Scala中是不能写类似a = b = 3 这种表达式

21. 和Java不太一样的是，Scala中是可以有函数的（比如在方法内部，函数内部，而Java只能有静态方法）。对于函数，如果不是递归函数的话，可以省略返回类型，返回类型会被编译器根据=右边的表达式类型推断出来。对于返回类型是Unit的函数，可以省略返回类型和=，但是不建议这样用

22. Scala 的函数参数类型必须定义，但是可以有默认值（因此不能有重载？），在传递参数的时候还可以使用命名传递

23. 可变参数：定义时使用Int*作为参数的类型，编译器会将多个参数转换成Seq。但是，如果你想将一个Seq直接传递给函数，则需要使用: _\* 操作符，将一个Seq转换成多个参数。比如1 to 5: _\* 

24. Scala中没有checked exceptions，因此方法和函数不用声明异常。throw 表达式的返回值是Nothing，因此一个包含了异常抛出的if/else或者block中，返回值可能是Nothing

# 数组和Map，Tuple

1. Scala 的不可变数组，使用Array类，对应Java内置数组类型。比如Array[Int]对应int[]。
2. Scala 中的可变数组，类似Java中的ArrayList，使用ArrayBuffer类，可以使用+=来append元素进去
3. Scala 中的Map也分可变/不可变. Map默认是immutableMap。如果需要可变的Map，则需要使用mutable.Map类
4. Map内部其实是二元Tuple的组合，-> 操作符可以用来创建Tuple，比如 "alice" -> 3 相当于("alice", 3)
5. Map 可以用+=，-=来添加/删除元素。对于可变的Map，则直接在其自身上操作；对于不可变的Map，则会生成一个新的Map对象，因此变量需要使用var而不是val
6. 通过引入 scala.collection.JavaConversions._ 来进行Scala和Java之间的数据和Map转换
7. Tuple类型的元素引用，起始下标是1，而不是0.
8. 使用val (first, second, third) = t 这样的语法，可以对Tuple内的各个元素命名。如果只想要前面的几个元素，可以使用val (first, second, _) = t 这样的语法
9. Zippping方法，可以将多个数组元素组成一个zip后tuple的数组，比如val a = Array(1, 2, 3)  val b = Array("one", "two", "three"). 那么a.zip(b)的结果就是Array((1, "one"), (2, "two"), (3, "three"))
10. 



## 与Java不同点

1. 语句末尾的分号是可选的，在一行上有多个语句的时候才需要分号；
2. object 关键字，用来声明一个单例。它创建了一个类的同时，创建了一个单例对象，对象名和类名相同；并且Scala中没有静态对象的概念，scala中使用单例对象的方法来替代静态对象。因此，虽然object也同时创建了一个伴生类，但是不能用new去创建新的对象，否则就违反了单例的原则。此外，Object关键字声明的单例对象构造函数不能有参数
3. Scala的编译：使用scalac编译器，将scala文件编译成java的class文件。然后使用scala命令，执行对应的类文件即可
4. Scala可以和Java类无缝对接，因为二进制级别是兼容的。默认情况下，java.lang包下面的类是全部引入进来的，其他的需要手动引入。Scala中甚至可以直接继承Java的类或者实现Java的接口
5. import语句比java的功能强一些，可以使用{}来包含一个包下面的多个类；在java中，使用\*表示一个包下的所有类，但是在scala中，需要使用下划线_来代替，原因是\* 在scala中是一个合法的标识符（比如方法名）
6. Scala中一切东西都是对象，这点比java还纯粹。因为Java中，基本类型不是对象（虽然有自动打包可以解决绝大多数问题），并且函数不能被当作对象来处理。因此在Scala中，数字也是对象，像这种表达式：1+2*3/x，其实完全就是方法的调用，其中+, \* 和/都是方法名，这也说明\* 确实是一个合法的标识符。
7. 函数也是对象，因此函数可以被赋值给变量，并且当作参数传递
8. Scala中，函数无返回值使用Unit来表示（类似于void）
9. Scala中的类定义是可以带参数的，这被称为Scala类的主构造函数。是为了方便绝大多数类都只有一个构造函数这种实际情况而设计的语法糖；当需要有多个构造函数时，在类内通过this关键字可以重载构造函数 

```scala
class Student(id:Int){  
    def this(id:Int, name:String)={  
        this(id)  // 注意，构造函数调用其他构造函数的时候，要写在第一行，这点和Java类似
        println(id+" "+name)  
    }  
    println(id)  
}  

object Demo{  
    def main(args:Array[String]){  
        new Student(101)  
        new Student(100,"Minsu")  
    }  
}
```



1. Scala中还可以定义无参方法，和没有参数的方法不同的是，定义无参方法时不可以带括号，同时调用的时候也不带括号
2. Scala中的所有类的最终基类为：scala.AnyRef，在继承的过程中，可以使用override关键字，重写相关的方法
3. <span style='color:red'>Case Class是个很神奇的东西，需要花点时间专门看一下，一下子没看懂</span>
4. Trait，类似于Java中，可以包含部分实现的接口，或者类似于抽象类
5. Scala 中的泛型，使用[]来表示，比如List[T]
6. scala中，使用下划线_，来表示各种类型的默认值



# Pattern Match

Scala 中使用Pattern Match来完成Java中的switch功能，但是具有更强的功能

1. match 是一个表达式，不是一个语句（和if，block类似），因此，是有返回值的。因此可以用做

```scala
val sign = ch match {
    case '+' => 1
    case '-' => -1
    case _ => 0
}
```



2. match 中的case部分只会执行一个，因此不需要break语句；如果需要多个条件，可以使用|分割，比如

```scala
case "0"|"0x"|"0X" => "non-dec"
```

3. 除了多个条件之外，还可以在每个case上设置一个条件过滤，称作guards，比如:

```scala
var digits = xxxx
val sign = ch match {
    case '+' => 1
    case '-' => -1
    case _ if Character.isDigit(ch) => digits = Character.digit(ch, 10)
    case _ => 0
}
```

4. 在case语句中可以使用变量名，你可以把_当作一个通用的变量名，因此你也可以自己设置一个变量名，不过要注意的是，自定义的变量名要**小写字母开头**的，否则会被认为是一个常量。

```scala
variable match {
    case Pi => 3.14	// Pi is math.Pi
    case others => others * 1.414	// user defined variables
}
```

5. Scala 中的Pattern Match 可以根据类型来match，因此应该避免使用isInstanceOf 和 asInstanceOf操作符进行类型转换，比如

```scala
val number = obj match {
    case n: Int => n
    case s: String => Integer.parseInt(s)
    case _: BigInt => Int.MaxValue
    case BigInt => xxx // Warning: matching classOf[BigInt] object
    case _: => 0
}
```



要注意的是，进行类型匹配的时候，一定要有一个变量名（当然还是要小写字母开头），否则匹配的是BigInt的class类的常量。

还有要注意的是，因为Match是运行时的操作，而JVM在运行时是没有范型类的信息的，所以没有办法对范型类的具体类型做匹配

6. 内部元素的匹配：Scala中有一个提取器(Extractor)的概念，也就是说当一个类定义了unapply或unapplySeq方法的话，在进行match的时候，会根据需要，调用对应的提取器，将对象提取出一个Seq，并依次进行匹配比较和变量绑定的操作，比如：

```scala
// 数组的Match
val desc = arr match {
    case Array(0) => "The array contains only one element which is number 0"
    case Array(x, y) => "The array of length 2"
    case Array(0, _*) => "The array of first element is number 0"
    case Array(x, y, rest @ _*) => "extract first two elements as x, y. and other elements as rest"
}

// List的Match，一般使用 :: 操作符
val desc = lst match {
    case Nil => "empty list"
    case 0 :: Nil => "list with only one element which is number 0"
    case x :: y :: Nil => "list with only 2 elements"
    case 0 :: tail => "first element of list is number 0"
    case _ => "others"
}

// Tuple的Match
val desc = pair match {
    case (0, _) => "pair of first element is number 0"
    case (y, 0) => "pair of second element is number 0"
    case others => "neither element is 0"
}

// 但是如果匹配中有多个条件，则不能使用非_的变量名：
obj match {
    case (0, _) | (_, 0) => "OK, pair contains 0"
    case (0, x) | (x, 0) => "ERROR"
}
```

7. 正则表达式中的用法：Scala 可以说把语法糖做到了极致，正则表达式的捕获，甚至都可以用match来表示

```scala
val pattern = "([0-9]+) ([a-z]+)".r 
"100 students" match {
    case pattern(count, groups) => s"there're $count persons who are $groups"
}

正则表达式匹配之后，捕获的count和groups分别是100和"students"
```

8. Tuple的赋值其实也是一种匹配：

```scala
val (num, item) = (100, "students") // 看上去是一种tuple的赋值，其实等价于：

val result = (100, "students") match {
    case (num, item) => (num, item)
}

val num = result._1
val item = result._2

```

9. 因此，for循环中的赋值，其实也是一种匹配

```scala
for ((k, v) <- System.getProperties()) {
    xxxxx
}

// 因此，可以直接筛选想要的部分，比如
for ((k, "") <- System.getProperties()) {
    // value 为空字符串的会被获取到，其余部分自动忽略
}

// 也可以使用guard来实现
for ((k, v) <- System.getProperties() if k.startsWith("spark.")) {
    // 只获取spark.开头的的属性
}

```

10. Partial Function：这是一个Java中没有的概念，简单来说，它就是一个只对部分输入会返回结果的函数，或者说，它是一个不完备的case 语句集合

```scala
val f : PartialFunction[Char, Int] = { case '+' => 1; case '-' => -1 }
// 也就是说，这个函数只有在f('+')和f('-')的时候有正常的返回，其他的时候都会抛出MatchError
```

## Case Class

目前理解下来，Case Class的核心是一个用来方便实现Pattern Match的类，让我们对自定义数据类型也能方便的进行PatternMatch

Case Class的特点包括：

1. 构造函数中的参数默认为val，一般不会设置为var
2. 会同时有一个伴生对象，伴生对象上提供了apply方法，方便大家省略new 操作符构造对象
3. 提供了unapply方法，使得它可以用来进行pattern match
4. 提供了toString, equals, hashCode, copy方法

Case Class 一般是不用来继承其他Case Class的，一般的通用模式是，Case Class会继承一个Sealed 普通类，然后为每一个可能的结果都生成一个Case Class（类似于List，Option的实现方式）

对于一个不带参数的case class，意味着它只有唯一的实例，这样我们使用case object来实现

Case class还允许不带参数的构造函数，这样的case class被称为case Object：







## 多行字符串和stripMargin函数

scala中，通过三个引号可以构建跨行字符串。然而在跨行字符串中，有些情况我们需要字符串对齐。因此，可以在字符串中添加相应的边界符号（默认是'|'）。最后再通过stripMargin方法，将边界符去掉。如果需要手动设置边界符，则可以使用带参数的stripMargin("#")方法，来设置边界符为"#"



## scala中下划线的用法

#### 作为占位符

在一个匿名函数中，如果参数只用到一次的话，可以省去参数的声明部分，使用下划线 _ 来作为占位符表示函数的参数，比如，一个标准的匿名函数：

```scala
(x: Double) => 2 * x
```



可以简写为:

```scala
_ * 2
```

比如，如下两种用法是等价的：

```scala
(1 to 9).filter(_ % 2 == 0)
(1 to 9).filter((x) => x % 2 == 0)
```





当函数需要多个参数的时候，也可以这样做，比如 sort方法

```scala
List(10, 5, 8, 1, 7).sortWith(_ < _)
```

但是这里涉及到两个参数都只能用一次，并且顺序比较确定的情况下才适用。一般情况主要使用在sort和reduce这种操作中

#### 在模式匹配中作为通配符

有很多种用法，首先是类似于Java中switch语句的default值：

```scala
def matchTest(x: Int): String = x match {
    case 1 => "one"
    case 2 => "two"
    case _ => "many"
  }
```

其次是被Some包裹的时候，表示它是有一个值的，而不是None

```scala
Some(5) match { case Some(_) => println("Yes") }
```

还有就是在元组中作为通配符，比如：

```scala
match {
     case List(1,_,_) => " a list with three element and the first element is 1"
     case List(_*)  => " a list with zero or more elements "
     case Map[_,_] => " matches a map with any key type and any value type "
     case _ =>
 }
val (a, _) = (1, 2)
```

#### 特殊用法:_*

当使用不定长参数的函数时，Java是可以允许传任意的对象进去，或者传一个该对象类型的数组的；而在scala中，需要指定你传进去的参数是数组本身还是把数组中的元素传进去，比如：

```scala
List(1 to 5:_*) // 代表List(1, 2, 3, 4, 5)， 而不是List([1, 2, 3, 4, 5])
```



## Option, 使用Try+getOrElse体系来处理异常

Option 类有两种case class，None和Some(xxx)



可以使用match来判断：

```scala
objectOpt match {
    case None => "opt is empty"
    case Some(a) => s"opt contains value $a"
}
```

不过一般情况下，还是用getOrElse:

```scala
objectOpt.getOrElse("opt is empty")
```



更常用的一个用法是，将Option当作一个包涵0个或者1个元素的集合，对它使用循环或者map类的操作。当Option为None的时候，会自动忽略该操作



## 集合和列表的连接操作符

在Scala中，经常看到各种奇妙的+，++， ::和:::操作符，这些操作符来整理一下

* +: 和 :+ 操作符：其实就是在列表的前端和后端加入元素，返回一个新生成的列表；要注意的规律就是冒号永远靠近列表

* :: 和 ::: 操作符：读作cons，其中::是用来连接第一个数组，第二个是列表；也就是类似+:操作符。不过::操作符是可以用在模式匹配中的，比如：

  ```scala
  list match { 
      case Nil => "was an empty list" 
      case car :: cdr => "head was " + car + ", tail was " + cdr
  }
  ```

  当列表match到第二个case的时候，会把第一个元素命名为car，然后剩下的列表部分命名为cdr

* \+ 操作符和\+\+操作符，是用来做集合操作的。集合添加元素的时候可以用\+操作符，甚至于集合集合添加链表中的多个元素时。而如果是集合和集合合并，则使用\+\+操作符



## 类型体系和特殊类型

### Any, AnyRef, AnyVal

这里有两个问题，一个是Scala中的类型，还有一部分是Scala中的类型如何和JVM中类型对应的问题。

首先我们知道，JVM中是有基本类型的，这些类型不是Object类的子类，因此需要Wrapper来处理。而Scala

中是没有基本类型的，所有的类型都是引用类型，都是某类的对象。因此，单从Scala的语言层面上来说，所有类的基类是Any，Any的声明了如下方法：

```scala
final def ==(that: Any): Boolean
final def !=(that: Any): Boolean
def equals(that: Any): Boolean
def hashCode: Int
def toString: String
```

可以看出，因为Any定义了这些方法，所以所有的Scala的对象都可以进行比较，使用hashcode和toString方法。

Any有两个子类型，分别是AnyRef和AnyVal。其中：

* AnyVal是Scala中所有值类型的基类，一共有9个值类型，分别是 Byte、Short、Char、Int、Long、Float、Double、Boolean和Unit。这些类的特点是抽象并且final，不能派生，也不能通过new操作符创建新对象。
* AnyRef是JVM中Object类的别名，除值类型之外，其他的类都是从这个类中派生的。

简单总结一句，Scala中为了去除基本类型，引入了Any和AnyVal这两个类来实现这个目的。但是，Any和AnyVal在JVM中是没有对应的类型的。JVM中的最基本类型Object也就是Scala中的AnyRef

### Nothing

Nothing这个类比较特殊，Java中没有类似的概念，它本身是一个trait，继承自Any，代表所有类的子类，但是不能生成对象。

它的用法主要有两个，比如在泛型的时候

还有一种就是和Unit类似，作为函数的返回类型，表示永远不会返回的函数（比如抛出用来抛出异常的）

## 函数最后一行执行结果为默认返回值









