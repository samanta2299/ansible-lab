# Exercise 2: Configuration of a Prometheus and Grafana Monitoring System

Installation and configuration of the Prometheus + Grafana stack on the Monitoring_Node VM to monitor the NGINX_Node VM (target node)

![Task image](images/task_ex_2.png)

From the Ansible Control Node, move to the directory roles:
```bash
cd /etc/ansible/roles
```
And create the roles:
```bash
sudo ansible-galaxy init grafana
```
```bash
sudo ansible-galaxy init prometheus
```
```bash
sudo ansible-galaxy init node-exporter
```

Move to the directory playbooks:
```bash
cd ../playbooks
```
Create the playbook monitoring_system.yml:
```bash
sudo nano monitoring_system.yml
```
```bash
---
- hosts: nginx_node
  become: yes
  vars_files:
    - /etc/ansible/roles/nginx/vars/nginx_vars.yml
  roles:
    - nginx
    - node-exporter

- hosts: monitoring_node
  become: yes
  vars_files:
    - /etc/ansible/roles/prometheus/vars/main.yml
  roles:
    - prometheus
    - grafana
```

Press CTRL + X to close the editor and enter y to save

Move to the directory roles/node-exporter/tasks:
```bash
cd ../roles/node-exporter/tasks
```
Modify the file main.yml:
```bash
---
# tasks file for node-exporter
- name: "apt-get update"
  apt:
    update_cache: yes
    cache_valid_time: 3600

- name: "Install node-exporter"
  apt:
    name: ['prometheus-node-exporter']
    state: latest

- name: "Output Node-exporter"
  debug:
    msg: "Connect from the Monitoring_Node to http://localhost:9100/metrics"
```
Press CTRL + X to close the editor and enter y to save

Move to the directory roles/prometheus/tasks:
```bash
cd ../../prometheus/tasks
```
Modify the file main.yml:
```bash
---
# tasks file for prometheus
- name: "apt-get update"
  apt:
    update_cache: yes
    cache_valid_time: 3600

- name: "Install & update prometheus"
  apt:
    name: ['prometheus']
    state: latest
    update_cache: yes
    cache_valid_time: 3600

- name: "Move the configuration file template"
  template:
    src: ../templates/prometheus.yml.j2
    dest: "{{ prom_dir_config }}/prometheus.yml"
    mode: '0755'
    owner: prometheus
    group: prometheus

- name: "Restart prometheus"
  service:
    name: prometheus
    state: restarted
    enabled: yes

- name: "Output Prometheus"
  debug:
    msg: "Prometheus is running. Connect from the Monitoring_Node to http://localhost:9090"
```
Press CTRL + X to close the editor and enter y to save

Move to the directory roles/prometheus/templates:
```bash
cd ../templates
```
Modify the file prometheus.yml.j2:
```bash
--
global:
  scrape_interval: {{ scrape_interval }}
  evaluation_interval: {{ evaluation_interval }}

scrape_configs:
  - job_name: "nginx_node"
    static_configs:
      - targets: ['{{ ansible_host_target }}:9100']
```
Press CTRL + X to close the editor and enter y to save

Move to the directory roles/prometheus/vars:
```bash
cd ../vars
```
Modify the file main.yml:
**Note.** In the variable ansible_host_target insert the IP of the VM you want to monitor (NGINX_Node):
```bash
---
# vars file for prometheus
prom_dir_config: "/etc/prometheus"
scrape_interval: "10s"
evaluation_interval: "10s"
ansible_host_target: "192.168.56.251"
```
Press CTRL + X to close the editor and enter y to save


Move to the directory /etc/ansible/roles/grafana/tasks:
```bash
cd ../../grafana/tasks
```
Modify the file main.yml:
```bash
sudo nano main.yml
```
```bash
--
# tasks file for grafana
- name: "Install gpg"
  apt:
    name: gnupg, software-properties-common
    state: present
    update_cache: yes
    cache_valid_time: 3600

- name: "add gpg key"
  apt_key:
    url: "https://packages.grafana.com/gpg.key"
    validate_certs: no

- name: "add repository"
  apt_repository:
    repo: "deb https://packages.grafana.com/oss/deb stable main"
    validate_certs: no

- name: "install and update Grafana"
  apt:
    name: grafana
    state: latest
    update_cache: yes
    cache_valid_time: 3600

- name: "Start service grafana-server"
  service:
    name: grafana-server
    enabled: true
    state: restarted

- name: "Check if Grafana is accessible"
  uri:
    url: http://127.0.0.1:3000
    method: GET
  register: __result
  until: __result.status == 200
  retries: 120
  delay: 1

- name: "Output Grafana"
  debug:
    msg: "Grafana is running. Connect from the Monitoring_Node to http://127.0.0.1:3000. Username: admin, Password:>
```
Press CTRL + X to close the editor and enter y to save

Move to the directory playbooks:
```bash
cd ../../../playbooks
```
Exeecute the playbook monitoring_system.yml:
```bash
ansible-playbook monitoring_system.yml
```

## Verify the correct installation
From the VM NGINX Node, connect to: http://localhost:9100/metrics
And on this pae will be displayed the types of metrics that the node-exporter exposes to Prometheus
There are three types of metrics:
- **Gauge:** value that can increase or decrease over time
- **Counter:** metric that represents a monotonically increasing value
- **Histogram:** represents the distribution of a set of values over time
![Task image](images/1.png)














