# ansible-setup-remote-server-userid-ssh-keys

# 🚀 Setup New User and SSH Key Auth on remote servers using Ansible on Ubuntu 🚀

https://github.com/coding-to-music/ansible-setup-remote-server-userid-ssh-keys


From / By Muhammad Arul

https://www.howtoforge.com/tutorial/setup-new-user-and-ssh-key-authentication-using-ansible/

## Environment variables:

```java
edit hosts file
```

## GitHub

```java
git init
git add .
git remote remove origin
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:coding-to-music/ansible-setup-remote-server-userid-ssh-keys.git
git push -u origin main
```

## Contents 

What we will do?
- Step 1 - Setup Ansible Control Machine
- Step 2 - Define User and SSH Key
- Step 3 - Create New Inventory
- Step 4 - Create Ansible Playbook
- Step 5 - Run the Playbook
- Step 6 - Testing

## Reference

Ansible is a simple automation tool that automates software applications deployment, cloud provisioning, and configuration management. It's a server orchestration tool that helps you to manage and control a large number of server nodes from single places called 'Control Machines'. Ansible was created by Michael DeHaan in 2012 and is written in Python and Powershell.

In this tutorial, we will learn how to deploy a new user and enable the SSH Key-Based authentication using the automation tool Ansible. We will also learn how to configure the Ansible 'Control Machine', as well as how to write simple ansible playbook.


## Prerequisites

- 2 or more Ubuntu 18.04 Servers
- 10.0.15.10      control-machine
- 10.0.15.21      ansi01
- 10.0.15.22      ansi02
- Root privileges

## What we will do?

- Setup Ansible Control Machine
- Define User and SSH Key
- Create Inventory File
- Create Ansible Playbook
- Deploy Server Using Playbook
- Testing

## Step 1 - Setup Ansible Control Machine

In this tutorial, we will be using the Ubuntu 16.04 servers as the Ansible 'Machine Control' and ansible hosts. The first step we need to do is to set up the 'control machine'.

We will install python and ansible on the ansible 'control machine' by running the following command.

```java
sudo apt install python ansible -y
```

Output

```java
Package python is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source
However the following packages replace it:
  2to3 python2-minimal:i386 python2:i386 python2-minimal python2 dh-python python-is-python3

E: Package 'python' has no installation candidate
```

## Setup Ansible Control Machine

After the installation is complete, we will add a new system user.

We will add a new user named 'provision' in order to perform server provisioning using Ansible.

Add new user 'provision' and give the user a password.

```java
sudo useradd -m -s /bin/bash provision

sudo passwd provision
```

Now add the 'provision' user for sudo without the password by creating new configuration file under the `/etc/sudoers.d/`` using the command below.

```java
sudo echo  -e 'provision\tALL=(ALL)\tNOPASSWD:\tALL' > /etc/sudoers.d/provision
```

A new user has been created, and now it can use sudo without a password.

## Add user

## Step 2 - Define User and SSH Key

Make sure the 'whois' package is installed on the system, or you can install using the following command.

```java
sudo apt install whois -y
```

In this step, we will define the user for ansible hosts. This user will be automatically created by ansible, so we just need to define the username, password, and the ssh public key.

For each server ('ansi01' and 'ansi02'), we will create a new user named 'provision' with password 'secret01'. And we need to encrypt the 'secret01' password using the mkpasswd command.

Encrypt the 'secret01' password using the command below.

```java
mkpasswd --method=SHA-512

# TYPE THE PASSWORD 'secret01'
```

And you will get the SHA-512 encrypted password.

## Define User and SSH Key

Next, we will generate a new ssh-key.

Login to the 'provision' user and generate the ssh key using the ssh-keygen command.

```java
su - provision
ssh-keygen -t rsa
```

Now the user and password have been defined, and the ssh key has been created l(ocated at the '.ssh' directory).

user and password have been defined

## Step 3 - Create New Inventory

In this step, we will define the inventory files for all server hosts.

Login as the 'provision' user and create a new directory for the project.

```java
su - provision
mkdir -p ansible01/
```

Go to the 'ansible01' directory and create a new inventory file 'inventory.ini' using vim.

```java
cd ansible01/
vim inventory.ini
```

Paste the following configuration there.

```java
[webserver]
 ansi01 ansible_host=10.0.15.21
 ansi02 ansible_host=10.0.15.22
```

Save and exit.

Now create a new ansible configuration file 'ansible.cfg'.

```java
vim ansible.cfg
```

Paste the following configuration there.

```java
[defaults]
 inventory = /home/provision/ansible01/inventory.ini
