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
├── inventories/ # Folder where inventories are stored
│   └── inventory.ini # Main inventory
├── playbooks/ # Folder where playbooks are stored
│   └── ...
├── roles/ # Folder where roles are stored
│   └── ...
...
```

### 5.2 My first ansible Code - playbooks/helloworld.yaml
We have generated these 2 easy yaml files to run our first ansible playbook
- [inventories/inventory.ini](inventories/helloworld.yaml)

So let's check it:
```bash
ansible-playbook -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex playbooks/basic_playbook.yaml --check
```
And if everything went well, run it:
```bash
ansible-playbook -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex playbooks/basic_playbook.yaml
```
![im2.png](pictures/im2.png)

- `-i`: to set te inventory
- `--private-key`: to pass a custom private key an not the default ones
- `-u`: to pass the user
- `--check`: to do not exec the playbook, just check

> :paperclip: **NOTE:** User the `-u` flag is the same as `--extra-vars ansible_user` or to define the variable inside the inventory or the playbook.

### 5.1 Executing more complex packages

All the playbooks are designed using:
- `playbooks/PLAYBOOK_NAME.yaml`: to configure the playbook
- `roles/ROLE_NAME/tasks`: to configure the tasks

The execution should follow:
```bash
ansible-playbook -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex playbooks/PLAYBOOK_NAME.yaml --diff --tags tag1,tag2,..,tagn --check
```

> :paperclip: If you want to not pass every time the pubkey you can stored in your ssh default keys using the `ssh-agent`, [documentation](https://www.linode.com/docs/guides/using-ssh-agent/)
>
>Starting up ssh-agent:
>```bash
> eval `ssh-agent`
>```
>
> Check the `ssh-agent` is running:
>```bash
> echo $SSH_AUTH_SOCK
>```
>
> Add the key you want to 
>```bash
> ssh-add ~/.ssh/custom_key
>```
> 
> To get a list of all the keys added
>```bash
> ssh-add -l
>```

- `--diff`: when some file is modified by ansible, show it

```bash
ansible-playbook -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex playbooks/PLAYBOOK_NAME.yaml --diff --tags tag1,tag2,..,tagn
```

### 5.3.1 Managing Packages

- playbook: [playbooks/install_packages.yaml](playbooks/install_packages.yaml)
- role: [roles/packages/tasks](roles/packages/tasks)

```bash
ansible-playbook -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex playbooks/install_packages.yaml --diff --tags debug --check
```

### 5.3.2 Managing Users
- playbook: [playbooks/users.yaml](playbooks/users.yaml)
- role: [roles/users/tasks](roles/users/tasks)

```bash
ansible-playbook -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex playbooks/users.yaml --diff --tags users --extra-vars ansible_sudo_pass=THEPASSWORD --check
```

> :warning: **WARNING:** The hosts do not have the `alex` user to have sudo permissions without password, you can run the `users` packages with `sudo` tags to do it, if not you should pass the `alex sudo password` to ansible: 
```bash
ansible-playbook -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex playbooks/users.yaml --diff --tags users --check
``

### 5.3.3 Ensuring services
- playbook: [playbooks/service_ensure.yaml](playbooks/service_ensure.yaml)
- role: [roles/services/tasks](roles/services/tasks)

```bash
ansible-playbook -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex playbooks/service_ensure.yaml --diff --tags debug --check
```


### 5.3.4 Executing commands
- playbook: [playbooks/ls-authorized_keys.yaml](playbooks/ls-authorized_keys.yaml)

```bash
ansible-playbook -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex playbooks/ls-authorized_keys.yaml --diff --check
```

### 5.3.5 Executing scripts
- playbook: [playbooks/copy-and-exec.yaml](playbooks/copy-and-exec.yaml)

```bash
ansible-playbook -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex playbooks/exec_hello_sh.yaml --diff --tags debug --check
```

### 5.3.4 Ensure lines in files
- playbook: []()
- role: [roles/lineinfile/tasks](roles/lineinfile/tasks)

```bash
ansible-playbook -i inventories/inventory.ini --private-key ~/.ssh/id_rsa_shared -u alex playbooks/ensure_file_content.yaml --diff --check
```

## Next Steps
| **Status** | **Task** |
|----------|----------|
| :clock10: | Check how to encrypt passwords with ansible for the users |
| :clock10: | Change the debian image version to not latest |
| :clock10: | Check run services in Debian Hosts |
| :clock10: | Check install packages |
| :clock10: | Ensure ssh authorized keys |
