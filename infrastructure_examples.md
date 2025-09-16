# Infrastructure Automation Examples

This document provides **basic starter configurations** for four major automation and monitoring tools:

- **Terraform**
- **Nagios**
- **Ansible**
- **Puppet**

---

## 1. Terraform Example (`main.tf`)

Creates an AWS EC2 instance:

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-08c40ec9ead489470" # Ubuntu 22.04 in us-east-1
  instance_type = "t2.micro"

  tags = {
    Name = "Terraform-Example"
  }
}
```

**Usage:**

```bash
terraform init
terraform apply
```

---

## 2. Nagios Example (`localhost.cfg`)

Monitors localhost with **ping** and **SSH** checks:

```cfg
define host {
    use             linux-server
    host_name       localhost
    alias           Localhost
    address         127.0.0.1
}

define service {
    use                 generic-service
    host_name           localhost
    service_description PING
    check_command       check_ping!100.0,20%!500.0,60%
}

define service {
    use                 generic-service
    host_name           localhost
    service_description SSH
    check_command       check_ssh
}
```

Place this file under:

```
/usr/local/nagios/etc/objects/
```

and include it in your main `nagios.cfg`.

---

## 3. Ansible Example (`playbook.yml`)

Installs and configures Apache on managed nodes:

```yaml
- name: Install Apache on managed hosts
  hosts: managed
  become: yes
  tasks:
    - name: Install Apache package
      apt:
        name: apache2
        state: present
        update_cache: yes

    - name: Ensure Apache is running
      service:
        name: apache2
        state: started
        enabled: yes
```

**Run:**

```bash
ansible-playbook playbook.yml
```

---

## 4. Puppet Example (`init.pp`)

Ensures Apache is installed and running:

```puppet
class apache {
  package { 'apache2':
    ensure => installed,
  }

  service { 'apache2':
    ensure     => running,
    enable     => true,
    require    => Package['apache2'],
  }
}

include apache
```

**Apply:**

```bash
puppet apply init.pp
```

---

## Summary

- **Terraform** → Provision infrastructure  
- **Ansible** → Configure software  
- **Puppet** → Enforce state  
- **Nagios** → Monitor systems  

These are **minimal working templates** to help you get started with infrastructure automation.
