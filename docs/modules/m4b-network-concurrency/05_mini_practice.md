# 05 · Мини-практика руками

**Мост:** четыре главы построили картину от socket FD до async task. Теперь
нужно увидеть два края этой картины живьём: байты HTTP/TCP на локальном
порту и переключение задач во время ожидания. Здесь нет приложения Harbor,
фреймворка и готового репозитория. После двух маленьких программ материал
сойдётся в домашке.

**Вопрос сюжета:** можете ли вы набрать минимальный сетевой пример и заранее
объяснить порядок вывода async-программы?

---

## Правила практики

1. Создавайте файлы в любом временном учебном каталоге обычным редактором.
   Не инициализируйте проект, пакет, CI, линтер или `pyproject.toml`.
2. Набирайте код руками. После каждого блока останавливайтесь и
   проговаривайте, что сейчас является процессом, FD, ожиданием и
   результатом.
3. Используйте только стандартную библиотеку Python.
4. Сетевые серверы привязаны к `127.0.0.1`, то есть доступны только на
   локальной машине. Не меняйте адрес на `0.0.0.0`, если не понимаете,
   зачем открывать порт другим узлам.
5. Перед запуском предскажите вход, выход и порядок строк по циклу M0.5.

Маршрут:

```text
выберите один сетевой опыт
   ├── A. один HTTP-запрос через http.server
   └── B. один TCP echo через socket
                    │
                    ▼
обязательный опыт C. два ожидания через asyncio.gather
                    │
                    ▼
изменить пример → сначала прогноз → затем запуск
```

Если сеть пока совсем новая, берите маршрут A. Маршрут B показывает больше
механики TCP, но требует два файла.

---

## Опыт A · Один HTTP-запрос

Создайте вручную файл `tiny_http.py`:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer


class RoomsHandler(BaseHTTPRequestHandler):
    def do_GET(self) -> None:
        if self.path == "/rooms":
            status = 200
            body = b'["Atlas", "Bay"]\n'
            content_type = "application/json; charset=utf-8"
        else:
            status = 404
            body = b"not found\n"
            content_type = "text/plain; charset=utf-8"

        self.send_response(status)
        self.send_header("Content-Type", content_type)
        self.send_header("Content-Length", str(len(body)))
        self.end_headers()
        self.wfile.write(body)


with HTTPServer(("127.0.0.1", 8000), RoomsHandler) as server:
    print("listening on http://127.0.0.1:8000")
    server.handle_request()
    print("served one request, stop")
```

Что здесь важно:

- `HTTPServer` создаёт listening socket и привязывает его к локальному
  порту `8000`;
- `handle_request()` блокирует главный поток до одного запроса;
- библиотека сама делает `accept()`, читает TCP byte stream и разбирает
  HTTP;
- метод `do_GET` получает уже разобранный путь;
- `wfile` — бинарный поток ответа, поэтому тело записано как `bytes`;
- `Content-Length` считается в байтах;
- после одного запроса программа завершается намеренно.

### Запуск

В первом терминале:

```bash
python tiny_http.py
```

Команда запускает серверный процесс. Он напечатает адрес и остановится в
блокирующем ожидании запроса:

```text
listening on http://127.0.0.1:8000
```

Во втором терминале:

```bash
curl -i http://127.0.0.1:8000/rooms
```

`curl` играет роль клиентского процесса, а `-i` просит показать HTTP-статус
и заголовки вместе с телом. Существенная часть ответа:

```http
HTTP/1.0 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 17

