# ChronosAtlas – Dev Environment Setup (Phase 1 Alignment)

This document consolidates the Django + PostgreSQL containerized development environment, aligned with the **Chronos-Atlas blueprint** and leaving room for future phases (Celery/Redis, observability, scaling).

---

## 1. Dockerfile

```dockerfile
FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# System dependencies for psycopg2 and other libs
RUN apt-get update \
    && apt-get install -y build-essential libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --upgrade pip \
    && pip install -r requirements.txt

COPY . .

EXPOSE 8000

ENTRYPOINT ["/app/entrypoint.sh"]
```

---

## 2. docker-compose.yml

```yaml
version: "3.9"

services:
  db:
    image: postgres:14-alpine
    container_name: chronosatlas_db
    restart: unless-stopped
    environment:
      POSTGRES_DB: chronosatlas
      POSTGRES_USER: chronos_user
      POSTGRES_PASSWORD: chronos_pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  api:
    build: .
    container_name: chronosatlas_api
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      DB_NAME: chronosatlas
      DB_USER: chronos_user
      DB_PASSWORD: chronos_pass
      DB_HOST: db
      DB_PORT: 5432

  # Phase 2 (MVP) – placeholders
  # redis:
  #   image: redis:7-alpine
  #   container_name: chronosatlas_redis
  #   restart: unless-stopped
  #
  # worker:
  #   build: .
  #   command: celery -A ChronosAtlas worker -l info
  #   volumes:
  #     - .:/app
  #   depends_on:
  #     - db
  #     - redis

volumes:
  postgres_data:
```

---

## 3. requirements.txt

```
Django>=4.2
psycopg2-binary>=2.9
graphene-django>=3.0
python-decouple>=3.8

# Phase 2 (to be uncommented later)
# celery>=5.3
# redis>=5.0
```

---

## 4. entrypoint.sh

```bash
#!/bin/sh

set -e

echo "Applying database migrations..."
python manage.py migrate --noinput

echo "Collecting static files..."
python manage.py collectstatic --noinput

echo "Starting Django development server..."
exec "$@"
```

⚠️ Don’t forget to make it executable:
```bash
chmod +x entrypoint.sh
```

---

## 5. settings.py (snippet)

```python
import os
from decouple import config
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = config("SECRET_KEY", default="insecure-secret-key")
DEBUG = config("DEBUG", default=True, cast=bool)
ALLOWED_HOSTS = config("ALLOWED_HOSTS", default="*", cast=lambda v: v.split(","))

INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "graphene_django",
    # project apps...
]

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": config("DB_NAME", default="chronosatlas"),
        "USER": config("DB_USER", default="chronos_user"),
        "PASSWORD": config("DB_PASSWORD", default="chronos_pass"),
        "HOST": config("DB_HOST", default="db"),
        "PORT": config("DB_PORT", default=5432, cast=int),
    }
}

GRAPHENE = {
    "SCHEMA": "ChronosAtlas.schema.schema",
}

STATIC_URL = "/static/"
STATIC_ROOT = BASE_DIR / "staticfiles"
MEDIA_URL = "/media/"
MEDIA_ROOT = BASE_DIR / "media"
DEFAULT_AUTO_FIELD = "django.db.models.BigAutoField"
```

---

## 6. .env (local dev)

```
SECRET_KEY=dev-secret-key
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

DB_NAME=chronosatlas
DB_USER=chronos_user
DB_PASSWORD=chronos_pass
DB_HOST=db
DB_PORT=5432
```

---

## ✅ Summary

- **Phase 1 (Foundation)** → Django + Postgres are fully containerized and ready for development.  
- **Phase 2 (MVP)** → Redis & Celery placeholders already exist in docker-compose and requirements, so future work won’t conflict.  
- **Phase 3 (Production)** → Entrypoint script is extendable (can add gunicorn, monitoring, etc.).  
