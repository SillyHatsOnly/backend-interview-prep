# План · ядро ~8 месяцев + фаза 2

Актуально: **сентябрь 2026**. Публичный трек middle / middle+ Python backend.

**Ядро:** ~30–32 рабочих недели (~7.5–8 месяцев) при 10–12 ч/нед.
Попытка в крупную компанию — **гейт** после ядра, не потолок обучения.
**Фаза 2** — открытый трек: глубже, с удовольствием, без спешки календаря.

Скелет старого интенсива ~5 недель — только [калибровка темпа](../reference/legacy-5w.md),
без привязки к чужим путям на диске.

Педагогика слоёв — как Berkeley 61A → 61B → 61C и MIT (язык → алгоритмы →
машина → системы → дизайн): сначала абстракции и **как решать задачу**, потом
структуры, потом машина/ОС, потом concurrency, данные, API, доставка.

---

## 1. Бюджет

| | |
|---|---|
| Дни | пн · ср · пт |
| Блок | 3–4 ч до работы |
| План | **10–12 ч / неделя** |
| Выходные | отдых или ≤2 ч повторения |
| Ядро | **~30–32** рабочих недель |
| Оценка часов ядра | ~320–380 ч полезной работы |

Микрозадачи на работе: карточка голоса **или** 1 easy через
[цикл мышления](../modules/m0b-thinking/index.md). Не заменяют утренний блок.

---

## 2. Цель и гейты

**Цель ядра:** уверенный middle backend на Python: coding + мышление вслух,
pytest, async, SQL, рассказ про опыт; рынок РФ + повтор крупной компании
после гейта.

**Срезы-моки** (по материалу, **не** задачи из Harbor):

| После пакета | Формат |
|--------------|--------|
| M0.5–M1 | голос Python + короткий applied/typing |
| M2–M3 | алго 60–80 мин (с циклом мышления) |
| M4–M5 | concurrency устно + async pytest на чужом каркасе |
| M6–M7 | SQL/API устно + applied 90–120 мин |
| M7–M8 | короткий устный **system design** (карточки) |
| M8–M9 | сервис + compose / поиск устно |
| M10–M11 | mixed + опционально Go |

Полигон длинной практики стека — [Harbor](./capstone-harbor.md)
(переговорки). Моки с ним не смешиваем: [mocks](../mocks/index.md).

---

## 2a. Сквозной проект · Harbor

Один продукт на весь горизонт: **бронирование переговорок**, async API,
Postgres, поиск, Docker, в конце Go-воркер. Спека:
[capstone-harbor.md](./capstone-harbor.md).

Типы в Harbor: **Pyright** (строго) с M1.

Каждый модуль **добавляет слой** в Harbor. Отдельный git-репо для кода.

---

## 3. Принцип наслоения

```text
Как решать задачу (цикл мышления)
        ↓
Абстракции Python (+ SICP избр. фоном)
        ↓
Алгоритмы + структуры  (+ аллокатор Harbor)
        ↓
ОС-каркас (процесс/память/FD) → threads → asyncio
        ↓
Тесты (в т.ч. async)
        ↓
Данные: Postgres → (NoSQL/Elastic)
        ↓
HTTP API + Docker  (+ карточки system design)
        ↓
K8s карточки · Go-воркер
        ↓
Гейт → Фаза 2 (углубление)
```

