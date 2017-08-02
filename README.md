# AnsibleMonitoring
## Description
![Alt text](/Sch%C3%A9ma%20Monitoring.png?raw=true "Schéma Monitoring")
Dockers Description
|  Docker Name  | Description   |
| ------------- | ------------- |
| cAdvisor  | https://github.com/google/cadvisor  |
| Node-Exporter |  https://github.com/prometheus/node_exporter  |
## Installation (on Remote Server)
~~~ shell
$ sudo apt-get install ssh
~~~
## Installation (on Monitoring Server)
Ansible Installation
http://docs.ansible.com/ansible/latest/intro_installation.html#installing-the-control-machine

~~~ shell
$ sudo apt-get install git ssh
$ git clone https://github.com/shakapark/AnsibleMonitoring/
$ cd AnsibleMonitoring
~~~

Configuration:

Change the file "hosts" : for all machines, change ip address, ssh port if needed, user name of the remote machine and password or private key for ssh connection.
~~~ shell
[local]
127.0.0.1 ansible_connection=ssh ansible_port=22 ansible_user=toto ansible_become_pass=Bonjour123 ansible_ssh_private_key_file=/home/toto/PrivKey.pem

[distant]
192.168.1.10 ansible_connection=ssh ansible_port=22 ansible_user=tetu ansible_become_pass=Bonjour456 ansible_ssh_private_key_file=/home/tetu/PrivKey2.pem
~~~
In the file install.yml, you can modify your installation by comment or supress the line you don't need (prerequisites.yml, prometheus_install.yml and grafana_install.yml must be install).
~~~ shell
- hosts: local 
  name: Installing monitoring server 
  tasks: 
    - include: prerequisites.yml 
    - include: node_exporter_install.yml 
#    - include: node_exporter_service_install.yml 
    - include: sensors_install.yml 
#    - include: cadvisor_install.yml 
    - include: prometheus_install.yml 
    - include: grafana_install.yml 

- hosts: distant 
  name: Installing Distant Exporter 
  tasks: 
    - include: prerequisites.yml 
    - include: node_exporter_install.yml 
    - include: node_exporter_service_install.yml 
    - include: sensors_install.yml 
    - include: cadvisor_install.yml 

~~~
In 'conf' folder, you must change the prometheus configuration ("prometheus.yml") to monitor remote machine (change 'IpNode1' by an ip address that the monitoring server can contact, add more if you want).
~~~ shell
global:
 scrape_interval: 15s

scrape_configs:
 - job_name: prometheus
   static_configs:
    - targets: ['localhost:9090']

 - job_name: node
   metrics_path: /metrics
   static_configs:
    - targets: ['192.168.1.9:9100','192.168.1.10:9100',...]
 
 - job_name: service
   metrics_path: /metrics
   static_configs:
    - targets: ['192.168.1.9:9110','192.168.1.10:9110',...]

 - job_name: cadvisor
   static_configs:
     - targets: ['192.168.1.9:8090','192.168.1.10:8090',...]
~~~

You can change too the grafana configuration ("grafana.ini")

Monitoring Installation
~~~ shell
$ ansible-playbook -i hosts install.yml
~~~
