autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/kleisli/](http://arrow-kt.io/docs/datatypes/kleisli/)
slidenumbers: true

# Kleisli 

__`Kleisli`__ is a data type used in __Λrrow__ to model a transformation 

from one type into another inside a monadic context

---

# Kleisli :: ADT

__`Kleisli`__ represents an arrow from __`<D>`__ to a monadic value __`Kind<F, A>`__.

That means when we create a `Kleisli<Id,Int,Double>` 

we are doing `<Int> -> Id<Double>`

---

# Kleisli :: ADT

Inside the `Kleisli` we specify the transformation.

If we want to transform from the `Int` to the `Id<Double>`

```kotlin
val doubleIdKleisli = Kleisli { number: Int ->
  Id.pure(number.toDouble())
}

val doubleId = doubleIdKleisli.run(1)
//Id(1.0)
```

---

# Kleisli :: Local

The __`local`__ function allows doing a conversion inside the `Kleisli` 

to the original input value before the Kleisli will be executed,

creating a Kleisli with the input type of the conversion

```kotlin
val localId = doubleIdKleisli.local { optNumber :Option<Int> -> 
optNumber.getOrElse { 0 } 
}.run(None)
//Id(0.0)
```

---

# Kleisli :: Map

The `map` function modify the `Kleisli` output value with a function

once the `Kleisli` has been executed.

```kotlin
import arrow.syntax.functor.map

val mapId = doubleIdKleisli.map { output -> output + 1.0 }.fix().run(1)
//Id(2.0)
```

---

# Kleisli :: FlatMap
The `flatMap` function map the `Kleisli` output into another `Kleisli`
 
with the same input type and monadic context

```kotlin
import arrow.data.fix

val stringIdKleisli = Kleisli { number: Int ->
  Id.pure(number.toString())
}
  
val strId = doubleIdKleisli.flatMap({stringIdKleisli},Id.monad()).fix().run(1)
// Some(1.0)
```

---

# Kleisli :: AndThen

__`andThen`__ compose the `Kleisli` output

It can be used with another `Kleisli` like the `flatMap` function

```kotlin
import arrow.data.fix

val doubleOptionKleisli = Kleisli { number: Double ->
   Some(number+1.0)
}
  
val doublePlusId = doubleIdKleisli.andThen(doubleOptionKleisli,Option.monad()).fix().run(1)
// Some(2.0)
```

---

# Kleisli :: AndThen

With another function like the `map` function

```kotlin
val doublePlusId =doubleIdKleisli.andThen({
number: Double -> Some(number+1.0)
}, Option.monad()).fix().run(1)
// Some(2.0)
```

---

# Kleisli :: AndThen

Or can be used to replace the `Kleisli` result

```kotlin
val doubleReplaced =doubleIdKleisli.andThen(Id(2.0), Id.monad()).fix().run(1)
// Id(2.0)
```

---

# Kleisli :: Conclusion

__`Kleisli`__ is really useful to encapsulate a transformation 

which return a Monadic value or concatenate them

---

# Kleisli :: Conclusion

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow 
- 47 Degrees : [http://47deg.com](http://47deg.com)
