 ## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

https://github.com/bmmurad/BCS-ClassWork/blob/main/Diagrams/RedTemAzureTopology.pdf

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the _____ file may be used to install only certain pieces of it, such as Filebeat.

https://github.com/bmmurad/BCS-ClassWork/blob/main/Ansible

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network. The load balancer ensures that work to process incoming traffic will be shared by both vulnerable web servers. Access controls will ensure that only authorized users — namely, we — will be able to connect in the first place.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file systems of the VMs on the network, as well as watch system metrics, such as CPU usage, attempted SSH logins, `sudo` escalation failures, etc.

-Filebeat: Filebeat detects changes to the filesystem. Specifically, we use it to collect Apache logs.
-Metricbeat: Metricbeat detects changes in system metrics, such as CPU usage. We use it to detect SSH login   attempts, failed `sudo` escalations, and CPU/RAM statistics.
-Packetbeat: Packetbeat collects packets that pass through the NIC, similar to Wireshark. We use it to generate   a trace of all activity that takes place on the network, in case later forensic analysis should be warranted.


The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function   | IP Address | Operating System |
|----------|------------|------------|------------------|
| Jump Box | Gateway    | 10.0.0.4   | Linux            |
| DVWA1    | Web Server | 10.0.0.5   | Linux            |
| DVWA2    | Web Server | 10.0.0.6   | Linux            |
| ELK      | ELK        | 10.1.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the jump-Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 13.72.87.109

Machines within the network can only be accessed by each other. The DVWA 1 and DVWA 2 VMs send traffic to the ELK server.
- 72.69.122.223

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 13.72.87.109         |
| DVWA1    | No                  | 10.0.0.5-254         |
| DVWA2    | No                  | 10.0.0.6-254         |
| ELK      | No                  | 10.1.0.4-254         |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- A quick and robust way to set up a standardized configuration on, potentially, multiple machines.

The playbook implements the following tasks:

- Docker and Ansible on Jumpbox
- Elk ELK VM to host 
- Run Playbook


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

https://github.com/bmmurad/BCS-ClassWork/blob/main/Diagram 

The playbook is duplicated below.

```yaml
---
# install_elk.yml
- name: Configure Elk VM with Docker
  hosts: elkservers
  remote_user: elk
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present

      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
```


### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- DVWA 1 Web Server at 10.0.0.5
- DVWA 2 Webserver  at 10.0.0.6

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat
- Packetbeat


These Beats allow us to collect the following information from each machine:
- Filebeat: Filebeat detects changes to the filesystem. Specifically, we use it to collect Apache logs.
- Metricbeat: Metricbeat detects changes in system metrics, such as CPU usage. We use it to detect SSH login   attempts, failed `sudo` escalations, and CPU/RAM statistics.
- Packetbeat: Packetbeat collects packets that pass through the NIC, similar to Wireshark. We use it to generate   a trace of all activity that takes place on the network, in case later forensic analysis should be warranted.

The playbook below installs Metricbeat on the target hosts. The playbook for installing Filebeat is not included, but looks essentially identical — simply replace `metricbeat` with `filebeat`, and it will work as expected.

```yaml
---
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
      src: /etc/ansible/files/metricbeat-config.yml
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
```


### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. We use the Jump-Box for this purpose. 

SSH into the control node and follow the steps below:
- Copy the playbooks and hosts file to etc/ansible.
- Update the hosts file to include 10.0.0.5, 10.0.0.6 and 10.1.0.4

```bash
$ cd /etc/ansible
$ cat > hosts <<EOF
[webservers]
10.0.0.5
10.0.0.6

[elk]
10.1.0.4
EOF
```

After this, the commands below run the playbook:

 ```bash
 $ cd /etc/ansible
 $ ansible-playbook install_elk.yml elk
 $ ansible-playbook install_filebeat.yml webservers
 $ ansible-playbook install_metricbeat.yml webservers
 ```

To verify success, wait five minutes to give ELK time to start up. 
Then, run: `curl http://10.1.0.4:5601`. This is the address of Kibana. If the installation succeeded, this command should print HTML to the console.
