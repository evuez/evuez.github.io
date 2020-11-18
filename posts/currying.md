title = "Currying"
date = 2017-03-30
tags = ["haskell"]
%%%

In Haskell, every function takes a *single parameter*. So how come you can do `max 3 6`? Or `map sqrt [1..9]`? Well there’s a trick, and it’s called *currying*. What happens when you do `max 3 6` is that you’re actually doing `(max 3) 6`.

We can see this by looking at its declaration:

    max :: (Ord a) => a -> a -> a

Which is the equivalent of:

    max :: (Ord a) => a -> (a -> a)

As we can see in the second declaration, the `max` function takes a single parameter of type a and returns a function that takes another parameter of type a that finally returns a value of the same type. This kind of functions are called *curried functions*.

Now what’s great with this is that it makes specialization really easy:

    λ> let maxThree = max 3
    λ> maxThree 5
    5
    λ> maxThree 2
    3
    λ> let mapDivByTen = map (/10)
    λ> mapDivByTen [1..3]
    [0.1,0.2,0.3]

The second example `mapDivByTen` demonstrates the use of currying with higher order functions:

1. You first create a new function `(/10)` that takes a number and divides it by 10,
2. Then you create another function with `map` that takes a list of numbers and divides them by 10,
3. You now have a new, specialized function, that you can use on any list of numbers!

It could also be written like this:

    λ> let divByTen = (/10)
    λ> let mapDivByTen = map divByTen

So currying is nice because it allows to easily create new functions on the fly (specialization!) with a clean syntax. This is especially useful when dealing with higher order functions. Also, there’s no special case: every function just takes a single argument.

Note that it differs from *partial application*: a curried function always takes a single parameter and always returns a function that takes a single parameter, so what you get is *a chain of unary functions*, whereas with partial application you can apply `n` parameters to your function and you’ll just get *a function that takes `arity-n` parameters*.

In Haskell, *every* function is curried! 🎆
