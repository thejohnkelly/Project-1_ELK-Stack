## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

[Image of the Elk Stack diagram](Diagrams/johnKelly_ELKStack_project.drayio.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the YAML file may be used to install only certain pieces of it, such as Filebeat.

---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:

    - name: Uninstall apache2
      apt:
        name: apache2
        state: absent

    - name: docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

    - name: Install pip3
      apt:
        name: python3-pip
        state: present

    - name: Install Python Docker Module
      pip:
        name: docker
        state: present

    - name: Enable docker service
      systemd:
        name: docker
        enabled: yes

    - name: Download and launch a docker web container
      docker_container:
        name: dvwa
        image: cyberxsecurity/dvwa
        state: started
        restart_policy: always
        published_ports: 80:80

---
- name: Configure ELK VM with Docker
  hosts: elk
  remote_user: elkadmin
  become: TRUE
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

    # Use apt module 
    - name: Install python3-pip
      apt:
        update_cache: yes
        name: python3-pip
        state: present

    # Use the pip module
    - name: Install Docker module
      pip:
        name: docker
        state: present

    # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: 262144
        state: present
        reload: yes

    # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          -  5601:5601
          -  9200:9200
          -  5044:5044

    # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes

---
- name: install and launch filebeats
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: install the filbeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start

  - name: enable filebeat service on boot
    systemd:
      name: filebeat
      enabled: yes

---
- name: install and launch metricbeats
  hosts: webservers
  become: yes
  tasks:

  - name: download metricbeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

  - name: install the metricbeat deb
    command: dpkg -i metricbeat-7.4.0-amd64.deb

  - name: update and drop in metricbeat.yml
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

  - name: enable and configure docker module
    command: metricbeat modules enable docker

  - name: setup metricbeat
    command: metricbeat setup

  - name: redirect output to standard error
    command: service metricbeat start

  - name: enable metricbeat service on boot
    systemd:
      name: metricbeat
      enabled: yes

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build

### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available and reliable, in addition to restricting traffic the network thereby defending against potential DDoS attacks. The use of a Jump Box allows the user to directly connect to the network via a single, controlled source. A Jump Box also has the advantage of being able to configure the network machine from a central location without the need to manually configure each machine.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the log files and system metrics.

The configuration details of each machine may be found below.

| Name      | Function  | IP Address | Operating System |
|-----------|-----------|------------|------------------|
| Jump Box  | Gateway   | 10.1.0.4   | Linux            |
| Web 1     | DVMA      | 10.1.0.5   | Linux            |
| Web 2     | DVMA      | 10.1.0.6   | Linux            |
| Web 3     | DVMA      | 10.1.0.7   | Linux            |
| Rocinante | ELK Stack | 10.0.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet, and can only be accessed by Jump Box (10.1.0.4).

Only the Jump Box machine (10.1.0.4) can accept connections from the Internet. Access to this machine is only allowed from the following IP address:
- Admin IP

A summary of the access policies in place can be found in the table below.

| Name      | Publicly Accessible | Allowed IP Addresses  |
|-----------|---------------------|-----------------------|
| Jump Box  | No                  | Admin IP              |
| Web 1     | No                  | 10.1.0.4              |
| Web 2     | No                  | 10.1.0.4              |
| Web 3     | No                  | 10.1.0.4              |
| Rocinante | No                  | Admin IP and 10.1.0.4 |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because the automated operation can be performed on multiple systems at once and the playbooks can be updated for future use in the configuration of the existing machines or later additional machines.

The playbook implements the following tasks:
- Install Docker.io
- Install Python3-pip
- Install Docker module
- Allocate additional memory with sysctl
- Download and launch the Docker ELK container
- Enable Docker module on boot

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

[Image of the output of docker ps command](Diagrams/docker_ps_output.png)

### Target Machines & Beats

This ELK server is configured to monitor the following machines:
- 10.1.0.5
- 10.1.0.6
- 10.1.0.7

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- Filebeats collects log data from a specified location, which it it forwards on to a centralized location for more detailed analysis.
- Metricbeat collects metric data from systems and services and ships that data where it can be cataloged and analysed.

### Using the Playbook

In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the docker-playbook elk-playbook, filebeat-playbook, and metricbeat-playbook YAML files to the etc/ansible/roles folder.
- Update the etc/ansible/hosts file to include the private IP addresses of each machine you wish the playbook to run on, remembering to include ansible_python_interpreter=/usr/bin/python3. For each seperate instance you must create a new named category (i.e., [elk]) and add the desired IP addresses to ensure that the instances are installed on the correct machines.
- Run the playbook, and navigate to http://<ELK.VM.External.IP>:5601/app/kibana to check that the installation worked as expected. Additionally the installation can be verified by running either the "sudo docker ps" or "curl http://localhost:5601/app/kibana" in the command line within the ELK machine.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_


- To download the playbooks, use "curl -LJO https://github.com/thejohnkelly/elk-stack-project.git"
- Use "sudo mv docker-playbook.yml elk-playbook.yml filebeat-playbook.yml metricbeat-playbook.yml elkstack-main.yml > etc/ansible/roles" to move the playbook files into the roles folder.
- Use "sudo nano etc/ansible/hosts" and add the private IP addresses of you VMs. Be sure to include to include "ansible_python_interpreter=/usr/bin/python3" after each entry. If you are installing ELK and the beats to different VMs, you will need to create a new name for the instance and then add the private IP address of the machine you want to be configured.
- Run the command "sudo ansible-playbook /etc/ansible/roles/elkstack-main.yml". This file will run all of the playbooks.
- Locate the configuration files for Filebeat (etc/filebeat/filebeat.yml) and Metricbeat (etc/metricbeat/metricbeat.yml) and use "sudo nano <filepath>" to edit the files. under output.elasticsearch, add the IP address of your ELK machine as the host. Do the same under setup.kibana.
