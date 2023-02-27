---

theme: uncover
style: |
  .small-text {
    font-size: 0.75rem;
    letter-spacing: 1px;
    font-family: "Times New Roman", Tahoma, Verdana, sans-serif;
  }
  li {
    font-size: 28px;
    letter-spacing: 1px;
  }
  p.quote {
    line-height: 38px;
  }
  q {
    font-size: 32px;
    letter-spacing: 1px;
  }
  cite {
    text-align: right;
    font-size: 28px;
    margin-top: 12px;
    margin-bottom: 128px;
  }
paginate: true
# The footer collides with many of the slides and images
# footer: 'Курс по Elixir 2023, ФМИ'

backgroundColor: #FFFFFF
marp: true


---
## Функционално програмиране с Elixir

---
![Image-Absolute](assets/elixir-logo.png)

---

## Въпроси

* инсталирахте ли си Elixir?
    - препоръчвам инструмента asdf
* отворихте ли elixir-lang.{org, bg}?
* присъединихте ли се във {Facebook, Discord}?
    - не е задължително, но лесно ще си взимате домашното/информацията

---

## Малко преговор

* Elixir се компилира до:
    a) JavaScript
    b) Java
    c) Erlang
    d) Machine Code

---

## Малко преговор

* Elixir & Erlang използват какво планиране(Scheduling)?
* Elixir:
  a) е Статично-типизиран?
  b) е Динамично-типизиран?
  c) имплементира Actor модела?
  d) има строга типизация(strongly typed)?
  e) има слаба типизация(weakly typed)?

---
## Модули и функции

---

### Дефиниране на модул

```elixir
defmodule Times do
  def double(n) do
    n * 2
  end
end
```

---

### Компилиране

* като използваме `elixirc times.ex`
* ще генерира файл на име `Elixir.Times.beam`, който съдържа байткода
* `iex` автоматично ще зареди всякакви `*.beam` файлове, ако са в същата директория
* никога няма да го правим по този начин

---

### Забелязвате ли префикса `Elixir.` ?

---

### Компилиране и зареждане на файл от конзолата

```bash
iex(1)> c "times.ex"
[Times]
iex(2)> Times.double(10)
20
```

---

### .ex vs .exs

* .ex е за файлове, които искаме да се компилират.
* .exs е за файлове, които са скриптове и ще изпълняваме.

---

### elixirc vs elixir

* elixirc само компилира код до .beam
* elixir ще компилира в паметта и изпълни дадения файл, няма да произведе .beam файл
* компилират кода по един и същ начин

---

## Функции

---

### Публични фукнции дефинираме с def/2

