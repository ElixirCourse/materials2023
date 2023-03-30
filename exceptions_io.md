---
theme: uncover
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
## Грешки
## Вход-изход

---
### Да си припомним

* Какво е поведение?
* Какво е OTP?
* GenServer/Supervisor/Application?

---
## Грешки

![Image-Absolute](assets/exception.jpg)

---
Част от общата концепция на Elixir и Erlang е грешките да бъдат **фатални** и да **убиват процеса**,
в който са възникнали.

Нека някой друг процес (supervisor) се оправя с проблема.

---
![Image-Absolute](assets/fatality.jpg)

---
### Let It Crash!

* Грешките в Elixir **не са** препоръчителни за употреба.
* Имат славата на GOTO програмиране и наистина е хубаво да помислим дали има нужда от тях в дадена ситуация.

---
## Грешки

- Прието е функции, при които има проблем да връщат:
    ```elixir
      {:error, <проблем>}
    ```
- Ако се изпълняват с успех ще имат резултат:
    ```elixir
      {:ok, <резултат>}
    ```
- Имената на функции, които биха могли да 'вдигат' грешка, обикновено завършват на '!'.

---
### *'Вдигане'* и *'спасяване'*

![Image-Absolute](assets/iamerror.jpg)

---
#### Просто със стринг

```elixir
raise "Ужаст!"
#=> (RuntimeError) Ужаст!
```

---
#### Със структура-грешка

```elixir
raise RuntimeError
#=> (RuntimeError) runtime error
```

---
#### Със структура-грешка с аргумент

```elixir
raise ArgumentError, message: "Грешка, брато!"
#=> (ArgumentError) Грешка, брато!
```

---
```elixir
try do
  1 / 0
rescue
  [RuntimeError, ArgumentError] ->
    IO.puts("Няма да стигнем до тук.")
  error in [ArithmeticError] ->
    IO.puts("На нула не се дели, #{error.message}")
  any_other_error ->
    IO.puts("Лошаво... #{any_other_error.message}")
else
  IO.puts("Няма грешка.")
after
  IO.puts("Finally!")
end
```

---
### Създаване на нови типове грешки

![Image-Absolute](assets/new_exception.jpg)

---
#### Да се запознаем със структура-грешка

```elixir
defmodule VeryBadError do
  defexception message: "Лошо!!!"
end
```

---
```elixir
try do
  raise VeryBadError
rescue
  error in VeryBadError ->
    IO.puts(inspect(error, structs: false))
end
#=> %{
#=>    __exception__: true,
#=>    __struct__: VeryBadError,
#=>    message: "Лошо!!!"
#=> }
```

---
#### Да се запознаем със структура-грешка

```elixir
defmodule EvenWorstError do
  defexception [:message]

  @impl true
  def exception(value) do
    msg = "An even worst error was raised with value #{inspect(value)}"

    %EvenWorstError{message: msg}
  end
end
```

---
```elixir
try do
  raise EvenWorstError, :my_bad
rescue
  error in EvenWorstError ->
    IO.puts(error.message)
end
```

---
### Throw/Catch

![Image-Absolute](assets/spaghetti.jpg)

---
- С `throw` *'подхвърляме'* стойност, която може да се *'хване'* по-късно:

```elixir
try do
  b = 5
  throw b
  IO.puts "this won't print"
catch
  :some_atom           -> IO.puts "Caught #{:some_atom}!"
  x when is_integer(x) -> IO.puts(x)
end
#=> 5
```

---
- Нещо още по-рядко:

```elixir
try do
  exit("I am exiting") # така можем да излезем от процес
catch
  :exit, _ -> IO.puts "not really"
end # и процесът всъщност остава жив
```

---
### Сега - забравете за тях и не ги ползвайте.

![Image-Absolute](assets/forget.jpg)

---
1. В кода на mix няма прихващане на грешки.
2. В кода на компилатора на Elixir има само няколко прихващания.

---
### Грешките могат да се третират като съобщения

```elixir
defmodule Quitter do
  def run do
    Process.sleep(3000)

    exit(:i_am_tired)
  end
end

Process.flag(:trap_exit, true)

spawn_link(Quitter, :run, [])
receive do msg -> IO.inspect(msg); end

# {:EXIT, #PID<0.95.0>, :i_am_tired}
```

---
### Случай 1 : Нормално излизане на процес

