# 02 · classmethod · staticmethod · декораторы

**Мост:** в [01](./01_dataclasses.md) появились модели комнаты и запроса. Данные
в проде приходят dict/JSON/строкой БД — нужен способ **собрать** объект и
иногда **обернуть** вызов (замер, лог, позже retry).

**Вопрос сюжета:** чем фабрика на классе отличается от «просто функции», и
как декоратор не убить `__name__`?

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

## classmethod vs staticmethod

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

    @staticmethod
    def is_valid_capacity(n: int) -> bool:
        return n > 0
```

**Вход:** `MeetingRoom.from_row({"id": "A", "capacity": "3"})`  
**Выход:** `MeetingRoom(id='A', capacity=3)`.

`from_row` читает «сырой» словарь (как строка БД или JSON) и собирает доменную
модель. Так граница «внешние данные → наш тип» живёт в одном месте.

`is_valid_capacity` не использует ни экземпляр, ни класс: это правило про
число. Его можно вынести в модуль. Static оставляют, когда хотят, чтобы в
документации и автодополнении правило торчало **рядом** с `MeetingRoom`.

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

- `@classmethod from_mapping` — строка БД / JSON → домен (домашка M1).
- Декораторы: retry позже, метрики latency, `@app.get` в FastAPI.
- Harbor: валидация capacity; позже — идемпотентность на handler.

## Мини-голос

> classmethod — фабрика от класса. staticmethod — функция рядом с типом без
> self. Декоратор возвращает обёртку; wraps сохраняет имя. Несколько
> декораторов — скобки от нижнего к верхнему.

**Дальше:** [03 · typing](./03_typing.md) — контракт полей так, чтобы Pyright
ловил враньё до рантайма.
