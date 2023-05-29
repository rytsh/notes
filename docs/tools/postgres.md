# Postgres

Check details in here https://hub.docker.com/_/postgres

Set a data path for the database.

```sh
mkdir -p ~/data/postgres
```

```sh
docker run -d --restart always --name postgres \
    -p 5432:5432 \
    -e POSTGRES_USER="awesome" \
    -e POSTGRES_PASSWORD="awesome" \
    -v ${HOME}/data/postgres:/var/lib/postgresql/data \
    postgres:15.2-alpine3.17
```

For precommand, run this after that:

```sh
/usr/local/bin/docker-entrypoint.sh postgres
```