["Atlas", "Bay"]
```

`Server` и `Date` могут отличаться — их добавляет стандартная библиотека.
В первом терминале появится строка доступа от `BaseHTTPRequestHandler`, а
затем:

```text
served one request, stop
```

Запустите сервер снова и запросите неизвестный путь:

```bash
curl -i http://127.0.0.1:8000/missing
```

Перед запуском предскажите статус, `Content-Type`, длину тела и текст.

### Что скрыла библиотека

`http.server` закрывает механическую часть:

```text
socket → bind → listen → accept → recv до границы HTTP → parse → do_GET
```

Она не придумывает смысл `/rooms`, статус и тело — это написал разработчик.
И она не является production-сервером: пример обслуживает один запрос,
не задаёт таймауты, не ограничивает тело, не решает TLS, нагрузку и
безопасность. Здесь библиотека нужна как лупа на HTTP, а не как фундамент
Harbor.

---

## Опыт B · Один TCP echo

Этот маршрут показывает socket API без HTTP. Протокол будет предельно
маленьким: клиент отправляет произвольные байты и закрывает **направление
записи**; сервер читает до EOF и возвращает те же байты с префиксом
`echo: `.

Создайте `echo_server.py`:

```python
import socket


with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind(("127.0.0.1", 9000))
    server.listen()
    print("listening on 127.0.0.1:9000")

    connection, peer = server.accept()
    with connection:
        print("connected", peer)
        chunks: list[bytes] = []

        while chunk := connection.recv(1024):
            chunks.append(chunk)

        request = b"".join(chunks)
        connection.sendall(b"echo: " + request)

print("served one connection, stop")
```

Создайте `echo_client.py`:

```python
import socket


with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client:
    client.connect(("127.0.0.1", 9000))
    client.sendall(b"rooms\n")
    client.shutdown(socket.SHUT_WR)

    chunks: list[bytes] = []
    while chunk := client.recv(1024):
        chunks.append(chunk)

print(b"".join(chunks).decode())
```

Сначала в одном терминале запустите:

```bash
python echo_server.py
```

Это создаёт серверный процесс, listener и блокирующий `accept()`. Затем в
другом терминале:

```bash
python echo_client.py
```

Клиентский вывод:

```text
echo: rooms
```

Сервер покажет адрес клиента с временным портом, например:

```text
listening on 127.0.0.1:9000
connected ('127.0.0.1', 53124)
served one connection, stop
```

Реальный номер порта будет другим. Это нормально: его назначила ОС.

### Зачем нужен `shutdown`

TCP не хранит границу сообщения «вызов `sendall` закончился». Клиент
вызывает `shutdown(SHUT_WR)` и сообщает: новых байтов запроса не будет.
Сервер дочитывает поток до `recv() == b""`, но соединение ещё позволяет
отправить ответ в обратную сторону. После выхода из `with connection`
сервер закрывает сокет, и клиентский цикл тоже получает EOF.

Такой протокол годится только для опыта. Он не позволяет отправить второе
сообщение в том же соединении: EOF завершает всю клиентскую сторону потока.
Настоящий протокол задаёт длину, разделитель или кадры. Цикл из нескольких
`recv()` намеренно показывает, что один вызов не обязан получить всё
сообщение.

`SO_REUSEADDR` упрощает повторный запуск после недавнего закрытия порта. Он
не делает сервер многопользовательским и не заменяет `bind`.

---

## Опыт C · Два ожидания через `asyncio.gather`

Этот опыт обязателен независимо от выбранного сетевого маршрута. Создайте
`async_waits.py`:

```python
import asyncio
from time import perf_counter


async def check_room(room_id: str, delay: float) -> str:
    print("start", room_id)
    await asyncio.sleep(delay)
    print("ready", room_id)
    return room_id


async def main() -> None:
    started = perf_counter()

    rooms = await asyncio.gather(
        check_room("Atlas", 0.20),
        check_room("Bay", 0.10),
    )

    elapsed = perf_counter() - started
    print("result", rooms)
    print(f"elapsed {elapsed:.2f}s")


