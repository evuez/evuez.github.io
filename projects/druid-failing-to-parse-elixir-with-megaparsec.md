title = "druid: Failing to parse Elixir with Megaparsec"
%%%

[Repo](https://github.com/evuez/druid/)

Last year I spent some time trying to parse Elixir with [Megaparsec](https://github.com/mrkkrp/megaparsec).

One of my colleague had started writing an LR(1) parser for Elixir in Rust, and I thought it'd be fun to write one with a parser combinator so we could compare our implementations.

We never got to that point. Turns out, the Elixir syntax is not as simple as you might think.

To be clear, my failure to parse Elixir has nothing to do with Megaparsec: I'm sure anyone with a decent knowledge of it would have no problem solving the many issues I faced. But having spent many hours trying to solve these issues, I now have a greater appreciation for all the effort that has been put into making the Elixir syntax so easy to pick up.

So this is a list of syntax quirks I learned about while writing this parser. Some of these might be obvious, but hopefully you'll still learn a thing or two while reading this.

## The [access syntax](https://hexdocs.pm/elixir/Access.html)

I've never been a huge fan of the `data[key]` syntax: I feel like it doesn't make it clear that `data[unknown_key][key]` won't throw an error and will just return `nil` instead. I can't say spending too many hours trying to parse it properly changed my mind on this syntax.

I'm not sure exactly what made it so complex to parse, but here a few examples that I had a hard time with:


  - `data[:key]`: this is the proper access syntax, it's the same as `Access.get(data, :key)`.
  - `data [:key]`: this is a function call, it's the same as `data([:key])`.
  - `1 [:key]`: What do you think? Syntax error? Wrong! This is a valid access syntax, it's the same as `Access.get(1, :key)`.
  - `data [:a] [:b]`: That's valid too. It's equivalent to `data([:a][:b])`.
  - `data[:a, :b]` or `data[]`: Now these are syntax errors.
  - `data[\n:a\n]`: Access syntax, newlines are allowed inside `[]`.
  - `data\n[:a]`: This on the other hand, is _obviously_ just a block of code with two separate statements (I know it really does seem obvious if you're used to writing Elixir. It took me a while to make it obvious to my parser too).

Overall these examples don't look too bad, but the fact that the access syntax and lists both use `[]` and that you don't need parens for function application made this a lot harder to parse that I thought it would be.

If you don't know what came before a `[` when you reach it, you can't decide whether it's a list, a syntax error or the access syntax.

## Function application: parens or spaces?

Being able to write `def foo(:bar) do...` instead of `def(foo(:bar)) do...` is pretty nice, but making sense of this syntax to try to parse it really isn't.

For example, `foo (a)` is valid but `foo (a, b)` isn't.

It makes it kind of hard to know where you can ignore spaces and where you can't. Maybe there's a rule for that but I haven't figured it out yet.

## Line breaks and semicolons

Remember when I said `data\n[:a]` is just a block of code with to separate statements? Well that's true, but it isn't _always_ true. It depends on what's at the end of your first line or at the start of your second line.

```
a
= 1
```
is equivalent to
```
a =
1
```
which is the same as `a = 1`.

```
a
* 1
```
is the same as `a * 1`, but

```
a +
1
```
is different from

```
a
+ 1
```
because the unary `+` as higher precedence than the binary `+`. This last example is the same as `a; +1`.

Long pipelines look nice when you can split them over multiple lines, and I don't miss having to add a semicolon to the end of every statement. But it's definitely not something that comes for free in a parser.

## Structs and maps updates with `|`

`|` is defined as a [custom, overridable binary operator](https://hexdocs.pm/elixir/master/operators.html#defining-custom-operators), but it's also used as a special operator in the update syntax for [maps and structs](https://hexdocs.pm/elixir/Map.html).

This makes parsing this operator a little more complex, since you have to carry around some context to know if you're currently in a map or a struct and parse it accordingly.
It's not too bad, but given my Megaparsec skills are not great, I ended up creating a [separate parser](https://github.com/evuez/druid/blob/18055b60ecd679439f415ca7f412fac02e02e01b/src/Lib.hs#L550) just for this.

## [Keywords lists](https://hexdocs.pm/elixir/syntax-reference.html#keywords)

This is a keywords list: `[a: 1]`. This is a keywords list in a function call: `foo(a: 1)`. And another one: `foo([a: 1])`. This is a keywords list too: `[{:a, 1}, b: 2]`.

This is a syntax error: `[a: 1, {:b, 2}]`.

This isn't a keyword list: `[{"a", 1}]`. Nor this is: `[{:a, 1}, {"b", 2}]` (the first element of every tuple must be an atom).

These are both valid access syntax: `data[a: 1]` and `data[{:a, 1}]`. Do you think they are equivalent? Well, let's ask `quote`:

```
iex> quote do: data[a: 1]
{{:., [], [Access, :get]}, [], [{:data, [], Elixir}, [a: 1]]}

iex> quote do: data[{:a, 1}]
{{:., [], [Access, :get]}, [], [{:data, [], Elixir}, {:a, 1}]}
```

So `data[{:a, 1}]` is probably what you'd expect: it's the same as `Access.get(data, {:a, 1})`.

On the other hand, `data[a: 1]` might not be what you expected: it's equivalent to `Access.get(data, [a: 1])`, or `Access.get(data, [{:a, 1}])`.

Hopefully I don't need more examples for this one: parsing keywords lists is **hard**.

## And more...

There are many other things you have to be careful about when parsing Elixir. You can check out [the tests](https://github.com/evuez/druid/blob/2baec508b0cba8e4371ccc054c6b91f864d9d2ba/test/Spec.hs) I wrote for my incomplete parser for a more extensive list.

The official [Elixir parser](https://github.com/elixir-lang/elixir/blob/master/lib/elixir/src/elixir_parser.yrl), the [syntax reference](https://hexdocs.pm/elixir/syntax-reference.html) and [operators page](https://hexdocs.pm/elixir/operators.html#content) are all very good references if you want to learn more about the Elixir syntax.

Even though I definitely failed at writing a complete parser, I have learned a lot about Elixir and definitely improved my Megaparsec skills! This was a great exercise, and if you're feeling adventurous and want to learn more about Elixir or programming languages parsing in general, I'd definitely encourage you to try it.
