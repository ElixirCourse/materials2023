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

* RM (relational mapper) има за цел да *преобразува* структурирани данни между системи, които имат доста различни структури от данни.
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
- Командата `ecto.gen.repo` ще ни каже какво да променим:

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

- `Repo.all` също така поддържа всичко, нужно на `SELECТ`, като `group_by`, `limit`, `join` и други:

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
### Ecto changeset

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
### Ecto changeset

* Функциите за валидация идват от `Ecto.Changeset`, можем да си ги `import`-нем.
* Функцията `cast/3` създава `changeset` от позната структура, неструктурирани атрибути и полета, които да извлечем от тези атрибути.
* Само зададените полета попадат в `changeset`-a.

---
### Ecto changeset

* С функцията `validate_required/2` си осигуряваме, че задължителните полета съществуват.
* С функцията `validate_length/3` валидираме дължините на низовете.
* С функцията `unique_constraint/2` си осигуряваме, че вече няма такива стойности в базата данни.
* Можем да си пишем и наша си валидация, използвайки структурата и свойствата на `changeset`.

---
### Ecto changeset

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
### Ecto changeset и бази данни

- Невалиден `changeset` не може да се запази в базата, даже няма да има заявка:

```elixir
changeset = Country.changeset(%Country{}, %{name: "Bulgaria", code: "bg"})

{:error, changeset} = Repo.insert(changeset)

changeset.valid?
#=> false

changeset.errors
#=> [code: {"Country code has to be in uppercase!", []}]
```

---
## Ecto асоциации

* В релационните бази от данни има релации между таблиците.
* Ecto, в ролята си на *mapper* поддържа представяне на тези релации като асоциации.
* Схемите, които са реферирани имат `has_one` или `has_many` асоциации.
* Схемите, които реферират използват `belongs_to` за да изразят това.

---
## Ecto асоциации

- Нека имаме миграцията:

```elixir
  def change do
    create table(:authors) do
      add :first_name, :string, size: 32, null: false
      add :last_name,  :string, size: 32, null: false
      add :birth_date, :date
      add :country_id, references(:countries), null: false

      timestamps()
    end

    create index(:authors, [:first_name, :last_name])
  end
```

---
### Ecto асоциации

* Декларираме, че таблицата `authors` има *FK* (foreign key) `country_id`.
* `references` приема таблицата, която се реферира и опции като какво да стане при триене или промяна на зависимостта.

---
### Ecto асоциации

- Схемата `Author` представлява:

```elixir
  schema "authors" do
    field(:first_name, :string)
    field(:last_name, :string)
    field(:birth_date, :date)

    field(:name, :string, virtual: true)

    belongs_to(:country, Books.Country)

    timestamps()
  end
```

---
### Ecto асоциации : *belongs_to*

* С `belongs_to` изразяваме, че таблицата на схемата реферира друга таблица с *FK*.
* Има множоство от опции. Пример е, че ако реферираната схема се казва `Country`, то автоматично се ползва `country_id`.
* Ако *FK* се казваше `the_country_id` нямаше да стане. Бихме дефинирари `belongs_to(:country, Books.Country, foreign_key: :the_country_id)`.
* Вижте документацията за всички опции.

---
### Ecto асоциации

- Схемата `Country` се променя на:

```elixir
  schema "countries" do
    field(:name, :string)
    field(:code, :string)

    has_many :authors, Books.Author

    timestamps()
  end
```

---
### Ecto асоциации : *has_many*/*has_one*

* С `has_many`/`has_one` изразяваме, че таблицата на схемата е реферирана от друга таблица с *FK*.
* Поддържа подобни опции на *belongs_to*, филтри, име на *FK* и други.
* Можем да изразим *many-to-many* връзка с междинна таблица, като използваме `through: [<междинна-асоциация>, <асоциация>]` от двете страни на релацията.

---
### Ecto асоциации : *many_to_many*

- За many_to_many релацията има по добър начин за изразяване:

```elixir
  def change do
    create table(:authors_books) do
      add :author_id, references(:authors), null: false
      add :book_id, references(:books), null: false
    end
  end

# Books.Book:
  many_to_many :authors, Author, join_through: "authors_books"

# Books.Author:
  many_to_many :books, Book, join_through: "authors_books"
```

---
### Ecto асоциации и changesets

- Можем просто да задаваме id-тата на асоцияциите:

