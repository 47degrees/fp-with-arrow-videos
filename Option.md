autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@raulraja](https://twitter.com/raulraja) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/option/](http://arrow-kt.io/docs/datatypes/option/)
slidenumbers: true

# Option 

__`Option`__ is a data type used in __Λrrow__ to model the potential absence of a value.

---

# Option :: ADT

__`Option`__ is modeled as an __Algebraic Data Type__.
In Kotlin ADTs are encoded with sealed class hierarchies.

```kotlin
sealed class Option<out T>
object None : Option<Nothing>()
data class Some<out T>(val t: T) : Option<T>()
```

---

# Option :: ADT

This encoding states that an Option can only have 2 possible states.

```kotlin
sealed class Option<out T>
object None : Option<Nothing>()
data class Some<out T>(val t: T) : Option<T>()
```

---


# Option :: None

If the value is absent it's represented with the singleton __`None`__.

```kotlin, [.highlight: 2]
sealed class Option<out T>
object None : Option<Nothing>()
data class Some<out T>(val t: T) : Option<T>()
````

---

# Option :: Some

If the value is non absent it's represented with __`Some(value)`__.

```kotlin, [.highlight: 3]
sealed class Option<out T>
object None : Option<Nothing>()
data class Some<out T>(val t: T) : Option<T>()
```

---

# Option :: invoke

We can easily construct values with __`Option(value)`__

```kotlin
val maybeInt: Option<Int> = Option(1)
// Some(1)
```

---

# Option :: invoke

To represent absent values we use __`None`__

```kotlin
val maybeInt: Option<Int> = None
// None
```

---

# Option :: nullable isomorphism

__Option__ is __isomorphic__ to Kotlin __? nullable types__.
We can go from Option to nullable and back without loosing information. 

```kotlin
Option<A> <=> A? 
```

---

# Option :: fromNullable

We use __Option#fromNullable__ to convert nullable values to __Option__.

```kotlin, [.highlight: 2]
val a: Int? = null
val maybeInt: Option<Int> = Option.fromNullable(a)
// None
```

---

# Option :: orNull

We can go back from __Option__ to a __? nullable type__ with __Option#orNull__

```kotlin
val maybeInt: Option<Int> = Option(1)
val result: Int? = maybeInt.orNull()
// 1
```

---

# Option :: Extension syntax

Λrrow provides syntax extensions that allows you to put __A__ values in __Option< A >__

```kotlin
import arrow.syntax.option.*

val maybeInt: Option<Int> = 1.some()
// Some(1)

val maybeInt: Option<Int> = none<Int>()
// None
```

---

# Option :: Transformations

We can transform __Option__ values through several built in functions
- when
- fold
- getOrElse
- map
- flatMap
- Monad # binding
- Applicative # map

---

# Option :: when

We can extract __Option__ inner values by using kotlin's __when__ expressions

```kotlin
val maybeInt: Option<Int> = Option(1)

when(maybeInt) {
    is None -> 0
    is Some -> maybeInt.t
}
// 1
```

---

# Option :: fold

__fold__ contemplates both __Some__ and __None__ cases.

```kotlin
val maybeInt: Option<Int> = Option(1)

maybeInt.fold(
  { 0 }, 
  { it }
)
// 1
```

---

# Option :: fold

The first argument is a function that addresses the __None__ case

```kotlin, [.highlight: 4]
val maybeInt: Option<Int> = Option(1)

maybeInt.fold(
  { 0 }, 
  { it + 1 }
)
// 2
```

---

# Option :: fold

The second argument is a function that addresses the __Some__ case

```kotlin, [.highlight: 5]
val maybeInt: Option<Int> = Option(1)

maybeInt.fold(
  { 0 }, 
  { it + 1 }
)
// 2
```

---

# Option :: getOrElse

__getOrElse__ allows us to provide the default value if __None__

```kotlin
val maybeInt: Option<Int> = None

maybeInt.getOrElse { 0 }
// 0
```

---

# Option :: map

__map__ allows us to transform __A__ into __B__ in __Option< A >__

```kotlin
val maybeInt: Option<Int> = Option(1)

maybeInt.map { it + 1 }
// Some(2)
```

---

# Option :: map

When we __map__ over __Option__ values that are absent the transformation is never applied and __None__ is returned

```kotlin
val maybeInt: Option<Int> = None

maybeInt.map { it + 1 }
// None
```

---

# Option :: flatMap

__flatMap__ allows us to compute over the contents of multiple __Option< * >__ values 

```kotlin
val maybeOne: Option<Int> = Option(1)
val maybeTwo: Option<Int> = Option(2)

maybeOne.flatMap { one -> 
  maybeTwo.map { two -> 
    one + two  
  }
}.ev()
// Some(3)
```

---

# Option :: Monad binding

Λrrow allows imperative style comprehensions to make computing over Option values easy.

```kotlin
val maybeOne: Option<Int> = Option(1)
val maybeTwo: Option<Int> = Option(2)
val maybeThree: Option<Int> = Option(3)

Option.monad().binding {
    val one = maybeOne.bind()
    val two = maybeTwo.bind()
    val three = maybeThree.bind()
    yields(one + two + three)
}.ev()
// Some(6)
```

---

# Option :: Monad binding

Each call to __bind()__ is a coroutine suspended function which will bind to it's value only if the __Option__ is a __Some__

```kotlin, [.highlight: 6]
val maybeOne: Option<Int> = Option(1)
val maybeTwo: Option<Int> = Option(2)
val maybeThree: Option<Int> = Option(3)

Option.monad().binding {
    val one = maybeOne.bind()
    val two = maybeTwo.bind()
    val three = maybeThree.bind()
    yields(one + two + three)
}.ev()
// Some(6)
```

---

# Option :: Monad binding

If any of the values is __None__, __bind()__ will shortcircuit yielding __None__

```kotlin, [.highlight: 2, 7]
val maybeOne: Option<Int> = Option(1)
val maybeTwo: Option<Int> = None
val maybeThree: Option<Int> = Option(3)

Option.monad().binding {
    val one = maybeOne.bind()
    val two = maybeTwo.bind() //shortcircuits because `maybeTwo` is `None`
    val three = maybeThree.bind()
    yields(one + two + three)
}
// None
```

---

# Option :: Applicative Builder

Λrrow contains methods that allow you to preserve type information when computing over different __Option__ typed values.

```kotlin
data class Person(id: UUID, name: String, year: Int)

// Note each Option is of a different type
val maybeId: Option<UUID> = Option(UUID.randomUUID())
val maybeName: Option<String> = Option("William Alvin Howard")
val maybeAge: Option<Int> = Option(1926)

Option.applicative().map(maybeId, maybeName, maybeAge, { (id, name, age) ->
  Person(id, name, age)
}).ev()
// Some(Person(<uuid>, "William Alvin Howard", 1926))
```

---

# Option :: Applicative Builder

If a value turns out to be __None__ the computation short-circuits

```kotlin, [.highlight: 3, 10]
data class Person(id: UUID, name: String, year: Int)

val maybeId: Option<UUID> = None
val maybeName: Option<String> = Option("William Alvin Howard")
val maybeAge: Option<Int> = Option(1926)

Option.applicative().map(maybeId, maybeName, maybeAge, { (id, name, age) ->
  Person(id, name, age)
}).ev()
// None
```

---

# Option :: Conclusion

- Option is __used to model absence__ of a potential value
- We can easily construct values of `Option` with `Option(1)`, `Option.fromNullable(nullable)`, `1.some()` or `none<Int>()`
- __fold__, __map__, __flatMap__ and others are used to compute over the internal contents of an Option value.
- __Option.monad().binding { ... } Comprehensions__ can be __used to imperatively compute__ over multiple Optional values in sequence.
- __Option.applicative().map { ... }__ can be used to compute over multiple Optional values preserving type information and __abstracting over arity__ with `map`

---

# Option :: Conclusion

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
