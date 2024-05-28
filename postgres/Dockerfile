FROM postgres:14.1-alpine

ENV POSTGRES_DB=db

COPY ./docker-entrypoint-initdb.d/ /docker-entrypoint-initdb.d/
