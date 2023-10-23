# Rady Ansible Servers using Docker Compose

## Contents

## 1. Introduction

In this project we will run using docker-compose a few servers ready to play with using Ansible. The way we have configured the image and the servers from scratch is in the following [GitHub Project - SSH docker container](https://github.com/titocampis/ssh-docker-container)

## X. Start the service
### X.1 Start just one server 
To start just one server, we have to tell docker-compose to run only one of the services:
```bash
docker compose up -d ubuntu1
```

### X.2 Start a couple of servers
```bash
docker compose up -d ubuntu1 ubuntu2
```

### X.3 Start all the servers
```bash
docker compose up -d
```