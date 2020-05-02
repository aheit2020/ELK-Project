## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![TODO: Update the path with the name of your diagram](Images/diagram_filename.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the filebeat-playbook.yml file may be used to install only certain pieces of it, such as Filebeat.

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build

### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly redundant, in addition to restricting access to the network.

- Load balancers allow a singular point of control and prevents any VM from being overloaded, which in turn maintains availability. The Jumpbox operates as a highly-secure computer that all admins must connect to first and is only used for administrative tasks, making the system less likely to be exploited.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the metrics and system files/logs.

- Filebeat collects data about the system, specifically log data.
- Metricbeat collects data about machine metrics such as CPU usage and uptime.

The configuration details of each machine may be found below.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway & Ansible Host | 10.2.0.4   | Linux (Ubuntu 18.04 LTS)   |
| DVWA-VM1 |Hosts DVWA| 10.2.0.5   | Linux (Ubuntu 18.04 LTS)   |
| DVWA-VM2 |Hosts DVWA| 10.2.0.6   | Linux (Ubuntu 18.04 LTS)   |
| DVWA-VM3 |Redundancy| 10.2.0.7   | Linux (Ubuntu 18.04 LTS)   |
| ELK      |Hosts ELK (public) | 10.2.0.10  | Linux (Ubuntu 18.04 LTS)   |

### Access Policies

The machines on the internal network are not exposed to the public Internet.

Only the load balancer machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 99.239.0.0

Machines within the network can only be accessed by the Jumpbox VM (10.2.0.4) through SSH.

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 99.239.0.0           |
| DVWA-VM1 | No                  | 10.2.0.4                     |
| DVWA-VM2 | No                  | 10.2.0.4                     |
| DVWA-VM3 | No                  | 10.2.0.4                     |
| ELK      | No                  | 10.2.0.4                     |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because this ensures that provisioning scripts run identically which eliminates variability between configurations.

The playbook implements the following tasks:
- First it configures the hosts and remote user
- Second, it installs the docker.io with apt_get
- Third, it installs python
- Fourth, it installs the docker module
- Fifth, it increases virtual memory with sysctl -w vm.max_map_count=262144
- Lastly, it configures the elk container via name, state and published ports which allows access via Kibana

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- DVWA-VM1 10.2.0.5, DVWA-VM2 10.2.0.6, DVWA-VM3 10.2.0.7

We have installed the following Beats on these machines:
- Filebeat, Metricbeat

These Beats allow us to collect the following information from each machine:
- Filebeat collects log data about the system by monitoring changes to specified log files and locations. It then forwards this information to Elasticsearch.
- Metricbeat collects data about machine metrics from the operating system from services such as CPU usage or Uptime, and then sends the statistical information to a specified location. 

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned:

SSH into the control node and follow the steps below:
- Copy the configuration file to /etc/ansible/configfile.
- Update the configuration file to include the IP of your Elk VM
- Run the playbook, and navigate to Kibana via http://[your.ELK.VM.IP]:5601 to check that the installation worked as expected. In my case, I would navigate to http://20.42.89.172:5601 . 

## Filebeat Steps
- Create a directory called /configfile under /etc/ansible
- Copy configuration file to /etc/ansible/configfile
- Update the configuration file on lines #1106 and #1806 to include your ELK VM IP 
- Ensure /etc/ansible/hosts is properly configured with the IP of the machines you want filebeat to monitor
- Download playbook to /etc/ansible/roles, or alternatively copy the following playbook to create your own in /etc/ansible/roles:
---
- name: Install and Launch Filebeat
  hosts: webservers
  remote_user: [your_username_here]
  become: yes
  tasks:
  
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb
    
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb
 
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/configfile/filebeat-configuration.yml
      dest: /etc/filebeat/filebeat.yml
  
  - name: Enable and Configure System Module
    command: filebeat modules enable system
 
  - name: Setup filebeat
    command: filebeat setup
    
  - name: Start filebeat service
    command: service filebeat start
    
- Ensure the playbook is properly formatted with correct spacing, has the full path to filebeat-playbook.yml in the playbook, and that your own username is filled out next to remote_user: 
- Use command in /etc/ansible/roles: ansible-playbook filebeat-playbook.yml to run the playbook
- On Kibana navigate to the Filebeat installation page on the ELK server GUI
- Scroll down to the bottom of the page and click Check Data, if the steps worked you will see an affirmation of its success in a green message
- Click Verify Incoming Data 
- Done!

## Metricbeat Steps
- Copy configuration file to /etc/ansible/configfile
- Update the configuration file on lines #62 and #96 and replace the default IP with the IP of your ELK VM
- Ensure /etc/ansible/hosts is properly configured with the IP of the machines you want metricbeat to monitor
- Download playbook to /etc/ansible/roles, or alternatively copy the following playbook to create your own:
---
- name: Install and Launch Metricbeat
  hosts: webservers
  become: true
  tasks:
  
  - name: Download metricbeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb
    
  - name: Install metricbeat .deb
    command: dpkg -i metricbeat-7.4.0-amd64.deb
 
  - name: Drop in metricbeat.yml
    copy:
      src: /etc/ansible/configfile/metricbeat-configuration.yml
      dest: /etc/metricbeat/metricbeat.yml
  
  - name: Enable and Configure System Module
    command: metricbeat modules enable system
 
  - name: Setup metricbeat
    command: metricbeat setup
    
  - name: Start metricbeat service
    command: service metricbeat start
    
- Ensure the playbook is properly formatted with correct spacing and has the full path to metricbeat-playbook.yml
- Use command in /etc/ansible/roles: ansible-playbook metricbeat-playbook.yml to run the playbook
- On Kibana navigate to the metricbeat installation page on the ELK server GUI
- Click Add Metric Data, Click Docker Metrics, Click the DEB tab under Getting Started, Scroll down to the bottom of the page and click Check Data
- If the process is a success you will be sent to a page with a list of your VMs and their metrics
- Done!
