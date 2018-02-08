autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@calvellido](https://twitter.com/calvellido) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/validated/](http://arrow-kt.io/docs/datatypes/validated/)
slidenumbers: true

# Validated

__`Validated`__ is a data type used in __Λrrow__ to model a return type which can be a success value or a collection of errors.

---

# Validated :: ADT

__`Validated`__ is modeled as an __Algebraic Data Type__.
In Kotlin ADTs are encoded with sealed class hierarchies.

```kotlin
sealed class Validated<out E, out A>
data class Invalid<out E>(val e: E) : Validated<E, Nothing>()
data class Valid<out A>(val a: A) : Validated<Nothing, A>()
```

---

# Validated :: ADT

This encoding states that a Validated can only have 2 possible states.

```kotlin
sealed class Validated<out E, out A>
data class Invalid<out E>(val e: E) : Validated<E, Nothing>()
data class Valid<out A>(val a: A) : Validated<Nothing, A>()
```

---

# Validated :: Invalid

If the value is considered invalid, it's represented as an instance of __Invalid__

```kotlin, [.highlight: 2]
sealed class Validated<out E, out A>
data class Invalid<out E>(val e: E) : Validated<E, Nothing>()
data class Valid<out A>(val a: A) : Validated<Nothing, A>()
```

---

# Validated :: Valid

If the value is what's expected AKA a happy path, it's represented in the __Valid__ case

```kotlin, [.highlight: 3]
sealed class Validated<out E, out A>
data class Invalid<out E>(val e: E) : Validated<E, Nothing>()
data class Valid<out A>(val a: A) : Validated<Nothing, A>()
```

---

# Validated :: Invalid(b)

We can easily construct values with __`Invalid(value)`__

```kotlin
object KnownError

val result: Validated<KnownError, Int> = Invalid(KnownError)
// Invalid(KnownError)
```

---

# Validated :: Valid(b)

We can easily construct values with __`Valid(value)`__

```kotlin
object KnownError

val result: Validated<KnownError, Int> = Valid(1)
// Valid(1)
```

---

# Validated :: Extension syntax

Λrrow provides syntax extensions that allows you to put values in both __Invalid__ and __Valid__

```kotlin
import arrow.syntax.validated.*

object KnownError

val result: Validated<KnownError, Int> = KnownError.invalid()
// Invalid(KnownError)

val result: Validated<KnownError, Int> = 1.valid()
// Valid(1)

```

---

# Validated :: Transformations

We can transform __Validated__ values through several built in functions
- when
- fold
- getOrElse
- map
- Applicative # map

---

# Validated :: when

We can extract __Validated__ inner values by using Kotlin's __when__ expressions

```kotlin
object KnownError

val result: Validated<KnownError, Int> = Valid(1)

when(result) {
    is Invalid -> 0
    is Valid -> result.b
}
// 1
```

---

# Validated :: fold

__fold__ contemplates both __Invalid__ and __Valid__ cases.

```kotlin
object KnownError

val result: Validated<KnownError, Int> = Valid(1)

result.fold(
  { 0 },
  { it }
)
// 1
```

---

# Validated :: fold

The first argument is a function that addresses the __Invalid__ case

```kotlin, [.highlight: 6]
object KnownError

val result: Validated<KnownError, Int> = Valid(1)

result.fold(
  { 0 },
  { it + 1 }
)
// 2
```

---

# Validated :: fold

The second argument is a function that addresses the __Valid__ case

```kotlin, [.highlight: 7]
object KnownError

val result: Validated<KnownError, Int> = Valid(1)

result.fold(
  { 0 },
  { it + 1 }
)
// 2
```

---

# Validated :: getOrElse

__getOrElse__ allows us to provide the default value if __Invalid__

```kotlin
object KnownError

val result: Validated<KnownError, Int> = Invalid(KnownError)

result.getOrElse { 0 }
// 0
```

---

# Validated :: map

__map__ allows us to transform __B__ into __C__ in __Validated< A, B >__

```kotlin
object KnownError

val result: Validated<KnownError, Int> = Valid(1)

result.map { it + 1 }
// Valid(2)
```

---

# Validated :: map

When we __map__ over __Validated__ values that are __Invalid__ the transformation is never applied and the __Invalid__ is returned

```kotlin
object KnownError

val result: Validated<KnownError, Int> = Invalid(KnownError)

result.map { it + 1 }
// Invalid(KnownError)
```

---


# Validated :: Applicative Builder

Λrrow contains methods that allow you to preserve type information when computing over different __Validated__ typed values.

```kotlin
object KnownError
data class Person(id: UUID, name: String, year: Int)

// Note each Validated is of a different type
val vId: Validated<KnownError, UUID> = Valid(UUID.randomUUID())
val vName: Validated<KnownError, String> = Valid("William Alvin Howard")
val vAge: Validated<KnownError, Int> = Valid(1926)

Validated.applicative<KnownError>().map(vId, vName, vAge, { (id, name, age) ->
  Person(id, name, age)
}).ev()
// Valid(Person(<uuid>, "William Alvin Howard", 1926))
```

---

# Validated :: Applicative Builder

If a value turns out to be __Invalid__ the computation doesn't short-circuit, but instead combines the possible

```kotlin, [.highlight: 6, 12]
object KnownError
data class Person(id: UUID, name: String, year: Int)

// Note each Validated is of a different type
val vId: Validated<KnownError, UUID> = Invalid(KnownError)
val vName: Validated<KnownError, String> = Invalid(KnownError)
val vAge: Validated<KnownError, Int> = Right(1926)

Validated.applicative<KnownError>().map(vId, vName, vAge, { (id, name, age) ->
  Person(id, name, age)
}).ev()
// Invalid(KnownError |+| KnownError)
```

---

# Validated :: Conclusion

- Validated is __used to assure computation that throw exceptions__
- We can easily construct values of `Validated` with `Invalid(1)`, `Valid(1)`, `1.invalid()` or `1.valid()`.
- __fold__, __map__ and others are used to compute over the internal contents of a Validated computation.
- __Validated.applicative<E>().map { ... }__ can be used to compute over multiple Optional values preserving type information and __abstracting over arity__ with `map`
---

# Validated :: Conclusion

- __All techniques demonstrated are also available to other data types__ such as `Either`, `Option`, `IO` and you can build adapters for any data types.
- We will learn more about other data types like `Either`, `Option`, `IO` and type classes that power these abstraction such as `Functor`, `Applicative` and `Monad` in other videos.
- __Λrrow encourages a unified programming model__ in which you can solve problems cohesively in all contexts following Typed Functional Programming principles applied to the Kotlin Programming Language.

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @calvellido : [https://twitter.com/raulraja](https://twitter.com/calvellido)
