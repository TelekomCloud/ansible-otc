Ansible for Open Telekom Cloud
==============================

Intro
=====

Deutsche Telekom offers since March 2016 an IaaS Service named
Open Telekom Cloud (OTC). The service includes
* Virtual Private Cloud (VPC)
* Elastic Cloud Server (ECS)
* Elastic Load Balancer (ELB)
* Elastic Volume Service (EVS)
and other useful things. The portfolio will rapidly developed.


Content
=======
Here are some roles to demonstrate how to interact with OTC-API.
ECS-API is origin developed by Huawei and described here:
https://docs.otc.t-systems.com/api/ecs/index.html

Roles
=====
|role         | description|
|-------------|------------|
|ecs                    | list virtual machines|
|ecs_create             | create and start virtual machine|
|ecs_delete             | delete a specific virtual machine|
|ecs_show               | information about a specific virtual machine|
|elb                    | list elastic loadbalancers|
|elb_create             | create elastic loadbalancer|
|elb_delete             | delete elastic loadbalancer|
|elb_show               | show elastic loadbalancer|
|elb_certificate        | show elastic loadbalancer certificates|
|elb_certificate_create | create elastic loadbalancer certificate|
|elb_certificate_delete | delete elastic loadbalancer certificate|
|elb_healthcheck_create | create elastic loadbalancer healthcheck|
|elb_healthcheck_delete | delete elastic loadbalancer healthcheck|
|elb_healthcheck_show   | show elastic loadbalancer healthcheck|
|elb_listener           | list listener for elastic loadbalancer|
|elb_listener_create    | create listener for elastic loadbalancer|
|elb_listener_delete    | delete listener from elastic loadbalancer|
|elb_backends           | list backends for elastic loadbalancer|
|elb_backends_create    | create backends for elastic loadbalancer|
|elb_backends_delete    | delete backends for elastic loadbalancer|
|flavors                | show flavors|
|floatingip             | show floating ip-addresses|
|images                 | show images|
|job                    | show job status|
|keypairs               | show keypairs|
|secgroups              | show security groups|
|subnet                 | show subnet|
|token                  | get auth token|
|vpc                    | show vpc|

Requirements
============
* Ansible >=2.0.1.0

  *Ubuntu 12.04/14.04/16.04:*
  
  ```
     apt-get install software-properties-common
     apt-add-repository ppa:ansible/ansible
     apt-get update
     apt-cache policy ansible
     # should have version >2.1.0
     apt-get install ansible
  ```
  
  *OpenSuSE 13.2:*
  
  ```
     zypper ar http://download.opensuse.org/repositories/systemsmanagement/openSUSE_13.2/systemsmanagement.repo
     zypper up
     zypper install ansible
  ```    
      
(should work on all other *nix systems)

* :exclamation: credentials on OTC (username, projectid, generated API key)

Files
=====
| filename       | description|
|----------------|------------|
|ajob            | shell script to fetch job status from OTC|
|secrets.yml     | var file for OTC credentials and endpoints (ansible-vault)|
|ecs_secrets.yml  | var file for virtual machine conf (ansible-vault)|
|elb_secrets.yml | var file for elastic loadbalancer conf (ansible-vault)|
|vaultpass.txt   | password file for ansible-vault. The default password is: linux :-)|
|hosts           | host file for ansible (we use only localhost)|

Examples
========

```
    cp secrets.yml  _secrets.yml 
    cp ecs_secrets.yml  _ecs_secrets.yml 
    cp elb_secrets.yml _elb_secrets.yml
```
  
:exclamation: **adjust your own data in this file before you using the examples:**

list virtual machines

    ansible-playbook -i hosts ecs.yml --vault-password-file vaultpass.txt

create and start virtual machine (defined in ecs_secrets.yml)

    ansible-playbook -i hosts ecs_create.yml --vault-password-file vaultpass.txt

create and start virtual machine with file injection 
(inject up to 5 max 1k base64 encoded files)

    ansible-playbook -i hosts -e "ecs_fileinject_1=/etc/hosts ecs_fileinject_data_1=$(base64 -w 0 hosts.txt) ecs_fileinject_2=/root/README.md2 ecs_fileinject_data_2=$(base64 -w 0 hallo.txt)" ecs_create.yml --vault-password-file vaultpass.txt

create and start virtual machine with injection user_data
(inject max 32k base64 encoded user-data files)

    ansible-playbook -i hosts -e "ecs_user_data=$(base64 -w 0 user-data.txt)" ecs_create.yml --vault-password-file vaultpass.txt

(!) You can define ecs_fileinject_1, ecs_fileinject_data_1 and ecs_user_data also in _ecs_secrets.yml. Files must be base64 encoded.

delete virtual machine (only the machine)

    ansible-playbook -e "ecs_id=51b6558a-7a6d-49f4-94e5-f4ec94314746 ecs_name=test05-ansible" -i hosts ecs_delete.yml --vault-password-file vaultpass.txt

