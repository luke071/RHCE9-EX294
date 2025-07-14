This repository will provide solutions to the Red Hat Certified Engineer (RHCE EX294) exam preparation tasks. Tasks will be executed using Rocky Linux 9 systems in an Oracle VirtualBox environment. 

## 1.1 Preparing the environment in VirtualBox
![alt text](./assets/diagram1.png)  

control.lab.com - control node (Ansible)    
node1.lab.com - managed node 
node2.lab.com - managed node  
node3.lab.com - managed node  
node4.lab.com - managed node  
node5.lab.com - managed node  

## 1.2 Creating the user 'user' and 'automation'
```bash
 useradd automation
 echo "devops" | passwd --stdin automation
 usermod -aG wheel automation
```

## 1.3 Passwordless SSH Configuration 

Log in to your automation account on control.lab.com.
```bash
ssh-keygen -t rsa
ssh-copy-id node1.lab.com
ssh-copy-id node2.lab.com
ssh-copy-id node3.lab.com
ssh-copy-id node4.lab.com
ssh-copy-id node5.lab.com
```

## 1.4 Creating directories and configuring Ansible
```bash
dnf install -y epel-release
dnf install -y ansible
```
## 1.5 Creating directories and configuring Ansible
```bash
mkdir -p /home/automation/plays/roles
mkdir -p /home/automation/plays/inventory
chown -R automation:automation /home/automation
```

/home/automation/plays/ansible.cfg

[defaults]  
 inventory = /home/automation/plays/inventory/hosts  
 roles_path = /home/automation/plays/roles  
 forks = 10  
 remote_user = automation  
 host_key_checking = False  
 [privilege_escalation]  
 become = False  

 /home/automation/plays/inventory/hosts  

 [proxy]  
 node1.lab.com  
 [webservers]  
 node2.lab.com  
 node3.lab.com
 [database]  
 node4.lab.com  
 node5.lab.com  

 ## 1.6 Connection test

Log in to your automation account on control.lab.com. 

```bash
cd /home/automation/plays
ansible all --list-hosts
ansible all -m ping
```

![alt text](./assets/1-6.png)  