# Chapter 3

## 3.1 Creating project specific ansible configuration
###  Ansible configuration file
In chapter3 directory create a file called ansible.cfg  
Put the following content in ansible.cfg

{title="Listing ", lang=html, linenos=off}
~~~~~~~
[defaults]

remote_user = vagrant
inventory   = myhosts.ini
~~~~~~~

## 3.2 Setting up inventory of hosts  
### Inventories:  

For setting up inventories, create a new file called *myhosts.ini*   
Let's create a group called *localhost* by editing myhosts.ini  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
[local]
localhost ansible_connection=local
~~~~~~~
Adding other groups to the inventory file
{title="Listing ", lang=html, linenos=off}
~~~~~~~
[app]
192.168.61.12
192.168.61.13

[db]
192.168.61.11
~~~~~~~

Now save and quit the file. This is called an inventory and this is how it should look like...  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
[local]
localhost ansible_connection=local

[app]
192.168.61.12
192.168.61.13

[db]
192.168.61.11

~~~~~~~

Make sure it has a *.ini* extension.  


## 3.3 Setting up passwordless ssh access to inventory hosts  
### 3.3.1 Generating ssh keypair on control host  

Now on control host, execute the following command  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ssh-keygen -t rsa
~~~~~~~

Now press enter for the passphrase and other queries.
{title="Listing ", lang=html, linenos=off}
~~~~~~~
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
c5:a5:6d:60:56:5a:7b:3c:60:23:b5:0f:1b:cf:f9:fd root@ansible
The key's randomart image is:
+--[ RSA 2048]----+
|          =oO    |
|         + X *   |
|          = B +  |
|         . . O o |
|        S   . =  |
|               ..|
|                o|
|                .|
|                E|
+-----------------+

~~~~~~~

### 3.3.2 Copying public key to inventory hosts  
Copy public key of control node to other hosts  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ssh-copy-id vagrant@192.168.61.11

ssh-copy-id vagrant@192.168.61.12

ssh-copy-id vagrant@192.168.61.13

ssh-copy-id vagrant@192.168.61.14
~~~~~~~  

See this example output to verify with your output  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
The authenticity of host '192.168.61.11 (192.168.61.11)' can't be established.
RSA key fingerprint is 32:7f:ad:d7:da:63:32:b6:a9:ff:59:af:09:1e:56:22.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.61.11' (RSA) to the list of known hosts.

~~~~~~~  

The password for user *vagrant* is *vagrant*  

### 3.3.3 Validate the passwordless login  
Let us check the connection of control node with other hosts  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ssh vagrant@192.168.61.11

ssh vagrant@192.168.61.12

ssh vagrant@192.168.61.13

ssh vagrant@192.168.61.14
~~~~~~~

### 3.3.4 Ansible ping  
We will use Ansible to make sure all the hosts are reachable  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ansible all -m ping
~~~~~~~  

Output of this command will be...  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
192.168.61.13 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.61.11 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.61.12 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
~~~~~~~

## 3.4 Ad Hoc commands:  
Try running following *fire-and-forget* Ad-Hoc commands...  

### 3.4.1 Run *hostname* command on all hosts  
Let us print the hostname of all the hosts  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ansible all -a hostname
~~~~~~~  

The output will be...  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
localhost | SUCCESS | rc=0 >>
ansible

192.168.61.11 | SUCCESS | rc=0 >>
db

192.168.61.12 | SUCCESS | rc=0 >>
app

192.168.61.13 | SUCCESS | rc=0 >>
app

~~~~~~~

### 3.4.2 Check the *uptime*  
How long the hosts are *up*?  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ansible all -a uptime
~~~~~~~  

The output will be...  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
localhost | SUCCESS | rc=0 >>
 13:17:13 up  2:21,  1 user,  load average: 0.16, 0.03, 0.01

192.168.61.12 | SUCCESS | rc=0 >>
 13:17:14 up  1:50,  2 users,  load average: 0.00, 0.00, 0.00

192.168.61.13 | SUCCESS | rc=0 >>
 13:17:14 up  1:47,  2 users,  load average: 0.00, 0.00, 0.00

192.168.61.11 | SUCCESS | rc=0 >>
 13:17:14 up  1:36,  2 users,  load average: 0.00, 0.00, 0.00
~~~~~~~

### 3.4.3 Check memory info on app servers  
Does my app servers have any disk space *free*?  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ansible app -a free
~~~~~~~  

The output will be...  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
192.168.61.13 | SUCCESS | rc=0 >>
             total       used       free     shared    buffers     cached
Mem:        372916     121480     251436        776      11160      46304
-/+ buffers/cache:      64016     308900
Swap:      4128764          0    4128764

192.168.61.12 | SUCCESS | rc=0 >>
             total       used       free     shared    buffers     cached
Mem:        372916     121984     250932        776      11228      46336
-/+ buffers/cache:      64420     308496
Swap:      4128764          0    4128764
~~~~~~~  


### 3.4.4 Installing packages  
Let us *install* Docker on app servers  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ansible app -a "yum install -y docker-engine"
~~~~~~~  

This command will fail.

{title="Listing ", lang=html, linenos=off}
~~~~~~~
192.168.61.13 | FAILED | rc=1 >>
Loaded plugins: fastestmirror, prioritiesYou need to be root to perform this command.

