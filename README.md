# ansible-playbooks

### Description

This is a list of my playbook.
If you came here because you used my stack :
* [AWS/README.md] (https://github.com/s0nny78/AWS)

"host" is the playbook used for Ansible Server to configure its own host file 
```sh
---
#describe all the instances with the tag ansible at true and select only the privateIP in text form and not JSON
- name: make a command
  command: 'aws ec2 describe-instances --region "us-east-1" --filters "Name=tag:ansible,Values=true" --query "Reservations[*].Instances[*].[PrivateIpAddress]" --output text'
  register: aws

#format the content and make multiple line, each one for a different ip and put it in a temp file
- name: copy content
  shell: printf '{{aws.stdout}}' > /tmp/ip

#copy the content of the prev file to put it in /etc/ansible/host and add the ansible_user at the end of each line
- name: copy cat
  shell : sed -e "s/$/ ansible_user=ubuntu/" /tmp/ip > /etc/ansible/hosts

#finally add a first line for the name of the hosts's group
- name: copy ca
  shell : sed -i '1s/^/[cluj]\n/' /etc/ansible/hosts
```

"tomcat" is the one deployed to others hosts to install tomcat8, and put clojure collector
````sh
---
#Add the repo of H.-Dirk Schmitt
#http://askubuntu.com/users/20015/h-dirk-schmitt
  - name: repo for tomcat8
    apt_repository: repo='ppa:dirk-computer42/c42-edge-server'
    sudo: yes

#update
  - name: apt-get update
    apt: update_cache=yes
    sudo: yes

#install jdk
  - name: ensure jdk is there
    sudo : yes
    apt:
      name: default-jdk
      state : present

#install tomcat8
  - name: ensure tomcat is present
    sudo: yes
    apt:
      name: tomcat8
      state: present

#be sure tomcat8 is running
  - name: ensure tomcat is running
    sudo: yes
    service:
      name: tomcat8
      state: started

#dl clojure and put it on the webapp folder to run it
  - name: dl war file
    get_url: url=http://d2io1hx8u877l0.cloudfront.net/2-collectors/clojure-collector/clojure-collector-1.1.0-standalone.war dest=/var/lib/tomcat8/webapps/
    sudo: yes
```

### Thanks to:
Special thanks to : 
* Mohammad working at BoomerangCommerce (https://github.com/ironmanMA/ansible-tomcat-8)
* H.-Dirk Schmitt working at Computer42 (http://www.computer42.org/)
