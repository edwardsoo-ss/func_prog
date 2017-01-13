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

## Functor and Monads in Java and Groovy
`map` and `flatMap` (they come in different names depending on the library used) are commonly seen methods. The classes defining `map` and `flatMap` represent some computational context, containing potentially one or more values. e.g. `java.util.stream`, `java.util.Optional`, `ratpack.exec.Promise`, `com.google.common.util.concurrent.ListenableFuture`, `scala.concurrent.Future`
They aim to make chaining of operations easier, so the developer can just declare the steps in the chain/pipeline.

