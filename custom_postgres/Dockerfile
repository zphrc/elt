FROM python:3.11-slim

RUN pip install --no-cache-dir dbt-postgres

ENV DBT_PROFILES_DIR=/root

WORKDIR /dbt

ENTRYPOINT ["dbt"]