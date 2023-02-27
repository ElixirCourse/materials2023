---
theme: uncover
hidden: true
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
## Типове и основни конструкции

![Image-Absolute](assets/basics.jpg)

---
## Да си преговорим!

* Правим си проекти с какво?
* Как пишем unit тестове?
* Програмите ни са?
* Четерите слоя на езика са?
* Трябва да можем да боравим с git!

---
## Основни типове
![Image-Absolute](assets/types.png)

---
## Типове и основни конструкции
### Числа

```elixir
1
10_000

0x53 # В шестнадесетична
0o53  # В осмична
0b11 # В двоична
3.14 # С плаваща запетая
1.0e-10
```

---
```elixir
1 + 41
21 * 2
54 / 6 # Връща резултат с плаваща запетая
Kernel.div(54, 6) # В повечето езици `/` прави това
Kernel.rem 11, 3 # А ето как получаваме остатъка.
```

---
```elixir
1 < 2
1 >= 1
1 != 2
1 == 2
1 == 1.0 # Операторът == сравнява по стойност
1 === 1.0 # Операторът === сравнява по стойност И тип
1 !== 1.0 # Операторът !== сравнява по стойност И тип
```

---
### Булеви стойности: true/false

```elixir
true
false
Kernel.is_boolean(true)
is_boolean(0)
true == false
```

---
### Атоми
* Атомите са константи, чието име е стойността им.
* Булевите стойности `true` и `false` всъщност са атомите `:true` и `:false`
* Имената на модули (колекции от функции и нещо повече) в Elixir също са атоми.
* Модули идващи от Erlang са реферирани от атоми.
* GC не ги събира.

---
### Атоми
* Удобни са за ползване като ключове в `map`-ове.
* Задължителна част от `keyword lists`.
* Често се използват в tuple-и за означаване на резултат от функция. Пример - ``{:ok, 2}``

---
### Атоми
* Освен ако не са в двойни кавички, атомите могат да съдържат подчертавки, цифри и латински букви, както и at(`@`).
* Атомите могат да завършват на `!` или на `?`.
* Идеални за pattern matching (съпоставянето им е еквивалентно на съпоставяне на числа)

---
### Атоми
```elixir
:atom
:true
SomeModule # Може и да не е дефиниран
is_atom(:atom)
is_atom(true)
true == :true
:"atom with a space" # Могат да се дефинират и така
```

---
### Низове
Низовете в `Elixir` се дефинират с двойни кавички и са с _UTF-8_ encoding:

---
### Низове
```elixir
"Здрасти"
"Здрасти #{:Danchou}" # Интерполация
```

---
```elixir
is_binary("Здрасти")
String.length("Здрасти") # Брой на символи
byte_size("Здрасти") # Брой на байтове
"Бял" <> " мерцедес!" # Конкатенация
```

---
### Списъци
* Има специален модул, `List`, за работа с тях.
* Не държат стойностите си подредени в паметта.
* Намирането на дължината им, четене на стойност по index, добавяне на стойност на index и триене на стойност на index са все линейни операции.

---
### Списъци
```elixir
[1, 2, "три", 4.0] # Не са хомогенни
length [1, 2, 3, 5, 8] # Дължината
hd [1, 2, 3, 5, 8]
tl [1, 2, 3, 5, 8]
is_list([1, 2])
```

---
### Списъци
* Списък може да се представи рекурсивно като главата на списъка (head), в която е стойността на първия му елемент и указател към следващия елемент.
* Тази дефиниция ни дава възможност да пишем функции, които работят със списъци.

---
```elixir
[1 | [2 | [3 | [4 | [5 | []]]]]]
```

---
#### Списъци от символи

```elixir
[83, 79, 83]

is_list('SOS')
```

---
```elixir
List.to_atom([83, 79, 83])
List.to_atom([83, 79, 83, 1])
List.to_atom([83, 79, 83, 1, :s])

# List.to_integer/1 и List.to_float/1 ще опитат да върнат числа от charlist:

List.to_integer('54')
List.to_float('45.2')

List.to_string([83, 77, 69, 82, 67, 72])
```

---
### Кортежи
![Image-Absolute](assets/tuple.jpg)

---
### Кортежи
* Кортеж ни е официалният превод от "tuple".
* Кортежите съхраняват елементите си подредени един след друг в паметта.
* Достъпът до елемент по индекс и взимането на дължината им са константни операции.

