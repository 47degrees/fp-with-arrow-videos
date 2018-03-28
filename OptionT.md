autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@raulraja](https://twitter.com/raulraja) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/OptionT/](http://arrow-kt.io/docs/datatypes/OptionT/)
slidenumbers: true

# OptionT 

__`OptionT`__ is a data type used in __Λrrow__ to model values of types `F<Option<A>>`

---

# OptionT 

We can use `OptionT` when we find ourselves with an `Option` value nested inside any other
effect.

```kotlin
import arrow.core.*
import arrow.data.*

val optionInDeferred: DeferredK<Option<Int>> = async { Some(1) }.k() 
val optionInList: ListK<Option<Int>> = listOf(Some(1)).k() 
val optionInIO: IO<Option<Int>> = IO(Some(1))
```

---

# OptionT 

We can construct `OptionT` instances by wrapping existing `F<Option<A>>` values

```kotlin
val optionTDeferred: OptionT<ForDeferredK, Int> = OptionT(async { Some(1) }.k())
val optionTList: OptionT<ForListK, Int> = OptionT(listOf(Some(1)).k()) 
val optionTIO: OptionT<ForIO, Int> = OptionT(IO(Some(1)))
```

---

# OptionT :: fold

__fold__ contemplates both __Some__ and __None__ cases inside the nested effect

```kotlin
val optionTIO: OptionT<ForIO, Int> = OptionT(IO(None))
optionTIO.fold({ 0 }, { it }, IO.functor())
// IO(0)
```

---

# OptionT :: fold

The first argument is a function that addresses the __None__ case

```kotlin
val optionTIO: OptionT<ForIO, Int> = OptionT(IO(None))
optionTIO.fold({ 0 }, { it }, IO.functor())
// IO(0)
```

---

# OptionT :: fold

The second argument is a function that addresses the __Some__ case

```kotlin
val optionTIO: OptionT<ForIO, Int> = OptionT(IO(None))

optionTIO.fold({ 0 }, { it }, IO.functor())
// IO(0)
```

---

# OptionT :: getOrElse

__getOrElse__ allows us to provide the default value if __None__

```kotlin
val optionTIO: OptionT<ForIO, Int> = OptionT(IO(None))

optionTIO.getOrElse ({ 0 }, IO.functor())
// IO(0)
```

---

# OptionT :: map

__map__ allows us to transform __A__ into __B__ in __F<Option< A >>__

```kotlin
val optionTIO: OptionT<ForIO, Int> = OptionT(IO(Some(1)))

optionTIO.map(IO.functor()) { it + 1 }
// IO(Some(2))
```

---

# OptionT :: map

When we __map__ over __OptionT__ values that are absent the transformation is never applied and __None__ is returned

```kotlin
val optionTIO: OptionT<ForIO, Int> = OptionT(IO(None))

optionTIO.map(IO.functor()) { it + 1 }
// IO(None)
```

---

# OptionT :: flatMap

__flatMap__ allows us to compute over the contents of multiple __OptionT< * >__ values 

```kotlin
val a: OptionT<ForIO, Int> = OptionT(IO(Some(1)))
val b: OptionT<ForIO, Int> = OptionT(IO(Some(2)))

a.flatMap(IO.functor()) { one -> 
  b.map(IO.functor()) { two -> 
    one + two  
  }
}.fix()
// IO(Some(3))
```

---

# OptionT :: Monad binding

Λrrow allows imperative style comprehensions to make computing over __OptionT__ values easy.

```kotlin
val a: OptionT<ForIO, Int> = OptionT(IO(Some(1)))
val b: OptionT<ForIO, Int> = OptionT(IO(Some(2)))
val c: OptionT<ForIO, Int> = OptionT(IO(Some(3)))

OptionT.monad(IO.monad()).binding {
    val one = a.bind()
    val two = b.bind()
    val three = c.bind()
    one + two + three
}.fix()
// IO(Some(6))
```

---

# OptionT :: Monad binding

Each call to __bind()__ is a coroutine suspended function which will bind to it's value only if the __OptionT__ is a __IO(Some)__

```kotlin
val a: OptionT<ForIO, Int> = OptionT(IO(Some(1)))
val b: OptionT<ForIO, Int> = OptionT(IO(Some(2)))
val c: OptionT<ForIO, Int> = OptionT(IO(Some(3)))

OptionT.monad(IO.monad()).binding {
    val one = a.bind()
    val two = b.bind()
    val three = c.bind()
    one + two + three
}.fix()
// IO(Some(6))
```

---

# OptionT :: Monad binding

If any of the values is __None__, __bind()__ will shortcircuit yielding __None__

```kotlin
val a: OptionT<ForIO, Int> = OptionT(IO(Some(1)))
val b: OptionT<ForIO, Int> = OptionT(IO(None))
val c: OptionT<ForIO, Int> = OptionT(IO(Some(3)))

OptionT.monad(IO.monad()).binding {
    val one = a.bind()
    val two = b.bind()
    val three = c.bind()
    yields(one + two + three)
}.fix()
// IO(None)
```

---

# OptionT :: Applicative Builder

Λrrow contains methods that allow you to preserve type information when computing over different __OptionT__ typed values.

```kotlin
data class Person(id: UUID, name: String, year: Int)

// Note each OptionT is of a different type
val maybeId: OptionT<ForIO, UUID> = OptionT(IO(UUID.randomUUID())
val maybeName: OptionT<ForIO, String> = OptionT(IO("William Alvin Howard")
val maybeAge: OptionT<ForIO, Int> = OptionT(IO(1926)

OptionT.applicative(IO.monad()).map(maybeId, maybeName, maybeAge, { (id, name, age) ->
  Person(id, name, age)
}).fix()
// IO(Some(Person(<uuid>, "William Alvin Howard", 1926)))
```

---

# OptionT :: Conclusion

- OptionT is __used to model optional nested values__ 
- __fold__, __map__, __flatMap__ and others are used to compute over the internal contents of an OptionT value.
- __OptionT.monad(F.monad()).binding { ... } Comprehensions__ can be __used to imperatively compute__ over multiple OptionTal values in sequence.
- __OptionT.applicative(F.monad()).map { ... }__ can be used to compute over multiple OptionT values preserving type information and __abstracting over arity__ with `map`

---

# OptionT :: Conclusion

- __All techniques demonstrated are also available to other data types__ such as `Try`, `Either`, `IO` and you can build adapters for any data types
- We will learn more about other data types like `Try`, `Either`, `IO` and type classes that power these abstraction such as `Functor`, `Applicative` and `Monad` in other videos.
- __Λrrow encourages a unified programming model__ in which you can solve problems cohesively in all contexts following Typed Functional Programming principles applied to the Kotlin Programming Language.

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow 
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @raulraja : [https://twitter.com/raulraja](https://twitter.com/raulraja)
