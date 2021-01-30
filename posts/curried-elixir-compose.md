title = "Curried Elixir: Compose"
tags = ["elixir"]
date = 2021-01-30
%%%

In a [previous post](https://liftm.io/posts/curried-elixir.html), we implemented a `curry` function for Elixir, and in the [following post](https://liftm.io/posts/curried-elixir-flip.html), we built a `flip` function for our curried functions.

I want to build one last thing for our curried function, and that's a composition operator. Elixir doesn't support function composition, you have to use anonymous functions or the [capture operator](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#&/1) instead:

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
    iex> flipped_map = (&Enum.map/2) |> curry() |> flip()
    iex> flipped_get = (&Map.get/2) |> curry() |> flip()
    iex> map_mul_2 = flipped_map.(fn x -> x * 2 end)
    iex> f = map_mul_2 <~> flipped_get
    iex> a = %{x: [1, 2, 3], y: [100, 200, 300]}
    iex> f.(:x).(a)
    [2, 4, 6]
    iex> g = flip(f).(a)
    iex> g.(:x)
    [2, 4, 6]
    iex> g.(:y)
    [200, 400, 600]

With [currying](https://liftm.io/posts/curried-elixir.html), composing and transforming functions becomes a lot easier!

Obviously, Elixir probably isn't the best language for that, as its syntax and the lack of support for curried functions in the language make things look a lot more convoluted than they should.
