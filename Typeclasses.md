autoscale: true
footer: ![Arrow](arrow-brand-128x128.png) [@raulraja](https://twitter.com/raulraja) [@47deg](https://twitter.com/47deg) :: [Λrrow](http://arrow-kt.io) :: [http://arrow-kt.io/docs/typeclasses/intro/](http://arrow-kt.io/docs/typeclasses/intro/)
slidenumbers: true

# Type Classes 

A __`Type class`__ is an abstract interface used in __Λrrow__ to generically describe behaviors.

---

# Type Classes 

Arrow provides a `arrow-typeclasses` module which includes the core type classes used in Typed Functional Programming

```groovy
compile 'io.arrow-kt:arrow-typeclasses:$arrow_version' 
```

---

# FP Type Classes

Arrow provides out of the box type classes and instances for many data types covering
most of the concerns used in FP applications.

- __Semigroup__ (combining values)
- __Monoid__ (combination * potential absence)
- __Functor__ (Context transformation)
- __Applicative__ (Computing over independent contextual values)
- __Monad__ (Computing over dependent contextal values)
- __MonadError__ (Managing and handling known errors and unknown exceptions)
- ...

---

# Type Classes

A __Type Class__ is declared using an interface and extending the __TC__ marker trait

```kotlin
import arrow.*
import arrow.typeclasses.*

@typeclass interface Semigroup<A> : TC {
  fun combine(a: A, b: A): A
}
```

---

# Type Classes

All type classes are parametric to a generic type which represents the data type on which behaviors are targeted

```kotlin
import arrow.*

@typeclass interface Semigroup<A> : TC {
  fun combine(a: A, b: A): A
}
```

---

# Instances

Type classes require evidence in the form of instances that prove a given data type can implement such behavior

```kotlin
@instance(ListK::class)
interface ListKSemigroupInstance<A> : Semigroup<ListK<A>> {
    override fun combine(a: ListK<A>, b: ListK<A>): ListK<A> =
      (a + b).k()
}
```

---

# Instances

Once an instance is declared the typeclass can be used in concrete code

```kotlin
semigroup<ForListK>.combine(listOf(1, 2).k(), listOf(3, 4).k())
// [1,2,3,4]
```

---

# Instances

Or polymorphic declarations

```kotlin
inline fun <reified A> uberCombine(a: A, b: A, SG: Semigroup<A> = semigroup()): A = 
  SG.combine(a, b)

uberCombine(listOf(1, 2).k(), listOf(3, 4).k())
// [1,2,3,4]
```

---

# Compile time guarantes

Arrow can verify at compile time type class declarations and their instances with Dagger 2

```kotlin
import javax.inject.*

class MyService<A> @Inject constructor(val SG: Semigroup<A>) {
  fun uberCombine(a: A, b: A): A = SG.combine(a, b)
}
```

---

# Provided instances

Arrow provides out of the box Dagger2 exported instances for most common data types as part of the __ArrowInstances__ module.

```kotlin
import javax.inject.*

@Singleton
@Component(modules = [ArrowInstances::class])
interface Instances {
  fun listKSemigroup(): Semigroup<ListK<A>>
}

object Arrow {
  val instances: Instances = DaggerInstances.builder().build()
}
```

---

# Tagless Final

With __`arrow-dagger`__ we can build Tagless Final style architectures

```kotlin
class Repository<F> @Inject constructor(
  val M: Monad<F>,
  val dataSource: DataSource<F>,
  val log: Log<F>
  ) {
  
  fun saveUser(user: User): Kind<F, Unit> =
    M.flaMap(dataSource.saveUser(user), { user ->
      log.info("user $user saved")
    })

}
```

---

# Syntax

All type classes automatically include a generated syntax interface to make
syntax easier on target types. Syntax is activated in its contained scoped.

```kotlin
class Repository<F> @Inject constructor(
  val MS: MonadSyntax<F>,
  val dataSource: DataSource<F>,
  val log: Log<F>
  ) : MonadSyntax<F> by MS {
  
  fun saveUser(user: User): Kind<F, Unit> =
    dataSource.saveUser(user).flatMap { user ->
      log.info("user $user saved")
    }

}
```

---

# Type Classes :: Conclusion

- Type classes are used to enable ad-hoc polymorphism
- Arrow provides the FP type classes and instances for most known data types
- Type classes enable the Tagless Final patterns which is used to construct entire pure FP architectures.
- `arrow-dagger` is able to verify at compile time if type class instances are missing

---

# Conclusion

- __Λrrow encourages a unified programming model__ in which you can solve problems cohesively in all contexts following Typed Functional Programming principles applied to the Kotlin Programming Language.

---

Thanks for watching!

- Λrrow : [http://arrow-kt.io](http://arrow-kt.io)
- Gitter : [https://gitter.im/arrow-kt/Lobby](https://gitter.im/arrow-kt/Lobby)
- Slack : [https://kotlinlang.slack.com/messages/C5UPMM0A0](https://kotlinlang.slack.com/messages/C5UPMM0A0)
- FP with Arrow 
- 47 Degrees : [http://47deg.com](http://47deg.com)
- @raulraja : [https://twitter.com/raulraja](https://twitter.com/raulraja)
