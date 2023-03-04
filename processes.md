---

theme: uncover
style: |
  .center_img {
    margin: auto;
    display: block;
  }

  .side_by_side {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
  }

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

## Увод в конкурентното програмиране с Elixir

<img class="center_img" src="assets/processes/concurrency_vs_parallelism.png" width="40%" />

---

## Дефиниции

* **Конкурентност**
  * Способността различни части от една програма да се изпълняват out-of-order или частично, без това да повлияе на резултата.
  * [Concurrent Programming, Mutual Exclusion (1965; Dijkstra)](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=30e83735eb72af97e7ab3ec7f0823b9a9ae5493c)
* **Паралелизъм**
  * Способността проблем да бъде разделен на две или повече под-задачи, които се изпълняват едновременно, използвайки множество ресурси (процесорни ядра)
* **Скалируемост**
  * Способността на една система да се справи с повишаването на натоварването чрез добавяне на хардуерни ресурси.

---

<img class="center_img" src="assets/processes/concurrency_vs_parallelism_coffe.png" width="70%" />

---
## Конкурентност и Паралелизъм

* **Конкуретност** се отнася до структурата на една програма. Тя се отнася до това как една програма е построена и 
* **Паралелизъм** се отнася до изпълнението на една програма.

* Пример за **конкурентна** програма:
  * Уеб сървър, който използва 1 процесорно ядро, но обслужва множество заявки, изпълнявайки частично всяка една заявка до нейното завършване.
* Пример за **паралелна** програма:
  * Същият пример като горния, но разполагаме с 8 процесорни ядра. Уеб сървърът може да обработва 8 заявки наистина едновременно.

---

## Конкурентност и Паралелизъм

* Една програма може да е конкурентна и паралелна.
* Една програма може да е конкуретна, но не паралелна.
* Една програма може да е паралелна, но не конкурентна.
* Една програма може да е нито паралелна, нито конкурентна.

---

## Модели за конкурентно програмиране

* **Чрез споделена памет**
  * Пример: A и B са две програми, които се изпълняват на един компютър и споделят файловата система.
  * Пример: А и B са две нишки, които достъпват обща променлива-стек и добавят и премахват елементи от него.
* **Чрез изпращане на съобщения**
  * Пример: A и B са уеб браузър и уеб сървър - А отваря connection към B и изпраща заявка за уеб страница; B обработва заявката и изпраща уеб страницата на A.

---

<div class="side_by_side">
<img src="assets/processes/shared_memory_concurrency_model.png" width="50%" />

<img  src="assets/processes/message_passing_concurrency_model.png" width="50%" />
</div>

---

## Процеси

* Всичко в Elixir се изпълнява в процес
* Кодът, изпълняван в процеса, е последователен и функционален
* Ако искаме да изпълним конкурентно задача A и задача B, то трябва да ги изпълним в отделни процеси.
* Когато един процес приключи своята работа, то той "умира". (`Process.alive?` връща `false`).
  
---

## Създаване на процеси

- `spawn/1` - създава процес, който изпълнява функцията, подадена като аргумент
- `spawn/3` - създава процес, който изпълнява функцията зададена чрез тройката Module, Function, Arguments (MFA)
- `spawn_link/{1,3}` - създава процес, подобно на `spawn`, но също така двата процеса се "свързват" (повече за това по-късно)
- `spawn_monitor/{1,3}` - създава процес, подобно на `spawn`, но също така създаденият процес бива "наблюдаван" от създаващия го процес (повече за това по-късно)

---

```elixir
iex> spawn(fn -> IO.puts("Hello from a process #{inspect(self())}!") end)
Hello from a process #PID<0.1303.0>!
#PID<0.1303.0>
```

```elixir
defmodule T do
  def print_sum(a, b, c) do
    IO.puts("#{a} + #{b} + #{c} = #{a + b + c}")
  end
end

iex> spawn(T, :print_sum, [1, 2, 3])
1 + 2 + 3 = 6
#PID<0.1344.0>
```

---

## Шаблони

---

![](assets/processes/beam_schedulers_os_strucutre.png)

---

## Типове данни - PID

* Пример: `#PID<0.481.0>`
* `pid` - Process identifier. Уникално име на жив процес. PID на приключил процес може да бъде преизползван.
* `pid` се визуализира като наредена тройка числа: `{node, id, serial}`.
  * Всички процеси на един node имат `node` равен на `0`.
  * `id` се увеличава с 1 при всяко създаване на процес.
  * Когато стигнем `MAXPROCS`, `serial` се увеличава с 1
  * Ако един `pid` има `node` различно от 0, то процесът е на  node.
  * 
* `self`, `spawn` и `spawn_link` връщат `pid`
* https://www.erlang.org/doc/efficiency_guide/advanced.html

---

## Типове данни - Ref

- Пример: `#Reference<0.3339068074.245366785.27512>`
- `ref` - Терм, уникален измежду всички свързани nodes
- Използва се за имплементация на синхронна комуникация чрез асинхронни съобщения (Как? За бонус точка.)
- https://www.erlang.org/doc/efficiency_guide/advanced.html

---

## Имплементация на синхронна комункциация чрез асинхронни съобщения

```elixir
ref = make_ref()
send(pid, {:get_state, ref})
receive do
  {:state, ^ref, state} -> state
end
```
```elixir
state = %{a: 1, b: 2, c: 3}
receive do
  {:get_state, ref} -> send(self(), {:state, ref, state})
end
```

---

## Типове данни

```elixir
iex(1)> self()
#PID<0.111.0>
iex(2)> :erlang.term_to_binary(node())                  
<<131, 100, 0, 13, 110, 111, 110, 111, 100, 101, 64, 110, 111, 104, 111, 115,
  116>>
iex(3)> :erlang.term_to_binary(self())                  
<<131, 88, 100, 0, 13, 110, 111, 110, 111, 100, 101, 64, 110, 111, 104, 111,
  115, 116, 0, 0, 0, 111, 0, 0, 0, 0, 0, 0, 0, 0>>

```

---

## Комуникация между процеси

- Съобщения
- Сигнали

---

## Демо

```elixir
Process.list()
 |> Enum.map(&{&1, Process.info(&1)}) 
 |> Enum.map(fn {pid, info} -> {pid, info[:reductions]} end) 
 |> Enum.sort_by(&elem(&1, 1), :desc)
 |> Enum.take(10)
```

- :dbg.tpl(:_, []); Process.sleep(1); :dbg.stop()