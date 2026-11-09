# Книги · каталог отсылок

Файлы книг **не** в git. В материалах — **автор, название, издание**;
в главе — **глава / § / страница** + пересказ. Пути на диске и имена файлов
в курс не пишем. Verbatim из книги — нельзя.

**Как устроена полка:** [канон источников](./sources-canon.md) — тиры A/B/C/D,
слои и выходы. **Какие главы когда:** [reading-map](./reading-map.md).
Этот файл — ID и библиография.

## Как ссылаться в главе

```text
Книга: Моргунов Е. П. — PostgreSQL. Профессиональный SQL, 2025.
Место: гл. … / § … / с. …
Дальше — пересказ своими словами.
```

Тир в таблице:

| Тир | Смысл |
|-----|--------|
| A | ядро слоя (не «все страницы») |
| B | спутник модуля |
| C | фаза 2 / по голоду |
| D | не must; читать критически или как словарь |

---

## Каталог

### Мышление и абстракция

| ID | Тир | Книга | Модули |
|----|-----|--------|--------|
| `sicp` | A фон / C полный | Харольд Абельсон, Джеральд Джей Сассман — *Структура и интерпретация компьютерных программ* (SICP); MIT Press / рус. перевод по имеющемуся изданию | F* ≤1 гл./мес; P2 полный |
| `htdp` | C | Matthias Felleisen et al. — *How to Design Programs*, **2nd edition** (мягкий процесс «задача → данные → функции»; если SICP тяжело как *старт*) | P2 / мост к циклу |
| `crafting-interpreters` | C | Robert Nystrom — *Crafting Interpreters* (два интерпретатора; язык перестаёт быть магией) | P2 |
| `mcs-mit` | C | Eric Lehman, F. Thomson Leighton, Albert Meyer — *Mathematics for Computer Science* (MIT; логика, индукция, графы — точечно, не курс матана) | P2 / дыры |

### Машина, ОС, сеть

| ID | Тир | Книга | Модули |
|----|-----|--------|--------|
| `code-petzold` | B | Charles Petzold — *Code: The Hidden Language of Computer Hardware and Software* (снять магию битов/логики; мост к CSAPP) | M4a фон |
| `nand2tetris` | C | Noam Nisan, Shimon Schocken — *The Elements of Computing Systems* (Nand2Tetris; машина руками) | P2 |
| `csapp` | A/B | Рэндал Э. Брайант, Дэвид Р. О'Халларон — *Компьютерные системы: архитектура и программирование*, **3-е изд., 2022** (CSAPP) | M4a выборочно |
| `ostep` | A | Remzi H. Arpaci-Dusseau, Andrea C. Arpaci-Dusseau — *Operating Systems: Three Easy Pieces* (виртуализация, concurrency, persistence; свободно онлайн) | M4a |
| `kurose` | B | James F. Kurose, Keith W. Ross — *Computer Networking: A Top-Down Approach*, **9th edition** (картина сверху: HTTP→TCP→IP) | M4b обзор |
| `unp-stevens` | B/C | W. Richard Stevens, Bill Fenner, Andrew M. Rudoff — *UNIX Network Programming, Volume 1: The Sockets Networking API*, **3rd edition** (сокеты; справочник, не «1000 стр. до FastAPI») | M4b / P2 |

### Алгоритмы

| ID | Тир | Книга | Модули |
|----|-----|--------|--------|
| `skiena` | A | Steven S. Skiena — *The Algorithm Design Manual*, **3rd edition, 2020** (**рабочий** текст: выбор алгоритма, каталог, практика) | M2–M3 |
| `clrs4` | B | Cormen, Leiserson, Rivest, Stein — *Introduction to Algorithms*, **4th edition**, MIT Press, ISBN 978-0-262-04630-5 (справочник / доказательства по нужде) | M2–M3 |

### Python

| ID | Тир | Книга | Модули |
|----|-----|--------|--------|
| `typed-py` | A/B | Алексей Голобурдин (команда «Диджитализируй!») — *Типизированный Python для профессиональной разработки*, редакция **от 8 июня 2022** (актуальные версии — у автора) | M1 / M1b |
| `fluent-py` | B | Luciano Ramalho — *Fluent Python: Clear, Concise, and Effective Programming*, **2nd edition** (модель данных, последовательности, декораторы, concurrency) | M1b / M4b |
| `asyncio` | A/B | Мэттью Фаулер — *Asyncio и конкурентное программирование на Python* | M4b |

### Ремесло, тесты, чужой код

