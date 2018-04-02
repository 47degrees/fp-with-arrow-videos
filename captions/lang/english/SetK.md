### Intro (external)

Welcome to the series of videos about functional programming in Kotlin with Arrow.

Arrow is a library, packed with data types and type classes bringing FP to Kotlin.

In this video, we're going to learn about the SetK data type and what it's used for.



### SetK

SetK is a data type belonging to the wrappers category. These types wrap over some of Kotlin’s collections and functions to give them capabilities related to type classes provided by Arrow.



### SetK :: Constructor

A SetK can be initialized through a classic constructor, which expects a Set collection.



### SetK :: k()

There's also availavle a convenient .k() function, that can be called over the Set interface.



### SetK :: just()

Or you can get a SetK, by invoking the SetK .just() method. This method expects only one parameter, and will return a SetK of just one element.



### SetK :: Instances

These are the type classes available for a SetK. Semigroup, SemigroupK, Monoid, MonoidK, and Foldable.

Unlike other wrappers like ListK or SequenceK, SetK does not derive Functor, Applicative, Monad or Traverse. This is due to the lack of ordering guarantees on a Set data structure.



### SetK :: Semigroup

A semigroup is a set `A` with an associative operation, which we will be calling `combine`.

Here, we are combining two Sets, which in our case is performed as a concatenation of the content on each structure.



### SetK :: Monoid

We can define Monoids as combinable objects, that have an empty value, which functions as an identity.

In our code snippet, we can check that by calling the .empty() method we will receive a SetK with no inner values. And then combining it with a SetK, the same SetK will be returned.



### SetK :: SemigroupK, MonoidK

SemigroupK and MonoidK are special cases. They are universal type classes which operates on `kinds`, as the `K` on their names stands for.

The subtle difference between Semigroup and SemigroupK, Monoid and MonoidK, is that while Semigroup and Monoid operate on the values contained in the type contructor, SemigroupK and MonoidK operate exclusively over the entire structure and not the individual items.

As they have enough content on their own, we will cover them deeply in next videos.



### SetK :: Foldable (foldLeft)

A Foldable has a structure from which a value can be computed from visiting each element.

We can fold over a set transforming the values contained in the set to produce a single result. In this case, we have two arguments: the first parameter is the initial value, and the second is a function for transforming the value in each iteration.

In this example we are accumulating a power of each number contained in our set.



### SetK :: Foldable (foldRight)

Also, Foldable type classes offer another method called foldRight. As the name implies, foldRight, starts folding from the right side, and the values are processed in right-to-left order. 

Here, we are again accumulating a power of each number of a set.

But you can see some differences in its use. The reason for this is because, in contrast with foldLeft which operates in an eager way, foldRight will evaluate the passed function lazily, only performing the computation when we call the `value` method. This allows us to support laziness in a stack-safe way.

In Arrow, for this kind of lazy computed values, Eval can be used. For more information on this data type, you can check the dedicated Eval video in the series.



### Outro (external)

In this video, we learned about SetK.

In the same way as with other data types, SetK serves a lot of the functions and different functionality as other higher kinded wrappers.

Arrow proposes a unified programming model for all data types and type classes.

We'll learn more about those in the next videos. Thanks for watching.
