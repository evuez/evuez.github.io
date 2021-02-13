title = "Cursed curried Elixir"
tags = ["elixir"]
date = 2021-02-13
%%%

In [Curried Elixir](https://liftm.io/posts/curried-elixir.html), I said this:

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

Just to get started, let's see if we can write a function that generates this AST for us.
We want this function to take as arguments the length of the chain and the body of the innermost function in the chain:

    def make_chain(length_, body) do
      Enum.reduce(1..length_, body, fn x, acc ->
        {:fn, [], [{:->, [], [[{:"arg#{x}", [], Elixir}], acc]}]}
      end)
    end

Or better, use [`Macro.generate_unique_arguments/2`](https://hexdocs.pm/elixir/Macro.html#generate_unique_arguments/2) to generate the list of parameters for us:

    defmodule Func do
      def make_chain(length_, body) do
        length_
        |> Macro.generate_unique_arguments(Elixir)
        |> Enum.reverse() # Without this, the outermost function would get the last argument, which could make things confusing
        |> Enum.reduce(body, fn arg, acc ->
          {:fn, [], [{:->, [], [[arg], acc]}]}
        end)
      end
    end

We can use [`Macro.to_string/1`](https://hexdocs.pm/elixir/Macro.html#to_string/2) to check if this works as expected:

    iex> Macro.to_string(make_chain(2, "Foo"))
    "fn arg1 -> fn arg2 -> \"Foo\" end end"

Perfect! 🎉

We still have a few things to do before we start rewriting our `curry` function. First, we need to find a way to generate the AST for the function call:

    iex> make_call(&Map.get/2)
    {{:., [], [{:__aliases__, [alias: false], [:Map]}, :get]}, [],
     [{:arg1, [], Elixir}, {:arg2, [], Elixir}]}


This is actually pretty easy: [`Function.info/1`](https://hexdocs.pm/elixir/master/Function.html#info/1) and [`Macro.generate_unique_arguments/2`](https://hexdocs.pm/elixir/Macro.html#generate_unique_arguments/2) can do most of the work for us:

    def make_call(fun) do
      info = Function.info(fun)

      arity = Keyword.fetch!(info, :arity)
      mod = Keyword.fetch!(info, :module)
      name = Keyword.fetch!(info, :name)

      {{:., [], [{:__aliases__, [alias: false], [mod]}, name]}, [],
       Macro.generate_unique_arguments(arity, Elixir)}
    end

With [`Macro.to_string/1`](https://hexdocs.pm/elixir/Macro.html#to_string/2):

    iex> Macro.to_string(make_call(&Map.get/2))
    "fun.(arg1, arg2)"

So we can now generate the AST for a chain of unary functions, and the AST for an anonymous function call. We need to do something with all these tuples now!

If you're scared of [`defmacro/2`](https://hexdocs.pm/elixir/Kernel.html#defmacro/2) and [`unquote/1`](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#unquote/1), don't worry. We don't need any of that. Everyone knows macros are scary. You know what's not scary? Runtime evaluation!

With [`Code.eval_quoted/2`](https://hexdocs.pm/elixir/Code.html#eval_quoted/3), we should finally be able to write our `curry` function:

    defmodule Func do
      def curry(fun) do
        info = Function.info(fun)
        params = Macro.generate_unique_arguments(info[:arity], Elixir)
        body_ast = body_ast(info[:module], info[:name], params)

        curried_ast =
          params
          |> Enum.reverse()
          |> Enum.reduce(body_ast, fn arg, acc ->
            {:fn, [], [{:->, [], [[arg], acc]}]}
          end)

        {curried_fun, _} = Code.eval_quoted(curried_ast)
        curried_fun
      end

      defp body_ast(mod, name, params) do
        {{:., [], [mod_ast(mod), name]}, [], params}
      end

      defp mod_ast(mod) do
        # Module names like `:erlang` or `:code` need a different AST
        case to_string(mod) do
          "Elixir." <> _ -> {:__aliases__, [alias: false], [mod]}
          _ -> mod
        end
      end
    end

Does it work?

    iex> curried_length = Func.curry(&length/1)
    #Function<...>
    iex> curried_length.([1, 2, 3])
    3
    iex> Func.curry(&Map.get/2).(%{foo: "bar"}).(:foo)
    "bar"

It does! 🎉
Who needs macros??
