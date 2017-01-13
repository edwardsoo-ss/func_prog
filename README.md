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

# Key concepts in functional programming:
## First-class functions
Functions that can be assigned and passed as arguments to other functions, as do other primitive or object variables.

In Java, first-class functions are simulated using interfaces from `java.util.function`
```java
Function<Integer, Integer> doubleFn = (x) -> x * 2;
```

In Groovy, first-class functions are Groovy closures, weakly-typed objects
```groovy
Closure<Integer> doubleFn = { int i -> i * 2 }
```

In Scala, functions are strongly-typed objects
```scala
val doubleFn: Int => Int = _ * 2
```

## Higher-order functions:
Functions that take function(s) as argument(s)
Java:
```java
List<Integer> list = new ArrayList<>(Arrays.asList(1,2,3));
list.stream().map(doubleFn)
```

Groovy:
```groovy
[1,2,3].collect(doubleFn) // [2, 4, 6] 
```

Scala
```scala
List(1,2,3).map(doubleFn) // List(2, 4, 6)
```
