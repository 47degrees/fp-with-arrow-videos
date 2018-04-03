autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/kleisli/](http://arrow-kt.io/docs/datatypes/kleisli/)
slidenumbers: true

# Intro

Welcome to the series of videos about functional programming in Kotlin with Arrow. 
Arrow is a library that is packed with data types and type classes bringing typed FP to Kotlin. 
In this video, we're going to learn about the Kleisli data type and what it's used for.

# Kleisli 

```
__`Kleisli`__ is a data type used in __`Λrrow`__ to model a sequence of chained functions 

of the shape __`(A) -> F<B>`__ where __`A`__ is the result of a previously executed computation 

and __`F<B>`__ represents any data type that has a type argument such as __`DeferredK`__, __`IO`__, __`ObservableK`__, __`Option`__, etc.
```

Kleisli is a data type used in Λrrow to model a sequence of chained functions 
of the shape from A to F<B> where A is the result of a previously executed computation 
and F<B> represents any data type that has a type argument such as IO, Option, etc.

---

# Kleisli

```
__`Kleisli represents an arrow from __`<D>`__ to a monadic value __`Kind<F, A>`__.

That means, when we create a __`Kleisli<Id,Int,Double>`__

we are wrapping a value of __`(Int) -> Id<Double>`__.
```

Kleisli represents an arrow from <D> to a monadic value Kind<F, A>.
That means, when we create a Kleisli<Id,Int,Double>
we are wrapping a value of Int to Id<Double>.

---

# Kleisli

```
Inside the __`Kleisli`__, we specify the transformation.

If we want to transform from the __`Int`__ to the __`Id<Double>`__

val doubleIdKleisli = Kleisli { number: Int ->
  Id.pure(number.toDouble())
}

val doubleId = doubleIdKleisli.run(1)
//Id(1.0)
```

Inside the Kleisli, we specify the transformation.
If we want to transform from the Int to the Id<Double>, 
we create a Kleisli with a function which receives an Integer as parameter and returns the Id<Double>.

---


# Kleisli :: Local

```
The __`local`__ function allows us to do a conversion on the original input value inside the __`Kleisli`__ before it's executed, 
creating a __`Kleisli`__ with the input type of the conversion.

val k1: Kleisli<ForOption, Int, String> = Kleisli { Some(it.toString()) }
val k2: Kleisli<ForOption, Double, String> = Kleisli { Some(it.toString()) }

data class Config(val n: Int, val d: Double)

val configKleisli: Kleisli<ForOption, Config, String> =
  Kleisli.monad<ForOption, Config>().binding {
    val a = k1.local<Config> { it.n }.bind()
    val b = k2.local<Config> { it.d }.bind()
    a + b
  }.fix()
  
val composedConfig = configKleisli.run(Config(1,2.0))
//Some(12.0)
```

The local function allows us to do a conversion on the original input value 
inside the Kleisli before it's executed, creating a Kleisli with the input type of the conversion.
In this sample, we are creating a Kleisli which receives a Config object and uses local to transform 
the Config parameter into an Integer or Double.


---

# Kleisli :: Ask

```
The __`ask`__ function creates a __`Kleisli`__ with the same input and output type 

inside the monadic context, so you can extract the dependency into a value:

val askKleisli = Kleisli.monad<ForOption, Config>().binding {
    val (n, d) = Kleisli.ask<ForOption, Config>().bind()
    n + d
  }.fix()

val askOption = askKleisli.run(Config(1,2.0))
// Some(3.0)
```

The ask function creates a Kleisli with the same input and output type inside the monadic context, 
so you can extract the dependency into a value. 
In this case, we are creating a Kleisli from Config to Option<Config> 
with ask and then extracting the values to do another operation.


---

# Kleisli :: Map

```
The __`map`__ function modifies the __`Kleisli`__ output value with a function

once the __`Kleisli`__ has been executed.

import arrow.syntax.functor.map

val mapId = doubleIdKleisli.map { output -> output + 1.0 }.fix().run(1)
//Id(2.0)
```

The map function modifies the Kleisli output value with a function
once the Kleisli has been executed.

---

# Kleisli :: FlatMap

```
The __`flatMap`__ function maps the __`Kleisli`__ output into another __`Kleisli`__
 
with the same input type and monadic context:

import arrow.data.fix

val stringIdKleisli = Kleisli { number: Int ->
  Id.pure(number.toString())
}
  
val strId = doubleIdKleisli.flatMap({stringIdKleisli},Id.monad()).fix().run(1)
// Id("1.0")
```

The flatMap function maps the Kleisli output into another Kleisli
with the same input type and monadic context:

---

# Kleisli :: AndThen

```
__`andThen`__ composes the __`Kleisli`__ output.

It can be used with another __`Kleisli`__ like the flatMap`__ function.

import arrow.data.fix

val doubleOptionKleisli = Kleisli { number: Double ->
  Some(number+1.0)
}
  
val doublePlusId = doubleIdKleisli.andThen(doubleOptionKleisli,Option.monad()).fix().run(1)
// Some(2.0)
```

The andThen function composes the Kleisli output.
We have different ways to use andThen,
it can be used with another Kleisli like the flatMap function.


---

# Kleisli :: AndThen

```
With another function like the __`map`__ function:

val doublePlusId =doubleIdKleisli.andThen({
  number: Double -> Some(number+1.0)
}, Option.monad()).fix().run(1)
// Some(2.0)
```

With another function like the map function.

---

# Kleisli :: AndThen

```
Or can be used to replace the Kleisli result:

val doubleReplaced =doubleIdKleisli.andThen(Id(2.0), Id.monad()).fix().run(1)
// Id(2.0)
```

Or, can be used to replace the Kleisli result.

---

# Final

Kleisli is really useful for encapsulating a transformation which returns a Monadic value or concatenates them.
In this video, we learned about Kleisli and the different methods to create and use it. 
We'll learn more about those in the next episode. Thanks for watching.

# Kleisli :: Conclusion

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow 
- 47 Degrees : [http://47deg.com](http://47deg.com)
