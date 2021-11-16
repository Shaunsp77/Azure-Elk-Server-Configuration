# Azure Elk Deployment
Azure cloud set up for webservers, load balancer and ELK sever for Kibana reporting
Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

__Project 1: RedTeam_Diagram__

![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Diagrams/RedTeamDiagram.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the yml and config file may be used to install only certain pieces of it, such as Filebeat.

<p>
<details>
  <summary>Ansible ELK Server Installation Playbook</summary>
  
<pre><code>---
- name: Configure ELK
  hosts: ELK
  remote_user: sysadmin
  become: True
  tasks:
  - name: use more memory
    sysctl:
      name: vm.max_map_count
      value: '262144'
      state: present
      reload: yes

  - name: docker.io
    apt:
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
     force_apt_get: yes
     name: python3-pip
     state: present

  - name: install python module
    pip:
      name: docker
      state: present

  - name: elk container
    docker_container:
      name: elk
      image: sebp/elk:761
      state: started
      restart_policy: always
      published_ports:
        - 5601:5601
        - 9200:9200
        - 5044:5044
        
  - name: Enable Service docker on boot
    systemd:
      name: docker
      enabled: yes</code></pre>

  </details>
  </p>

<p>
<details>
  <summary>DVWA Install File</summary>
  
  <pre><code>
  ---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:
  - name: docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: Install Docker python module
    pip:
      name: docker
      state: present

  - name: download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      published_ports: 80:80

  - name: Enable docker service
    systemd:
      name: docker
      enabled: yes</code></pre>
  </details>
  </p>

<p>
<details>
  <summary>Filebeats Playbook</summary>
  
  <pre><code>---
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

    # Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

    # Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

    # Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system

    # Use command module
  - name: Setup filebeat
    command: filebeat setup

    # Use command module
  - name: Start filebeat service
    command: service filebeat start

    # Use systemd module
  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes</code></pre>
  </details>
  </p>
  
<p>
<details>
  <summary>Metricbeats Playbook</summary>
  
  <pre><code>---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: service metricbeat start

    # Use systemd module
  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes</code></pre>
  </details>
  </p>

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting traffic to the network.

**What aspect of security do load balancers protect? What is the advantage of a Jumpbox?**

_Load balancers are used to protect the system from DDoS attacks by shifting attack traffic. It is also used to offset the load of traffic from one server to the next to avoid downtime. The advantage of a jump box is to give access to the user from a single node that can be secured and monitored.
Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the data and system logs._

**What does Filebeat watch for?** 

_Filebeat monitors the log files, collects log events, and forwards them either to the Elasticsearch or Logstash for indexing_

**What does Metricbeat record?** 

_Metricbeat takes the metrics that it collects and sends them to the output specified such as Elasticsearch or Docker Metrics_

**The configuration details of each machine may be found below.**

| Name of VM | Function | IP: Private | IP: Public | OS  |
| :---       | :---:     | :---:        | :---:       | :---: |
| RedTeamVM  | Gateway  | 10.2.0.5    | 20.115.73.196 | Linux |
| Web 1      | Web Server | 10.2.0.6  | Static    | Linux |
| Web 2      | Web Server | 10.2.0.7  | Static    | Linux |
| load Balancer | Load Balancer | Static | Static | Linux |
| ELK        | Elk Server | 10.0.0.4 | 13.67.141.182 | Linux |
| Workstation | Access Control | External | External | Mac |


### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Elk Server machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
Whitelisted IP addresses
- Any Public IP through TCP 5601

Machines within the network can only be accessed by the Jumpbox Provisioner (RedTeamVM)

**Which machine did you allow to access your ELK VM? What was its IP address?**
- RedTeamVM: 10.2.0.5 via SSH port 22
- Workstation Public IP via TCP 5601

A summary of the access policies in place can be found in the table below.


| Name | Publicly Accessible | Allowed IP Address |
| :---  | :---:                 | :---                |
| RedTeam VM | No | Workstation Public IP on SSH 22 |
| Web 1 | No | 10.2.0.5 via SSH 22 |
| Web 2 | No | 10.2.0.6 via SSH 22 |
| ELK Server | No | Workstation Public IP on TCP 5601 |
| Load Balancer | No | Workstation Public IP on HTTP 80 |


### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because Ansible allows you to quickly and easily deploy applications. 

**What is the main advantage of automating configuration with Ansible?**

- You do not need to write custom scripts to automate your system or deployments. Ansible just requires you to create self explained tasks via playbook yml and then easily deploy to all your machines designated by the host file. Ansible also figures out how to get each system to your desired state using these playbook instructions. 

The playbook implements the following tasks:

- **Machine groups and Remote User Specifications**
<pre><code>- name: Config ELK VM with Docker
  	  hosts: elk
  	  remote_user: sysadmin
  	  become: true
      tasks:</code></pre>

- **Increase System Memory**
 <pre><code>- name: Use more memory
    sysctl:
    name: vm.max_map_count
    value: '262144'
    state: present
    reload: yes</code></pre>

- **Install the following services**
  - Docker.io
  - Python3-pip
  - Docker elk container

- **Launching and Exposing the container with these published ports:**
  - _5601:5601_
  - _9200:9200_
  - _5044:5044_


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

_Elk Server_

![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Images/Elk%20Server%20Docker%20PS.png)

_Web-1_

![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Images/Web%201%20Docker%20PS.png)

_Web-2_

![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Images/Web%202%20Docker%20PS.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
 
 **List the IP addresses of the machines you are monitoring**

- Web-1: 10.2.0.6
- Web 2: 10.2.0.7 

We have installed the following Beats on these machines:
- Elk Server 
- Web-1 
- Web-2

Specify which Beats you successfully installed
- FileBeats
- Metricbeats

These Beats allow us to collect the following information from each machine:
- Filebeats allow us to collect log events.
  - _Ex: Files generated by Apache or MS Azure Tools_

- Metricbeats allow us to collect metrics and statistics 
  - _Ex: CPU Usage_ 


### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the [ELK Server Config](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Ansible/ELK%20Configuration%20.rtf)  file to `/etc/ansinble`.
![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Images/Ansible%20File.png)

- Update the hosts file to include Address of your webservers and ELK Private IP Address
![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Images/hosts%20file.png)

- Update the filebeats-config.yml and the metricbeats-config.yml to designate the address of the ELK Server
  - output.elasticsearch: 
  - output.kibana: 
  
- **Filebeats-config**
![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Images/filebeats_Output.Elasticsearch.png)

![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Images/filebeats_setup.kibana.png)
    
- **Metricbeats-config**
![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Images/metricbeat_output.elasticsearch.png)

![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Images/metricbeats_setup.kabana.png)

  
- Run the filebeats playbook, and navigate to Kibana, under "add data logs/System_logs" to check that the installation worked as expected.
![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Images/Filebeats%20data.png) 

- Run the metricbeats playbook, and navigate to Kibana under "add metric data/Docker" to check that the installation worked as expected.
![Alt text here](https://github.com/Shaunsp77/Azure-Elk-Server-Configuration/blob/main/Images/Metricbeats%20Data.png)



**Which file is the playbook?** 
- filebeat-playbook.yml
- metricbeat-playbook.yml

**Where do you copy it?**
- `/etc/ansible`

**Which file do you update to make Ansible run the playbook on a specific machine?** 
- `/etc/ansible/hosts` and add the private IP addresses under the [WEBSERVERS] section or [ELK Section] of the host

**How do I specify which machine to install the ELK server on versus which to install Filebeat on?**
- By specifying groups in the `/etc/ansible/hosts` file, labled [WEBSERVERS] for filebeat, and [ELK] for Elk installation.

**Which URL do you navigate to in order to check that the ELK server is running?**
- __http:[Elk.VM.IP]:5601/app/kibana__ where the [Elk.VM.IP] is the **Public address of the server**

<p>
<details>
  <summary>Terminal Commands Needed:</summary>

  | Command | Purpose |
  | :---    | :---    | 
  | SSH -i [name of keygen file] (user@ipaddress) | Remote into your Jump Desktop |
  | SSH-Keygen | generate public and private key |
  | sudo apt install docker.io | Install Docker |
  | sudo service docker start | Start the Docker service |
  | sudo docker container list -a | show all containers services |
  | sudo docker start <container_name> | Start the container specified |
  | sudo docker attach <container_name> | Remote into the specified container |
  | ansible -m ping all | check the connection of ansible containers |
  | ansible-playbook <playbook.yml_file> | Run a playbook.yml file |
  | nano <name_of_playbook.yml> | Create an Ansible playbook |
  | sudo docker pull cyberxsecurity/ansible bash | run and create a docker image |
  | sudo docker ps -a | List all active/inactive containers |
  

 </detals>
 </p>
