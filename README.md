To get this running:

1. Install/update latest [Docker](https://www.docker.com/get-started) and [docker-compose](https://docs.docker.com/compose/install/)
2. Create a file `postgres-passwd` (no extension) in the `docker/postgis` directory of the project which contains
the text of the password you would like to set
3. Create a file `.env` in the root directory of the project to set `POSTGRES_PORT`, `OMOP_DBMS`, and `OMOP_JDBC_CONNECTION_STRING`
  - example `.env`
    ```
    POSTGRES_PORT=5432
    OMOP_DBMS='sql server'
    OMOP_JDBC_CONNECTION_STRING='jdbc:sqlserver://localhost:1433;database=OMOP;user=omop_gis_etl;password=password'
    ```
  - example `.env` without connecting to an OMOP instance
    ```
    POSTGRES_PORT=5432
    OMOP_DBMS=''
    OMOP_JDBC_CONNECTION_STRING=''
    ```
4. Build the loader image
  - `docker run --privileged -it --rm -v <absolute path to working directory>/docker/loader/:/src lnl7/nix sh -c 'nix-build /src && cp result /src/'`
  - `docker load < docker/loader/result`
5. `docker-compose up`

`docker-compose down` to stop.

`docker volume rm gis_pgdata` to delete the database (start all over with `docker-compose up`)

## Loading data
`docker exec -it -w /src/SVI gis_loader sh load.sh`

## Geocoding
1. Load tiger data (this takes a long time ~12 hrs and requires ~100 GB)
  - `docker exec -it -w /src/TIGER gis_loader sh load.sh`
2. Load OMOP location table into postgis
  - `docker exec -it -w /src/OMOP gis_loader R --vanilla -f load_location.R`
3. Geocode (This runs at about 700 records per minute after 3 minute startup time on a powerful laptop)
  - `docker exec -it -w /src/OMOP gis_loader sh geocode.sh`

## Loading EPA AQS data
`docker exec -it -w /src/EPA gis_loader Rscript load.R`