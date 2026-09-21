# 01 · dataclass · frozen · slots

**Мост:** цикл из M0.5 учит начинать с примеров. Здесь первый «пример» домена —
объекты комнаты и запроса, без которых не о чем говорить в Harbor.

**Вопрос сюжета:** как представить переговорку и запрос на бронь так, чтобы
не писать руками `__init__`/`__repr__` и не словить общий мутабельный default?

### С собесов

«Что такое dataclass?», «чем frozen отличается», «можно ли dataclass ключом
dict», «slots зачем». Редко просят написать декоратор `@dataclass` с нуля —
ждут, когда брать и какие грабли.

---

## Словарь

| Термин | English | На пальцах |
|--------|---------|------------|
| датакласс | dataclass | Класс «данные»: поля в аннотациях, `__init__` пишет за вас |
| поле | field | Атрибут с типом и опциями (`default`, `default_factory`) |
| замороженный | frozen | После создания поля не менять; хешируемый, если поля хешируемы |
| слоты | `__slots__` | Фиксированный набор имён; меньше памяти, нет `__dict__` |
| DTO | data transfer object | Объект «перенести данные», без жирной логики |

---

## Под капотом (модель)

Датакласс — генератор методов по **аннотациям полей**. Интерпретатор не
проверяет типы сам: он создаёт обычный класс с `__init__`, который кладёт
аргументы в атрибуты. `frozen` подменяет присваивание на ошибку.
`default_factory` вызывается **на каждый** новый экземпляр — поэтому
`list` не шарится между объектами.

---

## Руками → принято

**Руками** (идея):

```python
class BookingRequestManual:
    def __init__(self, guest_count: int, duration_minutes: int) -> None:
        self.guest_count = guest_count
        self.duration_minutes = duration_minutes

    def __repr__(self) -> str:
        return (
            f"BookingRequestManual({self.guest_count!r}, "
            f"{self.duration_minutes!r})"
        )
```

**Принято:**

```python
from dataclasses import dataclass


@dataclass
class BookingRequest:
    guest_count: int
    duration_minutes: int


# вход
req = BookingRequest(2, 60)
# выход
assert req.guest_count == 2 and req.duration_minutes == 60
print(req)  # BookingRequest(guest_count=2, duration_minutes=60)
```

Логику аллокации комнаты в dataclass **не** запихивайте — отдельные функции.

---

## default и default_factory

```python
from dataclasses import dataclass, field


@dataclass
class MeetingRoom:
    id: str
    capacity: int
    tags: list[str] = field(default_factory=list)
```

**Вход:** `MeetingRoom("r1", 2)` → `tags == []`.  
**Плохо:** `tags: list[str] = []` — один список на все экземпляры.

В Harbor для неизменяемых тегов удобнее `tuple[str, ...] = ()` (см. домашку).

---

## frozen — когда и зачем

```python
@dataclass(frozen=True)
class MeetingRoomId:
    value: str


rid = MeetingRoomId("A101")
# rid.value = "B"  # FrozenInstanceError
```

Замороженный экземпляр с хешируемыми полями — в `set` / ключ `dict`.
Мутабельный `list` внутри frozen ломает хеш-контракт.

**Сюжет Harbor:** id комнаты и снимок запроса — frozen DTO; занятость по
времени живёт в хранилище броней.

---

## slots (3.10+)

```python
@dataclass(slots=True)
class Point:
    x: float
    y: float
```

Меньше памяти на миллионы мелких объектов; нельзя навесить случайный
атрибут. Для маленького Harbor не must; знать флаг полезно.
`frozen=True, slots=True` — нормальный паттерн жёстких DTO.

---

## До / после

```python
# BAD
class Room:
    def __init__(self, name: str, features: list[str] = []):
        self.name = name
        self.features = features


# GOOD
@dataclass
class Room:
    name: str
    features: list[str] = field(default_factory=list)
```

---

## Применение

- JSON → dataclass (pydantic — в M7; идея контракта та же).
- Ключ кэша: frozen dataclass из параметров.
- Harbor: `MeetingRoom`, `BookingRequest` в домене.

## Мини-голос

> Dataclass убирает бойлерплейт полей. default_factory для list/dict.
> Frozen — неизменяемый и потенциально hashable. Slots — память и жёсткий
> набор атрибутов. Логику брони не прятать в dataclass.

**Дальше:** [02 · методы и декораторы](./02_methods_decorators.md) — собрать
комнату из строки «как из БД» и обернуть вызов.
