# -------- Stage 1: Builder --------
FROM python:3.10-slim AS builder

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# Install build dependencies only
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

RUN pip install --upgrade pip \
    && pip install --no-cache-dir --user -r requirements.txt


# -------- Stage 2: Final --------
FROM python:3.10-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# Create non-root user
RUN addgroup --system app && adduser --system --group app

WORKDIR /app

# Install only runtime dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq-dev \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Copy installed python packages from builder
COPY --from=builder /root/.local /home/app/.local

# Update PATH
ENV PATH=/home/app/.local/bin:$PATH

# Copy project
COPY . .

# Collect static
RUN python manage.py collectstatic --noinput

# Create log directory
RUN mkdir -p /app/restapi/log \
    && touch /app/restapi/log/api.log

# Set ownership
RUN chown -R app:app /app

USER app

EXPOSE 8000

CMD ["gunicorn", "lms_main.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "3", "--timeout", "120"]
