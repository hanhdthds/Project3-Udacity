FROM python:3.8-slim-bookworm

USER root

WORKDIR /src

COPY ./requirements.txt requirements.txt

RUN apt update -y && apt install -y build-essential libpq-dev
RUN pip install --upgrade pip setuptools wheel

ENV DB_USERNAME=myuser
ENV DB_PASSWORD=mypassword
ENV DB_HOST=127.0.0.1
ENV DB_PORT=5432
ENV DB_NAME=mydatabase

RUN pip install -r requirements.txt

COPY . .

CMD python app.py