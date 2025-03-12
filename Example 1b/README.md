# Example 1b: Configuration of NGINX Web Server

Create a new role, start from the the default directory:
```bash
$ cd /etc/ansible/roles
```
And execute the command:
```bash
$ sudo ansible-galaxy init nginx
```
```bash
$ cd /nginx/templates/site.conf.j2
```

Modify the connfiguration file using Jinja2 Template:
```bash
$ sudo nano site.conf.j2
```

```bash
server{
   listen {{ port_number }};
   root /var/www/{{ server_name }};
   server_name {{ server_name }};
   location / {
        try_files $uri $uri/ =404;
   }
}
```

Move to the directory vars, using the command: 
```bash
$ cd ../vars
```
Modify the nginx_vars.yml file:
```bash
$ sudo nano nginx_vars.yml
```

```bash
---
# vars file for nginx
port_number: 8080
server_name: nginx_static
```

Move to the directory files, using the command: 
```bash
$ cd ../files
```

Modify the index.html file:
```bash
$ sudo nano index.html
```

```bash
<!DOCTYPE html>
<html>
  <head>
     <title> Welcome to NGINX example! </title>
  </head>
  <body>
     <h1> Welcome to Ansible x NGINX </h1>
     <img src="/ansible_logo.png" />
     <img src="/nginx_logo.png" />
  </body>
</html>
```
Move to the directory files, using the command: 
```bash
$ cd ../tasks
```

Modify the main.yml file:
```bash
$ sudo nano main.yml
```

```bash
---
# tasks file for nginx
- name: "apt-get update"
  apt:
    update_cache: yes
    cache_valid_time: 3600

- name: "create /var/www/{{ server_name }} directory"
  file:
    path: /var/www/{{ server_name }}
    state: directory
    mode: '0755'
    owner: "{{ ansible_user }}"
    group: "{{ ansible_user }}"

- name: "install nginx"
  apt:
    name: ['nginx']
    state: latest

- name: "copy the nginx config file"
  template:
    src: /etc/ansible/roles/nginx/templates/site.conf.j2
    dest: /etc/nginx/sites-enabled/{{ server_name }}
    owner: root
    group: root
    mode: '0644'

- name: "copy the content of the website"
  copy:
    src: /etc/ansible/roles/nginx/files/
    dest: /var/www/{{ server_name }}
    owner: root
    group: root
    mode: '0644'

- name: "restart nginx"
  service:
    name: nginx
    state: restarted
```

Move to the directory: "/etc/ansible/playbooks":

```bash
$ cd ../../../playbooks
```

```bash
$ sudo nano nginx_playbook.yml
```

```bash
---
- hosts: nginx_node
  become: yes
  vars_files:
    - /etc/ansible/roles/nginx/vars/nginx_vars.yml
  roles:
    - nginx
```























