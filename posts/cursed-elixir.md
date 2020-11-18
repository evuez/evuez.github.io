title = "Cursed Elixir"
date = 2020-08-12
tags = ["elixir"]
%%%

Let's write some Elixir.

    defmodule FooBar do
      def foo(a) do
        if a < 0 do
          bar(a, -1)
        else
          bar(a, 1)
        end
      end

      defp bar(a, b) do
        IO.inspect(a * b)
      end
    end

Not very useful, but that's good enough for our purpose.

I like Elixir, but I think most of the time it just looks like functional Ruby. I want to make this code look like Elixir.

First, this code is lacking every Elixir developer's best friend: `|>`. Let's add some `|>`s.

    defmodule FooBar do
      def foo(a) do
        if a < 0 do
          a |> bar(-1)
        else
          a |> bar(1)
        end
      end

      defp bar(a, b) do
        (a * b) |> IO.inspect()
      end
    end

Meh. It's definitely better, but I mean, that's only 3 `|>`s. I want more `|>`s.

We have an `if` in there, so maybe we can do something with it?

    defmodule FooBar do
      def foo(a) do
        (a < 0) |> if do
          a |> bar(-1)
        else
          a |> bar(1)
        end
      end

      defp bar(a, b) do
        (a * b) |> IO.inspect()
      end
    end

We sure can! That's one more `|>`. Can we do better than this?

Well... `>` and `*` are `Kernel` functions, so maybe...

    defmodule FooBar do
      def foo(a) do
        a |> Kernel.<(0) |> if do
          a |> bar(-1)
        else
          a |> bar(1)
        end
      end

      defp bar(a, b) do
        a |> Kernel.*(b) |> IO.inspect()
      end
    end

This is great, can we keep going?

The Elixir docs say `defmodule` is just a macro. Does that mean I can just `|>` into `defmodule`?

    FooBar |> defmodule do
      def foo(a) do
        a |> Kernel.<(0) |> if do
          a |> bar(-1)
        else
          a |> bar(1)
        end
      end

      defp bar(a, b) do
        a |> Kernel.*(b) |> IO.inspect()
      end
    end

Yes you can!

`def` and `defp` are macros too right?

    FooBar |> defmodule do
      a |> foo() |> def do
        a |> Kernel.<(0) |> if do
          a |> bar(-1)
        else
          a |> bar(1)
        end
      end

      a |> bar(b) |> defp do
        a |> Kernel.*(b) |> IO.inspect()
      end
    end

So many pipes! 😍

We're getting somewhere, but something still doesn't feel right. This module really isn't doing much, so maybe it should not be that long? Also, I think we need more `:`. Atoms are very Elixir-y, so let's do more of that:

    FooBar |> defmodule(do: (
      a |> foo() |> def(do: a |> Kernel.<(0) |> if(do: a |> bar(-1), else: a |> bar(1)))

      a |> bar(b) |> defp(do: a |> Kernel.*(b) |> IO.inspect())
    ))

We're `:do`ing great!

You know what's also very Elixir-y? Lists. Lists and tuples.

    FooBar |> defmodule([{:do, (
      a
      |> foo()
      |> def([{:do, a |> Kernel.<(0) |> if([{:do, a |> bar(-1)}, {:else, a |> bar(1)}])}])

      a |> bar(b) |> defp([{:do, a |> Kernel.*(b) |> IO.inspect()}])
    )}])

Who's going to say this looks like Ruby now? ⚗️

---

[discussion on hackernews](https://news.ycombinator.com/item?id=24818706) /
[discussion on reddit](https://www.reddit.com/r/elixir/comments/jd2hr4/cursed_elixir/)
