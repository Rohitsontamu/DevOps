# 🛠️ Ansible Setup Guide (Controller + Agents in Docker without Dockerfiles)

## 📌 Prerequisites
- Docker Desktop (or Docker Engine)
- Basic Linux knowledge

---

## 1️⃣ Create Network
All containers must share a network:
```bash
docker network create ansible-net
```

---

## 2️⃣ Start Controller Container
Run Ubuntu container for controller:
```bash
docker run -dit --name ansible-controller --hostname ansible-controller --network ansible-net ubuntu:20.04 bash
```

Enter the container:
```bash
docker exec -it ansible-controller bash
```

Install Ansible and tools:
```bash
apt-get update
apt-get install -y ansible sshpass nano iputils-ping
```

---

## 3️⃣ Start Agent Containers
Start two agent containers:
```bash
docker run -dit --name ansible-agent1 --hostname ansible-agent1 --network ansible-net ubuntu:20.04 bash
docker run -dit --name ansible-agent2 --hostname ansible-agent2 --network ansible-net ubuntu:20.04 bash
```

Enter each agent and configure SSH + Python:

```bash
docker exec -it ansible-agent1 bash
docker exec -it ansible-agent2 bash
```

Inside each agent:
```bash
apt-get update
apt-get install -y openssh-server python3
mkdir /var/run/sshd
echo 'root:rootpassword' | chpasswd
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
/etc/init.d/ssh start
```

---

## 4️⃣ Configure Inventory on Controller
Back in the **controller**:
```bash
docker exec -it ansible-controller bash
nano /etc/ansible/hosts
```

Add:
```ini
[agents]
ansible-agent1 ansible_host=ansible-agent1 ansible_user=root ansible_password=rootpassword
ansible-agent2 ansible_host=ansible-agent2 ansible_user=root ansible_password=rootpassword
```

---

## 5️⃣ Verify Connection
Inside controller:
```bash
ansible -m ping agents
```

Expected result:
```yaml
ansible-agent1 | SUCCESS => { "changed": false, "ping": "pong" }
ansible-agent2 | SUCCESS => { "changed": false, "ping": "pong" }
```

---

## 6️⃣ Run a Test Playbook
Create **playbook.yml** inside controller:
```yaml
---
- name: Install curl on all agents
  hosts: agents
  become: yes
  tasks:
    - name: Ensure curl is installed
      apt:
        name: curl
        state: present
        update_cache: yes
```

Run:
```bash
ansible-playbook playbook.yml
```

---

## 7️⃣ Cleanup
Stop and remove containers:
```bash
docker rm -f ansible-controller ansible-agent1 ansible-agent2
docker network rm ansible-net
```

---

✅ You now have a **working Ansible lab** with **one controller and multiple agents** inside Docker — no Dockerfiles needed.
