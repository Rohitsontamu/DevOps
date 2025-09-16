# Puppet Master-Agent Setup in Docker (with Volumes)

This guide explains how to set up a Puppet Master and Puppet Agents
inside Docker containers, using **volumes** to persist SSL certificates,
logs, and configuration files.

------------------------------------------------------------------------

## 🛠 Prerequisites

-   Windows with **Docker Desktop** installed
-   Basic knowledge of PowerShell and Linux commands

------------------------------------------------------------------------

## 1. Create Docker Volumes

We use Docker volumes to ensure data persists even if containers are
removed.

``` powershell
docker volume create puppet-master-ssl
docker volume create puppet-master-logs
docker volume create puppet-master-code

docker volume create puppet-agent1-ssl
docker volume create puppet-agent1-logs

docker volume create puppet-agent2-ssl
docker volume create puppet-agent2-logs
```

------------------------------------------------------------------------

## 2. Start Puppet Master Container

Run Puppet Master with attached volumes:

``` powershell
docker run -d --name puppet-master `
  -h puppet `
  -p 8140:8140 `
  -v puppet-master-ssl:/etc/puppetlabs/puppet/ssl `
  -v puppet-master-logs:/var/log/puppetlabs `
  -v puppet-master-code:/etc/puppetlabs/code `
  puppet/puppetserver:8.6.1
```

Verify the container is running:

``` powershell
docker ps
```

Inside the container, check service:

``` bash
docker exec -it puppet-master bash
/opt/puppetlabs/bin/puppetserver ca list
```

------------------------------------------------------------------------

## 3. Start Puppet Agent Containers

### Agent 1

``` powershell
docker run -d --name puppet-agent1 `
  -h puppet-agent1 `
  --link puppet-master `
  -v puppet-agent1-ssl:/etc/puppetlabs/puppet/ssl `
  -v puppet-agent1-logs:/var/log/puppetlabs `
  puppet/puppet-agent:8.6.1 sleep infinity
```

### Agent 2 (with port mapping for Apache test)

``` powershell
docker run -d --name puppet-agent2 `
  -h puppet-agent2 `
  --link puppet-master `
  -p 8081:80 `
  -v puppet-agent2-ssl:/etc/puppetlabs/puppet/ssl `
  -v puppet-agent2-logs:/var/log/puppetlabs `
  puppet/puppet-agent:8.6.1 sleep infinity
```

------------------------------------------------------------------------

## 4. Configure Puppet Agents

Inside **each agent**:

``` bash
puppet config set server puppet --section main
puppet agent --test --waitforcert=60
```

On **master**, sign certificates if autosigning is disabled:

``` bash
docker exec -it puppet-master bash
/opt/puppetlabs/bin/puppetserver ca list
/opt/puppetlabs/bin/puppetserver ca sign --certname puppet-agent1
/opt/puppetlabs/bin/puppetserver ca sign --certname puppet-agent2
```

------------------------------------------------------------------------

## 5. Write a Simple Puppet Manifest

Create a **site.pp** on the master:

``` bash
docker exec -it puppet-master bash
nano /etc/puppetlabs/code/environments/production/manifests/site.pp
```

Example manifest (Apache installation for agent2):

``` puppet
node 'puppet-agent2' {
  package { 'apache2':
    ensure => installed,
  }

  service { 'apache2':
    ensure => running,
    enable => true,
  }

  file { '/var/www/html/index.html':
    ensure  => file,
    content => "<h1>Hello from Puppet!</h1>",
  }
}

node default {
  notify { 'default-node':
    message => "This is the default node manifest",
  }
}
```

------------------------------------------------------------------------

## 6. Apply Manifest from Agent

On **agent2**:

``` bash
puppet agent --test --waitforcert=60
```

Check Apache status:

``` bash
service apache2 status
```

Verify in browser (from Windows host):

    http://localhost:8081

------------------------------------------------------------------------

## 7. Persistency Notes

-   Stopping containers with `docker stop` will **not lose data**.\
-   Removing containers with `docker rm` will also **not lose data**
    since SSL, logs, and code are in Docker volumes.\
-   To inspect volumes:

``` powershell
docker volume ls
docker run --rm -it -v puppet-master-code:/data ubuntu ls /data
```

------------------------------------------------------------------------

✅ Now you have a working **Puppet Master-Agent** setup inside Docker,
with volumes ensuring persistence of logs, SSL certs, and manifests.
