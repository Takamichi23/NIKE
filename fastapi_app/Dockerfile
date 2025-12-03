# Stage 1: build environment
FROM python:3.12-slim as builder
WORKDIR /app
# Install build deps if any (wheel, setuptools)
RUN pip install --upgrade pip
COPY requirements.txt .
RUN pip wheel --wheel-dir=/wheels -r requirements.txt

# Copy application source
COPY . .

# Stage 2: runtime image
FROM python:3.12-slim
WORKDIR /app
# create non-root user
RUN useradd -m appuser
COPY --from=builder /wheels /wheels
RUN pip install --no-index --find-links=/wheels -r requirements.txt
COPY --from=builder /app /app
USER appuser
ENV PYTHONUNBUFFERED=1
# Expose your API port (adjust)
EXPOSE 8000
# Example entrypoint (adjust to how you run your app)
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
