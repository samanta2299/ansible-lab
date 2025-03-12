# ansible-lab

Configuration files of the VM "Ansible_Control_Node"

# Example 1a: Deployment of NGINX Web Server

To start the configuration you must setup the inventroy file, which can be in two formats:
  -  hosts(.ini) 
  -  hosts.yml

For the configuration of the file "hosts":

nginx_node  ansible_host=192.168.56.251 ansible_user=samanta ansible_ssh_private_key_file=/home/samanta/.ssh/id_rsa

- nginx_node: is the alias of the Node that should be controlled using Ansible
- ansible_host=192.168.56.251: is the IP of the Host-Only network Adapter. To retrieve this information, you must execute the following command from the terminal of the VM "Nginx_Node":
  ```bash
ip -color a
```
