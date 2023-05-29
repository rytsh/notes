# GoProxy

We use goproxy in a corporate environment where access to the public internet is restricted.  
pkg.go.dev locally need to be configured to use a proxy.

> Gitlab has issue to finding the repo if inside a private group.
> So always add the `.git` as module name extensions.

## Athens

Check https://github.com/gomods/athens

Create a folder for the athens data:

```sh
mkdir -p ~/data/goproxy
```

Record configuration to goproxy folder (https://docs.gomods.io/configuration/download/)

```sh
cat <<EOF > data/goproxy/athens.hcl
downloadURL = "https://proxy.golang.org"

# redirect to not store modules
mode = "sync"

download *.mycompany.com/* {
    mode = "sync"
}

EOF
```

After that we can copy to gitconfig, sshconfig and ssh private key to that folder.

```sh
cat <<EOF > data/goproxy/sshconfig
Host *.mycompany.com
    HostName %h
    StrictHostKeyChecking no
    IdentityFile /root/.ssh/id_rsa
EOF
```

```sh
cat <<EOF > data/goproxy/gitconfig
[url "ssh://git@git.mycompany.com:"]
    insteadOf = https://git.mycompany.com
[url "ssh://git@git2.mycompany.com:"]
    insteadOf = https://git2.mycompany.com
EOF
```

```sh
cp ~/.ssh/id_rsa ~/data/goproxy/id_rsa
chown 0:0 ~/data/goproxy/id_rsa
chown 0:0 ~/data/goproxy/sshconfig
```

Run athens

```sh
docker run -d --restart always --name athens-proxy \
    -p 4444:3000 \
    -v ${HOME}/data/goproxy/athens:/var/lib/athens \
    -v ${HOME}/data/goproxy/sshconfig:/root/.ssh/config:ro \
    -v ${HOME}/data/goproxy/gitconfig:/root/.gitconfig:ro \
    -v ${HOME}/data/goproxy/id_rsa:/root/.ssh/id_rsa:ro \
    -v ${HOME}/data/goproxy/athens.hcl:/root/athens.hcl:ro \
    -e ATHENS_STORAGE_TYPE="disk" \
    -e ATHENS_DISK_STORAGE_ROOT="/var/lib/athens" \
    -e ATHENS_DOWNLOAD_MODE="file:/root/athens.hcl" \
    -e ATHENS_NETWORK_MODE="fallback" \
    -e ATHENS_GONOSUM_PATTERNS="*.mycompany.com/*" \
    gomods/athens:v0.12.0
```

After that use like that in your `.bashrc` file:

```sh
export GOPROXY=http://localhost:4444
export GONOSUMDB=*.mycompany.com/*
```
