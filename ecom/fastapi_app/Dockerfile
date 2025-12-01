FROM python:3.12-slim

WORKDIR /app

# Copy only the FastAPI folder into the container
COPY . /app

# Install dependencies from the requirements inside fastapi folder
RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
