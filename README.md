This repository will provide solutions to the Red Hat Certified Engineer (RHCE EX294) exam preparation tasks. Tasks will be executed using Rocky Linux 9 systems in an Oracle VirtualBox environment. 

## 1.1 Preparing the environment in VirtualBox
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

## 1.2 Creating the user 'automation' on a control node and managed nodes

```bash
 useradd automation
 echo "devops" | passwd --stdin automation
 usermod -aG wheel automation
```

## 1.3 Passwordless SSH Configuration on control.lab.com
```bash
ssh-keygen -t rsa
ssh-copy-id node1.lab.com
ssh-copy-id node2.lab.com
ssh-copy-id node3.lab.com
ssh-copy-id node4.lab.com
ssh-copy-id node5.lab.com
```

## 1.4 Executing sudo commands without requiring a password on managed nodes

Ansible will execute the command with elevated privileges (become: true). This grants the automation user the ability to execute any commands with sudo without requiring a password on managed hosts. Add the following to the /etc/sudoers file:

student ALL=(ALL) NOPASSWD: ALL

## 1.5 Ansible installation and configutation on control.lab.com
```bash
dnf install ansible-core python3-pip vim -y
su - student
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
 inventory = /home/student/ansible/inventory  
 roles_path = /home/student/ansible/roles  
 collections_path = /home/student/ansible/mycollection  
 remote_user = student  
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
returns in this case "/home/student/ansible.cfg"

We add an entry and check it:
```bash
echo "export ANSIBLE_CONFIG=/home/student/ansible.cfg" >> .bashrc
cat .bashrc
```
We add the source:
```bash
source .bashrc
```

 ## 1.6 Connection test

Log in to your automation account on control.lab.com. 

```bash
cd /home/automation/plays
ansible all --list-hosts
ansible all -m ping
```

![alt text](./assets/1-6.png)  