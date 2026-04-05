# ELT Pipeline with Docker, PostgreSQL, and dbt

A containerized ELT (Extract, Load, Transform) pipeline that extracts data from a source PostgreSQL database, loads it into a destination PostgreSQL database, and transforms it using [dbt](https://www.getdbt.com/) — all orchestrated with Docker Compose.

Built while following freeCodeCamp.org's [Data Engineering Course for Beginners](https://youtu.be/PHsC_t0j1dU?si=MOOuzkemvdXU1uS8).

## Architecture

```
┌──────────────────┐       pg_dump       ┌───────────────────────┐
│ Source PostgreSQL│  ──────────────►    │                       │
│   (port 5433)    │                     │  ELT Script (Python)  │
└──────────────────┘                     │                       │
                                         └───────────┬───────────┘
                                                     │ psql
                                                     ▼
                                         ┌───────────────────────┐
                                         │   Dest PostgreSQL     │
                                         │     (port 5434)       │
                                         └───────────┬───────────┘
                                                     │
                                                     ▼
                                         ┌───────────────────────┐
                                         │    dbt (Transform)    │
                                         │   dbt-postgres:1.9    │
                                         └───────────────────────┘
```

All four services run on a shared Docker bridge network (`elt_network`).

## Project Structure

```
elt/
├── docker-compose.yaml
├── elt/
│   ├── Dockerfile
│   └── elt_script.py
├── source_db_init/
│   └── init.sql
└── custom_postgres/              # dbt project
    ├── dbt_project.yml
    ├── models/
    │   └── example/
    │       ├── sources.yml       # Source definitions
    │       ├── schema.yml        # Model schemas & tests
    │       ├── actors.sql
    │       ├── films.sql
    │       ├── film_actors.sql
    │       ├── film_ratings.sql
    │       └── specific_movie.sql
    └── macros/
        └── film_ratings_macro.sql
```

| Path | Description |
|---|---|
| `docker-compose.yaml` | Defines source DB, destination DB, ELT script, and dbt services |
| `elt/elt_script.py` | Python script that waits for Postgres readiness, then runs `pg_dump` and `psql` |
| `elt/Dockerfile` | Builds the ELT runner image with Python 3.12 and PostgreSQL client 17 |
| `source_db_init/init.sql` | Seeds the source database with sample `users`, `films`, `film_category`, `actors`, and `film_actors` tables |
| `custom_postgres/` | dbt project for data transformations |

## dbt Models

| Model | Description |
|---|---|
| `actors` | Staging model for actor data |
| `films` | Staging model for film data |
| `film_actors` | Staging model for film-actor relationships |
| `film_ratings` | Aggregates film data with rating categories and actor lists |
| `specific_movie` | Filters films by a specific title (e.g., "Inception") |

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Getting Started

1. **Clone the repository**

   ```bash
   git clone https://github.com/zphrc/elt.git
   cd elt
   git checkout dbt
   ```

2. **Set up dbt profile**

   Create `~/.dbt/profiles.yml`:

   ```yaml
   custom_postgres:
     target: dev
     outputs:
       dev:
         type: postgres
         host: destination_postgres
         user: postgres
         password: secret
         port: 5432
         dbname: destination_db
         schema: public
   ```

3. **Start the pipeline**

   ```bash
   docker compose up --build
   ```

4. **Verify dbt models were created**

   ```bash
   docker exec -it elt-destination_postgres-1 psql -U postgres -d destination_db -c "SELECT * FROM film_ratings LIMIT 5;"
   ```

5. **Run dbt manually (optional)**

   ```bash
   docker compose run dbt run
   docker compose run dbt test
   ```

6. **Shut down**

   ```bash
   docker compose down -v
   ```

## Roadmap

- [x] Add [dbt](https://www.getdbt.com/) for data transformations
- [ ] Schedule pipeline runs with cron
- [ ] Orchestrate with [Apache Airflow](https://airflow.apache.org/)
- [ ] Integrate [Airbyte](https://airbyte.com/) for data ingestion

## Acknowledgements

- [Data Engineering Course for Beginners](https://youtu.be/PHsC_t0j1dU?si=MOOuzkemvdXU1uS8) by freeCodeCamp.org

## Author

**Zophia Maureen Roca** — [@zphrc](https://github.com/zphrc)
