autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@javitaiyou](https://twitter.com/javitaiyou) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/statet/](http://arrow-kt.io/docs/datatypes/statet/)
slidenumbers: true

# StateT

`StateT` is a datatype that is intended to handle application state in a functional way, while also allowing us to operate within the context of a different monad data type.

---

# StateT as a Monad Transformer

One issue we face working with monads is that they don’t compose. For instance, if we want to handle operations in our state that can fail, a good idea would be to wrap these using the `Either` monad. This data type gives us a branching structure that allows us to differentiate errors (on the "left" side) and correct results (in the "right" side).

Trying to combine structures like `Either` and `State` can make your code to get really hairy. But there’s a simple solution, and we’re going to explain how you can use __Monad Transformers__ to alleviate this problem and avoid nested structures and unneeded boilerplate.

---

# StateT :: Example

Let's see all this with an example. We'll model a simple elevator, which is nothing but a `data class` containing an `Integer` specifying the number of available floors and another stating the value of the current floor:

```kotlin
data class Elevator(val floors: Int, val currentFloor: Int)
```

---

# StateT :: Example

As you would imagine, our elevator is going to be able to lift up and go down. We don't want it to crash on the ground nor the ceiling, so our operations will support a series of errors in case we ask it to go beyond its upper or lower limits.

```kotlin
sealed class ElevatorError {
    object ElevatorLiftError : ElevatorError()
    object ElevatorDownError : ElevatorError()
}
```

---

# StateT :: Example

The simplest approach would be to model these operations using the `Either` monad. For instance, to lift up our elevator:

```kotlin
fun liftUpE(elevator: Elevator, floors: Int = 1): Either<ElevatorError, Tuple2<Elevator, Int>> {
    val tentativeFloor = elevator.currentFloor + floors

    return if (tentativeFloor > elevator.floors) {
        Either.left(ElevatorError.ElevatorLiftError)
    } else {
        Either.right(
           Elevator(elevator.floors, tentativeFloor) toT tentativeFloor
        )
    }
}
```

---

# StateT :: Example

We'll follow the same pattern to make our elevator to go down:

```kotlin
fun goDownE(elevator: Elevator, floors: Int = 1): Either<ElevatorError, Tuple2<Elevator, Int>> {
    val tentativeFloor = elevator.currentFloor - floors

    return if (tentativeFloor < 0) {
        Either.left(ElevatorError.ElevatorDownError)
    } else {
        Either.right(Elevator(elevator.floors, tentativeFloor) toT tentativeFloor)
    }
}
```

---

# StateT :: Example

So let's use these functions together now:

```kotlin
fun elevatorOpsE(elevator: Elevator): Either<ElevatorError, Tuple2<Elevator, Int>> {
    return liftUpE(elevator, 2).flatMap { (e1, _) ->
        goDownE(e1, 1).flatMap { (e2, _) ->
            goDownE(e2, 1)
        }
    }
}

val resultE = elevatorOpsE(Elevator(3, 0))
val resultErrorE = elevatorOpsE(Elevator(1, 0))
```

We'll get the following output:

```
ResultE: Right(b=Tuple2(a=Elevator(floors=3, currentFloor=0), b=0))
ResultErrorE: Left(a=ElevatorError$ElevatorLiftError@214c265e)
```

---

# StateT :: Example

That's kind of good, but we haven't tried to keep and handle our internal state using `State`. This is when things get a little bit hairier:


```kotlin
// fun _liftUpS() = Either<ElevatorError, State<Elevator, Int>> = TODO()
```

We've gotten into the kind of nested structures we wanted to avoid, as we'll need to deal with these different contexts in order to combine our operations. We can do better than this using the `StateT` __Monad Transformer__.

---

# StateT :: Example

`StateT` is defined as follows:

`class StateT<F, S, A>`

Where `F` is another data type that provides an instance for the `Monad` typeclass, `S` represents our state and `A` the result of the operations we'll perform on it.

The trick here is use this `F` placeholder to set our `Either` data type which we'll use to handle our possible errors. Let's see new implementations for our lift up and go down methods using this __Monad Transformer__. Notice the function signature on both of them.

---

# StateT :: Example

```kotlin
fun liftUpS(floors: Int = 1) = StateT<EitherPartialOf<ElevatorError>, Elevator, Int>(Either.monad()) { elevator: Elevator ->
    val tentativeFloor = elevator.currentFloor + floors

    if (tentativeFloor > elevator.floors) {
        ElevatorError.ElevatorLiftError.left()
    } else {
        (Elevator(elevator.floors, tentativeFloor) toT tentativeFloor).right()
    }
}
```

---

# StateT :: Example

```kotlin
fun goDownS(floors: Int = 1) = StateT<EitherPartialOf<ElevatorError>, Elevator, Int>(Either.monad()) { elevator: Elevator ->
    val tentativeFloor = elevator.currentFloor - floors

    if (tentativeFloor < 0) {
        ElevatorError.ElevatorDownError.left()
    } else {
        (Elevator(elevator.floors, tentativeFloor) toT tentativeFloor).right()
    }
}
```

---

# StateT :: Monad instance

__Monad Transformers__ are monads too, so we can use `flatMap` to combine the previous operations:

```kotlin
fun elevatorOpsFlatMap(): StateT<EitherPartialOf<ElevatorError>, Elevator, Int> {
    val eitherMonad = Either.monad<ElevatorError>()
    return liftUpS(2).flatMap(eitherMonad) { _ ->
        goDownS(1).flatMap(eitherMonad) { _ ->
            goDownS(1)
        }
    }
}
```

Notice how we don't worry about being under the `Either` context or an `State` context. Our `flatMap` calls __just work__.

---

# StateT :: Monad instance

Each `flatMap` call will update the original state of our Elevator:

```kotlin
val resultFM = elevatorOpsFlatMap().runM(Either.monad(), Elevator(3, 0))
val resultErrorFM  = elevatorOpsFlatMap().runM(Either.monad(), Elevator(1, 0))

// resultFM = Right(b=Tuple2(a=Elevator(floors=3, currentFloor=0), b=0))
// resultErrorFM = Left(a=ElevatorError$ElevatorLiftError@214c265e)
```

If you notice both in the success and error cases we get both the capabilities of `Either` to handle errors in our operations together with the ability of keeping state in a functional way like with `State`. All that without any boilerplate at all.

---

# StateT :: Monad binding

And being a Monad instance it means we can combine operations with our `StateT` instances in a more imperative-looking way:

```kotlin
fun elevatorOpsS() = StateT.monad<EitherPartialOf<ElevatorError>, Elevator>(Either.monad()).binding {
    liftUpS(2).bind()
    goDownS(1).bind()
    val result = goDownS(1).bind()
    result
}
```

Each call to bind() is a coroutine suspended function which will bind to it's value after each `State` has been updated to their new values:

```kotlin
val result = elevatorOpsS().runM(Either.monad(), Elevator(3, 0))
val resultError = elevatorOpsS().runM(Either.monad(), Elevator(1, 0))

// resultFM = Right(b=Tuple2(a=Elevator(floors=3, currentFloor=0), b=0))
// resultErrorFM = Left(a=ElevatorError$ElevatorLiftError@214c265e)
```

---

# StateT :: Map transformations

As in `State` we can also use `map` to transform the values we obtain from the changes in our state. Let's transform the result of going down two floors to return a more printable result of the change in our state. Also notice how the error handling still keeps doing its job even after the transformation happens:

```kotlin
fun goDownPrint() = goDownS(2).map(Either.monad()) { i ->
    "Current floor: $i"
}

val printResult = goDownPrint().runM(Either.monad(), Elevator(3, 3))
val printResultE = goDownPrint().runM(Either.monad(), Elevator(1, 0))

// printResult = Right(b=Tuple2(a=Elevator(floors=3, currentFloor=1), b=Current floor: 1))
// printResultE = Left(a=ElevatorError$ElevatorDownError@50134894)
``` 

---

# State :: Available instances

`StateT` has instances for the following typeclasses, and thus has already implemented all of their associated operations:

- `Applicative`
- `Functor`
- `Monad`

---

# StateT :: Conclusion

- `StateT` is really useful to combine operations that handle states in a functional way together with other monadics data types.
- Monads are not easilly composable so in order to be able to combine them in a easy way, we need __Monad Transformers__ like `StateT`.
- In order to create `StateT` instances, we simply provide the Monad instance for the context we want to operate within.
- Changes in our state are represented by a function that takes a state `S` and returns a tuple combining the future `State` and a resulting value of the operation (`S -> Tuple2<S, A>`).
- Functions like __map__ and __flatMap__ allow us to transform the resulting value of a `StateT` and also combine it with other `StateT` instances.
- __State.monad().binding { ... } Comprehensions__ can be used to imperatively define a chain of transformations over a `State` in sequence, together with operations that work in other monadic contexts.

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow 
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @raulraja : [https://twitter.com/JaviTaiyou](https://twitter.com/JaviTaiyou)

---