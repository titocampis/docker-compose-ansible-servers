# Rady Ansible Servers using Docker Compose

## Contents

## 1. Introduction

In this project we will run using docker-compose a few servers ready to play with using Ansible. The way we have configured the image and the servers from scratch is in the following [GitHub Project - SSH docker container](https://github.com/titocampis/ssh-docker-container)

## 2. Severs Configuration

### 2.1 Docker Images

#### 2.1.1 Ubuntu Image

The ubuntu server image configuration is in [Dockerfile](Dockerfile).

Pull the ubuntu:latest image:
```bash
docker pull ubuntu:latest
```

Build the image using the custom Dockerfile:
```bash
docker build -t ubuntu:ssh .
```

#### 2.1.2 Centos Image

The centos server image configuration is in [centos.Dockerfile](centos.Dockerfile).

Pull the centos/systemd:latest image:
```bash
docker pull centos/systemd:latest
```

Build the image using the custom Dockerfile:
```bash
docker build -t centos:ssh . -f centos.Dockerfile
```

### 2.2 Containers configuration - Docker Compose

We have configured 3 services in the [docker-compose.yaml](docker-compose.yaml):
- `ubuntu1`: ubuntu:latest image with user and ssh service configured
- `ubuntu2`: ubuntu:latest image with user and ssh service configured
- `centos1`: centos/systemd:latest image with user and ssh service configured

We have configured a secret to pass into the ubuntu containers our `id_rsa.pub` public key. Also we have configured a volume for centos container to share the public key, because **centos needs to run as privileged container to can run services using systemd**. And secrets are not compatible with `priviliged=true` container. You can check all in [docker-compose.yaml](docker-compose.yaml).


## 3. RSA Key used in the servers

We have choosen to share `~/.ssh/id_rsa_shared.pub` in all servers, but it is not in the default ssh checked keys:
- `~/.ssh/id_ecdsa`
- `~/.ssh/id_ecdsa_sk`
- `~/.ssh/id_ed25519`
- `~/.ssh/id_ed25519_sk`
- `~/.ssh/id_xmss`
- `~/.ssh/id_xmss`
- `~/.ssh/id_dsa`
- `~/.ssh/id_rsa`

So we will have to choose the private key we want using `--private-key`. Also the user configured inside the server is `alex`, you can check it into the [Dockerfile](Dockerfile) or [centos.Dockerfile](centos.Dockerfile). So by default, if no `username` is passed to ssh, it tries to stablish the connection with your local linux user. So to run the playbooks on any machine regardless the `username` you can user the flag `-u my_username`
```bash
ansible-playbook ... --private-key ~/.ssh/id_rsa_shared -u jiminycricket
```

## 4. Start the Services
### 4.1 Start one server
To start just one server, we have to tell docker-compose to run only one of the services:
```bash
docker compose up -d ubuntu1
```

### 4.2 Start both ubuntu servers
```bash
docker compose up -d ubuntu1 ubuntu2
```

### 4.3 Start all servers
```bash
docker compose up -d
```

### 4.4 Stop both ubuntu servers
```bash
docker compose down ubuntu1 ubuntu2
```

### 4.5 Stop all servers
```bash
docker compose down
```

## 5. Ansible

### 5.1 Ansible Project Structure
```bash
├── inventories/ # Folder where the inventories will be stored
│   └── inventory.ini # Main inventory
├── playbooks/ # Folder where the playbooks will be stored
...
```

### 5.2 My first ansible Code
We have generated these 2 easy yaml files to run our first ansible playbook
- [inventories/inventory.ini](inventories/inventory.ini)
- [playbooks/basic_playbook.yaml](playbooks/basic_playbook.yaml)

So let's run it:
```bash
ansible-playbook playbooks/basic_playbook.yaml -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex
```
![im2.png](pictures/im2.png)