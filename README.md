# AnsibleMonitoring
## Installation (on Monitoring Server)
Ansible Installation
http://docs.ansible.com/ansible/latest/intro_installation.html#installing-the-control-machine

~~~ shell
$ sudo apt-get install git
$ git clone https://github.com/shakapark/AnsibleMonitoring/
$ cd AnsibleMonitoring
~~~

Configuration:

Change the file "hosts" : for the distant machines, change ip address, ssh port if needed, user name of the remote machine and password or private key for ssh connection.

In the file install.yml, you can modify your installation by comment or supress the line you don't need (prerequisites.yml, prometheus_install.yml and grafana_install.yml must be install).

In 'conf' folder, you must change the prometheus configuration ("prometheus.yml") to monitor remote machine (change 'IpNode1' by an ip address that the monitoring server can contact, add more if you want).

You can change too the grafana configuration ("grafana.ini")

Monitoring Installation
~~~ shell
$ ansible-playbook -i hosts install.yml
~~~
