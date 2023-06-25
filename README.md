[![Continuous integration](https://github.com/solectrus/sungrow-importer/actions/workflows/push.yml/badge.svg)](https://github.com/solectrus/sungrow-importer/actions/workflows/push.yml)
[![wakatime](https://wakatime.com/badge/user/697af4f5-617a-446d-ba58-407e7f3e0243/project/f8429171-106d-4ef2-877c-b5fd73495e6e.svg)](https://wakatime.com/badge/user/697af4f5-617a-446d-ba58-407e7f3e0243/project/f8429171-106d-4ef2-877c-b5fd73495e6e)

# Sungrow importer

Import CSV data downloaded from portaleu.isolarcloud.com and push it to InfluxDB.

## Requirements

- SOLECTRUS installed and running
- CSV files downloaded from portaleu.isolarcloud.com

## Usage

- Login to your host machine where SOLECTRUS is running
- CD into the folder where the .env of SOLECTRUS file is located
- Create a folder `csv` and put the CSV files into it (subfolders allowed)
- Run the following command:

```bash
docker run -it --rm \
           --env-file .env \
           --mount type=bind,source="$PWD/csv",target=/data,readonly \
           --network=solectrus_default \
           ghcr.io/solectrus/sungrow-importer
```

(Name of the network may vary, see `docker network ls`)

This imports all CSV files from the folder `./csv` (it uses $PWD because Docker requires an absolute path here) and pushes them to your InfluxDB.
The process is idempotent, so you can run it multiple times without any harm.

First Note: If the import is performed after SOLECTRUS has already been used, caching issues may occur, meaning that older periods will not be displayed. In this case, the Redis cache must be flushed once after the import:

```bash
docker exec -it solectrus-redis-1 redis-cli FLUSHALL
```

(Name of the Redis container may vary, see `docker ps`)

Second note: Check the `.env` variable `INSTALLATION_DATE`. This must be set to the day your PV system was installed.

## ENV variables

| Variable                               | Description                                     | Default |
| -------------------------------------- | ----------------------------------------------- | ------- |
| `INFLUX_HOST`                          | Hostname of InfluxDB                            |         |
| `INFLUX_SCHEMA`                        | Schema (http/https) of InfluxDB                 | `http`  |
| `INFLUX_PORT`                          | Port of InfluxDB                                | `8086`  |
| `INFLUX_TOKEN_WRITE` or `INFLUX_TOKEN` | Token for InfluxDB (requires write permissions) |         |
| `INFLUX_ORG`                           | Organization for InfluxDB                       |         |
| `INFLUX_BUCKET`                        | Bucket for InfluxDB                             |         |
| `INFLUX_MEASUREMENT`                   | Measurement for InfluxDB                        |         |
| `INFLUX_OPEN_TIMEOUT`                  | Timeout for InfluxDB connection (in seconds)    | `30`    |
| `INFLUX_READ_TIMEOUT`                  | Timeout for InfluxDB read (in seconds)          | `30`    |
| `INFLUX_WRITE_TIMEOUT`                 | Timeout for InfluxDB write (in seconds)         | `30`    |
| `IMPORT_FOLDER`                        | Folder where CSV files are located              | `/data` |
| `IMPORT_PAUSE`                         | Pause after each imported file (in seconds)     | `0`     |

## License

Copyright (c) 2023 Georg Ledermann, released under the MIT License

Based on some work by [Rainer Drexler](https://github.com/holiday-sunrise/)
