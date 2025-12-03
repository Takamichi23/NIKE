# Stage 1: build environment
FROM python:3.12-slim as builder
WORKDIR /app

RUN pip install --upgrade pip wheel setuptools

COPY requirements.txt .
RUN pip wheel --wheel-dir=/wheels -r requirements.txt

COPY . .

# Stage 2: runtime image
FROM python:3.12-slim
WORKDIR /app

RUN useradd -m appuser

COPY --from=builder /wheels /wheels
COPY --from=builder /app/requirements.txt /app/requirements.txt
RUN pip install --no-index --find-links=/wheels -r requirements.txt

COPY --from=builder /app /app

USER appuser
ENV PYTHONUNBUFFERED=1

EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
