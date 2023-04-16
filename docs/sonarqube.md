# Sonarqube

Check the details in here https://hub.docker.com/_/sonarqube

Recommanded values:

```sh
sysctl -w vm.max_map_count=524288
sysctl -w fs.file-max=131072
ulimit -n 131072
ulimit -u 8192
```

Create a folder for sonarqube, I changed the owner to 1000:1000 because I container is running with that user.

```sh
mkdir -p ~/data/sonarqube

mkdir -p ~/data/sonarqube/conf
mkdir -p ~/data/sonarqube/data
mkdir -p ~/data/sonarqube/logs
mkdir -p ~/data/sonarqube/extensions

chown -R 1000:1000 ~/data/sonarqube
```

Create sonar database before to start the container.

I took 172.17.0.1 for postgres, could be different in your case!

```sh
docker run -d --restart always --name sonarqube \
    -p 9000:9000 \
    -e SONAR_JDBC_USERNAME=awesome \
    -e SONAR_JDBC_PASSWORD=awesome \
    -e SONAR_JDBC_URL=jdbc:postgresql://172.17.0.1:5432/sonar \
    -v ${HOME}/data/sonarqube/conf:/opt/sonarqube/conf \
    -v ${HOME}/data/sonarqube/data:/opt/sonarqube/data \
    -v ${HOME}/data/sonarqube/logs:/opt/sonarqube/logs \
    -v ${HOME}/data/sonarqube/extensions:/opt/sonarqube/extensions \
    sonarqube:10.0.0-community
```

If you put the extensions (jar file) in the extensions folder, restart the container.

Open http://localhost:9000, username and password is admin, admin.

Go to administration > security > users > create user, I created a user with name sonar.

After that create a token for sonar user, I created a token with name ci.

## Golang

```sh
go install gotest.tools/gotestsum@latest
```

Now open a go project and run this command to send reports to sonarqube.

```sh
gotestsum --format standard-verbose --junitfile report-junit.xml --jsonfile report-test.json -- -race -cover -coverpkg=./... -covermode=atomic -coverprofile=coverage.out ./...

GOPATH="$(dirname ${PWD})" golangci-lint run --out-format checkstyle > report-lint.xml
```

Configurations

```sh
# go test -coverprofile=coverage.out
sonar.go.coverage.reportPaths
# go test -json > report-test.json
sonar.go.tests.reportPaths
# GOPATH="$(dirname ${PWD})" golangci-lint run --out-format checkstyle > report-lint.xml
sonar.go.golangci-lint.reportPaths
```

Run sonarscanner, paste sonarqube url like `http://172.17.0.1:9000` with your token also unique project-key.

```sh
docker run --rm -t -u $(id -u):$(id -g) \
    -e SONAR_HOST_URL="http://${SONARQUBE_URL}" \
    -e SONAR_SCANNER_OPTS="-Dsonar.projectKey=${YOUR_PROJECT_KEY} -Dsonar.go.coverage.reportPaths=./coverage.out -Dsonar.go.tests.reportPaths=./report-test.json -Dsonar.go.golangci-lint.reportPaths=./report-lint.xml -Dsonar.exclusions=**/*_test.go,**/vendor/**,**/*.sql,**/*test*/**" \
    -e SONAR_TOKEN="myAuthenticationToken" \
    -v ${PWD}:/usr/src \
    sonarsource/sonar-scanner-cli
```