---
### Кортежи
* Заедно с атомите за връщане на множество стойности от функция.
* За `pattern matching` - ще видим малко по-долу.
* Read-only колекция, защото писането в тях е скъпа операция.

---
### Кортежи
```elixir
{:ok, 7}
tuple_size({:ok, 7, 5})
is_tuple({:ok, 7, 5})
```

---
### Keyword lists
Списъци, които съдържат `tuple`-и от по два елемента - атом и каквато и да е стойност.

---
### Keyword lists
```elixir
[{:one, 1}, {:two, 2}]
[one: 1, two: 2]
```

---
Ако keyword list е последен аргумент на функция, можем да пропуснем квадратните скоби при извикване:
```elixir
f(1, 2, three: 3, four: 4)
```

---
### Keyword lists
* Ключовете им могат да се повтарят.
* Използват се и за предаване на command line параметри или опции на функции.
* Пример е `String.split/3`.

---
```elixir
String.split("one,two,,,three,,,four", ",", trim: true)
```

---
### Maps
* Колекции от ключове и стойности.
* `Map`-овете в `Elixir` не позволяват еднакви ключове.
* За ключове може да се използва всичко и дори няма нужда да бъдат един и същи тип,
но обикновено се използват низове или атоми.

---
### Maps

```elixir
pesho = %{
  name: "Пешо",
  age: 35,
  hobbies: {:drink, :smoke, :eurofootball},
  job: "шлосер"
}

pesho[:name]
pesho.name
Map.fetch(pesho, :name)
Map.fetch!(pesho, :name)
Map.get(pesho, :name)
```

---
### Бинарен тип (Binaries)
* Представляват поредици от битове и байтове.

---
### Бинарен тип (Binaries)
```elixir
<< 2 >> # Цялото число 2 в 1 байт
byte_size << 2 >>
<< 255 >> # Цялото число 255 в 1 байт
<< 256 >> # Превърта и става 0
<<1, 2>> # Две цели числа в два байта.
byte_size << 1, 2 >>
```

---
```elixir
<< 5::size(3), 1::size(1), 5::size(4) >>
0b10110101
byte_size << 5::size(3), 1::size(1), 5::size(4) >>
is_bitstring << 5::size(3), 1::size(1) >>
is_binary << 5::size(3), 1::size(1) >>
```

---
### Бинарен тип (Binaries)
* Интересен факт - низовете в `Elixir` са имплементирани като `binary` тип.
* Спомняте си че `is_binary("Стринг")` връщаше `true`.

```elixir
<<208, 170, 208, 156>> = "ЪМ"
```

---
### Анонимни функции
```elixir
fn (x) -> x + 1 end
(fn (x) -> x + 1 end).(4) # Извикване
is_function((fn (x) -> x + 1 end))
```

---
```elixir
&(&1 + 1)
(&(&1 + 1)).(4)
```

---
### Други типове
* Други типове са `Port`, `Reference` и `PID`, които се използват с процеси.
* Има и регулярни изрази. `~r/\w+/im`
* Ranges : `(1..1000)`
* Има различни shortcut-синтаксиси за дефиниране на някои от типовете.


---
## Съпоставяне на образци
![Image-Absolute](assets/patterns.jpg)

---
### Съпоставяне на образци
* В Elixir `pattern matching`-ът е еднa от най-важните и основни особености.
* Операторът `=` се нарича `match operator`.
* Можем да го сравним с знака `=` в математиката.

---
### Съпоставяне на образци
* Използвайки го, превръщаме целия израз в уравнение, в което сравняваме лявата с дясната страна.
* Ако сравнението е успешно се връща стойността на това уравнение, ако не - има грешка.

---
## Съпоставяне на образци
```elixir
x = 5
5 = x
4 = x
```

---
Засега за match operator-а знаем:
1. С него могат да се дефинират променливи.
2. С него могат да се правят проверки - дали дадена променлива има дадена стойност.

---
## Съпоставяне на образци
* Имената на променливи задължително започват с малка латинска буква или подчертавка (`_`),
следвана от букви, цифри или подчертавки.
* Могат да завършват на `?` или `!`.
* Операторът `=` ще опита да присвои на всички възможни променливи от ляво стойности от дясно.

---
```elixir
{one, tWo, t3, f_our, five!} = {1, 2, 3, 4, 5}
one
tWo
t3
f_our
five!
```

