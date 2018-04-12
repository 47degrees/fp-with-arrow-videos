autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@raulraja](https://twitter.com/raulraja) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/state/](http://arrow-kt.io/docs/datatypes/either/)
slidenumbers: true

# State

`State` is a structure that provides a functional approach to handling application state. `State<S, A> is basically a function `S -> Tuple2(S, A)`, where `S` is the type that represents your state and `A` is the result the function produces. In addition to returning the result of type `A`, the function returns a new `S` value, which is the updated state.

---

# Stack

As an example, let's build a simple Stack using Arrow's `NonEmptyList` and `Option`:

```
import arrow.*
import arrow.core.*
import arrow.data.*

typealias Stack = Option<Nel<String>>
```

---

# Stack

A Stack needs a `pop` operation:

```
import arrow.*

fun pop(stack: Stack) = stack.fold({
    None toT None
}, {
    Nel.fromList(it.tail) toT it.head.some()
})
```

Invoking `pop` on a `Stack` will return a tuple with the resulting Stack and the previous element that was at its top.

---

# Stack

We also need a `push` operation:

```
fun push(stack: Stack, s: String) = stack.fold({
    Nel.of(s).some() toT Unit
}, {
    Nel(s, it.all).some() toT Unit
})
```

In this case, `push` will also return a tuple containing the new version of the `Stack` (including the pushed element) and an `Unit` value we can discard.

---

# Stack

Let's try those out!

```
fun stackOperations(stack: Stack): Tuple2<Stack, Option<String>> {
    val (s1, _) = push(stack, "a")
    val (s2, _) = pop(s1)
    return pop(s2)
}
```

```
stackOperations(Nel.of("hello", "world", "!").some())
// Tuple2(a=Some(NonEmptyList(all=[world, !])), b=Some(hello))
```

```
stackOperations(Nel.of("hello").some())
// Tuple2(a=None, b=Some(hello))
```

Our `Stack` is inmutable so we need to create a new instance every time we push or pop values from it. For that same reason we have to return the newly created Stack with every operation. We can avoid this with `State`.

---

# Cleaning it up with State

`State`’s special power is keeping track of state and passing it along.

For instance, our `pop` function takes a `Stack` and returns:

- An updated Stack
- A String. 

Thus, it can be represented as `Stack -> Tuple2(Stack, String)`, and therefore matches the pattern `S -> Tuple2(S, A)` where `S` is `Stack` and `A` is `String`.

---

# Stack implemented via State

Let’s write a new version of `pop` and `push` using `State`:

```
import arrow.*

fun pop() = State<Stack, Option<String>> { stack ->
    stack.fold({
        None toT None
    }, {
        Nel.fromList(it.tail) toT it.head.some()
    })
}

fun push(s: String) = State<Stack, Unit> { stack ->
    stack.fold({
        Nel.of(s).some() toT Unit
    }, {
        Nel(s, it.all).some() toT Unit
    })
}
```

---

# State :: composition

The flatMap method on `State<S, A>` lets you use the result of one `State` in a subsequent `State`. The updated state (`S`) after the first call is passed into the second call. These `flatMap` and `map` methods allow us to use State in for-comprehensions:

```
import arrow.typeclasses.*
import arrow.instances.*

fun stackOperations() = State().monad<Stack>().binding {
    val a = push("a").bind()
    val b = pop().bind()
    val c = pop().bind()

    c
}.fix()
```

---

# Interacting with our Stack

In order to interact with any `Stack`, we need to pass an initial stack value, and then we actually apply our operations to it via the `run` function:

```
stackOperations().run(Nel.of("hello", "world", "!").some())
// Tuple2(a=Some(NonEmptyList(all=[world, !])), b=Some(hello))
```

```
stackOperations().run(Nel.of("hello").some())
// Tuple2(a=None, b=Some(hello))
```

If we only care about the resulting String and not the final state, then we can use `runA`:

```
stackOperations().runA(Nel.of("hello", "world", "!").some())
// Some(hello)
```

# State :: map

We can also operate on the results of our `a` without having to run anything on it yet. `map` allows us to transform the resulting value from a change in a `State`, that is: the `A` in our tuple to a different type `B`:

```
stackOperations()
	.map { value -> value.toInt() }
	.run(Nel.of("1").some())
// Tuple2(a=None, b=Some(1))
```

---

# State :: Available instances

`State` has instances for the following typeclasses, and thus has already implemented all their associated operations:

- `Applicative`
- `Functor`
- `Monad`
- `MonadState`

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