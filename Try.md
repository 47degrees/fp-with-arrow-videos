autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@raulraja](https://twitter.com/raulraja) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/try/](http://arrow-kt.io/docs/datatypes/try/)
slidenumbers: true

# Try 

__`Try`__ is a data type used in __Λrrow__ to model the invocation of a function that may throw exceptions

---

# Try :: ADT

__`Try`__ is modeled as an __Algebraic Data Type__.
In Kotlin ADTs are encoded with sealed class hierarchies.

```kotlin
sealed class Try<out A>
data class Failure<out A>(val exception: Throwable) : Try<A>()
data class Success<out A>(val value: A) : Try<A>()
```

---

# Try :: ADT

This encoding states that a Try can only have 2 possible states.

```kotlin
sealed class Try<out A>
data class Failure<out A>(val exception: Throwable) : Try<A>()
data class Success<out A>(val value: A) : Try<A>()
```

---

# Try :: Failure

If the computation results in an exception __Try__ will
be an instances of `Failure`

```kotlin, [.highlight: 2]
sealed class Try<out A>
data class Failure<out A>(val exception: Throwable) : Try<A>()
data class Success<out A>(val value: A) : Try<A>()
```

---

# Try :: Success

If the computation results in an exception __Try__ will
be an instances of `Failure`

```kotlin, [.highlight: 2]
sealed class Try<out A>
data class Failure<out A>(val exception: Throwable) : Try<A>()
data class Success<out A>(val value: A) : Try<A>()
```

---

# Try :: invoke(() -> A)

We can construct values with __`Try { computation() }`__

```kotlin
val result: Try<Int> = Try { 1 }
// Success(1)
```

---

# Try :: invoke(() -> A)

If computing __`Try { computation() }`__ fails it's represented as a `Failure`

```kotlin
val result: Try<Int> = Try { throw RuntimeException("BOOM!") }
// Failure(RuntimeException("BOOM!))
```

---

# Try :: Transformations

We can transform __Try__ values through several built in functions
- when
- fold
- recover
- map
- flatMap
- Monad # binding
- Applicative # map

---

# Try :: when

We can extract __Try__ inner values by using kotlin's __when__ expressions

```kotlin
val result: Try<Int> = Try { 1 }

when(result) {
    is Failure -> 0
    is Success -> result.b
}
// 1
```

---

# Try :: fold

__fold__ contemplates both __Failure__ and __Success__ cases.

```kotlin
val result: Try<Int> = Try { 1 } 

result.fold(
  { e -> 0 }, 
  { it }
)
// 1
```

---

# Try :: fold

The first argument is a function that addresses the __Failure__ case

```kotlin, [.highlight: 4]
val result: Try<Int> = Try { 1 } 

result.fold(
  { e -> 0 }, 
  { it }
)
// 1
```

---

# Try :: fold

The second argument is a function that addresses the __Success__ case

```kotlin, [.highlight: 5]
val result: Try<Int> = Try { 1 } 

result.fold(
  { e -> 0 }, 
  { it }
)
// 1
```

---

# Try :: recover

__recover__ allows us to handle the exception if it's __Failure__

```kotlin
val result: Try<Int> = Try { throw RuntimeException("BOOM!") } 

result.recover { e -> 0 }
// 0
```

---

# Try :: map

__map__ allows us to transform __A__ into __B__ in __Try< A >__

```kotlin
val result: Try<Int> = Try { 1 } 

val transformed: Try<String> = result.map { "got a $it" }
// "got a 1"
```

---

# Try :: map

When we __map__ over __Try__ in values that are __Failure__ the transformation is never applied and the __Failure__ is returned

```kotlin
val result: Try<Int> = Try { throw RuntimeException("BOOM!") } 

val transformed: Try<String> = result.map { "got a $it" }
// Failure(RuntimeException("BOOM!"))
```

---

# Try :: flatMap

__flatMap__ allows us to compute over the contents of multiple __Try< A >__ values 

```kotlin
val result1: Try<Int> = Try { 1 }
val result2: Try<Int> = Try { 2 } 

result1.flatMap { one -> 
  result2.map { two -> 
    one + two  
  }
}
// Success(3)
```

---

# Try :: Monad binding

Λrrow allows imperative style comprehensions to make computing over Try values easy.

```kotlin
val result1: Try<Int> = Try { 1 }
val result2: Try<Int> = Try { 2 }

Try.monadError().bindingCatch {
    val one = result1.bind()
    val two = result2.bind()
    yields(one + two)
}.ev()
// Success(3)
```

---

# Try :: Monad binding

Each call to __bind()__ is a coroutine suspended function which will bind to it's value only if the __Try__ is a __Success__

```kotlin, [.highlight: 5-6]
val result1: Try<Int> = Try { 1 }
val result2: Try<Int> = Try { 2 }

Try.monadError().bindingCatch {
    val one = result1.bind()
    val two = result2.bind()
    yields(one + two)
}.ev()
// Success(3)
```

---

# Try :: Monad binding

If any of the values is __Failure__, __bind()__ will shortcircuit yielding __Failure__

```kotlin, [.highlight: 2, 6, 9]
val result1: Try<Int> = Try { 1 }
val result2: Try<Int> = Try { throw RuntimeException("BOOM!") }

Try.monadError().bindingCatch {
    val one = result1.bind()
    val two = result2.bind()
    yields(one + two)
}.ev()
// Failure(RuntimeException("BOOM!"))
```

---

# Try :: Monad binding

__bindingCatch__ captures all thrown exceptions and converts them to __Failure__

```kotlin, [.highlight: 2, 6, 9]
val result1: Try<Int> = Try { 1 }
val result2: Try<Int> = Try { 1 }

Try.monadError().bindingCatch {
    val one = result1.bind()
    val two = result2.bind()
    throw RuntimeException("BOOM!")
    yields(one + two)
}.ev()
// Failure(RuntimeException("BOOM!"))
```

---

# Try :: Conclusion

- Try is __used to capture computation that throw exceptions__ 
- We can easily construct values of `Try` with `Success(1)`, `Failure(1)`.
- __fold__, __map__, __flatMap__ and others are used to compute over the internal contents of a Try computation.
- __Try.monadError().bindingCatch { ... } Comprehensions__ can be __used to imperatively compute__ over multiple Try values in sequence.

---

# Try :: Conclusion

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
- @raulraja : [https://twitter.com/raulraja](https://twitter.com/raulraja)
