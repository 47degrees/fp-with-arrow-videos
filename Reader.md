autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@fperezp](https://twitter.com/fperezp) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/reader/](http://arrow-kt.io/docs/datatypes/reader/)
slidenumbers: true

# Reader

__`Reader`__ is a higher kinded wrapper around a function that goes from A to B `(A) -> B`.
This function will be ran at some point in the future, when we can provide a proper execution context for it.

An execution context could be an input value, a service instance, or any other resource that the funtion could need to be ran.

---

# Reader

In __Λrrow__, it is actually defined in the terms of a `Kleisli`:

```kotilin
typealias Reader<D, A> = ReaderT<IdHK, D, A>
typealias ReaderT<F, D, A> = Kleisli<F, D, A>
```

__Reader__ is available in the arrow-data module under the import `arrow.data.Reader`:

```gradle
// gradle
compile ' io.arrow-kt:arrow-data:$arrowVersion'
```

```kotlin
import arrow.data.Reader
```

---

# ReaderFun

The function that __`Reader`__ postpons is called `Computation` and has the form of `(D) -> A`, where `D` represents a Dependency. This computation is defined as an alias called __`ReaderFun`__.

```kotlin

data class CharContext(val source: String, val searchChar: Char, val replacement: Char)

fun charExists(): ReaderFun<CharContext, Boolean> = { ctx ->
    ctx.source.contains(ctx.searchChar)
}
```

---

# Reader :: Instantiation

__`Reader`__ can be instantiated in two different ways.

- Constructor
```kotlin
val readerInstance1: Reader<Int, String> = charExists.reader()
```
- From a ReaderFun
```kotlin
val readerInstance2: Reader<Int, String> = Reader(charExists)
```

---

# Dependency Injection

One of the most useful uses of the __`Reader`__ monad is `dependency injection` since we can postpone the execution of a function until we have a resource available.

---

# Examples helpers

Let’s establish a couple of functions to see how it works.
```kotlin
data class CharContext(val source: String, val searchChar: Char, val replacement: Char)

fun charExists(): ReaderFun<CharContext, Boolean> = { ctx ->
    ctx.source.contains(ctx.searchChar)
}

val charExistsReader = charExists().reader()
```
The __`Reader`__ monad will defer the execution of these functions until we call the `run` method by specifying a Context instance as a parameter.

---

# Reader :: map

When we __map__ over __Reader__ we can operate with the Dependency, resulting in a new __Reader__ with the same Dependency, with a different computation result.

```kotlin
val accesingReader: Reader<CharContext, Int> =
  charExistsReader.map {
    if (it) 1 else 0
  }
```

---

# Reader :: flatMap

__flatMap__ allows us to compute over the contents of multiple __Reader<D, *>__ values, where they need to have the same `Dependency`.

Let's define a new Reader:

```kotlin
fun removeCharReader(remove: Boolean): Reader<CharContext, String> = Reader { ctx ->
    if (remove) ctx.source.replace(ctx.searchChar, ctx.replacement, true)
    else ctx.source
}
```

We can combine this with the previously defined charExistsReader function using flatMap:

```kotlin
val composedReader: Reader<CharContext, String> =
  charExistsReader.flatMap {
    removeCharReader(it)
}
```

---

# Reader :: Monad binding

Λrrow allows imperative style comprehensions to make computing over __Reader__ values easy.

For this usage example, we are going to define a new reader:

```kotlin
fun countChars(): ReaderFun<CharContext, Int> = { ctx ->
    ctx.source.count { it.toLowerCase() == ctx.searchChar.toLowerCase() }
}

val countCharReader = countChars().reader()

```

---

# Reader :: Monad binding

This way, we can define a call cascade using the a __Reader__ result as an input param for a different computation.

```kotlin
val bindingReader: Reader<CharContext, String> =
	Reader().monad<CharContext>().binding {
        val a = charExistsReader.bind()
        removeCharReader(a).bind()
    }.fix()
```

---

# Reader :: Applicative Builder

Λrrow contains methods that allow you to preserve type information when computing over different __Reader__ typed values.

Let's define a couple of more functions to ilustrate the Application Builder

```kotlin
fun generateList(): ReaderFun<CharContext, List<Int>> = { ctx ->
    List(ctx.source.length, { x -> Random(x.toLong()).nextInt() })
}

val listReader = generateList().reader()
```

---

# Reader :: Applicative Builder
```kotlin
val applicativeReader: Reader<CharContext, ProcessedResult> =
   Reader().applicative<CharContext>()
     .map(countCharReader, listReader, { (charOccurrences, randomList) ->
         ProcessedResult(charOccurrences, randomList)
     }).fix()

```

---
