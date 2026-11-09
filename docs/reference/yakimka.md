# Yakimka · checklist

Источник:
[questions.md](https://github.com/yakimka/python_interview_questions/blob/master/questions.md).

Берём **весь** банк как чеклист покрытия. Известные ошибки/опечатки/кривую
логику в ответах банка — исправляем в наших карточках, не копируем слепо.

Статусы ниже обновляем по мере модулей (`todo` → `covered`).

## Python core

| Тема | Статус | Модуль |
|------|--------|--------|
| Последовательности (list/tuple/str) | covered | M1 (база) + drills |
| Множества и отображения | covered | M1 / далее алго |
| Функции (args/kwargs, scope) | covered | M1b · 03 |
| Итераторы и генераторы | covered | M1b · 02 |
| Классы, MRO | todo | later |
| Модули, пакеты | covered | M1b · 05 (+ M0/M1) |
| Исключения | covered | M1b · 01 |
| Декораторы | covered | M1 · 02 |
| Метаклассы | todo | карточка later (не deep) |
| Ввод-вывод | todo | M1 / терминал |
| Тестирование | covered | M0 smoke · глубже M5 |
| FP (map/filter/reduce) | todo | M1 |
| GIL, потоки, процессы | covered | M4a · 01 + M4b · 02 |
| Singleton варианты | todo | паттерны (`patterns-dive`) |
| Линтеры / code style | covered | M1b · 06 Pyright (+ M0/M1) |
| Comprehensions | covered | M1b · 03 |
| `_` / `__` | todo | M1 |
| copy vs deepcopy | covered | M1b · 04 |
| GC | todo | M1 |
| Интроспекция / рефлексия | todo | карточка |
| dataclass / frozen | covered | M1 · 01 |
| classmethod / staticmethod | covered | M1 · 02 |
| typing 3.10+ / Protocol | covered | M1 · 03 |

## Web / HTTP (без Django-курса)

| Тема | Статус | Модуль |
|------|--------|--------|
| AuthN vs AuthZ, XSS, CSRF идея | todo | M7 + карточки |
| REST (+ SOAP одной фразой) | todo | M7 |
| HTTP/HTTPS | todo | M7 |
| Django/DRF блоки | todo | только «зачем», не курс |

## Общее

| Тема | Статус | Модуль |
|------|--------|--------|
| ООП принципы | todo | M1 + паттерны |
| SOLID / cohesion / coupling | todo | M1 + паттерны |

Новые строки добавляйте, когда закрываете тему в модуле.
