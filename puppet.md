## Checks Before Installation
- Ensure VM's OS is Ubuntu 22.0
- Ensure Both VM's are on same network (VPC for Amazon , VNET for Azure)
- If on Amazon Add a Inbound port rule for **All Traffic** from the **security-group** of Agent
- - Better to Keep all Machines in Same **security-group** 
- Make sure You are able to ping VM's from each other  
Example
```sh
ping <Other-VM-Private-IP>
```

## Installation Of Puppet

**Machine :** Both
```sh
sudo apt update -y
sudo apt upgrade -y
sudo apt install wget curl gnupg -y
```

**Machine :** Both
```sh
wget https://apt.puppet.com/puppet8-release-$(lsb_release -cs).deb
sudo dpkg -i puppet8-release-$(lsb_release -cs).deb
sudo apt update
```

**Machine :** Master
```sh
sudo apt install puppetserver -y

```

**Machine :** Master
```sh
sudo nano /etc/default/puppetserver
JAVA_ARGS="-Xms512m -Xmx512m"
```

**Machine :** Master
```sh
sudo systemctl enable puppetserver
sudo systemctl start puppetserver
```

**Machine :** Agent
```sh
sudo apt install puppet-agent -y
```

**Machine :** Agent
```sh
sudo systemctl enable puppet
sudo systemctl start puppet
```

**Machine :** Both  
Edit : `/etc/hosts` Using Nano `sudo nano /etc/  hosts`  
**Add Following**
```txt
<Private-IP-Master> puppet
<Private-IP-Agent> puppetagent
```
**Machine :** Agent  
Edit `/etc/puppetlabs/puppet/puppet.conf`  Using Nano `sudo nano /etc/puppetlabs/puppet/puppet.conf`  
**Add Following**
```ini
[main]
server = puppet
```

**Machine :** Agent
```sh
sudo /opt/puppetlabs/bin/puppet agent --test --waitforcert 60
```

Master
```sh
sudo /opt/puppetlabs/bin/puppetserver ca list
sudo /opt/puppetlabs/bin/puppetserver ca sign --all
```

**Machine :** Agent
```sh
sudo /opt/puppetlabs/bin/puppet agent -t
```