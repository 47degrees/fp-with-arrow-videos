autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@aMateoPaolo](https://twitter.com/aMateoPaolo) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/typeclasses/monoid/](http://arrow-kt.io/docs/typeclasses/monoid/)
slidenumbers: true


# Semigroup

__`Semigroup`__ is available in the __arrow-typeclasses__ module.

```groovy
// gradle
compile 'io.arrow-kt:arrow-typeclasses:$arrowVersion'

import arrow.typeclasses.Semigroup
```
^ The Semigroup is provided on the arrow-typeclasses artifact.
^ You can use it as soon as you add it to your build.gradle file.

---

# Semigroup

`Semigroup` is a __Type Class__, so it defines a given behavior.

`Monoid` inherit its combinators. 

```
Semigroup

Monoid
```

^ Semigroup is defined as a Type Class. In pure Functional Programming, type classes define behaviors.
^ Other very well known type classes are Monoids, and we will talk about it later in this same video since they are intimately related.

---

# Semigroup :: combine

`Semigroup` abstracts the ability to __combine__ two values of the same type `A`.

In other words, it provides a combining function for the `A` type:

```kotlin
fun A.combine(b: A): A
```

^ The most important behaviour provided by Semigroup is the combine function.
^ With combine, we can combine/concat/join two A values.
^ Semigroup defines on a type A a combination function which takes as input two elements of A to return an element of the same type A.

---

# Semigroup :: Data types

Arrow provides instances of `Semigroup` defined for many types found in Arrow and the Kotlin std lib. 

- Int
- Double
- Short
- Long
- Sting
- ...and many more.

^ There are instances of Semigroup defined for many types found in Arrow. For example, Int values are combined using addition by default but multiplication is also associative and forms another Semigroup.
^ Basicaly any type that can be combinable can adhere to Semigroup.

---

# Semigroup :: combine

In this example we are showing the main characteristic about `Semigroup`, the `combine` method.
As you can see here, we can apply it for both a `Double` and a `String` type, but remember you can use it with many other types.


```kotlin
with(Double.semigroup()){
    4.5.combine(7.0).combine(2.5)
}
//14.0
```

```kotlin
with(String.semigroup()){
    "This is ".combine("functional ").combine("programming.")
}
//This is functional programming.
```

Additionaly `Semigroup` adds `+` syntax to all types for which a `Semigroup` instance exists

^ This is an example which explain the main characteristic about Semigroup´s combine method in a clear way.
^ We are applying it to a Double and a String in this case, but remember you can use it with many other types.
^ Additionaly Semigroup adds + syntax to all types for which a Semigroup instance exists
^ With a semigroup, we gain the capability to combine, and that is the main value of it.

---

# Semigroup :: Laws

All Typeclasses __must satisfy some mathematical laws__ in order to be considered a Typeclass.

Arrow has tests in place for those laws so typeclasses can keep their integrity over time.

^ When we think about typeclasses, we assume they need to satisfy some mathematical laws in order to be considered a typeclass.
^ Those laws are coded and enforced by tests inside Arrow, so we can ensure typeclasses integrity over time.
^ Some laws could be identity, associativity, composition, and so on.

---

# Semigroup :: Laws

Semigroup satisfies the following law:

- __Associativity__: When adding or multiplying, it doesn't matter how we combine the group of elements.

```kotlin
    (a.combine(b)).combine(c)
```
 must be the same as: 
 
```kotlin
    (a.combine(b.combine(c))
```
 
^ Semigroup satisfies one law: Associativity.
^ The combine function of Semigroup must be associative, which is to say, in pseudo code: "(a.combine(b)).combine(c)" is equivalent to "a.combine(b.combine(c))".

---

# Monoid

__`Monoid`__ is available in the __arrow-typeclasses__ module.

```groovy
// gradle
compile 'io.arrow-kt:arrow-typeclasses:$arrowVersion'

import arrow.typeclasses.Monoid
```
^ At this point, we could talk about Monoid type class.
^ Monoid is provided on the arrow-typeclasses artifact.
^ You can use it as soon as you add it to your build.gradle file.

---

# Monoid

`Monoid` is a __Typeclass__ which extends the power of `Semigroup` providing an additional `empty` method, to semigroup´s `combine`.

```
interface Monoid<A> : Semigroup<A> {
    //...
}
```

^ Monoid extends the semigroup type class, adding an empty method to semigroup´s combine.

---

# Monoid :: empty

The __empty__ method provides a default value to the `combineAll` method. That means that if we try to use `combineAll` but not giving any arguments, or try to combine an empty list, it will use `empty()` method to define the result, since we have a specific value to fall back to.

```kotlin
Int.monoid().combineAll()
//0
```

^ The empty method provides a default value to the combineAll method. So, if using combineAll we are not receiving arguments, or try to combine an empty list, it will use empty method to define the result since we have a specific value to fall back to. In this case the default value would be 0.

---

# Monoid :: combineAll

This is one more example about what we are talking about where we are combining both the __context__ and the __content__, so we need to apply the instance of Monoid both the `Option` and the `Int`.

```kotlin

import arrow.core.*

Option.monoid(Int.monoid()).run {
    listOf<Option<Int>>(Some(24), None, Some(23)).combineAll()
}
//Some(47)
```

^ Here we can see another example about the combineAll method. This is really interesting because in this example we can see that is possible to combine both the context and the content, so we need to apply the instance of Monoid both the Option and the Int. At the end what we obtain in this particular example is a some of 47.

---

# Monoid

The advantage of using these type class provided methods is that we can compose monoids to allow us to operate on more complex types.

```kotlin
import arrow.data.*

with(ListK.foldable()){
  listOf("Λ", "R", "R", "O", "W").k().foldMap(String.monoid(), ::identity)
}
//ΛRROW
```

^ The main adventage of using these type class provided methods, rather than the specific ones for each type, is that we can compose monoids to allow us to operate on more complex types.

---

# Monoid

We can also define our own instances. Let´s use as an example `Foldable`'s `foldMap`, which maps over values accumulating the results, using the available `Monoid` for the type mapped onto. With `foldMap` we can also apply a function to the accumulated value.

```kotlin
with(ListK.foldable()) { 
  listOf(1, 2, 3, 4).k().foldMap(Int.monoid(), {it * 2})
}
//20
```

^ This is also true if we define our own instances. For example, we are going to use foldMap, which maps over values accumulating the results, using the available Monoid for the type mapped onto, and also provides a function which we can apply to that accumulated value (mark "it * 2"). We can see this in the example above where we are multiplying by two that accumulated value, as a result, we would get twenty.

---

# Monoid

Here we can see a more complex example where we are defining a function that returns a tuple, and in this case we can define a `Monoid` for a tuple that will be valid for any tuple where the types it contains also have a `Monoid` available.


```kotlin
fun <A, B> monoidTuple(MA: Monoid<A>, MB: Monoid<B>): Monoid<Tuple2<A, B>> =
  object: Monoid<Tuple2<A, B>> {
  //...
```

^ In this new example, we can see one more way of composing monoids that allow us to operate on more complex types. Using this with a function that produces a tuple, we can define a Monoid for a tuple that will be valid for any tuple where the types it contains also have a Monoid available.

---

# Monoid

Let's have a look to the whole `monoidTuple` function. What we have here is a new `Monoid` in which we are providing more functionality to the `data class` `Tuple2` defining both, the combine extension function and the `empty` method, for a `Tuple2<A, B>`.

```kotlin
fun <A, B> monoidTuple(MA: Monoid<A>, MB: Monoid<B>): Monoid<Tuple2<A, B>> =
  object: Monoid<Tuple2<A, B>> {

    override fun Tuple2<A, B>.combine(y: Tuple2<A, B>): Tuple2<A, B> {
      val (xa, xb) = this
      val (ya, yb) = y
      return Tuple2(MA.run { xa.combine(ya) }, MB.run { xb.combine(yb) })
    }

    override fun empty(): Tuple2<A, B> = Tuple2(MA.empty(), MB.empty())
  }
```  

^ Let's have a deeper look to the monoidTuple function. This is a new Monoid in which we are providing more functionality to the data class Tuple2 defining both, the combine extension function and the empty method, for a Tuple2 of A and B.
^ The implementation of the combine method provides that behaviour to the monoids that conform the Tuple2, and the empty method is overrided here adding that functionallity to the monoids in the Tuple2 of A and B.
^ We can see an example of how this function can be used, in the next slide.

---

# Monoid

In this example, we see a function where the `foldMap` receives a Monoid `M` and a function that takes an element from the list and returns the type of the `Monoid`.
The `foldMap` uses the `Monoid.empty` as base instance and uses the function `Tuple2.combine` for combining the base instance with the result of the function doing it recursively among every element in the list.
This way we are able to combine both values in one pass!

```kotlin
with(ListK.foldable()) {
  val m = monoidTuple(Int.monoid(), String.monoid())
  val list = listOf(1, 1).k()

  list.foldMap(m) { n: Int ->
   Tuple2(n, n.toString())
  }
}
// Tuple2(a=2, b=11)
```

^ In this example, we see a function where the foldMap receives a Monoid M and a function that takes an element from the list and returns the type of the Monoid. In this case is a Tuple composed by a Monoid of Int and a Monoid of String.
^ The foldMap uses the Monoid.empty as base instance and uses the function Tuple2.combine for combining the base instance with the result of the fuction doing it recursiverly among every element in the list.
^ Magically, we are able to combine both values in one pass!

---

# Closing sentence

Both Type Classes, Semigroup and Monoid are really useful to combine, concat or join any type of combinable element.
In this video, we have learned how to use the methods those Types Classes provide.
We will learn more about other Type Classes like `Monad`, `Foldable` or `Applicative` in future videos.
Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @aMateoPaolo : [https://twitter.com/aMateoPaolo](https://twitter.com/aMateoPaolo)
