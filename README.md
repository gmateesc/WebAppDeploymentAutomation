# WebAppDeploymentAutomation


## Table of Contents

- [Overview](#p0)

- [Simple usage of the Ansible code](#p1)
  - [Get the code from Git](#p11)
  - [Run ansible-playbook](#p12)
  - [Start the service](#p13)

- [The inventory file](#p1a)

- [Description of the playbook](#p2)

- [Configuration of the web app](#p3)
  - [File group_vars/all](#p31)
  - [File roles/app/defaults/main.yml](#p32)




<a name="p0" id="p0"></a>
## Overview

This page discusses a solution to automating the deployment of a Flask web application.




<a name="p1" id="p1"></a>
## Simple usage of the Ansible code

The simplest usage of the ansible code is to clone this repository 
and then run the playbook deploy_web_service.yml.



<a name="p11" id="p11"></a>
## Get the code from Git

```shell
$ git clone https://github.com/gmateesc/WebAppDeploymentAutomation

$ cd WebAppDeploymentAutomation
```




<a name="p12" id="p12"></a>
## Run ansible-playbook


Then set GIT_USER and GIT_PASSWORD for accessing the Git repository for the 
Web Service and run the playbook deploy_web_service.yml 

```shell
  $ ansible-playbook -e gituser=$GIT_USER -e gitpassword=$GIT_PASSWORD deploy_web_service.yml 2>&1 \
                     | tee deploy_web_service.log
```


This will deploy the web app to the local host. 


An example output from running the playbook is shown [here](https://github.com/gmateesc/WebAppDeploymentAutomation/blob/master/doc/deploy_web_service.log).


To deploy to other hosts, edit the [inventory file](#p1a) and add the desired hosts.



<a name="p13" id="p13"></a>
## Start the service


Run the start/stop script created by Ansible. For the above run the script is:

```script
  $ egrep "start\|stop" deploy_web_service.log 
  /tmp/Scheduler_app/app/WebService/schedule_service.sh {start|stop|status}
```


So one can start the service with

```script
  $ /tmp/Scheduler_app/app/WebService/schedule_service.sh start
```






<a name="p1a" id="p1a"></a>
## The inventory file


The default inventory file includes the local host, and shows how to add remote hosts, 
and, depending on the site settings, use a different remote user or use a jump-host 
to access the remote host:

```script
$ more inventory#
# If the default version of python is not 2.6 or 2.7, 
# set ansible_python_interpreter to point to the correct 
# version of Python, e.g., 
#
# localhost ansible_connection=local ansible_python_interpreter=/usr/bin/python
#

#
# Add remote machines here using one of these formats
#
#  HOST_NAME_OR_IP_ADDR
#  HOST_NAME_OR_IP_ADDR ansible_user=USER
#  HOST_NAME_OR_IP_ADDR ansible_user=USER  ssh_args="-o ProxyCommand ssh -W %h:%p JUMP_HOST"
[webservers]
localhost ansible_connection=local

[all:children]
webservers
```





<a name="p2" id="p2"></a>
## Description of the playbook


The playbook invokes several roles:

```yaml
$ more deploy_web_service.yml
#
# Run with
#
#  ansible-playbook  -e gituser=$GIT_USER -e gitpassword=$GIT_PASSWORD deploy_web_service.yml
#
# If http_proxy or https_proxy is required, then define them in group_vars/all
#
---
- hosts: all
  gather_facts: no
  become: no

  roles:
  - { role: "http_proxy" }
  - { role: "python" }
  - { role: "pip" }
  - { role: "git" }
  - { role: "venv" }
  - { role: "app" }
```

where

  - role "http_proxy" enables deploying the web app to hosts that must use an http or http proxy to access Git and python repositories;

  - the roles "python" and "pip" check the presense of  the required versions of python and pip

  - the role: "git" clones the web app from Git

  - the role: "venv" creates a python virtual environment containing all the modules required by the web app;
  - the role: "app" configures the web app as shown below.






<a name="p3" id="p3"></a>
## Configuration of the web app


The app role includes the file roles/app/defaults/main.yml that defines application configuration. 
Additionally, group-level variables are defined in group_vars/all





<a name="p31" id="p31"></a>
## File group_vars/all



The file *group_vars/all* defines application configuration, including: 

- the location where to deploy the web app: variable project_dir

- the minimum required Python version;

- the directory where to store the JSON documents POSTed by client: varible service_document_root;

- the proxy to use for accesing http and https resources: variables http_proxy and https_proxy;


The full file *group_vars/all* is:


```yaml

$ more group_vars/all 
---
#
# Settings defined here:
#
# 1. Override role defaults, e.g., ../roles/*/defaults
#
# 2. Are overriden by role vars, e.g., ../roles/*/vars
#

project_dir: /tmp/Scheduler_app

python_version: 2.6
python_virtualenv_path: "{{ project_dir}}/venv"

app_dir:  "{{ project_dir}}/app"


#
# If http/https proxy is needed, set the environment 
# variable http_proxy and https_proxy or set the 
# variables below. Setting the variables here is 
# needed when different host groups use different proxies.
#
#http_proxy: YOUR_HTTP_PROXY
#https_proxy: YOUR_HTTPS_PROXY
```






<a name="p32" id="p32"></a>
## File roles/app/defaults/main.yml


The file roles/app/defaults/main.yml defines application configuration, including: 

- the location of the SSL certificates used by the web app: variable ssl_cert_root;

- the directory where to store the JSON documents POSTed by client: varible service_document_root;


The file roles/app/defaults/main.yml is shown here:

```yaml

$ more roles/app/defaults/main.yml 
---

#
# IP address and port to which the web service listens
#
webserver_host: "0.0.0.0"
webserver_port: 8888

#
# Service name and name of program providing the service
#
service_name: WebService
service_code: schedule_service.py
service_script: "{{ service_code | replace('.py', '.sh') }}"


#
# Directory where to deploy the SSL keys and cert
#
ssl_cert_root: "{{ app_dir }}/ssl"

#
# Directory where to store the JSON documents POSTed by client
#
service_document_root: "{{ app_dir}}/status"

#
# Log directory of the web service
#
service_log_dir: "{{ app_dir}}/log"





