# 02 · classmethod · staticmethod · декораторы

**Мост:** в [01](./01_dataclasses.md) появились модели комнаты и запроса. Данные
в проде приходят dict/JSON/строкой БД — нужен способ **собрать** объект и
иногда **обернуть** вызов (замер, лог, позже retry).

**Вопрос сюжета:** зачем методу знать класс, а не объект, и как декоратор
не убить `__name__`?

### С собесов

«Зачем `@classmethod`», «чем отличается от static», «напишите декоратор с
аргументами», «что будет без `functools.wraps`». Метаклассы — не этот пакет.

---

## Словарь

| Термин | English | На пальцах |
|--------|---------|------------|
| метод экземпляра | instance method | Первый аргумент — `self`, уже созданный объект |
| метод класса | classmethod | Первый аргумент — `cls`, сам класс; часто «другой конструктор» |
| статический метод | staticmethod | Обычная функция, лежащая в пространстве имён класса |
| декоратор | decorator | Берёт функцию и возвращает обёртку вокруг неё |
| обёртка | wrapper | То, что реально вызывают после `@декоратор` |
| wraps | `functools.wraps` | Копирует имя и докстринг с оригинала на обёртку |

---

## Под капотом (модель)

Обычный метод получает **экземпляр** (`self`): «эта конкретная комната».
`classmethod` получает **класс** (`cls`): «тип MeetingRoom или его наследник».
Поэтому фабрика `from_row` у наследника создаст экземпляр наследника, а не
жёстко базовый тип — в `cls(...)` подставится актуальный класс.

`staticmethod` — способ положить чистую функцию рядом с типом в API, без
`self` и `cls`. Поведение как у функции модуля; отличие в основном в том,
где её ищут глазами (`MeetingRoom.is_valid_capacity`).

Декоратор — функция, которая на входе получает другую функцию и возвращает
новую (обёртку). Синтаксис `@timed` над `def f` — это «замени `f` результатом
`timed(f)`». Если декоратору нужны свои параметры (`@repeat(3)`), сначала
вызывается внешняя функция с аргументами и только она возвращает уже
обычный декоратор — получается два уровня вложенности.

---

## `@classmethod` — метод класса, не объекта

Обычный метод привязан к **уже созданной** комнате: первый аргумент `self`,
внутри можно читать `self.capacity`. `@classmethod` привязан к **самому
классу**. Python подставляет первым аргументом не объект, а класс — его
принято называть `cls`.

Вызов `MeetingRoom.from_row(row)` и вызов у наследника
`QuietRoom.from_row(row)` выглядят одинаково, но `cls` внутри разный: во
втором случае это `QuietRoom`. В этом и смысл. Если внутри написать жёстко
`return MeetingRoom(...)`, наследник получит объект базового класса и
потеряет свои поля и поведение. `return cls(...)` создаёт экземпляр **того
класса, у которого метод вызвали**.

У `@staticmethod` такого аргумента нет вообще: ни объекта, ни класса. Это
функция, которую положили рядом с типом, чтобы её было легко найти. Наследник
её не «перенастраивает» через `cls`.

Ниже три случая, где `cls` реально нужен. Первый — основной для Harbor и
собеса.

### 1. Другой способ создать объект

У класса один `__init__`. Данные снаружи приходят по-разному: словарём из
БД, строкой из CSV, JSON. Второй и третий `__init__` объявить нельзя.
Фабричный `@classmethod` — обычный способ: разобрать чужой формат и вернуть
готовый объект.

```python
from __future__ import annotations

from dataclasses import dataclass


@dataclass(frozen=True)
class MeetingRoom:
    id: str
    capacity: int

    @classmethod
    def from_row(cls, row: dict) -> MeetingRoom:
        return cls(id=row["id"], capacity=int(row["capacity"]))

    @classmethod
    def from_csv(cls, line: str) -> MeetingRoom:
        room_id, capacity = line.split(",")
        return cls(room_id.strip(), int(capacity))

    @staticmethod
    def is_valid_capacity(n: int) -> bool:
        return n > 0
```

**Вход:** `MeetingRoom.from_row({"id": "A", "capacity": "3"})`  
**Выход:** `MeetingRoom(id='A', capacity=3)`.

**Вход:** `MeetingRoom.from_csv("B, 8")`  
**Выход:** `MeetingRoom(id='B', capacity=8)`.

Граница «сырые данные → наша модель» живёт на классе, а не размазана по
обработчикам. Если появится подкласс (комната с дополнительным полем), его
`from_row` через `cls` соберёт **подкласс**, а не голый `MeetingRoom`.

`is_valid_capacity` классу не нужен: это проверка числа. Её можно держать
модульной функцией. `@staticmethod` оставляют, когда хотят, чтобы в
автодополнении правило искалось как `MeetingRoom.is_valid_capacity`.

### 2. Состояние, общее для всех экземпляров

Иногда нужно число или реестр, которые относятся к типу целиком, а не к одной
комнате: сколько объектов уже собрали, общий кэш, лимит. Такое поле вешают
на класс и меняют через `@classmethod`, чтобы наследник мог иметь **свой**
счётчик, а не писать в счётчик родителя.

```python
class MeetingRoom:
    created: int = 0

    def __init__(self, room_id: str) -> None:
        self.room_id = room_id
        type(self).note_created()

    @classmethod
    def note_created(cls) -> None:
        cls.created += 1

    @classmethod
    def how_many(cls) -> int:
        return cls.created
```

