# An introduction to functional programming
For the Java/Groovy/Scala developer

### What is functional programming?
* Derived from lambda calculus, the system of expressing computation as evaluation of mathematical functions using values binding and substitution.
* Declarative programming; uses expression instead of imperatives statements.

### Why functional programming?
Referential transparency: a pure function will always return the same value given the same argument(s)
* Easier to reason, there is no internal/global opaque state to concern
* Trivial to parallelize, no concurrency control needed

Concise; delcarative functional code can express complex computation in few lines

### Why not pure functional programming?
* Real problem requires side effects
* Not all problem can be easily modeled as pure functions
* Performance; imperative solutions are sometimes more efficient for computer architecture we have
* Most languages have some elements of functional programming

# Some key concepts in functional programming found in Java and Groovy:
## First-class functions
Functions that can be assigned and passed as arguments to other functions, as do other primitive or object variables.

In Java, first-class functions are simulated using interfaces from `java.util.function`
```java
Function<Integer, Integer> double = (x) -> x * 2;
```

In Groovy, first-class functions are Groovy closures, weakly-typed objects
```groovy
Closure<Integer> double = { int i -> i * 2 }
```

In Scala, functions are strongly-typed objects
```scala
val double: Int => Int = _ * 2
```

## Higher-order functions:
Functions that take function(s) as argument(s)
Java:
```java
List<Integer> list = new ArrayList<>(Arrays.asList(1,2,3));
list.stream().map(double) // Stream of [2, 4, 6]
```

Groovy:
```groovy
[1,2,3].collect(double) // [2, 4, 6] 
```

Scala
```scala
List(1,2,3).map(double) // List(2, 4, 6)
```
## Currying
Transforming the evaluation of a function that takes multiple arguments into evaluations of a sequence of functions.

Java, currying has so little syntical sugar it is ugly to use:
```java
IntBinaryOperator add = (a, b) -> a + b;
IntFunction<IntUnaryOperator> curriedAdd = a -> b -> add.applyAsInt(a, b);
curriedAdd.apply(2).applyAsInt(3); // 5
```

Groovy:
```groovy
Closure<Integer> add = {x , y -> x + y }
Closure<Closure<Integer>> curriedAdd = {x -> {y -> add(x , y)}}
curriedAdd(2)(3) // 5
```

Scala:
```scala
val add: (Int, Int) => Int = (x, y) => x + y
val curriedAdd: Int => Int => Int = x => y => add(x, y)
curriedAdd(2)(3) // 5
```

## Immutability
immutable objects cannot be modified so they are thread-safe

### immutable value/variable/reference
In Java and Groovy this is done by `final`

In Scala this is done by value assignment vs variable assigment 
```scala
val x = 1
var y = 1
x = 2 // compile error
y = 2 // ok
```

### Immutable objects
Java string is immutable.
In Scala classes can be immutable or mutable
```scala
val map1 = Map(
  1 -> 2,
  2 -> 4
)
val map2 = map1.updated(3, 6) // produces a new map Map(1 -> 2, 2 -> 4, 3 -> 6), map1 is unmodified

val map3 = scala.collection.mutable.Map[Int,Int] = Map(1 -> 2)
map3.update(1,3) // modifies map3

```

## Lazy evaluation
The concept of delaying evaulation of an expression until its value is needed
* flow control (expression evalutaion using binding and substitution is usually done in a DFS fashion)
* allows evaluation of part of a potentially infinite data structure

In Java this is found in if/else and short-circuit evaluation:
```java
int x = (1 > 0) ? 1 : largest_prime(1 << 32); // largest_prime not evaluated
bool b = true || (largest_prime(1 << 32) > 1); // largest_prime not evaluated
```
In Groovy this is done by closure without argument
```Groovy
def number = 1 
def eagerGString = "value == ${number}"
def lazyGString = "value == ${ -> number }"

assert eagerGString == "value == 1" 
assert lazyGString ==  "value == 1"
```

In Scala, there are call-by-value and call-by-name parameters
```scala
def foo(a: =>  Unit): Unit = {
  print("world")
  a
}
def bar(a: Unit): Unit = {
  print("world")
  a
}
foo(print("hello")) // prints world hello
bar(print("hello")) // prints hello world
```

```scala
val fibs: Stream[Int] = 0 #:: 1 #:: (fibs zip fibs.tail).map{ t => t._1 + t._2 }
fibs(10) // 55
```

## Functors and Monads in Java and Groovy
Functor: any container data type `C[T]` that supports `map[T, U](c: C[T], f: T => U): C[U]`

Monad: any container data type `C[T]` that supports `flatMap[T, U](c: C[T], f; T => C[U]): C[U]`

`map` and `flatMap` (they come in different names depending on the library used) are commonly seen methods. The classes defining `map` and `flatMap` represent some computational context, containing potentially one or more values. e.g. `java.util.stream`, `java.util.Optional`, `ratpack.exec.Promise`, `com.google.common.util.concurrent.ListenableFuture`, `scala.concurrent.Future`
They aim to make chaining of operations easier, so the developer can just declare the steps in the chain/pipeline.

Zero elements: exceptions and undefined captured as value
```scala
maybeInt: Option[Int] = None
ints: List[Int] = List() // Nil
triedInt: Try[Int] = Try(throw new Exception())
eventualInt: Future[Int] = Future(throw new Exception())
```
most `map` and `flatMap` implementation defines the behavior of applying a transform to an empty container as no-op
```scala
def echo: Int => Try[Int] = i => Try{
  println(i)
  i
}
def fail: Int => Try[Int] = _ => Failure(new Exception())
Try(1).flatMap(failed).flatMap(echo) // Failure(java.lang.Exception), does not print
```

futures in Scala
```scala
trait Service[T, U] {
  def call(t: T): Future[U]
}

val serviceA: Service[Int, Int]
val serviceB: Service[Int, Double]
val serviceC: Service[Double, String]

// style 1
serviceA.call(1000)
  .flatMap(int =>serviceB.call(int))
  .flatMap(str => serviceC.call(str))
  .recover {
    case e: SocketTimeoutException => "500" 
  }

// style 2
val eventualStr = for {
  aInt <- serviceA.call(1000)
  bDou <- serviceB.call(aInt)
  cStr <- serviceC.call(bDou)
} yield cStr
eventualStr.recover {
  case e: SocketTimeoutException => "500"
}
```
