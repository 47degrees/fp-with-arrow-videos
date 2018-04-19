# Intro

TODO: to be filled with the actual intro, we recorded that without script

---

# Slide 1

State is a structure that provides a functional approach to handle application state. A State of "S" and "A" is basically a function that takes a state "S" and returns a tuple containing a new evolution of that state and the result of whatever the function produces.

This data type is purely functional and allows us to chain operations, passing the state as arguments or returning it. It's a great way of avoiding mutation and the use of variables.

---

# Slide 2

If you notice from its definition, a State is nothing more than an instance of "StateT" within the context of "Id". You'll be able to learn more about the StateT data type in upcoming videos.

---

# Slide 3

To better illustrate the use of this data type, let's create a function that handles a counter with an accumulated value (which would be our state `S`),  and adds a value to it. The result of this operation would be our "A". Our function returns a simple "State<Int, Int>", in which we simply add the provided value to the counter, and return a new version of the state and the operation bundled together in a tuple. 

The state and the result of the operation are not usually the same, but we'll keep it this way for clarity.

---

# Slide 4

In order to run our operations, we call the "runM" method of the State. As you can see in this example, by running the State we defined in our previous slide and providing the value "2" we'll get in return a tuple with the update State and the operation result.

---

# Slide 5

As we can expect from a functional data type, State provides several ways to compose and transform the result of operations in our States. Some of them include: map, flatMap, Monadic for comprehensions, and the use of Applicative maps.

---

# Slide 6

Map allows us to modify the results of our States without having to run anything in them, by transforming the resulting value from a change in a State (that is, the "A" in our tuple) to a different type "B". In the following example, we're transforming our Integer results into String values. But those transformations won't be actually applied until we actually run our State operations.

---

# Slide 7

FlatMap allows us to chain multiple operations on States instances and combine their resulting values. For illustration purposes we've added an additional stateful operation, in this case a multiplication operation.

Then we can then combine both the addition and multiplication operations to obtain a new function that squares the values we got from the addition operation.

---

# Slide 8

Arrow allows us to combine several operations in a State in a more readable and imperative-looking way with the use of Monad bindings. Each call to bind() in our example is a coroutine suspended function which will bind to its value after each `State` has been updated to its new values. The operations are performed and combined with a mix of FlatMap and Map calls.

--- 

# Slide 9

Λrrow also offers us ways to combine the results of several subsequent operations over a `State` instance, by the use of the Map operation within the context of an Applicative. In our example, we're combining the results of several stateful operations together in a pipe-like way with ease.

---

# Slide 10

As is usual for the data types in Arrow, State has instances for several typeclasses, thus implementing all their associated operations. In our case these instances are:

- `Applicative`
- `Functor`
- `Monad`

---

# Slide 11

- State is a data type that can be used to model state and handle its changes in a functional way.
- In order to create State instances, we define a function that takes a State "S" as an input and returns a tuple that wraps together the next evolution of the State and a resulting value of the operation.
- State also implements functions like map and flatMap that let us transform the resulting value of a State, while also combining them with other State instances.
- Monad for comprehensions can be used to imperatively define a chain of transformations over a State in sequence.
- Applicative maps can also be used to combine the result of an arbitrary State change.

--- 

# State 12

- We'll learn more about other data types like Try, Either ir IO, and also those type classes that power their abstractions like Functor, Applicative and Monad in other videos.
- Arrow encourages a unified programming model, in which users can solve problems cohesively in all contexts following Typed Functional Programming principles applied to the Kotlin Programming Language.

---

# Outtro

TODO: to be filled with the actual outtro, we recorded that without script

---
