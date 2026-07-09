FROM python:3.13-alpine

WORKDIR /app

COPY index.html .

EXPOSE 8081

CMD ["python3", "-m", "http.server", "8081"]
