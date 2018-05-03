autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@vejeta](https://twitter.com/vejeta) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/eithert/](http://arrow-kt.io/docs/datatypes/EitherT/)
slidenumbers: true

# EitherT

__`EitherT`__ is a data type used in __Λrrow__ for computing an Either that is nested inside another monad.

---

# EitherT

We can use `EitherT` when we find ourselves with an `Either` value nested inside another
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
^ EitherT datatype is used when an Either value is wrapped inside another type. In this sense is similar to the OptionT datatype.
^ We will see later what are the benefits of operating with this type.
^ In this example we can see 3 examples where Either is shown wrapped inside an asynchronous computation, inside a List, or inside an IO.

---

# EitherT

We can construct `EitherT` instances by wrapping existing `F<Either<E, A>>` values:

```kotlin
val eitherTDeferred: EitherT<ForDeferredK, StringToNumberError, Int> = EitherT(async { Either.right(1) }.k())
val eitherTList: EitherT<ForListK, StringToNumberError, Int> = EitherT(listOf(stringToInt("one"), stringToInt("1")).k())
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(1)})
```

^ One way to create this EitherT is to take the values of the previous examples and just pass it to an EitherT constructor, as you can see here.

---

# EitherT :: fold

__fold__ contemplates both __Left__ and __Right__ cases inside the nested effect:

```kotlin
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(99)})
eitherTIO.fold(IO.functor(), { "There was an error" }, { it })
// IO(99)
```

^ So now we will see how we can operate with EitherT. In the first line we see how an EitherT of an IO effect is defined.
^ And then we can use fold to operate directly with the nested Either values.

---

# EitherT :: fold

The second argument is a function that addresses the __Left__ case:

```kotlin
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{ stringToInt("a")})
eitherTIO.fold(IO.functor(), { "There was an error" }, { it })
// IO("There was an error")
```

^ The second argument is a function to operate with the Left case of the nested either.
^ In this example we will return an IO of "There was an error" if the Either is a Left.
^ Since this Either is a left, we will return an IO of "There was an error"

---

# EitherT :: fold

The third argument is a function that addresses the __Right__ case:

```kotlin
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(99)})
eitherTIO.fold(IO.functor(), { "There was an error" }, { it })
// IO(99)
```

^ The third argument is a function to operate with the Right case of the nested either.
^ In this example, we simply return the value of the right case if the Either is a right.
^ Since this Either is a right, when will return an IO of 99.

---

# EitherT :: map

__map__ allows us to transform __A__ into __B__ in __F<Either<E, A>>__:

```kotlin
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.right(99)})

eitherTIO.map(IO.functor()) { it + 1 }
// IO(Either.right(100))
```

^ Another advantage of EitherT is that we can operate over the right values with map.
^ In this example we can see an EitherT defined and how to provide a transformation for the right value.
^ The result of this, and since this Either was a right case, the transformation will be 99 + 1, hence 100.

---

# EitherT :: map

When we __map__ over __EitherT__, values that are __Left__ in the transformation are never applied and __Left__ is returned:

```kotlin
val eitherTIO: EitherT<ForIO, StringToNumberError, Int> = EitherT(IO{Either.left(StringToNumberError("not good"))})

EitherTIO.map(IO.functor()) { it + 1 }
// IO(Either.left(StringToNumberError("not good")))
```
^ We have mentioned that map operates over the right value, so in this example we can see that if the wrapped Either is a left,
^ the defined transformation inside the map will never be applied and that EitherT will return the left case.

---

# EitherT :: flatMap

__flatMap__ allows us to compute over the contents of multiple __EitherT<E, * >__ values:

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

^ Let's go with flatMap. Imagine that we define two EitherT instances.
^ With EitherT and flatMap we can compose them.
^ The result of operating over the first EitherT, if it is a right, we will take the integer and su it with the second.
^ In the examples, the respective right values are: 1 and 2. In the flatMap operation we sum them.

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

^ Composing multiple EitherT can result in a lot of nested code if we would do it with flatMap.
^ There is an easier way to do it with Λrrow, and this is by doing a comprehension with binding.
^ As we see in the code, we compose the values by using .bind() in each EitherT.

---

# EitherT :: Monad binding

Each call to __bind()__ is a coroutine suspended function which will bind to it's value only if the __EitherT__ is a __IO(Right)__:

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

^ With the Monad binding for EitherT we can call bind() sequentially and get the right values for each EitherT instance.

---

# EitherT :: Monad binding

If any of the values are __Left__, __bind()__ will shortcircuit yielding __Left__:

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

^ If any of the EitherT instances is a Left value, the comprehension would shortcircuit yielding the first Left it finds.
^ In this case the result would return the Left value for b.

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

^ So let's proceed with the last example.
^ In this example we have several EitherT with UUID, String and Int in the right values,
^ With Λrrow we can map over the three of them with the Applicative Builder.
^ If all them are contains a right value we could compose them inside a new data class where we see that the types
^ are preserved.


---

# EitherT :: Conclusion

- EitherT is __used to model computations with two possible results, the Left and the Right__.
- __fold__, __map__, __flatMap__, and others are used to compute over the internal contents of an EitherT value.
- __EitherT.monad(F.monad()).binding { ... } Comprehensions__ can be __used to imperatively compute__ over multiple EitherT values in sequence.
- __EitherT.applicative(F.monad()).map { ... }__ can be used to compute over multiple EitherT values preserving type information and __abstracting over arity__ with `map`.

^ So, let's review the concepts:
^ fold, map, flatMap, and others are used to compute over the internal contents of an EitherT value.
^ Monad binding comprehensions can be used to compute over multiple EitherT values in sequence.
^ Applicative can be used to compute over multiple EitherT values preserving type information.

---

# EitherT :: Conclusion

- __All techniques demonstrated are also available to use with other data types__ such as `Try`, `Either`, `IO` and you can build adapters for any data type.
- We can learn more about other data types like `Try`, `Either`, `IO` and the type classes that power these abstractions such as `Functor`, `Applicative`, and `Monad` in other videos.
- __Λrrow encourages a unified programming model__ in which you can solve problems cohesively in all contexts following Typed Functional Programming principles applied to the Kotlin Programming Language.

^ In Conclusion, you will see that the examples here are very similar to other data types, like Try, Either or IO.
^ You can see other videos to learn more about the concepts that power these abstractions.
^ Thanks for watching!

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow 
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @vejeta : [https://twitter.com/vejeta](https://twitter.com/vejeta)
