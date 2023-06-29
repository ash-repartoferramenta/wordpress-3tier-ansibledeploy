# Automate the deploy of a simple 3-tier Wordpress app running on Centos 9 Stream

## Before we begins
** Requirements **
- HashiCorp Vagrant for building and deploying virtual inftractructure
>  https://developer.hashicorp.com/vagrant/downloads
- Virtualbox as virtualization engine  (the selected vagrant box supports also paralles, libvirt, hyperv and VMware workstation but you need to edit vagrant file accordingly)
> https://www.virtualbox.org/wiki/Downloads
- Ansible for provisioning the VMs
>  https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

**Disclaimer**
The following project has been realised for training and learning purpose and the material here provided  is only suitable for testing and learning enviroments;
please review the code carefully before utilizing it for any other scope.

## Implementation description
![ansible-wordpress](https://github.com/ash-repartoferramenta/wordpress-3tier-ansibledeploy/assets/135543207/fbc808e1-5b38-45a8-b79c-e129522887e8)

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

**Variables**
VWEBIP : this is the private network address that Vagrant will assign to the web server, it's used in absible to permit access from this host to mariadb
VDIP: this is the private network address that Vagrant will assign to the database, it's used in ansible to bind mariadb service to this ip
VSUB: subnet mask of the private network

PS.
though it's possible to obtain network informations directly from ansible I chose to pass VWEBIP and VDIP to ansible as special vars in case the VMs has multiple interfaces like 
in a default vagrant configuration.

