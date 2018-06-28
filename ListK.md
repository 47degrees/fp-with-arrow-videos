# Intro

Welcome to the series of videos about functional programming in Kotlin with Arrow. Arrow is a library that is packed with data types and type classes bringing typed FP to Kotlin. In this video, we're going to learn about the List data type and what it's used for.

# Slide 1
```
ListK 

ListKW is available in the arrow-data module under the import arrow.data.ListK 

// gradle
compile ' io.arrow-kt:arrow-data:$arrowVersion' 

import arrow.data.ListK 

```
ListK is a higher kinded wrapper around the List collection interface and, it's available in the arrow data library.

# Slide 2
```
ListK 

ListK can be initialized with the following:

import arrow.*
import arrow.core.*
import arrow.data.*

listOf(1, 2, 3).k()

ListK(listOf(1, 2, 3))

ListK.pure(1)

```
In the following examples, ListK is created from the Kotlin List type with a convenient k() function, using the constructor and invoking the pure method.

# Slide 3
```
ListK :: Monoid 

ListK auto derives its Monoid instance.

val list1 = listOf(1, 2, 3).k()

val list2 = ListKW.pure(1)

list1.combineK(list2)
// 1, 2, 3, 1

```

ListK auto derives its Monoid instance and can be guaranteed to be associative by using the combineK method to combine two different lists.

# Slide 4
```
ListK :: foldable

val numbers= listOf(1, 2, 3).k()

numbers.foldLeft(0) {acc, n  -> acc + n }
// 6

```
We can fold over the list transforming the values contained in the list to produce a single result. In this case, we have two arguments: the first parameter is the initial value and the second value is a function for transforming the value in each iteration.

# Slide 5
```
ListK :: map

map allows us to transform A into B in ListK< A >

val numbers= listOf(1, 2, 3).k()

numbers.map { it * 2 }
// ListK(2, 4, 6)

```

When we map a ListK, we are applying a function to each element in the list in order to change the content value or transform to another type.

# Slide 6

```
ListK :: flatMap 

flatMap allows us to compute over the contents of multiple ListK< * > values.

val listOne= listOf(1, 2, 3).k()
val listTwo= listOf(4, 5, 6).k()

listOne.flatMap { itOne -> listTwo.map { itOne + it } }
numbers.map { it * 2 }

// ListK(list=[5, 6, 7, 6, 7, 8, 7, 8, 9])

```

ListK supports flapMap in order to allow us to compute over different ListK values and execute them sequentially.


# Slide 7

```
ListK :: Monad binding 

val listOne = listOf(1, 2, 3).k()
val listTwo = listOf(4, 5, 6).k()

val result = ListK.monad().binding {
    val v1 = listOne.bind()
    val v2 = listTwo.bind()
    v1 + v2
}
// ListK(list=[5, 6, 7, 6, 7, 8, 7, 8, 9])

```

ListK supports monad comprehensions for composing sequential chain of actions with imperative style.

# Slide 8

```
ListK :: Applicative Builder

val listOne = listOf(1, 2, 3).k()
val listTwo = listOf(4, 5, 6).k()

ListK.applicative().map(listOne, listTwo) {
    it.a + it.b
}
// ListK(list=[5, 6, 7, 6, 7, 8, 7, 8, 9])

```

With Applicative Builder, we can use functions of any number of arguments.

# Slide 9

```
SequenceK

SequenceK implements lazy lists representing lazily-evaluated ordered sequences of homogenous values.

val sequenceOne = sequenceOf(1, 2, 3).k()
val sequenceTwo = sequenceOf(4, 5, 6).k()

sequenceOne.combineK(sequenceTwo )
// 1, 2, 3, 4, 5, 6

```

Finally, you can use SequenceK in the same way as ListK for representing lazily-evaluated ordered sequences.


# Final

In this video, we learned about ListK and SequenceK and the different methods to create it. We learned about combine, fold, and map list.

We'll learn more about those in the next episode. Thanks for watching.