delete virtual machine (delete also floating ip and attached volumes)

    ansible-playbook -e "ecs_id=f6b7536e-b954-4d73-940f-248de71ce58b ecs_name=test06-ansible delete_publicip=1 delete_volume=1" -i hosts ecs_delete.yml --vault-password-file vaultpass.txt


show information about a single virtual machines

    ansible-playbook -e "ecs_id=f6b7536e-b954-4d73-940f-248de71ce58b ecs_name=test06-ansible" -i hosts ecs_info.yml --vault-password-file vaultpass.txt
    
list elastic loadbalancers

    ansible-playbook -i hosts elb.yml --vault-password-file vaultpass.txt

create elastic loadbalancer

    ansible-playbook -i hosts elb_create.yml --vault-password-file vaultpass.txt

delete elastic loadbalancer

    ansible-playbook -i hosts -e "elb_id=43848329789145988d1e0bf25edb5ea8" elb_delete.yml --vault-password-file vaultpass.txt

show elastic loadbalancer

    ansible-playbook -i hosts -e "elb_id=43848329789145988d1e0bf25edb5ea8" elb_show.yml --vault-password-file vaultpass.txt

list elastic loadbalancer certificates

    ansible-playbook -i hosts  elb_certificate.yml --vault-password-file vaultpass.txt

create elastic loadbalancer certificate

    ansible-playbook -i hosts -e "elb_certificate_name=ansible-cert elb_certificate_key_file=cert.key elb_certificate_certificate_file=cert.crt"  elb_certificate_create.yml --vault-password-file vaultpass.txt

delete elastic loadbalancer certificates

    ansible-playbook -i hosts -e "elb_certificate_id=43848329789145988d1e0bf25edb5ea8"  elb_certificate_delete.yml --vault-password-file vaultpass.txt

create elastic loadbalancer healthcheck

    ansible-playbook -i hosts -e "elb_listener_id=1595f0e7b6984395ab2832a22cd246f2" elb_healthcheck_create.yml --vault-password-file vaultpass.txt

delete elastic loadbalancer healthcheck

    ansible-playbook -i hosts -e "elb_healthcheck_id=e12454b93f304b759be699cb0270648c" elb_healthcheck_delete.yml --vault-password-file vaultpass.txt

show elastic loadbalancer healthcheck

    ansible-playbook -i hosts -e "elb_healthcheck_id=e12454b93f304b759be699cb0270648c" elb_healthcheck_show.yml --vault-password-file vaultpass.txt

list listener for elastic loadbalancer

    ansible-playbook -i hosts -e "elb_id=e12454b93f304b759be699cb0270648c" elb_listener.yml --vault-password-file vaultpass.txt

create listener for elastic loadbalancer

    ansible-playbook -i hosts -e "elb_id=e12454b93f304b759be699cb0270648c" elb_listener_create.yml --vault-password-file vaultpass.txt

delete listener for elastic loadbalancer

    ansible-playbook -i hosts -e "elb_listener_id=e12454b93f304b759be699cb0270648c" elb_listener_delete.yml --vault-password-file vaultpass.txt

list backends for elastic loadbalancer

    ansible-playbook -i hosts -e "elb_listener_id=e12454b93f304b759be699cb0270648c elb_backends.yml --vault-password-file vaultpass.txt

create backends for elastic loadbalancer

    ansible-playbook -i hosts -e "elb_listener_id=e12454b93f304b759be699cb0270648c ecs_id=f6b7536e-b954-4d73-940f-248de71ce58b ecs_address=192.168.0.112" elb_backends_create.yml --vault-password-file vaultpass.txt

delete backends for elastic loadbalancer

    ansible-playbook -i hosts -e "elb_listener_id=e12454b93f304b759be699cb0270648c elb_backends_id=f6b7536e-b954-4d73-940f-248de71ce58b" elb_backends_delete.yml --vault-password-file vaultpass.txt

show flavors

    ansible-playbook -i hosts flavors.yml --vault-password-file vaultpass.txt

show floating ip-addresses

    ansible-playbook -i hosts floatingip.yml --vault-password-file vaultpass.txt

show images

    ansible-playbook -i hosts images.yml --vault-password-file vaultpass.txt

show job status

    ansible-playbook -e "job_id=2c9eb2c15693b00901571e32ad5e1755" -i hosts job.yml --vault-password-file vaultpass.txt

    ./ajob 2c9eb2c15693b00901571e32ad5e1755

show keypairs

    ansible-playbook -i hosts keypairs.yml --vault-password-file vaultpass.txt

show security groups

    ansible-playbook -i hosts secgroups.yml --vault-password-file vaultpass.txt

show subnets

    ansible-playbook -i hosts subnet.yml --vault-password-file vaultpass.txt

show vpc

    ansible-playbook -i hosts vpc.yml --vault-password-file vaultpass.txt

Contributing
------------
Very welcome. We are in a very early state of automated platform deployment
on OTC. So each help is still welcome

1. Fork it.
2. Create a branch (`git checkout -b my_markup`)
3. Commit your changes (`git commit -am "Added Snarkdown"`)
4. Push to the branch (`git push origin my_markup`)
5. Open a [Pull Request][1]
6. Enjoy a refreshing Diet Coke and wait

