FROM python:3.12-slim-bullseye

COPY . /app

WORKDIR /app

RUN python3 -m vevn /opt/venv

RUN pip install pip --upgrade

RUN /opt/venv/bin/pip install -r requirements.txt

RUN chmod +x entrypoint.sh

CMD [ "/app/entrypoint.sh" ]
=================================================
docker build -t django-k8s:v1 -f Dockerfile ./web
=================================================
version: '3.9'
services:
  web:
    depends_on:
      - postgres_db
    build:
      context: ./web
      dockerfile: Dockerfile
    image: django-k8s:v1
    environment:
      - PORT=8020
    env_file:
      - web/.env
    ports:
      - "8001:8020"
    command: sh -c "chmod +x /app/migrate.sh && sh /app/migrate.sh && /app/entrypoint.sh"
    volumes:
      - staticfiles:/app/staticfiles
  postgres_db:
    image: postgres
    restart: always
    command: -p 5434
    env_file:
      - web/.env
    expose:
      - 5434
    ports:
      - "5434:5434"
    volumes:
      - postgres_data:/var/lib/postgresql/data/
  redis_db:
    image: redis
    restart: always
    expose:
      - 6388
    ports:
      - "6388:6388"
    volumes:
      - redis_data:/data
    entrypoint: redis-server --appendonly yes --port 6388


volumes:
  staticfiles:
      external: true
  postgres_data:
  redis_data: