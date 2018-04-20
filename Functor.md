# Intro

Welcome to this series of videos about Functional Programming in Kotlin with Arrow. In this video, we will talk about the
Functor Typeclass. Whenever you find yourself in a situation where you need to map over or transform your data, you will want
to use the Functor.

# Slide 1
Functor is available in the arrow-typeclasses module.
```
// gradle
compile 'io.arrow-kt:arrow-typeclasses:$arrowVersion'

import arrow.typeclasses.Functor

```

# Slide 2
`Functor` is a **Typeclass**, so it defines a behavior.
`Applicative` and `Monad` inherit its properties. You will learn more about those
in further videos.

Functor
Applicative
Monad

# Slide 3
```
interface Functor<F> {
    //...
}
```
* The `F` on the declaration stands for a **type constructor**. That's why `Functor`is considered **polymorphic**.

# Slide 4
The `Functor` abstracts the hability to **map** over the computational context of a type constructor `F`.
In other words, it provides a mapping function for the `F` type:
```
fun F<A>.map(f: (A) -> B): F<B>
```

# Slide 5
```
fun F<A>.map(f: (A) -> B): F<B>
```
* `f` is the function transform the wrapped value.
* `map()` returns the transformed value wrapped into the same context: `F<A> -> F<B>`

# Slide 6
Any type constructor whose contents can be transformed can provide an instance of `Functor`.
* `Option<A>`
* `Try<A>`
* `List<A>`
* `IO<A>`
* ...and many more.

# Slide 7
You can request the functor instance statically from any type constructor supporting it:
```
Option.functor()
Try.functor()
Either.functor<A>()
IO.functor()
//...
```

# Slide 8
See an example on how to map over some Optional data.
```
import arrow.*
import arrow.core.*
import arrow.data.*

Option(1).map { it * 2 }
// Some(2)
```
Here `f` is `{ it *  2}`, which will transform the inner data.

# Slide 9
Sometimes mapping the value just makes sense over one of the implementations of a data type. One exmaple of this is the `Option<A>` type.

As you know, Option is defined like:
```
sealed class Option<A> {
  object None : Option<Nothing>()
  data class Some<T>(val t: T) : Option<T>()
}
```

# Slide 10
`Option` just contains a value when it is a `Some<A>`. So we can just `map` its content when it's a `Some`.
Because of that, we say that `Option` is a *"biased"* to one of its implementations.

```
val some: Option<Int> = Option(1)
some.map { it + 1 }
// Some(2)

val none: Option<Int> = None
none.map { it + 1 }
// None
```
As you can see, the error gets short-circuited when it's a `None`.

# Slide 11
Same thing happens for other data types like `Try<A>`. `Try` wraps a potentially throwing computation, and it is defined like:
```
sealed class Try<out A> {
  data class Success<out A>(val value: A) : Try<A>()
  data class Failure<out A>(val exception: Throwable) : Try<A>()
}
```

# Slide 12
`Try` is biased to its `Success` implementation, so map just works over it. Otherwise it short-circuits the error:
```
val failingOp = Try { failingOperation() }
// Failure(exception=java.lang.RuntimeException)

failingOp.map { it * 2 }
// Failure(exception=java.lang.RuntimeException)
```

# Slide 13
This is how it looks when it's a `Success`.
```
val successfulOp = Try { successfulOp() }
// Success(value=10)

val mappedSuccessfulOp = successfulOp.map { it * 2 }
// Success(value=20)
```

# Slide 14
The `Functor` also provides the `lift` combinator to lift a function to the functor context.
```
fun lift(f: (A) -> B): (F<A>) -> F<B>
```
So you can apply it over values wrapped in the same data type context.
```
val lifted = Option.functor().lift({ n: Int -> n + 1 })
val next = lifted(Option(1))
// Some(2)
```

# Final

In this video, we learned about `Functor` and how to use its main combinators to map over your data or lift a function to a type constructor's context. We also learned about how biased types short-circuit errors when we use combinators like `map` over them.

In following videos we'll learn about other related and important Typeclasses like `Applicative` and `Monad`.