| ID | Тир | Книга | Модули |
|----|-----|--------|--------|
| `pragmatic` | A фон | David Thomas, Andrew Hunt — *The Pragmatic Programmer*, **2nd edition / 20th Anniversary, 2020** | F* / весь горизонт |
| `oust-psd` | A/B | John Ousterhout — *A Philosophy of Software Design*, **2nd edition** (сложность, глубокие модули; спорит с лозунгами Clean Code) | M1b / M5 / F* |
| `feathers-legacy` | A/B | Michael Feathers — *Working Effectively with Legacy Code*; рус. перевод: М. Физерс — *Эффективная работа с унаследованным кодом*, **2009** | M1b / M5 |
| `refactoring` | B | Martin Fowler — *Refactoring: Improving the Design of Existing Code*, **2nd edition** | M5 |
| `khorikov-ut` | A/B | Vladimir Khorikov — *Unit Testing: Principles, Practices, and Patterns*; Manning, **2020** (принципы; примеры на C#, идеи переносятся) | M5 |
| `pro-git` | B | Scott Chacon, Ben Straub — *Pro Git*, **2nd edition** (свободно на git-scm.com) | F* / git |
| `patterns-dive` | B | Александр Швец — *Погружение в паттерны проектирования*, **v2021-2.33** (не путать с GoF) | F* 8–12 паттернов |
| `gof` | D/C | Erich Gamma et al. — *Design Patterns* (GoF) — словарь; практика у нас через Швеца | P2 словарь |
| `clean-code` | D | Robert C. Martin — *Clean Code* — только критически, рядом с `oust-psd` | не must |
| `code-complete` | D | Steve McConnell — *Code Complete*, **2nd edition** — энциклопедия-справка | не сюжет модуля |

### Данные и системы

| ID | Тир | Книга | Модули |
|----|-----|--------|--------|
| `morgunov-2018` | A | Е. П. Моргунов — *PostgreSQL. Основы языка SQL*, учеб. пособие; БХВ-Петербург, **2018**, ISBN 978-5-9775-4022-3 | M6 старт |
| `morgunov-2025` | A | Е. П. Моргунов — *PostgreSQL. Профессиональный SQL*, учеб. пособие; ДМК Пресс, **2025**, ISBN 978-5-93700-373-7 | M6 |
| `art-of-pg` | A | Dimitri Fontaine — *The Art of PostgreSQL*, **2nd edition, Updated 2026** (SQL как код приложения; Python-примеры) | M6 |
| `pg-inside16` | C | Егор Рогов — *PostgreSQL 16 изнутри*; Postgres Professional, Москва, **2024** | M6 опц. / P2 |
| `ddia` | A | Martin Kleppmann, Chris Riccomini — *Designing Data-Intensive Applications*, **2nd edition, 2026**, O’Reilly, ISBN 978-1-098-11906-5 | после M6; SD / M7–M8 |
| `elastic` | B | Анураг Шривастава — *Elasticsearch для разработчиков*, **2-е изд., 2025** | M8 |
| `release-it` | B/C | Michael T. Nygard — *Release It!*, **2nd edition** (таймауты, каскады, устойчивость) | SD / P2 |
| `fsa` | C | Mark Richards, Neal Ford — *Fundamentals of Software Architecture* (свежее издание) | SD / P2 |
| `se-google` | C | Titus Winters, Tom Manshreck, Hyrum Wright — *Software Engineering at Google* (свободно онлайн) | P2 |

### Прочее по стеку курса

| ID | Тир | Книга | Модули |
|----|-----|--------|--------|
| `graphql` | C | Алекс Бэнкс, Ева Порселло — *GraphQL: язык запросов для современных веб-приложений*; Питер / O’Reilly, **2019**, ISBN 978-5-4461-1143-5 | M7 опц. |
| `ai-huyen` | C | Chip Huyen — *AI Engineering* (продукты на моделях — не «как жить с Copilot») | P2 спец. |

---

## Онлайн (не книги, но канон слоя)

| Источник | Роль |
|----------|------|
| PostgresPro DEV1 / DEV2 / QPT / DBS | M6 практика; в главах — код курса + тема площадки |
| [exploring-os-ru](https://github.com/SillyHatsOnly/exploring-os-ru) | M4a / F* выборочно |
| [yakimka/questions.md](https://github.com/yakimka/python_interview_questions/blob/master/questions.md) | checklist; трекер [yakimka.md](./yakimka.md) |
| OSTEP, Pro Git, SE at Google, Crafting Interpreters, Nand2Tetris, MCS MIT | свободные опоры соответствующих ID |

---

## Полка ядра (опоры в наличии у читателя)

Локальные файлы **не** перечисляем. Карта глав: [reading-map](./reading-map.md).

Опоры, уже заведённые в канон для ядра: `skiena`, `ddia` (2e), `oust-psd`,
`feathers-legacy`, `pragmatic`, `khorikov-ut`, `refactoring`, `fluent-py`,
`art-of-pg` (Updated 2026), `kurose` (9e).

Бесплатно рядом: `ostep`, `pro-git`, куски `se-google`, exploring-os-ru.
Stevens / CSAPP / Petzold / Моргунов — по прежнему каталогу.

---

## Чеклист автору главы

- [ ] ID и название как в таблице (не имя файла)
- [ ] Тир не повышен молча: новый A → правка [sources-canon](./sources-canon.md)
- [ ] Издание / год / ISBN, если известны
- [ ] Глава или § (страница — при первом проходе)
- [ ] Пересказ, не цитата абзацем
- [ ] Не требовать «прочитай книгу целиком» вместо домашки модуля