`MeetingRoom("A")` увеличивает `MeetingRoom.created`, потому что
`type(self).note_created()` передаёт в `cls` фактический класс объекта.
Если у наследника свой `created = 0`, его объекты крутят его счётчик.
Жёсткое `MeetingRoom.created += 1` этот смысл ломает: все пишут в счётчик
родителя.

На собесе этот случай узнают. В Harbor на старте он не обязателен: комнаты
живут в хранилище, а не в счётчике класса.

### 3. Настройка класса, которую наследник подменяет

Метод читает атрибут **класса**. Дочерний класс меняет атрибут — тот же
метод начинает вести себя иначе, без копипасты тела.

```python
class ScheduleExporter:
    extension = "txt"

    @classmethod
    def filename(cls, name: str) -> str:
        return f"{name}.{cls.extension}"


class PdfExporter(ScheduleExporter):
    extension = "pdf"
```

`ScheduleExporter.filename("week")` → `week.txt`.  
`PdfExporter.filename("week")` → `week.pdf`.

Если написать `ScheduleExporter.extension` вместо `cls.extension`, наследник
навсегда останется с `txt`.

### Когда какой метод

| Тип | Первый аргумент | Когда брать |
|-----|-----------------|-------------|
| обычный метод | `self` — этот объект | Нужны поля конкретной комнаты: `self.capacity` |
| `@classmethod` | `cls` — этот класс | Собрать объект из другого формата, или прочитать/изменить данные класса так, чтобы наследник подставил себя |
| `@staticmethod` | ничего само не приходит | Утилита рядом с типом: ни объект, ни класс ей не нужны |

Если метод не использует ни `self`, ни `cls`, `@classmethod` только
притворяется нужным. Если внутри зашито имя класса вместо `cls`,
наследование молча сломано — это как раз то, что фабрика должна чинить.

---

## Декоратор руками → принято

Нужно что-то сделать «вокруг» вызова: замерить время, залогировать, позже —
повторить при ошибке. Вместо копипасты в каждой функции пишут обёртку один раз.

```python
import time
from functools import wraps
from typing import Callable, TypeVar

F = TypeVar("F", bound=Callable[..., object])


def timed(fn: F) -> F:
    @wraps(fn)
    def wrapper(*args, **kwargs):
        t0 = time.perf_counter()
        try:
            return fn(*args, **kwargs)
        finally:
            dt = time.perf_counter() - t0
            print(f"{fn.__name__}: {dt:.4f}s")

    return wrapper  # type: ignore[return-value]


@timed
def allocate_stub(n: int) -> int:
    return n * 2


assert allocate_stub(3) == 6
# побочный выход: печать "allocate_stub: 0.000x s"
```

**Зачем `wraps`.** Без него у `allocate_stub` в рантайме имя станет
`wrapper`: сломаются логи, доктесты, отладка («кто меня вызвал?»).
`@wraps(fn)` копирует `__name__`, `__doc__` и связанное с оригинала на
обёртку — снаружи функция «как будто та же», внутри добавлено поведение.

**Декоратор с аргументами** — ещё один уровень: сначала `repeat(3)` возвращает
декоратор, уже он оборачивает функцию.

```python
def repeat(times: int):
    def decorator(fn: Callable[..., object]):
        @wraps(fn)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(times):
                result = fn(*args, **kwargs)
            return result

        return wrapper

    return decorator
```

`@repeat(3)` читается как `f = repeat(3)(f)`.

---

## Порядок нескольких декораторов

```python
@timed
@repeat(2)
def f():
    ...
```

Ближайший к `def` применяется первым: сначала `repeat(2)`, потом `timed`.
В скобках: `timed(repeat(2)(f))`. На собесе нарисуйте скобки — меньше путаницы,
кто во что вложен.

---

## До / после

```python
# BAD — нет wraps, в лог попадает бессмысленное имя обёртки
def log(fn):
    def wrapper(*a, **k):
        print("call", wrapper)
        return fn(*a, **k)
    return wrapper


# GOOD — wraps + имя оригинала
def log(fn):
    @wraps(fn)
    def wrapper(*a, **k):
        print("call", fn.__name__)
        return fn(*a, **k)
    return wrapper
```

---

## Применение

- `@classmethod from_row` / `from_csv` — чужой формат → домен; `cls`, не имя класса (домашка M1).
- Общий счётчик или настройка класса — через `cls`, чтобы наследник не писал в родителя.
- `@staticmethod` — проверка без объекта и без класса (`is_valid_capacity`).
- Декораторы: retry позже, метрики latency, `@app.get` в FastAPI.
- Harbor: валидация capacity; позже — идемпотентность на handler.

## Мини-голос

> classmethod получает класс (`cls`), не объект. Чаще всего это второй
> конструктор: из строки БД или CSV, и наследник получит свой тип.
> Ещё — общее состояние и атрибут класса, который потомок подменяет.
> staticmethod — утилита рядом с типом, ей класс не нужен.
> Декоратор возвращает обёртку; wraps сохраняет имя. Несколько декораторов —
> скобки от нижнего к верхнему.

**Дальше:** [03 · typing](./03_typing.md) — контракт полей так, чтобы Pyright
ловил враньё до рантайма.
