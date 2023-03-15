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

## Конкурентно програмиране 2

![Image-Absolute](assets/To-be-a-one-man-band.jpg)


---

## Но първо да си преговорим!

* **Конкурентност**
  * Способността различни части от една програма да се изпълняват out-of-order, частично и припокриващо се, без това да повлияе на резултата.
* **Паралелизъм**
  * Способността две или повече задачи или подзадачи да се изпълняват едновременно, използвайки множество ресурси (процесорни ядра).
* **Скалируемост**
  * Способността на една система да се справи с повишаването на натоварването чрез добавяне на хардуерни ресурси.

---

## Но първо да си преговорим!

* Какви начини имаме за създаване на процеси?
* Какво беше link?
  * Какво става с един процес, когато някой негов свързан процес терминира?
  * Ами, ако процесът `trap`-ва изходи?
  * Какви сигнали получавахме?
  * `{:EXIT, pid, error}`
* Какво беше monitor?
  * Какво съобщение получаваме, като някой "наблюдаван" процес умре?
  * `{:DOWN, ref, :process, pid, reason}`
* Какви са разликите между link & monitor?

---
## Съдържание

0. Неща, които ще трябва да знаем
1. Абстракцията `Agent`
2. Абстракцията `Task`

---
## Неща, които ще трябва да знаем

---

### Process.sleep/1

* "Приспива" процеса, от който викаме функцията.
* Единственият аргумент, който получава е време (в милисекунди), за което процеса да "спи".
* Докато един процес "спи" той не прави нищо.
* Не го ползваме, за да чакаме "нещо" да се случи
* Не го ползваме, за да чакаме процес да умре

---

### Една възможна имплементация на sleep

```elixir
sleep = fn milliseconds ->
  receive do
  after
    milliseconds -> :ok
  end
end

### Process.sleep/1

Използване:

```elixir
Process.sleep(10_000)
# Спим за 10 секунди
Process.sleep(:infinity)
# Спим до безкрай - процесът спира да отговаря на съобщения
```

---

#### Лошо:

```elixir
pid = spawn(fn -> do_something() end)

# Чакаме докато работата е свършена, или процесът умре
Process.sleep(2_000)

Process.alive?(pid)
```

---

#### Добро:

```elixir
parent = self()

pid = spawn(fn ->
  do_something()
  send(parent, :work_done)
end)

receive do
  :work_done -> continue_doing_something()
after
  30_000 ->
    # Можем да помислим и за таймоут, ако искаме
    {:error, :timeout}
end
```

---

#### Добро:

```elixir
parent = self()

pid = spawn(fn ->
  do_something()
  send(parent, :work_done)
end)

ref = Process.monitor(pid)

receive do
  {:DOWN, ^ref, _, _, _} -> :process_is_down
after
  30_000 ->
    # Можем да помислим и за таймоут, ако искаме
    {:error, :timeout}
end
```

---

### Kernel.apply/2

* Позволява ни да извикаме анонимна функция с аргументите ѝ
* Ако compile-time знаем функцията е по-добре да не го използваме

---
### Kernel.apply/2

Използва се така:

```elixir
func = fn x, y -> x * y end
apply(func, [3, 4]) # => 12
# горното е аналогично на func.(3, 4)
apply(func, [3]) # => Грешка
# ** (BadArityError) #Function<41.3316493/2 in :erl_eval.expr/6> with arity 2 called with 1 argument (3)

```

---

### Kernel.apply/3

* Същото като Kernel.apply/2, но за именувани функции
* Използва се така: </br>apply(Module, Function, Arguments)
* Module е атом
* Function е атом
* Arguments е списък с аргументи, които да бъдат подадени на функцията
* Често на това ще казваме MFA

---

### Kernel.apply/3

Използване:

```elixir
defmodule A do
  def mult(x, y), do: x * y
end

apply(A, :mult, [3, 4]) # => 12
# горното е аналогично на A.mult(3, 4)
apply(A, :mult, [3]) # => Грешка
```

---

### Kernel.then/2

* Взима 2 аргумента: стойност и функция
* Подава първия аргумент на функцията и връща резултатът от извикването
на функцията с този аргумент
* Удобно когато искаме да използваме pipe (`|>`) оператора към анонимна
функция

---

### Kernel.then/2

Използване:
```elixir
1 |> then(fn x -> x * x end)


