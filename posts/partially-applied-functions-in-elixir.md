title = "Partially applied functions in Elixir"
date = 2017-12-13
%%%
Thanks to the [capture operator](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#&/1) it’s easy to create partially applied functions in Elixir:

```
iex> sum = fn (a, b) -> a + b end
iex> sum.(1, 2)
3
iex> add42 = &sum.(&1, 42)
iex> add42.(3)
45
iex> greet = &IO.puts("Hello " <> &1 <> "!")
iex> greet.("Marvin")
Hello Marvin!
:ok
```
