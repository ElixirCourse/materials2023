---
theme: uncover
style: |
  .small-text {
    font-size: 0.75rem;
  }
  code.language-elixir {
    background: #000;
    color: #f8f8f8;
  }
  section {
    letter-spacing: 1px !important;
  }
  span.hljs-comment,
  span.hljs-quote,
  span.hljs-meta {
    color: #7c7c7c;
  }
  .hljs-keyword,
  .hljs-selector-tag,
  .hljs-tag,
  .hljs-name {
    color: #f96a6a;
  }
  .hljs-attribute,
  .hljs-selector-id {
    color: #ffffb6;
  }
  .hljs-string,
  .hljs-selector-attr,
  .hljs-selector-pseudo,
  .hljs-addition {
    color: #a8ff60;
  }
  .hljs-subst {
    color: #daefa3;
  }
  .hljs-regexp,
  .hljs-link {
    color: #e9c062;
  }
  .hljs-title,
  .hljs-section,
  .hljs-type,
  .hljs-doctag {
    color: #f0f05b;
  }
  .hljs-symbol,
  .hljs-bullet,
  .hljs-variable,
  .hljs-template-variable,
  .hljs-literal {
    color: #ffc57a;
  }
  .hljs-number,
  .hljs-deletion {
    color:#ff73fd;
  }
  .hljs-emphasis {
    font-style: italic;
  }
  .hljs-strong {
    font-weight: bold;
  }
  li {
    font-size: 32px;
  }
paginate: true
footer: 'Курс по Elixir 2023, ФМИ'
backgroundImage: "linear-gradient(to bottom, #E0EAFC, #CFDEF3)"
marp: true

---

## Структури и протоколи
![Image-Absolute](assets/title.jpg)

---

## Речници (преговор+)

* Как се създава речник (`Map`)?
* Какви могат да бъдат ключовете в един речник?
* Как досъпваме елемент на речник?
* Как променяме нещо в даден речник?
* Как можем да създадем речник подобен на вече съществуващ, но:
  * с добавен ключ
  * с премахнат ключ
  * с променена стойност за даден ключ
* Как проверяваме дали нещо е речник?
* Как намираме размера на даден речник?

---

## Структури
![Image-Absolute](assets/structures.jpg)

---

Една структура всъщност много прилича на `Map`, с няколко разлики:
* Ключовете ѝ са атоми.
* Задължително атоми!
* Ключовете ѝ са предварително дефинирани.

---

Дефиниране на структура:
* Структурите се дефинират в модул използвайки макроса `defstruct`.
* Структурата взема името на модула в който е дефинирана.
* Може да дaдадем стойности по-подразбиране на някoe поле на структурата.
* Може да направим някое поле задължително с модул атрибута `@enforce_keys`.

---

#### Пример

```elixir
defmodule Person do
  @enforce_keys [:name]
  defstruct [:name, :location, children: []]
end
```

---

#### Създаването на "инстанция" на структура, също е подобно на създаването на `Map`

---

Разликите са:
- Траява да дадем името на структурата

```elixir
pesho = %Person{
  name: "Пешо",
  location: "Некаде",
  children: "Нема"
}
```

---

Разликите са:
- Ако пропуснем да дадем стойност на даден дефиниран ключ, той ще получи стойността си по подразбиране
- Ако няма дефинирана стойност по подразбиране, то стойността му ще е `nil`

```elixir
pesho = %Person{ name: "Пешо" }
```

---

Разликите са:
- Ако не дадем стойност на ключ, който е обявен за задължителен получаваме грешка

```elixir
%Person{ children: "Пет или шес'" }
```
---

Разликите са:
- Не можем да даваме ключове, които не са дефинирани в структурата

```elixir
%Person{name: "Пешо", drinks: "ВодKа"}
```

---

Нека пробваме няколко неща

```elixir
pesho = %Person{
  name: "Пешо",
  children: "Нема",
  location: "НикАде"
}

IO.inspect(pesho, label: "Pesho is")
IO.inspect(is_map(pesho), label: "Pesho is Map")
IO.inspect(map_size(pesho), label: "Pesho size is")
```
---

Структурите всъщност са `Map`-ове със специалния ключ `__struct__`

```elixir
IO.inspect(pesho, structs: false, label: "Pesho actually is")
```

---

Хубава новина е, че `Map` модулът работи със структури.

```elixir
Map.put(pesho, :name, "Стойчо")
```

---

Операторът за обновяване също работи.

```elixir
%{ pesho | children: "Жената ги брои" }
```

---

Достъпът до елементите на структура със `.` също работи:

```elixir
pesho.name
```

----

Достъп до елементите на структура с `[]` НЕ раобти:

```elixir
pesho[:name]
```

---

### Можем да съпоставяме структури с речници и други структури

---

#### Но първо, бързо припомняне.

```elixir
%{ age: x } = %{ name: "Гошо", age: 25 }
x
```

