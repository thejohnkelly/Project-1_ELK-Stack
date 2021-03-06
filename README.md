## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

[Image of the Elk Stack diagram](Diagrams/johnKelly_ELKStack_project.drayio.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select  YAML files may be used to install only certain pieces of it, such as Filebeat.

[Link to YAML files to run in Ansible](Ansible/)


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available and reliable, in addition to restricting traffic the network thereby defending against potential DDoS attacks. The use of a Jump Box allows the user to directly connect to the network via a single, controlled source. A Jump Box also has the advantage of being able to configure the network machine from a central location without the need to manually configure each machine.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the log files and system metrics. Filebeats collects log data from each of the DVWAs, which it it forwards on to a centralized location for more detailed analysis. Metricbeat collects metric data from the DVWAs and ships that data where it can be cataloged and analysed.


The configuration details of each machine may be found below.

| Name      | Function  | IP Address | Operating System |
|-----------|-----------|------------|------------------|
| Jump Box  | Gateway   | 10.1.0.4   | Linux            |
| Web 1     | DVWA      | 10.1.0.5   | Linux            |
| Web 2     | DVWA      | 10.1.0.6   | Linux            |
| Web 3     | DVWA      | 10.1.0.7   | Linux            |
| Rocinante | ELK Stack | 10.0.0.4   | Linux            |

### Access Policies

The only machines that can accept connections from the internet are the Jump Box machine (10.1.0.4) via port 80 and the ELK server (10.0.0.4) via port 5601. The machines on the internal network are not exposed to the public Internet, and can only be accessed by the Jump Box machine.

Access to both the Jump Box machine and the ELK server are only allowed from the following IP address:
- Admin IP

A summary of the access policies in place can be found in the table below.

| Name      | Publicly Accessible | Allowed IP Addresses  |
|-----------|---------------------|-----------------------|
| Jump Box  | Yes                 | Admin IP              |
| Web 1     | No                  | 10.1.0.4              |
| Web 2     | No                  | 10.1.0.4              |
| Web 3     | No                  | 10.1.0.4              |
| Rocinante | Yes                 | Admin IP and 10.1.0.4 |

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

### Using the Playbooks

In order to use the playbooks, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Download the playbooks from your command line using "curl -LJO https://github.com/thejohnkelly/elk-stack-project.git"
- Use the command "mkdir etc/ansible/roles" to create the 'roles' directory that will store the playbook files.
- Navigate to /usr/elk-stack-project and use the command "sudo cp ./Ansible/* etc/ansible/roles" to copy the playbook files into the new 'roles' directory. The 'roles' directory will now contain all of the YAML files needed to fully install and launch the ELK Stack, Docker on all the DVWAs, and both the beats. 
- Use "sudo nano etc/ansible/hosts" and add the private IP addresses of you VMs under the heading '[webservers]', and add the private IP of your ELK server under the heading '[elk]' to ensure the instances are installed and configured on the correct machines.Be sure to include ansible_python_interpreter=/usr/bin/python3 after each IP address
- Run the command "sudo ansible-playbook /etc/ansible/roles/elkstack-main.yml". This file will import and run all of the playbooks.
- Locate the configuration files for Filebeat (etc/filebeat/filebeat.yml) and Metricbeat (etc/metricbeat/metricbeat.yml) and use "sudo nano [your_file_path]" to edit each of the files. Add the IP address of your ELK machine as the host under output.elasticsearch in both configuration files. Do the same under setup.kibana.
- Navigate to http://<ELK.VM.External.IP>:5601/app/kibana to check that the installation worked as expected. Alternately the installation can be verified by running either the "sudo docker ps" or "curl http://localhost:5601/app/kibana" in the command line within the ELK machine.
