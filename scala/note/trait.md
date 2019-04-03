* 基础知识
    * 将trait作为接口使用
    * 在trait中定义具体方法
    * 在trait定义具体字段
    * 在trait中定义抽象字段

* 高级知识
    * 为实例对象混入trait
    * trait调用链
    * 在trait中覆盖抽象方法
    * 混合使用trait的具体方法和抽象方法
    * trait的构造机制
    * trait字段的初始化
    * 让trait继承类

## 将trait作为接口使用
 scala中的trait是一种特殊的概念，首先我们可以将trait当做接口来使用，此时的trait就与Java的接口非常类似。在Trait中可以定义抽象方法，就与抽象类中的抽象方法一样，只要不给出方法的具体实现即可。类可以使用extends关键字继承trait，注意，这里不是implement，而是extends，在scala中没有implement的概念，无论继承类还是trait，统一都是extends。类继承trait后，必须实现其中的抽象方法，实现时不需要使用override关键字。scala不支持对类进行多继承，但是支持多重继承trait，使用with关键字即可。
 ``` scala code
trait HelloTrait{
    def sayHello(name:String)
}
trait MakeFriendsTrait{
    def makeFriends(p:Person)
}
class Person(val name:String) extends HelloTrait with MakeFriendsTrait with Serializable{
    def sayHello(name:String) = println("Hello," + name)
    def makeFriends(p:Person) = println("Hello, my name is " + name + "your name is " + p.name )
}
```
   
## 在trait中定义具体方法
 scala中的trait可以不是只定义抽象方法，还可以定义具体方法，此时trait更像是包含了通用工具方法的东西。有一个专有的名词来形容这种情况，就是说trait的功能混入了类。举例来说，trait中可以包含一些很多类都通用的功能方法，比如打印日志等等，spark中就使用了trait来定义了通用的日志打印方法
 ``` scala code
trait Logger{
    def log(message:String) = println(message)
}
class Person(val name:String) extends Logger{
    def makeFriends(p:Person) {
        println("Hi,I'm " + name + "I'm glad to make friends with youk" +p.name) 
        log("makeFriends method is invoked with parameter Person[name =" + p.name + "]")
    }
}
```

## 在Trait中定义具体的字段
scala中的trait可以定义具体field，此时继承trait的类就自动获得了trait中定义的field。但是这种获取field的方式与继承class是不同的；如果是继承class获取的field，实际是定义在父类中的；而继承trait获取的field，就直接被添加到了类中

```scala code
trait Person {
    val eyeNum:Int = 2
}
class Student(val name:String) extends Person {
    def sayHello = println("Hi,I'm " + name + ", I have " + eyeNum + " eyes.")
}
```

## 在trait中定义抽象字段
scala中的trait可以定义抽象field，而trait中的具体方法则可以基于抽象field来编写，但是继承trait的类，则必须覆盖抽象field，提供具体的值
``` scala code
trait SayHello {
    val msg:String 
    def sayHello(name:String) = println(msg + "," + name)

}
class Person(val name:String) extends SayHello {
    val msg:String = "hello" 
    def makeFriends(p:Person) {
        sayHello(p.name)
        println("Hi,I'm " + name + "I'm glad to make friends with you" ) 
    }
}
```
----
## 为实例混入trait
有时我们可以在创建类的对象的时候,指定该对象混入某个trait，这样，就只有这个对象混入该trait的方法，而类的其他对象则没有。
``` scala code
trait Logged {
    def log(msg:String) {}
}
trait MyLogger extends Logged {
    override def log(msg:String) { println("log:" + msg)}
}
class Person(val name:String) extends Logged {
    def sayHello {
        println("Hi,I'm " + name);
        log("sayHello is invoked")
    }
}

scala> val p1 = new Person("leo")
p1: Person = Person@adb092

scala> p1.sayHello
Hi,I'm leo

scala> val p2 = new Person("Jack") with MyLogger
p2: Person with MyLogger = $anon$1@8fe7db

scala> p2.sayHello
Hi,I'm Jack
log:sayHello is invoked
``` 
## trait调用链
scala中支持让类继承多个trait后，依次调用多个trait中的同一个方法，要让多个trait的同一方法中，在最后都执行super方法即可。类中调用多哥trait中都有的这个方法时，首先会从最右边的trait方法开始执行，然后依次往左执行，形成一个调用链条，这种特性非常强大，其实就相当于设计模式中的职责链模式的一种具体实现
```scala code
trait Handler{
    def handle(data:String){}
}
trait DataValidHandler extends Handler {
    override def handle(data:String) {
        println("check data:" + data)
        super.handle(data)
    }
}
trait SignatureValidHandler extends Handler{
    override def handle(data:String) {
        println("check Signature:" + data)
        super.handle(data)
    }
}
class Person(val name:String) extends SignatureValidHandler with DataValidHandler{
    def sayHello {
        println("Hello," + name)
        handle(name)
    }
}
```

