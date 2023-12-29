# ansible-setup-remote-server-userid-ssh-keys

# 🚀 Setup New User and SSH Key Auth on remote servers using Ansible on Ubuntu 🚀

https://github.com/coding-to-music/ansible-setup-remote-server-userid-ssh-keys


From / By https://poweradm.com/create-user-copy-ssh-key-ansible/

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

## Instructions

In this article, we’ll show you how to create user and add your public SSH key to remote Linux servers using Ansible.


Create anscfg user that will be used for remote management via Ansible:

```java
# groupadd -g 2002 anscfg
# useradd -u 2002 -g 2002 -c "Automation Account" -s /bin/bash -m -d /home/anscfg anscfg
```

Grant the user sudo permissions and set a password:

```java
# usermod -aG wheel anscfg
# passwd anscfg
```

Log in with a new user account:

```java
# su – anscfg
```

Create several Ansible directories:

```java
$ mkdir -p {playbooks,scripts,templates}
```

Generate an SSH key pair:

```java
$ ssh-keygen -t ed25519 -o -a 100 && ssh-keygen -t rsa -b 4096 -o -a 100
```

Add the addresses of the remote servers that you will manage to the hosts file:

```java
# nano /etc/hosts
192.168.31.21 nginx1
192.168.31.22 nginx2
192.168.31.23 nginx3
```

Now create an inventory file with Ansible client names and add your Linux hosts:

```java
$ nano /home/anscfg/inventory
```

file contents:

```java
[webservers]
nginx1
nginx2
nginx3
```

image of ansible inventory file

By default, Ansible uses the /etc/ansible/ansible.cfg configuration file. We will create our own configuration file:

```java
$ nano /home/anscfg/.ansible.cfg
```

contents of .ansible.cfg

```java
[defaults]
inventory = /home/anscfg/inventory
host_key_checking = False
```

- Note the dot before the ansible.cfg name. It tells Ansible to look for configuration files in the user’s home directory;
- The host_key_checking = False setting prevents Ansible from running a request to add a client key to known_hosts when running the Ansible playbook.

Create a new playbook useradd.yml:

$ nano /home/anscfg/playbooks/useradd.yml

```java
---
- hosts: webservers
  become: true
  tasks:

  - name: Create user anscfg
    user:
      name: anscfg
      password: set user password hash here. Ansible will not let you pass the password in clear text. Get your password hash with python (see the command below).
      shell: /bin/bash
      groups: wheel
      append: yes

  - name: Create an ssh key for anscfg user in ~anscfg/.ssh/id_rsa
    user:
      name: anscfg
      generate_ssh_key: yes
      ssh_key_bits: 2048
      ssh_key_file: .ssh/id_rsa

  - name: Add authorized key from id_rsa.pub file
    authorized_key:
      user: anscfg
      state: present
      key: "{{ lookup('file', '/home/anscfg/.ssh/id_rsa.pub') }}"
```

image of ansible playbook to create user and add ssh keys

Use the following command to get the password hash:

```java
python -c 'import crypt; print crypt.crypt("Passw0212")'
```

This playbook will create a sudo user anscfg on a remote host, and copy your public SSH key.

Set file permissions:

```java
$ chmod 644 /home/anscfg/playbooks/useradd.yml
```

Now you can run your playbook:

```java
$ ansible-playbook useradd.yml -u remote_user --ask-pass
```

After running the playbook on remote hosts, you can SSH into them without a password (using the private key):

```java
$ ssh nginx1
```

If you want to deploy playbooks without entering a password, you can add a username and password to the inventory file:

$ nano /home/anscfg/inventory

```java
[webservers]
nginx1 ansible_ssh_user=anscfg ansible_sudo_pass=<PASS>
nginx2 ansible_ssh_user=anscfg ansible_sudo_pass=<PASS>
nginx3 ansible_ssh_user=anscfg ansible_sudo_pass=<PASS>
```

Don’t forget to change the permissions on the inventory file to 600
