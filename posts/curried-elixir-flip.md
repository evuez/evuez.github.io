title = "Curried Elixir: Flip"
tags = ["elixir"]
date = 2021-01-29
%%%

In the [previous post](https://liftm.io/posts/curried-elixir.html), we implemented a `curry` function for Elixir:

    iex> f = curry(&Enum.map/2)
    #Function<...>
    iex> f.(1..3).(&"i = #{&1}")
    ["i = 1", "i = 2", "i = 3"]

Currying on its own is nice, but it also gives us the ability to easily define a `flip` function that would let us flip the order of the first two arguments of a curried function:

    iex> f = curry(&Enum.map/2)
    #Function<...>
    iex> flipped = flip(f)
    #Function<...>
    iex> f.(&"i = #{&1}").(1..3)
    ["i = 1", "i = 2", "i = 3"]

The only thing we have to do is wrap the given curried function in a chain of two anonymous functions, and reverse the order of the arguments:

    defmodule Func do
      def flip(fun) when is_function(fun) do
        fn b ->
          fn a -> fun.(a).(b) end
        end
      end
    end

And this is how you could use it:

    iex> curried_add = curry(&Kernel.+/2)
    iex> curried_mul = curry(&Kernel.*/2)
    iex> curried_map = curry(&Enum.map/2)
    iex> flipped_map = flip(curried_map)
    iex> map_3 = curried_map.(1..3)
    iex> f = flipped_map.(map_3)
    iex> f.([curried_add.(2), curried_mul.(3)])
    [[3, 4, 5], [3, 6, 9]]
    iex> f.([curried_add.(1), curried_mul.(4)])
    [[2, 3, 4], [4, 8, 12]]

It would be a lot more useful if the standard library functions were able to handle our curried functions (for example, [`Enum.reduce/3`](https://hexdocs.pm/elixir/Enum.html#reduce/3) expects a 2-arity function as its third argument, not a curried 2-arity function), but I never said we were building something useful 🤷
