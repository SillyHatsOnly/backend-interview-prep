# 03 · Typing · Python 3.10–3.13

**Мост:** модели и декораторы из [01](./01_dataclasses.md)–[02](./02_methods_decorators.md)
уже с аннотациями. Здесь — зачем они, если `python` их не принуждает, и как
Protocol помогает тестам Harbor без жёсткого наследования.

**Вопрос сюжета:** как зафиксировать контракт комнаты так, чтобы IDE и
Pyright ловили враньё до прода?

### С собесов

«Зачем типы, если Python динамический?», «чем Protocol отличается от ABC»,
«что такое TypeAlias». Не просят цитировать PEP наизусть.

**Книга (`typed-py`):** Голобурдин — *Типизированный Python…* (2022) —
углубление; здесь минимум для собеса и Harbor.

---

## Словарь

| Термин | English | На пальцах |
|--------|---------|------------|
| аннотация | type annotation | Подсказка типа у аргумента/поля/возврата |
| объединение | union | `int \| None` — либо int, либо None |
| необязательный | Optional | Старое имя для `T \| None` |
| протокол | Protocol | «Утиный» интерфейс: есть поля/методы — подходит |
| псевдоним | TypeAlias | Имя для сложного типа |
| дженерик | generic | `list[str]`, `dict[str, int]` |
| проверка | type checker | mypy/pyright; не сам `python` |

---

## Под капотом (модель)

Аннотации — метаданные объекта функции/класса. Интерпретатор их почти не
использует. **Type checker** читает тот же исходник статически и строит граф
типов. Поэтому `f("no")` при `f(x: int)` в рантайме может «работать», а
Pyright — нет. В Harbor с M1 канон — **Pyright** (движок Pylance).

---

## База 3.10+

```python
def capacity_left(total: int, taken: int) -> int:
    return total - taken


def find_id(row: dict[str, str] | None) -> str | None:
    if row is None:
        return None
    return row.get("id")
```

Пишите `str | None`, не обязательно `Optional[str]`.  
`list[str]`, `dict[str, int]` — встроенные generic (не `List` из typing с 3.9).

---

## TypeAlias

```python
from typing import TypeAlias

ResourceId: TypeAlias = str
Tags: TypeAlias = tuple[str, ...]
```

С 3.12: `type ResourceId = str`. На собесе достаточно «псевдоним сложного
типа». В Harbor на 3.10–3.11 — `TypeAlias`.

---

## Protocol — утиная типизация

```python
from typing import Protocol


class HasCapacity(Protocol):
    capacity: int


def ok_for_guests(item: HasCapacity, guests: int) -> bool:
    return item.capacity >= guests
```

Любой объект с `capacity: int` подходит — не нужно общее наследование.
Удобно для фейков в тестах и границ модулей.

ABC — когда хотите запретить создание базы и требовать наследование.
Protocol — когда важны поля/методы, а не родословная.

---

## Callable и декораторы

```python
from collections.abc import Callable

Handler = Callable[[dict], dict]
```

Строгие проекты: `ParamSpec` / `TypeVar`. На middle — не бояться `Callable`
и `wraps`.

---

## Что проверки типов не делают

```python
def f(x: int) -> int:
    return x


f("no")  # python выполнит; pyright — ошибка
```

Типы — дисциплина и инструменты, не новая ВМ.

---

## Применение

- Публичные функции домена Harbor — аннотации + `pyright` clean.
- Protocol `HasCapacity` для комнаты.
- FastAPI/pydantic опираются на аннотации (M7).

## Мини-голос

> Аннотации — контракт для людей и type checker, не runtime-магия.
> `|` вместо Optional. list[str]/dict. Protocol — утиный интерфейс без
> наследования. В Harbor — Pyright.

**Дальше:** [домашка](./homework.md) — слой домена переговорок; в тексте
задач оставьте следы цикла из M0.5.
