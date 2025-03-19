# ansible-lab

## Example 1a: Deployment of an NGINX Web Server
## Example 1b: Configuration of an NGINX Web Server
## Example 2: Configuration of a Prometheus + Grafana Monitoring System

## Prerequisites
**VirtualBox** v7.1.6
**VM Ansible Control Node** with Ubuntu 24.04 OS or 22.04, that will be cloned to create 2 VMs: NGINX_Node and Monitoring_Node
**VM Ansible Control Node** will have NAT network interface and Ansible installed using the following commands:
```bash
sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```
To verify that Ansible has been correctly installed run the command:
```bash
ansible --version
```

The VMs that will be controlled, **NGINX_Node** and **Monitoring_Node** should have NAT network interface and also "Host-Only" network interface
To add a "Host-Only" network adapter follow the instruction in the figure below:

Then, on the VMs that we want to controll using the Ansible Control Node, shoul be installed **openssh-server**
Open the terminal of the VM "NGINX_Node" and enter the following command:
```bash
sudo apt-get install openssh-server
```
Now do the same from the terminal of the VM "Monitoring_Node"

To verify the correct installation run the command:
```bash
sudo service ssh status
```
If it is not working, run the command:
```bash
sudo service ssh start
```

## Setup SSH
To ensure proper communication between the VM Ansible Control Node and the VM we want to control (target VMs: NGINX_Node and Monitoring_Node), it is necessary for target VMs to have the public SSH key of the Ansible Control Node:
### Check if on the VM Ansible Control Node exists a pair of SSH keys
```bash
ls -l ~/.ssh/id_*.pub
```
If the command returns something like "No such file or directory" or similar, it means that no SSH keys exist and they need to be generated, by using the command:
```bash
ssh-keygen -t rsa -b 4096 -C "your@email.com"
```
Press Enter for all subsequent prompts
If successful, the output of the process should be similar to the image

Verify the created keys using the following command:
```bash
ls ~/.ssh/id_*
```
Copy the public key from the VM Ansible Control Node to the target VM by running the following command from the VM Ansible Control Node terminal:
```bash
ssh-copy-id <remote-host-user>@<remote-host-ip> 
```
Knowing that:
- usename-host: corrispondes to the user of the VM target, for e.g., NGINX_Node user nginx
- ip-host: corrispondes to the IP of the VM target, for e.g., NGINX_Node IP 192.168.56.101
  To find the IP address of a VM, from its terminal run the command:
  ```bash
  ip -color a
  ```
  The IP address that we will be using, is the one related to the Host-Only network adapter, which is usually in the form 192.168.X.X (while the IP address of the NAT network adapter is typically in the form 10.0.X.X)


The command would be:
ssh-copy-id nginx@192.168.56.101
















