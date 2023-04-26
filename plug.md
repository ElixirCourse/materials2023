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
backgroundColor: #FFFFFF
marp: true
---

## Увод в уеб програмирането с Elixir

<img class="center_img" src="assets/plug_chain.png" width="40%" />

---

### Съдържание

1. Low-level протоколи
2. HTTP протокол
3. Plug
4. Демо

---

### Уеб програмиране

* Разработката на клиент и сървър приложения.
* Комуникацията между клиента и сървъра се осъщестява по мрежата.
* Използват се различни протоколи за комуникация - HTTP и Websocket.
* Тези протоколи са само една част от всички протоколи и технологии, чрез които се осъществява комуникацията.
* Част от протоколите, които трябва да познавате: TLS, TCP, UDP, IP и др.
* [OSI модел](https://en.wikipedia.org/wiki/OSI_model)
* [Internet Protocol Suite (TCP/IP) модел](https://en.wikipedia.org/wiki/Internet_protocol_suite)

---

### Web Stack

* TCP/IP
* Sockets
* HTTP
* Plug
* Phoenix
* Phoenix LiveView

---

### IP (Internet Protocol)

* Протокол за изпращане на datagrams (header-payload структури) в мрежата.
* Connectionless протокол. Не гарантира доставянето на пакетите. Не гарантира реда на получаване на пакетите. (Best-effort delivery).
* Проверка за грешки се прави само за хедърите.
* Не разбира от портове.
* Рутирането/маршрутизиране е негово задължение (с помощта на други протоколи.)
  * Изпращате данни към адрес 35.129.109.24 без да се интересувате какъв е маршрутът до този адрес.
* IPv4 (32 бита) и IPv6 (128 бита) версии.
* **Никога** не го имплементирате вие.

---

### IP (Internet Protocol)

* Всеки хост в мрежата има IP адрес.
* Да си свързан с мрежата означава да имаш IP адрес и да можеш да достигаш до други хостове чрез техния IP адрес.
* Единственото нещо, което IP прави, е да доставя datagrams на някой хост.

---

```bash
➜  ~ traceroute -I elixir-lang.bg # Показва някой възможен път до крайната дестинация
traceroute to elixir-lang.bg (165.227.142.119), 64 hops max, 72 byte packets
 1  192.168.100.1 (192.168.100.1)  10.702 ms  3.244 ms  2.937 ms
 2  78-83-64-2.spectrumnet.bg (78.83.64.2)  7.807 ms  4.186 ms  4.599 ms
 3  * * *
 4  * de-cix.spnet.net (80.81.192.229)  30.842 ms  30.122 ms
 5  fra2-edge1.digitalocean.com (80.81.195.151)  32.423 ms  31.958 ms  32.689 ms
 6  * * *
 7  * * *
 8  * * *
 9  * * *
10  themeddle.com (165.227.142.119)  34.735 ms  30.794 ms  30.474 ms
```

---

```bash
➜  ~ traceroute -I elixir-lang.bg # С пуснат VPN. Познайте държавата
traceroute to elixir-lang.bg (165.227.142.119), 64 hops max, 72 byte packets
 1  10.5.0.1 (10.5.0.1)  294.600 ms  301.037 ms  290.308 ms
 2  86.48.12.253 (86.48.12.253)  290.676 ms  290.853 ms  348.011 ms
 3  vl203.tyo-eq8-core-1.cdn77.com (185.229.188.146)  291.204 ms  290.459 ms  291.060 ms
 4  vl250.tyo-eq8-core-2.cdn77.com (138.199.0.217)  298.274 ms  311.895 ms  348.898 ms
 5  63-217-103-9.static.pccwglobal.net (63.217.103.9)  307.997 ms  306.715 ms  308.902 ms
 6  ae-16.a00.tokyjp09.jp.bb.gin.ntt.net (129.250.9.101)  304.970 ms  301.934 ms  307.026 ms
 7  ae-1.r31.tokyjp05.jp.bb.gin.ntt.net (129.250.7.39)  294.160 ms  290.743 ms  290.470 ms
 8  ae-4.r25.lsanca07.us.bb.gin.ntt.net (129.250.3.193)  402.408 ms  394.059 ms *
 9  * * *
10  ae-7.r21.londen12.uk.bb.gin.ntt.net (129.250.2.110)  602.453 ms  511.247 ms  512.267 ms
11  ae-3.a02.londen14.uk.bb.gin.ntt.net (129.250.3.191)  529.022 ms  614.286 ms  513.600 ms
12  ae-0.a03.londen14.uk.bb.gin.ntt.net (129.250.4.251)  617.506 ms  613.340 ms  512.530 ms
13  62.73.179.138 (62.73.179.138)  613.974 ms  517.075 ms  607.136 ms
14  138.197.244.91 (138.197.244.91)  613.742 ms  583.656 ms  614.136 ms
15  * * *
16  * * *
17  * * *
18  * * *
19  themeddle.com (165.227.142.119)  586.333 ms  614.011 ms  614.794 ms
```

---

### Port

* Elixir Port != Networking Port.
* Използва се от протоколите на транспортния слой.
* 16 битово число, което участва в идентификацията на connection endpoint.
  * `(source_ip, source_port, dest_ip, dest_port, protocol)`
* Това позволява един IP адрес да има множество сървъри, всеки от които слуша на различен порт.
* Например: 80 - HTTP, 443 - HTTPS, 22 - SSH, 53 - DNS.

---

### Кой слуша на порт?

- Само един процес може да слуша на даден адрес:порт по едно време.
- Често може да забравите в някой друг таб да се изпълнява някоя програма, която заема порта.
- Ако това се случи, ще видите грешка `:eaddrinuse`

---

### Грешки, ако портът вече се използва

```sh
** (Mix) Could not start application pento: Pento.Application.start(:normal, []) returned an error: shutdown: failed to start child: PentoWeb.Endpoint
    ** (EXIT) shutdown: failed to start child: {:ranch_listener_sup, PentoWeb.Endpoint.HTTP}
        ** (EXIT) shutdown: failed to start child: :ranch_acceptors_sup
            ** (EXIT) {:listen_error, PentoWeb.Endpoint.HTTP, :eaddrinuse}
```
```sh
** (Mix) Could not start application presentem: Presentem.Application.start(:normal, []) returned an error: shutdown: failed to start child: PresentemWeb.Endpoint
    ** (EXIT) shutdown: failed to start child: {:ranch_listener_sup, PresentemWeb.Endpoint.HTTP}
        ** (EXIT) an exception was raised:
            ** (ArgumentError) errors were found at the given arguments:

  * 2nd argument: not a key that exists in the table

                (stdlib 4.3) :ets.lookup_element(:ranch_server, {:addr, PresentemWeb.Endpoint.HTTP}, 2)
```

---

### Кой слуша на порт - решение

```sh
~ lsof -nP -i4TCP:4000 | grep LISTEN
COMMAND    PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
beam.smp 82109 ivan   51u  IPv4 0x767589897efaeaed      0t0  TCP 127.0.0.1:4000 (LISTEN)

# kill 82109 # Убиваме процеса, за да освободим порта

# Други команди с подобен ефект
# lsof -i -P -n | grep LISTEN
# netstat -tulpn | grep LISTEN
# ss -tulpn | grep LISTEN
# lsof -i:22
# nmap -sTU -O IP-address-Here
```

---

### who_listens


```bash
➜ cat ~/.zshrc | rg who_listens -A3
who_listens()
{
  lsof -nP -i4TCP:$1 | rg LISTEN
}

➜ who_listens 4000
beam.smp 7698 ivan   37u  IPv4 0xb09ace71891ae715      0t0  TCP 127.0.0.1:4000 (LISTEN)
```

---

### UDP (User Datagram Protocol)

* Транспортен протокол. (Почти винаги) използва IP за преноса на данните.
* Също като IP не гарантира доставяне на пакетите или техния ред.
* Надгражда с няколко функционалности:
  * `Port` - IP не знае какво е порт.
  * (По желание) Проверка с чексума на данните.
* Полезен е, когато данните са нужни в реално време и е безполезно да се изпращат наново на по-късен етап (Пример: First Person Shooter игри, Video streaming)
* Netflix използват TCP: [тук](https://www.quora.com/Why-does-Netflix-use-TCP-and-not-UDP-for-its-streaming-video) и [тук](https://www.geeksforgeeks.org/why-does-netflix-use-tcp-but-not-udp-for-streaming-video/)
* **Никога** не го имплементирате вие.

---

### TCP (Transmission Control Protocol)

* Транспортен протокол. (Почти винаги) използва IP за преноса на данните.
* Connection-based. Една връзка се определя от четворката `(source ip, source port, destination ip, destination port)`.
* Позволява да се адресират отделни процеси на сървъра.
* Добавя функционалност за:
  * **Connection** - Чрез handshake процес се установява постоянна връзка.
  * **Наредба** - Голямо съобщение може да се разбие в много IP пакети, които могат да бъат получени в разбъркан ред. TCP ги подрежда в правилния ред.
  * **Надежност** - Ако съдържанието на някое съобщение е повредено, TCP се грижи то да бъде изпратено отново. Ако някое съобщение не се получи, то се изпраща отново.
* **Никога** не го имплементирате вие.

---

### DNS  (Domain Name System)

* Не е протокол.
* Дистрибутирана и йерархична система за имена.
* Отговаря на въпроси като "Какъв е IP адресът на google.com?"
* Използва `UDP` и/или `TCP` за комуникация.
* Може да използва допълнителни протоколи, ако има нужда от сигурност.
* Запазен порт 53.

---

```bash
➜  ~ dig elixir-lang.bg
# ...
;; ANSWER SECTION:
elixir-lang.bg.		1527	IN	A	165.227.142.119
# ...

➜  ~ nslookup elixir-lang.bg
\Server:        192.168.100.1
Address:        192.168.100.1#53

Non-authoritative answer:
Name:   elixir-lang.bg
Address: 165.227.142.119
```

---

### Socket

* Крайна точка в двупосочна комуникация.
* Служи като като крайна точка за получаване и изпращане на данни.
* Ако работите с API-та от по-ниско ниво, ще работите директно със socket.
* Ще говорим повече за сокети на лекцията за LiveView. 
  * Сървърът изпраща данни като "пише" в socket и получава данните като "чете" от socket

---

### Low-level networking

```elixir
defmodule KVServer do
  require Logger

  def accept(port) do
    {:ok, socket} =
      :gen_tcp.listen(
        port,
        [:binary, packet: :line, active: false, reuseaddr: true]
      )
    Logger.info("Accepting connections on port #{port}")
    loop_acceptor(socket)
  end

  defp loop_acceptor(socket) do
    {:ok, client} = :gen_tcp.accept(socket)
    serve(client)
    loop_acceptor(socket)
  end

  defp serve(socket) do
    socket
    |> read_line()
    |> write_line(socket)

    serve(socket)
  end

  defp read_line(socket) do
    {:ok, data} = :gen_tcp.recv(socket, 0)
    data
  end

  defp write_line(line, socket) do
    :gen_tcp.send(socket, line)
  end
end
```

---

```elixir
KVServer.accept(4001)
```

```bash
➜ telnet localhost 4001
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Hello
Hello
Bye
Bye
```

---

### Socket Server

* Не е приятно да се пише ръчно accept/listen/bind/send/recv/etc.
* Решение: Socker server/TCP acceptor pool библиотеки
* [Ranch](https://github.com/ninenines/ranch)
* [Thousand Island](https://github.com/mtrudel/thousand_island)

---

```elixir
Mix.install [:thousand_island]

defmodule Echo do
  use ThousandIsland.Handler

  @impl ThousandIsland.Handler
  def handle_data(data, socket, state) do
    ThousandIsland.Socket.send(socket, data)
    {:continue, state}
  end
end

{:ok, pid} = ThousandIsland.start_link(port: 1234, handler_module: Echo)
```
---

### HTTP (Hypertext Transfer Protocol)

* Фундаментален за WWW протокол.
* Текстов протокол - headers/payload в четим за човек вид.
* Главната му употреба е за заявки и сервиране на THML уеб страници или
  уеб ресурси в друг формат - JSON, XML, CSS, JS и т.н.

---

<img src="assets/http_request_response.png" width="80%">

---

### HTTP Методи

* `GET` - Чете (read-only) ресурси от сървъра.
* `POST` - Изпраща данни към сървъра, за да създаде/обнови някой ресурс.
* `HEAD`, `PUT`, `DELETE` и други, които сега не ни интересуват.

---

### Status Code

* 200 - OK
* 301 - Moved Permanently
* 401 - Unauthorized
* 403 - Forbidden
* 404 - Not found
* 500 - Internal server error
* 418 - I'm a teapot ([RFC2324](https://www.rfc-editor.org/rfc/rfc2324), [RFC7168](https://www.rfc-editor.org/rfc/rfc7168))

---

### HTTP Request/Response

* Клиентът изпраща заявка към сървъра. Сървърът връща уеб ресурс.
  * Заявка за CSS файлове -> Сървърът връща CSS файлове.
  * Заявка за JS файлове -> Сървърът връща JS файлове.
  * Заявка за JPG файл -> Сървърът връща JPG файл.
  * Заявка за данни -> Сървърът връща данни (HTML, JSON, etc.).
  * Клиентът използва всичките ресурси, за да визуализира уеб страницата.

---

### Полезни конзолни инструменти

* `curl` - Инструмент за трансфер на данни, използвайки различни протоколи
  * Най-често се използва за изпращане на HTTP заявки
* `jq` - `sed` за JSON данни. [Линк към команди](https://www.baeldung.com/linux/jq-command-json)
  * Най-често се използва за pretty-prining на JSON резултати от curl. Може да филтрира/селектира JSON данни.
* `pbcopy` и `pbpaste` - Запазване на резултат в clipboard. 
  * Използва се, за да не правите една заявка много пъти, ако искате да обработвате резултата по някакъв начин в терминала.

---

```
➜ curl 'https://api.openweathermap.org/data/2.5/weather?q=Sofia&appid=186ad7469ef56057ab9ef8522b8efda3' | pbcopy
➜ pbpaste | jq '.main'
{
  "temp": 282.98,
  "feels_like": 281.41,
  "temp_min": 282.98,
  "temp_max": 282.98,
  "pressure": 1014,
  "humidity": 66
}
➜ pbpaste | jq '.main.temp'
282.98
➜ pbpaste | jq keys
[
  "base", "clouds", "cod", "coord", "dt", "id", "main",
  "name", "sys", "timezone", "visibility", "weather", "wind"
]
```

---


### Ситуации, неподходящи за request/response

* Когато сървърът иска да изпраща съобщения на клиента без клиентът да е направил заявка.
* Нотификации, чат, Publish/subscribe системи, високо интерактивни системи и др.
* Възможни решения:
  * Клиентът също така да е сървър.
  * `Long Polling` - Сървърът не отговаря и не затваря връзката, а чака докато данните са готови. Когато клиентът получи данни, веднага изпраща заявка, която чака. Трябва reconnect след всяко съобщение.
  * `Keep Alive` - Изпращане на много заявки/отговори в една TCP връзка.

---

### WebSocket

* Не е Socket.
* Протокол, предоставящ full-duplex комуникация над TCP.
* Клиентът и сървърът могат да си изпращат съобщения взаимно в една връзка.
* Използва портовете на HTTP(S): 80 и 443.
* За изграждане на връзка се използва HTTP Upgrade хедър, за да се смени комуникацията от HTTP към WebSocket.

---

### URL и URI

* `URI` - Universal Resource Identifier. Суперсет на `URN` и `URL`
* `URL` - Universal Resource Locator.
* `URN` - Universal Resource Name. Пример: `mailto:john.doe@example.com`
* Синтаксис: `scheme:[//authority]path[?query][#fragment]`
  * Пример: `https://example.org/test/test1?search=test-question#part2`
* Модулът в Elixir е `URI`.
* Често URL и URI се използват взаимнозаменяемо в блогпостове, книги, лекции и т.н.

---

### Какво не е Plug

* Plug не е HTTP сървър. 
  * Plug използва съществуващи HTTP сървъри (Cowboy, Bandit) чрез адаптери.
* Plug **не** е MVC/MVP/MVVM фреймуърк.
* Plug е централна част във Phoenix фреймуърка
* Plug **не** е "по-малко" функционалният начин за изграждане на уеб приложения, който бива замествен от по-мощен фреймуърк.

---

### Plug

* Спецификация за изграждане на уеб приложения чрез композиция на функции.
* Или чрез модули, които имплементират две функции: `init/1` и `call/2`.
* Всяка функция извършва една малка трансформация.
* Pipeline от много функции = обработка на цялата заявка.
* `Plug` предоставя готови функции и модули за голяма част от основните функции.

---

```elixir
Mix.install([:plug, :plug_cowboy])

defmodule MyPlug do
  import Plug.Conn

  def init(options), do: options

  def call(conn, _opts) do
    conn
    |> put_resp_content_type("text/plain")
    |> send_resp(200, "Hello world")
  end
end

webserver = {Plug.Cowboy, plug: MyPlug, scheme: :http, options: [port: 4000]}
{:ok, _} = Supervisor.start_link([webserver], strategy: :one_for_one)
```

---

```elixir
Mix.install([:plug, :bandit, :websock_adapter])

defmodule Router do
  use Plug.Router

  plug Plug.Logger
  plug :match
  plug :dispatch

  get "/", do: send_resp(conn, 200, "Hello")

  match _, do: send_resp(conn, 404, "Not Found")
end

webserver = {Bandit, plug: Router, scheme: :http, options: [port: 4001]}
{:ok, _} = Supervisor.start_link([webserver], strategy: :one_for_one)
```

---

### CRC: Constructors, Reducers, Converters

```elixir
defmodule Number do
  def new(string), do: Integer.parse(string) |> elem(0)
  def add(number, addend), do: number + addend
  def to_string(number), do: Integer.to_string(number)
end
```
---

### CRC: Constructors, Reducers, Converters

* Много добре описва потока на работа във функционалните езици.
* Много добре описва и изпълнението на една уеб заявка
  * 1. Заявката се приема и се създава подходяща структура, която я описва.
  * 2. Изпълняват се поредица от трансформации върху структурата:
    * 2.1. Извличат се детайли за автентикация и потребителят се добавя в структурата;
    * 2.2. Пресмята се отговорът и се добавя в структурата;
    * 2.n. ...други;
  * 3. Структурата се трансформира до JSON/HTML/etc. отговор и се изпраща на клиента.

---

### CRC: Constructors, Reducers, Converters

* В Elixir много от модулите са асоциирани с определен тип (core type)
  * `String` за низове, `Enum` за Enumerables, `Integer` за цели числа и т.н.
* Тези модули съдържат:
  * Функции за създаване на инстанция от този тип от някакъв вход.
    * `Integer.parse/1` създава число от стринг 
  * Функции, които приемат този тип и връщат този тип:
    *  `Integer.pow/2`, `Integer.floor_div/2`, `Enum.map/2`
  * 3. Функции, които превръщат този тип до друг тип:
    * `Integer.to_string/1`, `String.to_integer/1`

---

### Plug.Conn

* Структура, която напълно описва една заявка.
* Включва данните за самата заявка (Request)
* По време на изпълнение, към структурата се добавят данните за отговора (Response)
* В края структурата се превръща до HTML/JSON/XML/etc. формат и се изпраща към клиента.
* [Създава се автоматично](https://github.com/elixir-plug/plug_cowboy/blob/1796e77aa42c2284f3667c7aeac7074236f47c07/lib/plug/cowboy/handler.ex#L7), т.е. ние не се грижим за конструирането ѝ.

---

```elixir
%Plug.Conn{
  adapter: {Plug.MissingAdapter, :...},
  assigns: %{},
  body_params: %Plug.Conn.Unfetched{aspect: :body_params},
  cookies: %Plug.Conn.Unfetched{aspect: :cookies},
  halted: false,
  host: "www.example.com",
  method: "GET",
  owner: nil,
  params: %Plug.Conn.Unfetched{aspect: :params},
  path_info: [],
  path_params: %{},
  port: 0,
  private: %{},
  query_params: %Plug.Conn.Unfetched{aspect: :query_params},
  query_string: "",
  remote_ip: nil,
  req_cookies: %Plug.Conn.Unfetched{aspect: :cookies},
  req_headers: [],
  request_path: "",
  resp_body: nil,
  resp_cookies: %{},
  resp_headers: [{"cache-control", "max-age=0, private, must-revalidate"}],
  scheme: :http,
  script_name: [],
  secret_key_base: nil,
  state: :unset,
  status: nil
}
```

---

### Plug функции

* Всяка функция, която приема `%Plug.Conn{}` и Keyword списък и връща `%Plug.Conn{}`
* Добавят се във веригата чрез:
  *  `plug :match` (без opts аргумент)
  *  `plug :put_root_layout, {PentoWeb.Layouts, :root}`

---

```elixir
plug :basic_auth, username: "admin", password: "admin"

import Plug.Conn
def auth(conn, options \\ []) do
  with ["Apikey " <> apikey] <- get_req_header(conn, "authorization"),
       true <- valid_apikey?(apikey) do
       conn
    else
    _ -> conn |> halt()
  end
end

defp valid_apikey?(_), do: true
```

---

### Plug модули

* Вместо функция, `plug` макрото приема и модули.
* Plug модулът:
  * `import Plug.Conn` за достъп до функции за обработка на `conn` структурите
  * Дефинира функциите `init/1` и `call/2`
* Резултатът от `init/1` се подава като втори аргумент на `call/2`
* `init/1` *може* да се извика по време на компилация, затова не трябва да връща pid, ref или други runtime структури.
  * Ако искаме да се извиква всеки път runtime: `config :phoenix, plug_init_mode: :runtime`

---

```elixir
defmodule MyBasicAuth do
  import Plug.Conn

  def init(opts), do: opts

  def basic_auth(conn, _options \\ []) do
    with ["Apikey " <> apikey] <- get_req_header(conn, "authorization"),
         true <- valid_apikey?(apikey) do
         conn
      else
      _ -> conn |> request_basic_auth(options) |> halt()
    end 
  end
end
```

---

### `plug/1` и `plug/2`

- Макро, което се използва за изграждане на pipeline (converters)

```elixir
plug Plug.MethodOverride
plug Plug.Head
plug Plug.Session, @session_options
plug PentoWeb.Router
plug Plug.Logger
plug :match
plug :dispatch

# Ако не използваме метапрогрмиране, то горното е почти еквивалентно на:
conn
|> Plug.MethodOverride.call()
|> Plug.Head.call()
|> Plug.Session.call(@session_options)
|> PentoWeb.Router.call()
|> match()
|> dispatch()
```

---

### Грешки и Early send

* `plug/{1,2}` добавя механизми за обработка на грешки (`Plug.ErrorHandler`, `Plug.Exception`)
* Всеки `plug` може да извика `halt` и да изпрати резултат
* Ако някой `plug` направи това, останалите след него не се изпълняват.
* Пример: Ако `AuthenticataionPlug` установи, че паролата е сгрешена, то не трябва да продължаваме нататък.

---

### Plug.Router

* `Router` е стандартен термин в уеб програмирането.
* Рутерът определя коя част от кода ще обработи една заявка.
* Рутирането става по `path`(!!!) частта от URL.
  * `query` получаваме като параметър на функцията, която ще обработи заявката.
  * `fragments` не е достъпно на сървъра според [RFC2396](https://www.rfc-editor.org/rfc/rfc2396#section-4.1)

---

### Plug.Router - имена в пътя

```elixir
Mix.install([:plug, :bandit, :websock_adapter])

defmodule Router do
  use Plug.Router

  plug Plug.Logger
  plug :match
  plug :dispatch

  get "/hello", do: send_resp(conn, 200, "hello anonymous")
  get "/hello/:name", do: send_resp(conn, 200, "hello #{name}")
  match _, do: send_resp(conn, 404, "not found")
end

require Logger
webserver = {Bandit, plug: Router, scheme: :http, options: [port: 4000]}
{:ok, _} = Supervisor.start_link([webserver], strategy: :one_for_one)
Logger.info("Plug now running on localhost:4000")
```

---

### Phoenix.Router

* `Phoenix` библиотеката надгражда функционалността на `Plug.Router`.
* Неща, които ни предоставя:
  * `pipeline` - групиране и именуване на група от `plug`.
  * `scope` - по-лесно изграждане на йерархична структура от пътища
* Ще я разгледаме на лекцията за Phoenix

---

### Plug.Session

* Сесията е начин да съхраним данни, които трябва да живеят по-дълго от живота на една заявка.
  * Пример: Не искаме потребителят да дава име и парола всеки път.
  * Запазваме дали потребителят използва light theme или dark theme.
* Запазваме данните на сървъра и даваме ключа към тях на клиента.
* Записваме всички данни в cookie и го даваме на клиента.
  * Данните се пазят в криптиран вид и не могат да бъдат прочетени в браузъра.


---

### Compile-time VS Runtime оценяване

* Важно е да знаем кога нещо се изпълнява по време на компилация и кога по време на изпълнение.
* Ако някои от ресурсите не са налично по време на компилация, то трябва да ги "прочетем" по време на изпълнение.
* Рутерът се компилира. 
* Изнасяме runtime логиката във функции.

---

### Compile-time VS Runtime пример

* **Никога** не трябва да публикувате secrets в хранилището с код.
* Стандартна практика е да четете данни от environment variables.
* Друга част от инфраструктурата ще се грижи за съхраняването на тайните, например Kubernetes Vault, git-crypt и др..
* Когато deploy-ва някое изображение, тези променливи ще бъдат добавени в средата. 

---

### Тестове

```elixir
defmodule MyRouter do
  use Plug.Router

  plug :match
  plug :dispatch

  get "/hello" do
    send_resp(conn, 200, "world")
  end

  forward "/users", to: UsersRouter

  match _ do
    send_resp(conn, 404, "oops")
  end
end
```

---

```elixir
defmodule MyPlugTest do
  use ExUnit.Case, async: true
  use Plug.Test

  @opts MyRouter.init([])

  test "returns hello world" do
    # Create a test connection
    conn = conn(:get, "/hello")

    # Invoke the plug
    conn = MyRouter.call(conn, @opts)

    # Assert the response and status
    assert conn.state == :sent
    assert conn.status == 200
    assert conn.resp_body == "world"
  end
end
```

---

```elixir
defmodule MyRouterTest do
  use ExUnit.Case, async: true
  use Plug.Test

  @opts MyRouter.init([])

  test "returns hello text" do
    response = conn(:get, "/") |> MyRouter.call([])
    assert response.body == "hello"
    assert response.status == 200
  end
end
```

---

### Демо

* [Plug.BasicAuth](https://hexdocs.pm/plug/Plug.BasicAuth.html)
* [PlugAttack](https://hexdocs.pm/plug_attack/PlugAttack.html)

---

### Ресурси