---
```elixir
g = fn
  0 -> 0
  x -> x - 1
end
g.(0)
g.(3)
```

---
![Image-Absolute](assets/pins.jpg)

---
## Съпоставяне на образци (pin)
* В `Elixir` e възможно да променим стойността на променлива (не точно, но shadows!).
* В `Erlang` това не е възможно.

---
Ако искаме една променлива, която вече съществува да не промени стойността си при съпоставяне,
можем да използваме `pin` оператора - `^`.
```elixir
x = 5
^x = 6
```

---
```elixir
{y, ^x} = {5, 4}
```

---
Ако се опитаме да присвоим стойност на `unbound` променлива (досега не е съществувала),
използвайки `pin` оператора, ще получим грешка.
```elixir
^z = 4
```


---
### Параметри на функции

```elixir
defmodule Example do
  def factorial(0), do: 1
  def factorial(n), do: n * factorial(n - 1)
end
```

---
### Гардове

```elixir
defmodule Questionnaire do
  def how_old_are_you?(age) when age > 30 do
    "Имаме си чичко или леличка"
  end
  def how_old_are_you?(age) when age > 20 do
    "Имаме си милениалче, тригърнато от живота"
  end
  def how_old_are_you?(_), do: "Дете"
end

Questionnaire.how_old_are_you?(21)
```

---
### Списъци

```elixir
[a | b] = [1, 2, 3]

[a | b] = [25]
```

---
```elixir
[a | b] = []
```

---
```elixir
defmodule ListUtils do
  def length([]), do: 0
  def length([_head | tail]), do: 1 + length(tail)
end

ListUtils.length(["cat", "dog", "fish", "horse"])
```

---
```elixir
defmodule Square do
  def of([]), do: []
  def of([head | tail]), do: [head * head | of(tail)]
end

Square.of([1, 2, 3, 4, 5])
```

---
#### Фибоначи
```elixir
defmodule Fib do
  def bad_of(n), do: bad_fib(n, 0, 1, [])

  defp bad_fib(0, _current, _next, seq), do: seq

  defp bad_fib(n, current, next, seq) do
    bad_fib(n - 1, next, current + next, seq ++ [current])
  end
end
```

---
```elixir
defmodule Fib do
  def of(n), do: fib(n, 0, 1, [])

  defp fib(0, _current, _next, seq), do: seq |> Enum.reverse()

  defp fib(n, current, next, seq) do
    fib(n - 1, next, current + next, [current | seq])
  end
end
```

---
#### map/reduce
```elixir
defmodule ListUtils do
  def map([], _func), do: []
  def map([head | tail], func), do: [func.(head) | map(tail, func)]
end

ListUtils.map([1, 2, 3, 4, 5], fn x -> x * x end)
```

---
```elixir
defmodule ListUtils do
  def reduce([], acc, _func), do: acc
  def reduce([head | tail], acc, func), do: reduce(tail, func.(head, acc), func)
end

ListUtils.reduce(["cat", "dog", "horse"], 0, fn _head, acc -> 1 + acc end)
```

---
#### Речници
```elixir
pesho = %{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}

%{name: x} = pesho

%{name: _, age: _} = pesho
```

---
```elixir
defmodule A do
  def f(%{name: name} = person) do
    name
  end
end

A.f(pesho)
```


---
## Control Flow
![Image-Absolute](assets/control_flow.jpg)

---
## Control Flow
```elixir
a = :rand.uniform(1000)

if rem(a, 2) == 0 do
  "a's value is #{a}, it is even"
else
  "a's value is #{a}, it is even"
end
```

---
#### if / unless

```elixir
defmodule Questionnaire do
  def how_old_are_you?(age) do
    if age > 30 do
      "Имаме си чичко или леличка"
    else
      if age > 20 do
        "Имаме си милениалче, тригърнато от живота"
      else
        "Дете"
      end
    end
  end
end
```

---
Стойността на `if` или на `unless` е стойността на израза, който се оценява.

```elixir
age = 34
name = "Слави"

if age < 30 do
  "Младеж"
end

if age > 30, do: "Чичо #{name}", else: name
```

---
#### Cond
![Image-Absolute](assets/cond.png)

