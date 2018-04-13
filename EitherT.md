autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@vejeta](https://twitter.com/vejeta) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/eithert/](http://arrow-kt.io/docs/datatypes/EitherT/)
slidenumbers: true

# EitherT

__`EitherT`__ is a data type used in __Λrrow__ when we want to compute an Either that is nested inside another monad.

---

# EitherT

We can use `EitherT` when we find ourselves with an `Either` value nested inside any other
effect.

```kotlin
import arrow.core.*
import arrow.data.*
import arrow.effects.*


sealed class NumberError
data class StringToNumberError(val stringNumber: String): NumberError()

fun stringToInt(a: String): Either<StringToNumberError, Int> =
    Try{ a.toInt()}.fold({Either.left(StringToNumberError(a))}, { v -> Either.right(v) })

val eitherInDeferred: DeferredK<Either<StringToNumberError, Int>> = async { Either.right(1) }.k()
val eitherInList: ListK<Either<StringToNumberError, Int>> = listOf(stringToInt("one"), stringToInt("1")).k()
val eitherInIO: IO<Either<StringToNumberError, Int>> = IO( {Either.right(1) })
```

---

# EitherT

We can construct `EitherT` instances by wrapping existing `F<Either<E, A>>` values

```kotlin
val eitherTDeferred: EitherT<ForDeferredK, StringToNumberError, Int> = EitherT(async { Either.right(1) }.k())
val eitherTList: EitherT<ForListK, StringToNumberError, Int> = EitherT(listOf(stringToInt("one"), stringToInt("1")).k())
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(1)})
```

---

# EitherT :: fold

__fold__ contemplates both __Left__ and __Right__ cases inside the nested effect

```kotlin
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(99)})
eitherTIO.fold(IO.functor(), { "There was an error" }, { it })
// IO(99)
```

---

# EitherT :: fold

The second argument is a function that addresses the __Left__ case

```kotlin
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{ stringToInt("a")})
eitherTIO.fold(IO.functor(), { "There was an error" }, { it })
// IO("There was an error")
```

---

# EitherT :: fold

The third argument is a function that addresses the __Right__ case

```kotlin
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(99)})
eitherTIO.fold(IO.functor(), { "There was an error" }, { it })
// IO(99)
```

---

# EitherT :: map

__map__ allows us to transform __A__ into __B__ in __F<Either<E, A>>__

```kotlin
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(99)})

eitherTIO.map(IO.functor()) { it + 1 }
// IO(Either.right(100))
```

---

# EitherT :: map

When we __map__ over __EitherT__ values that are a __Left__ the transformation is never applied and __Left__ is returned

```kotlin
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.left(StringToNumberError("not good"))})

EitherTIO.map(IO.functor()) { it + 1 }
// IO(Either.left(StringToNumberError("not good")))
```

---

# EitherT :: flatMap

__flatMap__ allows us to compute over the contents of multiple __EitherT<E, * >__ values

```kotlin
val a: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(1)})
val b: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(2)})

a.flatMap(IO.monad()) { one ->
  b.map(IO.functor(), { two ->
    one + two
  })
}.fix()

// IO(Either.right(3))
```

---

# EitherT :: Monad binding

Λrrow allows imperative style comprehensions to make computing over __EitherT__ values easy.

```kotlin

val a: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(1)})
val b: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(2)})
val c: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(3)})

EitherT.monad<ForIO, StringToNumberError>(IO.monad()).binding {
     val one = a.bind()
     val two = b.bind()
     val three = c.bind()
     one + two + three
}.fix()
// IO(Either.right(6))
```

---

# EitherT :: Monad binding

Each call to __bind()__ is a coroutine suspended function which will bind to it's value only if the __EitherT__ is a __IO(Left)__

```kotlin
val a: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(1)})
val b: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(2)})
val c: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(3)})

EitherT.monad<ForIO, StringToNumberError>(IO.monad()).binding {
     val one = a.bind()
     val two = b.bind()
     val three = c.bind()
     one + two + three
}.fix()
// IO(Either.right(6))
```

---

# EitherT :: Monad binding

If any of the values is __Left__, __bind()__ will shortcircuit yielding __Left__

```kotlin
val a: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(1)})
val b: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.left(StringToNumberError("not good"))})
val c: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(3)})

EitherT.monad<ForIO, StringToNumberError>(IO.monad()).binding {
     val one = a.bind()
     val two = b.bind()
     val three = c.bind()
     one + two + three
}.fix()
// IO(Either.left(StringToNumberError("not good")))
```

---

# EitherT :: Applicative Builder

Λrrow contains methods that allow you to preserve type information when computing over different __EitherT__ typed values.

```kotlin
data class Person(val id: UUID, val name: String, val year: Int)

val eitherTId: EitherT<ForIO, Throwable, UUID> = EitherT(IO{Either.right(UUID.randomUUID())})
val eitherTName: EitherT<ForIO, Throwable, String> = EitherT(IO{Either.right("Peter Parker")})
val eitherTAge: EitherT<ForIO, Throwable, Int> = EitherT(IO{Either.right(23)})

EitherT.applicative<ForIO, Throwable>(IO.monad()).map(eitherTId, eitherTName, eitherTAge, { (id, name, age) ->
    Person(id, name, age)
}).fix()

// IO(Either.right(Person(<uuid>, "Peter Parker", 23)))
```

---

# EitherT :: Conclusion

- EitherT is __used to model computations with two possible results, the Left and the Right__
- __fold__, __map__, __flatMap__ and others are used to compute over the internal contents of an EitherT value.
- __EitherT.monad(F.monad()).binding { ... } Comprehensions__ can be __used to imperatively compute__ over multiple EitherT values in sequence.
- __EitherT.applicative(F.monad()).map { ... }__ can be used to compute over multiple EitherT values preserving type information and __abstracting over arity__ with `map`

---

# EitherT :: Conclusion

- __All techniques demonstrated are also available to other data types__ such as `Try`, `Either`, `IO` and you can build adapters for any data types
- We can learn more about other data types like `Try`, `Either`, `IO` and type classes that power these abstraction such as `Functor`, `Applicative` and `Monad` in other videos.
- __Λrrow encourages a unified programming model__ in which you can solve problems cohesively in all contexts following Typed Functional Programming principles applied to the Kotlin Programming Language.

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow 
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @vejeta : [https://twitter.com/vejeta](https://twitter.com/vejeta)
