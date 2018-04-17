# Intro

Welcome to the series of videos about functional programming in Kotlin with Arrow. 
Arrow is a library packed with data types and type classes bringing typed functional programming to Kotlin. 
In this video, we're going to learn about the Kleisli data type and what it's used for.

#Slide 1

Kleisli is a data type used in Λrrow to model a sequence of chained functions 
of the shape from A to F<B> 
This allows us to chain transformations of types with a Monadic Context.

#Slide 2

Kleisli represents an arrow from <D> to a monadic value Kind<F, A>.
That means, inside the Kleisli, we transform the type D into the type A inside F

#Slide 3

Inside the Kleisli, we specify the transformation.
We can specify a function from the input type to the Monadic context with the output type.
For example, a Kleisli<Id,Int,Double>,
contains a function which receives, as parameter, an Int and returns the Id<Double>.

#Slide 4

We can use the Applicative Builder to create a Kleisli. 
In this case, we specify the monadic context and the input type.
This creates a Kleisli Applicative Instance 
that we can use to create a Kleisli with functions like `map`.
We are using `map`, in this case, 
to get the result of the other two Kleisli and return the sum of both.

#Slide 5

The local function allows us to do a conversion on the original input value before it's executed, 
creating a Kleisli with the input type of the conversion and the same context and output type.
We can create a Kleisli which receives a Config object and uses local to transform 
the Config parameter into an Int or Double, before the Kleisli k1 or k2 is executed.

#Slide 6

The ask function creates a Kleisli with the same input and output types inside the monadic context.
So, if we don´t need to change to another output type, 
we will use ask to get the same type inside the monadic context.

#Slide 7

The map function modifies the Kleisli output value once the Kleisli has been executed.
Then, returns a new Kleisli with the new output type to continue with more transformations

#Slide 8

The flatMap function composes the Kleisli with another Kleisli 
which must have the same input type as the output type 
from the first Kleisli and the same monadic context to create a new one,
with the initial input type and monadic context and the second kleisli output type.

#Slide 9

The andThen function composes the Kleisli output.
We have a few ways to use andThen.
It can be used with another Kleisli like the flatMap function, 
and return another Kleisli.

#Slide 10

With another function like the map function, 
changing the output type and returning a new Kleisli with a new output type.

#Slide 11

Or, it can be used to replace the Kleisli result.
In this case, we ignore the original Kleisli result and only get the specified before.
This is useful if change the result for one of our transformations 
and get a new result inside a new Kleisli.

# Final

Kleisli is really useful for encapsulating a transformation which returns a Monadic value or concatenates them.
In this video, we learned about Kleisli and the different methods to create and use it. 
We'll learn more about those in future videos. Thanks for watching.


