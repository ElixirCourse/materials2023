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
    font-size: 80%;
    background: #f2f0ed;
    color: #080808;
  }
  li code,
  h4 code,
  p code {
    color: #089808;
    background-image: linear-gradient(to bottom, #E0EAFC, #CFDEF3);
    border-radius: 15px;
  }
  section {
    letter-spacing: 1px !important;
  }
  span.hljs-comment,
  span.hljs-quote,
  span.hljs-meta {
    color: #3d3f8f;
  }
  .hljs-keyword,
  .hljs-selector-tag,
  .hljs-tag,
  .hljs-name {
    color: #096a6a;
  }
  .hljs-attribute,
  .hljs-selector-id {
    color: #ffffb6;
  }
  .hljs-string,
  .hljs-selector-attr,
  .hljs-selector-pseudo,
  .hljs-addition {
    color: #489707;;
  }
  .hljs-subst {
    color: #526228;
  }
  .hljs-regexp,
  .hljs-link {
    color: #e9c062;
  }
  .hljs-title,
  .hljs-section,
  .hljs-type,
  .hljs-doctag {
    color: #724b99;
  }
  .hljs-symbol,
  .hljs-bullet,
  .hljs-variable,
  .hljs-template-variable,
  .hljs-literal {
    color: #b90f0f;
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
## Ecto

---
### Да си припомним

1. Метапрограмирането на Elixir може да манипулира кода до такава степен, че писането на собствен синтаксис не е трудно.
2. Всичко в Elixir е процеси, често OTP Application-и

---
### To ORM or not to ORM?

* RM (relational mapper) има за цел да *преобразува* структурирани между системи, които имат доста различни структури от данни.
* ORM (Object-relational mapper) често прави това между таблици в релационни бази от данни и обектно-ориентирани езици.
* При Elixir нямаме *O*-то, но пък имаме структури.

---
### To ORM or not to ORM?

* Можем да *map*-нем структура към таблица и можем да дефинираме структурата като схема на тази таблица.
* Можем да *map*-нем релации между таблици към структури и техни полета, сочещи към други структури.

---
### To ORM or not to ORM?

* При повечето ORM-ове правим доста усилие да имаме *mutable state*, представляващ *DB state*.
* При Ecto нямаме *mutable state*, имаме структура, която представлява ред от таблица, на която можем да приложим промяна(*changeset*) и да я запазим.

---
### Какво е Ecto?

* Ecto е *relational mapper*, чиито миграции са на DSL (domain specific language), който е направо SQL в Elixir.
* Ecto e *relational mapper*, чиито модели са просто структури, често 1-1 с таблицата в базата.

---
### Какво е Ecto?

* Ecto е *query builder*, който с метапрограмиране ни дава възмоцност да пишем DSL за заявки, много близък до SQL.
* Ecto е абстракция над *adapter* към дадена база от данни, така поддържа подобен език за множество различни бази.

---
### Какво е Ecto?

* Ecto ни позволява да пишем и изпращаме чист SQL с фрагменти и string-заявки.
* Еcto идва с мощен инструмент за валидиране и трансформиране на данни.
* Ecto е инструмент за работа с бази от данни, може да се използва по много различни начини, в зависимост кои части от него изберем да ползваме.

---
## Подготовка

- Да си направим нов проект!
- Малка библиотека за пазене на книги:

```bash
mix new books --sup
```


---
## Ecto Setup

- За да си инсталираме Ecto ни трябва `ecto_sql` и адаптер.
- За адаптер ще ползваме postgres, но има [много други](https://www.bigdatamark.com/list-of-available-ecto-sql-adapters-for-elixir).
- Това са ни началните `deps`:

```elixir
  defp deps do
    [
      {:ecto_sql, "~> 3.10"},
      {:postgrex, ">= 0.0.0"}
    ]
  end
```

---
### Ecto Setup

- Сега само трябва да ги инсталираме и да направим няколко промени.
- Командата `exto.gen.repo` ще ни каже какво да променим:

```bash
cd books

mix deps.get
mix ecto.gen.repo -r Books.Repo
```

---
### Ecto Setup

- Промените са да добавим `Books.Repo` като дете на Supervisor-a и да го конфигурираме:

```elixir
# Books.Application

children = [
  Books.Repo
]
```

---
### Ecto Setup

- Промените са да добавим `Books.Repo` като дете на Supervisor-a и да го конфигурираме:

```elixir
# config/config.exs

import Config

config :books,
  ecto_repos: [Books.Repo]


config :books, Books.Repo,
  database: "books",
  username: "booker",
  password: "bookerpass",
  hostname: "localhost"
```

---
### Ecto Setup

- Ще ни трябва и потребител в базата:

```sql
CREATE USER booker WITH LOGIN ENCRYPTED PASSWORD 'bookerpass' CREATEDB;
```

---
### Ecto Setup

- И можем да създадем базата си, защото имаме всичко необходимо:

```bash
mix ecto.create
```

---
### Ecto Repo

- Ecto ни генерира Repo (`Books.Repo`) което ни дава начин да се свържем с базата данни.
- То представлява процес, който включваме към application-а си.
- Също така задава адаптера към базата данни.

```elixir
defmodule Books.Repo do
  use Ecto.Repo,
    otp_app: :books,
    adapter: Ecto.Adapters.Postgres
end
```

---
### Ecto Repo

- Можем да го използваме да пишем заявки към базата.

```elixir
Books.Repo.query("SELECT 1")
#=> {:ok,
#=>  %Postgrex.Result{
#=>    command: :select,
#=>    columns: ["?column?"],
#=>    rows: [[1]],
#=>    num_rows: 1,
#=>    connection_id: 2860814,
#=>    messages: []
#=>  }}
```

---
### Ecto и миграции

* Ecto работи с миграции, които дефинират схемата на базата.
* Миграциите се намират в `priv/repo/migrations/` и са Elexir модули имплементиращи поведението `Ecto.Migration`.
* Имат функция `change` която задава промяната в базата, но също може да имат `up` и `down`, ако искаме по-добър контрол на отмяна на миграция.

---
### Ecto и миграции

- Ecto идва с mix команда за генериране на миграции:

```bash
mix ecto.gen.migration create_books
```

---
### Ecto и миграции

* Файловете на миграциите започват с дата на генериране, за да имат определен ред при прилагане.
* Скелетът на миграцията е модул, който има празен `change`.
* Ние ще го заменим с `up` и `down` за да демонстрираме `create table` и `drop table`.

---
### Ecto и миграции

```elixir
  def up do
    create table(:books) do
      add :isbn,        :string, size: 32
      add :title,       :string
      add :description, :string
      add :author_name, :string
      add :year,        :integer
      add :language,    :string

      timestamps()
    end
  end


  def down do
    drop table(:books)
  end
```

---
### Ecto и миграции

* Командата `mix ecto.migrate` ще приложи, неприложените миграции.
* Какво е приложено се пази в таблицата `schema_migrations`.
* С `mix ecto.rollback --step N` можем да върнем *N* миграции назад.
* За това е нужно да имаме `up` и `down` функциите за по сложни случаи.
* C `mix ecto.rollback` се връщаме с една стъпка назад.
* C `mix ecto.rollback --all` за да се върнем назад до началната база.

---
### Ecto и миграции

```
   Column    |              Type              | Nullable |
-------------+--------------------------------+----------+
 id          | bigint                         | not null |
 isbn        | character varying(32)          |          |
 title       | character varying(255)         |          |
 description | character varying(255)         |          |
 author_name | character varying(255)         |          |
 year        | integer                        |          |
 language    | character varying(255)         |          |
 inserted_at | timestamp(0) without time zone | not null |
 updated_at  | timestamp(0) without time zone | not null |
Indexes:
    "books_pkey" PRIMARY KEY, btree (id)

```

---
### Ecto и миграции

- Добра идея е да си добавим още една миграция с уникален индекс по isbn:

```elixir
  def change do
    create unique_index(:books, [:isbn], name: :books_isbn_unique_idx)
  end
```

---
### Ecto и миграции

* Миграциите ни дават възможност да си създаваме всякакви структури в схемата.
* Има синтаксис за индекси, релации и много други.
* Можем и да пишем чист SQL с `execute`.
* За повече информация [тук](https://hexdocs.pm/ecto_sql/Ecto.Migration.html)

---
### Ecto и заявки
