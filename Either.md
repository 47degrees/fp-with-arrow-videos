autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@raulraja](https://twitter.com/raulraja) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/either/](http://arrow-kt.io/docs/datatypes/either/)
slidenumbers: true

# Either 

__`Either`__ is a data type used in __Λrrow__ to model a return type that may return one of two possible values

---

# Either :: ADT

__`Either`__ is modeled as an __Algebraic Data Type__.
In Kotlin ADTs are encoded with sealed class hierarchies.

```kotlin
sealed class Either<out A, out B>
data class Left<out A, out B>(val a: A) : Either<A, B>()
data class Right<out A, out B>(val b: B) : Either<A, B>()
```

---

# Either :: ADT

This encoding states that an Either can only have 2 possible states.

```kotlin
sealed class Either<out A, out B>
data class Left<out A, out B>(val a: A) : Either<A, B>()
data class Right<out A, out B>(val b: B) : Either<A, B>()
```

---


# Either :: Left

If the value is exceptional it's by convention placed in the __Left__ case

```kotlin, [.highlight: 2]
sealed class Either<out A, out B>
data class Left<out A, out B>(val a: A) : Either<A, B>()
data class Right<out A, out B>(val b: B) : Either<A, B>()
````

---

# Either :: Right

If the value is what's expected AKA a happy path, it's represented in the __Right__ case

```kotlin, [.highlight: 3]
sealed class Either<out A, out B>
data class Left<out A, out B>(val a: A) : Either<A, B>()
data class Right<out A, out B>(val b: B) : Either<A, B>()
```

---

# Either :: Right(b)

We can easily construct values with __`Right(value)`__

```kotlin
object KnownError

val result: Either<KnownError, Int> = Right(1)
// Right(1)
```

---

# Either :: Left(b)

We can easily construct values with __`Left(value)`__

```kotlin
object KnownError

val result: Either<KnownError, Int> = Left(KnownError)
// Left(KnownError)
```

---

# Either :: Extension syntax

Λrrow provides syntax extensions that allows you to put values in both __Left__ and __Right__

```kotlin
import arrow.syntax.either.*

object KnownError

val result: Either<KnownError, Int> = 1.right()
// Right(1)

val result: Either<KnownError, Int> = KnownError.left()
// Left(KnownError)
```

---

# Either :: Transformations

We can transform __Either__ values through several built in functions
- when
- fold
- getOrElse
- map
- flatMap
- Monad # binding
- Applicative # map

---

# Either :: when

We can extract __Either__ inner values by using kotlin's __when__ expressions

```kotlin
object KnownError

val result: Either<KnownError, Int> = Right(1)

when(result) {
    is Left -> 0
    is Right -> result.b
}
// 1
```

---

# Either :: fold

__fold__ contemplates both __Left__ and __Right__ cases.

```kotlin
object KnownError

val result: Either<KnownError, Int> = Right(1)

result.fold(
  { 0 }, 
  { it }
)
// 1
```

---

# Either :: fold

The first argument is a function that addresses the __Left__ case

```kotlin, [.highlight: 6]
object KnownError

val result: Either<KnownError, Int> = Right(1)

result.fold(
  { 0 }, 
  { it + 1 }
)
// 2
```

---

# Either :: fold

The second argument is a function that addresses the __Right__ case

```kotlin, [.highlight: 7]
object KnownError

val result: Either<KnownError, Int> = Right(1)

result.fold(
  { 0 }, 
  { it + 1 }
)
// 2
```

---

# Either :: getOrElse

__getOrElse__ allows us to provide the default value if __Left__

```kotlin
object KnownError

val result: Either<KnownError, Int> = Left(KnownError)

result.getOrElse { 0 }
// 0
```

---

# Either :: map

__map__ allows us to transform __B__ into __C__ in __Either< A, B >__

```kotlin
object KnownError

val result: Either<KnownError, Int> = Right(1)

result.map { it + 1 }
// Right(2)
```

---

# Either :: map

When we __map__ over __Either__ values that are __Left__ the transformation is never applied and the __Left__ is returned

```kotlin
object KnownError

val result: Either<KnownError, Int> = Left(KnownError)

result.map { it + 1 }
// Left(KnownError)
```

---

# Either :: flatMap

__flatMap__ allows us to compute over the contents of multiple __Either< A, B >__ values 

```kotlin
object KnownError
val result1: Either<KnownError, Int> = Right(1)
val result2: Either<KnownError, Int> = Right(2)

result1.flatMap { one -> 
  result2.map { two -> 
    one + two  
  }
}
// Right(3)
```

---

# Either :: Monad binding

Λrrow allows imperative style comprehensions to make computing over Either values easy.

```kotlin
object KnownError
val result1: Either<KnownError, Int> = Right(1)
val result2: Either<KnownError, Int> = Right(2)

Either.monad<KnownError>().binding {
    val one = result1.bind()
    val two = result2.bind()
    yields(one + two)
}.ev()
// Right(3)
```

---

# Either :: Monad binding

Each call to __bind()__ is a coroutine suspended function which will bind to it's value only if the __Either__ is a __Right__

```kotlin, [.highlight: 6-7]
object KnownError
val result1: Either<KnownError, Int> = Right(1)
val result2: Either<KnownError, Int> = Right(2)

Either.monad<KnownError>().binding {
    val one = result1.bind()
    val two = result2.bind()
    yields(one + two)
}.ev()
// Right(3)
```

---

# Either :: Monad binding

If any of the values is __Left__, __bind()__ will shortcircuit yielding __Left__

```kotlin, [.highlight: 3, 7, 10]
object KnownError
val result1: Either<KnownError, Int> = Right(1)
val result2: Either<KnownError, Int> = Left(KnownError)

Either.monad<KnownError>().binding {
    val one = result1.bind()
    val two = result2.bind()
    yields(one + two)
}.ev()
// Left(KnownError)
```

---

# Either :: Applicative Builder

Λrrow contains methods that allow you to preserve type information when computing over different __Either__ typed values.

```kotlin
object KnownError
data class Person(id: UUID, name: String, year: Int)

// Note each Either is of a different type
val eId: Either<KnownError, UUID> = Right(UUID.randomUUID())
val eName: Either<KnownError, String> = Right("William Alvin Howard")
val eAge: Either<KnownError, String> = Right(1926)

Either.applicative<KnownError>().map(eId, eName, eAge, { (id, name, age) ->
  Person(id, name, age)
}).ev()
// Right(Person(<uuid>, "William Alvin Howard", 1926))
```

---

# Either :: Applicative Builder

If a value turns out to be __Left__ the computation short-circuits

```kotlin, [.highlight: 6, 12]
object KnownError
data class Person(id: UUID, name: String, year: Int)

// Note each Either is of a different type
val eId: Either<KnownError, UUID> = Right(UUID.randomUUID())
val eName: Either<KnownError, String> = Left(KnownError)
val eAge: Either<KnownError, String> = Right(1926)

Either.applicative<KnownError>().map(eId, eName, eAge, { (id, name, age) ->
  Person(id, name, age)
}).ev()
// Left(KnownError)
```

---

# Either :: Conclusion

- Either is __used to model unions__ of types over a potential value
- We can easily construct values of `Either` with `Right(1)`, `Left(1)`, `1.right()` or `1.left()`
- __fold__, __map__, __flatMap__ and others are used to compute over the internal contents of an Either value.
- __Either.monad<L>().binding { ... } Comprehensions__ can be __used to imperatively compute__ over multiple Either values in sequence.
- __Either.applicative<L>().map { ... }__ can be used to compute over multiple Optional values preserving type information and __abstracting over arity__ with `map`

---

# Either :: Conclusion

- __All techniques demonstrated are also available to other data types__ such as `Try`, `Option`, `IO` and you can build adapters for any data types.
- We will learn more about other data types like `Try`, `Option`, `IO` and type classes that power these abstraction such as `Functor`, `Applicative` and `Monad` in other videos.
- __Λrrow encourages a unified programming model__ in which you can solve problems cohesively in all contexts following Typed Functional Programming principles applied to the Kotlin Programming Language.

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow 
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @raulraja : [https://twitter.com/raulraja](https://twitter.com/raulraja)
