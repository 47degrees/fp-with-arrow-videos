autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@aMateoPaolo](https://twitter.com/aMateoPaolo) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/typeclasses/monoid/](http://arrow-kt.io/docs/typeclasses/monoid/)
slidenumbers: true

# Monoid

__`Monoid`__ is available in the __arrow-typeclasses__ module.

```groovy
// gradle
compile 'io.arrow-kt:arrow-typeclasses:$arrowVersion'

import arrow.typeclasses.Monoid
```

^ Monoid is provided on the arrow-typeclasses artifact.
^ You can use it as soon as you add it to your build.gradle file.

---

# Monoid

`Monoid` is a __Typeclass__ which extends the power of `Semigroup` providing an additional `empty` method, to semigroup´s `combine`. You will learn more about `Semigroup` in other videos in this series.

```
interface Monoid<A> : Semigroup<A> {
    //...
}
```

^ Monoid is defined as a Typeclass, and like all Typeclasses in Functional Programming, has encoded behaviors that we will be talking about in the next slides.
^ It extends the semigroup type class, adding an empty method to semigroup´s combine.

---

# Monoid :: empty

Having an __empty__ defined allows us to combine all the elements of an potentially empty collection of `T` for which a `Monoid <T>` is defined and as a result we would get a `T` element instead of an `Option <T>`, since we have a specific value to fall back to. In this way we do not have to worry about possible empty values in our collection to be combined.

```kotlin
ForString extensions {
    listOf<String>("Try", " ", "Λ", "","R", "R", "O", "W").combineAll()
}
//Try ΛRROW
```

^ And what is special about this empty method? The answer is that it allows us to combine all the elements of an hypothetical empty list of T for which a Monoid T is defined. As a result we would get a T element instead of an Option T, since we have a specific value to fall back to. So we can forget about the concern of having a possible empty value in our collection to combine. (mark empty value)

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

ForListK extensions {
    listOf("Λ", "R", "R", "O", "W").k().foldMap(String.monoid(), ::identity)
}
//ΛRROW
```

^ The main adventage of using these type class provided methods, rather than the specific ones for each type, is that we can compose monoids to allow us to operate on more complex types.

---

# Monoid

We can also define our own instances. Let´s use as an example `Foldable`'s `foldMap`, which maps over values accumulating the results, using the available `Monoid` for the type mapped onto. With `foldMap` we can also apply a function to the accumulated value.

```kotlin
ForListK extensions { 
  listOf(1, 2, 3, 4).k().foldMap(Int.monoid(), {it * 2})
}
//20
```

^ This is also true if we define our own isntances. For example, we are going to use foldMap, which maps over values accumulating the results, using the available Monoid for the type mapped onto, and also provides a function which we can applies to that accumulated value (mark "it * 2"). We can see this in the example above where we are multiplying by two that accumulated value, as a result we would get twenty.

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
The `foldMap` usses the `Monoid.empty` as base instance and uses the function `Tuple2.combine` for combining the base instance with the result of the fuction doing it recursively among every element in the list.
This way we are able to combine both values in one pass!

```
ForListK extensions {
  val M = monoidTuple(Int.monoid(), String.monoid())
  val list = listOf(1, 1).k()

  list.foldMap(M) { n: Int ->
   Tuple2(n, n.toString())
  }
}
// Tuple2(a=2, b=11)
```

^ In this example, we see a function where the foldMap receives a Monoid M and a function that takes an element from the list and returns the type of the Monoid. In this case is a Tuple composed by a Monoid of Int and a Monoid of String.
^ As a note, ForListK extensions allows to invoke the .k function which provides the use of foldMap method we are using in the list. (mark the .k())
^ The foldMap uses the Monoid.empty as base instance and uses the function Tuple2.combine for combining the base instance with the result of the fuction doing it recursiverly among every element in the list.
^ Magically, we are able to combine both values in one pass!

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @aMateoPaolo : [https://twitter.com/aMateoPaolo](https://twitter.com/aMateoPaolo)
