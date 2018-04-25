autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@jorgecastillopr](https://twitter.com/jorgecastillopr) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/typeclasses/functor/](http://arrow-kt.io/docs/typeclasses/functor/)
slidenumbers: true

# Functor
__`Functor`__ is available in the __arrow-typeclasses__ module.
```groovy
// gradle
compile 'io.arrow-kt:arrow-typeclasses:$arrowVersion'

import arrow.typeclasses.Functor
```

---

# Functor
`Functor` is a __Typeclass__, so it defines a given behavior. `Applicative` and `Monad` inherit its combninators. You will learn more about those in further videos.

Functor
Applicative
Monad

---

# Functor :: Polymorphism
```kotlin
interface Functor<F> {
    //...
}
```
- Functor is parametric over a type constructor `F`. This allows us to build abstract functions over the behaviors that Functor defines and forgetting about the concrete types that F may refer to.

- We call this __ad-hoc polymorphism__, the ability to write polymorphic programs that can be defined in generic terms.

---

# Functor :: map
`Functor` abstracts the hability to __map__ over the computational context of a type constructor `F`.
In other words, it provides a mapping function for the `F` type:
```kotlin
fun F<A>.map(f: (A) -> B): F<B>
```

---

# Functor :: map
```kotlin
fun F<A>.map(f: (A) -> B): F<B>
```
- `f` is the function transforming the wrapped value.
- `map()` returns the transformed value wrapped into the same context: `F<A> -> F<B>`

---

# Functor :: Data types
Any type constructor whose contents can be transformed can provide an instance of `Functor`.
- `Option<A>`
- `Try<A>`
- `List<A>`
- `IO<A>`
- ...and many more.

---

# Functor :: Polymorphism
Using Typeclasses you can define __completely polymorphic programs that can work over any data types__ providing an instance of `Functor`. This is actually the biggest power Typeclasses have.
```kotlin
// Abstract program, we just know we need a Functor
fun <F> Functor<F>.addOne(fa: Kind<F, Int>): Kind<F, Int> =
  fa.map { it + 1 }

// Making it concrete for Option and Try
Option.functor().addOne(Option(1)).fix() // Option(2)
Try.functor().addOne(Try { 1 }).fix() // Success(2)
```
Look at how we declare the abstract program first and we make it concrete afterwards for two different data types: `Option` and `Try`.

---

# Functor :: Option<Functor>
Here you have an example on how to map over some Optional data.
```kotlin
import arrow.*
import arrow.core.*
import arrow.data.*

Option(1).map { it * 2 }
// Some(2)
```
Here `f` is `{ it *  2 }`, which will transform the inner data.

---

# Functor :: Data type bias
Sometimes mapping the value just makes sense over one of the implementations of a data type. One exmaple of this is the `Option` type, with two different implementations:
```kotlin
sealed class Option<A> {
  object None : Option<Nothing>()
  data class Some<T>(val t: T) : Option<T>()
}
```

---

# Functor :: Data type bias
`Option` just contains a value when its type is `Some<A>`. So we can `map` over its content just for that case.
Because of that, we say __`Option` is "biased" toward the `Some` case__.

```kotlin
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

---

# Functor :: Data type bias
Same thing happens for other data types like `Try<A>`. `Try` models safe access to API's that may throw exceptions:
```kotlin
sealed class Try<out A> {
  data class Success<out A>(val value: A) : Try<A>()
  data class Failure<out A>(val exception: Throwable) : Try<A>()
}
```

---

# Functor :: Data type bias
__`Try` is biased toward its `Success` case__, so map just works over that one. Otherwise it short-circuits the error:
```kotlin
// defining abstract program
fun <F> Functor<F>.increment(fa: Kind<F, Int>): Kind<F, Int> =
  fa.map { it + 1 }

// Let's make it concrete
Try.functor().increment(Try { 1 }).fix()
// Success(value=2)

Try.functor().increment(Try { throw RuntimeException() }).fix()
// Failure(exception=java.lang.RuntimeException)
```

---

# Functor :: lift
The `Functor` also provides the `lift` combinator to __lift a function to the functor context__.
```kotlin
fun lift(f: (A) -> B): (F<A>) -> F<B>
```
Thanks to this you can apply it over values wrapped in the same data type context.
```kotlin
val lifted = Option.functor().lift({ n: Int -> n + 1 })
val next = lifted(Option(1))
// Some(2)
```

---

# Functor :: Laws
Any Typeclass __must satisfy some mathematical laws__ in order to be considered a Typeclass. Arrow has tests in place for those laws so typeclasses can keep their integrity over time.

---

# Functor :: Laws
Functor Satisfies the following laws:
- __Identity:__ Mapping the identity function over every item in a container has no effect.
    ```kotlin
    fa.map(::identity) = fa
    ```
- __Composition:__ Mapping a composition of two functions over every item in a container is the same as first mapping one function, and then mapping the other.
    ```kotlin
    fa.map(f).map(g) = fa.map(f andThen g)
    ```

---

# Functor :: Laws
You can __test your own functor instances__ by using the Arrow encoded laws:
```groovy
// build.gradle
testCompile 'io.arrow-kt:arrow-test:0.7.1'
```
Depend from the `arrow-test` gradle module on your tests.
```kotlin
import arrow.test.laws.FunctorLaws // Functor laws are defined here

@Test
fun testFunctorLaws() {
  FunctorLaws.laws(Option.functor(), ::Some, Eq.any())
}
```
- We are passing the default functor instance provided by `Option`, but __you could pass your own one__.
- `Eq.any()` provides an implementation of the `Eq` typeclass for equality. It uses `==` as its equality operator. It's enough for `Option` since its both implementations are defined as `data classes`, but you can (and should) pass your own `Eq` implementation for more complex types.

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @jorgecastillopr : [https://twitter.com/jorgecastillopr](https://twitter.com/jorgecastillopr)
