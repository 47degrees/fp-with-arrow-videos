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

val readerFun: ReaderFun<Int, String> = { input: Int -> input.toString() }
```

# Slide 4
```
Reader

Reader can be instantiated from:

- its constructor, where `run` is ReaderFun:

val reader1: Reader<Int, String> = readerFun.reader()

- from a ReaderFun computation:

val readerValue2: Reader<Int, String> = Reader(readerFun)

- or inline:

val readerValue3: Reader<Int, String> = Reader {
    it.toString()
}
```

# Slide 4
```
Reader :: Dependency Injection

One of the most useful uses of the Reader monad is dependency injection since we can postpone the execution of a function until we have a resource available.
````

# Slide 5
```
Let’s establish a couple of functions to see how it works.

data class Context(val greeting: String, val times: Int)

fun greetingMessage(name: String): Reader<Context, String> = Reader {
    "${it.greeting}, $name."
}

fun printGreeting(message: String): ReaderFun<Context, Unit> = {
    for (x in 0..it.times) {
        println(message)
    }
}

The Reader monad will defer the execution of these functions until we call the `run` method by specifying a Context instance as a parameter.
```

# Slide 6
```
We can additionally combine them into a single Reader instance:

fun salute(name: String): Reader<Context, Unit> = Reader {
    greetingMessage(name).run(it).flatMap { message ->
        printGreeting(message)
    }
}
```

# Slide 7
```
Reader :: map
```

# Slide 8
```
Reader :: flatMap
```

# Slide 9
```
Reader :: Monad binding
```

# Slide 10
```
Reader :: AndThen
```

# Final

In this video, we learned about Reader and the different methods to create it and deal with it. We learned about bind, combine, fold, map and flatMap a Reader.

We'll learn more about those in the next episode. Thanks for watching.
