autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@anamariamarvel](https://twitter.com/anamariamarvel) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/datatypes/id/](http://arrow-kt.io/docs/datatypes/id/)
slidenumbers: true

# Id

__`Id`__ is a data type used in __Λrrow__ to model pure values.

---


# Id

Using this type declaration, we can treat our __`Id`__ type constructor as a **Monad** and as a **Comonad**

```kotlin
A -> Id<A>

val id: Id<Int> = Id.pure(3)
id.map{it + 3}
// Id(value=6)
```

---
# Id :: Transformations

We can transform __Id__ values through several built in functions

- map
- fold
- flatMap
- coflatMap
- Monad binding
- Applicative#map

---

# Id :: map

__map__ allows us to transform __A__ into __B__ in __Id< A >__

```kotlin
val id: Id<Int> = Id.pure(1)

id.map { it + 1 }
// Id(value=2)
```

---


# Id :: flatMap

__flatMap__ allows us to compute over the contents of multiple __Id< * >__ values 

```kotlin
val id1: Id<Int> = Id.pure(1)
val id2: Id<Int> = Id.pure(2)
id1.flatMap { one ->
        id2.map { two ->
            one + two
        }
    }
//Id(value=3)
```

---
# Id :: coflatMap

__coflatMap__ allows us to compute over the contents of multiple __Id< * >__ values 

```kotlin
id.coflatMap { "string" }
// Id(value="string")
```

---

# Id :: Monad Binding
Λrrow allows imperative style comprehensions to make computing over **Id** values easy.


```kotlin
val id1: Id<Int> = Id.pure(1)
val id2: Id<Int> = Id.pure(2)

Id.monad().binding {
        val one = id1.bind()
        val two = id2.bind()
        yields(one + two)
    }.ev()
// Id(value=3)
```

---

# Id :: Monad Binding
Each call to **bind()** is a coroutine suspended function which will bind to it's value.


```kotlin
val id1: Id<Int> = Id.pure(1)
val id2: Id<Int> = Id.pure(2)
    
Id.monad().binding {
        val one = id1.bind()
        val two = id2.bind()
        yields(one + two)
    }.ev()
// Id(value=3)
```

---

# Id :: Applicative Builder

 Λrrow contains methods that allow you to preserve type information when computing over different **Id** typed values.

```kotlin
data class Person(id: UUID, name: String, year: Int)

val idName: Id<String> = Id.pure("William Alvin Howard")
val idAge: Id<Int> = Id.pure(1926)
val idId : Id<UUID> = Id.pure(UUID.randomUUID())

Id.applicative().map(idId, idName, idAge, { (id, name, age) ->
    Person(id, name, age)
})
//Id(value=Person(<uuid>, "William Alvin Howard", 1926))
```

---

# Id :: Conclusion

- Id is used to model pure values. git
- We can easily construct values of `Id` with `pure(1)`
- __map__, __flatMap__ and others are used to compute over the internal contents of an Id value.
- __Id.monad<L>().binding { ... } Comprehensions__ can be __used to imperatively compute__ over multiple Id values in sequence.
- __Id.applicative<L>().map { ... }__ can be used to compute over multiple Id values preserving type information and __abstracting over arity__ with `map`

---

# Id :: Conclusion

- __All techniques demonstrated are also available to other data types__ such as `Try`, `Option`, `IO` and you can build adapters for any data types.
- We will learn more about other data types like `Try`, `Option`, `IO` and type classes that power these abstraction such as `Functor`, `Applicative` and `Monad` in other videos.
- __Λrrow encourages a unified programming model__ in which you can solve problems cohesively in all contexts following Typed Functional Programming principles applied to the Kotlin Programming Language.

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow 
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @anamariamarvel : [https://twitter.com/raulraja](https://twitter.com/anamariamarvel)
