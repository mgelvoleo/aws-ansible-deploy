# How to deploy EC2 instances using Ansible

## Step 1
Install Ansible, Pip, Boto3 module

Make IAM role and attach in controller instances

Create an inventory file and directory
```sudo mkdir /etc/ansible```

```sudo vi /etc/ansible/hosts```


Add the below two lines in the end of the file:
[localhost]
local

```
cd ~
mkdir playbooks  
cd playbooks
```

Create Ansible playbook
```sudo vi create_ec2.yml ```

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
      local_action: ec2_tag resource={{ item.id }} region={{ region }} stat>
      with_items: "{{ ec2.instances }}"
      args:
        tags:
          Name: MyTargetEc2Instance




```



now execute the ansible playbook by
```ansible-playbook create_ec2.yml```
