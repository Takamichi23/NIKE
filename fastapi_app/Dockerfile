# Stage 1: build environment
FROM python:3.12-slim as builder
<<<<<<< HEAD
WORKDIR /fastapi_app

# Upgrade pip and install build tools
RUN pip install --upgrade pip wheel setuptools

# Copy requirements and build wheels
COPY requirements.txt .
RUN pip wheel --wheel-dir=/wheels -r requirements.txt

# Copy application source
COPY . .

# Stage 2: runtime image
FROM python:3.12-slim
WORKDIR /fastapi_app

# Create non-root user
RUN useradd -m appuser

# Copy and install wheels
COPY --from=builder /wheels /wheels
COPY requirements.txt .
RUN pip install --no-index --find-links=/wheels -r requirements.txt

# Copy application code
COPY --from=builder /fastapi_app /fastapi_app

# Switch to non-root user
USER appuser
ENV PYTHONUNBUFFERED=1

# Expose the port FastAPI will run on
EXPOSE 8000

# Command to run FastAPI
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]

=======
WORKDIR /app

RUN pip install --upgrade pip wheel setuptools

COPY requirements.txt .
RUN pip wheel --wheel-dir=/wheels -r requirements.txt

COPY . .

# Stage 2: runtime image
FROM python:3.13.10-slim
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
>>>>>>> b141838556e44d40de503a9f359bae6138c3d60c