```elixir
action = fn -> :nothing end
system_process = true # Ако е false този процес ще си чака

Process.flag(:trap_exit, system_process)
pid = spawn_link(action)

receive do
  msg -> IO.inspect(msg)
end

# {:EXIT, <pid>, :normal}
```

---
### Случай 2 : Излизане с **exit/1**

```elixir
action = fn -> exit(:stuff) end
system_process = true # Ако е false този процес ще умре с ** (EXIT from <pid>) :stuff

Process.flag(:trap_exit, system_process)
pid = spawn_link(action)

receive do
  msg -> IO.inspect(msg)
end

# {:EXIT, <pid>, :stuff}
```

---
### Случай 3 : Излизане с **exit(:normal)**
- Аналогично на случай 1:

```elixir
action = fn -> exit(:normal) end
system_process = true # Ако е false този процес ще си чака

Process.flag(:trap_exit, system_process)
pid = spawn_link(action)

receive do
  msg -> IO.inspect(msg)
end

# {:EXIT, <pid>, :normal}
```

---
### Случай 4 : Излизане с **raise**

```elixir
action = fn -> raise("Stuff") end
system_process = true # Ако е false този процес умира с
# [error] Process <pid> raised an exception ** (RuntimeError) Stuff

Process.flag(:trap_exit, system_process)
pid = spawn_link(action)

receive do
  msg -> IO.inspect(msg)
end

# {:EXIT, <pid>,
#   {%RuntimeError{message: "Stuff"},
#     [{:erl_eval, :do_apply, 6, [file: 'erl_eval.erl', line: 668]}]}}
```

---
### Случай 5 : Излизане с **throw**

```elixir
action = fn -> throw("Stuff") end
system_process = true # Ако е false този процес умира с
# [error] Process <pid> raised an exception ** (ErlangError) erlang error: {:nocatch, "Stuff"}

Process.flag(:trap_exit, system_process)
pid = spawn_link(action)

receive do
  msg -> IO.inspect(msg)
end

# {:EXIT, <pid>,
#  {{:nocatch, "Stuff"},
#    [{:erl_eval, :do_apply, 6, [file: 'erl_eval.erl', line: 668]}]}}
```

---
### Функцията **Process.exit/2**

