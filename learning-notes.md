# Intro to Scala

## Immutability is King

In Scala, we deal with immutable values:

```scala
val a = 3
a = 4 // <- impossible
```

We turn things into **immutable facts**. Immutable data does not lie.

## Types & Type Signatures

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

### Identity Functions are King

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

## Intro Exercises 1

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

#### Tuples and Tuple Unpacking

We can also use Tuples too:

```scala
def pair(name: String, age: Int): (String, Int) = (name, age)
```

Here we have a return type `(String, Int)`.

We can extract or unpack elements from the tuples too:

```scala
def first(pair: (String, Int)): String = {
  val (name, _) = pair
  name
}
```

Similarly, we can do the same for the second element using index notation

```scala
def second(pair: (String, Int)): Int = pair._2
```

You can have up to 22 elements inside tuples, although you'd probably want to use another data type!
