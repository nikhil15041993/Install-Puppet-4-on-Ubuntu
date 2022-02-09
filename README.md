# Install-Puppet-4-on-Ubuntu

Puppet is a configuration management tool that helps system administrators automate the provisioning, configuration and management of a server infrastructure.
Puppet comes in two varieties, Puppet Enterprise and open source Puppet.
The Puppet master server—which runs the Puppet Server software—can be used to control all your other servers, called Puppet agent nodes.

## Prerequisites
* at least 4GB of memory
* at least 2 CPU cores

## Step 1 — Configuring /etc/hosts

On every machine
On each machine, edit the /etc/hosts file. At the end of the file, specify the Puppet master server as follows, substituting the IP address for your Puppet master:

sudo nano /etc/hosts
/etc/hosts
```
puppet_ip_address    puppet
```

## Step 2 — Installing Puppet Server

We’ll enable the official Puppet Labs collection repository with these commands:

```
wget https://apt.puppetlabs.com/puppet6-release-focal.deb
sudo dpkg -i puppet6-release-focal.deb
sudo apt-get update
```
When apt-get update is complete, ensuring that we’ll be pulling from the Puppet Labs repository, we’ll install the puppetserver package:
```
sudo apt-get install puppetserver -y
```
## Configure memory allocation

By default, Puppet Server is configured to use 2 GB of RAM. You can customize this setting based on how much free memory the master server has and how many agent nodes it will manage.

To customize it, open /etc/default/puppetserver:
```
sudo nano /etc/default/puppetserver
```
Then find the JAVA_ARGS line, and use the -Xms and -Xmx parameters to set the memory allocation. We’ll increase ours to 3 gigabytes:
```
/etc/default/puppetserver
JAVA_ARGS="-Xms3g -Xmx3g -XX:MaxPermSize=256m"
```

## Open the firewall
When we start Puppet Server, it will use port 8140 to communicate, so we’ll ensure it’s open:

```
sudo ufw allow 8140
```
Next, we’ll start Puppet server.

Start Puppet server
We’ll use systemctl to start Puppet server:
```
sudo systemctl start puppetserver
```
Now that we’ve ensured the server is running, we’ll configure it to start at boot:
```
sudo systemctl enable puppetserver
```
## Step 3 — Installing the Puppet Agent

Enable the official Puppet Labs repository
First we’ll enable the official Puppet Labs collection repository with these commands:
```
wget https://apt.puppetlabs.com/puppet6-release-focal.deb
sudo dpkg -i puppet6-release-focal.deb
sudo apt-get update
```
Install the Puppet agent package
Then, we’ll install the puppet-agent package:
```
sudo apt-get install puppet-agent -y
```
We’ll start the agent and enable it to start on boot:
```
sudo systemctl start puppet
sudo systemctl enable puppet
```
## Step 4 — Signing Certificates on Puppet Master

The first time Puppet runs on an agent node, it sends a certificate signing request to the Puppet master. Before Puppet Server will be able to communicate with and control the agent node, it must sign that particular agent node’s certificate.

List current certificate requests
To list all unsigned certificate requests, run the following command on the Puppet master:
```
sudo /opt/puppetlabs/bin/puppet cert list
```
There should be one request for each host you set up, that looks something like the following:
```
Output:
  "db1.localdomain"  (SHA256) 46:19:79:3F:70:19:0A:FB:DA:3D:C8:74:47:EF:C8:B0:05:8A:06:50:2B:40:B3:B9:26:35:F6:96:17:85:5E:7C
  "web1.localdomain" (SHA256) 9D:49:DE:46:1C:0F:40:19:9B:55:FC:97:69:E9:2B:C4:93:D8:A6:3C:B8:AB:CB:DD:E6:F5:A0:9C:37:C8:66:A0
```

### Sign requests
To sign a single certificate request, use the ```puppet cert sign command```, with the hostname of the certificate as it is displayed in the certificate request.

For example, to sign db1’s certificate, you would use the following command:
```
sudo /opt/puppetlabs/bin/puppet cert sign db1.localdomain
```

Output similar to the example below indicates that the certificate request has been signed:
```
Output:
Notice: Signed certificate request for db.localdomain
Notice: Removing file Puppet::SSL::CertificateRequest db1.localdomain at '/etc/puppetlabs/puppet/ssl/ca/requests/db1.localdomain.pem'
```
You can also sign all current requests at once.

We’ll use the --all option to sign the remaining certificate:
```
sudo /opt/puppetlabs/bin/puppet cert sign --all
```
## Step 5 — Verifying the Installation

Puppet uses a domain-specific language to describe system configurations, and these descriptions are saved to files called “manifests”, which have a .pp file extension.We’ll begin by creating the default manifest, site.pp, in the default location:

```
sudo nano /etc/puppetlabs/code/environments/production/manifests/site.pp
```

We’ll use Puppet’s domain-specific language to create a file called it_works.txt on agent nodes located in the tmp directory which contains the public IP address of the agent server and sets the permissions to-rw-r--r--:

```
file {'/tmp/it_works.txt':                        # resource type file and filename
  ensure  => present,                             # make sure it exists
  mode    => '0644',                              # file permissions
  content => "It works on ${ipaddress_eth0}!\n",  # Print the eth0 IP fact
}
```
By default Puppet Server runs the commands in its manifests by default every 30 minutes. 

We can also test the manifest on a single node using ```puppet agent --test```. 

Rather than waiting for the Puppet master to apply the changes, we’ll apply the manifest now on db1:
```
sudo /opt/puppetlabs/bin/puppet agent --test
```
The output should look something like:
```
Output
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts
Info: Caching catalog for db1.localdomain
Info: Applying configuration version '1481131595'
Notice: /Stage[main]/Main/File[/tmp/it_works.txt]/ensure: defined content as '{md5}acfb1c7d032ed53c7638e9ed5e8173b0'
Notice: Applied catalog in 0.03 seconds
```
When it’s done, we’ll check the file contents:
```
cat /tmp/it_works.txt
```