# Иначе би изглеждало така:
1 |> (fn x -> x * x end).()
```

---

### Kernel.tap/2

* Взима 2 аргумента: стойност и функция
* Подава първия аргумент на функцията и връща първия аргумент
* Удобно когато искаме да правим странични ефекти когато имаме някакъв
pipe-chain


---

### Kernel.tap/2

Използване:

```elixir
tap(5, fn x -> IO.puts(x * x) end
# Ще изпринтира 25, но ще върне като резултат 5


1..10
|> Enum.map(fn x -> x * x end)
|> tap(&IO.puts/1)
|> Enum.sum()
# Ще изпринтира квадратите преди да ги събере
```

---
## Агенти
![Image-Absolute](assets/agent.jpg)

---
### Агент

* `Agent` е абстракция върху процесите.
* Агентът представлява състояние, което е съхранено в отделен процес.
* Състоянието е достъпно от текущия или други процеси. Комуникацията се извършва чрез съобщения.
* Комуникацията чрез съобщения минава през пощенската кутия, т.е. достъпът е от множество паралелни процеси е безопасен .

---
### Агент

* Може да се стартира с `Agent.start/2` с анонимна функция задаваща начално състояние.
* Може да се стартира с и с `Agent.start/4` с MFA.
* Последният аргумент са Keyword списък с настройки (например `:name` за именуване на процеса)
* Има и версии `Agent.start_link/2` и `Agent.start_link/4`.
* Тези версии биха "убили" процеса на агента, ако този, който го е пуснал "умре" и обратното.

---
### Агентът като стойност

```elixir
{:ok, agent} = Agent.start(fn -> 1 end)
#=> {:ok, #PID<0.187.0>}

Agent.get(agent, fn v -> v end) #=> 1
Agent.update(agent, fn v -> v + 1 end) #=> :ok
Agent.get(agent, fn v -> v end) #=> 2
```

---
### Агентът като споделена стойност

```elixir
defmodule DependentTask do
  def execute(agent) when is_pid(agent) do
    agent
    |> Agent.get(&(&1))
    |> tap(fn value ->
      Process.sleep(500)
      IO.puts("Process running with value: #{inspect(value)}")
    end)
    |> run(agent)
  end

  defp run(n, agent) when n < 5, do: execute(agent)
  defp run(n, _), do: n * n
end
```

---
### Стойността е достъпна от множество процеси!

```elixir
{:ok, agent} = Agent.start(fn -> 1 end)
pid = spawn(DependentTask, :execute, [agent])


# => Process running with value: 1
# => Process running with value: 1
# ...

Agent.update(agent, fn _ -> 10 end) #=> :ok

# => Process running with value: 1
# => Process running with value: 10
Process.alive?(pid)
# => false
```

---
### Стойността е достъпна от множество процеси!

* Задачата и текущият процес имат достъп до една и съща стойност от трети процес - агента.
* Задачата по дефиниция зависи от стойността в агента.
* На практика можем да пуснем друга задача, която може да промени тази стойност (стига да има адреса или името на агента).

---
### Стойността е достъпна от множество процеси!

* По такъв начин получаваме нещо като споделено състояние.
* Разликата е, че това състояние е винаги immutable.
* Съобщенията които постъпват в процеса-агент се обработват в реда в който са получени.
* Възможно е да счупим нещата, но ако използваме правилно интерфейса на агента това няма да стане.

---
![Image-Absolute](assets/race_conditions.jpg)

---
#### Възможно е да имаме race conditions.

```elixir
defmodule Value do
  def init(v) do
    {:ok, pid} = Agent.start(fn -> v end)
    pid
  end

  def get(value), do: Agent.get(value, &(&1))
  def set(value, v), do: Agent.update(value, fn _ -> v end)
end
```

---
#### Възможно е да имаме race conditions.

```elixir
pid = Value.init(5)
action = fn ->
  Process.sleep(50)
  v = Value.get(pid)
  :ok = Value.set(pid, v * 2)
end

spawn(action)
spawn(action)
spawn(action)
spawn(action)

Process.sleep(200)
Value.get(pid)
# не е задължително да е 80
```

---
### Функциите на Agent

* `Agent.get` - "взема" стойност от агента.
* `Agent.update` - променя стойността на агента.
* `Agent.get_and_update` - комбинира предните 2 функции и ги изпълнява атомарно.
* С първите две дадохме примери.
* С третата можем да оправим предния пример!

---
### Функциите на Agent

* Досега описаните функции са "синхронни" (чака за отговор).
* `Agent.cast` - дава ни възможност да променим състоянието на агента асинхронно.

---
#### Защо всичко се изпълнява във функция?
![Image-Absolute](assets/thinking.jpg)

---
#### Защо всичко се изпълнява във функция?

* Тези функции се изпълняват в процеса-агент.
* Трябва да бъдат кратки, за да не взимат много от времето на агента.
* Промяната на състоянието на даден процес би трябвало да става в самия процес.
* Именно затова промяната и инициализирането на състоянието на `Agent`-а става във функции, които се изпълняват в процеса му.

---
#### Агента като сървър?

* Можем да мислим за `Agent`-а и като за сървър.
* Процесите, които го ползват са клиенти, които комуникират с този сървър.
* Агентът трябва да е много прост сървър, неговата роля е просто да съхранява състояние.
* Това състояние може да бъде инициализирано, четено и променяно.
* Тежки изчисления, свързани със състоянието на агента не са част от неговата роля.


---
### За какво може да позлваме Агент?

* Проста обвивка на някакъв state.
* Може да го ползваме, като кеш.
* В реалния живот за кеш няма да ползвате агенти, защото има по-добри инструменти за това.
* За тестване и междинна работа агентите са добър избор.


---
## Задачи
![Image-Absolute](assets/taskmaster.webp)

---
### Задача

* Най-лесният начин да изпълним нещо конкурентно на текущия ни код е с абстракцията, дадена ни в модула Task.
* Функциите от този модул работят със специална структура, описваща задачата.
* За сега не е необходимо напълно да разбираме какво съдържа тази структура.

---
### Задача

* Създаваме задача с една от функциите:
  * `Task.async/1` - вариант с анонимна функция
  * `Task.async/3` - вариант с MFA
* Можем да получим резултата от изпълнената задача с функциите `Task.await/2`

---

```elixir
task = Task.async(fn -> 1 + 1 end)

# Друга логика може да се изпълни тук...

# Когато сме готови:
Task.await(task) #=> 2
```

---
### async и await

* Функцията `Task.async` всъщност създава нов Elixir-ски процес.
* Този процес е специализиран да изпълни едно основно действие и да върне резултатът от него.
* Функцията `Task.await` ще блокира текущия процес докато задачата завърши и резултатът не е наличен.
* Ако резултатът е наличен, когато я извикаме, направо ще върне стойността му.

---

#### Какво ще стане, ако задачата отнеме твърде много време?

---
### async и await

```elixir
task = Task.async(fn ->
  Process.sleep(10_000) # Симулираме дълга задача
  1 + 2
end)

Task.await(task)
#=> ** (exit) exited in: Task.await(%Task{...}, 5000)
#=>     ** (EXIT) time out
```
---

### await и timeout

* Вторият параметър на Task.await/2 е timeout (5_000 по подразбиране).
* Ако извикаме await и това време изтече се случва грешка.
* Можем да дадем цяло положително число за timeout, което означава колко милисекунди да чакаме.
* Също така може да дадем и атома :infinity (означава, че няма timeout).

---
### await и timeout

```elixir
task = Task.async(fn ->
  Process.sleep(10_000) # Симулираме дълга задача
  1 + 2
end)

# Ще почакаме около 10 секунди
Task.await(task, 11_000) #=> 3
```

---

#### Какво ще стане ако извикаме `await` повторно, след като вече имаме резултат?

---

```elixir
task = Task.async(fn -> 2 + 2 end)

Task.await(task) #=> 4
Task.await(task)
#=> ** (exit) exited in: Task.await(%Task{...}, 5000)
#=>     ** (EXIT) time out
```

---
### async и await

* Повторното извикване ще блокира и ще чака резултат.
* Тъй като задачата вече не съществува, няма да има нов резултат.
* След пет секунди ще има timeout грешка.
* Изводът е, че едно извикване на async трябва да съответства на точно едно извикване на await!

---
### MFA

```elixir
task = Task.async(Kernel, :+, [2, 3])
Task.await(task) #=> 5
```

---
#### Parallel map

```elixir
defmodule TaskEnum do
  def map(enumerable, fun) do
    enumerable
    |> Enum.map(& Task.async(fn -> fun.(&1) end))
    |> Enum.map(& Task.await(&1))
  end
end
```

---

### Проверка за резултат на даден интервал
![Image-Absolute](assets/are_we_there_yet.jpeg)

---
### yield

* Абстракцията Task може да се ползва като future стойностите от други езици и системи.
* За тази цел, бихме могли да ползваме функцията `Task.yield/2`.
* Подобно на `Task.await/2`, yield блокира текущия процес за дадено време:
  * Ако резултатът не е готов за даденото време връща `nil`.
  * В противен случай връща `{:ok, result}`

---

```elixir
task = Task.async(fn ->
  Process.sleep(5_000)
  3 + 3
end)
Task.yield(task, 1_000) #=> nil
Task.yield(task, 2_000) #=> nil
Task.yield(task, 3_000) #=> {:ok, 6}
```

---

#### Какво ще се случи, ако извикаме await или yield от процес различен от този, който е стартирал задачата?

---

```elixir
task1 = Task.async(fn -> 4 + 3 end)

Task.async(fn -> Task.yield(task1) end)
#=> ** (EXIT from #PID<0.10199.0>) process exited with reason:
#=>     ** (ArgumentError) task %Task{owner: #PID<0.10199.0>, ...}
#=>       must be queried from the owner
#=>       but was queried from #PID<0.10205.0>
```

---
#### Структурата Task

```elixir
Task.async(Kernel, :+, [1, 2])
%Task{
  mfa: {Kernel, :+, 2},
  owner: #PID<0.111.0>,
  pid: #PID<0.125.0>,
  ref: #Reference<0.2737067043.2134179841.80100>
}
```

---
#### Структурата Task

* mfa - тройка от модул, име на функция и аргументи, от които започва
задачата
* owner - pid-а на процеса стартирал задачата
* pid - e pid-а на процеса изпълняващ задачата
* ref - е уникална референция

---
#### Структурата Task

* На пръв поглед изглежда, че с yield и await poll-ваме резултата от задачата, но не е така.
* Процесите в Elixir комуникират помежду си чрез размяна на съобщения, което прилича на изпращане на push нотификации.
* Когато задачата си свърши работата, резултатът бива изпратен до процеса, който я е стартирал (owner-а).
* Затова само owner-а на задачата може да извика Task.await/2 или Task.yield/2.
* Използваме ref, за да идентифицираме точно върнатото от задачата съобщение.

---

#### За какво тогава изпозваме pid?

---

```elixir
defmodule Calc do
  def sum(a, b) when is_number(a) and is_number(b), do: a + b
  def sum(_, _), do: Kernel.exit(:normal)
end
```

---

```elixir
task = Task.async(Calc, :sum, [4, 4])
Task.yield(task) #=> {:ok, 8}

task = Task.async(Calc, :sum, ["5", 4])
{:exit, :normal} = Task.yield(task) #=> {:exit, normal}
```

---

### yield

* Всъщност освен `nil` и `{:ok, result}`, `yield` може да върне `{:exit, reason}`
* Това се случва, когато процесът изпълняващ задачата е приключил действието си преди да върне резултат на owner-а
* В този случай owner-а ще получава служебно съобщение, което идентифицираме по pid-а на задачата.

---

### shutdown

- Друго нещо, за което ни е необходимо да имаме pid-а на задачата е, ако искаме да я прекратим.
- Това се прави с функцията `Task.shutdown/1`.
- `Task.shutdown/1` може да върне същите неща като `Task.yield/1`
- Тази функция също може да бъде извикана само от "собственика" на задачата.
- Когато "загасим" дадена задача, процесът изпълняващ я бива убит, ако не е завършил изпълнението си.

---

### shutdown

```elixir
task = Task.async(fn -> Process.sleep(10_000) end)

case Task.yield(task) || Task.shutdown(task) do
  {:ok, result} -> result
  {:exit, _} -> {:error, "Task exited!"}
  nil -> {:error, "Task took too much time!"}
end
#=> {:error, "Task took too much time!"}

Process.alive?(task.pid) #=> false
```

---

### shutdown

- Има версия `Task.shutdown/2`.
- Вторият аргумент е период, който даваме на процеса изпълняващ задачата да се "загаси" сам.
- След това задачата бива "загасена" на сила.
- Вторият аргумент може да е и атомът `:brutal_kill`, това означава: "Убий задачата моментално по специален начин!".

---

### start

- Можем да стартираме задача без да се интересуваме от резултата.
- Това се прави с функциите `Task.start/1` и `Task.start/3`.
- Използваме ги, ако просто искаме да "генерираме" някакъв страничен ефект.

---
### start

```elixir
Task.start(fn ->
  Process.sleep(5_000)
  IO.puts "Няма време!"
end)
```

---

### async_stream
![Image-Absolute](assets/async_stream.gif)

---
### async_stream

* Приема колекция и функция, която да изпълни на всеки елемент на колекцията.
* Подадената функция се изпълнява в различен Task за всеки елемент.
* Когато опитаме да консумираме този stream, всяка задача ще бъде изчакана дадено време.

---

#### Parallel map. </br>Пак ли?

```elixir
defmodule TaskEnum do
  def map(enumerable, fun) do
    enumerable
    |> Task.async_stream(fun)
    |> Stream.map(fn {:ok, val} -> val end)
    |> Enum.to_list()
  end
end
```

---
## Време за Демо
![Image-Absolute](assets/demo-time.jpg)


---
## Край
![Image-Absolute](assets/end.jpg)
