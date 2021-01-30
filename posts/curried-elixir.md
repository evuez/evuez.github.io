title = "Curried Elixir"
tags = ["elixir"]
date = 2021-01-28
%%%

Elixir already supports [partially applied functions](https://liftm.io/posts/partially-applied-functions-in-elixir.html) thanks to the [capture operator](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#&/1), and partial application is nice. But you know what's even better? [Currying](https://liftm.io/posts/currying.html)!

Let's say we define this function:

    def my_fun(a, b, c) do
      a + b + c
    end

What we would want is to have a `curry/1` function that would allow us transforming this function into a curried function:

    iex> curried_fun = curry(&my_fun/3)
    iex> curried_fun.(1).(2).(3)
    6
    iex> add_42 = curried_fun.(40).(2)
    iex> add_42.(10)
    52

We're going to do this by wrapping `my_fun` in a chain of anonymous functions, each binding exactly one name:

    curried_fun =
      fn (a) ->
        fn (b) ->
          fn (c) ->
            my_fun.(a, b, c)
          end
        end
      end

The most (and only?) important thing we need to know in order to implement this `curry` function is the arity of the function we are given. We can use [`:erlang.fun_info/2`](https://erlang.org/doc/man/erlang.html#fun_info-2) for this:

    iex> :erlang.fun_info(fn x, y -> x * y end, :arity)
    {:arity, 2}

Now that we have this, let's write this `curry` function!

    defmodule Func do
      def curry(fun) when is_function(fun) do
        {:arity, arity} = :erlang.fun_info(fun, :arity)
        curry(fun, arity, [])
      end

      def curry(fun, 0, []), do: fn -> fun.() end
      def curry(fun, 1, args), do: fn x -> apply(fun, Enum.reverse([x | args])) end
      def curry(fun, arity, args), do: fn x -> curry(fun, arity - 1, [x | args]) end
    end

And that's it! Here's what happens when we're using `curry`:

    iex> f = curry(&Enum.map/2)
    #Function<...> # fn x -> curry(&Enum.map/2, 1, [x]) end
    iex> f.(1..3)
    #Function<...> # fn x -> apply(&Enum.map/2, Enum.reverse([x, 1..3])) end
    iex> f.(&"i = #{&1}")
    ["i = 1", "i = 2", "i = 3"] # apply(&Enum.map/2, [1..3, &"i = #{&1}"])

0-arity functions are a special case, as we don't want `curry` to apply them for us, i.e.:

    iex> curry(&self/0)
    #PID<...> # apply(&self/0, [])

Instead, we wrap the function in a 0-arity anonymous function:

    iex> f = curry(&self/0)
    #Function<...> # fn -> apply(&self/0, []) end
    iex> f.()
    #PID<...> # apply(&self/0, [])