---

#### Нека си припоним и Пешо

```elixir
pesho = %Person{name: "Пешо", children: "Нема", location: "НикАде"}
```

---

#### Мачинг на Map и структура

```elixir
%{children: x} = pesho
x
```

---

#### Съпоставяне на 2 структури

```elixir
%Person{location: x} = pesho
x
```
---

#### Съпоставяне на структура и Map

```elixir
%Person{name: x} = %{name: "Гошо"}
x
```

---

#### Съпоставяне на структура и Map

```elixir
%Person{name: x} = pesho
```

```elixir
%{__struct__: Person, name: x} = pesho
```

Горните две два записа са еквивалентни.

* Това е дборе защото, така можем да проверяваме (асъртваме) дали нещо е "инстанция" на дадена структура
* Още по-добре компилатора може да асъртва дали нещо е "инстанция" на дадена структура

---

Пример

```elixir
defmodule P do
  def children_names(%Person{children: []}), do: "No children"
  def children_names(%Person{children: [name]}), do: "My childe is called: #{name}"
  def children_names(%Person{} = p) do
    p.childern
    |> Enum.join(", ")
    |> then(&"The name of my children are: #{&1}")
  end
end
```

```elixir
P.children_names(%Person{name: "Пешо", children: []})
```

```elixir
neo_pesho = %Person{name: "Пешо", children: ["Иванчо", "Драганчо"]}
P.children_names(neo_pesho)
```

---

### За какво и как да ползваме структури
![Image-Absolute](assets/types.jpg)

---

### За какво и как да ползваме структури

* Структура се дефинира в модул с идеята, че е нещо като тип дефиниран от нас.
* В модула обикновено слагаме функции, които да работят с този тип:
  * такива които "конструира" инстанция на структрата по дадени аргументи
  * или фунциим, които приемат инстанция на структурата, като първи акгумент
* Така имаме на едно място дефиницията на типа и функциите за работа с него.

---
### Структурите НЕ СА класове
![Image-Absolute](assets/i_see.jpg)

---
#### Примери за структури от стандартната бибиотека

---
Range

```elixir
range = 3..93
IO.inspect(range, structs: false)
```

---
Regex

```elixir
regex = ~r("Red")
IO.inspect(regex, structs: false)
```
---
 MapSet

```elixir
set = MapSet.new([2, 2, 3])
IO.inspect(set, structs: false)
```

---
Time

```elixir
{:ok, time} = Time.new(12, 34, 56)
IO.inspect(time, structs: false)
```
---
## Протоколи
![Image-Absolute](assets/protocols.jpg)

---
#### Дефиниране на протокол
---
### Но първо бърз краш курс за JSON
---
Прости JSON типове:

* null (това е и стойността му)
* boolean (false или true)
* number (2, 5.5, -12 и др.)
* string ("Пешо")
---
Съставни JSON типове:

* array (нехомогенен списък от JSON типове)
  * [1, null, [false, true]]
* object (мап с ключове стрингове и стойности произволен JSON тип)
  * {"name": "Пешо", "children": ["Гошо", "Ташо"], "location" : {"country": "БAлгариА", "city": "Карнобат"}}

---

Всеки JSON тип е валиден JSON

---

Дефиниране на протокол:
* използваме макроса `defprotocol`
* в блок описваме сигнатурите на функциите от протокола
* можем да имплементираме протокола за произволен *erlang ТИП* или *elixir СТРУКТУРА*
---
#### JSON encoder

```elixir
defprotocol JSON do
  @doc "Converts the given data to its JSON representation"
  def encode(data)
end
```

```elixir
JSON.encode(nil)
```
---
- Протокол се имплементира с макрото `defimpl`.
- Ето кака бихме имплементираме `JSON` за атоми:

```elixir
defimpl JSON, for: Atom do
  def encode(true), do: "true"
  def encode(false), do: "false"
  def encode(nil), do: "null"

  def encode(atom) do
    JSON.encode(Atom.to_string(atom))
  end
end
```

---
```elixir
JSON.encode(true)
```

```elixir
JSON.encode(false)
```

```elixir
JSON.encode(:name)
```

---
```elixir
defimpl JSON, for: BitString do
  def encode(<< >>), do: ~s("") # <=> "\"\""
  def encode(str) do
    cond do
      String.valid?(str) -> ~s("#{str}")
      true -> str |> bitstring_to_list() |> JSON.encode()
    end
  end

  # Измислена конвенция за кодиране на байтове и битов
  defp bitstring_to_list(binary) when is_binary(binary) do
    list_of_bytes(binary, [:bytes])
  end

  defp bitstring_to_list(bits), do: list_of_bits(bits, [:bits])

  defp list_of_bytes(<<>>, list), do: list |> Enum.reverse()
  defp list_of_bytes(<< x, rest::binary >>, list), do: list_of_bytes(rest, [x | list])

  defp list_of_bits(<<>>, list), do: list |> Enum.reverse()
  defp list_of_bits(<< x::1, rest::bits >>, list), do: list_of_bits(rest, [x | list])
end
```

