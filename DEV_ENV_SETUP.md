# ChronosAtlas – Professional Development Environment Setup

This document provides a comprehensive guide to setting up and running the ChronosAtlas development environment. The setup uses Docker and Docker Compose to ensure consistency and reliability.

---

## 1. Multi-Stage Dockerfile

We use a multi-stage `Dockerfile` to create optimized images. The `builder` stage installs dependencies and builds Python wheels, while the `final` stage creates a minimal runtime image, resulting in smaller image sizes and improved security.

```dockerfile
# --- STAGE 1: Builder (Optimized for Dependency Installation) ---
    FROM python:3.11-slim as builder

    # Set environment variables for Python optimization
    ENV PYTHONDONTWRITEBYTECODE 1
    ENV PYTHONUNBUFFERED 1
    
    # Install system dependencies and build tools needed for psycopg2
    RUN apt-get update \
        && apt-get install --no-install-recommends -y \
        gcc \
        libpq-dev \
        # Install dependencies needed for static files (if any)
        # and clean up in a single layer
        && pip install --upgrade pip
    
    # Set the working directory
    WORKDIR /app
    
    # Copy requirements file and install Python dependencies
    COPY requirements.txt .
    
    # Install dependencies and then remove build tools to keep the layer lean
    RUN pip wheel --no-cache-dir --no-deps --wheel-dir /usr/src/app/wheels -r requirements.txt \
        && apt-get purge -y --auto-remove gcc libpq-dev
    
    # --- STAGE 2: Final (Minimal Runtime Image) ---
    FROM python:3.11-slim as final
    
    # Set environment variables
    ENV PYTHONDONTWRITEBYTECODE 1
    ENV PYTHONUNBUFFERED 1
    
    # Set the working directory
    WORKDIR /app
    
    # Install runtime dependencies (libpq-dev dependencies without the dev headers)
    # FIX: Added 'postgresql-client' here, which provides the 'pg_isready' command
    RUN apt-get update \
        && apt-get install --no-install-recommends -y \
        libpq5 \
        postgresql-client \
        # Clean up APT cache to reduce image size
        && rm -rf /var/lib/apt/lists/*
    
    # Copy pre-built wheels from the builder stage
    COPY --from=builder /usr/src/app/wheels /wheels
    # Install packages from wheels
    RUN pip install --no-cache-dir /wheels/*
    
    # Copy the rest of the application code
    COPY . /app
    
    # Ensure entrypoint.sh is executable and copy it to the bin directory
    # Note: The entrypoint script path must match the ENTRYPOINT instruction below.
    COPY scripts/entrypoint.sh /usr/local/bin/entrypoint.sh
    RUN chmod +x /usr/local/bin/entrypoint.sh
    
    # Expose the application port
    EXPOSE 8000
    
    # Specify the default command to run the application
    # Use the correct path for the entrypoint script
    ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
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

## 7. Common Development Tasks

### Checking Logs

Since the development server runs in detached (`-d`) mode, you can use the `docker compose logs` command to view its output.

**1. View all logs at once:**
To see a snapshot of logs from all services (`api` and `db`):
```bash
docker compose -f docker-compose.dev.yml logs
```

**2. Follow logs in real-time (most common):**
To stream logs live as they happen, use the `-f` or `--follow` flag. This is the best way to monitor your application.
```bash
docker compose -f docker-compose.dev.yml logs -f
```
*(Press `Ctrl+C` to stop streaming.)*

**3. View logs for a specific service:**
If you only need to see the logs from the Django `api` container:
```bash
docker compose -f docker-compose.dev.yml logs -f api
```

---

## ✅ Summary

This setup provides a robust, containerized environment that mirrors a production deployment while offering the flexibility needed for local development, such as hot-reloading.
