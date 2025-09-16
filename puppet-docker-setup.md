# Puppet Master and Agent Setup in Docker (with Persistent Volumes)

This guide explains how to set up a Puppet Master and multiple Puppet Agents inside Docker containers on **Ubuntu 22.04 base images**, with **persistent volumes** so you don’t lose SSL certificates, logs, or configuration data.

---

## 1. Create a Docker Network
We’ll create a dedicated bridge network so the containers can communicate by hostname.

```powershell
docker network create puppet-net
```

---

## 2. Start the Puppet Master Container

Run the Puppet Master with volumes for persistence:

```powershell
docker run -dit --name puppet-master `
  --hostname puppet `
  --network puppet-net `
  -v puppet-server-ssl:/etc/puppetlabs/puppet/ssl `
  -v puppet-server-data:/opt/puppetlabs/server/data/puppetserver `
  -v puppet-server-logs:/var/log/puppetlabs/puppetserver `
  ubuntu:22.04 bash
```

### Inside the container (Puppet Master):
```bash
apt-get update
apt-get install -y wget gnupg lsb-release sudo nano

# Add Puppet 8 repo
wget https://apt.puppet.com/puppet8-release-$(lsb_release -cs).deb
dpkg -i puppet8-release-$(lsb_release -cs).deb
apt-get update

# Install Puppet Server
apt-get install -y puppetserver

# Start Puppet Server
/opt/puppetlabs/bin/puppetserver start
```

---

## 3. Start a Puppet Agent Container

Run the Puppet Agent with its own volumes:

```powershell
docker run -dit --name puppet-agent1 `
  --hostname puppet-agent1 `
  --network puppet-net `
  -v puppet-agent1-ssl:/etc/puppetlabs/puppet/ssl `
  -v puppet-agent1-logs:/var/log/puppetlabs/puppet `
  ubuntu:22.04 bash
```

### Inside the container (Agent):
```bash
apt-get update
apt-get install -y wget gnupg lsb-release sudo nano

# Add Puppet 8 repo
wget https://apt.puppet.com/puppet8-release-$(lsb_release -cs).deb
dpkg -i puppet8-release-$(lsb_release -cs).deb
apt-get update

# Install Puppet Agent
apt-get install -y puppet-agent

# Configure master
/opt/puppetlabs/bin/puppet config set server puppet --section main
```

---

## 4. Certificate Signing

### On the Agent:
Request a certificate:
```bash
puppet agent --test --waitforcert=60
```

### On the Master:
List pending requests:
```bash
/opt/puppetlabs/bin/puppetserver ca list
```

Sign the agent’s certificate:
```bash
/opt/puppetlabs/bin/puppetserver ca sign --certname puppet-agent1
```

---

## 5. Verify Agent Connection

Run the agent again to confirm successful catalog application:
```bash
puppet agent --test
```

---

## 6. Deploy a Simple Manifest (Example: Apache)

### On Master:
Edit the manifest:
```bash
nano /etc/puppetlabs/code/environments/production/manifests/site.pp
```

Add:
```puppet
node 'puppet-agent1' {
  package { 'apache2':
    ensure => installed,
  }

  service { 'apache2':
    ensure => running,
    enable => true,
  }
}
```

---

## 7. Test on Agent

On the agent:
```bash
puppet agent --test
```

Verify Apache:
```bash
service apache2 status
```

---

## 8. Persistence

- SSL certs are stored in `puppet-server-ssl` and `puppet-agent1-ssl`.
- Logs are stored in volumes `puppet-server-logs` and `puppet-agent1-logs`.
- Data is preserved across container restarts.

---

## 9. Restarting Containers

To restart Puppet Master:
```powershell
docker restart puppet-master
```

To restart Agent:
```powershell
docker restart puppet-agent1
```

Volumes ensure certificates and configs are **not lost**.

---

✅ You now have a working Puppet Master-Agent setup with Docker, persistent storage, and custom manifests.
