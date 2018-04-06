# Ansible playbooks for provisiong a centos host server with KVM to provision any vm.

Ansible playbooks to provision a centos with KVM and bunch of vms with fixed ip address and well defined reachable hostnames.
Can be used on Hetzner or OVH for example. In this example we are deploying RHEL server and RHEL Atomic.

* Place your qcow2 images in the distrib folder
* You may tweak 01-setup-hostmachine.yml to setup what images you want to upload.
* Change your usr/password for in group_vars/hostmachines.yml for Red Hat subs
* Tweak group_vars/hostmachines.yml to specify the number of vms you want. IP addresses of the vms will be 192.168.122.mac_ip_suffix

To be executed on a distant machine to setup the host machine

ansible-playbook 01-setup-hostmachine.yml -i inventory

To provision the vms

ansible-playbook 02-provision-vms.yml -i inventory

To delete the vms

ansible-playbook 03-provision-vms.yml -i inventory


