autoscale: true
footer:  [@calvellido](https://twitter.com/calvellido) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/setK/](http://arrow-kt.io/docs/datatypes/setK/)
slidenumbers: true

# SetK

__`SetK`__ is a data type used in __Λrrow__ as a higher kinded wrapper around the Kotlin Set collection interface.

---

# SetK :: Constructor

__`SetK`__ can be created by directly passing a __Set__.

```kotlin
SetK(setOf(1, 2, 5, 3, 2))
// SetK(set=[1, 2, 5, 3])
```

---

# SetK :: k()

It can also be created invoking the __`k()`__ function on a __Set__.

```kotlin
setOf(1, 2, 5, 3, 2).k()
// SetK(set=[1, 2, 5, 3])
```

---

# SetK :: just()

Or through the __`just()`__ function on __SetK__.

```kotlin
SetK.just(1)
// SetK(set=[1])
```

---

# SetK :: Instances

__`SetK`__ auto derives the following different type classes instances, giving it all of their capabilities.

* `Semigroup`
* `SemigroupK`
* `Monoid`
* `MonoidK`
* `Foldable`

---

# SetK :: Semigroup

A __`Semigroup`__ for some given type `A` has a single operation (called __`combine`__), which takes two values of type `A`, and returns a value of type `A`. This operation must be guaranteed to be associative.


```kotlin
val evenNumbers = setOf(4, 2, 4, 6).k()
// SetKW(set=[4, 2, 6])
val oddNumbers = setOf(3, 5, 3, 1).k()
// SetKW(set=[3, 5, 1])
val numbers = evenNumbers.combine(oddNumbers)
// SetKW(set=[4, 2, 6, 3, 5, 1])

```
---

# SetK :: SemigroupK

A __`SemigroupK`__ for some given type `A` has a single operation (called __`combine`__), which takes two values of type `A`, and returns a value of type `A`. This operation must be guaranteed to be associative.

The same can be said about `SemigroupK`, but for data types instead of objects.


```kotlin
val evenNumbers = setOf(4, 2, 4, 6).k()
// SetKW(set=[4, 2, 6])
val oddNumbers = setOf(3, 5, 3, 1).k()
// SetKW(set=[3, 5, 1])
val numbers = evenNumbers.combine(oddNumbers)
// SetKW(set=[4, 2, 6, 3, 5, 1])

```
---

# SetK :: Monoid

__`Monoid`__ extends the `Semigroup` type class, adding an __`empty`__ method to semigroup's `combine`. The empty method must return a value that when combined with any other instance of that type returns the other instance.

The same will occur with `MonoidK`, but working with data types instead of objects.

```kotlin
val primes = setOf(7, 2, 5, 3, 2).k()
// SetK(set=[7, 2, 5, 3])
val emptySet = primes.empty()
// SetKW(set=[])
val combined = primes.combine(emptySet)
// SetK(set=[7, 2, 5, 3])

```
---

# SetK :: MonoidK

__`MonoidK`__ extends the `SemigroupK` type class, adding an __`empty`__ method to semigroup's `combineK`. The empty method must return a value that when combined with any other data type returns the other one.

The same will occur with `MonoidK`, but working with data types instead of objects.

```kotlin
val primes = setOf(7, 2, 5, 3, 2).k()
// SetK(set=[7, 2, 5, 3])
val emptySet = primes.empty()
// SetKW(set=[])
val combined = primes.combine(emptySet)
// SetK(set=[7, 2, 5, 3])

```
---

# SetK :: Foldable (foldLeft)

In a __`Foldable`__ we can compute a single value from visiting each element.

`foldLeft` eagerly folds the SetK content from __left to right__.


```kotlin
val primes = setOf(7, 2, 5, 3, 2).k()
// SetK(set=[7, 2, 5, 3])
val foldLeft = primes.foldLeft(0, {sum, number -> sum + (number * number)})
// 87

```

---

# SetK :: Foldable (foldRight)


In a __`Foldable`__ we can compute a single value from visiting each element.

`foldRight` lazily folds the SetK content from __right to left__.


```kotlin
val primes = setOf(7, 2, 5, 3, 2).k()
// SetK(set=[7, 2, 5, 3])
val foldRight = primes.foldRight(Eval.now(0), { number, sum -> Eval.now(sum.value() + (number *number)) })
// 87

```

---

# SetK :: Conclusion

- SetK is __used to wrap over Kotlin’s Set collections__ to give them capabilities related to type classes provided by Arrow.
- We can easily construct values of `SetK` with `setOf(2, 5, 2).k()`, `SetK(setOf(2, 5, 2))` or `SetK.pure(2)`.
- `SetK` auto derives different type classes instances like `Semigroup`, `SemigroupK`, `Monoid`, `MonoidK`, and `Foldable` .

---

# SetK :: Conclusion

- __All techniques demonstrated are also available to other data types__ such as `ListK` or `SequenceK`, and you can build adapters for any data types.
- We will learn more about other data types like `MapK`, `SortedMapK`, and type classes that power these abstractions such as `Semigroup`, `Monoid` and `Foldable` in other videos.
- __Λrrow encourages a unified programming model__ in which you can solve problems cohesively in all contexts following Typed Functional Programming principles applied to the Kotlin Programming Language.

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @calvellido : [https://twitter.com/calvellido](https://twitter.com/calvellido)
