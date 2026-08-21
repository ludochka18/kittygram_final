# Kittygram

[![Kittygram workflow](https://github.com/ludochka18/kittygram_final/actions/workflows/main.yml/badge.svg?branch=main)](https://github.com/ludochka18/kittygram_final/actions/workflows/main.yml)

Kittygram — веб-приложение для публикации информации о котах.

Пользователи могут работать с данными через веб-интерфейс, а приложение состоит из отдельных frontend и backend частей. Проект контейнеризирован с помощью Docker и автоматически тестируется и разворачивается через GitHub Actions.

Проект доступен по адресу:
https://kittygram-luda.duckdns.org

## Технологии

### Backend
- Python 3.12
- Django
- Django REST Framework
- Djoser
- PostgreSQL
- Gunicorn

### Frontend
- React
- Node.js

### Инфраструктура
- Docker
- Docker Compose
- Nginx
- GitHub Actions
- Docker Hub

## Архитектура

Проект состоит из четырёх Docker-контейнеров:

- `backend` — Django-приложение;
- `frontend` — React-приложение;
- `gateway` — Nginx;
- `db` — PostgreSQL.

Для хранения данных используются Docker volumes:

- `pg_data` — данные PostgreSQL;
- `static` — статические файлы frontend и backend;
- `media` — пользовательские медиафайлы.

## Локальный запуск

Клонируйте репозиторий:

```bash
git clone https://github.com/ludochka18/kittygram_final.git
```

Перейдите в директорию проекта:

```bash
cd kittygram_final
```

Создайте файл `.env` в корне проекта.

Пример содержимого:

```env
SECRET_KEY=your_secret_key

POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=your_password
DB_HOST=db
DB_PORT=5432

DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
```

Соберите и запустите контейнеры:

```bash
docker compose up --build
```

После запуска приложение будет доступно по адресу:

```text
http://localhost:9000
```

## Запуск в production

Для production используется файл:

```text
docker-compose.production.yml
```

На сервере должны быть установлены Docker и Docker Compose.

Создайте директорию проекта и перейдите в неё:

```bash
mkdir -p ~/kittygram
cd ~/kittygram
```

Поместите в эту директорию файл `docker-compose.production.yml`.

Создайте файл `.env` с переменными окружения:

```env
SECRET_KEY=your_secret_key
POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=your_password
DB_HOST=db
DB_PORT=5432
DEBUG=False
ALLOWED_HOSTS=your_domain
```

Загрузите актуальные Docker-образы:

```bash
docker compose -f docker-compose.production.yml pull
```

Запустите контейнеры:

```bash
docker compose -f docker-compose.production.yml up -d
```

Выполните миграции:

```bash
docker compose -f docker-compose.production.yml exec backend python manage.py migrate
```

Соберите статические файлы backend:

```bash
docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic --noinput
```

Перенесите собранную статику в общий Docker volume:

```bash
docker compose -f docker-compose.production.yml exec backend mkdir -p /backend_static/static
docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```

Gateway приложения доступен на порту `9000`.

Внешний Nginx на сервере должен проксировать запросы к Kittygram на:

```text
http://127.0.0.1:9000
```

В проекте основной production-деплой выполняется автоматически через GitHub Actions после успешного push в ветку `main`.

В production используются образы:

```text
ludochka/kittygram_backend
ludochka/kittygram_frontend
ludochka/kittygram_gateway
postgres:13
```

## CI/CD

Workflow находится в:

```text
.github/workflows/main.yml
```

При push в любую ветку GitHub Actions:

- проверяет backend на соответствие PEP8;
- запускает тесты backend;
- запускает тесты frontend.

При успешном push в ветку `main` дополнительно:

- собираются Docker-образы;
- образы отправляются на Docker Hub;
- новые образы загружаются на сервер;
- приложение перезапускается;
- выполняются миграции;
- собирается статика;
- после успешного деплоя отправляется уведомление в Telegram.

## Переменные окружения

Основные переменные:

| Переменная | Назначение |
|---|---|
| `SECRET_KEY` | секретный ключ Django |
| `POSTGRES_DB` | имя базы данных PostgreSQL |
| `POSTGRES_USER` | пользователь PostgreSQL |
| `POSTGRES_PASSWORD` | пароль PostgreSQL |
| `DB_HOST` | адрес базы данных |
| `DB_PORT` | порт PostgreSQL |
| `DEBUG` | режим отладки Django |
| `ALLOWED_HOSTS` | разрешённые хосты |

Настоящие секретные данные не должны попадать в GitHub. Для примера используется файл `.env.example`.

## Автор

Людмила Барсукова
GitHub: [ludochka18](https://github.com/ludochka18)
