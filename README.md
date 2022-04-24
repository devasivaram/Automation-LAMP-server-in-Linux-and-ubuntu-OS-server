# Automation-LAMP-server-in-Linux-and-ubuntu-OS-servers

## Description

Here we will build a LAMP server for Linux and Ubuntu OS servers using Ansible, also we add sample website template to those setup. Here we use LAMP services, sample site template and set_fact service as well.

## Prerequisites

1. Need 3 Amazon LInux instances, one with ansible installed (Manager) and 1 with Linux and other with Ubuntu
~~~sh
sudo amazon-linux-extras install ansible2 -y
~~~
2. Add second instnace (Worker) private IP to hosts file (Inventory file) as well as the third instance
3. Download an sample HTML template to the EC2 instance, you can use this link for getting [sample HTML](https://www.tooplate.com/) . 
   Here we downloaded the template to /home/ec2-user/website/ location.
4. Purchase a domain name or point you domain name to server IP. You can use this to purchase new [domain_name](http://www.freenom.com/en/index.html).
5. Need to point A record for linux.example.com pointing to your Linux server’s (Worker) public IP address.
6. Need to point A record for ubuntu.example.com pointing to your ubuntu server’s (Worker) public IP address.

## Procedure

## Step 1. To configure Hosts file (Inventory file)

The orginal Hosts file for Ansible is "/etc/ansible/hosts", but am not using this file as my Hosts file, instead am creating new file with name "inventory". Here I used grouping in host file by declaring a variable in "[ variable ]".

~~~sh
vim inventory
~~~
>Added this
~~~
[linux]
server-private-ip ansible_user="ec2-user" ansible_port=22 ansible_ssh_private_key_file="private-key-file-name.pem"

[ubuntu]
server-private-ip ansible_user="ubuntu" ansible_port=22 ansible_ssh_private_key_file="private-key-file-name.pem"
~~~

>Make sure that you add the private ssh key file for your worker instance to the manager server and change its permission to "400"!

And run this command for checking SSH is working on worker server/instance:
~~~sh
ansible -i inventory amazon -m ping
~~~

## Step 2. Sample template

To get sample HTML template

~~~sh
wget https://www.tooplate.com/zip-templates/2124_vertex.zip /home/ec2-user/website/
cd /home/ec2-user/website/
unzip 2124_vertex.zip
rm -rf 2124_vertex.zip
~~~

## Step 3. Create playbook

We need to write the yml file for creating LAMP server for Linux and Ubuntu:

~~~sh
vim lamp-setup
~~~

Added:
~~~
- name: "Installing LAMP On Amazon & Ubuntu Os"
  become: true
  hosts: all       
  tasks:
    
    - name: "Linux - Setting variables"
      when: ansible_os_family == "RedHat" and ansible_distribution == "Amazon"
      set_fact:
        package: httpd
        httpd_user: apache
        httpd_group: apache
        httpd_service: httpd
            
    - name: "Ubuntu - Setting variables"
      when: ansible_os_family == "Debian" and ansible_distribution == "Ubuntu"
      set_fact:
        package: apache2
        httpd_user: www-data
        httpd_group: www-data
        httpd_service: apache2
    
    - name: "linux packages"
      when: ansible_os_family == "RedHat" and ansible_distribution == "Amazon"
      yum:
        name: "{{ package }}"
        state: present
            
    - name: "sample site"
      when: ansible_os_family == "RedHat" and ansible_distribution == "Amazon"
      copy:
        src: ./websites/
        dest: /var/www/html/
        owner: "{{httpd_user}}"
        group: "{{httpd_group}}"
        
    - name: "service restart"
      when: ansible_os_family == "RedHat" and ansible_distribution == "Amazon"
      service:
        name: "{{ linux_service}}"
        state: restarted
        enabled: true
        
    - name: "ubuntu package"
      when: ansible_os_family == "Debian" and ansible_distribution == "Ubuntu"
      apt:
        name: "{{ ubuntu_packages }}"
        state: present
        update_cache: true
        
    - name: "sample site"
      when: ansible_os_family == "Debian" and ansible_distribution == "Ubuntu"
      copy:
        src: ./website/
        dest: /var/www/html/
        owner: "{{httpd_user}}"
        group: "{{httpd_group}}"

    - name: "service restart"
      when: ansible_os_family == "Debian" and ansible_distribution == "Ubuntu"
      service:
        name: "{{ ubuntu_service}}"
        state: restarted
        enabled: true
~~~

## 3. Check syntax and execute playbook

Once the playbook is created, we need to check the syntax and execute the playbook:

~~~sh
# ansible-playbook -i hosts docker-container --syntax-check
# ansible-playbook -i hosts docker-container
~~~

Once those are successfull, we can access the site linux.example.com and ubuntu.example.com.

## Conclusion

This is how we create LAMP server for Linux and Ubuntu OS servers using Ansible. Please contact me when you encounter any difficulty error while using this terrform code. Thank you and have a great day!


### ⚙️ Connect with Me
<p align="center">
<a href="https://www.instagram.com/dev_anand__/"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/dev-anand-477898201/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a>






