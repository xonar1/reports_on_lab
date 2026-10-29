# 🌱 Habit Tracker

[![CI/CD](https://github.com/xonar1/habit-tracker/actions/workflows/ci-cd.yml/badge.svg)](https://github.com/xonar1/habit-tracker/actions/workflows/ci-cd.yml)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110%2B-009688)](https://fastapi.tiangolo.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Habit Tracker** — учебный fullstack-проект для отслеживания полезных привычек. Пользователь заводит привычки, ежедневно отмечает их выполнение, видит текущую серию (streak) и статистику.

Проект развивался поэтапно в рамках курса: от настройки Git и Jira до полноценного REST API с автоматическим деплоем.

---

## 📚 Этапы разработки

| Практика | Тема | Что сделано |
|----------|------|-------------|
| №1 | Agile / Jira | Декомпозиция на эпики и user story, Scrum-спринт и Kanban-поток |
| №2 | Git & GitHub | Работа с ветками, PR, разрешение конфликтов |
| №3 | Git в консоли | `init`, `merge`, `rebase`, конфликты, `reflog`, Git Flow vs Trunk-Based |
| №4 | GitHub Actions + Cloudflare | Первый CI/CD: тесты → деплой статики на Pages |
| №5 | FastAPI + CI/CD | REST API, матрица Python, кэш pip, покрытие, защита `main` |

---

## 🛠 Стек технологий

| Слой | Технологии |
|------|------------|
| Backend | Python 3.10–3.12, FastAPI, Pydantic, uvicorn |
| Тесты | pytest, pytest-cov, httpx (TestClient) |
| Линтер | ruff |
| CI/CD | GitHub Actions (матрица, кэш, артефакты, concurrency, paths) |
| Хостинг | Cloudflare Pages (фронтенд), Cloudflare Tunnel (демо API) |
| Управление | Jira (Scrum + Kanban) |

---

## 📁 Структура проекта

```
habit-tracker/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py          # FastAPI-приложение и эндпоинты
│   │   ├── schemas.py       # Pydantic-модели
│   │   └── streak.py        # бизнес-логика подсчёта серии
│   ├── tests/
│   │   ├── test_api.py
│   │   └── test_streak.py
│   ├── pytest.ini
│   ├── requirements.txt
│   └── requirements-dev.txt
├── web/
│   ├── index.html
│   └── style.css
├── .github/
│   ├── workflows/
│   │   └── ci-cd.yml
│   └── dependabot.yml
├── .gitignore
└── README.md
```

---

## 🚀 Быстрый старт

### 1. Клонировать репозиторий

```bash
git clone https://github.com/xonar1/habit-tracker.git
cd habit-tracker
```

### 2. Установить зависимости

```bash
cd backend
python -m venv venv
source venv/bin/activate          # Linux / macOS
# venv\Scripts\activate           # Windows

pip install -r requirements-dev.txt
```

### 3. Запустить API

```bash
uvicorn app.main:app --reload --port 8000
```

- API: <http://127.0.0.1:8000/health>
- Swagger UI: <http://127.0.0.1:8000/docs>
- ReDoc: <http://127.0.0.1:8000/redoc>

### 4. Запустить тесты

```bash
pytest -v
pytest -v --cov=app --cov-report=term-missing --cov-fail-under=80
```

---

## 🔌 API Endpoints

| Метод | Путь | Описание |
|-------|------|----------|
| `GET`  | `/health` | Проверка работоспособности |
| `POST` | `/habits` | Создать привычку |
| `GET`  | `/habits` | Список всех привычек |
| `POST` | `/habits/{id}/checkins` | Отметить выполнение (done: bool) |
| `GET`  | `/habits/{id}/streak` | Получить текущую серию |

### Примеры через `curl`

```bash
# Создать привычку
curl -X POST http://127.0.0.1:8000/habits \
  -H "Content-Type: application/json" \
  -d '{"name": "Morning run"}'

# Добавить отметку
curl -X POST http://127.0.0.1:8000/habits/1/checkins \
  -H "Content-Type: application/json" \
  -d '{"done": true}'

# Получить серию
curl http://127.0.0.1:8000/habits/1/streak
# → {"habit_id": 1, "streak": 1}
```

---

## ⚙️ CI/CD пайплайн

Файл `.github/workflows/ci-cd.yml` описывает полный пайплайн:

```
┌──────────┐     ┌──────────────┐     ┌──────────────┐
│   lint   │     │ test (3.10)  │     │ test (3.11)  │  ──┐
└──────────┘     └──────────────┘     └──────────────┘    │  needs: [lint, test]
                   ┌──────────────┐                        ├──────────────────▶ deploy (Pages)
                   │ test (3.12)  │  ─────────────────────┘
                   │ + coverage   │
                   └──────────────┘
```

**Что проверяется:**
- ✅ Линтер `ruff` — стиль и ошибки
- ✅ Тесты `pytest` на Python 3.10, 3.11, 3.12 (матрица)
- ✅ Покрытие кода ≥ 80% (артефакт `coverage-html`)
- ✅ Кэш `pip` для ускорения сборок
- ✅ `concurrency` — отмена устаревших запусков
- ✅ `paths` — запуск только при изменении `backend/`, `web/` или workflow
- ✅ Защита `main` через Ruleset — merge заблокирован при красной проверке

---

## 🌐 Демо API через Cloudflare Tunnel

Чтобы показать API с другого устройства (например, с телефона через мобильный интернет):

```bash
# Терминал 1 — запустить API
cd backend && uvicorn app.main:app --port 8000

# Терминал 2 — пробросить тоннель
cloudflared tunnel --url http://localhost:8000
```

В выводе `cloudflared` появится адрес вида `https://xxx-xxx-xxx.trycloudflare.com` — по нему API доступен из интернета, включая `/docs`.

> ⚠️ Быстрый тоннель не защищён паролем и не подходит для продакшена.

---

## 📊 Покрытие тестами

Отчёт о покрытии сохраняется как артефакт GitHub Actions (`coverage-html`). Локально:

```bash
pytest --cov=app --cov-report=html
# открыть htmlcov/index.html в браузере
```

---

## 🧩 Ключевые решения

**Почему `deploy` зависит от `test` через `needs`?**  
Потому что `needs:` — это явная зависимость в графе выполнения GitHub Actions. Если `test` упадёт, job `deploy` не запустится вовсе. Без этого сломанный код моментально оказывался бы у пользователей.

**Почему матрица Python, а не три одинаковых job'а?**  
Матрица устраняет дублирование конфигурации и гарантирует, что все версии тестируются одинаково. Добавление Python 3.13 — одна строка, а не копипаста целого блока.

**Почему Cloudflare Tunnel, а не деплой backend из Actions?**  
Runner в GitHub Actions — временная VM, она исчезает после завершения job'а. Тоннель хорош там, где процесс живёт долго: на ноутбуке или постоянно работающем сервере.

---

## 📖 Отчёты по практикам

Подробные отчёты с скриншотами и рефлексией лежат в отдельном репозитории:

👉 **[college-reports](https://github.com/xonar1/college-reports)**

- `Lab1.md` — Agile, Scrum, Kanban в Jira
- `Lab3.md` — Git в консоли
- `Lab4.md` — GitHub Actions + Cloudflare Pages
- `Lab5.md` — FastAPI + углублённый CI/CD *(в процессе)*

---