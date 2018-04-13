autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@raulraja](https://twitter.com/raulraja) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/state/](http://arrow-kt.io/docs/datatypes/either/)
slidenumbers: true

# State

`State` is a structure that provides a functional approach to handling application state. `State<S, A> is basically a function `S -> Tuple2(S, A)`, where `S` is the type that represents your state and `A` is the result the function produces. In addition to returning the result of type `A`, the function returns a new `S` value, which is the updated state.

---

# State

Actually, `State` is just an alias of `StateT` within the context of `Id`:

```
typealias State<S, A> = StateT<IdHK, S, A>
```

We'll learn more about the `StateT` datatype in next videos.

--- 

# State :: invoke

Let's create a function that handles a counter with an accumulated value (which would be our `S`) and adds the provided value to it (being the result our `A`). This results in a simple `State<Int, Int>` in which both the state and the returned value are the same:

```
fun addCount(n: Int): State<Int, Int> = State { acc ->
    acc + n toT acc + n
}

```

---

# State :: changing states

To run an operation on it we use the `runM` function:

```
val result = addCount(1).runM(2)
// result: Id(value=Tuple2(a=3, b=3))
```

---

# State :: Transformations

We can transform __State__ values through several built in functions:
- map
- flatMap
- Monad # binding
- Applicative # map

---

# State :: map

We can also operate on the results of our `State` without having to run anything on it yet. `map` allows us to transform the resulting value from a change in a `State`, that is: the `A` in our tuple to a different type `B`. In this case we'll transform our `Int` results into `String`:

```
val result = addCount(1).runM(2).map { add ->
	add.toString() + "!" 
}
// result: Id(value=Tuple2(a=3, b=3!))
```

---

# State : flatMap

We can compute multiple operations on `State` instances, combining their resulting values. First let's add a different `State` operation to multiply values:

```
fun multiplyCount(n: Int): State<Int, Int> = State { acc ->
    acc * n toT acc * n
}
```

We can combine this with the previously defined `addCount` function using `flatMap`:

```
val result = addCount(1).flatMap { add ->
    multiplyCount(add)
}.runM(2)
// val result = Id(value=Tuple2(a=9, b=9))
```

---

# State :: Monad binding

Each call to bind() is a coroutine suspended function which will bind to it's value after each `State` has been updated to its new values:

```
fun bindings(n: Int) = State().monad<Int>().binding {
    val a = addCount(n).bind()
    val b = multiplyCount(a).bind()
    val c = addCount(b).bind()

    c
}

val result = bindings(2).runM(1)
// result = Id(value=Tuple2(a=18, b=18))
```

---

# State :: Applicative Builder

Λrrow contains methods that allow you to combine the results of several subsequent operations over `State`:

```
val result = State().applicative<Int>().map(addCount(1), addCount(2), multiplyCount(3), { (add1, add2, mult) ->
    "Combined values: $add1, $add2, $mult"
}).runM(1)
// result: Id(value=Tuple2(a=12, b=Combined values: 2, 4, 12))
```

---

# State :: Available instances

`State` has instances for the following typeclasses, and thus has already implemented all their associated operations:

- `Applicative`
- `Functor`
- `Monad`

---

# State :: Conclusion

- `State` is used to model state and handle its changes in a functional way.
- We can create `State` instances by defining a function that takes a `State` `S` and returns a tuple combining the future `State` and a resulting value of the operation (`S -> Tuple2<S, A>`).
- Functions like __map__ and __flatMap__ allow us to transform the resulting value of a `State` and also to combine it with other `State` instances.
- __State.monad().binding { ... } Comprehensions__ can be used to imperatively define a chain of transformations over a `State` in sequence.
- __State.applicative().map { ... }__ allows us to combine the results of multiple `State` changes, __abstracting over arity__ with `map`.

---

# State :: Conclusion

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