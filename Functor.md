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

^ The Functor is provided on the arrow-typeclasses artifact.
^ Just add it to your build.gradle file and you'll be good to go.

---

# Functor

`Functor` is a __Typeclass__, so it defines a given behavior.

`Applicative` and `Monad` inherit its combinators. You will learn more about those in other videos in the series.

Functor
Applicative
Monad

^ Functor is defined as a Typeclass. On pure Functional Programming, Typeclasses define behaviors.
^ We'll know about which behavior is encoded in the Functor in further slides.
^ Other very well known typeclasses are Applicative and Monad, and this is how they're related to each other.
(necesitaremos diagrama de cajitas con flechas de dependencia entre ellas)
^ We'll talk about those in other Arrow videos.

---

# Functor :: Polymorphism

```kotlin
interface Functor<F> {
    //...
}
```

- Functor is __parametric over a type constructor `F`__. This allows us to build abstract functions over the behaviors that Functor defines and forget about the concrete types that F may refer to.

- We call this __ad-hoc polymorphism__, the ability to write polymorphic programs that can be defined in generic terms.

^ Functor is parametric over a generic type F. F here, stands for a data type or what we also call a "Type Constructor".
^ Thanks to this, we can declare generic functions encoded on top of the behaviors provided by Functor and forget all the way about the concrete data type used for F.
^ This is called "Ad-Hoc polymorphism" and that's how typeclasses allow us to encode completely generic and polymorphic programs that can work over many different data types.

---

# Functor :: map

`Functor` abstracts the ability to __map__ over the computational context of a type constructor `F`.

In other words, it provides a mapping function for the `F` type:

```kotlin
fun Kind<F, A>.map(f: (A) -> B): F<B>
```

^ The most important behavior provided by Functor is the map function.
^ With map, we can map over the computational context of the type constructor F.

---

# Functor :: map

```kotlin
fun F<A>.map(f: (A) -> B): F<B>
```

- `f` is the function transforming the wrapped value.
- `map()` returns the transformed value wrapped into the same context: `F<A> -> F<B>`

^ This just means, that you'll pass in a mapping function that can operate over the wrapped value inside the type constructor, and then return the already mapped value wrapped again inside of F.
^ So you're basically providing a mapping function for the F type.

---

# Functor :: Data types

Any type constructor whose contents can be transformed can provide an instance of `Functor`.

- `Option<A>`
- `Try<A>`
- `List<A>`
- `IO<A>`
- ...and many more.

^ We previously said that the Functor can work over any type constructor F.
^ Truth is that F must be a data type able to provide an instance of Functor for it. Some examples would be: Option, Try, List, IO, and many more.

---

# Functor :: Polymorphism

Using Typeclasses you can define __completely polymorphic programs__ that can work over any data types providing an instance of `Functor`.

This is actually the biggest power Typeclasses have.

```kotlin
// Abstract program, we just know we need a Functor
fun <F> Functor<F>.addOne(fa: Kind<F, Int>): Kind<F, Int> =
  fa.map { it + 1 }
```

^ The hability to write polymorphic programs working over any data type is the most important bit to learn about typeclasses.
^ Here you have an example: We can write a program in a completely abstract way, working over ANY instance of Functor of F, so the functor behavior will be used to map over the inner value of it.
^ Here we're just adding one to that value, and then returning the already mapped value wrapped again into F.

---

# Functor :: Polymorphism

```kotlin
// Abstract program, we just know we need a Functor
fun <F> Functor<F>.addOne(fa: Kind<F, Int>): Kind<F, Int> =
  fa.map { it + 1 }

// Making it concrete for Option and Try
Option.functor().addOne(Option(1)).fix() // Option(2)
Try.functor().addOne(Try { 1 }).fix() // Success(2)
```

^ So, after having that abstract declaration, we can make it concrete afterwards.
^ On this example we're fixing the abstract behavior for two different data types: `Option` and `Try`.
^ The moment to make your program concrete and fix it to a given type is basically when you're providing the implementation details. So you'll do it when you want to finally run your program.
^ Any point in time before that, your program will be completely abstract, and you'll be highly interested on keeping that fact as long as possible.

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

^ After knowing about polymorphic programs, we can also read some simple examples about how map works over different data types.
^ Here you have a simple mapping over an optional value. We can do it because Option is able to provide an instance of functor for it.
^ Note that types able to provide a functor instance, already provide the functor syntax activated over them. That's why we can call map over the optional value without using an explicit instance of Functor. Arrow is providing it for you under the hood.

