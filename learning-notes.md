# Intro to Scala

Here are some notes I took during the Scala training.

- [Some Foundational Concepts](#some-foundational-concepts)
  - [Immutability is King](#immutability-is-king)
  - [Types & Type Signatures](#types--type-signatures)
- [Introductory Exercises](#introductory-exercises)
  - [Basics](#basics)
  - [Currying](#currying)
  - [Combining Curried Functions](#combining-curried-functions)
  - [Parametric Types](#parametric-types)
  - [Units are like Procedures](#units-are-like-procedures)
  - [What about if-then-else?](#what-about-if-then-else)
  - [String Interpolation](#string-interpolation)
  - [Tuples and Tuple Unpacking](#tuples-and-tuple-unpacking)
- [Algebraic Data Types (ADTs)](#algebraic-data-types-adts)
  - [Product Types in Scala are `case class`es](#product-types-in-scala-are-case-classes)
  - [Sum Types in Scala use `trait`s](#sum-types-in-scala-use-traits)
- [Types Exercises](#types-exercises)
  - [Defining a Person Product Type](#defining-a-person-product-type)
  - [Pattern Matching on a Person](#pattern-matching-on-a-person)
  - [Dot Notation](#dot-notation)
  - [Mutability and `.copy`](#mutability-and-copy)
  - [String Interpolation with Dot Notation](#string-interpolation-with-dot-notation)
  - [More complex `.copy` syntax](#more-complex-copy-syntax)
  - [Exhaustive Pattern Matching](#exhaustive-pattern-matching)
  - [Traits and Case Objects and Pattern Matching](#traits-and-case-objects-and-pattern-matching)
  - [Non-Exhaustive Pattern Matching == No Compile](#non-exhaustive-pattern-matching--no-compile)
- [List Exercises](#list-exercises)
  - [List Definition](#list-definition)
  - [Prepending to a list](#prepending-to-a-list)
  - [Appending to a list](#appending-to-a-list)
  - [The `head` and `tail` functions on lists](#the-head-and-tail-functions-on-lists)
  - [Pattern Matching for Lists](#pattern-matching-for-lists)
  - [Map](#map)
  - [Filter](#filter)
  - [FoldLeft](#foldleft)
  - [FoldRight](#foldright)
  - [Bringing List Concepts Together: FoldLeft + Pattern Matching + Case Classses](#bringing-list-concepts-together-foldleft--pattern-matching--case-classses)
- [Null Exercises](#null-exercises)
  - [Why Nulls Suck](#why-nulls-suck)
  - [Introducing `Option`al Types](#introducing-optional-types)
  - [`parseTrafficLightOrNull` but with `Optional`](#parsetrafficlightornull-but-with-optional)
  - [Using Pattern Matching and `map` to Safely Unpack Optionals](#using-pattern-matching-and-map-to-safely-unpack-optionals)
  - [The Map Type](#the-map-type)

## Some Foundational Concepts

### Immutability is King

In Scala, we deal with immutable values:

```scala
val a = 3
a = 4 // <- impossible
```

We turn things into **immutable facts**. Immutable data does not lie.

### Types & Type Signatures

Consider the following Java function signature:

```java
<T> T x(T t);
```

In Java, there are some serious no-nos which violate this type signature:

* We can return `null`, which isn't `T`:
  ```java
  <T> T x(T t) {
    return null
  }
  ```
* We can raise exceptions, which isn't `T`:
  ```java
  <T> T x(T t) {
    throw new RuntimeException();
  }
  ```
* We can peforms side-effects, like formatting my hard drive:
  ```java
  <T> T x(T t) {
    reformatHdd();
    return null;
  }
  ```
* We can infinitely recur, which ins't all that useful:
  ```java
  <T> T x(T t) {
    T(x);
  }
  ```

In Scala, it is impossible for any of the above to compile because:

* Nulls don't exist;
* Exceptions are really hard to throw;
* You can't perform un-expected side effects inline code.

#### Identity Functions are King

The identity function is the one that always compiles:

```java
<T> T x(T t){
  return t;
}
```

**The identity function is just a function that returns whatever it.**

The types tell you everything here. We have some type, `T`, for a function, `x`, which accepts a parameter `t` of type `T` and returns `t`. This type signature tells you everything that will happen.

Let's _humanise_ this:

```java
Human whoIs(Human person) {
  return person;
}
```

This is a **pure function**, because:

* For a given input there is a defined output
* Just like in maths
* No side-effects
* The types give you the guide rails

Replacing `Human` with a defined generic type `T` will mean that we have the identify function!

## Introductory Exercises

### Basics

Let's look at this code, where we define a function with `def`:

```scala
def add(x: Int, y: Int): Int = x + y
```

Here we have a function that will _always_:

* accept two `Int` parameters, `x` and `y`
* return an `Int`

It is impossible for this function to compile and do anything else. **This is key!** You can't have `null`s here, can't raise exceptions here, or can't perform alternative side effects!

We implement it with `x + y`; we don't need a `return` statement here because **the last expression in a function is always returned**.

To call this, we'd call it like any other number:

```scala
add(1, 3) // = 4
```

### Currying

Currying lets you **partially apply** a function using a _curried_ `def` syntax:

```scala
def addCurried(x: Int)(y: Int): Int = ???
```

This introduces the concept of _curried_ functions:

> A curried function is a function which takes multiple parameters one at a time, **by taking the first argument, and returning a series of functions which each take the next argument until all the parameters have been fixed**, and the function application can complete, at which point, the resulting value is returned.

Therefore, we add the result of the first function is _curried_ to the result of the section function.

The way we call this now is now different, this is a **fully-applied version** of this function:

```scala
addCurried(1)(2) // Returns Int
```

Here, we are applying two integers matching each of the two arguments. However, we can also **partially apply arguments** to this function:

```scala
addCurried(1) // Returns Int => Int
```

This is a **partially applied function**. Here, we have returned a function that accepts an `Int` and returns an `Int`.

Alternatively, we can define `addCurried` in _arrowed-syntax_ (a.k.a., as a function _value_ using `val` instead of using `def`):

```scala
val addCurriedOther: Int => Int => Int = x + y
```

We can then partially apply `addCurriedOther` by applying a single `Int` to each. When we partially apply, we return another function.

```scala
val add5: Int => Int = addCurriedOther(5) + 5
val add6: Int = add5(5) + 6
```

In `add5`, we return a function that accepts an `Int` and returns an `Int`. We **reduce** this down to its final value, `Int`; notice the reduction in return types:

* **`addCurriredOther`**: `Int => Int => Int`
* **`add5`**: `Int => Int`
* **`add6`**: `Int`

When we define curried functions using `def`, it is just syntactic sugar for the `val` approach.

### Combining Curried Functions

We can introduce functions as variables using the following:

```scala
def addCurried(x: Int)(y: Int): Int = x + y
// or
val addCurried: Int => Int => Int = x + y

def add5(x: Int): Int = {
  // Partially apply addCurried with just 1 argument
  val f: Int => Int = addCurried(x)
  // Now fully-apply f with just its 1 argument
  f(5)
}
```

This is saying `f` is of a type `Int => Int`, meaning its type is a function that accepts an `Int` returns an `Int`.

### Parametric Types

These can be like any types.

```scala
def foo[A](a: A): A = ???
```

This function can accept any type, e.g.:

```scala
foo[Int](a: Int): Int
foo[Float](a: Float): Float
```

This is the **identity function**. We can only return `a`:

```scala
def foo[A](a: A): A = a
foo[Int](100) // returns 100
```

If we defined

```scala
def bar(a: Int): Int = a
```

we can pass in:

```scala
bar(666) // returns 666
```

### Units are like Procedures

The `Unit` return type is has a side-effect:

```scala
def pandora(x: Int): Unit = {
  println("Hello, World " + Int)
}
```

This is like a non-returnable function (i.e., like a procedure in C that returns `void`). We can perform side-effects here, like printing to the console.

As we said earlier, the last expression in a function is always returned; the resulting type of a `println` statement is, therefore, a `Unit`.

### What about if-then-else?

Well Scala has if statements too. E.g., a function to multiply even numbers by two:

```scala
def timesTwoIfEven(x: Int): Int = {
  // if even
  if (x % 2 == 0) {
    x * 2
  } else {
    x
  }
}
```

Keeping in mind that there is no `return`, as in Scala an if statement is an expression. It is more appropriate, then, to have these in single lines:

```scala
def timesTwoIfEven(x: Int): Int = if (x % 2 == 0) x * 2 else x
```

### String Interpolation

Let's write a function to return a `String` interpolated with an `Int`:

```scala
def showNumber(x: Int): String = "The number is " + x
```

We can also use string templates using an `s` prefix, similar to formatted strings with `f` in Python.

```scala
def showNumber(x: Int): String = s"The number is $x"
```

### Tuples and Tuple Unpacking

We can also 'zip' values togetehr into a Tuple type. E.g., a tuple of a `String` and `Int` would be expressed as `(String, Int)`:

```scala
val nameAndAge: (String, Int) = ("Alex", 27)
```

This, of course, can be used in a function, such as one to `pair` them together:

```scala
def pair(name: String, age: Int): (String, Int) = (name, age)
```

Here we have a return type `(String, Int)`.

We can extract or **unpack** elements from the tuples too using `_` to ignore elements we do not want. E.g., if we just wanted the first element from our tuple (i.e., name), we can do:

```scala
def first(pair: (String, Int)): String = {
  val (name, _): String = pair
  name
}
```

Similarly, we can do the same for the age element using index-dot notation (to reference the index). Note that this is _not_ zero-based:

```scala
def second(pair: (String, Int)): Int = pair._2
```

You can have [up to 22 elements inside tuples](https://www.scala-lang.org/api/2.12.0/scala/Tuple22.html), although you'd probably want to use another data type!

## Algebraic Data Types (ADTs)

ADTs are types which are composed of other types.

Let's visualise this; imagine the primitive types as coloured circles, where:

* :red_circle: = `Int`;
* :blue_circle: = `String`;
* :green_circle: = `Char`;
* :yellow_circle: = `Double`

There are two main ADTs, Product Types (AND) and Sum Types (OR).

* **Product Types (AND)**<br>
  Combine new types using an `and`:
  ![](https://i.imgur.com/rDwHz9s.png)
* **Sum Types (OR)**<br>
  Combine new types using an `or`:
  ![](https://i.imgur.com/39xpouZ.png)
  Famous example is traffic lights. :vertical_traffic_light:
* **A Combined Sum and Product Type**:<br>
  ![](https://i.imgur.com/39xpouZ.png)
* **Something whacky:**<br>
  ![](https://i.imgur.com/WMdRZTw.png)

### Product Types in Scala are `case class`es

An example of a **product type** in Scala would be a **`case class`**. Case classes combine individual members into one type.

For example, a new `Coordinates` case class which is a product of two `Int`s:

```scala
case class Coordinate(x: Int, y: Int)
```

Use `case class`es when we want to add additional information to existing data types (i.e., members). This is analogous to `struct`s in C.

#### `case class` vs `class`

From this [webpage](https://www.scala-exercises.org/scala_tutorial/classes_vs_case_classes), creating a class instance requires the keyword `new`, whereas this is not required for case classes.

Also, we see that the case class constructor parameters are promoted to members, whereas this is not the case with regular classes.

It feels that standard classes are more Java-esque.

```scala
class BankAccount {
  private var balance = 0

  def deposit(amount: Int): Unit = {
    if (amount > 0) balance = balance + amount
  }
  def withdraw(amount: Int): Int =
    if (0 < amount && amount <= balance) {
      balance = balance - amount
      balance
    } else throw new Error("insufficient funds")
}

case class Note(name: String, duration: String, octave: Int)

val aliceAccount = new BankAccount // Note the "new"
val c3 = Note("C", "Quarter", 3)
```

### Sum Types in Scala use `trait`s

Example of a **sum type** in Scala use **`trait`**s. These traits can then be used by `case class`es or `case object`s to **`extend`** these classes or objects, thereby linking them as an _or_.

For example, `place`, `move` or `rotate`, which all extend `Command`:

```scala
trait Command
case class Place(x: int, y: Int) extends Command
case object Move extends Command
case object Rotate extends Command
```

#### Pattern Matching

We can use pattern matching to 'extract' a type and perform an action:

```scala
trait TrafficLight
case object Red extends TrafficLight
case object Yellow extends TrafficLight
case object Green extends TrafficLight

val something: TrafficLight
something match {
  case Red    => "Stop!"
  case Yellow => "Get ready to slow down"
  case Green  => "Go go go!"
}
```

You can think of this like a `switch` statement in C, but on types, to perform specific actions on specific types:

```scala
def perform(command: Command): String => {
  command match {
    case Place(x, y) => s"command was place at ${x}, ${y}"
    case Move        => "command was move"
    case Rotate      => "command was rotate"
  }
}
```

Helps checks for errors to prevent bugs, e.g. the following does not compile:

```scala
perform("Banana") // won't compile as "Banana" is a String, not a Command
```

## Types Exercises

### Defining a Person Product Type

Let's define a `Person` product type, where a `Person` is a **"product"** of `String` and `Int`:

```scala
case class Person(name: String, age: Int)
```

We can then create 'instances' (two variables) of the `Person` by invoking the constructor on `Person`:

```scala
val alex = Person(name = "Alex", age = 27)
val jake = Person(age = 26, name = "Jake")
```

We can use **named parameters** here, so it ordering does not matter.

### Pattern Matching on a Person

Lets look at this `tellMeAbout` function, which accepts a `Person` type and returns a `String` that describes them:

```scala
def tellMeAbout(person: Person): String = {
  person match {
    case Person(name, age) => s"$name is $age years old"
  }
}
```

The pattern matching will match against the `Person` type and unpack their `name` and `age`:

```
tellMeAbout(alex) // "Alex is 27 years old"
tellMeAbout(jake) // "Jake is 26 years old"
```

### Dot Notation

We can also use dot notation to access sub-attributes within the

```scala
def showPerson2(person: Person): String =
  s"${person.name} is ${person.age} years old"
```

Note: You need to use **curly braces** to access dot notation. Therefore it is advised to always use them for string interpolation.

### Mutability and `.copy`

We can't modify a `person` as arguments are immutable.

However, we can use the `.copy` method to copy them; this is part of the builtin `case class`.

For example, if we want to change the person's age, we can copy the person and update their age by explicitly passing in their `newAge`:

```scala
def changeAge(newAge: Int, person: Person): Person = person.copy(age = newAge)
```

Alternatively, we can invoke the `Person` constructor again:

```scala
def changeAge(newAge: Int, person: Person): Person = Person(person.name, newAge)
```

The beautiful part of `.copy`, we only need to copy over the one variable, which saves us time!

### String Interpolation with Dot Notation

Let's consider this wallet:

```scala
case class Wallet(amount: Double)
```

We can define methods to extract values from this case class. E.g., printing out the `amount` in a `wallet` of $20.50:

```scala
val wallet = Wallet(20.5)
def showWallet(wallet: Wallet): String = s"The wallet amount is ${wallet.amount}"
showWallet(wallet) // The wallet amount is 20.50
```

### More complex `.copy` syntax

Let's say we want to make a purchase with our wallet, which will take a `Wallet` with its cost (a `Double`), returning a new `Wallet` with the new amount:

```scala
def purchase(cost: Double, wallet: Wallet): Wallet = {
  val newAmount: Double = wallet.amount - cost
  wallet.copy(amount = newAmount)
}
```

Of course, this can be a one liner:

```scala
def purchase(cost: Double, wallet: Wallet): Wallet = wallet.copy(amount = wallet.amount - cost)
```

### Exhaustive Pattern Matching

Let's say we have want to have a function to print out only valid :vertical_traffic_light:. It should return `"The traffic light is invalid"` unless `"red"`, `"yellow"`, or `"green"` are provided, in which case it returns `"The traffic light is` and the relevant colour.

Here we can use pattern matching with a default catch-all for all other cases using the `_`:

```scala
def showTrafficLightStr(trafficLight: String): String = {
  trafficLight match {
    case "green" | "red" | "yellow" => s"The traffic light is $trafficLight"
    case _                          => "The traffic light is invalid"
  }
}
```

We can also using a pipe as an `or` for each of the three colours too!

### Traits and Case Objects and Pattern Matching

Remember, a **`trait`** is an Sum Type ADT. We can **`sealed`** the trait, meaning it can only be extended in the same file that it is defined. We can also have **`case object`**s to `extend` the trait we define, making them _singletons_ of those traits.

In the traffic light analogy, we could, instead of using strings, replace our `String`s above into actual `TrafficLight` types.

```scala
sealed trait TrafficLight

case object Red extends TrafficLight
case object Yellow extends TrafficLight
case object Green extends TrafficLight
```

The beauty of this is that we don't need the catch-all `_` default in the `showTrafficLightStr` function, since it is now impossible to provide anything but `Red`, `Yellow`, or `Green` case objects:

```scala
def showTrafficLight(trafficLight: TrafficLight): String = {
  trafficLight match {
    case Red => "The traffic light is red"
    case Yellow => "The traffic light is yellow"
    case Green => "The traffic light is green"
  }
}
```

This technique helps you make invalid states/values irrepresentable in your programs.

### Non-Exhaustive Pattern Matching == No Compile

Lets say we introduce a new traffic light case object, `Flashing`. Our above pattern match would no longer work since the above pattern match is not exhaustive (i.e., it no longer covers the new `Flashing` case):

```scala
// Lets introduce this new guy...
case object Flashing extends TrafficLight

// The following cannot compile, as it is not exhaustive!
trafficLight match {
  case Red => "The traffic light is red"

  case Yellow => "The traffic light is yellow"
  case Green => "The traffic light is green"
}

// Once we introduce the 4th case, it will compile:
trafficLight match {
  case Red => "The traffic light is red"
  case Yellow => "The traffic light is yellow"
  case Green => "The traffic light is green"
  case Flashing => "The traffic light is flashing" // <-- Now exhaustive!
}
```

## List Exercises

### List Definition

In Scala, lists are:

* immutable
* linked lists
* have the same data type
* is a recursive data structure

Lists in Scala are a Sum type ADT:

```scala
sealed trait List[+A]
case class ::[A](head: A, tail: List[A]) extends [A]
case object Nil extends List[Nothing]
```

The `::` is the case class, often known as the _cons operator_, which is a product type composed of:

1. the head, of type `A`
2. a tail, a List of type `A`

The singleton case class object, `Nil`, refers to an empty List. It is generally used to refer to the _sentinel value_ of the list, i.e.:

```scala
val listWith1: List = 1 :: Nil // <- Nil here will 'terminate' the list
```

The following lists have all the same values:

```scala
val list1: List = List(1, 2, 3)
val list2: List = 1 :: 2 :: 3 :: Nil
val list3: List = 1 :: (2 :: (3 :: Nil)
val list4: List = ::(1, ::2, (::3, Nil)))
```

Note, we defined List with `[+A]`. This refers to Covariance and contravariance. Read more [here](https://blog.knoldus.com/covariance-and-contravariance-in-scala/).

### Prepending to a list

To prepending to a list, we just need to add the new element the `head` of the list, providing the original list into the `tail`:

```scala
def prependToList[A](x: A, xs: List[A]): List[A] = ::(head=x, tail=xs)
```

Obviously this syntax is a little nasty, but it is explicit to show what is going on here. We can make it a lot easier by dropping the variable

```scala
def prependToList[A](x: A, xs: List[A]): List[A] = x :: xs
```

You can also use the left-applicative `+:` operator:

```scala
0 +: List(1,2,3) // => List(0,1,2,3)
```

### Appending to a list

To append to the end of the list, we can use

```scala
def appendToList[A](x: A, xs: List[A]): List[A] =
```

You can also use the right-applicative `:+` operator:

```scala
List(0,1,2) :+ 3 // => List(0,1,2,3)
```

### The `head` and `tail` functions on lists

Just think of this guy:

![](https://i.imgur.com/zH5nVSP.png)

The `head` of a list is always the first element:

```scala
List(1,2,3).head // 1
```

The `tail` of a list is always the whole list without its first element:

```scala
List(1,2,3).tail // List(2,3)
```

The `isEmpty` method on a is self-explanatory :wink:

### Pattern Matching for Lists

For implementing a `isEmptyList` function or `showListSize` function, we can use pattern matching against `Nil`:

```scala
def isEmptyList[A](xs: List[A]): Boolean = {
  xs match {
    case Nil => true
    case _   => false
  }
}

def showListSize[A](xs: List[A]): String {
  xs match {
    case Nil => 0
    case _   => xs.length
  }
}
```

If we want to **unpack** the `head` and `tail` in a pattern match, we can again use the `::` operator:

```scala
xs match {
  case ::(head, tail) => println(s"My head is $head and my tail is $tail")
  case Nil            => println("I am empty!")
}
```

We can also do the same by using `::` as an operator:

```scala
xs match {
  case head :: tail => println(s"My head is $head and my tail is $tail")
  case Nil          => println("I am empty!")
}
```

### Map

A map function applies a function, `f`, to each element of the list, where `f` transforms an element type from type `A` to type `B`:

```scala
map[B](f: A => B): List[B]
```

<!-- For example, type `A` could be `UnpeeledBanana` and `map` transforms it to type `B` of `PeeledBanana`:

```scala
val bananas:  List[PeeledBanana]   = List(b1, b2, b3)
val unpeeled: List[UnpeeledBanana] = bananas.map(banana => banana.peel()) // creating a new banana copy that is peeled
```

![](https://i.imgur.com/P0punkF.png) -->

For example, lets say we have a list of grades:

```scala
val grades: List[Int] = List(90, 70, 88, 89)
```

A teacher is working out which of the students pass or fail. The pass mark is `75` or more. So, he applied the `didPass` function to the grades, which takes an `Int` and returns an `Boolean`:

```scala
def didPass(grade: Int): Boolean = grade >= 75
```

Then he applied a map to the list of grades with the `didPass` function:

```scala
grades.map(didPass) // List(True, False, True, True)
```

You can do this with an anonymous function, or lambda, instead of having to define `didPass`:

```scala
grades.map(grade => grade >= 75)
```

There is also a shorthand syntactic sugar approach, where you do not have to define `grade` or a function arrow, instead using `_` to reperesent individual elements:

```scala
grades.map(_ >= 75)
```

### Filter

A filter function applies a predicate function, `p`, to each element of the list, where `p` is applied to each element of the list `A` and returns a `Boolean` to keep or reject the element:

```scala
filter(p: A => Boolean): List[A]
```

For example, instead of using a map with the `didPass` function, the teacher now decides to filter out all grades instead with this function. (He can do this because `didPass` has a type signature `Int => Boolean`.)

```scala
val grades: List[Int] = List(90, 70, 88, 89)
val passingGrades : List[Int] = grades.filter(didPass)

passingGrades //= List(90, 88, 89)
```

And, of course, there's syntactic sugar for `filter`:

```scala
grades.filter(_ >= 75)
```

### FoldLeft

Fold left is similar to `reduce` in TypeScript or `inject` in Ruby.

Fold left can be used with a `List` of type `A` (`List[A]`). To start, we provide an **i**nitial value, `i`, of type `B`. The operation function, `op`, is applied to every element in the list, and it has two parameters: the **a**ccumulator, `a` of type `B`, which is applied to the each **el**ement, `el`, of type `A`. The result is a single value of type `B`.

The method signature for fold left is:

```scala
foldLeft[B](i: B)(op: (acc: B, el: A) => B): B
```

For example, lets say the teacher wants to find the average grade. To do this, he can use fold left to calculate the total and divide by the number of students:

```scala
val grades: List[Int] = List(90, 70, 88, 89)
val total: Int = grades.foldLeft(0.0)((acc: Double, grade: Int) => acc + grade) // = 337.0
val mean: Double = total / grades.length                                        // = 84.25
```

Let's hand execute this to break down how `foldLeft` worked for each iteration:

Iteration|Accumulator|Element|Result
---|---|---|---
@0|0.0|90|0.0+90=90.0
@1|90.0|70|90+70=160.0
@2|160.0|88|160+88=248.0
@3|248.0|89|248+89=**337.0**

And, of course, there's syntactic sugar for `foldLeft`:

```scala
val total: Float = grades.foldLeft(0.0)(_ + _)
```

### FoldRight

Works very similarly to FoldLeft, however it will iterate through the list in reverse:

```scala
def foldRight[B](i: B)(op: (el: A, acc: B) => B): B
```

Let's apply FoldRight to a list of characters `A`, `B`, and `C` with an initial value `LETTERS`:

```scala
val letters: List[Char] = List('A', 'B', 'C')
val initial: String = "LETTERS"

letters.foldRight("LETTERS")((el: Char, acc: String) => s"$acc $el") // = "LETTERS C B A"
```

Let's see how this worked:

![](https://i.imgur.com/5DX54Xu.png)

### Bringing List Concepts Together: FoldLeft + Pattern Matching + Case Classses

Lets say we have a list of students with their average grade:

```scala
case class Student(name: String, averageGrade: Double)
val students: List[Student] = List(
  Student("Matt Murdock", 30.0),
  Student("Karen Page", 27.0),
  Student("Franklin 'Foggy' Nelson", 31.0),
  Student("Claire Temple", 32.0),
  Student("Wilson Fisk", 42.0),
  Student("Elektra Natchios", 27.0)
)
```

We now want to find out which student has the lowest average grade. The teacher wants this to be re-usable, so it needs to handle a case of empty lists too! Let's implement it:

```scala
def lowestGrade(students: List[Student]): Student = {
  students match {
    case head :: tail => tail.foldLeft(head) { (worstStudentSoFar: Student, el: Student) =>
      if (worstStudentSoFar.averageGrade <= el.averageGrade) worstStudentSoFar else el
    }
    case Nil => Student("Nobody", 0.0)
  }
}
```

Let's break this down:

* We pattern match the students to check for an empty (`Nil`) student list.
* We unpack the `head` and `tail` for non-empty student lists.
* We apply `foldLeft` on the `tail` of the list, providing the first student (i.e., the `head`) as the initialiser
* For each element inside the `tail`, we compare whether the accumulator `worstStudentSoFar` (initialised to the first student, i.e., `head`) has a grade lower than the current element, `el`.
* If they do, we return `worstStudentSoFar`, retaining them as the worst student
* If they do not, we return `el`, which updates the accumulator `worstStudentSoFar` to `el`.

## Null and `Option` Exercises

> [_To null, or not to null. That is the question._](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/)

### Why Nulls Suck

Null does exist in Scala, as it does in many other langauges. However, we can use the [Wartremover linter](https://www.wartremover.org/) to ensure that there are no nulls at compile time.

Conceptually, null can be good. It means something that does not exist or is invalid. For example, if we want to parse `String`s into `TrafficLight` case classes, we might want to return `null` for invalid strings provided:

```scala
trait TrafficLight
case object Red extends TrafficLight
case object Yellow extends TrafficLight
case object Green extends TrafficLight

def parseTrafficLightOrNull(str: String): TrafficLight = {
    str match {
      case "red" => Red
      case "yellow" => Yellow
      case "green" => Green
      case _ => null
    }
}
```

However, we are still using `null` here. This makes us sad because:

* null is not a type;
* null breaks referential transparency (i.e., in the `parseTrafficLightOrNull` example, we always want to return a `TrafficLight` type, but sometimes we return `null`, and that ain't documented!);
* null introduces bugs (e.g., pointer hell and null pointer exceptions).

So what can we do?

### Introducing `Option`al Types

Scala, instead, comes with conceptually similar construct, `Option`, which is not `null`.

Option is a Sum Type ADT which is composed of **either**:

* `Some`<i>thing</i>
* `None`<i>thing</i>

It is defined as:

```scala
sealed trait Option[+A]
case class Some[A](value: A) extends Option[A]
case object None extends Option[Nothing]
```

Optionals say either "there is a value, and it equals x" or "there isn’t a value at all". Visually we can represent it like so:

![](https://i.imgur.com/F3wvPV3.png)

This visualisation illustrates the following:

* 42 (on its own) is a non-optional type;
* the box represents a safe 'wrapper';
* 42 inside the box indicates an optional with `Some`thing (with a value);
* the empty box indicates an optional with `None` (with no value)

In code, this looks like:

```scala
val always42: Int = 42

val maybe42: Option[Int] = Some(42) // or Some(value = 42)
val maybe42: Option[Int] = None
```

We also typically prefix `Optional` variables with `maybe`, to reinforce that there _may_ be a value in the variable, or there _may_ not be.

We can use the method `Option.getOrElse` to unwrap the value, or provide an alternative.

### `parseTrafficLightOrNull` but with `Optional`

Let's try and redo the `parseTrafficLightOrNull` returning an `Option[TrafficLight]` type instead. Instead of returning `null` where there is an invalid string provided, we will now provide `None`. This is conceptually similar as the original `parseTrafficLightOrNull` function but without the yuckiness of having `null`s returned:

```scala
def parseTrafficLight(str: String): Option[TrafficLight] = {
    str match {
      case "red"    => Some(Red)
      case "yellow" => Some(Yellow)
      case "green"  => Some(Green)
      case _        => None()
    }
}

val maybeRedLight: Optiona[TrafficLight] =   parseTrafficLight("red")    // = Some(Red)
val maybeYellowLight: Option[TrafficLight] = parseTrafficLight("yellow") // = Some(Yellow)
val maybePurpleLight: Option[TrafficLight] = parseTrafficLight("purple") // = None
```

The benefit of using the `Option` here also assists with documenting our code; we don't need to call the function <code>parseTrafficLight<u>OrNull</u></code> because the return type `Option[TrafficLight]` documents that our function may, or may not, return a `TrafficLight`.

### Using Pattern Matching and `map` to Safely Unpack Optionals

It is very common to see the following pattern with Optional types which allows you to safely unpack the optional type and work with the value inside if it exists:

```scala
val maybe42: Option[Int] = ??? // We don't know if there's a number in here!

maybe42 match {
  case Some(anInt: Int) => // Do something with `anInt` inside maybe42
  case None             => // Do something else, as there is no value in maybe42
}
```

In cases where we want to _modify the contents_ within an option and also returning an option, we can use pattern matching like so:

```scala
def intToStr(maybeNumber: Option[Int]): Option[String] = {
  maybeNumber match {
    case Some(anInt: Int) => Some(anInt.toString())
    case None => None
  }
}

intToStr(Some(1)) // Some("1")
intToStr(None)    // None
```

However it is annoying to match against the default no case, where `case None => None`. Instead, we can `map` over `Option` types which will do the hard work for us:

```scala
def intToStr(maybeNumber: Option[Int]): Option[String] = maybeNumber.map(_.toString)

intToStr(Some(1)) // Some("1")
intToStr(None)    // None
```

### The Map Type

The `Map` **type** (not to be confused with the **`map` function**!) in Scala is similar to a hashmap or dictionary in many languages:

```scala
val colouredFood: Map[String, String] = Map(
  "brown" -> "potato",
  "green" -> "capsicum",
  "beige" -> "hummus"
)
```

Now we can use the `get` method on the map help us **_safely_** access attributes:

```scala
colouredFood.get("brown")    // Some("potato")
colouredFood.get("rainbow")  // None
```

Or, if we don't want to get `None` where a key in the map doesn't exist, we can use `getOrElse`:

```scala
colouredFood.getOrElse("rainbow", "yasss") // Some("yasss") 🌈
```