---
#### Cond
```elixir
defmodule FizzBuzz do
  def of(n), do: Enum.map(1..n, &number_value/1)

  defp number_value(n) do
    cond do
      rem(n, 3) == 0 and rem(n, 5) == 0 -> "FizzBuzz"
      rem(n, 3) == 0 -> "Fizz"
      rem(n, 5) == 0 -> "Buzz"
      true -> n
    end
  end
end
```

---
#### Cond
* В 'cond' слагаме списък от условия, със свързан с тях код.
* Оценяват се докато стигнем до някое, което се оцени като true.
* Когато това стане, се изпълнява асоциираният с него код и това е стойността на cond-a.


---
#### Cond
* Обикновено последното условие на cond е 'true', за да има код, който да се изпълни,
* Ако не го сложим и нищо не се оцени като истина ще има грешка.

---
#### Специални форми
* За разлика от if, cond е 'специална форма'.
* Специалните форми са най-базовите градивни единици в Elixir.
* Не могат да се пренапишат от нас.
* Те са специални macro-си, които не са на написани на езика.

---
#### Специални форми
* Конструкциите if и unless са добавени за да се справи един императивен програмист с навлизането в Elixir.
* Според нас са повече вредни за обучението, отколкото помагат.
* Забравете че ги има.

---
#### Case
![Image-Absolute](assets/case.jpg)


---
#### Case
```elixir
defmodule Questionnaire do
  def asl(age, sex, location) do
    case sex do
      :male ->
        "#{age}, М, #{location}"
      :female ->
        "#{age}, F, #{location}"
      _ ->
        "#{age}, ?, #{location}"
    end
  end
end
```

---
#### Case
* Нужна е последна клауза, която match-ва всякакви стойности.
* Ако нищо не се match-не се изпълнява тази последна клауза.
* Ако я няма ще имаме грешка (CaseClauseError).

---
#### Case
* Важно е да знаем, че case, също като cond е специална форма.
* Ползваме го само ако кодът ни стане прекалено разхвърлян или труден за разбиране с функции pattern matching.
* Забравете за if и unless, по добре използвайте case.

---
Защо case е по-добър избор пред cond?

* Проверките които правим в case, са свързани с data-та която му даваме.
* Лесно можем да съобразим какво и защо проверяваме.
* По-трудно да вкараме странични ефекти.

---
#### Case
* Проверките в cond могат да са всякакви и да са свързани с всякакви данни.
* Много по-лесно е да напишем код, в който се чудим кое от къде идва.
* Много по-лесно е да имаме странични ефекти.

---
#### Конструкцията with
![Image-Absolute](assets/with.jpg)

---
#### Конструкцията with
* Още една специална форма, която опростява писането на код.
* Това, което прави with е да комбинира множество успешни съпоставяния.
* Ако искаме няколко различни условия да са изпълнени за да направим нещо и ако само едно от тези условия не е изпълнено, да не го правим.

---
#### Конструкцията with
```elixir
defmodule HR do
  def work_experience(years) when years > 10, do: :experienced
  def work_experience(years) when years > 5, do: :advanced
  def work_experience(_), do: :not_experienced

  def knows_elixir?([]), do: false
  def knows_elixir?([:elixir | _]), do: true
  def knows_elixir?([_ | rest]), do: knows_elixir?(rest)

  def read_cv(file_path), do: File.read(file_path)
end
```

---
```elixir
years = 11

languages = [:erlang, :elixir, :rust]

cv_path = "/tmp/cv.txt"

with :experienced <- HR.work_experience(years),
     true <- HR.knows_elixir?(languages),
     {:ok, cv} <- HR.read_cv(cv_path),
     do: cv
```


---
Ако някое от условията не е изпълнено, ще получим стойността му:

```elixir
with :experienced <- HR.work_experience(3),
     true <- HR.knows_elixir?(languages),
     {:ok, cv} <- HR.read_cv(cv_path),
     do: cv
```

---
#### Конструкцията with

* Можем да правим още неща с with - погледнете документацията
* Погледнете и другите специални форми!

---
## Неизменимост
![Image-Absolute](assets/immutable.png)

---
## Неизменимост
```elixir
base_list = [1, 2, 3]
new_list = [0 | base_list]
```

---
## Материали
* [Повече за типове](https://elixir-lang.bg/materials/posts/types)
* [Повече за списъци](https://elixir-lang.bg/materials/posts/lists)
* [Повече за речници](https://elixir-lang.bg/materials/posts/maps)

---
## Край
![Image-Absolute](assets/end.jpg)
