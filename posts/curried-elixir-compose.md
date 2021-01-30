title = "Curried Elixir: Compose"
tags = ["elixir"]
date = 2021-01-30
%%%

In a [previous post](https://liftm.io/posts/curried-elixir.html), we implemented a `curry` function for Elixir, and in the [following post](https://liftm.io/posts/curried-elixir.html), we built a `flip` function for our curried functions.

I want to build one last thing for our curried function, and that's a composition operator. Elixir doesn't support function composition, you have to use anonymous functions or the [capture operator](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#&/1) if you want to compose functions:

    iex> add_2 = fn x -> x + 2 end
    iex> mul_3 = fn x -> x * 3 end
    iex> add_2_and_mul_3 = fn x -> mul_3.(add_2.(x)) end # or &mul_3.(add_2.(&1))
    iex> add_2_and_mul_3.(5)
    21

  It's... ok I guess? But I'd really prefer if I could just do something like `add_2_and_mul_3 = mul_3 . add_2`.
  `.` isn't overridable, but we can use any of the [custom operators](https://hexdocs.pm/elixir/master/operators.html#defining-custom-operators). I'm going to pick `<~>`.

    defmodule Func do
      def f <~> g when is_function(g), do: fn x -> f <~> g.(x) end # Reduce the right-hand function first
      def f <~> x, do: f.(x) # When we're done reducing the right-hand side, reduce the left-hand side
    end

Remember, we expect `f` and `g` to be curried here, which is why we don't have to worry about their arity: it's always `f/1` and `g/1`.

Let's try this!

    iex> add_2 = fn x -> x + 2 end
    iex> mul_3 = fn x -> x * 3 end
    iex> (mul_3 <~> add_2).(5)
    21
