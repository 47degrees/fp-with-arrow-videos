autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@javitaiyou](https://twitter.com/javitaiyou) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/statet/](http://arrow-kt.io/docs/datatypes/statet/)
slidenumbers: true

# StateT

`StateT` is a datatype that is intended to handle application state in a functional way, while also allowing us to operate within the context of a different monad data type.

^ StateT is a datatype that is intended to handle application state in a functional way, while also allowing us to operate within the context of a different monad data type.

---

# StateT as a Monad Transformer

One issue we face working with monads is that they don’t compose. 

* i.e.: trying to combine a `State<S, A>` with an `Either<E, A>` to manage state while handling errors would result in nested structures and unneeded boilerplate.
* There's a simple solution to avoid this issue.

^ One issue we face working with monads is that they don’t compose. For instance, imagine we want to handle a mutable state with the State monad. But operations on our state can fail, so we may also need to handle errors with the Either monad, that provides a branching structure allowing us to differentiate errors (on the left side) and correct results (in the right side).

Combining these two structures will result in nested structures, and we may need to work with unneeded boilerplate in order to operate with our state. Luckily there's a really simple solution to avoid this issue.

---

# StateT :: Example

We'll model a simple elevator, which is nothing but a `data class` specifying the number of available floors and the current floor:

```kotlin
data class Elevator(val floors: Int, val currentFloor: Int)
```

^ Let's see how to deal with all this with an example. We'll model a simple elevator, which is nothing but a data class containing an Integer specifying the number of available floors and another stating the value of the current floor.

---

# StateT :: Example

Our elevator is going to be able to lift up and go down. These operations will support a series of errors in case we try to make it go beyond its upper or lower limits.

```kotlin
sealed class ElevatorError {
    object ElevatorLiftError : ElevatorError()
    object ElevatorDownError : ElevatorError()
}
```

^ As you would imagine, our elevator is going to be able to lift up and go down. But we don't want it to crash on the ground nor the ceiling, so our operations will support a series of errors in case we try to make it go beyond its upper or lower limits.

---

# StateT :: Example

__Lift__ operation based on `Either`:

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

^ The simplest approach would be to model these operations using the `Either` monad. For instance, to lift up our elevator we simply check the current floor of our elevator and see if lifting it up will make it go beyond its limits. In that case we return a Left value containing the corresponding error. Otherwise we return a Right with a tuple containing an updated instance of the elevator and the result of the lift up operation.

---

# StateT :: Example

__Go down__ operation based on `Either`:

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

^ We follow the same pattern to make our elevator go down. We check if the future position of our elevator after applying the operation will exceed its lower limits. If that's the case we return a left value containing the error. Otherwise we return again an updated instance of the elevator and the result of the operation.

---

# StateT :: Example

Combining `Either`-based operations:

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

^ So let's use these functions together now. We can use `flatMap` to operate within the context of each call returning `Either`. In our case we simply care about the updated instances of our elevators so we ignore the results of our operations.
Calling our new function on a three-stories elevator on the ground floor will succeed, as we'll make it go up two floors and then go down one floor twice. But doing the same on a one-story elevator will produce an error, as trying to lift it up two floors will exceed its upper limit.

---

# StateT :: Example

Things get complicated when we try to introduce `State`:

```kotlin
// fun _liftUpS() = Either<ElevatorError, State<Elevator, Int>> = TODO()
```

We've gotten into the kind of nested structures we wanted to avoid, as we'll need to deal with these different contexts in order to combine our operations. We can do better than this using the `StateT` __Monad Transformer__.

^ But until now we haven't tried to keep and handle our internal state using `State`. This is when things get a little bit complicated. As you can see, just the signature of this example shows us that we'll have to deal with a nested structure which will make it hard for us to combine it with other operations. But we can do better than this using the StateT Monad Transformer.

---

# StateT :: Example

`StateT` is defined as follows:

`class StateT<F, S, A>`

Where `F` is another data type that provides an instance for the `Monad` typeclass, `S` represents our state and `A` the result of the operations we'll perform on it.

* `F` -> `Either` (__Monad__)
* `S` -> `Elevator`
* `A` -> `Int`

^ StateT is defined in terms of F (representing another data type that provides an instance for the `Monad` typeclass), S (which will contain our state), and A (which will be the result of the operations we perform to update it). 
The trick here is to use this F placeholder to correspond to our `Either` data type. S will be our state (in this case our elevator), and A will represent the different floors resulting from our lift up and go down operations applied on our state. Let's see new implementations for these methods using this __Monad Transformer__. Notice the function signature on both of them.

---

# StateT :: Example

```kotlin
fun liftUpS(floors: Int = 1) = StateT<EitherPartialOf<ElevatorError>, Elevator, Int>(Either.monad()) { 
	elevator: Elevator ->
	    val tentativeFloor = elevator.currentFloor + floors

	    if (tentativeFloor > elevator.floors) {
	        ElevatorError.ElevatorLiftError.left()
	    } else {
	        (Elevator(elevator.floors, tentativeFloor) toT tentativeFloor).right()
	    }
}
```

^ Our updated operations work in the context of StateT with the following type parameters: an Either able to return ElevatorError, Elevator as our state, and Int. The body of our function (that takes as an input the current state of the elevator) is basically the same as in the previous examples, as we work with Either types to return errors or correct results.

---

# StateT :: Example

```kotlin
fun goDownS(floors: Int = 1) = StateT<EitherPartialOf<ElevatorError>, Elevator, Int>(Either.monad()) { 
	elevator: Elevator ->
	    val tentativeFloor = elevator.currentFloor - floors

	    if (tentativeFloor < 0) {
	        ElevatorError.ElevatorDownError.left()
	    } else {
	        (Elevator(elevator.floors, tentativeFloor) toT tentativeFloor).right()
	    }
}
```

^ The same is applied here to our go down operation. Notice how we haven't had to change anything in our implementation of the operation to work with a Monad Transformer like StateT.

---

# State :: Transformations

We can transform __StateT__ values through several built-in functions:
- map
- flatMap
- Monad # binding
- Applicative # map

^ Let's see how we can use our usual available transformations to modify and combine the values contained in our StateT instances.

---

# StateT :: Map

We can use `map` to define transformations on our values before updating our state:

```kotlin
fun goDownPrint() = goDownS(2).map(Either.monad()) { i ->
    "Current floor: $i"
}

val printResult = goDownPrint().runM(Either.monad(), Elevator(3, 3))
val printResultE = goDownPrint().runM(Either.monad(), Elevator(1, 0))

// printResult = Right(b=Tuple2(a=Elevator(floors=3, currentFloor=1), b=Current floor: 1))
// printResultE = Left(a=ElevatorError$ElevatorDownError@50134894)
``` 

^ As in State we can also use map to transform the values we obtain from the changes in our state before we actually apply them. Let's transform the result of going down two floors to return a more printable result of the change in our state. Also notice how our error handling still keeps doing its job even after specifying our transformation.

---

# StateT :: FlatMap

We can use `flatMap` to combine the previous operations:

```kotlin
fun elevatorOpsFlatMap(): StateT<EitherPartialOf<ElevatorError>, Elevator, Int> {
    val eitherMonad = Either.monad<ElevatorError>()
    return liftUpS(2).flatMap(eitherMonad) { _ ->
        goDownS(1).flatMap(eitherMonad) { _ ->
            goDownS(1)
        }
    }
}

val resultFM = elevatorOpsFlatMap().runM(Either.monad(), Elevator(3, 0))
val resultErrorFM  = elevatorOpsFlatMap().runM(Either.monad(), Elevator(1, 0))

// resultFM = Right(b=Tuple2(a=Elevator(floors=3, currentFloor=0), b=0))
// resultErrorFM = Left(a=ElevatorError$ElevatorLiftError@214c265e)
```

^ Monad Transformers are monads too, so we can use flatMap to combine the previous operations.
Notice how we don't worry about being under the Either context or an State context. Each call will update the original state of our Elevator. See how  both in the success and error cases we get the capabilities of Either to handle errors in our operations, together with the ability of keeping state in a functional way like with State.

---

# StateT :: Monad binding

```kotlin
fun elevatorOpsS() = 
	StateT.monad<EitherPartialOf<ElevatorError>, Elevator>(Either.monad()).binding {
	    liftUpS(2).bind()
	    goDownS(1).bind()
	    val result = goDownS(1).bind()
	    result
}

val result = elevatorOpsS().runM(Either.monad(), Elevator(3, 0))
val resultError = elevatorOpsS().runM(Either.monad(), Elevator(1, 0))

// result = Right(b=Tuple2(a=Elevator(floors=3, currentFloor=0), b=0))
// resultError = Left(a=ElevatorError$ElevatorLiftError@214c265e)
```

Each call to bind() is a coroutine suspended function which will bind to it's value after each state has been updated to their new values.

^ Being a Monad instance it means we can combine operations with our `StateT` instances in a more imperative-looking way with monad bindings. Each call to bind() is a coroutine suspended function which will bind to its value after each state has been updated to their new values.

---

# StateT :: Applicative map

We can chain multiple operations that don't depend on the result of the others:

fun multipleOpsS() = StateT.applicative<EitherPartialOf<ElevatorError>, Elevator>(Either.monad()).map(goDownS(1), liftUpS(1), goDownS(1), {
    (r1, r2, r3) ->
})

val result = multipleOpsS().runM(Either.monad(), Elevator(3, 2))
val resultError = multipleOpsS().runM(Either.monad(), Elevator(3, 0))

// result = Right(b=Tuple2(a=Elevator(floors=3, currentFloor=1), b=kotlin.Unit))
// resultError = Left(a=ElevatorError$ElevatorDownError@50134894)

^ Λrrow also offers us ways to combine the results of several subsequent operations over a StateT instance, by the use of the Map operation within the context of an Applicative. In our example, imagine a user presses several buttons in our imaginary elevator. Thanks to the map operation we can combine the results of these multiple stateful operations together with ease. Notice how if any of the operations fail, the map call will short-circuit and return an error.
---

# State :: Available instances

`StateT` has instances for the following typeclasses, and thus has already implemented all of their associated operations:

- `Applicative`
- `Functor`
- `Monad`

^ StateT has instances for the following typeclasses, allowing us to use all of their associated operations: applicative, functor and monad.

---

# StateT :: Conclusion

- `StateT` allows us to combine operations that handle state in a functional way, together with other monads.
- Monads are not easilly composable so in order to be able to combine them in a easy way, we need __Monad Transformers__ like `StateT`.
- In order to create `StateT` instances, we simply provide the Monad instance for the context we want to operate within.
- Changes in our state are represented by a function that takes a state `S` and returns a tuple combining the future `State` and a resulting value of the operation (`S -> Tuple2<S, A>`).
- Functions like __map__ and __flatMap__ allow us to transform the resulting value of a `StateT` and also combine it with other stateful operations.
- __State.monad().binding { ... } Comprehensions__ can be used to imperatively define a chain of transformations over a `State` in sequence, together with operations that work in other monadic contexts.

^ StateT is really useful to combine operations that handle state in a functional way along with other monads.
Even though composing monads isn't easy, we can use Monad Transformers like StateT to combine State with other structures like Either.
In order to create a `StateT` instance, we simply need to provide the Monad instance for the context we want to operate within.
Changes in our state are represented by a function that takes a state S, and returns a tuple combining the future state and the resulting value of the applied operation.
We can use functions like map and flatMap to transform the results of our stateful operations, and combining them with other stateful operations.
Monad binding comprehensions allows us to define chain of transformations over a state in sequence, together with operations that work in other monadic contexts.

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow 
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @JaviTaiyou : [https://twitter.com/JaviTaiyou](https://twitter.com/JaviTaiyou)
