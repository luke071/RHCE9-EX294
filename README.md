This repository will provide solutions to the Red Hat Certified Engineer (RHCE EX294) exam preparation tasks. Tasks will be executed using Rocky Linux 9 systems in an Oracle VirtualBox environment. 

## Question 1 
### 1.1 Preparing the environment in VirtualBox
![alt text](./assets/diagram1.png)  

control.lab.com - control node (Ansible)    
node1.lab.com - managed node  
node2.lab.com - managed node  
node3.lab.com - managed node  
node4.lab.com - managed node  
node5.lab.com - managed node  

```bash
cat /etc/hosts
```
![alt text](./assets/1-1.png)  

### 1.2 Creating the user 'automation' on a control node and managed nodes

```bash
 useradd automation
 echo "devops" | passwd --stdin automation
 usermod -aG wheel automation
```

### 1.3 Passwordless SSH Configuration on control.lab.com
```bash
ssh-keygen -t rsa
ssh-copy-id node1.lab.com
ssh-copy-id node2.lab.com
ssh-copy-id node3.lab.com
ssh-copy-id node4.lab.com
ssh-copy-id node5.lab.com
```

### 1.4 Executing sudo commands without requiring a password on managed nodes

Ansible will execute the command with elevated privileges (become: true). This grants the automation user the ability to execute any commands with sudo without requiring a password on managed hosts. Add the following to the /etc/sudoers file:

automation ALL=(ALL) NOPASSWD: ALL

### 1.5 Ansible installation and configutation on control.lab.com
```bash
dnf install ansible-core python3-pip vim -y
su - automation
pip install ansible-navigator
```

Creating directories and configuring
```bash
mkdir -p /home/automation/ansible/mycollection
mkdir -p /home/automation/ansible/roles
chown -R automation:automation /home/automation
```

/home/automation/plays/ansible.cfg

 [defaults]  
 inventory = /home/automation/ansible/inventory  
 roles_path = /home/automation/ansible/roles  
 collections_path = /home/automation/ansible/mycollection  
 remote_user = automation  
 host_key_checking = False  
 [privilege_escalation]  
 become = True  
 become_method = sudo  
 become_user = root  
 become_ask_pass = false  

 /home/automation/plays/inventory/hosts  

 [proxy]  
 node1.lab.com  
 [webservers]  
 node2.lab.com  
 node3.lab.com  
 [database]  
 node4.lab.com  
 node5.lab.com  


Then we execute the command:

```bash
ansible --version
```
returns in this case "/home/automation/ansible.cfg"

We add an entry and check it:
```bash
echo "export ANSIBLE_CONFIG=/home/automation/ansible.cfg" >> .bashrc
cat .bashrc
```
We add the source:
```bash
source .bashrc
```

### 1.6 Connection test

Log in to your automation account on control.lab.com. 

```bash
cd /home/automation/plays
ansible all --list-hosts
ansible all -m ping
```

![alt text](./assets/1-6.png)  

## Question 2  
Create and run an Ansible ad-hoc command. As a system administrator, you will need to install software on the managed node.  
a -  Create a shell script called yum-repo.sh that runs Ansible ad-hoc commands to create the yum repositories on ech of the managed nodes as per the followind details.  
b - NOTE: you need to create 2 repos (BaseOS & AppStream) un the managed nodes.   

[BaseOS]  
name = BaseOS  
baseurl = file:///mnt/BaseOS/  
description : Base OS Repo  
gpgcheck = 1  
enabled = 1  
gpkey: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release  

 
[AppStream]  
name = AppStream  
baseurl = file:///mnt/AppStream/  
description : AppStream Repo  
gpgcheck = 1  
enabled = 1  
gpkey: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release  

```bash
cd ansible/
nano yum-repo.sh
```

```bash
#!/bin/bash

ANSIBLE_CONFIG=/home/automation/ansible/ansible.cfg

ansible all -m copy -a "content='[BaseOS]
name=BaseOS
baseurl=file:///mnt/BaseOS/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
' dest=/etc/yum.repos.d/baseos.repo"

ansible all -m copy -a "content='[AppStream]
name=AppStream
baseurl=file:///mnt/AppStream/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
' dest=/etc/yum.repos.d/appstream.repo"
```

```bash
chmod +x yum-repo.sh
./yum-repo.sh
ansible all -m command -a 'dnf repolist all'
ansible all -m command -a 'ls /etc/yum.repos.d/' -b
```
Checking if repository files have been created in node1.lab.com:  

![alt text](./assets/2-1.png)  

## Question 3 
Create a playbook called /home/automation/automation/ansible/packages.yml that:  
- Installs the php and postgresql packages on hosts in the dev, test, and prod host groups only.  
- Installs the RPM Development Tools package group on hosts in the dev host group only.  
- Updates all paclages to the lates veriosn on hosts in the dev host group only.  

```bash
nano /home/automation/ansible/packages.yml
```
Create a Playbook:

```bash
--- 
- name: Install PHP and PostgreSQL  
  hosts: all  
  become: true
  tasks:  
    - name: install the packages  
      dnf:  
       name: "{{ item }}"  
       state: present  
      loop:  
       - php  
       - postgresql  
      when: inventory_hostname in groups['dev'] or inventory_hostname in groups ['test']or inventory_hostname in gropups['prod']  
    - name: install the RPM development tool package group  
      dnf:  
       name: "@RPM Development tools"  
       state: present  
      when: inventory_hostbame in groups['dev']  
    - name: update all packages  
      dnf:  
       name: '*'  
       state: latest  
      when: inventory_hostname in groups['dev']   
```

Execute the Playbook:

```bash
ansible-playbook /home/automation/ansible/packages.yml
```

## Question 4

Install the RHEL system roles package and create a playbook called:  
/home/automation/ansible/timesync.yml  
1. Runs on all the managed hosts.  
2. Uses the timesync role.  
3. Configure the role to use the time server 0.pool.ntp.org  
4. Configures the role to set the iburst parameter as enabled.  

```bash
dnf install rhel-system-roles -y
nano /home/automation/ansible/timesync.yml
```

Create a Playbook:

```bash
--- 
- name: Time Sync  
  hosts: all  
  become: true
  vars:  
    timesync_ntp_servers:
      - hostname: 0.pool.ntp.org
      iburst: yes  
  roles:
    - /usr/share/ansible/roles/rhel-system-roles.timesync
```

Execute the Playbook:

```bash
ansible-playbook /home/automation/ansible/timesync.yml
```

The iburst option in NTP involves sending eight queries to servers simultaneously during the initial synchronization.  