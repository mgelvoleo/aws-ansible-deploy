# How to deploy EC2 instances using Ansible

## Step 1 Install ansible, PIP, Boto module

* Install Ansible, Pip, and Boto3 module on your system.

## Step 2 Make IAM role and attach in controller instances

* Create an IAM role and attach it to the controller instances you will be using.
  ![image](https://github.com/mgelvoleo/aws-ansible-deploy/assets/21300768/3c064f55-75f6-46b4-ac3a-1fbd638f8c06)


### Step 3 Create an inventory file and directory

* Open a terminal and execute the following commands:

```sudo mkdir /etc/ansible```

```sudo vi /etc/ansible/hosts```

* This will create the directory /etc/ansible and open the hosts file using the vi text editor.

### Step 4: Add the Following Two Lines at the End of the hosts File
```
[localhost]
local

```

* Add the above lines to the end of the hosts file and save the changes.

### Step 5 create playbooks folder and go inside

* Execute the following commands:

```
cd ~
mkdir playbooks  
cd playbooks
```

* This will create a folder named playbooks in your home directory and navigate inside it.

### Step 6: Create the Ansible Playbook named create_ec2.yml

* Execute the following command to create the playbook:
```sudo vi create_ec2.yml ```

* Copy and paste the following content into the create_ec2.yml file:yaml

```
---
- name: provisioning EC2 instances using Ansible
  hosts: localhost
  connection: local
  gather_facts: False
  tags: provisioning

  vars:
    keypair: aws_keypair
    instance_type: t2.micro
    image: ami-0df7a207adb9748c7
    wait: yes
    group: webserver
    count: 1
    region: ap-southeast-1
    security_group: my-jenkins-security-grp

  tasks:
    - name: Task # 1 - Create my security group
      local_action:
        module: ec2_group
        name: "{{ security_group }}"
        description: Security Group for webserver Servers
        region: "{{ region }}"
        rules:
          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 8080
            to_port: 8080
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0
        rules_egress:
          - proto: all
            cidr_ip: 0.0.0.0/0
      register: basic_firewall

    - name: Task # 2 Launch the new EC2 Instance
      local_action: ec2
                    group={{ security_group }}
                    instance_type={{ instance_type}}
                    image={{ image }}
                    wait=true
                    region={{ region }}
                    keypair={{ keypair }}
                    count={{ count }}
      register: ec2

    - name: Task # 3 Add Tagging to EC2 instance
      local_action: ec2_tag resource={{ item.id }} region={{ region }} state=present
      with_items: "{{ ec2.instances }}"
      args:
        tags:
          Name: MyTargetEc2Instance




```

* Save the file after pasting the content.





### Step 7 now execute the ansible playbook by
```ansible-playbook create_ec2.yml```