* макрос е (ще имаме поне 2 лекции за макроси)
* публичните функции могат да се извикват от други модули
* имената могат да са utf8
* [има конвенции за функции завършващи на `?`/`!`](https://hexdocs.pm/elixir/master/naming-conventions.html)

---

### {Частни, поверителни, За собствено ползване} фукнции дефинираме с defp/2

* пак е макрос
* могат да се извикват само вътре в даден модул
* компилаторът може да се оплаче само за private функции, които не ползваме

---

### Пример

---

```
defmodule FMI do
  def смятай(a, b) do
    смятай_бре(a, b)
  end

  defp смятай_бре(a, b) do
    a + b
  end
end

IO.puts(FMI.смятай(3, 4)) # => 7
IO.puts(FMI.смятай_бре(3, 4))
# => ** (UndefinedFunctionError) function FMI.смятай_бре/2 is undefined or private.
```
---

### Функции с различен брой параметри

```bash
iex(1)> String.split("Elixir is awesome. It totally kicks bum.")
["Elixir", "is", "awesome.", "It", "totally", "kicks", "bum."]

iex(2)> String.split(
  "Elixir is awesome. It totally kicks bum.",
  ".",
  parts: 2)
["Elixir is awesome", " It totally kicks bum."]
```

---

### do..end блокове с малко код

```elixir
# Кратък синтаксис за one-liners
def double(n), do: n * 2
# do..end синтаксис, позволява много редове код
def again_double(n) do
  n * 2
end

---

### Дефиниране на функции и pattern matching

```elixir
defmodule Factorial do
  def of(0), do: 1
  def of(n), do: n * of(n - 1)
end
```

---

### Редът на дефиниране на "главите" на функцията има значение

```elixir
defmodule Factorial do
  def of(n), do: n * of(n - 1)
  def of(0), do: 1
end
```

---

### Guard клаузи при дефиниране на функции

```elixir
defmodule Fibonacci do
  def of(1), do: 1
  def of(2), do: 1
  def of(n) when is_number(n), do: of(n - 1) + of(n - 2)
end
```

---

### Параметри със стойности по подразбиране

* дефинират се с `\\`
* = се използва за съпоставяне (pattern matching), затова е нужен друг синтаксис

```elixir
defmodule Example do
  def func(p1, p2 \\ 2, p3 \\ 3, p4) do
    IO.inspect [p1, p2, p3, p4]
  end
end

Example.func("a", "b") # => ["a", 2, 3, "b"]
Example.func("a", "b", "c") # => ["a", "b", 3, "c"]
Example.func("a", "b", "c", "d") # => ["a", "b", "c", "d"]
```

---

### Pipe оператора - `|>`

---

#### Нека разгледаме един пример без този оператор

---

```elixir
Enum.join(
  String.split(
    String.upcase(
      "name,faculty_number,grade"),
    ","),
  " "
)
"NAME FACULTY_NUMBER GRADE"
```

---

### Pipe оператора - `|>`

```elixir
"name,faculty_number,grade"
|> String.upcase
|> String.split(",")
|> Enum.join(" ")
"NAME FACULTY_NUMBER GRADE"
```

---

### Връзки между модули и влагане

```elixir
defmodule Outer do
  defmodule Inner do
    def inner_func do
      "hello world"
    end
  end

  def outer_func do
    "Greeting from the inner func: #{Outer.Inner.inner_func}"
  end
end

Outer.outer_func # => "Greeting from the inner func: hello world"
Outer.Inner.inner_func # => "hello world"
```

---

### Няма нищо специално за вложените модули

* само синтактична захар
* крайните модули ще са `:"Elixir.Outer"` и `:"Elixir.Outer.Inner"`
* пак трябва да импортираме функциите от бащата модул във вложения модул
* [ето ви документация](https://hexdocs.pm/elixir/Kernel.html#defmodule/2)
* ето ви пример за неща, които **НЕ МОЖЕМ ДА ПРАВИМ**:

---

```elixir
defmodule Deep.Outer do
  def outer, do: :outer

  defmodule Inner do
    # Трябва Deep.Outer.outer/1 да го реферираме директно,
    # ще гръмне
    def inner_outer, do: Outer.outer()

    # Няма да се компилира
    def inner_outer2, do: outer()
  end
end
```

---

##### Между другото нищо не ни пречи директно да направим това:

```elixir
defmodule Deep.Outer.Inner do
  def inner, do: :inner
end
```

---

### Използване на Erlang модули

```elixir
:rand.uniform(100) # => 98
:rand.uniform(100) # => 41
```

---

### import

* Позволяват ни да викаме всички функции на чужд модул без да ги префиксваме с името на модула.
* с други думи не е нужно да използваме пълното име на функцията
* only
* except
* `:macros/:functions`
* може да импортираме в тялото на функция
* не се ползва толкова често

---
```elixir
defmodule CsvUtils do
  import String

  def upcase_space_transform(csv_line) do
    csv_line
    |> upcase
    |> split(",")
    |> Enum.map(&strip/1)
    |> Enum.join(" ")
  end
end
```

---

```elixir
defmodule Foo do
  def hi, do: "Hi from Foo"
end

defmodule Bar do
  def hi, do: "Hi from Bar"
end

defmodule Baz do
  import Foo
  import Bar

  def say_hi, do: hi()
end
# ** (CompileError) iex:16: function hi/0 imported from both Bar and Foo, call is ambiguous
#     (elixir 1.12.3) src/elixir_dispatch.erl:128: :elixir_dispatch.expand_import/6
#     (elixir 1.12.3) src/elixir_dispatch.erl:91: :elixir_dispatch.dispatch_import/5
#     iex:16: (module)
```

---

### alias

* Дава алтернативно име на модул, за да го използваме по-удобно.
* Указваме новото име с `as: <name>` - `alias Some.Other.Module, as: M`
* Ако не укажем `as:`, то частта от името след последната точка се използва
* ерлангски модули
* {синтаксис}

---

```elixir
defmodule Outer.Inner do
  def inner_func do
    "hello world"
  end
end

defmodule Outer do
  alias Outer.Inner, as: Inner

  def outer_func do
    "Greeting from the inner func: #{Inner.inner_func}"
  end
end
```

---

```elixir
defmodule Outer do
  defmodule Foo do
    def hi, do: "Hi from Foo"
  end

  defmodule Bar do
    def hi, do: "Hi from Bar"
  end
end

alias Outer.{Foo, Bar}
Foo.hi()
# => "Hi from Foo"

Bar.hi()
# => "Hi from Bar"
```

---

### require & use

* използват се когато искаме да използваме макроси дефинирани в даден модул
* compile time ще се "разшири" макроса
* повече за require/use когато стигнем до мета-програмирането
* пример:

---

```elixir
iex(1)> Integer.is_odd(3)
** (CompileError) iex:1: you must require Integer before invoking the macro Integer.is_odd/1
    (elixir) src/elixir_dispatch.erl:97: :elixir_dispatch.dispatch_require/6
iex(1)> require Integer
Integer
iex(2)> Integer.is_odd(3)
true
iex(3)>
```

---

### Модулни атрибути

* дефинират се с `@`
* мислете ги за константи
* `@doc` & `@moduledoc`
* имат и други приложения, повече за тях по време на мета-програмирането

---

```elixir
defmodule Greeter do
  @moduledoc """
  A module designed to greet strangers and friends alike!
  """

  @standard_greeting "Hello, stranger!"

  @doc "Greet someone!"
  def greet(nil) do
    IO.puts @standard_greeting
  end

  def greet(name) do
    IO.puts "Hello, #{name}!"
  end
end
```

---

### Анонимни функции

* всъщност може да ви покажем само анонимни функции и това е достатъчно за целия курс
* един лайк във фб, ако разбрахте шегата
* Meddle има два хубави поста по въпроса: [1](https://themeddle.com/posts/y) и [2](https://themeddle.com/posts/functions_all_the_way)
* за разлика от Erlang, анонимните функции в Elixir не са fun.
* дефинират се с `fn -> end`

---

#### Пример

```
iex(7)> fn a, b -> a + b end.(3, 4)
7
```

---

### Забелязахте ли как я извикахме?

---

### С точка!

* Повече информация [тук](https://stackoverflow.com/questions/18011784/why-are-there-two-kinds-of-functions-in-elixir)
* Нещо си нещо си lisp-2 език(Note to author: може и да го обясниш)

---


#### Могат да имат няколко глави като нормалните функции

```elixir
add = fn
  a, b when is_integer(a) and is_integer(b) -> a + b
  a, b when is_list(a) and is_list(b) -> a ++ b
  a, b when is_binary(a) and is_binary(b) -> <<a::binary, b::binary>>
end
res = add.(<<0::size(16)>>, <<1::size(16)>>)
<<0, 0, 0, 1>>
iex(10)> Kernel.bit_size(res)
32
```

---

### Залавяне на функции

* Function capturing
* става с така наречения "capture operator"
* &пълно-име-на-функция/арност
* може да прихващаме импортирани функции
* може да създаваме функции
* не може да създава функции с арност 0
* може да работи с други три оператора: `[]`, `{}` и `%{}`
  * `& [element | &1]`, `& {:ok, &1}`, `& %{name: &1}`

---

#### Пример

```elixir
iex(5)> fun = &is_integer/1
&:erlang.is_integer/1
iex(6)> fun.(0)
true
```

---

#### Oще пример

```elixir
iex(19)> succ = & &1 + 1
#Function<6.127694169/1 in :erl_eval.expr/5>
iex(20)> succ.(1)
2
```
---

#### Oще пример с оператор

```elixir
iex(13)> doubler = &{&1, &1}
#Function<6.127694169/1 in :erl_eval.expr/5>
iex(14)> doubler.(1)
{1, 1}
```

---

## IEx

* Interactive Elixir
* Това е REPL-ът на Elixir
* Има различни хелпъри документирани в `IEx.Helpers`
    - h
    - c
    - v()
    - recompile()
* Autocomplete


---

## Mix

* build инструмент идващ с инсталирането на Elixir
* позволява ни да #{insert} нашия проект
    1. създаваме
    2. изпълняваме
    3. компилираме
    4. тестваме
    5. менажира "зависимостите ни"
    6. всъщност може да го накараме да прави какво си искаме с нашия проект
* абе влиза в кода и си прави каквото си иска

---

### Малък cheat sheet с неща, които може да прави

* mix new
* mix compile
* mix test
* iex -S mix
* mix format
* mix help
  - потърси помощ
* mix help <command>
* iex --erl "-kernel shell_history enabled" -S mix

---

## ExUnit

* testing framework, който идва с Еликсир
* при генериране на микс проект идва с `test_helper.exs` файл
* конвенцията е за всеки файл в `lib/<my_module>.ex` е да дефинираме един
    тест файл в `test/<my_module>_test.exs`
* Използват `.exs` разширението
* За да пишем тестове трябва да използваме `use ExUnit.Case`
* Дефинираме различни тестовете с макрото `test`
* mix test автоматично ще стартира

---

## Small demo time

---

### Въпроси?
