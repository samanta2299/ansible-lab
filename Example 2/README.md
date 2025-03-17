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

## Verify the correct installation on the NGINX Node VM
From the VM NGINX Node, connect to: http://localhost:9100/metrics
And on this pae will be displayed the types of metrics that the node-exporter exposes to Prometheus
There are three types of metrics:
- **Gauge:** value that can increase or decrease over time
- **Counter:** metric that represents a monotonically increasing value
- **Histogram:** represents the distribution of a set of values over time
![Task image](images/NGINX_metrics.png)

## Promethus Web UI
Once Prometheus is installed, the Web User Interface (UI) can be accessed from the Monitoring_Node VM by connecting to: http://localhost:9090
![Task image](images/1.png)
Once in the Web UI, the nodes that Prometheus is connected to can be viewed by clicking: “Status -> Targets”
![Task image](images/2.png)
It is possible to check the monitored nodes and the configurations made in the "prometheus.yml" file (jobs_name and ansible_host_target)
![Task image](images/3.png)

## Grafana configuration
Access the Grafana web application from the Monitoring_Node VM by connecting to: http://127.0.0.1:3000
Log in with the default credentials:
- username: admin
- password: admin
![Task image](images/4.png)

### Data Source
To view the data, the data source must be specified in Grafana:
1. Click on Data Sources
2. Select Prometheus from the available options
![Task image](images/5.png)

Configuring the Data Source:
1. The name to associate with the source
2. The address of the source. In this case, the Prometheus server, which serves as the data source to be displayed, is located on the same VM as Grafana, so enter: http://localhost:9090
Once entered, click “save & test” at the bottom of the page
![Task image](images/6.png)

The monitored VM (NGINX_Node) hosts an NGINX web server. In addition to memory usage, it is possible to monitor network traffic as well
The following steps will show how to create a dashboard to monitor network traffic on the two interfaces of the NGINX VM

### Grafana dashboards
To create a new Dashboard, follow the highlighted boxes in the figure
1. This allows access to the Dashboard management panel
![Task image](images/7.png)

2. 	It allows the creation of a new Dashboard, a new folder to contain multiple Dashboards, or the import of a folder or a single dashboard
![Task image](images/8.png)


3. 	Select "Add visualization" to choose which data source to use
![Task image](images/9.png)

4.	Select Prometheus as data source
![Task image](images/10.png)

5.	Define the queries. Select Code (Builder is selected by default)
6. 	Enter the query
![Task image](images/11.png)

7. **First query: incoming traffic**: irate(node_network_receive_bytes_total{job=”nginx_node”, instance=”192.168.56.251”}[1m])
- irate() transforms the data provided by count-type metrics so that the actual trend (both positive and negative) of the metric inside the parentheses is shown on the graph. To do this, besides the metric, an evaluation interval must also be specified, in this case, [1m], which represents one minute
- node_network_receive_bytes_total and node_network_transmit_bytes_total are two count-type metrics that only provide monotonic incremental values, which does not allow for displaying a graph showing the traffic trend
![Task image](images/12.png)

In the "legend" section, select the "custom" option from the list and enter: traffic IN
8. 	Press the "Add query" button
![Task image](images/13.png)

9. **Second query: outgoing traffic**: irate(node_network_transmit_bytes_total{job=”nginx_node”, instance=”192.168.56.251”}[1m])
In the legend, select the "custom" option from the list and enter: traffic OUT
![Task image](images/14.png)

In the right menu, search for "unit" and select "data rate," then choose "kilobytes/sec"
![Task image](images/15.png)

To make the graph easier to understand, in the legend enter {{device}}, so that the names of the monitored network interfaces (enp0s3, enp0s8) are displayed
![Task image](images/16.png)

To **verify** that the network interfaces match, open the terminal of the NGINX_Node VM and run the following command:
```bash
ip --color a
```
Where:
enp0s3: NAT network interface
enp0s8: host-only network interface


