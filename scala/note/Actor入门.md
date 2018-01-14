# Actor入门
* Actor的创建，启动和消息收发
* 收发case class类型的消息
* Actor之间互相收发消息
* 同步消息和Future

>scala的Actor类似于java中的多线程编程。但是不同的是，scala的Actor提供的模型于多线程的有所不同。scala的Actor尽可能地避免锁和共享状态，从而避免多线程并发时出现资源争用的情况，进而提升多线程编程的性能。此外，scala Actor的这种模型还可以避免死锁等一系列的传统多线程编程的问题。<br>spark中使用的分布式多线程框架，是Akka。Akka也实现了类似Scala Actor的模型，其核心概念同样也是Actor。

## Actor的创建，启动和消息收发
>scala提供了Actor trait来让我们更方便地进行actor编程，actor trait类似于java中的Thread和Runnable一样，是基础的多线程基类和接口。我们只要重写Actor trail的act方法，即可实现自己的线程执行体，与java中重写run方法类似。此外，使用start（）方法启动actor；使用符号，想actor发送消息；actor内部使用receive和模式匹配接受消息。
```scala code 
import scala.actors.Actor

class HelloActor extends Actor {
    def act()
    while(true) {
        receive {
            case name:String => println("Hello, "+ name)
        }
    }
}

val helloActor = new HelloActor
helloActor.start()
helloActor ! "leo"
```

## 收发case class类型的消息
>scala的Actor模型与java的多线程模型之间，很大的一个区别就是，scala Actor天然支持线程之间的精准通信，即一个actor可以给其他actor直接发送消息。这个功能是非常强大和方便的。<br>要给一个scala发送消息，需要使用“actor！消息“的语法。在scala中，通常建议使用样例类，即case class来作为消息进行发送，然后在actor接收消息后，可以使用scala强大的模式匹配功能进行不同消息的处理。
``` scala code 
case class Login(username:String,password:String)
case class Register(username:String,password:String)
class UserManageActor extends Actor {
    def act() {
        while(true) {
            receive {
                case Login(username,password) => println("login,username is " + username + "password is " password)
                case Register(username,password) => println("Register,username is " + username + " password is " + password)
            }
        }
    }
}

val userManageActor = new UserManageActor
userManageActor.start()
userManageActor ! Register("leo","1234")
userManageActor ! Login("hua","2345")
```

## Actor之间相互收发消息
>如果两个Actor之间要互相发消息，那么scala的建议是，一个actor向另外一个actor发送消息时，同时带上自己的引用；其他actor收到自己的消息时，直接通过发送消息的actor的引用，即可以给它回复消息。
``` scala code 
case class Message(content:String,sender:Actor)
class leoTelephoneActor extends Actor {
    def act() {
        while(true) {
            receive {
                case Message(content,sender) => {
                    print("leo telephone:" + content)
                    sender ! "I'm leo, call me later"
                }
            }
        }
    }
}

class JackTelephoneActor extends Actor {
    def act() {
        leoTelephoneActor ! Message("Hello, leo, I'm jack",this)
        receive {
            case response:String => println("jack telephone:" + response)
        }
    }
}
```

## 同步消息和Future
>默认情况下，消息都是异步的；但是如果希望发送的消息是同步的，即对方接受后，一定要给自己返回结果，那么可以使用 !?  的方式发送消息。即`val reply = actor !? message`<br>
如果异步发送一个消息，但是在后续要获得消息的返回值，那么可以使用Future。即!!语法。`val future = actor !! message. val reply = future()`
