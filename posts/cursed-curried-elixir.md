title = "Cursed curried Elixir"
tags = ["elixir"]
date = 2021-02-13
%%%

In [Curried Elixir](https://evuez.net/posts/curried-elixir.html), I said this:

> We're going to do this by wrapping `my_fun` in a chain of anonymous functions, each binding exactly one name:
>
>     curried_fun =
>       fn (a) ->
>         fn (b) ->
>           fn (c) ->
>             my_fun.(a, b, c)
>           end
>         end
>       end

And then went on implementing `curry` as a recursive function. But what if I actually want my curried function to be a chain of anonymous functions, and get rid of this recursive call?

The obvious thing to do would be to try to implement this as a macro instead. But I think we can do better than this.

First, let's look at the [AST](https://hexdocs.pm/elixir/syntax-reference.html#the-elixir-ast) for the `curried_fun` above:

    iex> quote do
    ...>   fn (a) ->
    ...>     fn (b) ->
    ...>       fn (c) ->
    ...>         my_fun.(a, b, c)
    ...>       end
    ...>     end
    ...>   end
    ...> end
    #                ▼ metadata                ▼ nested anonymous function
    {:fn, [], [{:->, [], [[{:a, [], Elixir}], {:fn, [], [{:->, ...}]}]}]}
    #                      ▲ list of parameters

This might look a bit messy, but there really isn't much going on:

    {:fn, [], # The definition with an empty list of metadata
     [
       {:->, [],
        [
          [{:a, [], Elixir}], # Left-hand side of `->`, the head of the anonymous function
          {:fn , [], # Right-hand side of `->`, the body of the anonymous function
           ...


## Building the AST

Just to get started, let's see if we can write a function that generates this AST for us.
We want this function to take as arguments the length of the chain and the body of the innermost function in the chain:

    def make_chain(length_, body) do
      Enum.reduce(1..length_, body, fn x, acc ->
        {:fn, [], [{:->, [], [[{:"arg#{x}", [], nil}], acc]}]}
      end)
    end

Or better, use [`Macro.generate_unique_arguments/2`](https://hexdocs.pm/elixir/Macro.html#generate_unique_arguments/2) to generate the list of parameters for us:

    defmodule Func do
      def make_chain(length_, body) do
        length_
        |> Macro.generate_unique_arguments(nil)
        |> Enum.reverse() # Without this, the outermost function would get the last argument, which could make things confusing
        |> Enum.reduce(body, fn arg, acc ->
          {:fn, [], [{:->, [], [[arg], acc]}]}
        end)
      end
    end

We can use [`Macro.to_string/1`](https://hexdocs.pm/elixir/Macro.html#to_string/2) to check if this works as expected:

    iex> Macro.to_string(make_chain(2, "Foo"))
    "fn arg1 -> fn arg2 -> \"Foo\" end end"

Perfect!

We have one thing left to do before we can start rewriting `curry`: we need to generate the AST to call `fun`. This is actually pretty easy:

    iex> quote do: fun.(a, b)
    {{:., [], [{:fun, [], Elixir}]}, [], [{:a, [], Elixir}, {:b, [], Elixir}]}

All we need is [`Macro.generate_unique_arguments/2`](https://hexdocs.pm/elixir/Macro.html#generate_unique_arguments/2) and [`Function.info/2`](https://hexdocs.pm/elixir/master/Function.html#info/2) to get the arity of `fun`:

    iex> fun = &Map.get/2
    iex> {:arity, arity} = Function.info(fun, :arity)
    iex> params = Macro.generate_unique_arguments(arity, nil)
    iex> Macro.to_string({{:., [], [{:fun, [], nil}]}, [], params})
    "fun.(arg1, arg2)"

## Evaluating the AST

We now know how to generate the AST for our curried function. For the next step, we need to figure out how to use it!

If you're scared of [`defmacro/2`](https://hexdocs.pm/elixir/Kernel.html#defmacro/2) and [`unquote/1`](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#unquote/1), don't worry. We don't need any of that. Everyone knows macros are scary.

You know what's not scary? *Runtime evaluation!*

With [`Code.eval_quoted/2`](https://hexdocs.pm/elixir/Code.html#eval_quoted/3), we should finally be able to write our `curry` function:

    defmodule Func do
      def curry(fun) do
        {:arity, arity} = Function.info(fun, :arity)
        params = Macro.generate_unique_arguments(arity, nil)
        body_ast = {{:., [], [{:fun, [], nil}]}, [], params}

        {curried_fun, _} =
          params
          |> Enum.reverse()
          |> Enum.reduce(body_ast, fn arg, acc ->
            {:fn, [], [{:->, [], [[arg], acc]}]}
          end)
          |> Code.eval_quoted(fun: fun)

        curried_fun
      end
    end

Does it work?

    iex> curried_sum = Func.curry(fn x, y -> x + y end)
    #Function<...>
    iex> curried_sum.(1).(2)
    3
    iex> Func.curry(&Map.get/2).(%{foo: "bar"}).(:foo)
    "bar"

It does!
Who needs macros??