* Може да се използва да ликвидираме процес от друг процес
* Ако извикаме с `:kill` можем да убием даже процеси, които trap-ват сигнали.
* Малко повече на тема [Process.exit](https://furlough.merecomplexities.com/elixir/otp/2021/05/31/the-many-and-varied-ways-to-kill-an-otp-process.html)

---
## Вход-изход

![Image-Absolute](assets/pipes.jpg)

---
## Вход-изход
- Изход с `IO.puts/2` и `IO.write/2`

```elixir
IO.puts("По подразбиране пишем на стандартния изход.")
IO.puts(:stdio, "Можем да го направим и така.")
IO.puts(:stderr, "Или да пишем в стандартния изход за грешки.")
```

---
```elixir
IO.write(:stderr, "Това е грешка!")
```

---
### Принтиране

* Първият аргумент на `puts` и `write` може да е атом или *pid*. Нарича се `device` и представлява друг процес.
* Вторият аргумент се очаква да е нещо от тип *chardata*.

---
![Image-Absolute](assets/wait_what.jpg)

---
### Какво е *chardata*?

* Низ, да речем `"Далия"`.
* Списък от codepoint-и, да речем `[83, 79, 0x53]` или `[?S, ?O, ?S]` или `'SOS'`.
* Списък от codepoint-и и низове - `[83, 79, 83, "mayday!"]`.
* Списък от chardata, тоест списък от нещата в горните три точки : `[[83], [79, ["dir", 78]]]`.

---
### Какво е *chardata*?

```elixir
IO.chardata_to_string([1049, [1086, 1091], "!"])
#=> "Йоу!"
```

---
### **IO.inspect/2**

* Връща каквото му е подадено. Може да се `chain`-ва.
* Приема *pretty print* опции.
* Приема етикети.
* Чудесно за debugging.

---
### **IO.inspect/2**

```elixir
defmodule TaskEnum do
  def map(enumerable, fun) do
    enumerable
    |> IO.inspect(label: "Input", structs: false)
    |> Enum.map(& Task.async(fn -> fun.(&1) end))
    |> IO.inspect(label: "Tasks", width: 120, limit: :infinity)
    |> Enum.map(& Task.await(&1))
  end
end
```

---
### Четене от стандартния вход

- Вход с IO.read/2, IO.gets/2, IO.getn/2 и IO.getn/3

```elixir
IO.read(:line)
Хей, Хей<enter>
#=> "Хей, Хей\n"
```

```elixir
IO.gets("Кажи нещо!\n")
Кажи нещо!
Нещо!<enter>
#=> "Нещо!\n"
```

---
- IO Server Time

![Image-Absolute](assets/its-showtime.jpg)

---
### Още функции

* Функции като `write` и `read` имат версии наречени `binwrite` и `binread`.
* Разликата е, че приемат `iodata`, вместо `chardata`.
* По бързи са. Добри за четене на *binary*/не-unicode файлове.


---
### Какво е *iodata*?

* Подобно на *chardata*, *iodata* може да се дефинира като списък.
* За разлика от *chardata*, *iodata* списъкът е от цели числа които представляват байтове (0 - 255),
* *binary* с елементи със *size*, кратен на **8** (могат да превъртат) и такива списъци.
* Писане без конкатениране? Повече [тук](https://bignerdranch.com/blog/elixir-and-io-lists-part-1-building-output-efficiently/)

---
### Какво е *iodata*?

```elixir
IO.iodata_length([1, 2 | <<3, 4>>])
#=> 4
```

```elixir
IO.iodata_to_binary([1, << 2 >>, [[3], 4]])
#=> <<1, 2, 3, 4>>
```

---
### Файлове

![Image-Absolute](assets/folders.jpg)

---
### Файлове

```elixir
{:ok, file} = File.open("test.txt", [:write])
#=> {:ok, #PID<0.855.0>}

IO.binwrite(file, "some text!")
#=> :ok

File.close(file)
#=> :ok
```

---
### Процеси и файлове

* Така наречения device всъщност е PID на процес или атом, който е ключ, сочещ към PID на процес.
* Когато отваряме файл се създава нов процес, който знае file descriptor-а на файла и управлява писането и четенето към и от него.
* Това означава, че всяка операция с файла минава през комуникация между процеси
* Именно за това има функции, които направо работят с файлове, като File.read/1, File.read!/1, File.write/3, File.write!/3.

---
### Потоци и файлове

```elixir
{:ok, file} = File.open("program.txt", [:read])
#=> {:ok, #PID<0.82.0>}

IO.stream(file, :line)
|> Stream.map(fn line -> line <> "!" end)
|> Stream.each(fn line -> IO.puts line end)
|> Stream.run()
```

---
### Потоци и файлове

- За повече четене [тук](https://cloudless.studio/elixir-vs-ruby-file-i-o-performance-updated)

```elixir
File.stream!(input_name, read_ahead: <buffer_size>)
|> Stream.<transform-or-filter>
|> Stream.into(File.stream!(output_name, [:delayed_write]))
|> Stream.run
```

---
### Модула `IO.ANSI`

```elixir
IO.puts [IO.ANSI.blue(), "text", IO.ANSI.reset()]
```

---
### Модула `StringIO` и файлове в паметта

```elixir
{:ok, pid} = StringIO.open("data") #PID<0.136.0>}
StringIO.contents(pid) # {"data", ""}

IO.write(pid, "doom!") #:ok
StringIO.contents(pid) # {"data", "doom!"}
IO.read(pid, :line) # "data"
StringIO.contents(pid) # {"", "doom!"}

StringIO.close(pid) # {:ok, {"", "doom!"}}
```

---
### Модула `StringIO` и файлове в паметта

```elixir
{:ok, file} = File.open("data", [:ram])
#=> {:ok, {:file_descriptor, :ram_file, #Port<0.1578>}}
IO.binread(file, :all)
#=> "data"
```

```elixir
IO.binread(file, :all)
#=> ""
```

---
### Модула `StringIO` и файлове в паметта

```elixir
:file.position(file, :bof)
IO.binread(file, :all)
#=> "data"
```

---
### Модула `Path`

```elixir
Path.join("some", "path")
#=> "some/path"
Path.expand("~/development")
#=> "/home/meddle/development"
```

---
### Портове

![Image-Absolute](assets/process_file.jpg)


---
## The End
![Image-Absolute](assets/nap.jpg)