---

# Functor :: Data type bias

Sometimes mapping the value over one of the implementations of a data type just makes sense.

One exmaple of this is the `Option` type, with two different implementations:

```kotlin
sealed class Option<A> {
  object None : Option<Nothing>()
  data class Some<T>(val t: T) : Option<T>()
}
```

^ There are many cases where mapping over some implementations of a given data type do not make much sense.
^ One good example would be Option, which is defined as a sealed class with a couple of implementations: None, and Some.

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

^ Given that option just contains a value when it's a Some, that's the only case we would need to be able to map over.
^ That's why we say that Option is "biased" toward the Some case.
^ As you can see on this snippet, we can call the increment method for both implementations, Some and None.
^ The difference is that it will just work when it's a Some. Whenever we try to map over a None, will get a None as a result.
^ So, if we look at Option.None as a way to express an error representing the absence of a value, any mapping computations built over it will be short-circuiting the error and keep returning None.

---

# Functor :: Data type bias

The same thing happens for other data types like `Try<A>`.

`Try` models safe access to API's that may throw exceptions:

```kotlin
sealed class Try<out A> {
  data class Success<out A>(val value: A) : Try<A>()
  data class Failure<out A>(val exception: Throwable) : Try<A>()
}
```

^ Another good example of bias can be Try.
^ Try wraps a computation that might throw exceptions, like an access to an external API.
^ Try is also declared as a sealed class with two different implementations: Success and Failure, reflecting the two possible scenarios returned by a potentially failing computation.

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

^ Try is biased towards its Success implementation.
^ Right after making our program concrete and fixing it to Try, we call increment over it. This time we are using an explicit functor instance for Try. Since the program is valid, it will succeed and we'll get the value correctly mapped and wrapped into the same context.
^ In the other hand, when the operation turns out to be a Failure, any computations over it will keep returning Failure.

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

^ Another behavior provided by Functor is the lift combinator.
^ Lift is able to lift a function to the functor context.
^ So you pass a function from A to B, and lift will return a funcion from F of A to F of B. You can store that lifted function into a variable, and use it later on.
^ Once you have your function lifted to the given context, you can apply it over any values wrapped over the same context, like we're doing here.
^ We've chosen Option here as the F context.

---

# Functor :: Laws

All Typeclasses __must satisfy some mathematical laws__ in order to be considered a Typeclass.

Arrow has tests in place for those laws so typeclasses can keep their integrity over time.

^ When we think about typeclasses, we assume they need to satisfy some mathematical laws in order to be considered a typeclass.
^ Those laws are coded and enforced by tests inside Arrow, so we can ensure typeclasses integrity over time.
^ Some laws could be identity, associativity, composition, and so on.

---

# Functor :: Laws

Functor Satisfies the following laws:

- __Identity__: Mapping the identity function over every item in a container has no effect.

    ```kotlin
    fa.map(::identity) = fa
    ```

- __Composition__: Mapping a composition of two functions over every item in a container is the same as first mapping one function, and then mapping the other.

    ```kotlin
    fa.map(f).map(g) = fa.map(f andThen g)
    ```

^ Functor satisfies two laws: Identity and Composition.
^ Identity for Functor means that mapping any computational context using the identity function will have no effect. In other words, it'll return the same value wrapped into the same context. The Identity function is just a function that returns the input value as is.
^ About Composition law, it applies to Functor in a way that mapping a type constructor twice with two different functions "f" and "g" is equivalent to mapping it once using the composition of both functions.

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

- We are passing the default functor instance provided by `Option`, but __you could pass your own__.
- `Eq.any()` provides an implementation of the `Eq` typeclass for equality. It uses `==` as its equality operator. It's enough for `Option` since both its implementations are defined as `data classes`, but you can (and should) pass your own `Eq` implementation for more complex types.

^ If you are creating your own Functor instances for your custom data types, you'll also need a way to ensure those instances are satisfying the Functor laws. You can do it by fetching the arrow-test artifact.
^ Once you do it you can call the laws function over the FunctorLaws object and pass in your functor instance, and the Eq instance that you will need to define to provide a way to compare two instances of it for equality.

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @jorgecastillopr : [https://twitter.com/jorgecastillopr](https://twitter.com/jorgecastillopr)
