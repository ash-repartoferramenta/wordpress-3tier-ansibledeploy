# Automate the deploy of a simple 3-tier Wordpress app running on Centos 9 Stream

## Before we begins
**Requirements**
- HashiCorp Vagrant for building and deploying virtual inftractructures
>  https://developer.hashicorp.com/vagrant/downloads
- Virtualbox as virtualization engine  (the selected vagrant box supports also paralles, libvirt, hyperv and VMware workstation but you need to edit vagrant file accordingly)
> https://www.virtualbox.org/wiki/Downloads
- Ansible for provisioning the VMs
>  https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

**Disclaimer** 
The following project has been realised for training and learning purpose and the material here provided  is only suitable for testing and learning enviroments;
please review the code carefully before utilizing it for any other scope.

## Implementation description

![wp-app](https://github.com/ash-repartoferramenta/wordpress-3tier-ansibledeploy/assets/135543207/658c6b96-4ab0-4550-9527-67e1f1bf7f85)


## Vagrant

The  vagrant file available in the Vagrant folder deploy and builds two VMs running the generic/centos9s box from Vagrant Cloud.
Subsequently vagrant will set up a private network where the application will run and launch an ansible playbook for provisioning
the required applications.
For running the vagrant file you must travel to the Vagrant folder and use 
```
vagrant up
```
Vagrant will auto-generate an inventory for ansible with the informations provided in its Vagrantfile.
It's therefore important to set up correctly some variables.

**Security concerns:**
For the sake of ease of use, this Vagrantfile is set to utilize the default insecure ssh key provided by Vagrant

**Variables**
- VWEBIP : this is the private network address that Vagrant will assign to the web server, it's used in absible to permit access from this host to mariadb
- VDIP: this is the private network address that Vagrant will assign to the database, it's used in ansible to bind mariadb service to this ip
- VSUB: subnet mask of the private network

PS.
though it's possible to obtain network informations directly from ansible I chose to pass VWEBIP and VDIP to ansible as special vars in case the VMs has multiple interfaces like 
in a default vagrant configuration.



## Ansible

In the ansible folder, three different roles are described:
- webtier
- dbtier
- wordpress

### webtier

webtier only specifies the task needed to install the most recent versions of php and httpd and ensures that the latter is enabled and running.
To make sure apache works correctly this role also set firewalld to permit http access to the host and selinux to permit access to external DBs.

**Security concerns:**
- firewalld => httpd enabled on all interfaces
- SElinux => httpd_can_network_connect_db = True

### dbtier

dbtier installs and enables mariad service initialing a master user, a new db and the wordpress user. Then this role bind mariadb service to the address provided from the vagrant file and allows (only) the wordpress host to access mariadb service. 

**Security concerns:**

- firewalld => enable the address specified in Vagrantfile as VWEBIP to access port 3306 on this host.

**Variables:**
Those variables **MUST** be customized from ./dbtier/vars, they initialize db and db users
- mariadbroot: root password of the dmbs
- masteruser: user used by wordpress to connect to the database
- masterpassword: password used by wordpress to connect to the database

those other variables are provided by Vagrantfile however the roles is still indipendent as long as the following vars are declared in /dbtier/vars/main.yml
- webip: see vagrantfile vars
- dbip: see vagrantfile vars
The following variables are defined in the ./dbtier/defaults but may be customised accordingly to your needs, remember that if you're trying to deploy this infrastructure
some variables must correspond with the ones declared in wordpress role
- dbname: wp-database
- mysql_port: "3306"

### wordpress
this role download and extract LATEST version of wordpress from https://wordpress.org/latest;
using a template a task then creates a new wp-config.php file with the data required for connecting to the database.
Afterwards Ansible compile the template with salt variables acquired from wordpress API (https://api.wordpress.org/secret-key/1.1/salt/)
and cleans all installation datas.

**Security concerns:**
- SElinux => set setype httpd_sys_content_t to all files downloaded from the wordpress website

**Current architectural concerns:**
Idempontency not granted by the task related to wp-config salting given the randomic nature of those APIs.
Moreover every time this role is executed, the latest version of wordpress will be redownloaded and extracted.

**Variables:**
Those variables **MUST** be customized from ./wordpress/vars, they initialize db and db users
- masteruser: the same user set in dbtier vars
- masterpassword: the same password set in dbtier vars

The following variables are defined in the ./wordpress/defaults but may be customised accordingly to your needs, remember that if you're trying to deploy this infrastructure
some variables must correspond with the ones declared in dbtier role
- dbname: name of the database # controllare se è lo stesso del dbtier
- mysql_port: mariadb port #controllare se è lo stesso del dbtier
- db_prefix: wordpress tables prefix 
