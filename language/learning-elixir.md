# Elixir学习笔记

Functions in Elixir are identified by name and by number of arguments (i.e. arity). Therefore, `is_boolean/1` identifies a function named `is_boolean` that takes 1 argument. `is_boolean/2` identifies a different (nonexistent) function with the same name but different arity.

------

Atoms are constants where their name is their own value. Some other languages call these symbols.

The booleans `true` and `false` are, in fact, atoms.

------

Strings in Elixir are inserted between double quotes, and they are encoded in UTF-8.

Strings in Elixir are represented internally by binaries which are sequences of bytes.

```elixir
byte_size("hellö")
# 6
String.length("hellö")
# 5
```

------

Functions are “first class citizens” in Elixir meaning they can be passed as arguments to other functions just as integers and strings can.

Note a dot (.) between the variable and parenthesis is required to invoke an anonymous function:

```elixir
add = fn a, b -> a + b end
add.(1, 2)
```

Anonymous functions are closures, and as such they can access variables that are in scope when the function is defined.

------

When Elixir sees a list of printable ASCII numbers, Elixir will print that as a char list (literally a list of characters). 

```elixir
[11, 12, 13]
# '\v\f\r'
[104, 101, 108, 108, 111]
# 'hello'
```

Whenever you see a value in IEx and you are not quite sure what it is, you can use the `i/1` to retrieve information about it.

Keep in mind single-quoted and double-quoted representations are not equivalent in Elixir as they are represented by different types.

```elixir
'hello' == "hello"
# false
```

Single-quotes are char lists, double-quotes are strings.

------

Tuples store elements contiguously in memory. This means accessing a tuple element per index or getting the tuple size is a fast operation (indexes start from zero)

------

```elixir
[1, 2, 3] ++ [4, 5, 6]
# [1, 2, 3, 4, 5, 6]
[1, 2, 3] -- [2]
# [1, 3]

"foo" <> "bar"
# "foobar"
```

Elixir provides three boolean operators: `or`, `and` and `not`. These operators are strict in the sense that they expect a boolean (`true` or `false`) as their first argument:

```elixir
true and true
# true
false or is_atom(:example)
# true
1 and true
# ** (ArgumentError) argument error
```

Besides these boolean operators, Elixir also provides `||`, `&&` and `!` which accept arguments of any type. For these operators, all values except `false` and `nil` will evaluate to true:

------

Elixir also provides `==`, `!=`, `===`, `!==`, `<=`, `>=`, `<` and `>` as comparison operators.

The difference between `==` and `===` is that the latter is more strict when comparing integers and floats.

In Elixir, we can compare two different data types. The reason we can compare different data types is pragmatism. Sorting algorithms don’t need to worry about different data types in order to sort. The overall sorting order is defined below:

```
number < atom < reference < functions < port < pid < tuple < maps < list < bitstring
```

------

In Elixir, the `=` operator is actually called the match operator.

The match operator is not only used to match against simple values, but it is also useful for destructuring more complex data types:

```elixir
{a, b, c} = {:hello, "world", 42}
# {:hello, "world", 42}
a
# :hello
b
# "world"
```

A pattern match will error in the case the sides can’t match.

------

Variables in Elixir can be rebound:

```elixir
x = 1
# 1
x = 2
# 2
```

The pin operator `^` should be used when you want to pattern match against an existing variable’s value rather than rebinding the variable:

```elixir
x = 1
# 1
^x = 2
# ** (MatchError) no match of right hand side value: 2
{y, ^x} = {2, 1}
# {2, 1}
y
# 2
{y, ^x} = {2, 2}
# ** (MatchError) no match of right hand side value: {2, 2}
```

In some cases, you don’t care about a particular value in a pattern. It is a common practice to bind those values to the underscore, _. For example, if only the head of the list matters to us, we can assign the tail to underscore:

```elixir
[h | _] = [1, 2, 3]
# [1, 2, 3]
h
# 1
```
The variable `_` is special in that it can never be read from. Trying to read from it gives an unbound variable error.

------

Keep in mind errors in guards do not leak but simply make the guard fail:

```elixir
hd(1)
# ** (ArgumentError) argument error
#    :erlang.hd(1)
case 1 do
    x when hd(x) -> "Won't match"
    x -> "Got: #{x}"
end
# "Got 1"
```

------

`cond` considers any value besides `nil` and `false` to be true.

------

A string is a UTF-8 encoded binary, and a binary is a bitstring where the number of bits is divisible by 8. 

The Unicode standard assigns code points to many of the characters we know. For example, the letter `a` has code point `97` while the letter `ł` has code point `322`. When writing the string `"hełło"` to disk, we need to convert this code point to bytes. If we adopted a rule that said one byte represents one code point, we wouldn’t be able to write `"hełło"`, because it uses the code point `322` for `ł`, and one byte can only represent a number from `0` to `255`. But of course, given you can actually read `"hełło"` on your screen, it must be represented somehow. That’s where encodings come in.

In Elixir, you can define a binary using `<<>>`. A binary is just a sequence of bytes. 

A char list is nothing more than a list of characters:

```elixir
'hełło'
# [104, 101, 322, 322, 111]
is_list 'hełło'
# true
'hello'
# 'hello'
```

instead of containing bytes, a char list contains the code points of the characters between single-quotes. So while double-quotes represent a string (i.e. a binary), single-quotes represents a char list (i.e. a list).

In practice, char lists are used mostly when interfacing with Erlang, in particular old libraries that do not accept binaries as arguments. You can convert a char list to a string and back by using the `to_string/1` and `to_char_list/1` functions.

------

In Elixir, when we have a list of tuples and the first item of the tuple (i.e. the key) is an atom, we call it a keyword list.

```elixir
list = [{:a, 1}, {:b, 2}]
# [a: 1, b: 2]
list == [a: 1, b: 2]
# true
# 注意`a:`和`b:`冒号后边的空格不能少
list[:a]
# 1
```

Keyword lists are important because they have three special characteristics:

- Keys must be atoms.
- Keys are ordered, as specified by the developer.
- Keys can be given more than once.

keyword lists are simply lists, and as such they provide the same linear performance characteristics as lists. The longer the list, the longer it will take to find a key, to count the number of items, and so on.

------

A map is created using the `%{}` syntax:

```elixir
map = %{:a => 1, 2 => :b}
# %{2 => :b, :a => 1}
map[:a]
# 1
map[2]
# :b
map[:c]
# nil
```

- Maps allow any value as a key.
- Maps' keys do not follow any ordering.

------

In order to create our own modules in Elixir, we use the `defmodule` macro. We use the `def` macro to define functions in that module:

```elixir
defmodule Math do
	def sum(a, b) do
		a + b
	end
end

Math.sum(1, 2)
# 3
```

Inside a module, we can define functions with `def/2` and private functions with `defp/2`. A function defined with `def/2` can be invoked from other modules while a private function can only be invoked locally.

Function declarations also support guards and multiple clauses.

```elixir
defmodule Math do
	def zero?(0) do
		true
	end
	
	def zero?(x) when is_integer(x) do
		false
	end
end

IO.puts Math.zero?(0)         #=> true
IO.puts Math.zero?(1)         #=> false
IO.puts Math.zero?([1, 2, 3]) #=> ** (FunctionClauseError)
IO.puts Math.zero?(0.0)       #=> ** (FunctionClauseError)
```

Named functions in Elixir also support default arguments:

```elixir
defmodule Concat do
	def join(a, b, sep \\ " ") do
		a <> sep <> b
	end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
```

Any expression is allowed to serve as a default value, but it won’t be evaluated during the function definition; it will simply be stored for later use. Every time the function is invoked and any of its default values have to be used, the expression for that default value will be evaluated:

```elixir
defmodule DefaultTest do
  def dowork(x \\ IO.puts "hello") do
    x
  end
end
```

------

Mutating is not possible in Elixir, so we can not use loop. Instead, functional languages rely on recursion.

```elixir
defmodule Math do
	def sum_list([head | tail], accumulator) do
		sum_list(tail, head + accumlator)
	end
	
	def sum_list([], accumulator) do
		accumulator
	end
end

IO.pust Math.sum_list([1, 2, 3], 0) #=> 6 
```

```elixir
defmodule Math do
	def double_each([head | tail]) do
		[head * 2 | double_each(tail)]
	end
	
	def double_each([]) do
		[]
	end
end
```

The `Enum` module provides a huge range of functions to transform, sort, group, filter and retrieve iterms from enumerables.

As an alternative to `Enum`, Elixir provides the Stream module which supports lazy operations.

------

The basic mechanism for spawning new processes is with the auto-imported `spawn/1` function:

```elixir
spawn fn -> 1 + 2 end
```

`spawn/1` takes a function which it will execute in another process.

```elixir
parent = self()

spawn fn -> send(parent, {:hello, self()}) end

receive do
	{:hello, pid} -> "Got hello from #{inspect pid}"
end
```

While other languages would require us to catch/handle exceptions, in Elixir we are actually fine with letting processes fail because we expect supervisors to properly restart our systems. "Failing fast" is a common philosophy when writing Elixir software!

------

Although not a directive, `use` is a macro tightly related to `require` that allows you to use a module in the current context.

`use` requires the given module and then calls the `__using__/1` callback on it allowing the module to inject some code into the current context.

An alias in Elixir is a capitalized identifier (like `String`, `Keyword`, etc) which is converted to an atom during compilation. For instance, the `String` alias translates by default to the atom `:"Elixir.String"`.

------

Module attributes in Elixir serve three purposes:

1. They serve to annotate the module, often with information to be used by the user or the VM.
2. They work as constants.
3. They work as a temporary module storage to be used during compilation.

```elixir
defmodule Math do
	@moduledoc """
	Provides math-related functions.
	
	## Examples
	
		iex> Math.sum(1, 2)
		3
	"""
	
	@doc """
	Calculates the sum of two numbers.
	"""
	def sum(a, b), do: a + b
end
```

------

Structs are extensions built on top of maps that provide compile-time checks and default values.

Structs take the name of the module they’re defined in.

