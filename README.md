# Rady Ansible Servers using Docker Compose

## Contents

## 1. Introduction

In this project we will run using docker-compose a few servers ready to play with using Ansible. The way we have configured the image and the servers from scratch is in the following [GitHub Project - SSH docker container](https://github.com/titocampis/ssh-docker-container)

## 2. Ubuntu Severs Configuration

The server configuration is in [Dockerfile](Dockerfile).

We have configured 2 services in the [docker-compose.yaml](docker-compose.yaml):
- `ubuntu1`: ubuntu:latest image with ssh service configured
- `ubuntu2`: ubuntu:latest image with ssh service configured

We have configured a secret to pass into the ubuntu containers our `id_rsa.pub` public key:
```yaml
secrets:
  user_ssh_rsa:
    file: ~/.ssh/id_rsa.pub
```

## 3. Centos7 Server Configuration
For centos7 it was imposible to configure the servers using docker compose, because to use ssh centos7 needs priviliged container, which is not compatible with mounting secrets or volumes in centos7.

So the configuration of the image is in the [centos.Dockerfile](centos.Dockerfile).

Pull the centos/systemd:latest image:
```bash
docker pull centos/systemd:latest
```

Build the image using the custom Dockerfile:
```bash
docker build -t centos:ssh . -f centos.Dockerfile
```
## 4. RSA Key used in the servers

We have choosen to share `~/.ssh/id_rsa_shared.pub` in all servers, but it is not in the default ssh checked keys:
- `~/.ssh/id_ecdsa`
- `~/.ssh/id_ecdsa_sk`
- `~/.ssh/id_ed25519`
- `~/.ssh/id_ed25519_sk`
- `~/.ssh/id_xmss`
- `~/.ssh/id_xmss`
- `~/.ssh/id_dsa`
- `~/.ssh/id_rsa`

So we will have to choose the private key we want using `--private-key`. Also the user configured inside the server is `alex`, you can check it into the [Dockerfile](Dockerfile). So by default, if no `username` is passed to ssh, it tries to stablish the connection with your local linux user. So to run the playbooks on any machine regardless the `username` you can user the flag `-u my_username`
```bash
ansible-playbook ... --private-key ~/.ssh/id_rsa_shared -u jiminycricket
```

## 5. Start the Services
### 5.1 Start one ubuntu server
To start just one server, we have to tell docker-compose to run only one of the services:
```bash
docker compose up -d ubuntu1
```

### 5.2 Start both ubuntu servers
```bash
docker compose up -d ubuntu1 ubuntu2
```
```bash
docker compose up -d
```

## 5.3 Start centos server
```bash
docker run --rm -d --privileged=true --name centos_ssh1 -p 3333:22 -v ~/.ssh/id_rsa_shared.pub:/home/alex/.ssh/authorized_keys:ro centos:ssh
```

### 5.4 Stop both ubuntu servers
```bash
docker compose down
```

### 5.5 Stop centos server
```bash
docker rm -f centos1
```

## 6. Ansible

### 6.1 