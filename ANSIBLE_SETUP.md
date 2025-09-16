
# Ansible with Docker: Quick Start Guide

This guide demonstrates how to set up an Ansible controller and a managed node using Docker containers, and how to configure them for basic connectivity.

## 1. Create a Docker Network

```sh
docker network create ansible_network
```

## 2. Start the Managed Node Container

```sh
docker run -dit --name managed_node --network ansible_network ubuntu:22.04 bash
```

## 3. Configure the Managed Node

```sh
docker exec -it managed_node bash
# Inside the container, run:
apt update && apt install -y openssh-server sudo nano \
    && mkdir /var/run/sshd \
    && echo 'root:root' | chpasswd \
    && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config \
    && service ssh start
```

## 4. Start the Ansible Controller Container

```sh
docker run -dit --name ansible_controller --network ansible_network ubuntu:22.04 bash
```

## 5. Configure the Ansible Controller

```sh
docker exec -it ansible_controller bash
# Inside the container, run:
apt update && apt install -y ansible sshpass
```

### Edit Ansible Inventory

Edit `/etc/ansible/hosts` and add: If the folder is not preset then use ` mkdir /etc/ansible` to create the folder

```ini
[agents]
managed_node ansible_user=root ansible_password=root ansible_host=managed_node ansible_connection=ssh
```

### Disable Host Key Checking

```sh
export ANSIBLE_HOST_KEY_CHECKING=False
```

Or edit `/etc/ansible/ansible.cfg` and set:

```ini
[defaults]
host_key_checking = False
```

## 6. Test the Connection

```sh
ansible agents -m ping
```
