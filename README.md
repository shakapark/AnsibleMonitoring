# AnsibleMonitoring
## Description
![Schéma Monitoring](Screens/Sch%C3%A9ma%20Monitoring.png?raw=true "Schéma Monitoring")

The Optional dockers are use to get metrics on the machine :
- cAdvisor : Get metrics about containers
- Node-Exporter : Get hardware metrics of the machine 
- Node-Exporter Service : Get state about Systemd
- SensorsDocker : Get temperature of processor
- Snmp Exporter : Get results of snmp request (NOT IMPLEMENTED YET)

Prometheus collect those metrics and add a system of request

Grafana use prometheus requests to create graphs. 

### Dockers Links

|  Docker Name  | Description  |
| ------------- | ------------- |
| cAdvisor  | https://github.com/google/cadvisor |
| Node-Exporter |  https://github.com/prometheus/node_exporter |
| SensorsDocker | https://github.com/Shakapark/SensorsDocker |
| Snmp Exporter | https://github.com/prometheus/snmp_exporter |
| Prometheus | https://github.com/prometheus/prometheus |
| Grafana | https://github.com/grafana/grafana |

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

### Configuration:

Change the file "hosts" : for all machines, change ip address, ssh port if needed, user name of the remote machine and password or private key for ssh connection.
Example:
~~~ shell
[local]
127.0.0.1 ansible_connection=ssh ansible_port=22 ansible_user=toto ansible_become_pass=Bonjour123 ansible_ssh_private_key_file=/home/toto/PrivKey.pem

[distant]
192.168.1.10 ansible_connection=ssh ansible_port=22 ansible_user=tetu ansible_become_pass=Bonjour456 ansible_ssh_private_key_file=/home/tetu/PrivKey2.pem
~~~

If you don't want to use the sudo password in the hosts file ("ansible_become_pass"), you can disable it for the command sudo. For that, edit the file "/etc/sudoers" and add a line for the users you have put in hosts file (on each machine).
~~~ shell
#includedir /etc/sudoers.d
toto ALL=(ALL) NOPASSWD:ALL
~~~
In the file install.yml, you can modify your installation by comment or supress the line you don't need (prerequisites.yml, prometheus_install.yml and grafana_install.yml must be install) (SensorsDocker need Node-Exporter).
Example:
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

Example:
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
    - targets: ['192.168.1.9:9100','192.168.1.10:9100']
 
 - job_name: service
   metrics_path: /metrics
   static_configs:
    - targets: ['192.168.1.9:9110','192.168.1.10:9110']

 - job_name: cadvisor
   static_configs:
     - targets: ['192.168.1.9:8090','192.168.1.10:8090']
~~~

You can change too the grafana configuration ("grafana.ini")

The ports 3000 (grafana), 8090 (cAdvisor), 9090 (Prometheus), 9100 (Node-exporter), 9110 (Node-exporter Service), 9116 (Snmp-Exporter) are use by this system, make sure thy are free, if needed, you can change the port use by a docker in the install file of it. Example for cAdvisor (*cadvisor_install.yml*):
~~~ shell
restart_policy: unless-stopped
#published_ports: 8090:8080    #ExternePort:InternPort
published_ports: 9000:8080     #ExternePort:InternPort
volumes: 
~~~

You will need to change too the configuration of Prometheus.

### Monitoring Installation
~~~ shell
$ ansible-playbook -i hosts install.yml
~~~

Once the install is finished, you can connect to the prometheus interface by http://IpHost:9090 to verify it receive data, then you can connect to the grafana interface http://IpHost:3000 **Login: admin / mdp: admin**. To show the graphs, change the variable of all dashboard to correspond to prometheus configuration:
![Screen 1](Screens/Capture%20du%202017-08-03%2010-48-48.png?raw=true "Screen 1")
Put the same IP address that Prometheus configuration. 
![Screen 2](Screens/Capture%20du%202017-08-03%2010-55-10.png?raw=true "Screen 2")
