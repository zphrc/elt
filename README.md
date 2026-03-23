# ELT Pipeline with Docker and PostgreSQL

A containerized ELT (Extract, Load, Transform) pipeline that extracts data from a source PostgreSQL database, loads it into a destination PostgreSQL database using `pg_dump` and `psql`, all orchestrated with Docker Compose.

Built while following freeCodeCamp.org's [Data Engineering Course for Beginners](https://youtu.be/PHsC_t0j1dU?si=MOOuzkemvdXU1uS8).

## Architecture

```
┌──────────────────┐       pg_dump       ┌───────────────────────┐
│ Source PostgreSQL│  ──────────────►    │                       │
│   (port 5433)    │                     │  ELT Script (Python)  │
└──────────────────┘                     │                       │
                                         └───────────┬───────────┘
┌──────────────────┐        psql                     │
│  Dest PostgreSQL │  ◄──────────────────────────────┘
│   (port 5434)    │
└──────────────────┘
```

All three services run on a shared Docker bridge network (`elt_network`).

## Project Structure

```
elt/
├── docker-compose.yaml
├── elt/
│   ├── Dockerfile
│   └── elt_script.py
└── source_db_init/
    └── init.sql
```

| Path | Description |
|---|---|
| `docker-compose.yaml` | Defines the source DB, destination DB, and ELT script services |
| `elt/elt_script.py` | Python script that waits for Postgres readiness, then runs `pg_dump` and `psql` |
| `elt/Dockerfile` | Builds the ELT runner image with Python 3.12 and PostgreSQL client 17 |
| `source_db_init/init.sql` | Seeds the source database with sample `users`, `films`, `film_category`, `actors`, and `film_actors` tables |

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Getting Started

1. **Clone the repository**

   ```bash
   git clone https://github.com/zphrc/elt.git
   cd elt
   ```

2. **Start the pipeline**

   ```bash
   docker compose up --build
   ```

3. **Verify the data was loaded**

   ```bash
   docker exec -it elt-destination_postgres-1 psql -U postgres -d destination_db -c "SELECT * FROM users;"
   ```

4. **Shut down**

   ```bash
   docker compose down
   ```

## Roadmap

- [ ] Add [dbt](https://www.getdbt.com/) for data transformations
- [ ] Schedule pipeline runs with cron
- [ ] Orchestrate with [Apache Airflow](https://airflow.apache.org/)
- [ ] Integrate [Airbyte](https://airbyte.com/) for data ingestion

## Acknowledgements

- [Data Engineering Course for Beginners](https://youtu.be/PHsC_t0j1dU?si=MOOuzkemvdXU1uS8) by freeCodeCamp.org

## Author

**Zophia Maureen Roca** — [@zphrc](https://github.com/zphrc)
