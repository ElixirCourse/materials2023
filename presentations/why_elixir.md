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

![Image-Absolute](assets/elixir-logo.png)

---

## Малко информация за курса
![Image-Absolute](assets/Craft-Beer.jpeg)

---

## Кои сме ние?

* Ние ползваме Elixir в свободното си време.
* Ние ползваме професионално Elixir.
* Това е четвъртото издание на курса след 3-годишно прекъсване.

---

## Кои сте вие?

* Харесвате функционалното програмиране.
* Намирате начина за писане на конкурентен код на другите езици труден.
* Чували сте за Elixir/Erlang и искате да учите Elixir.
* Искате да си обогатите познанията с още един, различен от останалите, език.

---

## Защо да не записвате курса?

* Ако сте записали курса, за да вземете няколко лесни кредита.
* Ако смятате, че ООП е единственият начин.
* Ако не можете да отделите време за домашни и проекти.
* Ако очаквате монади, комонади, апликативи, мощни типови системи - запишете **Agda**.

---

## Организация на курса

* Провежда се в **понеделник** (зала 101) и **сряда** (зала 200) **от 19:00 до 21:00**.
* Оценката се образува от 5-6 домашни и проект.
* **Няма** тестове, писмен и теоретичен изпит.
* Много възможности за бонус точки:
  * Активно участие в лекции;
  * Решаване на задачи-предизвикателства;
  * Принос към open-source проекти или към материалите на курса.

---

## Организация на курса

