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

- Ecto не е framework, a по-скоро множество от инструменти за:
  - Имплементация на *Repository pattern* за комуникация с база данни (или други).
  - Дефиниране на схеми.
  - Дефиниране на валидации.
  - Дефиниране на операции чрез структури от данни (query).
  - Дефиниране на промени чрез структури от данни (changeset).
- Можем да използваме всеки от инструментите отделно (валидации без бази данни, например changeset за обработка на уеб форма).


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
## Ecto и заявки

* Можем да използваме Ecto DSL да пишем всякакви заявки към базата.
* Няма защо да дефинираме RM (relational mapping) за да правим заявки от `Repo`.
* Такива заявки наричаме *schemaless* - не сме дефинирали схема в Elixir но ги правим.

---
### Ecto и заявки : insert_all

- Този тип заявки могат да запишат в базата един или много редове:

```elixir
  def insert_book!(isbn, title, description, author_name, year, language) do
    {1, [%{id: id}]} =
      Repo.insert_all(
        "books",
        [
          [
            isbn: isbn,
            title: title,
            description: description,
            author_name: author_name,
            year: year,
            language: language,
            inserted_at: DateTime.utc_now(),
            updated_at: DateTime.utc_now()
          ]
        ],
        returning: [:id]
      )

    id
  end
```

---
### Ecto и заявки : all

- С `Repo.all` можем да правим `SELECT` заявки:

```elixir
  def books_by_year(year) do
    query =
      from("books",
        where: [year: ^year],
        select: [:title, :author_name]
      )

    Repo.all(query)
  end
```

---
### Ecto и заявки : all

- `Repo.all` също така поддържа всичко. нужно на `SELECТ`, като `group_by`, `limit`, `join` и други:

```elixir
  def books_by_years_or_authors(years, authors) do
    query =
      from(b in "books",
        where: b.year in ^years or b.author_name in ^authors,
        select: [:id, :isbn, :title, :author_name, :year, :description]
      )

    Repo.all(query)
  end
```

---
### Ecto и заявки : one

- Функцията `Repo.one` се ползва ако задължително очакваме един резултат максимално:

```elixir
  def book_by_id(id) do
    query =
      from("books",
        where: [id: ^id],
        select: [:id, :title, :author_name, :year]
      )

    Repo.one(query)
  end
```

---
### Ecto и заявки : update_all

- Можем да използваме цялата сила на езика за заявки за да направим `UPDATE`:

```elixir
  def update_book_description!(book_id, description) do
    query = from(b in "books", where: b.id == ^book_id)

    {1, _} = Repo.update_all(query, set: [description: description])
  end
```

---
### Ecto и заявки : delete_all

- Това се отнася и за `DELETE`:

```elixir
  def delete_book_by_isbn!(isbn) do
    query = from(b in "books", where: b.isbn == ^isbn)

    {1, _} = Repo.delete_all(query)
  end
```

---
## Ecto схеми

* Можем да използваме Ecto само като *query builder*.
* Но можем да изберем и да опишем схемата си със структури.
* Така получаваме няколко неща:
  * Описание и документация на таблиците в кода, както и структурирано представяне, не просто `map`-ове.
  * Валидация на ниво Elixir код.
  * Лесен начин на зареждане на свързани редове от други таблици (preload).

---
## Ecto схеми : с миграции

- Ако си дефинираме миграция за `countries` така:

```elixir
  def change do
    create table(:countries) do
      add :name, :string, size: 32, null: false
      add :code, :string, size: 2, null: false

      timestamps()
    end

    create unique_index(:countries, [:name])
    create unique_index(:countries, [:code])
  end
```

---
## Ecto схеми : с миграции

- Съответната схема би била:

```elixir
defmodule Books.Country do
  use Ecto.Schema

  import Ecto.Changeset

  schema "countries" do
    field(:name, :string)
    field(:code, :string)

    timestamps()
  end
end
```


---
## Ecto схеми : с миграции

* Схемата съответства на миграцията и дефинира Elixir структура.
* Вече е възможно да създадем `%Books.Country{name: "Bulgaria", code: "BG"}` и тази структура би могла да се запази като ред в базата.
* Схемата може да се ползва и за създаване на *changeset*.

---
## Ecto changeset

* Използват се за да трансформират (`cast`) неструктурирани данни (`map`), в нещо структурирано, като ни осигуряват валидни и сигурни данни.
* Могат да се ползват и за промени, като *diff*. Като промяната е от валидни данни до нови, отново валидни данни.
* Данните са винаги консистентни, защото ние сами си описваме правилата за валидация.
* Валидацията води до `changeset`, който е или валиден или не. Ако не е валиден съдържа структурирани грешки във формат, който очакваме.

---
## Ecto changeset

- Пример за `changeset` за схемата `Country`:

```elixir
  def changeset(%__MODULE__{} = country, attrs) do
    country
    |> cast(attrs, [:name, :code])
    |> validate_required([:name, :code])
    |> validate_length(:code, min: 2, max: 2)
    |> validate_length(:name, min: 4, max: 64)
    |> unique_constraint(:code)
    |> unique_constraint(:name)
    |> validate_code()
    |> validate_name()
  end
```

---
## Ecto changeset

* Функциите за валидация идват от `Ecto.Changeset`, можем да си ги `import`-нем.
* Функцията `cast/3` създава `changeset` от позната структура, неструктурирани атрибути и полета, които да извлечем от тези атрибути.
* Само зададените полета попадат в `changeset`-a.

---
## Ecto changeset

* С функцията `validate_required/2` си осигуряваме, че задължителните полета съществуват.
* С функцията `validate_length/3` валидираме дължините на низовете.
* С функцията `unique_constraint/2` си осигуряваме, че вече няма такива стойности в базата данни.
* Можем да си пишем и наша си валидация, използвайки структурата и свойствата на `changeset`.

---
## Ecto changeset

- Пример за специфична валидация:

```elixir
  defp validate_code(%Ecto.Changeset{valid?: false} = changeset), do: changeset

  defp validate_code(changeset) do
    code = get_field(changeset, :code)

    if code != String.upcase(code) do
      add_error(changeset, :code, "Country code has to be in uppercase!")
    else
      changeset
    end
  end
```

---
## Ecto changeset и бази данни

- Невалиден `changeset` не може да се запази в базата, даже няма да има заявка:

```elixir
changeset = Country.changeset(%Country{}, %{name: "Bulgaria", code: "bg"})

{:error, changeset} = Repo.insert(changeset)

changeset.valid?
#=> false

changeset.errors
#=> [code: {"Country code has to be in uppercase!", []}]
```