```elixir
  def changeset(%__MODULE__{} = author, %{country_id: _} = attrs) do
    author
    |> cast(attrs, [:first_name, :last_name, :birth_date, :country_id])
    |> validate_required([:first_name, :last_name, :country_id])
    |> foreign_key_constraint(:country_id)
    |> validate()
  end

  defp validate(changeset) do
    changeset
    |> validate_length(:first_name, min: 1, max: 32)
    |> validate_length(:last_name, min: 1, max: 32)
  end
```

---
### Ecto асоциации и changesets

- Можем и да задаваме целите асоциации:

```elixir
  def changeset(%__MODULE__{} = author, %{country: country} = attrs) do
    author
    |> cast(attrs, [:first_name, :last_name, :birth_date])
    |> validate_required([:first_name, :last_name])
    |> check_country(country)
    |> validate()
  end
```

---
### Ecto асоциации и changesets

- Важното е да проверим дали асоциацията вече съществува ако има *unique constraint*:

```elixir
  defp check_country(%{valid?: false} = changeset, _), do: changeset

  defp check_country(changeset, country) do
    country_changeset = Country.changeset(%Country{}, country)

    if country_changeset.valid? do
      put_country(Repo.get_by(Country, name: get_field(country_changeset, :name)), changeset)
    else
      changeset
    end
  end
```

---
### Ecto асоциации и changesets

- След, което да я запишем или само маркираме:

```elixir
  defp put_country(nil, changeset) do
    cast_assoc(changeset, :country, required: true, with: &Books.Country.changeset/2)
  end

  defp put_country(country, changeset) do
    put_assoc(changeset, :country, country)
  end
```

---
### Ecto асоциации и changesets

* При `many_to_many` подобно нещо става още по-сложно.
* Затова записване на асоциации по id-та винаги е най-лесният и по-SQL вариант.
* Когато *relational mapping*-ът започне да създава твърде много работа, просто опростете нещата, като използвате по-близък до базата код.

---
### Ecto асоциации и заявки

- По подразбиране асоциацията не се зарежда при `SELECT`:

```elixir
%Books.Author{
  id: 68,
  first_name: "Иван",
  last_name: "Александров",
  birth_date: ~D[1996-05-13],
  name: nil,
  country_id: 66,
  country: #Ecto.Association.NotLoaded<association :country is not loaded>,
  books: #Ecto.Association.NotLoaded<association :books is not loaded>,
  ...
}
```

---
### Ecto асоциации и заявки

- Можем да заредим асоциациите си с `preload`:

```elixir
  author = Books.Repo.preload(author, :country)

  %Country{name: "Bulgaria", code: "BG", id: ^country_id} = author.country
```

---
### Ecto асоциации и заявки

- Можем да задаваме preload и в заявки, елиминирайки *N+1* проблема:

```elixir
  def by_country_name(country_name) do
    query = from(
      a in __MODULE__,
      join: c in Country,
      on: a.country_id == c.id,
      where: c.name == ^country_name,
      preload: [:country]
    )

    query = from(a in query, select_merge: %{name: fragment("? || ' ' || ?", a.first_name, a.last_name)})

    Repo.all(query)
  end
```

---
### Повече за Ecto заявки

* Можем динамично да си строим завики като строим заявка от заявка `from q in query`.
* Можем да го правим и с `Ecto.query.where`, `Ecto.query.group_by` и тн.
* Можем да си дефинираме виртуални полета и да ги запълваме с `select_merge`.
* Когато искаме да направим нещо по специфично за базата, за което нямаме изразно средство в Ecto, можем да ползваме `fragment`.


---
## Ecto changeset без schema

* Можем да използваме `changeset` без схема.
* Така можем да си напишем валидация за JSON или за каквито и да е map-ове.
* Можем и да ползваме `changeset` със `schema`, но без база данни и таблица, да речем за валидация на по-структурирани данни без модел в базата.

---
## Ecto changeset без schema

```elixir
data = %{}
types = %{name: :string, email: :string}

changeset =
  {data, types}
  |> Ecto.Changeset.cast(params["sign_up"], Map.keys(types))
  |> validate_required(...)
  |> validate_length(...)

```

---
## Ecto changeset с embedded_schema

- Със схема, но без таблица свързана с нея:

```elixir
defmodule SignUp do
  @mail_regex ~r/^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,4}$/

  embedded_schema do
    field(:name, :string)
    field(:email, :string)
  end

  def changeset(sign_up, attrs \\ %{}) do
    sign_up
    |> cast(attrs, [:name, :email])
    |> validate_required([:name, :email])
    |> validate_length(:name, min: 4, max: 64)
    |> validate_format(field, @mail_regex)
  end
end
```

---
## Ecto транзакции

* Ecto поддържа работа с транзакции по няколко начина.
* Най-простият начин е просто код, съдържащ заявки да е подаден като функция, която да се изпълни в транзакция.
* Поддържа се и динамично построяване на транзакции (`Ecto.Multi`).
* Чрез `Ecto.Multi` можем да си строим и *batch* insert-и към базата данни.

---
## Ecto транзакции

- Примерна транзакция за запис на книга:

```elixir
  {:ok, book_id} =
    Books.Repo.transaction(fn ->
      {:ok, %{id: country_id}} = Books.insert_or_get_country("USA", "US")

      {:ok, %{id: author_id}} =
        Books.insert_or_get_author(
          "Дан",
          "Симънс",
          Date.from_iso8601!("1948-04-04"),
          country_id
        )

      book_id = Books.insert_book!("978-619-152-344-3", "Ужас", @desc, 2013, "български")
      :ok = Books.add_authors_to_book(book_id, [author_id])

      book_id
    end)
```

---
### Ecto транзакции

* Ако някъде из кода има грешка (`raise`) транзакцията ще се *rollback*-не.
* Ако извикаме `Book.Repo.rollback(:some_error)`, транзакцията ще се *rollback*-не.
* Ако няма грешки транзакцията ще мине и ще върне като резултат резултатът от функцията.
* Ако транзакцията не мине, ще върне `{:error, <some_error>}`.

---
### Ecto транзакции

```elixir
  {:ok, %{book_id: book_id}} =
    Multi.new()
    |> Multi.run(:country, fn _repo, _current_state ->
      {:ok, %Books.Country{}} = Books.insert_or_get_country("USA", "US")
    end)
    |> Multi.run(:author, fn _repo, %{country: %{id: country_id}} ->
      {:ok, %{id: author_id}} =
        Books.insert_or_get_author(
          "Дан",
          "Симънс",
          Date.from_iso8601!("1948-04-04"),
          country_id
        )
    end)
    |> Multi.run(:book_id, fn _repo, _current_state ->
      {:ok, Books.insert_book!("978-619-152-344-3", "Ужас", @desc, 2013, "български")}
    end)
    |> Multi.run(:authors_books, fn _repo, %{book_id: book_id, author: %{id: author_id}} ->
      case Books.add_authors_to_book(book_id, [author_id]) do
        :ok ->
          {:ok, nil}

        error ->
          error
      end
    end)
    |> Books.Repo.transaction()
```

---
### Ecto транзакции

* Ако някъде из кода има грешка (`raise`) транзакцията ще се *rollback*-не.
* Ако извикаме `Book.Repo.rollback(:some_error)`, транзакцията ще се *rollback*-не.
* Ако някоя стъпка върне `{:error, <some_error>}`,  транзакцията ще се *rollback*-не.
* Ако няма грешки транзакцията ще мине и ще върне като резултат map в който за името на всяка стъпка имаме резултата ѝ.
* Ако транзакцията не мине, ще върне `{:error, <стъпка>, changeset, current_state}`.

---
### Ecto транзакции

* `Ecto.Multi` има няколко функции, да речем за insert и update, но `run` е най-изразителната.
* Идеята е декларативно и динамично да си построим транзакцията.
* Трябва да се внимава да не се разхвърля кода из много функции и да стане труден за проследяване и четене.
* Може да се ползва с `Enum.reduce` за построяване на `batch insert` в транзакция например.

---
### Ecto транзакции

- Пример за `Ecto.Multi` и операции в транзакция, базирани на списък:

```elixir
  def add_authors_to_book(book_id, author_ids) do
    steps =
      Enum.reduce(author_ids, Ecto.Multi.new(), fn author_id, steps ->
        name = "step_#{book_id}_#{author_id}"
        Ecto.Multi.insert(steps, name, %AuthorBook{book_id: book_id, author_id: author_id})
      end)

    case Repo.transaction(steps) do
      {:ok, _} ->
        :ok

      {:error, _, changeset, _changes_so_far} ->
        {:error, changeset}
    end
  end
```

---
## Тестване с Ecto

