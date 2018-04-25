# Intro

Welcome to this series of videos about Functional Programming in Kotlin with Arrow. In this video, we will talk about the Functor Typeclass. Whenever you find yourself in a situation where you need to map over or transform your data, you will want to use the Functor.

# Slide 1
Functor is available in the arrow-typeclasses module.
```
// gradle
compile 'io.arrow-kt:arrow-typeclasses:$arrowVersion'

import arrow.typeclasses.Functor

```

# Slide 2
`Functor` is a **Typeclass**, so it defines a given behavior. `Applicative` and `Monad` inherit its combninators. You will learn more about those in further videos.

Functor
Applicative
Monad

# Slide 3
```
interface Functor<F> {
    //...
}
```
* Functor is parametric over a type constructor `F`. This allows us to build abstract functions over the behaviors that Functor defines and forgetting about the concrete types that F may refer to.

* We call this **ad-hoc polymorphism**, the ability to write polymorphic programs that can be defined in generic terms.

# Slide 4
`Functor` abstracts the hability to **map** over the computational context of a type constructor `F`.
In other words, it provides a mapping function for the `F` type:
```
fun F<A>.map(f: (A) -> B): F<B>
```

# Slide 5
```
fun F<A>.map(f: (A) -> B): F<B>
```
* `f` is the function transforming the wrapped value.
* `map()` returns the transformed value wrapped into the same context: `F<A> -> F<B>`

# Slide 6
Any type constructor whose contents can be transformed can provide an instance of `Functor`.
* `Option<A>`
* `Try<A>`
* `List<A>`
* `IO<A>`
* ...and many more.

# Slide 7
Using Typeclasses you can define completely polymorphic programs that can work over any data types providing an instance of `Functor`. This is actually the biggest power Typeclasses have.
```
// Abstract program, we just know we need a Functor
fun <F> Functor<F>.addOne(fa: Kind<F, Int>): Kind<F, Int> =
  fa.map { it + 1 }

// Making it concrete for Option and Try
Option.functor().addOne(Option(1)).fix() // Option(2)
Try.functor().addOne(Try { 1 }).fix() // Success(2)
```
Look at how we declare the abstract program first and we make it concrete afterwards for two different data types: `Option` and `Try`.

# Slide 8
Here you have an example on how to map over some Optional data.
```
import arrow.*
import arrow.core.*
import arrow.data.*

Option(1).map { it * 2 }
// Some(2)
```
Here `f` is `{ it *  2 }`, which will transform the inner data.

# Slide 9
Sometimes mapping the value just makes sense over one of the implementations of a data type. One exmaple of this is the `Option` type, with two different implementations:
```
sealed class Option<A> {
  object None : Option<Nothing>()
  data class Some<T>(val t: T) : Option<T>()
}
```

# Slide 10
`Option` just contains a value when its type is `Some<A>`. So we can `map` over its content just for that case.
Because of that, we say `Option` is *"biased"* toward the `Some` case.

```
// defining abstract program
fun <F> Functor<F>.increment(fa: Kind<F, Int>): Kind<F, Int> =
  fa.map { it + 1 }

// Let's make it concrete
Option.functor().increment(Option(1)).fix()
// Some(2)
Option.functor().increment(None).fix()
// None
```
As you can see, the computation gets short-circuited when it's a `None`.

# Slide 11
Same thing happens for other data types like `Try<A>`. `Try` models safe access to API's that may throw exceptions:
```
sealed class Try<out A> {
  data class Success<out A>(val value: A) : Try<A>()
  data class Failure<out A>(val exception: Throwable) : Try<A>()
}
```

# Slide 12
`Try` is biased toward its `Success` case, so map just works over that one. Otherwise it short-circuits the error:
```
// defining abstract program
fun <F> Functor<F>.increment(fa: Kind<F, Int>): Kind<F, Int> =
  fa.map { it + 1 }

// Let's make it concrete
Try.functor().increment(Try { 1 }).fix()
// Success(value=2)

Try.functor().increment(Try { throw RuntimeException() }).fix()
// Failure(exception=java.lang.RuntimeException)
```

# Slide 13
The `Functor` also provides the `lift` combinator to lift a function to the functor context.
```
fun lift(f: (A) -> B): (F<A>) -> F<B>
```
Thanks to this you can apply it over values wrapped in the same data type context.
```
val lifted = Option.functor().lift({ n: Int -> n + 1 })
val next = lifted(Option(1))
// Some(2)
```

# Slide 14
Any Typeclass **must satisfy some mathematical laws** in order to be considered a Typeclass. Arrow has tests in place for those laws so typeclasses can keep their integrity over time.

# Slide 15
Functor Satisfies the following laws:
* **Identity:** Mapping the identity function over every item in a container has no effect.
    ```
    fa.map(::identity) = fa
    ```
* **Composition:** Mapping a composition of two functions over every item in a container is the same as first mapping one function, and then mapping the other.
    ```
    fa.map(f).map(g) = fa.map(f andThen g)
    ```

# Slide 16
You can **test** your own functor instances by using the Arrow encoded laws:
```
// build.gradle
testCompile 'io.arrow-kt:arrow-test:0.7.1'
```
Depend from the `arrow-test` gradle module on your tests.
```
import arrow.test.laws.FunctorLaws // Functor laws are defined here

@Test
fun testFunctorLaws() {
  FunctorLaws.laws(Option.functor(), ::Some, Eq.any())
}
```
* We are passing the default functor instance provided by `Option`, but **you could pass your own one**.
* `Eq.any()` provides an implementation of the `Eq` typeclass for equality. It uses `==` as its equality operator. It's enough for `Option` since its both implementations are defined as `data classes`, but you can (and should) pass your own `Eq` implementation for more complex types.

# Final

In this video, we learned about `Functor` and how to use its main combinators to map over your data or lift a function to a type constructor's context. We also learned about how biased types short-circuit errors when we use combinators like `map` over them.

In following videos we'll learn about other related and important Typeclasses like `Applicative` and `Monad`.
