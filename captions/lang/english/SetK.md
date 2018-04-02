### Intro (external)

Welcome to the series of videos about functional programming in Kotlin with Arrow.

Arrow is a library packed with data types and type classes bringing FP to Kotlin.

In this video, we're going to learn about the SetK data type and what it's used for.



### SetK

SetK is a data type belonging to the wrappers category. These types wrap over some of Kotlin’s collections and functions to give them capabilities that are related to the type classes provided by Arrow.



### SetK :: Constructor

A SetK can be initialized through a classic constructor, which expects a Set collection.



### SetK :: k()

There's also a convenient .k() function available, that can be called over the Set interface.



### SetK :: just()

Or you can get a SetK, by invoking the SetK .just() method. This method expects only one parameter, and will return a SetK of just one element.



### SetK :: Instances

The type classes available for a SetK are Semigroup, SemigroupK, Monoid, MonoidK, and Foldable.

Unlike other wrappers, like ListK or SequenceK, SetK does not derive Functor, Applicative, Monad, or Traverse. This is due to the lack of ordering guarantees on a Set data structure.



### SetK :: Semigroup

A semigroup is a set `A` with an associative operation, which we will be calling `combine`.

Here, we are combining two Sets, which in our case is performed as a concatenation of the content on each structure.



### SetK :: Monoid

We can define Monoids as combinable objects that have an empty value, which functions as an identity.

In our code snippet, we can check that by calling the .empty() method, we will receive a SetK with no inner values. And then, when combining it with a SetK, the same SetK will be returned.



### SetK :: SemigroupK, MonoidK

SemigroupK and MonoidK are special cases. They are universal type classes which operate on `kinds`, which is what the `K` in their names stands for.

The subtle difference between Semigroup and SemigroupK, and Monoid and MonoidK, is that while Semigroup and Monoid operate on the values contained in the type contructor, SemigroupK and MonoidK operate exclusively over the entire structure and not the individual items.

We will take an in-depth look at those in future videos, as they have enough content on their own.



### SetK :: Foldable (foldLeft)

A Foldable has a structure from which a value can be computed from visiting each element.

We can fold over a set, transforming the values contained in the set to produce a single result. In this case, we have two arguments: the first parameter is the initial value, and the second is a function for transforming the value in each iteration.

In this example, we are accumulating a power of each number contained in our set.



### SetK :: Foldable (foldRight)

In addition, Foldable type classes offer another method called foldRight. As the name implies, foldRight starts folding from the right side, and processes the values in a right-to-left order. 

Here again, we're accumulating a power of each number of a set.

But, you can see some differences in its use. The reason for this is because, in contrast with foldLeft which operates in an eager way, foldRight will evaluate the passed function lazily, only performing the computation when we call the `value` method. This allows us to support laziness in a stack-safe way.

In Arrow, for these kinds of lazy computed values, Eval can be used. For more information on this data type, you can check out Eval video in this series.



### Outro (external)

In this video, we learned about SetK.

Similar to other data types, SetK serves many of the functions and different functionalities as other higher-kinded wrappers.

Arrow proposes a unified programming model for all data types and type classes.

We'll learn more about those in upcoming videos. Thanks for watching.
