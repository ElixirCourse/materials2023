---
theme: uncover
hidden: true
style: |
  .small-text {
    font-size: 0.75rem;
    letter-spacing: 1px;
    font-family: "Times New Roman", Tahoma, Verdana, sans-serif;
  }
  section {
    font-size: 28px;
    letter-spacing: 1px !important;
  }
  li {
    font-size: 28px;
    letter-spacing: 1px !important;
  }
  p.quote {
    line-height: 38px;
  }
  q {
    font-size: 32px;
    letter-spacing: 1px !important;
  }
  cite {
    text-align: right;
    font-size: 28px;
    margin-top: 12px;
    margin-bottom: 128px;
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
paginate: true
footer: 'Курс по Elixir 2023, ФМИ'
backgroundImage: "linear-gradient(to bottom, #E0EAFC, #CFDEF3)"
marp: true

---
## Низове и binaries

![Image-Absolute](assets/robots.jpg)

---
## Да си преговорим!

* Какви основни типове знаем?
* Как да правим наши "типове"?
* Какво е протокол?
* Кои са четирите слоя на езика?

---
## Съдържание

1. Как да работим с двоичните структури - binaries
2. Как са представени двоичните структури
3. Низове
4. Обхождане на низове
5. Списък тип charlist

---
## Binaries

![Image-Absolute](assets/binary_code.jpg)

---
## Binaries

### Конструкция
```elixir
<<>>

<< 123, 23, 1 >>

<< 0b01111011, 0b00010111, 0b00000001 >> == << 123, 23, 1 >>
# true

<< 280 >>
# <<24>>
```

---
### Конструкция

```elixir
<< 280::size(16) >> # Същото като << 280::16 >>
# <<1, 24>>

<< 0b00000001, 0b00011000 >> == << 280::16 >>

<< 129::7 >>
# <<1::size(7)>>
```

---
### Конструкция

- Интересно е съхранението на числа с плаваща запетая като `binaries`.
- Винаги се представят като `binary` от 8 байта (double precision):

```elixir
<< 5.5::float >>
# <<64, 22, 0, 0, 0, 0, 0, 0>>
```

---
### Конкатенация

![Image-Absolute](assets/concatenate.jpg)

---
### Конкатенация

```elixir
<< 83, 79, 83 >> <> << 24, 4 >>

<< 83, 79, 83, << 24, 4 >> >>
```

---
### Размер

```elixir
Kernel.byte_size(<< 83, 79, 83 >>)

byte_size(<< 34::5, 23::2, 12::2 >>)
```

---
### Размер

```elixir
Kernel.bit_size(<< 34::5, 23::2, 12::2 >>)
```

---
### Проверки

1. `Kernel.is_bitstring/1` - Винаги е истина за каквато и да е валидна поредица от данни между `<<` и `>>`. Няма значение дължината върната от `Kernel.bit_size/1`.
2. `Kernel.is_binary/1` - Истина е само ако `Kernel.bit_size/1` връща число кратно на `8` - тоест структурата е поредица от байтове.

---
### Sub-binaries

![Image-Absolute](assets/sub_binary.jpg)

---
### Sub-binaries

С функцията `Kernel.binary_part/3` можем да боравим с части от дадена `binary` структура:

```elixir
binary_part(<< 83, 79, 83, 23, 21, 12 >>, 1, 3)
# <<79, 83, 23>>

binary_part(<< 83, 79, 83, 23, 21, 12 >>, 2, -1)
binary_part(<< 83, 79, 83, 23, 21, 12 >>, 1, 6)
binary_part(<< 83, 79, 83, 23, 21, 12 >>, -1, 3)
```

---
### Използване в guard

Функцията `Kernel.binary_part/3` вътрешно ползва `:erlang.binary_part/3`, която е *BIF*, позволен в guard-ове:

```elixir
case <<17, 13, 14, 15>> do
  v when binary_part(v, 2, -2) == <<17, 13>> -> :ok
  _ -> :nope
end
```

---
Още една функция от Erlang : `:erlang.split_binary/2`, отново ползва `:erlang.binary_part/3`, но не може да е част от guard:

```elixir
:erlang.split_binary(<<4, 65, 129>>, 2)
# {<<4, 65>>, <<129>>}
```

---
### Pattern matching

Като всичко друго в Elixir, *binaries* също можем да съпоставяме:

```elixir
<< x, y, x >> = << 83, 79, 83 >>
x
y
```

---
```elixir
<< x, y, z::binary >> = << 83, 79, 83, 43, 156 >>
z
# <<83, 43, 156>>

<< x, y, z >> = << 83, 79, 83, 43, 156 >>

<< x, y, z::bitstring >> = << 83, 79, 83, 43, 156 >>
```

---
### Съпоставяне и числа с плаваща запетая

```elixir
<<
  sign::size(1),
  exponent::size(11),
  mantissa::size(52)
>> = << 4815.162342::float >>

sign
# 0
```

---
```elixir
:math.pow(-1, sign) *
  (1 + mantissa / :math.pow(2, 52)) *
  :math.pow(2, exponent - 1023)

```

---
### Модификатори

- *float*
- *binary* или *bytes*
- *integer* модификатор, който се прилага по подразбиране
- *bits* или *bitstrig*
- *utf8*
- *utf16*
- *utf32*
- суфикси *-little* и *-big* за избор на endianness

---
### Модификатори

```elixir
<< x::5, y::bits >> = << 225 >>
x
y

<< 0b11111::5 >> = x
<< 0b111::3 >> = y
```

---
### Модификатори

* *size* задава дължина в битове, когато работим с *integer*.
* Aко работим с *binary* модификатор, *size* е в байтове.

---
```elixir
<< x::binary-size(4), _::binary >> =
  << 83, 222, 0, 345, 143, 87 >>

x # 4 байта
# <<83, 222, 0, 89>>
```

---
## Binaries - имплементация

![Image-Absolute](assets/implementation.jpg)

---
### Иплементация

* Всеки процес в Elixir има собствен heap.
* Всеки процес съхранява различни структури от данни и стойности в този heap.
* Когато два процеса си комуникират, съобщенията се копират между heap-овете им.

---
### Heap Binary

* Ако *binary*-то е *64* **байта** или по-малко, то е съхранявано в *heap*-a на процеса си.
* Такива *binary* структурки наричаме **heap binaries**.

---
### Refc Binary

* Когато структурата ни е по-голяма от *64* **байта**, тя се пази в обща памет за всички процеси на даден `node`.
* В *porcess heap*-a се пази малко обектче - **ProcBin**.
* Binary структура може да бъде сочена от множество **ProcBin** указатели от множество процеси.

---
### Refc Binary

* Пази се *reference counter*, който брои всички тези указатели.
* Когато той стане *0*, Garbage Collector-ът ще може да изчисти *binary*-то от общия *heap*.
* Такива *binary* структури наричаме **refc binaries**.
* Също се създават при конкатенация (след малко)

---
### Sub binaries

* Указател към съществуващо *binary*, сочещ към част от него.
* Няма копиране.
* Броячът на референции се увеличава.
  * Това може да доведе до проблеми, когато имаме *sub binary* от много голямо *binary*, защото паметта няма да бъде освободена.
* Функцията *binary_part* създава *sub binary*.

---
### Match context

* Подобен на *sub binary*, но оптимизиран за *binary pattern matching*.
* Държи указател към двоичните данни в паметта.
* Когато нещо е *match*-нато, указателят се придвижва напред.

---
### Match context

* Компилаторът избягва създаването на *sub binary* (ако може).
* Ако е възможно се преизползва един и същ matching context.
* Повече на тази тема [тук](https://www.erlang.org/doc/efficiency_guide/binaryhandling.html)

---
### Копиране

```elixir
x = << 83, 222, 0, 89 >> # Heap binary (< 64 bytes)
y = x <> << 225, 21 >> # Refc (concat -> 2*y size or 256 => 256 bytes)
z = y <> << 125, 156 >> # Refc, but reuses the memory
u = z <> << 2, 4 >> # Refc, but reuses the memory
a = y <> << 15, 16, 19 >> # Refc (concat -> 2*a size or 256)
```

---
### Съпоставяне

```elixir
defmodule BPM do
  def binary_to_list(<< head, tail::binary>>) do
    [head | binary_to_list(tail)]
  end
  def binary_to_list(<<>>), do: []
end

BPM.binary_to_list("Let's there be list!")
```

---
### Динамично съпоставяне

- Можем да реферираме стойности, които вече сме съпоставили.

```elixir
str = String.duplicate("a", 56)              
# "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
size = byte_size(str)                            
# 56
size_value_bin = <<size::32, str::binary>>               
# <<0, 0, 0, 56, 97, 97, 97, ...>>
<<
  payload_size::32,
  payload::binary-size(payload_size)
>> = size_value_bin
# <<0, 0, 0, 56, 97, 97, 97, ...>>
payload_size
# 56
payload
# "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
```

---
### Интересен пример

* Erlang е създаден за решаване на проблеми в телеком индустрията
* Поради това, Erlang е особено подходящ за имплементация на binary протоколи
* В [тази поредица от блогпостве](http://blog.plataformatec.com.br/2018/11/building-a-new-mysql-adapter-for-ecto-part-i-hello-world/) може да видите какви са стъпките при имплементация на *MySQL* протокола в *Elixir*.

---
## Низове

![Image-Absolute](assets/strings.jpg)

---
Низовете в Elixir използват *unicode* - предтавляват binary структури -
поредица от *unicode codepoint*-и в *utf8* енкодинг.

---
## Низове

```
0XXXXXXX
```

- При *UTF-8* даден codepoint се състои от един до четири байта (може и в повече).
- Първите *128* кодпойнта съвпадат с *ASCII* *codepoint*-ите.
- Te се пазят в един байт, който започва с *0*:

---
## Низове

```elixir
<< 0b00110000 >>
# "0"
<< 0b00110001 >>
# "1"
<< 0b00110010 >>
# "2"
<< 0b00110101 >>
# "5"
<< 0b00111001 >>
# "9"
```

---
### UTF-8

```
110XXXXX 10XXXXXX
```

- След като *128*-възможности за *1* байт се изчерпат, започват да се ползват по *2* байта.
- В този шаблон за *2*-байтови *codepoint*-и, виждаме че първият байт винаги започва с *110*.
- Двете единици значат - *2* байта, байт започващ с *10*, означава че е част от поредица от байтове.

---

```elixir
?Ъ
#=> 1066

Integer.to_string(1066, 2)
# "10000101010"
```

---
Префиксваме с 110 и допълваме до байт с битовете от числото (110)10000

Префиксваме с 10 и допълваме до байт с битовете, които останаха (10)101010

```elixir
<< 0b11010000, 0b10101010 >>
"Ъ"
```

---
Шаблонът за *3*-байтови *codepoint*-и е:

```
1110XXXX 10XXXXXX 10XXXXXX
```

---
А шаблонът за *4*-байтовите:

```
11110XXX 10XXXXXX 10XXXXXX 10XXXXXX
```

---
Можем да заключим че има три типа байтове:

1. Единичен - започва с *0* и представлява първите *128* ASCII-compatible *codepoint*-a.
2. Байт-продължение - започва с *10*, следва други байтове в *codepoint* от повече от *1* байт.
3. Байт-начало - започва с *110*, *1110*, *11110*, първи байт от серия байтове представляващи *codepoint*.

---
### Графеми и codepoint-и

![Image-Absolute](assets/graphemes.jpg)

---
### Графеми и codepoint-и

* Символите или графемите не винаги са точно един *codepoint*.
* Има графеми, които могат да се представят и като един *codepoint* и като няколко.

---
### Графеми и codepoint-и

```elixir
name = "Николай"
String.codepoints(name)
# ["Н", "и", "к", "о", "л", "а", "й"]

String.graphemes(name)
String.graphemes(name) == String.codepoints(name)
```

---
### Графеми и codepoint-и

```elixir
name2 = "Николаи\u0306"
String.codepoints(name2)
#=> ["Н", "и", "к", "о", "л", "а", "и", ̆"<не може да се представи>"]
String.graphemes(name2)
# ["Н", "и", "к", "о", "л", "а", "й"]
```

---
А ето как да пишем директно codepoint-и `\u`:
```elixir
Integer.to_string(1066, 16) # Codepooint of "Ъ" in hex
# "42A"

"\u042Aгъл"

# Ако не са 4 hex цифри : \u{D...}
```

---
Функции като `String.reverse/1`, работят правилно, защото ползват `String.graphemes/1`.

```elixir
String.reverse("Николаи\u0306")

# Аналогично:
String.graphemes(name2) |> Enum.reverse |> Enum.join("")
```

---
### Дължина на низ

```elixir
Kernel.bit_size "Николаи\u0306"
Kernel.byte_size "Николаи\u0306"
String.length "Николаи\u0306"
```

---
### Под-низове

```elixir
String.at("Искам бира!", 6)
String.at("Искам бира!", -1)
String.at("Искам бира!", 12)
```

---
```elixir
<< _::binary-size(11), x::utf8, _::binary >> = "Искам бира!"

x
# 1073
<< x::utf8 >>
# б
```

---
```elixir
<< "Искам ", x::utf8, _::binary >> = "Искам бира!"
```

---
```elixir
String.next_codepoint("Николаи\u0306")
# {"Н", "иколай"}

String.next_codepoint("и\u0306")
String.next_grapheme("и\u0306")
# {"й", ""}

String.next_grapheme_size("и\u0306")
# {4, ""}
String.next_grapheme_size("\u0306")
```

---
```elixir
poem = """
  Ще строим завод,
  огромен завод,
  със яки
        бетонни стени!
  Мъже и жени,
  народ,
  ще строим завод
  за живота!
"""

large_factory = String.slice(poem, 18..30)
String.slice(poem, 18, 13)
```

---
## Обхождане на низове

![Image-Absolute](assets/iterate_path.jpg)

---
## Обхождане на низове

```elixir
str =
  (
    Enum.to_list(?a..?z) ++
    Enum.to_list(?A..?Z) ++
    Enum.to_list(?а..?я) ++
    Enum.to_list(?А..?Я) ++
    [32, 46]
  )
  |> Stream.cycle()
  |> Stream.take(2_000_000)
  |> Enum.shuffle()
  |> List.to_string()
```

---
```elixir
defmodule ACounterSlice do
  def count_it(str) do
    count(str, 0)
  end

  defp count("", n), do: n
  defp count(str, n) do
    the_next_n = next_n(String.starts_with?(str, "a"), n)
    count(String.slice(str, 1..-1), the_next_n)
  end

  defp next_n(true, n), do: n + 1
  defp next_n(false, n), do: n
end
```

---
Това е около *3.18* секунди за низ с дължина над 2_000_000

```elixir
{time, _result} = :timer.tc(ACounterSlice, :count_it, [str])
"#{time / 1000_000}s"
```

---
```elixir
defmodule ACounterNextGrapheme do
  def count_it(str) do
    count(str, 0)
  end

  defp count("", n), do: n
  defp count(str, n) do
    {next_grapheme, rest} = String.next_grapheme(str)
    count(rest, next_n(next_grapheme == "a", n)
    )
  end

  defp next_n(true, n), do: n + 1
  defp next_n(false, n), do: n
end
```

---
Добре същият отговор, но сега за *0.8* секунди

```elixir
{time, _result} = :timer.tc(ACounterNextGrapheme, :count_it, [str])
"#{time / 1000_000}s"
```

---
```elixir
defmodule ACounterMatch do
  def count_it(str) do
    count(str, 0)
  end

  defp count("", n), do: n

  defp count(<< c::utf8, rest::binary >>, n)
  when c == ?a do
    count(rest, n + 1)
  end

  defp count(<< _::utf8, rest::binary >>, n) do
    count(rest, n)
  end
end
```

---
Тази имплементация със същия низ е около *0.1* секунди.

```elixir
{time, _result} = :timer.tc(ACounterMatch, :count_it, [str])
"#{time / 1000_000}s"
```

---
## Списъци от codepoint-и (charlists)

![Image-Absolute](assets/64932835.jpg)

---
### Charlists

* Списък от codepoint-и е точно това: списък от цели числа, които са codepoint-и.
* Те са интересно наследство от Erlang, практически няма да ги ползвате много, но някои Erlang библиотеки може и да имат функции, които ги приемат като аргументи.
* Използваме единични кавички за да ги създадем.

---
## Списъци от codepoint-и (charlists)

```elixir
'Erlang'
[?E, ?r, ?l, ?a, ?n, ?g]

"Erlang" == 'Erlang'
```

---
Списъкът се принтира в кавички, само ако codepoint-ите са в ASCII-range (0-127, и могат да се принтират).

```elixir
'Ерланг'

is_list('Ерланг')

[69, 114, 108, 97, 110, 103]
[69, 114, 108, 97, 110, 103] ++ [33]

inspect([83, 79, 83], charlists: :as_lists)
```

---
За да виждаме списъци от числа винаги като списъци от числа в IEX:

```elixir
IEx.configure(inspect: [charlists: :as_lists])
```

---
### Списъци от codepoint-и и низове

* Можем да преобразуваме charlist към низ с `to_string`
* Обратното става с `to_charlist`

---
## Материали

* [Повече за binaries](https://elixir-lang.bg/materials/posts/binaries)
* [Повече за списъци](https://elixir-lang.bg/materials/posts/lists)
* [Повече за низове](https://elixir-lang.bg/materials/posts/strings)

---
## Край

![Image-Absolute](assets/end.jpg)
