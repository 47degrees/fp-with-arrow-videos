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
sealed class Validated<out A>
data class Failure<out A>(val exception: Throwable) : Validated<A>()
data class Success<out A>(val value: A) : Validated<A>()
```

---

# Validated :: Conclusion

- Validated is __used to capture computation that throw exceptions__
- We can easily construct values of `Validated` with `Success(1)`, `Failure(1)`.
- __fold__, __map__, __flatMap__ and others are used to compute over the internal contents of a Validated computation.
- __Validated.monadError().bindingCatch { ... } Comprehensions__ can be __used to imperatively compute__ over multiple Validated values in sequence.

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
