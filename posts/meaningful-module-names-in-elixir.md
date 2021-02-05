title = "Meaningful module names in Elixir"
date = 2021-02-05
tags = ["elixir"]
%%%

Just a reminder that you don't need to restrict yourself to Pascal case for module names.

If you like consistency and think module names should look like function names, don't think twice!

    defmodule :snake_case, do: nil

If you like symbols, you have [a lot of options](https://hexdocs.pm/elixir/operators.html)!

    defmodule :::, do: nil
    defmodule :.., do: nil
    defmodule :@, do: nil
    defmodule :<, do: nil

Or even

    defmodule :., do: nil

And don't forget about [custom operators](https://hexdocs.pm/elixir/operators.html#custom-and-overridden-operators)! You can get really fancy here:

    defmodule :. do
      def a +++ b, do: a + b
    end

    iex> :".".+++ 1, 2
    3

Or maybe

    defmodule :>>> do
      def a >>> b, do: "brrr"
    end

    iex> :>>>.>>> 1, 2
    "brrr"

Finally, don't:

    defmodule APerfectlyFineModule do
      @moduledoc "Here be dragons"
    end

This is just too easy to miss. Instead, do:

    defmodule :"Here be dragons", do: nil