* За да тестваме код свързан с база от данни трябва да си осигуриме празна база данни за всеки тест.
* Така няма да има изненади в резултатите, които очакваме.
* Ecto идва със специален *sandbox* за тестване, който ни осигурява това.

---
## Тестване с Ecto

- Първо се нуждаем от конфигурация за тестване (*config/test.exs*).
- Важно е да сложим за *pool* `Ecto.Adapters.SQL.Sandbox`, така базата ще бъде чистена на всеки тест:

```elixir
import Config

config :books, Books.Repo,
  pool: Ecto.Adapters.SQL.Sandbox,
  database: "books_test",
  username: "booker",
  password: "bookerpass",
  hostname: "localhost"
```

---
### Тестване с Ecto

- В *test/test_helper.exs* дефинираме mode-ът на `Sandbox`:

```elixir
ExUnit.start()

Ecto.Adapters.SQL.Sandbox.mode(Books.Repo, :manual)
```

---
### Тестване с Ecto

* С `:manual` mode, ще трябва преди всеки тест да си *checkout*-не connection към базата ръчно.
* Един такъв *connection* винаги се изпълнява в траназакция, която след теста се *rollback*-ва.
* Поддържат се `{:shared, <pid>>}` и `:auto` mode-ове. Ако искаме няколко процеса да ползват същия connection ползваме `shared` например.

---
### Тестване с Ecto

- Сега можем да си напишем `Books.RepoCase`:

```elixir
# In test/support/repo_case.ex

defmodule Books.RepoCase do
  use ExUnit.CaseTemplate

  using do
    quote do
      alias Books.Repo

      import Ecto
      import Ecto.Query
      import Books.RepoCase
    end
  end

  setup tags do
    :ok = Ecto.Adapters.SQL.Sandbox.checkout(Books.Repo)

    unless tags[:async] do
      Ecto.Adapters.SQL.Sandbox.mode(Books.Repo, {:shared, self()})
    end

    :ok
  end
end
```

---
### Тестване с Ecto

* Така ако сме в `:async` mode всеки тест ще си има собствен connection, със собствен *DB transaction*.
* Това позволява да имаме конкурентни тестове към базата.
* Postgrex driver-ът позволява това, други driver-и не го позволяват и затова трябва да се провери за даден driver дали се поддържа.
* Ако `:async` е `false`, сме в `:shared` mode, което значи, че всеки процес, който текущият тест пусне ще ползва същия connection.
* Повече прочетете [тук](https://hexdocs.pm/ecto_sql/Ecto.Adapters.SQL.Sandbox.html)

---
### Тестване с Ecto

- За да се компилира `RepoCase` ще трябва да променим *mix.exs*:

```elixir
  def project do
    [
      app: :books,
      version: "0.1.0",
      elixir: "~> 1.14",
      start_permanent: Mix.env() == :prod,
      elixirc_paths: elixirc_paths(Mix.env()),
      aliases: aliases(),
      deps: deps()
    ]
  end
```

---
### Тестване с Ecto

- За да се компилира `RepoCase` ще трябва да променим *mix.exs*:

```elixir
  defp aliases do
    [
      test: ["ecto.create --quiet", "ecto.migrate", "test"]
    ]
  end

  defp elixirc_paths(:test), do: ["lib", "test/support"]
  defp elixirc_paths(_), do: ["lib"]
```

---
### Тестване с Ecto

- Сега можем да си дефинираме DB тестове:

```elixir
defmodule BooksTest do
  use Books.RepoCase

  test "schemaless queries" do
    insert_test_books!()

    books = Books.books_by_year(2013)
    assert books == []

    #...
  end
end
```

---
## Advanced Ecto

- Работа с *JSONB* и blob колони.
- Възможността да си дефинираме наш Ecto тип.
- Фрагменти и специфични заявки към базата данни.
- Да разгледаме малко код!

---
## Край

- Можете да пробвате и други адаптери [Etso : Ecto+ETS](https://github.com/evadne/etso) е интересен.
- Можете да имате multi-tenant система с [префикси по схема](https://hexdocs.pm/ecto/multi-tenancy-with-query-prefixes.html).
- Можете да имплементирате multi-tenancy и с [FK](https://hexdocs.pm/ecto/multi-tenancy-with-foreign-keys.html).
- Има още много интересни теми и функционалности край Ecto, DBConnection, драйверите. Има и [книга](https://pragprog.com/titles/wmecto/programming-ecto).
