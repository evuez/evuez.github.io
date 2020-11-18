title = "Postgres JSON(B) helpers for Ecto"
date = 2020-11-18
tags = ["elixir", "postgres"]
%%%

If you've ever worked with JSON columns in Postgres with Ecto, you probably wrote a lot of [`fragment`](https://hexdocs.pm/ecto/Ecto.Query.API.html#fragment/1)s similar to this one:

    fragment("?->>'field'", x.column)

to access fields in a JSON object.

This is nice, but what if we could write `x.column->>field` directly instead?

Well we can! We just have to define a custom operator for this. Elixir has a list of [overridable operators](https://hexdocs.pm/elixir/operators.html#defining-custom-operators), and even though there's no `->>`, there's `~>>` which looks similar enough.

    defmodule JsonAccessor do
      defmacro left ~>> right do
        quote do: fragment("?->>?", unquote(left), unquote(right))
      end
    end

You can then `import JsonAccessor` and start using this new operator:

    Repo.all(from u in User, where: u.details~>>"email" == "user@example.com")

This works but... it's far from perfect:

    x.column ~>> "field" # Works
    x.column ~>> field # Raises "(Ecto.Query.CompileError) unbound variable `field` in query."

We also can't query nested fields without the [`->`](https://www.postgresql.org/docs/9.5/functions-json.html) operator:

    x.column ~>> "field" ~>> "nested" # Won't work because `->>` returns text instead of a JSON object

We can fix the first issue by converting the right-hand side to a string when we're given an atom:

    defmodule JsonAccessor do
      defmacro left ~>> right do
        quote do: fragment("?->>?", unquote(left), unquote(rhs(right)))
      end

      defp rhs({field, _ctx, nil}) when is_atom(field), do: to_string(field)
      defp rhs(field) when is_binary(field), do: field
    end

Now `x.column ~>> field` works as expected. For the nested access, we just need to add a new operator for `->`:

    defmodule JsonAccessor do
      defmacro left ~>> right do
        quote do: fragment("?->>?", unquote(left), unquote(rhs(right)))
      end

      defmacro left ~> right do
        quote do: fragment("?->?", unquote(left), unquote(rhs(right)))
      end

      defp rhs({field, _ctx, nil}) when is_atom(field), do: to_string(field)
      defp rhs(field) when is_binary(field), do: field
    end

And we can now write `x.column~>field~>>nested == "value"`, instead of `fragment("?->'field'->>'nested' = ?", x.column, "value")` 🎉
