---
title: "Cheatsheet - Docker / Podman"
categories: [Cheatsheets]
tags: [docker, podman]
---

This is series of "cheatsheet" articles, which gather tips, hacks, useful commands for given tool. 

## Commands

#### Run docker

```
docker run -it --rm -p 3000:3000 -e "DEFAULT_STEALTH=true" -v '/tmp/pdf:/usr/src/app/workspace' browserless/chrome:latest
```

`-it` interactive (runs in foreground)

`--rm` automatically removes container (when closed Ctrl+C)

`-p` map host port to container port

`-e` set environment variable

`-v` mounts host folder into container folder
 
#### Log into container:

```
docker exec -it 0d131117dbac bash
```


## Troubleshooting podman

#### No docker command

```
 echo "alias docker=podman" > ~/.zshrc && source ~/.zshrc
```

#### No "DOCKER_HOST"

```
systemctl --user start podman.service

export DOCKER_HOST=unix:///run/user/1000/podman/podman.sock

# ls /run/user/1000/podman/podman.sock
# if not exist either service didn't start or there is different path 
# see: podman info --format '{{.Host.RemoteSocket.Path}}'

# If docker-compose is used, then additionally:
sudo ln -s /run/user/1000/podman/podman.sock /var/run/docker.sock
```
Credit: https://docs.localstack.cloud/references/podman

#### Problem with mounting volumes on Fedora (or when SELinux policy is used)

Add `:Z` at the end.
```
docker run --volume="$PWD:/srv/jekyll:Z"
```
Credit: https://stackoverflow.com/questions/24288616/permission-denied-on-accessing-host-directory-in-docker/31334443#31334443