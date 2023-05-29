# Go-PKG locally

Before to start, install [postgres](./postgres.md) and [goproxy](./goproxy.md) locally.

First clone the pkgsite repository

```sh
git clone git@github.com:golang/pkgsite.git
```

Go to pkgsite folder and build `frontend` and `db` binaries

```sh
CGO_ENABLED=0 go build -o frontend cmd/frontend/main.go
CGO_ENABLED=0 go build -o db devtools/cmd/db/main.go
```

For starting we need to create a database and migrate it

Let's create a folder and copy all required files to that folder

```sh
mkdir -p ~/data/gopkg

cp frontend ~/data/gopkg/frontend
cp db ~/data/gopkg/db

cp -r static ~/data/gopkg/static
cp -r migrations ~/data/gopkg/migrations
```

First we need to run database commands

Check connection configuration in https://github.com/golang/pkgsite/blob/master/internal/config/config.go#L381

```sh
export GO_DISCOVERY_DATABASE_HOST=localhost
export GO_DISCOVERY_DATABASE_USER=awesome
export GO_DISCOVERY_DATABASE_PASSWORD=awesome
./db create
./db migrate
```

Now we can start the frontend, configuration is same as db

```sh
./frontend -proxy_url=http://localhost:4444 -host 0.0.0.0:8080 -bypass_license_check
```

Now you can access the frontend at http://localhost:8080

Let's try to download a package, it could be private because we give permission in the go-proxy tool

Example go library github.com/rytsh/call and request to fetch it

```sh
curl -X POST -fkSL http://localhost:8080/fetch/github.com/rytsh/call
```
