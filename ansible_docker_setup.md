# 🐳 Ansible Setup with Two Docker Containers (Control + Managed)

This guide shows how to set up Ansible in **two Docker containers**:  
- **Control Node** → Runs Ansible  
- **Managed Node** → Controlled by Ansible via SSH  

---

## ✅ Prerequisites
- Docker installed on your system  
- Basic Linux and Ansible knowledge  

---

## 1️⃣ Create a User-Defined Docker Network
So containers can talk to each other by name:

```bash
docker network create ansible-net
```

---

## 2️⃣ Start the Managed Node (Target Server)
Run an Ubuntu container with SSH installed:

```bash
docker run -d --name managed-node --network ansible-net ubuntu:22.04 sleep infinity
```

Access the container:

```bash
docker exec -it managed-node bash
```

Inside the container, install SSH server:

```bash
apt-get update && apt-get install -y openssh-server sudo
service ssh start
```

Create a user for Ansible:

```bash
useradd -m -s /bin/bash ansible
echo "ansible:ansible" | chpasswd
echo "ansible ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

Exit container:

```bash
exit
```

---

## 3️⃣ Start the Control Node (Ansible Host)
Run another Ubuntu container:

```bash
docker run -it --name control-node --network ansible-net ubuntu:22.04
```

Inside control-node, install dependencies:

```bash
apt-get update && apt-get install -y software-properties-common python3 python3-pip curl git sshpass sudo openssh-client
apt-get install -y ansible
```

Verify installation:

```bash
ansible --version
```

---

## 4️⃣ Configure SSH Access
Generate SSH key on **control-node**:

```bash
ssh-keygen -t rsa -b 2048 -f /root/.ssh/id_rsa -N ""
```

Copy key to **managed-node**:

```bash
sshpass -p "ansible" ssh-copy-id -o StrictHostKeyChecking=no ansible@managed-node
```

Test SSH connection:

```bash
ssh ansible@managed-node hostname
```

---

## 5️⃣ Setup Ansible Inventory
On control-node, create an inventory file:

```bash
mkdir -p /etc/ansible
echo "[docker-nodes]" > /etc/ansible/hosts
echo "managed-node ansible_user=ansible" >> /etc/ansible/hosts
```

---

## 6️⃣ Test Ansible Connectivity
Run:

```bash
ansible all -i /etc/ansible/hosts -m ping
```

Expected output:

```
managed-node | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

## 7️⃣ Run a Simple Playbook
Create `playbook.yml` inside control-node:

```bash
cat > playbook.yml <<EOF
- hosts: docker-nodes
  become: yes
  tasks:
    - name: Install curl on managed node
      apt:
        name: curl
        state: present
EOF
```

Run playbook:

```bash
ansible-playbook -i /etc/ansible/hosts playbook.yml
```

Verify on managed-node:

```bash
docker exec -it managed-node which curl
```

---

## 🎯 Summary
- Created **two Docker containers** (control + managed)  
- Installed SSH on managed node and Ansible on control node  
- Configured SSH key-based login  
- Tested connectivity with `ansible -m ping`  
- Ran a playbook to install a package  
