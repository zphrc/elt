# custom_postgres

dbt project for transforming film and actor data in the ELT pipeline.

## Models

| Model | Type | Description |
|-------|------|-------------|
| `actors` | Staging | Actor data from source |
| `films` | Staging | Film data from source |
| `film_actors` | Staging | Film-actor relationships |
| `film_ratings` | Mart | Films with rating categories and aggregated actors |
| `specific_movie` | Utility | Filter films by title |

## Sources

- `destination_db.films`
- `destination_db.actors`
- `destination_db.film_actors`

## Usage

```bash
# Run all models
dbt run

# Run tests
dbt test

# Run a specific model
dbt run --select film_ratings
```

## Macros

- `generate_film_ratings()` — Generates the film ratings query with CTEs