* **Сайт**: https://elixir-lang.bg
* **Презентации**: https://slides.elixir-lang.bg
* **Discord**: https://discord.gg/VvzJruw5PM
* **Фейсбук**: https://www.facebook.com/groups/636900123169076/
* **Github**: https://github.com/ElixirCourse
* **Moodle**: [Линк](https://learn.fmi.uni-sofia.bg/course/view.php?id=9236)
* **ElixirJobs**: https://elixirjobs.net/ (не е свързан с нас)

---

## Защо да учим нов език?

* Нали си знам PHP/JAVA/Python/Това-с-което-(ще-)си-вадя-хляба?
* Тези хипстъри, дето всяка година учат нов език, за нищо не стават!
* Всеки месец нова 'технология', то не може всичко, я!

---

![Image-Absolute](assets/Haters_gonna_hate.jpg)

---

## Какво е Elixir и какво ще учим в курса?

![Image-Absolute](assets/elixir-logo.png)

---

## Какво е Elixir?

* **Динамично-типизиран** и **функционален** език.
* Пълна и лесна **Erlang съвместимост**.
* Проектиран за **конкурентни**, **толерантни на грешки** и **скалируеми** програми.
* Практична имплементация на **Actor модела**.
* Мощно, просто и чисто **метапрограмиране**.
* **Preemptive scheduling**.
* Уникален начин за **справяне с грешки**.
* Мощни инструменти за **наблюдение** на изпълняващите се програми и **интеракция** с тях.

---

## Какво е Erlang?

![Image-Absolute](assets/what_is_erlang.png)

---

## Какво е Erlang?

* **Динамично-типизиран** и **функционален** език, създаден през 1986 г.
* Целта му е да реши проблемите на телеком индустрията:
  * **Конкурентност**;
  * **Скалируемост**;
  * **Дистрибутираност**;
  * **Толерантност към грешки**;
  * **Обновяване на кода без спиране на програмта**;
  * **Бърз отговор**, дори когато системата е под голямо натоварване.
* Телеком проблемите през 80-те са web server проблемите през 2023.

---

## Elixir е Erlang

* Elixir кодът се трансформира до [Erlang Abstract Format (EAF)](https://www.erlang.org/doc/apps/erts/absform.html).
* EAF се трансформира към byte код, който се изпълнява върху [BEAM VM](https://www.erlang.org/blog/a-brief-beam-primer/).
* Всички библиотеки написани на Erlang, могат да бъдат ползвани директно.

---

## Elixir е Erlang

<img src="assets/elixir_using_erlang.png" alt= “”>

---

![Image-Absolute](assets/joes_thesis.png)

---

<p class="quote"><q>You get simplicity by finding a slightly more sophisticated building block to build your theories out of.</q></p>
<cite>Alan Kay, "Power of Simplicity", 2015</cite>

<p class="quote"><q>Simplicity is a great virtue but it requires hard work to achieve it and education to appreciate it. And to make matters worse: complexity sells better.</q></p>
<cite>Edsger W. Dijkstra, "On the nature of Computer Science", 1984</cite>

---

## Функционално програмиране

![Image-Absolute](assets/functional.jpeg)

---

## Функционално програмиране

* Непроменими (immutable) данни.
* "Pure(er)" функции.
* Програмиране с трансформации вместо с мутации.
* Рекурсия вместо цикли.
* Кодът е съставен от и организиран във функции и модули (Elixir).
* Операторът "=" е оператор за съпоставяне (match operator), а не за присвояване (Elixir).
* Позволява писането на по-декларативен код. При правилна употреба води до по-ясен код.

---

```elixir
iex(1)> x = 1
1
iex(2)> 1 = x
1
iex(3)> {a, b, 2} = {2, 3, 2}
{2, 3, 2}
iex(4)> [h|t] = [1,2,3,4]
[1,2,3,4]
iex(5)> h
1
iex(6) t
[2,3,4]
iex(7)> 5 = b
** (MatchError) no match of right hand side value: 3
    (stdlib 4.2) erl_eval.erl:496: :erl_eval.expr/6
    iex:12: (file)
```

---

## Функционално програмиране

<img src="assets/non_functional_vs_functional_unrevealed.png" alt= “”>

---

## Функционално програмиране

<img src="assets/non_functional_vs_functional_revealed.png" alt= “”>

---

## Actor модел

* Математически модел за [конкурентни изчисления](https://en.wikipedia.org/wiki/Concurrent_computing), повлиян от физиката.
* Създаден, за да подпомага системи с висока конкурентност.
* Не е имплементиран от език извън академични проекти на 100%.
* [Swift Concurrency Manifest](https://gist.github.com/lattner/31ed37682ef1576b16bca1432ea9f782#introduction)

---

## Actor модел

* Всичко е Actor (подобно на "всичко е обект").
* Един Actor може:
  * Да изпраща краен брой съобщения до други Actor-и;
  * Да създава краен брой нови Actor-и;
  * Да дефинира поведение, което ще се изпълни, кога се получи ново съобщение.
* Съобщенията се пращат асинхронно.

---

## Actor модел

![Image-Absolute](assets/actor_model.png)

---

## Actor модел  

![Image-Absolute](assets/actor_model_working.png)

---

## Слоевете на Elixir

* Можем да разгледаме Elixir като език/технология от 4 слоя:
  * **Функционален** - Простият език, който върви вътре в процесите;
  * **Конкурентен** - Процесите, които вървят и си пращат собщения на една инстанция на BEAM (node);
  * **Толерантен към грешки** - Процесите се наблюдават взаимно с линкове и монитори. Грешките се прихващат и се обработват. Използват се предефинирани поведения;
  * **Дистрибутиран** - Системата върви на множество node-ове и процесите в тях могат да си комуникират.

---

## Elixir не е имплементация на Actor модела

* Създателите на Erlang не са били запознати с Actor модела.
* Създателите на Actor модела и създателите на Erlang са били повлияни от едни и същи идеи.
* Процесите се държат в голяма степен като Actor-и.
* Вътрешността на процесите не следва Actor модела.

---

## Конкурентност

* Блоковете за построяване на конкурентност са процесите.
* Целият код се изпълнява в тези процеси.
* Един процес е:
  * Дефиниран на ниво виртуална машина, не го бъркайте с OS процес;
  * Лек относно използвана памет - около ~2kb при създаването си;
  * Лек относно CPU - смяната на контекст се нуждае от 3 промени в регистрите, процес на OS ниво от към ~700;
  * Изолиран от другите процеси, има си собствен стек, хийп и кутия за съобщения;
  * Комуникира чрез асинхронни съобщения - изпраща и получава.

---

## Конкурентност

* Процесите се държат като Actor-и, защото могат да:
  * Изпращат асинхронно съобщения;
  * Създават нови процеси;
  * Решават какво ще се изпълни при получаване на следващото съобщение;
  * Получават асинхронно съобщения в кутията си за съобщения.
* *Communicate by sharing state* срещу *Share state by communicating*

---

## Конкурентност

![Image-Absolute](assets/beam_style.png)

---

## Amdahl’s Law

<q>Parallel programs only go as fast as their slowest sequential part</q>

![Image-Absolute](assets/amdahl.png)

---

<img src="assets/two_million_websocket_connections.png" alt="" width="80%" />

---

```elixir
n = 100
k = 1000
atomics_ref = :atomics.new(1, [])
:atomics.put(atomics_ref, 1, 0)

counter_add = fn _ -> for _ <- 1..n, do: :atomics.add(atomics_ref, 1, 1) end

:timer.tc(fn ->
  1..k
  |> Task.async_stream(
    counter_add,
    ordered: false,
    max_concurrency: System.schedulers_online()
  )
  |> Enum.to_list()
end) 
|> then(fn {microseconds, _} ->
  IO.puts("Completed #{n * k} actions in #{microseconds / 1000}ms")
  IO.puts("Counter = #{:atomics.get(atomics_ref, 1)}")
end)
```

* [Kotlin код решаващ същата задача](https://kotlinlang.org/docs/shared-mutable-state-and-concurrency.html#thread-safe-data-structures)

---

## Превантивно планиране (Preemptive Scheduling)

* Начин за "честно" разпределение на CPU ресурсите между процесите.
* Подобно на Scheduler-а на ОС - 1 CPU ядро "едновременно" изпълнява браузър, редактор, аудио, микрофон, дискорд, календар.
* Един процес не може да монополизира процесора.
  * Ако имаме 8 ядра и върху всяко ядро 1000 процеса изпълняват безкраен цикъл, то гарантирано 8001-вият процес ще получи процесорно време и ще свърши своята работа бързо.
* Нийл Армстронг и Бъз Олдрин са живи благодарение на тази идея!

---

<img src="assets/preemptive_scheduling_vs_others.png" alt= “”>

---

## Let it Crash

![Image-Absolute](assets/let_it_crash.png)

---

## Let it Crash

* Heisenbug vs Bohrbug.
* *Have You Tried Turning It Off And On Again?*
* Идеята не е да имаме неконтролируеми грешки навсякъде.
* Идеята е да трансформираме грешките и изключенията в инструменти, които можем да ползваме.
* Методът "Fight fire with fire" в селското стопанство.

---

## Let it Crash

![Image-Absolute](assets/supervision_of_let_it_crash.png)

---

## Let it Crash

![Image-Absolute](assets/types_of_supervision.png)

---

## Метапрограмиране

<q>Rule 1: Don't write macros</q>
<cite>Chris McCord, Metaprogramming in Elixir</cite>

---

## Метапрограмиране

* Програми, които манипулират програми.
* Макросите приемат код като аргументи и връщат код като резултат.
* Това спомага да разширим езика спрямо специфични нужди.
* Функция :: Данни -> Данни
* Макрос :: Код -> Код

---

## Метапрограмиране

![Image-Absolute](assets/macro_def.png)

---

```elixir
{:ok, str} = File.read(file)
{:ok, ast} = Code.string_to_quoted(str)

{_, list_of_functions} =
  ast
  |> Macro.prewalk(
    [],
    fn
      {:if, _, _} = ast_fragment, acc ->
        {ast_fragment, [:if | acc]}

      {:cond, _, _} = ast_fragment, acc ->
        {ast_fragment, [:cond | acc]}

      {:unless, _, _} = ast_fragment, acc ->
        {ast_fragment, [:unless | acc]}

      ast_fragment, acc ->
        {ast_fragment, acc}
    end
  )

list_of_functions |> Enum.count() 
```

---

## Метапрограмиране

![Image-Absolute](assets/macro_example.png)

---

## Метапрограмиране

![Image-Absolute](assets/macros_in_testing.png)

---

## Метапрограмиране

![Image-Absolute](assets/more_macros_in_testing.png)

---

## Метапрограмиране

![Image-Absolute](assets/defs_as_macros.png)

---

## Observability

[![Image-Absolute](assets/sasha.png)](https://www.youtube.com/watch?v=pO4_Wlq8JeI)

---

![Image-Absolute](assets/community.png)

---

![Image-Absolute](assets/present_and_future.png)

---

## Elixir и Машинно самообучение

* В последните години създателят на Elixir работи главно в тази посока.
* **Nx** - библиотека за  работа с многомерни тензори и линейна алгебра, използващи Google XLA и LibTorch.
  * Комбинира функционалността на numpy, jax и Huggingface pipelines.
* **Axon** - библиотека за създаване, оптимизация и трениране на невронни мрежи.
* **Explorer**, **LiveBook** и др. - библиотеки и продукти с функционалност, подобна jupyter notebook и pandas.
* Интеграция с Huggingface.
  
---

<img src="assets/hugging_face_1.png" alt="" />

---

<img src="assets/hugging_face_2.png" alt="" />

---

<img src="assets/hugging_face_3.png" alt="" />

---

<img src="assets/hugging_face_4.png" alt="" />

---

<img src="assets/hugging_face_image_classification.png" alt="" />

---

<img src="assets/hugging_face_text_classification.png" alt="" />

---

## Plug & Phoenix

![Image-Absolute](assets/phoenix_and_plug.jpg)

---

## Ecto

![Image-Absolute](assets/ecto.png)

---

## Ecto

<img src="assets/chat_gpt_writes_ecto.png" alt="" width="80%"/>

---

## Ще си говорим и за доста advanced неща

![Image-Absolute](assets/distribution.jpg)

---

## Кой използва Elixir?

* **Pepsi** - Да, напитките, ползват Elixir за софтуера си за маркетинг.
* **Spotify** - Хиляди рекуести в секунда, сайта се ползва доста.
* **Discord** - Разговори, съобщения, много request-и в реално време.
* **Apple** - Environment and supply chain innovation (ESCI)
  
---

## Кой използва Elixir?

* **Pinterest** - Явно имат трафик и нужди?
* **Toyota** - За сървъри, даващи информация за трафик. Също така car sharing услуги.
* **Financial Times** - Вътрешни GraphQL API-та.
* **Square Enix** - За да имате едни и същи акаунти за всичките им игри.

---

## Кой използва Elixir?

* **Aeternity** - Много неща около blockchain-и и smart contract-и
* **Weedmaps** - 420
* **Sketch** - Дизайнерите го ползват и харесват - софтуер за дизайн.
* **Moodle** - Имат нов проект - MoodleNet - социална мрежа за преподаватели

---

## Край

![Image-Absolute](assets/gimpy.jpg)
