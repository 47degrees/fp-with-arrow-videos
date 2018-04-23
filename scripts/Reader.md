# Intro

TODO: to be filled with the actual intro, we recorded that without script

---

# Slide 1

__`Reader`__ is a higher kinded wrapper around a function that goes from A to B. The execution of this function will be deferred until a resource is available. This resource is called Dependency.

This data type is purely functional and allows us to chain operations, allowing us to define dependenct contexts, which makes the Reader monad a great way to define Dependency Injenction.

---

# Slide 2

__Λrrow__ provides an implementation of the Reader monad defined in terms of a `Kleisli`.
This __`Reader`__ is available under the `arrow-data`, and we just need to include it as a Gradle dependency and import it from our source code to start taking advantage of it.

---

# Slide 3

The function that __`Reader`__ postpons is called `Computation`
`D` represents a Dependency, and `A` the result of the coputation.

This computation is defined as an alias called __`ReaderFun`__.

Here we can see a ReaderFun example. This `Computation` goes from a `CharContext` to a `Boolean`. CharContext is our Dependency, and Boolean our target type, meaning that charExists needs an instance of `CharContext` to be executed, and it will result in a `true|false` value.

---

# Slide 4

Let's see how we can instantiate a __`Reader`__. There are 2 different ways to define a Reader, both of them based on a `ReaderFun`.
The first one is calling the Reader constructor directly from the ReaderFun.

The second one is passing the `ReaderFun as a parameter to the Reader contructor.

---

# Slide 5

As we commented at the beginning, one of the most common uses of the Reader monad is Dependency Injenction, since we can defer the execution of a computation until a specific resource is available.

---

# Slide 6

Let's see again these couple of resource we already defined. We are going to use them for demo purposes.

Here we are defining a CharContext, which is just the `Dependency` the `charExists` computation needs to be ran.

Then,  we are creating a Reader instance based on that ReaderFun definition. We are using here a CharContext which is just a source string and a character for simpler examples, but we could be defining the Dependency as a Database resource, for instance, or any other dependency our couputation could have.

---

# Slide 7

Map allows us to modify the results of our Reader without having to provice the Dependency it has defined. It will create a new Reader which Dependency will be the original Dependency, and the result the one produced by the function that goes from `A` to `B`, where `A` is the original target type, and `B` the new one.

In the example we are showing, we are mapping over the `charExist` Reader, checking if it's returning `true` or `false`, and returning `1` or `0` depending on the case. Our original Reader goes from a CharContext to a Boolean, but the resulting one after mapping over it, is goes from CharContext to Int.

---

# Outtro

TODO: to be filled with the actual outtro, we recorded that without script

---