```

Save and exit.

## Create New Inventory

The ansible inventory file has been created, and our ansible scripts will be located under the 'provision' user, inside the 'ansible01' directory.

## Step 4 - Create Ansible Playbook

Ansible Playbook is set of instructions that you send to run on a single or group of server hosts. It represents the ansible-provisioning, where the automation is defined as tasks, and all jobs like installing packages, editing files, will be done by ansible modules.

In this step, we will create a new ansible playbook to deploy a new user, deploy the ssh key, and configure the ssh service.

Before we create a new ansible playbook, we will scan all server fingerprint using the ssh-keyscan command as below.

```java
ssh-keyscan 10.0.15.21 >> ~/.ssh/known_hosts
ssh-keyscan 10.0.15.22 >> ~/.ssh/known_hosts
```

Those servers fingerprint will be stored at the '.ssh/known_hosts' file.

## Create Ansible Playbook

Note:

If you have a lot of server nodes, you can save your host list and then manually scan the ssh key fingerprint using bash script as shown below.

```java
for i in $(cat list-hosts.txt)
do
ssh-keyscan $i >> ~/.ssh/known_hosts
done
```

Next, create the ansible playbook named 'deploy-ssh.yml' using vim.

```java
vim deploy-ssh.yml
```

Paste following the ansible playbook there.

```java
---
 - hosts: all
   vars:
     - provision_password: '$6$w9S3t7x1kRtmG0u$6nVU9KZsC12Q8DYI4FtgKPy.e/cq/jseB/.DViTO1SpUnoCy.dxcOf8hyfitGq5V0yhgXccxzlqm2o.I3SlDJ0'
  gather_facts: no
   remote_user: root
 
   tasks:
 
   - name: Add a new user named provision
     user:
          name=provision
          password={{ provision_password }}
 
   - name: Add provision user to the sudoers
     copy:
          dest: "/etc/sudoers.d/provision"
          content: "provision  ALL=(ALL)  NOPASSWD: ALL"
 
   - name: Deploy SSH Key
     authorized_key: user=provision
                     key="{{ lookup('file', '/home/provision/.ssh/id_rsa.pub') }}"
                     state=present
 
   - name: Disable Password Authentication
     lineinfile:
           dest=/etc/ssh/sshd_config
           regexp='^PasswordAuthentication'
           line="PasswordAuthentication no"
           state=present
           backup=yes
     notify:
       - restart ssh
 
   - name: Disable Root Login
     lineinfile:
           dest=/etc/ssh/sshd_config
           regexp='^PermitRootLogin'
           line="PermitRootLogin no"
           state=present
           backup=yes
     notify:
       - restart ssh
 
   handlers:
   - name: restart ssh
     service:
       name=sshd
       state=restarted
```

Save and exit.

On the playbook script:

- we create the 'deploy-ssh.yml' playbook script to be applied on all servers defined in the 'inventory.ini' file.
- we create the ansible variable 'provision_password', containing the encrypted password for the new user.
- Set the Ansible facts to 'no'.
- Define the 'root' user as a remote user to perform tasks automation.
- We create new tasks for adding a new user, add the user to the sudoers, and upload the ssh key.
- We create new tasks for configuring the ssh services, disabling the root login, and disable password authentication. Tasks for configuring the ssh will trigger the 'restart ssh' handlers.
- We create a handler to restart the ssh service.

## Step 5 - Run the Playbook
Login to the 'provision' user and go to the 'ansible01' directory.

```java
su - provision
cd ansible01/
```

Now run the the 'deploy-ssh.yml' playbook using the command as shown below.

```java
ansible-playbook deploy-ssh.yml --ask-pass

# Type your root password, and you will get the result as below.
```

Run the Playbook

All tasks for deploying a new user and ssh key have been completed successfully.

## Step 6 - Testing

Test using ansible command.

```java
ansible webserver -m ping
ansible webserver -m shell -a id
```

Now you will get the green messages as below.

## Testing Ansible

Now we can manage those 'ansi01' and 'ansi02' servers using Ansible, and the 'provision' user will be default user for Ansible.

Testing connection to the servers

```java
ssh 10.0.15.21
ssh 10.0.15.22
```

And you will be connected to each server using the default key '.ssh/id_rsa' file, and using the user 'provision'.

## Test server connection

Another test

Deploying new user and ssh-key using ansible has been completed successfully.

## Reference

http://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html
http://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html


