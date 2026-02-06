FROM python:3.11.14-alpine
WORKDIR /app
EXPOSE 8080
COPY main.py .
COPY server_8080.py .
RUN pip install --no-cache-dir uvicorn fastapi
CMD ["python", "server_8080.py"]
