# Intro

Welcome to the series of videos about functional programming in Kotlin with Arrow. 
Arrow is a library packed with data types and type classes bringing typed functional programming to Kotlin. 
In this video, we're going to learn about the Kleisli data type and what it's used for.

#Slide 1

Kleisli is a data type used in Λrrow to model a sequence of chained functions 
of the shape from A to F<B> where A is the result of a previously executed computation 
and F<B> represents any data type that has a type argument such as IO, Option, etc.

#Slide 2

Kleisli represents an arrow from <D> to a monadic value Kind<F, A>.
That means, when we create a Kleisli<Id,Int,Double>
we are wrapping a value of Integer to Id<Double>.

#Slide 3

Inside the Kleisli, we specify the transformation.
If we want to transform from the Integer to the Id<Double>, 
we create a Kleisli with a function which receives an Integer as parameter and returns the Id<Double>.

#Slide 4

The local function allows us to do a conversion on the original input value 
inside the Kleisli before it's executed, creating a Kleisli with the input type of the conversion.
In this sample, we are creating a Kleisli which receives a Config object and uses local to transform 
the Config parameter into an Integer or Double.

#Slide 5

The ask function creates a Kleisli with the same input and output type inside the monadic context, 
so you can extract the dependency into a value. 
In this case, we are creating a Kleisli from Config to Option<Config> with ask 
and then extracting the values to do another operation.

#Slide 6

The map function modifies the Kleisli output value with a function
once the Kleisli has been executed.

#Slide 7

The flatMap function maps the Kleisli output into another Kleisli
with the same input type and monadic context.

#Slide 8

The andThen function composes the Kleisli output.
We have different ways to use andThen,
it can be used with another Kleisli like the flatMap function.

#Slide 9

With another function like the map function.

#Slide 10

Or, can be used to replace the Kleisli result.

# Final

Kleisli is really useful for encapsulating a transformation which returns a Monadic value or concatenates them.
In this video, we learned about Kleisli and the different methods to create and use it. 
We'll learn more about those in future videos. Thanks for watching.


