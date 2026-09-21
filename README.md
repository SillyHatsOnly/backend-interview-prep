# Backend Interview Prep

Публичный курс подготовки к собеседованиям **middle / middle+** Python backend
(рынок РФ + крупные компании). Без персональных данных.

Сайт (GitHub Pages):
`https://sillyhatsonly.github.io/backend-interview-prep/`

Локально:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-docs.txt
mkdocs serve
```

## Как устроен репозиторий

| Путь | Зачем |
|------|--------|
| [docs/plan/](docs/plan/) | План ядра ~8 мес + фаза 2 + Harbor |
| [`docs/canon/`](docs/canon/) | Стиль и правила генерации материалов |
| [`docs/modules/`](docs/modules/) | Учебные модули (теория + домашка) |
| [`docs/mocks/`](docs/mocks/) | Срезы-моки по материалу |
| [`docs/solutions/`](docs/solutions/) | Эталоны **после** ревью |
| [`docs/reference/`](docs/reference/) | Книги, SD, yakimka, ОС, git, K8s |

Скелет старого интенсива ~5 недель — только идея калибровки темпа:
[legacy-5w](docs/reference/legacy-5w.md).

## Бюджет

Пн / ср / пт · 3–4 часа до работы · ~10–12 ч/неделя.  
Выходные — отдых или короткое повторение.

## Что не кладём в git

- PDF книг (библиография + глава/§/стр + пересказ; `docs/reference/books.md`)
- Эталоны **до** ревью (после сдачи — в `docs/solutions/`)
- Секреты, персональные данные, прогресс читателя