---

```elixir
JSON.encode(:name)
```

```elixir
JSON.encode("")
```

```elixir
JSON.encode("some")
```

```elixir
JSON.encode(<< 200, 201 >>)
```

---

```elixir
defimpl JSON, for: List do
  def encode(list) do
    list
    |> Enum.map(&JSON.encode/1)
    |> Enum.join(", ")
    |> then(&"[#{&1}]")
  end
end
```

---

```elixir
JSON.encode([nil, true, false])
```

```elixir
JSON.encode(<< 200, 201 >>)
```

---

```elixir
defimpl JSON, for: Integer do
  def encode(n), do: n
end
```

---

```elixir
JSON.encode(<< 200, 201 >>)
```

---

```elixir
defimpl JSON, for: Map do
  def encode(map) do
    map
    |> Enum.map(&encode_pair/1)
    |> Enum.join(", ")
    |> then(&"{ #{&1} }")
  end

  defp encode_pair({key, value}) when is_binary(key) do
    "#{JSON.encode(to_string(key))}: #{JSON.encode(value)}"
  end
end
```

---

```elixir
free_pesho = %{
  name: "Pesho",
  age: 43,
  likes: [:drinking, "eating shopska salad", "да гледа мачове"]
}
```

```elixir
JSON.encode(free_pesho)
```

---
### Структури и протоколи
![Image-Absolute](assets/adapters.jpeg)

---

```elixir
defmodule Man do
  defstruct [:name, :age, :likes]
end
```

```elixir
kosta = %Man{
  name: "Коста",
  age: 54,
  likes: ["Турбо фолк", "Телевизия", "да гледа мачове"]
}

```

```elixir
JSON.encode(kosta)
```

---

Вградените типове за които можем да имплементираме протокол са:

* `Atom`
* `BitString`
* `Float`
* `Function`
* `Integer`

---

* `List`
* `Map`
* `PID`
* `Port`
* `Reference`
* `Tuple`

---

Има и начин да имплементираме протокол за всички случаи за които не е имплементиран,
използвайки `Any`:

```elixir
defimpl JSON, for: Any do
  def encode(_), do: "null"
end
```

---

Нека да пробваме сега

```elixir
JSON.encode(kosta)
```

---

За да използваме имплементацията за `Any` трябва да упомененм, че го наследяваме с модулния атрибут `@derive ProtocolName`

```elixir
defmodule Man2 do
  @derive JSON
  defstruct [:name, :age, :likes]
end
```

```elixir
nikodim = %Man2{
  name: "Никодим", age: 15, likes: ["Да лежи", "GTA V"]
}

JSON.encode(nikodim)
```

---

При дефиниране на протокол можен да използваме атрибута `@fallback_to_any` двайки му стойност `true`.

```elixir
defprotocol JSON2 do
  @fallback_to_any true

  @doc "Converts the given data to its JSON representation"
  def encode(data)
end
```
---

 Това означава, че всеки тип, който не имплементира протокола ще изпозва `Any`.

```elixir
defimpl JSON2, for Any do
  def encode(_data), do: "It's-a-Me, Any!"
end

JSON2.encode({:ok, :KO})
```

---

### Протоколи идващи с езика

```elixir
path = :code.lib_dir(:elixir, :ebin)
Protocol.extract_protocols([path])
```

---
### Протоколи идващи с езика

* `Collectable` - това е протоколът, използван от `Enum.into`.
* `Inspect` - използва се за _pretty printing_.
* `String.Chars` - `Kernel.to_string/1` го използва.
* `List.Chars` - `Kernel.to_charlist/1` го използва.
* `Enumerable` - функциите от `Enum` модула очакват типове, за които е имплементиран, като първи аргумент.

---

Имплементатори на даден протокол можем да намерим така:

```elixir
Protocol.extract_impls(Enumerable, [path])
```

---

```elixir
defimpl Enumerable, for: BitString do
  def count(str), do: {:ok, String.length(str)}
  def member?(str, char), do: {:ok, String.contains?(str, char)}

  def reduce(_, {:halt, acc}, _fun), do: {:halted, acc}
  def reduce(str, {:suspend, acc}, fun) do
    {:suspended, acc, &reduce(str, &1, fun)}
  end

  def reduce("", {:cont, acc}, _fun), do: {:done, acc}
  def reduce(str, {:cont, acc}, fun) do
    {next, rest} = String.next_grapheme(str)
    reduce(rest, fun.(next, acc), fun)
  end

  def slice(str), do: {:error, __MODULE__}
end
```

---
```elixir
"Еликсир"
|> Enum.filter(fn
     c when c in ~w(а ъ о у е и) -> false
     _ -> true
   end)
|> Enum.join("")
```

---
## Край
