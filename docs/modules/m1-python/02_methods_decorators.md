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
| метод экземпляра | instance method | Первый аргумент — `self` |
| метод класса | classmethod | Первый аргумент — `cls`; часто альтернативный конструктор |
| статический метод | staticmethod | Функция в пространстве имён класса без `self`/`cls` |
| декоратор | decorator | Оборачивает другую функцию |
| обёртка | wrapper | То, что реально вызывают после декоратора |
| wraps | `functools.wraps` | Копирует `__name__` / `__doc__` на обёртку |

---

## Под капотом (модель)

`classmethod` передаёт **класс**, не экземпляр — поэтому `from_row` у наследника
создаст наследника. `staticmethod` — синтаксический сахар «положить функцию
рядом с типом». Декоратор с аргументами — фабрика декораторов: сначала
вызывается внешняя функция с параметрами, она возвращает обычный декоратор.

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

`is_valid_capacity` можно вынести в модуль; static оставляют, когда правило
должно жить **рядом** с типом в API.

---

## Декоратор руками → принято

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

Без `@wraps(fn)` имя станет `wrapper` — ломаются доктесты, логи, отладка.

**С аргументами:**

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

`@repeat(3)` → `repeat(3)` → `decorator` → обёртка функции.

---

## Порядок нескольких декораторов

```python
@timed
@repeat(2)
def f():
    ...
```

Ближайший к `def` применяется первым: `timed(repeat(2)(f))`.

---

## До / после

```python
# BAD
def log(fn):
    def wrapper(*a, **k):
        print("call", wrapper)
        return fn(*a, **k)
    return wrapper


# GOOD
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
- Декораторы: retry позже, метрики, `@app.get` в FastAPI.
- Harbor: валидация capacity; позже — идемпотентность на handler.

## Мини-голос

> classmethod — фабрика от класса. staticmethod — функция в классе без self.
> Декоратор возвращает обёртку; wraps сохраняет имя. Несколько декораторов —
> скобки от нижнего к верхнему.

**Дальше:** [03 · typing](./03_typing.md) — контракт полей так, чтобы Pyright
ловил враньё до рантайма.
