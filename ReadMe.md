# Ansible playbooks for provisiong a centos host server with KVM to provision any vm.

Ansible playbooks to provision a centos with KVM and bunch of vms with fixed ip address and well defined reachable hostnames.
Can be used on Hetzner or OVH for example. In this example we are deploying RHEL server and RHEL Atomic.

* Place your qcow2 images in the distrib folder
* You may tweak 01-setup-hostmachine.yml to setup what images you want to upload.
* Change your usr/password for in group_vars/hostmachines.yml for Red Hat subs
* Tweak group_vars/hostmachines.yml to specify the number of vms you want. IP addresses of the vms will be 192.168.122.mac_ip_suffix

To be executed on a distant machine to setup the host machine

```
ansible-playbook 01-setup-hostmachine.yml -i inventory
```

To provision the vms

```
ansible-playbook 02-provision-vms.yml -i inventory
```

To delete the vms

```
ansible-playbook 03-provision-vms.yml -i inventory
```

# To Install Openshift (OKD) 3.11 on a single centos guest machine 

## Base Install
```
kvsh alpha
sudo su
cd
yum update
yum install git
git clone https://github.com/alainpham/installcentos.git
cd installcentos
./install-openshift.sh
```

## Add elasticsearch & kibana
```
oc project openshift-logging
oc new-app elasticsearch:7.1.1 -e "discovery.type=single-node"
oc new-app kibana:7.1.1
oc expose svc/kibana
```

## Downloading Fuse images with Skopeo

Credentials to the redhat.io registry are required here to use these enterprise images.

```
user=usr
pass=pwd
srcreg="docker://registry.redhat.io/"
tag="1.3"
imglist="fuse7/fuse-java-openshift fuse7/fuse-karaf-openshift fuse7/fuse-eap-openshift fuse7/fuse-console fuse7/fuse-ignite-server fuse7/fuse-online-operator fuse7/fuse-ignite-meta fuse7/fuse-ignite-s2i fuse7/fuse-ignite-ui fuse7/fuse-ignite-upgrade fuse7/fuse-apicurito-generator fuse7/fuse-apicurito "

for img in $imglist
do
 echo $srcreg$img:$tag
 skopeo copy --screds $user:$pass $srcreg$img:$tag oci:./target:$img:$tag
done
```

## Downloading AMQ Broker images with Skopeo

Credentials to the redhat.io registry are required here to use these enterprise images.

```
user=usr
pass=pwd
srcreg="docker://registry.redhat.io/"
tag="7.3"
imglist="amq-broker-7/amq-broker-73-openshift"

for img in $imglist
do
 echo $srcreg$img:$tag
 skopeo copy --screds $user:$pass $srcreg$img:$tag oci:./target:$img:$tag
done
```

## Downloading AMQ Broker images with Skopeo

Credentials to the redhat.io registry are required here to use these enterprise images.

```
user=usr
pass=pwd
srcreg="docker://registry.redhat.io/"
tag="1.4"
imglist="amq7/amq-interconnect"

for img in $imglist
do
 echo $srcreg$img:$tag
 skopeo copy --screds $user:$pass $srcreg$img:$tag oci:./target:$img:$tag
done
```



