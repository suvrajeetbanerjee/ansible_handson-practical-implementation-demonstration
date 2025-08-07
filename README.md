

# 🚀 Practical Ansible Setup & HandsOn Demonstration on Play With Docker  
*A multi-node containerized Ansible learning lab — hands-on, documented, and ready for revision.*

[![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white&style=for-the-badge)](https://www.docker.com/)
[![Ansible](https://img.shields.io/badge/Ansible-EE0000?logo=ansible&logoColor=white&style=for-the-badge)](https://www.ansible.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?logo=ubuntu&logoColor=white&style=for-the-badge)](https://ubuntu.com/)
[![Play With Docker](https://img.shields.io/badge/Play_With_Docker-000000?logo=docker&logoColor=white&style=for-the-badge)](https://labs.play-with-docker.com/)

---

## 📑 Table of Contents
- [📚 Overview](#-overview)
- [🛠 Environment Setup](#-environment-setup)
- [📊 Container Network Topology](#-container-network-topology)
- [🐳 Installing Docker & Creating Containers](#-installing-docker--creating-containers)
- [🔧 Verifying Containers](#-verifying-containers)
- [📦 Configuring ansible_master](#-configuring-ansible_master)
- [🎯 Setting Up Target Machines](#-setting-up-target-machines)
- [🌐 Finding Target IPs](#-finding-target-ips)
- [📄 Configuring Ansible Hosts File](#-configuring-ansible-hosts-file)
- [🔑 SSH Key Setup](#-ssh-key-setup-for-passwordless-access)
- [🧪 Quick Test Playbook](#-quick-test-playbook)
- [🐛 Challenges Faced](#-challenges-faced)
- [📌 References](#-references)
- [⏭️ Next Steps](#-next-steps)

---

## 📚 Overview  
This repository captures my **practical-based Ansible learning journey**, revisiting core concepts by performing a **multi-node setup** using [Play With Docker](https://labs.play-with-docker.com).  
It’s structured for **quick recall** — so future me (or anyone following this) can re-run the lab easily.

---

## 🛠 Environment Setup  

### **🔹 Tools Used**
- [Play With Docker](https://labs.play-with-docker.com) — Free online Docker playground.
- Docker 🐳
- Ansible ⚙️
- Ubuntu Containers 🐧

### **🔹 Instances Created**
| Node Name        | Role          | Private IP   |
|------------------|--------------|--------------|
| `node1`          | target2      | 192.168.0.29 |
| `node2`          | target1      | 192.168.0.28 |
| `node3`          | master       | 192.168.0.27 |

---

## 📊 Container Network Topology  

```mermaid
graph TD
    A[ansible_master<br>172.17.0.4] -->|SSH| B[target1<br>172.17.0.2]
    A -->|SSH| C[target2<br>172.17.0.3]
````

---

## 🐳 Installing Docker & Creating Containers

> **Note:** Play With Docker uses **Alpine Linux** under the hood, so package management is done via `apk`.

```bash
apk update && apk add docker
docker run -dit --name ansible_master ubuntu /bin/bash
docker run -dit --name target1 ubuntu /bin/bash
docker run -dit --name target2 ubuntu /bin/bash
```

**💡 Key Points**

* `apk update` → Refresh package index on Alpine.
* `apk add docker` → Install Docker.
* `docker run -dit` → Run container in detached mode, allocate a terminal, and keep stdin open.

---

## 🔧 Verifying Containers

```bash
docker ps
```

Lists running containers, including IDs, names, images, and uptime.

---

## 📦 Configuring ansible\_master

```bash
docker exec -it ansible_master bash
apt update
apt install python-is-python3 vim iputils-ping openssh-client -y
apt install software-properties-common
add-apt-repository --yes --update ppa:ansible/ansible
apt install ansible
ansible --version
```

> **Pro Tip:** `python-is-python3` ensures compatibility since Ansible requires Python 3.

---

## 🎯 Setting Up Target Machines

Repeat for **target1** & **target2**:

```bash
docker exec -it target1 bash
apt update
apt install vim python-is-python3 iputils-ping openssh-client -y
apt-get install openssh-client openssh-server -y
cd /etc/ssh
vi sshd_config
```

### **🔹 SSH Configuration Changes**

* `PermitRootLogin yes`
* `PasswordAuthentication yes`

```bash
service ssh start
passwd root   # set to 'admin'
```

---

## 🌐 Finding Target IPs

```bash
docker inspect target1 | grep IPAddress
docker inspect target2 | grep IPAddress
```

---

## 📄 Configuring Ansible Hosts File

```bash
docker exec -it ansible_master bash
vi /etc/ansible/hosts
```

Example entry:

```
[target_nodes]
172.17.0.2
172.17.0.3
```

---

## 🔑 SSH Key Setup for Passwordless Access

```bash
ssh-keygen   # Press Enter for defaults
ssh-copy-id root@172.17.0.2
ssh-copy-id root@172.17.0.3
```

**Verification:**

```bash
ssh root@172.17.0.2
ping 172.17.0.2
```

---

## 🧪 Quick Test Playbook

Once SSH connectivity is set up, create a test playbook:

```yaml
# test-ping.yml
- name: Test Ansible Connectivity
  hosts: target_nodes
  gather_facts: no
  tasks:
    - name: Ping all target nodes
      ansible.builtin.ping:
```

Run the playbook:

```bash
ansible-playbook test-ping.yml
```

Expected output:

```
target1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
target2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

## 🐛 Challenges Faced

### **Package Manager Mismatch**

* PWD uses `apk` by default.

```bash
command -v apk   # returns path
command -v apt   # not found
```

### **SSH Server Not Running**

* Installed & started `openssh-server` in target containers before using `ssh-copy-id`.

---

## 📌 References

* [Ansible Installation on Ubuntu 20.04](https://www.ansiblepilot.com/articles/how-to-install-ansible-in-ubuntu-20.04-ansible-install/)

---

## ⏭️ Next Steps

* Automating playbook execution.
* Adding more nodes.
* Testing advanced Ansible modules.

