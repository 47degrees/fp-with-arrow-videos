autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@raulraja](https://twitter.com/raulraja) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/eval/](http://arrow-kt.io/docs/datatypes/eval/)
slidenumbers: true

# Eval 

__`Eval`__ is a data type used in __Λrrow__ to abstract away evaluation strategies .

---

# Eval :: ADT

__`Eval`__ is modeled as an __Algebraic Data Type__.
In Kotlin ADTs are encoded with sealed class hierarchies.

```kotlin
sealed class Eval<out A>
data class Now<out A>(val value: A) : Eval<A>()
data class Later<out A>(val f: () -> A) : Eval<A>()
data class Always<out A>(val f: () -> A) : Eval<A>()
data class Defer<out A>(val thunk: () -> Eval<A>) : Eval<A>()
```

---

# Eval :: Now

__`Eval.now(value)`__ allows us to construct eval instances from already computed values

```kotlin
import arrow.*
import arrow.core.*

val eager = Eval.now(1).map { it + 1 }
eager.value() 
//2
```

---

# Eval :: Later

Delays evaluation of a potentially expensive computation until __`.value()`__ is invoked and memoizes its results

```kotlin
val lazyEvaled = Eval.later { "expensive computation" }
lazyEvaled.value()
// expensive computation
```

---

# Eval :: Always

Defers evaluation of a computation until __`.value()`__ is invoked recomputing it's result each time

```kotlin
val alwaysEvaled = Eval.always { "expensive computation" }
alwaysEvaled.value()
// expensive computation
```

---

# Eval :: Stack safety

Eval is stack safe

```kotlin
fun even(n: Int): Eval<Boolean> =
  Eval.always { n == 0 }.flatMap {
    if(it == true) Eval.now(true)
    else odd(n - 1)
  }

fun odd(n: Int): Eval<Boolean> =
  Eval.always { n == 0 }.flatMap {
    if(it == true) Eval.now(false)
    else even(n - 1)
  }

// Would cause StackOverflowError if not wrapped in Eval
odd(100000).value() 
// false
```

---

# Eval :: Transformations

We can transform __Eval__ values through several built in functions
- map
- flatMap
- Monad # binding
- Applicative # map

---

# Eval :: map

__map__ allows us to transform __A__ into __B__ in __Eval< A >__

```kotlin
import arrow.*
import arrow.core.*

val eager = Eval.now(1).map { it + 1 }
eager.value() 
//2
````

---

# Eval :: flatMap

__flatMap__ allows us to compute over the contents of multiple __Eval< * >__ values 

```kotlin
val evalOne: Eval<Int> = Eval.now(1)
val evalTwo: Eval<Int> = Eval.now(2)

evalOne.flatMap { one -> 
  evalTwo.map { two -> 
    one + two  
  }
}.fix().value()
//3
```

---

# Eval :: Monad binding

Λrrow allows imperative style comprehensions to make computing over __Eval__ values easy.

```kotlin
val evalOne: Eval<Int> = Eval.now(1)
val evalTwo: Eval<Int> = Eval.now(2)
val evalThree: Eval<Int> = Eval.now(3)

Eval.monad().binding {
    val one = evalOne.bind()
    val two = evalTwo.bind()
    val three = evalThree.bind()
    yields(one + two + three)
}.fix().value()
//6
```

---

# Eval :: Applicative Builder

Λrrow contains methods that allow you to preserve type information when computing over different __Eval__ typed values.

```kotlin
data class Person(id: UUID, name: String, year: Int)

// Note each Eval is of a different type
val evalId: Eval<UUID> = Eval.now(UUID.randomUUID())
val evalName: Eval<String> = Eval.now("William Alvin Howard")
val evalAge: Eval<Int> = Eval.now(1926)

Eval.applicative().map(evalId, evalName, evalAge, { (id, name, age) ->
  Person(id, name, age)
}).fix().value()
// Person(<uuid>, "William Alvin Howard", 1926)
```

---

# Eval :: Conclusion

- Eval is __used to abstract away evaluation strategies__
- We can easily construct values of `Eval` with `now`, `later` and `always`
-__map__, __flatMap__ and others are used to compute over the internal contents of an Eval value.
- __Eval.monad().binding { ... } Comprehensions__ can be __used to imperatively compute__ over multiple Eval values in sequence.
- __Eval.applicative().map { ... }__ can be used to compute over multiple Eval values preserving type information and __abstracting over arity__ with `map`

---

# Eval :: Conclusion

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