## 在trait中覆盖抽象方法
在trait中，是可以覆盖父trait的抽象方法的，但是覆盖时，如果使用了super方法的代码，则无法通过编译，因为super方法就会去调用父trait的抽象方法，此时子trait的该方法还是会被认为是抽象的。此时如果要通过编译，就得给子trait方法加上abstract override修饰
``` scala code
trait Logger{
    def log(msg:String)
}
trait MyLogger extends Logger {
    abstract override def log(msg:String){
        super.log(msg)
    }
}
```

## 混合使用trait的具体方法和抽象方法
在trait中可以混合使用具体方法和抽象方法，可以让具体方法依赖于抽象方法，而抽象方法则放到继承trait的类中去实现，这种trait其实就是设计模式中的模板设计模式的体现
```scala code 
trait Valid {
    def getName:String
    def valid:Boolean = {
        getName == "leo"
    }
}
class Person(val name:String) extends Valid {
    print(valid)
    def getName = name
}
```

## trait的构造机制
在scala中，trait也是有构造代码的，也就是trait中的，不包含在任何方法中的代码。从而继承了trait的类的构造机制如下
1. 父类的构造函数执行
2. trait的构造代码执行，多个trait从左到右依次执行
3. 构造trait时会先构造父trait，如果多个trait继承同一个父trait，则父trait只会构造一次
4. 所有trait构造完毕之后，子类的构造函数执行
``` scala code 
class Person {println("Person's constructor!")}
trait Logger {println("Logger's constructor!")}
trait MyLogger extends Logger {println("MyLogger's constructor")}
trait TimeLogger extends Logger {println("TimeLogger's constructor")}
class Student extends Person with MyLogger with TimeLogger{
    println("Student's constructor")
}

//result:
scala> var p = new Student
Person's constructor!
Logger's constructor!
MyLogger's constructor
TimeLogger's constructor
Student's constructor
p: Student = Student@702361
```
## trait field的初始化
在scala中，trait是没有接收参数的构造函数的，这是trait与class的唯一区别，但是如果需求就是trait能够对field进行初始化，该怎么办呢？只能使用scala中非常特殊的一种高级特性--提前定义
``` scala code
trait SayHello {
    val msg:String
    println(msg.toString)
}

class Person 
val p = new { val msg:String = "init"
} with Person with SayHello

class Person extends {
    val msg:String = "init"
} with SayHello {}

//另外一种方式就是使用lazy value
trait SayHello {
    lazy val msg：String = null
    println(msg.toString)
}
class Person extends SayHello{
    override lazy val msg:String = "init"
}
```

## trait继承class
在scala中，trait也可以继承自class，此时这个class就会成为所有继承该trait的类的父类
```scala code
class MyUtil {
    def printMessage(msg:String) = println(msg)
}
trait Logger extends MyUtil {
    def log(msg:String) = printMessage("log:" + msg)
}

class Person(val name:String) extends Logger {
    def sayHello {
        log("Hi,I'm " + name)
        printMessage("Hi,I'm " + name)
    }
}
```
