autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@raulraja](https://twitter.com/raulraja) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/nonemptylist/](http://arrow-kt.io/docs/datatypes/nonemptylist/)
slidenumbers: true

# NonEmptyList 

__NonEmptyList__ is a data type used in __Λrrow__ to model ordered lists that guarantee to have at least one value.

---

# NonEmptyList

__NonEmptyList__ is available in the `arrow-data` module under the `import arrow.data.NonEmptyList`

```groovy
// gradle
compile 'io.arrow-kt:arrow-data:$arrowVersion'
```

```kotlin
// namespace
import arrow.data.NonEmptyList
```

---

# NonEmptyList :: of

A __NonEmptyList__ guarantees the list always has at least 1 element.

```kotlin
NonEmptyList.of(1, 2, 3, 4, 5) // NonEmptyList<Int>
NonEmptyList.of(1, 2) // NonEmptyList<Int>
NonEmptyList.of() // does not compile
```

---

# NonEmptyList :: head

Unlike __List#[0]__, __NonEmptyList#head__ it's a safe operation that guarantees no exception throwing.

```kotlin
val nel = NonEmptyList.of(1, 2, 3, 4, 5)
nel.head // 1
```

---

# NonEmptyList :: foldLeft

Whe we fold with turn a __NonEmptyList< A >__ into __B__ by providing a __seed__ value and a __function__ that carries the state on each iteration over the elements of the list.

```kotlin
fun sumNel(nel: NonEmptyList<Int>): Int = 
  nel.foldLeft(0) { acc, n -> acc + n }

sumNel(NonEmptyList.of(1, 1, 1, 1)) 
// 4
```

---

# NonEmptyList :: foldLeft

The first argument is a function that addresses the __seed value__, this can be any object of any type which will then become the resulting typed value

```kotlin
fun sumNel(nel: NonEmptyList<Int>): Int = 
  nel.foldLeft(0) { acc, n -> acc + n }

sumNel(NonEmptyList.of(1, 1, 1, 1)) 
// 4
```

---

# NonEmptyList :: foldLeft

The second argument is a function that takes the current state and element in the iteration and returns the new state after transformations have been applied.

```kotlin
fun sumNel(nel: NonEmptyList<Int>): Int = 
  nel.foldLeft(0) { acc, n -> acc + n }

sumNel(NonEmptyList.of(1, 1, 1, 1)) 
// 4
```

---

# NonEmptyList :: map

__map__ allows us to transform __A__ into __B__ in __NonEmptyList< A >__

```kotlin
NonEmptyList.of(1, 1, 1, 1).map { it + 1 }
// NonEmptyList(2, 2, 2, 2)
```

---

# NonEmptyList :: flatMap

__flatMap__ allows us to compute over the contents of multiple __NonEmptyList< * >__ values 

```kotlin
val nelOne: NonEmptyList<Int> = NonEmptyList.of(1)
val nelTwo: NonEmptyList<Int> = NonEmptyList.of(2)

nelOne.flatMap { one -> 
  nelTwo.map { two -> 
    one + two  
  }
}.ev()
// NonEmptyList(3)
```

---

# NonEmptyList :: Monad binding

Λrrow allows imperative style comprehensions to make computing over NonEmptyList values easy.

```kotlin
val nelOne: Option<Int> = Option(1)
val nelTwo: Option<Int> = Option(2)
val nelThree: Option<Int> = Option(3)

Option.monad().binding {
    val one = nelOne.bind()
    val two = nelTwo.bind()
    val three = nelThree.bind()
    yields(one + two + three)
}.ev()
// NonEmptyList(6)
```

---

# NonEmptyList :: Monad binding

Monad binding in `NonEmptyList` and other collection related data type can be used as generators

```kotlin
NonEmptyList.monad().binding {
    val x = NonEmptyList.of(1, 2, 3).bind()
    val y = NonEmptyList.of(1, 2, 3).bind()
    yields(x + y)
}.ev()
// NonEmptyList(6)
```

---

# NonEmptyList :: Applicative Builder

Λrrow contains methods that allow you to preserve type information when computing over different __NonEmptyList__ typed values.

```kotlin
data class Person(id: UUID, name: String, year: Int)

// Note each NonEmptyList is of a different type
val nelId: NonEmptyList<UUID> = NonEmptyList.of(UUID.randomUUID(), UUID.randomUUID())
val nelName: NonEmptyList<String> = NonEmptyList.of("William Alvin Howard", "Haskell Curry")
val nelYear: NonEmptyList<Int> = NonEmptyList.of(1926, 1900)

NonEmptyList.applicative().map(nelId, nelName, nelYear, { (id, name, year) ->
  Person(id, name, year)
}).ev()
// NonEmptyList(
//  Person(<uuid>, "William Alvin Howard", 1926), 
//  Person(<uuid>, "Haskell Curry", 1900)
// )
```

---

# Option :: Conclusion

- NonEmptyList is __used to model list that guarantee at least one element__ 
- We can easily construct values of `NonEmptyList` with `NonEmptyList.of`
- __foldLeft__, __map__, __flatMap__ and others are used to compute over the internal contents of an Option value.
- __NonEmptyList.monad().binding { ... } Comprehensions__ can be __used to imperatively compute__ over multiple NonEmptyList values in sequence.
- __NonEmptyList.applicative().map { ... }__ can be used to compute over multiple NonEmptyList values preserving type information and __abstracting over arity__ with `map`

---

# NonEmptyList :: Conclusion

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
