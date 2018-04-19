# Intro

Welcome to the series of videos about functional programming in Kotlin with Arrow. Arrow is a library that is packed with data types and type classes bringing typed FP to Kotlin. In this video, we're going to learn about the Reader type and what it's used for.

# Slide 1
```
Reader

Reader is a higher kinded wrapper around a function. This function will be ran at some point in the future, when we can provide a proper execution context for it.

An execution context could be an input value, a service instance, or any other resource that the funtion could need to be ran.

```

# Slide 2
```
Reader

In Arrow, it is actually defined in the terms of a Kleisli:

typealias Reader<D, A> = ReaderT<IdHK, D, A>
typealias ReaderT<F, D, A> = Kleisli<F, D, A>

Reader is available in the arrow-data module under the import arrow.data.Reader

// gradle
compile ' io.arrow-kt:arrow-data:$arrowVersion'

import arrow.data.Reader
```

# Slide 3
```
ReaderFun

The function that Reader postpons is called `Computation` and has the form of (D) -> A, where D represents a Dependency. This computation is defined as an alias called `ReaderFun`.

fun charExists(): ReaderFun<CharContext, Boolean> = { ctx ->
    ctx.source.contains(ctx.searchChar)
}
```

# Slide 4
```
Reader :: Instantiation

Reader can be instantiated in three different ways.

```

# Slide 5
```
Reader :: using its constructor

val readerInstance1: Reader<Int, String> = readerFun.reader()
```

#Slide 6
```
Reader :: from a ReaderFun

val readerInstance2: Reader<Int, String> = Reader(readerFun)

```

#Slide 7
```

Reader :: Inline

val readerInstance3: Reader<CharContext, String> = Reader { ctx ->
    ctx.source.contains(ctx.searchChar)
}
```

# Slide 8
```
Reader :: Dependency Injection

One of the most useful uses of the Reader monad is dependency injection since we can postpone the execution of a function until we have a resource available.
````

# Slide 9
```
Let’s establish a couple of functions to see how it works.

data class CharContext(val source: String, val searchChar: Char, val replacement: Char)

fun charExists(): ReaderFun<CharContext, Boolean> = { ctx ->
    ctx.source.contains(ctx.searchChar)
}

val charExistsReader = charExists().reader()

The Reader monad will defer the execution of these functions until we call the `run` method by specifying a Context instance as a parameter.
```

# Slide 10
```
Reader :: map

val accesingReader: Reader<CharContext, Int> =
  charExistsReader.map {
    if (it) 1 else 0
  }
```

# Slide 11
```
Reader :: flatMap

val composedReader: Reader<CharContext, String> =
  charExistsReader.flatMap {
    removeCharReader(it)
}
```

# Slide 12
```
Reader :: Monad binding

For this usage example, we are going to define a new reader:

fun countChars(): ReaderFun<CharContext, Int> = { ctx ->
    ctx.source.count { it.toLowerCase() == ctx.searchChar.toLowerCase() }
}

val countCharReader = countChars().reader()

```

# Slide 13
```
Reader :: Monad binding

    val bindingReader: Reader<CharContext, String> =
      Reader()
        .monad<CharContext>()
        .binding {
            val a = charExistsReader.bind()
            removeCharReader(a).bind()
        }
      .fix()
```

# Slide 14
```
Reader :: Applicative Builder

Let's define a couple of more functions to ilustrate the Application Builder

fun generateList(): ReaderFun<CharContext, List<Int>> = { ctx ->
    List(ctx.source.length, { x -> Random(x.toLong()).nextInt() })
}

val listReader = generateList().reader()
```

# Slike 15
```

val applicativeReader: Reader<CharContext, ProcessedResult> =
   Reader()
     .applicative<CharContext>()
     .map(countCharReader, listReader, { (charOccurrences, randomList) ->
         ProcessedResult(charOccurrences, randomList)
     })
     .fix()

```

# Final

In this video, we learned about Reader and the different methods to create it and deal with it. We learned about bind, combine, fold, map and flatMap a Reader.

We'll learn more about those in the next episode. Thanks for watching.
