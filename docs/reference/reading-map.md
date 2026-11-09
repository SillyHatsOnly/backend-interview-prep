# Карта чтения · глава → модуль

Актуально: **ноябрь 2026**. Книги лежат у читателя локально; в курс —
только **ID + глава/§**. Файлы и пути в git не пишем. Пересказ, не verbatim.

Канон слоёв: [sources-canon](./sources-canon.md) · каталог ID: [books](./books.md).

Правило: **не больше одной опоры за слот.** Сначала домашка модуля, потом
указанный кусок книги — или наоборот, если глава курса явно шлёт «прочитай §».

---

## Сразу к делу (минимум до следующего пакета материалов)

| Когда | ID | Что открыть |
|-------|-----|-------------|
| Старт M2 | `skiena` | гл. **1–2** (дизайн + анализ); каталог Part II — по нужде задачи |
| Фон привычек (1 topic / неделя) | `pragmatic` | Topics **1–7**, потом **8–11** |
| К M1b (когда пакет выйдет) | `oust-psd` | гл. **2–4**, **10**; `fluent-py` — Data Model + dataclasses/typing по отсылке |
| Чужой код | `feathers-legacy` | гл. **1–4** (изменения, обратная связь, швы) |

Остальное — по таблице ниже, не раньше своего слоя.

---

## `skiena` · Algorithm Design Manual, 3e (2020)

| Модуль курса | Главы Skiena |
|--------------|--------------|
| M2 сложность | **2** Algorithm Analysis |
| M2 хеш | **6** Hashing… (идея + когда хеш); при нужде **3** Data Structures |
| M2 указатели / окно / стек | практика курса; каталог Part II по типу задачи |
| M2 бинарный поиск | **2.7** / мотивы из **5** Divide and Conquer |
| M3 графы BFS/DFS | **7** Graph Traversal |
| M3 куча / приоритеты | **3** + каталог; взвешенные — задел **8** |
| M3 backtracking | **9** Combinatorial Search |
| Позже / hard | **10** DP, **11–12** NP / hard — не блокер ядра |
| Выбор приёма всегда | **13** How to Design Algorithms + **14+** Hitchhiker’s catalog |

**Выход:** как в L2 канона — brute → названный приём → код.

---

## `oust-psd` · Philosophy of Software Design, 2e

| Модуль | Главы |
|--------|-------|
| M1b / ремесло | **2** Nature of Complexity, **3** Working Code Isn't Enough, **4** Deep Modules |
| M1b исключения / края | **10** Define Errors Out Of Existence |
| M5 / правки Harbor | **16** Modifying Existing Code, **18** Code Should be Obvious |
| Спор с лозунгами | **9.8** vs Clean Code — читать критически |

---

## `feathers-legacy` · Working Effectively with Legacy Code  
(рус. изд.: Физерс — *Эффективная работа с унаследованным кодом*, 2009)

| Модуль | Главы (нумерация рус. перевода) |
|--------|----------------------------------|
| M1b чтение чужого | **1–5** (изменения, обратная связь, распознавание, **модель шва**, инструменты) |
| M5 | **6–9** (мало времени, бесконечные правки, новое свойство, класс в тесты) |
| По голоду | ч. III методы разрыва зависимостей — точечно под задачу |

---

## `fluent-py` · Fluent Python, 2e

| Модуль | Ориентир (по темам 2e) |
|--------|-------------------------|
| M1 | Data model; data class builders; type hints |
| M1b | последовательности; dict/set; функции как объекты; декораторы; ссылки/копии |
| M4b | concurrency / asyncio-части книги — рядом с курсовым asyncio, не вместо |

Точные номера глав в отсылках модуля M1b — когда пакет материалов выйдет.

---

## `pragmatic` · Pragmatic Programmer, 2e (2020)

| Когда | Topics |
|-------|--------|
| Фон с M0 | **1–7** (философия: ответственность, entropy, good-enough, портфель, коммуникация) |
| Рядом с циклом / дизайном | **8–11** (design essence, DRY, orthogonality, reversibility) |
| Инструменты / git | **16–20** |
| Перед M5 | **40–41** (refactoring, test to code) |
| Перед M4b | **33–35** (concurrency topics) |

Один topic за микрослот, не «проглотить книгу».

---

## `khorikov-ut` · Unit Testing Principles… (Manning, 2020)

| Модуль | Фокус |
|--------|-------|
| M5 старт | ч. о целях тестов: защита от регрессий vs сопротивление рефакторингу; ложные +/- |
| M5 | mocks — когда да/нет; managed vs unmanaged dependencies |
| M5 / Harbor | пирамида: unit vs integration; что покрывать в доменной модели |

---

## `refactoring` · Fowler, 2e

| Модуль | Главы |
|--------|-------|
| M5 | **1** First Example (прожить один раз руками), **2** Principles, **3** Bad Smells, **4** Building Tests |
| По нужде | каталог **6–12** — один запах → один приём, не заучивание списка |

---

## `art-of-pg` · Art of PostgreSQL, 2e Updated (2026)

| Модуль | Части книги |
|--------|-------------|
| M6 после Моргунова-старта | **Part 3** Writing SQL Queries → **Part 4** SQL Toolbox |
| M6 моделирование | **Part 5** Data Types → **Part 6** Data Modeling |
| M6 txn / гонки | **Part 7** Data Manipulation and Concurrency Control |
| Опц. | **Part 8** Extensions |

---

## `ddia` · Designing Data-Intensive Applications, 2e (2026)  
Kleppmann & Riccomini; ISBN 978-1-098-11906-5

| Модуль / SD | Главы 2e |
|-------------|----------|
| Конец M6 / старт SD | **1** Trade-offs, **2** Nonfunctional requirements (latency, reliability, scale) |
| M6 модели | **3** Data models (рядом с SQL-путём, не вместо) |
| Индексы / хранение | **4** Storage and Retrieval |
| Эволюция API/схем | **5** Encoding and Evolution |
| Репликация / scale | **6** Replication → **7** Sharding |
| Транзакции глубже | **8** Transactions — после живых txn в Postgres |
| Распределённые края | **9–10** — фаза 2 или сильный голод |
| Потоки / батч | **11–13** — P2 / спец. |

Не читать 2e «с 1 до конца» до первого SQL-мока.

---

## `kurose` · Computer Networking: A Top-Down Approach, 9e

| Модуль | Главы |
|--------|-------|
| M4b картина | **1** Internet overview → **2** Application (HTTP) → **3** Transport (TCP/UDP) |
| Позже / P2 | **4–5** network layer; **8** security — по голоду |
| Глубина сокетов | не вместо `unp-stevens`: Kurose — «зачем пакеты», Stevens — «как писать сокет» |

---

## Чего здесь нет

Пути к PDF/EPUB, имена файлов загрузчика, прогресс «я на главе N».
Это учебная карта курса, не личный трекер.
