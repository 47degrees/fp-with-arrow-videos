autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/kleisli/](http://arrow-kt.io/docs/datatypes/kleisli/)
slidenumbers: true

# Kleisli 

`Kleisli` is a data type used in `Λrrow` to model a sequence of chained functions 

of the shape `(A) -> F<B>` where `A` is the result of a previously executed computation 

and `F<B>` represents any data type that has a type argument such as `DeferredK`, `IO`, `ObservableK`, `Option`, etc.

---

# Kleisli

`Kleisli` represents an arrow from `<D>` to a monadic value `Kind<F, A>`.

That means, when we create a `Kleisli<Option,Int,Double>`

we are wrapping a value of `(Int) -> Option<Double>`.

---

# Kleisli

Inside the `Kleisli`, we specify the transformation.

If we want to transform from the `Int` to the `Option<Double>`

```kotlin
val doubleOptionKleisli = Kleisli { number: Int ->
  Some(number.toDouble())
}

val doubleOption = doubleOptionKleisli.run(1)
//Some(1.0)
```

---

# Kleisli : Applicative Builder

We can also create a `Kleisli` with binding and the applicative builder

```kotlin
val intConfigKleisli: Kleisli<ForOption, Config, String> = Kleisli { config: Config -> 
    Some(config.n.toString()) 
  }
  
val doubleConfigKleisli: Kleisli<ForOption, Config, String> = Kleisli { config: Config -> 
    Some(config.d.toString()) 
  }

val doubleOptionKleisli: KleisliApplicativeInstance<ForOption, Config> = 
  Kleisli.applicative<ForOption, Config>(Option.applicative())
  .map(intConfigKleisli,doubleConfigKleisli,{
    it.a + it.b //intConfigKleisli + doubleConfigKleisli 
  }).fix()

val doubleOption:Option<String> = doubleOptionKleisli.run(Config(1, 2.0))
//Some(12.0)
```

---

# Kleisli :: Local

The `local` function allows us to do a conversion on the original input value inside the `Kleisli` before it's executed, 
creating a `Kleisli` with the input type of the conversion.

```kotlin
val k1: Kleisli<ForOption, Int, String> = Kleisli { Some(it.toString()) }
val k2: Kleisli<ForOption, Double, String> = Kleisli { Some(it.toString()) }

data class Config(val n: Int, val d: Double)

val configKleisli: Kleisli<ForOption, Config, String> =
  Kleisli.monad<ForOption, Config>(Option.monad()).binding {
    val a = k1.local<Config> { it.n }.bind()
    val b = k2.local<Config> { it.d }.bind()
    a + b
  }.fix()
  
val composedConfig = configKleisli.run(Config(1,2.0))
//Some(12.0)
```

---

# Kleisli :: Ask

The `ask` function creates a `Kleisli` with the same input and output type 

inside the monadic context, so you can extract the dependency into a value:

```kotlin
val askKleisli = Kleisli.monad<ForOption, Config>(Option.monad()).binding {
    val (n, d) = Kleisli.ask<ForOption, Config>().bind()
    n + d
  }.fix()

val askOption = askKleisli.run(Config(1,2.0))
// Some(3.0)
```

---

# Kleisli :: Map

The `map` function modifies the `Kleisli` output value with a function

once the `Kleisli` has been executed.

```kotlin
import arrow.syntax.functor.map

val mapOption = doubleOptionKleisli.map(Option.functor()) { output -> output + 1.0 }.fix().run(1)
//Some(2.0)
```

---

# Kleisli :: FlatMap

The `flatMap` function maps the `Kleisli` output into another `Kleisli`
 
with the same input type and monadic context:

```kotlin
import arrow.data.fix

val stringOptionKleisli = Kleisli { number: Int ->
  Some(number.toString())
}
  
val strOption = doubleOptionKleisli.flatMap(Option.monad()) { stringOptionKleisli }.fix().run(1)
// Some("1.0")
```

---

# Kleisli :: AndThen

`andThen` composes the `Kleisli` output.

It can be used with another `Kleisli` like the flatMap` function.

```kotlin
import arrow.data.fix

val doubleOptionKleisli = Kleisli { number: Double ->
  Some(number + 1.0)
}
  
val doublePlusOption = doubleOptionKleisli.andThen(Option.monad(), doubleOptionKleisli).fix().run(1)
// Some(2.0)
```

---

# Kleisli :: AndThen

With another function like the `map` function:

```kotlin
val doublePlusOption = doubleOptionKleisli.andThen(Option.monad()) { number: Double ->
     Some(number + 1.0)
  }.fix().run(1)
// Some(2.0)
```

---

# Kleisli :: AndThen

Or can be used to replace the Kleisli result:

```kotlin
val doubleReplaced = doubleOptionKleisli.andThen(Option.monad(), Option(2.0)).fix().run(1)
// Some(2.0)
```

---

# Kleisli :: Conclusion

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow 
- 47 Degrees : [http://47deg.com](http://47deg.com)
