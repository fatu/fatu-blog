title: Scala Intro
date: 2014-09-11 06:34:37
tags:
categories: Scala
---
# Scala Intro
**Scala** is an object-functional programming and scripting language.


------

## **History**

The design of Scala started in 2001 at the École Polytechnique Fédérale de Lausanne (EPFL-洛桑联邦理工学院).
Scala was released internally in late 2003, before being released publicly in early 2004 on the Java platform, and on the .NET platform in June 2004. A second version of the language, v2.0, was released in March 2006. The .NET support was officially dropped in 2012.



### Author:  **Martin Odersky**

On 12 May 2011, Odersky and collaborators launched Typesafe Inc., a company to provide commercial support, training, and services for Scala.


## **Setup Env**

- [scala](http://www.scala-lang.org/)
- [sbt](http://www.scala-sbt.org/)

### **IDE**:
- Intellij IDEA
- Eclipse
- NetBeans

### REPL
- Scala Console
- WorkSheet
- Ensime
- Textmate/Sublime

## **Scala Syntactic Sugar**

- What's scala and why use it
- Reading File
- Creating lightweight immutable objects
- Anonymous classes
- Using Implicit conversions to create fluent interfaces
- Currying & Control Abstractions
- Applying tail recursion
- Creating XML
- Using traits
- Decorator using trait



### What's scala and why use it

- Object-Oriented Meets Functional
```scala
1.toString()

```

### Reading File

```scala
Source.fromFile("Main.scala").mkString
```

### Creating lightweight immutable objects

```scala
def createAgent() = { ("James", "Bond") }

val agent = createAgent()

println(agent._1 + " " + agent._2)
```

### Anonymous classes

```scala
def createAgent() = {
  new {
    val firstName = "James"
    val lastName = "Bond"
  }
}

val agent = createAgent()

println(agent.firstName + " " + agent.lastName)
```

### Using Implicit conversions to create fluent interfaces

```scala
import java.util._

class IntUnil(val number : Int) {
  def days() = this

  def ago() = {
    val today = Calendar.getInstance()
    today.add(Calendar.DAY_OF_MONTH, -number)
    today.getTime()
  }
}

implicit def int2IntUtil(number : Int) = new IntUnil(number)

println(2.days.ago)
```

### Currying

```scala
def mul(x: Int, y: Int) = x * y
def mulOneAtATime(x: Int) = (y: Int) => x * y
call:
mulOneAtATime(6)(7)
def mulOneAtATime(x: Int)(y: Int) = x * y
```

### Control Abstractions

```scala
def until(condition: => Boolean)(block: => Unit) {
  if (!condition) {
    block
    until(condition)(block)
  }
}

var x = 10
until (x == 0) {
  x -= 1
  println(x)
}
```

### Applying tail recursion

```scala
def factorial(n: Int): BigInt = {
  def factorial(n: Int, f: BigInt): BigInt = {
    if (n == 1)
      f
    else
      factorial(n - 1, n * f)
  }
  factorial(n, BigInt(1))
}

factorial(5)
factorial(10000)

```

### Creating XML

```scala
import scala.xml._

val stocks = Map("AAPL" -> 655, "GOOG" -> 756)

val xml = <symbols>{ stocks.map { createElementsForEachSymbol } }</symbols>

def createElementsForEachSymbol(element: (String, Int)) = {
  val (ticker, units) = element
  <symbol ticker={ticker}><units>{units}</units></symbol>
}

println(xml)

println("saving to file...")
XML save ("stocks2.xml", xml)
println("The # of symbols in the saved file is " +
  ((XML load "stocks2.xml") \\ "symbol").size
)
```

### Using traits

```scala
trait Friend {
  val name : String //emphasizes the class you mix in should have this
  def listen = println("I'm " + name + " listening...")
}

class Human(val name : String) extends Friend

def seekHelpFrom(friend : Friend) = friend.listen

val peter = new Human("Peter")
peter.listen
seekHelpFrom(peter)

class Animal(val name : String)
class Dog(override val name : String) extends Animal(name) with Friend

val rover = new Dog("Rover")
rover.listen
seekHelpFrom(rover)

class Cat(override val name : String) extends Animal(name)

val alf = new Cat("Alf")
//alf.listen  // Nope
//seekHelpFrom(alf) // Nope

//But you protest, your cat is a friend, OK, let's then selectively allow

val yourSweetCat = new Cat("Garfield") with Friend

yourSweetCat.listen
seekHelpFrom(yourSweetCat)
```

### Decorator using trait

```scala
abstract class Writer {
  def write(msg : String)
}

class StringWriter extends Writer {
  val target = new StringBuilder

  override def write(msg : String) = target.append(msg)
  override def toString = target.toString
}

def writeStuff(writer : Writer) = {
  writer write "This is stupid"
  println(writer)
}

writeStuff(new StringWriter)

trait UpperCaseFilter extends Writer {
  abstract override def write(msg : String) = super.write(msg.toUpperCase)
}

writeStuff(new StringWriter with UpperCaseFilter)


trait ProfanityFilter extends Writer {
  abstract override def write(msg : String) = super.write(msg.replace("stupid", "s*****"))
}

writeStuff(new StringWriter with ProfanityFilter)

writeStuff(new StringWriter with UpperCaseFilter with ProfanityFilter)

writeStuff(new StringWriter with ProfanityFilter with UpperCaseFilter)
```
---

## **BDD**
http://www.scalatest.org/
http://etorreborre.github.io/specs2/
https://code.google.com/p/scalacheck/
http://scalamock.org/

### Kata: PrimeFactor
[CodingDojo](http://codingdojo.org/)

---

## **Killer Application - Scala tech stack**
[Awesome-Scala](https://github.com/lauris/awesome-scala)

### Akka

```scala

import akka.actor.{ ActorRef, ActorSystem, Props, Actor, Inbox }
import scala.concurrent.duration._

case object Greet
case class WhoToGreet(who: String)
case class Greeting(message: String)

class Greeter extends Actor {
  var greeting = ""

  def receive = {
    case WhoToGreet(who) => greeting = s"hello, $who"
    case Greet           => sender ! Greeting(greeting) // Send the current greeting back to the sender
  }
}

object HelloAkkaScala extends App {

  // Create the 'helloakka' actor system
  val system = ActorSystem("helloakka")

  // Create the 'greeter' actor
  val greeter = system.actorOf(Props[Greeter], "greeter")

  // Create an "actor-in-a-box"
  val inbox = Inbox.create(system)

  // Tell the 'greeter' to change its 'greeting' message
  greeter.tell(WhoToGreet("akka"), ActorRef.noSender)

  // Ask the 'greeter for the latest 'greeting'
  // Reply should go to the "actor-in-a-box"
  inbox.send(greeter, Greet)

  // Wait 5 seconds for the reply with the 'greeting' message
  val Greeting(message1) = inbox.receive(5.seconds)
  println(s"Greeting: $message1")

  // Change the greeting and ask for it again
  greeter.tell(WhoToGreet("typesafe"), ActorRef.noSender)
  inbox.send(greeter, Greet)
  val Greeting(message2) = inbox.receive(5.seconds)
  println(s"Greeting: $message2")

  val greetPrinter = system.actorOf(Props[GreetPrinter])
  // after zero seconds, send a Greet message every second to the greeter with a sender of the greetPrinter
  system.scheduler.schedule(0.seconds, 1.second, greeter, Greet)(system.dispatcher, greetPrinter)

}

// prints a greeting
class GreetPrinter extends Actor {
  def receive = {
    case Greeting(message) => println(message)
  }
}

```
### Spray
### Play
### Slick
### Spark