192.168.61.12 | FAILED | rc=1 >>
Loaded plugins: fastestmirror, prioritiesYou need to be root to perform this command.
~~~~~~~  

Run the fillowing command with sudo permissions.  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ansible app -s -a "yum install -y docker-engine"
~~~~~~~  

This will install docker in our app servers  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
192.168.61.12 | SUCCESS | rc=0 >>
Loaded plugins: fastestmirror, priorities
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.nhanhoa.com
 * epel: mirror.rise.ph
 * extras: mirror.fibergrid.in
 * updates: mirror.fibergrid.in
283 packages excluded due to repository priority protections
Resolving Dependencies
--> Running transaction check
---> Package docker-engine.x86_64 0:1.7.1-1.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package             Arch         Version              Repository          Size
================================================================================
Installing:
 docker-engine       x86_64       1.7.1-1.el6          local_docker       4.5 M

Transaction Summary
================================================================================
Install       1 Package(s)

Total download size: 4.5 M
Installed size: 19 M
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : docker-engine-1.7.1-1.el6.x86_64                             1/1
  Verifying  : docker-engine-1.7.1-1.el6.x86_64                             1/1

Installed:
  docker-engine.x86_64 0:1.7.1-1.el6

Complete!

192.168.61.13 | SUCCESS | rc=0 >>
Loaded plugins: fastestmirror, priorities
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirror.fibergrid.in
 * epel: mirror.rise.ph
 * extras: mirror.fibergrid.in
 * updates: mirror.fibergrid.in
283 packages excluded due to repository priority protections
Resolving Dependencies
--> Running transaction check
---> Package docker-engine.x86_64 0:1.7.1-1.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package             Arch         Version              Repository          Size
================================================================================
Installing:
 docker-engine       x86_64       1.7.1-1.el6          local_docker       4.5 M

Transaction Summary
================================================================================
Install       1 Package(s)

Total download size: 4.5 M
Installed size: 19 M
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : docker-engine-1.7.1-1.el6.x86_64                             1/1
  Verifying  : docker-engine-1.7.1-1.el6.x86_64                             1/1

Installed:
  docker-engine.x86_64 0:1.7.1-1.el6

Complete!

~~~~~~~

### 3.4.5 Running commands one machine at a time  
Do you want a command to run on *one machine at a time* ?  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ansible all -f 1 -a "free"
~~~~~~~  

## 3.5 Using *modules* to manage the state of infrastructure  
### 3.5.1 Creating users and groups using *user* and *group*  
To create a group  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ansible app -s -m group -a "name=admin state=present"
~~~~~~~  

The output will be,

{title="Listing ", lang=html, linenos=off}
~~~~~~~
192.168.61.13 | SUCCESS => {
    "changed": true,
    "gid": 501,
    "name": "admin",
    "state": "present",
    "system": false
}
192.168.61.12 | SUCCESS => {
    "changed": true,
    "gid": 501,
    "name": "admin",
    "state": "present",
    "system": false
}
~~~~~~~

To create a user  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ansible app -s -m user -a "name=devops group=admin createhome=yes"
~~~~~~~  

This will create user *devops*,

{title="Listing ", lang=html, linenos=off}
~~~~~~~
192.168.61.13 | SUCCESS => {
    "changed": true,
    "comment": "",
    "createhome": true,
    "group": 501,
    "home": "/home/devops",
    "name": "devops",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 501
}
192.168.61.12 | SUCCESS => {
    "changed": true,
    "comment": "",
    "createhome": true,
    "group": 501,
    "home": "/home/devops",
    "name": "devops",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 501
}
~~~~~~~

### 3.5.2 Copy a file using *copy* modules  
We will copy file from control node to app servers.  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
ansible app -m copy -a "src=/vagrant/test.txt dest=/tmp/test.txt"
~~~~~~~  

File will be copied over to our app server machines...  

{title="Listing ", lang=html, linenos=off}
~~~~~~~
192.168.61.13 | SUCCESS => {
    "changed": true,
    "checksum": "3160f8f941c330444aac253a9e6420cd1a65bfe2",
    "dest": "/tmp/test.txt",
    "gid": 500,
    "group": "vagrant",
    "md5sum": "9052de4cff7e8a18de586f785e711b97",
    "mode": "0664",
    "owner": "vagrant",
    "size": 11,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1472991990.29-63683023616899/source",
    "state": "file",
    "uid": 500
}
192.168.61.12 | SUCCESS => {
    "changed": true,
    "checksum": "3160f8f941c330444aac253a9e6420cd1a65bfe2",
    "dest": "/tmp/test.txt",
    "gid": 500,
    "group": "vagrant",
    "md5sum": "9052de4cff7e8a18de586f785e711b97",
    "mode": "0664",
    "owner": "vagrant",
    "size": 11,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1472991990.26-218089785548663/source",
    "state": "file",
    "uid": 500
}
~~~~~~~

## 3.6 Exercises :
1. Add another group called *lb* in inventory with respective host ip
2. Add a user called *joe* in app servers. Make sure that user has a home directory.
3. Install the package cowsay using the correct Ad-Hoc command.