Фоном всю дорогу: git (аварии), ОС/терминал через
[exploring-os-ru](https://github.com/SillyHatsOnly/exploring-os-ru) выборочно,
микрозадачи с циклом мышления, срезы-моки, Harbor, карточки
[system design](../reference/system-design.md).

---

## 4. Модули (пакеты)

Нумерация — порядок **открытия**. Внутри: теория цельной нитью → домашка →
голос.

| ID | Пакет | Содержание | Ориентир |
|----|--------|------------|----------|
| **M0** | Старт | карта, ритуал, окружение, git basics | ~1 нед |
| **M0.5** | Как решать задачу | цикл мышления, speak-aloud, brute→лучше | ~1.5–2 нед |
| **M1** | Python 3.10–3.13 | dataclasses, методы, декораторы, typing — одна нить Harbor | ~3–4 нед |
| **M2** | Алгоритмы I | Big O, массивы/хеши, pointers, window, stack, BS + цикл | ~4 нед |
| **M3** | Алгоритмы II | BFS/DFS, деревья, backtracking, heap; паттерны задач | ~4 нед |
| **M4** | Машина + concurrency | OS/CSAPP-lite → processes → threads/GIL → asyncio | ~4–5 нед |
| **M5** | Тесты | pytest, fixtures, **async pytest**; тесты Harbor | ~2 нед |
| **M6** | Postgres | Моргунов → *Art of PostgreSQL* (SQL↔Python) → DEV1 → QPT → куски DEV2/DBS; схема Harbor | ~5–6 нед |
| **SD** | System design base | сквозные карточки с конца M4 через M6–M9 | ~6–8 карточек |
| **M7** | HTTP / FastAPI | REST, pydantic, DI; публичное API Harbor | ~3 нед |
| **M8** | Elastic / NoSQL | Elastic + каталог Harbor; Redis/Mongo минимум | ~2–3 нед |
| **M9** | Docker | compose Harbor | ~2 нед |
| **M10** | K8s | [потолок](../reference/k8s-ceiling.md) | ~1 нед |
| **M11** | Go | синтаксис, goroutines; Go-воркер Harbor | конец ядра |
| **F*** | Фон | git accidents, ОС, SICP избр., паттерны Швеца 8–12, yakimka | весь горизонт |
| **H** | Harbor | [сквозной проект](./capstone-harbor.md) | весь горизонт |
| **P2** | Фаза 2 | после гейта: hard DP по желанию, Postgres internals, полный SD, больше Go/SICP | открыто |

Доли времени ядра (грубо):

| Блок | Доля |
|------|------|
| Алго + мышление (M0.5, M2–M3) | 30–32% |
| Async / ОС (M4) | 18% |
| Postgres (M6) | 18% |
| Python (M1) | 10% |
| Тесты (M5) | 7% |
| FastAPI (M7) | 7% |
| Docker / K8s / Elastic / SD | 8% |
| Go + фон + Harbor glue | 6% |

~10–15% времени модулей сразу уходит в **слой Harbor**.

---

## 5. Источники (не копируем verbatim)

### Чеклист вопросов
- [yakimka/questions.md](https://github.com/yakimka/python_interview_questions/blob/master/questions.md) —
  весь как checklist. Трекер: [yakimka.md](../reference/yakimka.md).

### PostgresPro
- [DEV1](https://postgrespro.ru/education/courses/DEV1) · [DEV2](https://postgrespro.ru/education/courses/DEV2)
- [QPT](https://postgrespro.ru/education/courses/QPT) · [DBS](https://postgrespro.ru/education/courses/DBS)
- Порядок: SQL/Моргунов → *The Art of PostgreSQL* (Fontaine, мост
  приложение↔SQL) → DEV1 → QPT → выборочно DEV2/DBS
- Рогов *PostgreSQL 16 изнутри* — углубление / фаза 2, не must ядра

### Книги
[books.md](../reference/books.md) — библиография (без путей и имён файлов).

### Вакансии рынка
Периодически сверять hh.ru: Python backend middle — стек
(async, Postgres, FastAPI/Django, Docker, очереди, кэш).

### ОС
- [exploring-os-ru](https://github.com/SillyHatsOnly/exploring-os-ru) —
  [выборочный каркас](../reference/os-fundamentals.md), не второй полный трек.

### System design
- [system-design.md](../reference/system-design.md) + карточки в
  [sd-cards](../reference/sd-cards/index.md).

### Сквозной код
- [Harbor](./capstone-harbor.md) — отдельный репозиторий приложения.

### Скелет 5 недель
Идея короткого интенсива как калибровка темпа — [legacy-5w](../reference/legacy-5w.md).
Плотные главы async/SQL из того интенсива переносим идеями в M4/M6, не копируем
verbatim.

---

## 6. Сознательно не раздуваем (в ядре)

- Полный SICP (избранные главы; полный — фаза 2)
- Вся энциклопедия паттернов Швеца (только частотные, `patterns-dive`)
- Hard DP как must; «спроектируй Instagram» как must
- Кубер как работа админа кластера
- Все 70 дней Exploring OS как must
- Django-энциклопедия yakimka — карточками, не курсом Django
- PDF книг в git
- Эталоны solutions **до** сдачи домашки
- Три разных capstone вместо одного Harbor

---

## 7. Порядок генерации материалов (с агентом)

1. Каркас сайта + этот план + Harbor-спека
2. M0 + канон + мышление (M0.5)
3. M1 цельной нитью + SD-карточки по мере модулей
4. Пакеты по слоям с Harbor
5. После пакета: домашка, yakimka, мок если логично
6. Публикация — `mkdocs` → GitHub Pages (без EPUB)

---

## 8. Критерий «готов к гейту собеса»

- [ ] 4+ среза-мока по пройденным пакетам (не из Harbor)
- [ ] Цикл мышления на алго: голос без пустой тишины
- [ ] Async: loop/gather/blocking/session — голос + код
- [ ] Async pytest на своём маленьком сервисе
- [ ] Postgres: индекс, EXPLAIN, txn, lost update, ACID vs BASE
- [ ] FastAPI + Postgres Harbor через compose
- [ ] Аллокатор Harbor с тестами (edges)
- [ ] Базовый SD: 4–6 коробок + bottleneck вслух
- [ ] 1 Go-воркер или осознанный defer
- [ ] Yakimka checklist: закрыты P0-темы без «пустоты»
- [ ] Опыт: один STAR-кейс с цифрой
- [ ] ОС-голос: process vs thread vs FD/blocking I/O

После гейта — фаза 2 (см. модуль **P2** в таблице выше) без обрезки: больше
глубины и радости от разработки.