asyncio.run(main())
```

До запуска запишите четыре утверждения:

1. обе строки `start` появятся до первой строки `ready`;
2. `Bay` станет ready раньше `Atlas`;
3. `gather` вернёт список в порядке аргументов: `Atlas`, затем `Bay`;
4. полное время будет ближе к 0.20 секунды, чем к сумме 0.30.

Запуск:

```bash
python async_waits.py
```

Ожидаемая форма вывода:

```text
start Atlas
start Bay
ready Bay
ready Atlas
result ['Atlas', 'Bay']
elapsed 0.20s
```

Последнее число не является точным тестом: загруженная машина может добавить
задержку. Смысл не в сотых долях секунды, а в перекрытии двух таймеров.

### Сухой прогон

```text
t=0.00  main создаёт две работы в gather
t≈0     Atlas печатает start → await sleep(0.20) → отдаёт loop
t≈0     Bay печатает start   → await sleep(0.10) → отдаёт loop
t=0.10  таймер Bay готов      → Bay печатает ready
t=0.20  таймер Atlas готов    → Atlas печатает ready
t≈0.20  gather отдаёт результаты main в исходном порядке
```

`Bay` завершился первым, но результат не сортируется по моменту завершения.
`gather` сохраняет позицию каждого awaitable. Это две разные оси: порядок
событий и порядок результирующего списка.

### Сделайте ожидания последовательными

Не копируйте новый файл. Перепишите только тело `main` руками:

```python
async def main() -> None:
    started = perf_counter()

    atlas = await check_room("Atlas", 0.20)
    bay = await check_room("Bay", 0.10)

    elapsed = perf_counter() - started
    print("result", [atlas, bay])
    print(f"elapsed {elapsed:.2f}s")
```

Сначала предскажите вывод:

```text
start Atlas
ready Atlas
start Bay
ready Bay
result ['Atlas', 'Bay']
elapsed 0.30s
```

`await` не означает «запусти всё конкурентно». В этой версии `main`
полностью ждёт Atlas до создания корутины Bay. Конкурентность появилась в
первом варианте потому, что `gather` владел обеими незавершёнными работами.

---

## Упражнение в главе

Выполняйте изменения по одному и каждый раз сначала пишите прогноз.

1. В HTTP-маршруте добавьте путь `/health`, который возвращает
   `200`, `text/plain` и `ok\n`. Посчитайте `Content-Length` без запуска.
   Если выбрали echo, отправьте строку длиннее 1024 байтов и объясните,
   почему цикл всё равно соберёт её полностью.
2. В `async_waits.py` поменяйте задержки местами. Что изменится в строках
   `ready`, а что не изменится в `result`?
3. Добавьте третью комнату `Cedar` с задержкой `0`. Где она может появиться
   относительно строк `start`, и почему `await asyncio.sleep(0)` всё равно
   является возможностью отдать управление?
4. Временно замените ожидание Atlas на `time.sleep(0.20)` с нужным импортом.
   Предскажите, когда впервые запустится Bay. После наблюдения верните
   неблокирующий вариант.

Сдавать копию этих файлов как готовый Harbor не нужно. Их назначение —
соединить рисунок с наблюдаемым поведением.

---

## Harbor и интервью

- `http.server` показывает границу: библиотека разбирает протокол, а
  обработчик задаёт бизнес-ответ.
- Echo показывает, почему TCP требует собственного framing и нескольких
  чтений.
- `gather` показывает конкурентность ожиданий без параллельного CPU.
- Разница 0.20 против 0.30 секунды относится к двум искусственным ожиданиям,
  а не доказывает, что любой async-сервис быстрее.
- Все примеры одноразовые и локальные; переносить их как серверную
  архитектуру Harbor нельзя.

## Мини-голос

> Локальный HTTP-сервер слушает порт, принимает TCP-соединение и передаёт
> разобранный запрос обработчику.
> Raw socket возвращает части byte stream, поэтому границу сообщения задаёт
> мой протокол.
> В `gather` две корутины начинают ожидание до завершения друг друга.
> Event loop работает с другой task, пока таймер первой не готов.
> Время ожиданий перекрылось, но CPU не стал считать быстрее.

**Дальше:** [домашка M4b](./homework.md) — соберите сеть и конкурентность в
один рисунок, голос и несколько самостоятельных изменений.

<small>Если захочешь глубже позже: `unp-stevens`, главы 3–5, для socket API;
`asyncio` (Мэттью Фаулер), главы 1–4, для event loop и задач. Практика этой
страницы полностью выполняется без книг.</small>
