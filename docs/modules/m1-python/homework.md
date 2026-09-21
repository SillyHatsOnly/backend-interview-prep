# Домашка M1 · Python foundation

Теория: [index](./index.md) → 01 → 02 → 03.  
Ревью: «ревью M1». Эталонов заранее нет.

Где уместно — коротко приложите **цикл M0.5** (хотя бы примеры входа/выхода
и края) перед кодом.

Формат: код в drills и/или в `harbor-capstone`; текст — рядом или в чат.

---

## Задание 1. Dataclass *(обязательно)*

В `harbor-capstone` (заведёте сами в M0):

1. `@dataclass(frozen=True)` модель `MeetingRoom`: `id: str`, `capacity: int`,
   `tags: tuple[str, ...] = ()`.
2. `@dataclass` (не frozen) `BookingRequest`: `guest_count: int`,
   `duration_minutes: int`. В `__post_init__`: оба > 0, иначе `ValueError`.
3. Тесты: создание; `MeetingRoom` нельзя менять; невалидный запрос падает.
4. **`pyright`** — когда подключите руками; до этого достаточно тестов.

---

## Задание 2. classmethod *(обязательно)*

У `MeetingRoom` метод `from_mapping(cls, data: dict) -> MeetingRoom`:
ключи `"id"`, `"capacity"`, опционально `"tags"` (list → tuple).  
Тест round-trip: dict → объект → поля совпали.

---

## Задание 3. Декоратор *(обязательно)*

`@count_calls` — считает вызовы (`wrapper.call_count`), с `functools.wraps`.  
Тест: три вызова → `call_count == 3`, `__name__` сохранено.

---

## Задание 4. Typing · текст *(обязательно)*

8–10 предложений:

1. Зачем аннотации, если `python` их не принуждает.
2. Чем `Protocol` удобнее «общего базового класса» для теста с фейком.
3. Почему `tags: list[str] = []` в dataclass плохо.

---

## Задание 5. Разбор чужого кода *(обязательно)*

```python
@dataclass
class Cart:
    items: list[str] = []

    @staticmethod
    def from_json(data: dict) -> "Cart":
        return Cart(data["items"])
```

Что ломается (2+ проблемы)? Исправьте + 5–7 предложений почему.
Подсказка: default, staticmethod vs classmethod.

---

## Задание 6. Hard · позже

`allocate(rooms: list[MeetingRoom], req: BookingRequest) -> MeetingRoom | None`:
первая комната с `capacity >= guest_count`, иначе `None`.  
Без интервалов времени — заготовка к M2–M3.  
Сначала цикл мышления (примеры + brute «пройти список»), потом код.

---

## Когда готово

- [ ] 1–5
- [ ] Harbor: модели хотя бы закоммичены
- [ ] 6 — по желанию / вернуться в M2
