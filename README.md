# Install-Puppet-4-on-Ubuntu

Puppet is a configuration management tool that helps system administrators automate the provisioning, configuration and management of a server infrastructure.
Puppet comes in two varieties, Puppet Enterprise and open source Puppet.
The Puppet master server—which runs the Puppet Server software—can be used to control all your other servers, called Puppet agent nodes.

## Prerequisites
* at least 4GB of memory
* at least 2 CPU cores

## Step 1 — Configuring /etc/hosts

On each agent machine, edit the /etc/hosts file. At the end of the file, specify the Puppet master server as follows, substituting the IP address for your Puppet master:

sudo nano /etc/hosts
/etc/hosts
```
puppet-master_ip_address    puppet puppet-master
```

## Step 2 — Installing Puppet Server

We’ll enable the official Puppet Labs collection repository with these commands:

```
wget https://apt.puppetlabs.com/puppet7-release-focal.deb 
sudo dpkg -i puppet7-release-focal.deb 
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
wget https://apt.puppetlabs.com/puppet7-release-focal.deb 
sudo dpkg -i puppet7-release-focal.deb 

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
sudo /opt/puppetlabs/bin/puppetserver ca sign --all 
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

=========================================================================

package {'openssl':
         ensure => present,
         before => File['/etc/ssh/sshd_config'],
         }
         
==========================================================================

file {'ssh-config':
      ensure => file
      path => '/etc/ssh/ssh_config',
      mode => 600	,
      source => <source file name location>
      }
      
==========================================================================

service {'sshd':
        ensure => running,
        enable => true,
        subscribe => File['/etc/ssh/sshd_config']  # wheneven any changes in the main config file the service ssh restar
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














##############################################################################################################
                                      
                                      
                                      
                                      
#############################################################################################################



## Step 1 – Configure Hosts

Puppet master and client nodes uses hostnames to communicate with each other. So its good to start with assigning a unique hostname for each node.


1. Login to the master and each client node one by one and edit /etc/hosts file:
```
sudo nano /etc/hosts 
```
2. Add the following entries at the end of each hosts file:

```
10.132.14.239 puppetmaster puppet
10.132.14.240 puppetclient1
10.132.14.241 puppetclient2
```
Here:

10.132.14.239 is the IP address of the master node.
10.132.14.240 is the IP address of the client node.
10.132.14.242 is the IP address of another client node.
Add more client nodes, you required
Save your file and close it. To save file with nano editor press Ctrl + X and then type Y and press Enter to save the change and close file.

## Step 2 – Install Puppet Server (Master Node)

Now, login to the Master node with the shell access


3. Install latest Puppet debian package to configure PPA on the master node:

```
wget https://apt.puppetlabs.com/puppet7-release-focal.deb 
sudo dpkg -i puppet7-release-focal.deb 
```
4. Once you added the PPA, update Apt cache and install the Puppet server with the following command:
```
sudo apt update 
sudo apt install puppetserver -y 
```
5. After successfully installation of all the Puppet packages. Edit the puppet server file by using:

```sudo nano /etc/default/puppetserver```
The default puppet server file configured to use 2GB of memory. In case your server doesn’t have enough memory. Reduce the memory size to 1GB or any other value:
```
JAVA_ARGS="-Xms1g -Xmx1g -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
```
Save you changes and close puppetserver file. To save file with nano editor press Ctrl + X and then type Y to save the changes.

6. Next, start the Puppet service and set it to auto-start on system boot:

```
sudo systemctl start puppetserver 
sudo systemctl enable puppetserver 
```
7. Once service is started, verify the service status with:

```
sudo systemctl status puppetserver 
```
You will see the service status as running.

Now, start with the configuration of all client node.

## Step 3 – Install Puppet Agent (Client Node)

First of all, make sure you already have updated hosts file entries defines in step 1 in all client nodes.

8. Now, download and install latest Puppet debian package to configure PPA on your client node:

```
wget https://apt.puppetlabs.com/puppet7-release-focal.deb 
sudo dpkg -i puppet7-release-focal.deb 
```
9. Once you configured the PPA, Install the Puppet agent package on all client servers.

```
sudo apt update 
sudo apt install puppet-agent -y 
```
10. Once the packages installation finished. Edit the Puppet configuration file:

```
sudo nano /etc/puppetlabs/puppet/puppet.conf 
```
Add the following entries to the end of the Puppet configuration file to define the Puppet master node details:

```
[main]
certname = puppetclient1
server = puppetmaster
```
Save your file and close it.

11. Next, start the Puppet agent service on all the client nodes and set it to auto-start on system boot:

```
sudo systemctl start puppet 
sudo systemctl enable puppet 
```
12. Once done, verify the Puppet agent service is running properly:

```
sudo systemctl status puppet 
```
You should see a running status on all the agent systems

## Step 4 – Sign the Puppet Agent Certificates
13. Your have done with the configuration’s. Now, login to the Puppet master node and run the following command to list all the available certificates:

```
sudo /opt/puppetlabs/bin/puppetserver ca list --all 
```
Puppet list all certificates

14. Next, sign all the clients certificates using:

```
sudo /opt/puppetlabs/bin/puppetserver ca sign --all 
```
Puppet sign all client certificates

15. Finally, test the communication between Puppet master and client nodes using the following command.

```
sudo /opt/puppetlabs/bin/puppet agent --test 
```
Puppet test connection to client
