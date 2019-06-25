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
yum update -y
yum install git -y
reboot now
kvsh alpha
sudo su
cd
git clone https://github.com/alainpham/installcentos.git
cd installcentos
./install-openshift.sh
```

## Add elasticsearch & kibana
```
oc project openshift-logging
oc new-app elasticsearch:7.1.1 -e "discovery.type=single-node"

oc set probe dc/elasticsearch \
	--liveness \
	--failure-threshold 3 \
	--initial-delay-seconds 30 \
	-- echo ok
oc set probe dc/elasticsearch \
	--readiness \
	--failure-threshold 3 \
	--initial-delay-seconds 30 \
	--get-url=http://:9200/

oc new-app kibana:7.1.1
oc expose svc/kibana

oc set probe dc/kibana \
	--liveness \
	--failure-threshold 3 \
	--initial-delay-seconds 30 \
	-- echo ok
oc set probe dc/kibana \
	--readiness \
	--failure-threshold 3 \
	--initial-delay-seconds 30 \
	--get-url=http://:5601/app/kibana

```

## Add trusted certificate for okd image registry manipulations on distant machines
```
oc extract -n default secrets/registry-certificates --keys=registry.crt
cp registry.crt /etc/pki/ca-trust/source/anchors/registry-openshift-ca.crt
update-ca-trust extract
```

## From this point we will set the image registry url

We set the registry url and get an auth token (must be as cluster-admin) and we also set the crediantials for registry.redhat.io with user and pass.

```
REGISTRY="$(oc get route docker-registry -n default -o 'jsonpath={.spec.host}')"
ocuser=`oc whoami`
octoken=`oc whoami -t`

user=usr
pass=pwd

echo $REGISTRY
echo $ocuser:$octoken
```

## Downloading Fuse images with Skopeo

Credentials to registry.redhat.io are required here to use these enterprise images.

```
srcreg="docker://registry.redhat.io/"
tag="1.3"
ns="fuse7/"
origpfx=fuse-
targetpfx=fuse7-
imglist="java-openshift karaf-openshift eap-openshift console"

for img in $imglist
do
 echo $srcreg$ns$origpfx$img:$tag
 skopeo copy --screds $user:$pass $srcreg$ns$origpfx$img:$tag oci:./target:$ns$origpfx$img:$tag
done

for img in $imglist
do
 echo $srcreg$ns$origpfx$img:$tag
 skopeo --insecure-policy copy --dest-creds=$ocuser:$octoken oci:./target:$ns$origpfx$img:$tag docker://$REGISTRY/openshift/$targetpfx$img:$tag
done
```

## Downloading Apicurio images with Skopeo

Credentials to registry.redhat.io are required here to use these enterprise images.

```
srcreg="docker://registry.redhat.io/"
tag="1.3"
ns="fuse7/"
origpfx=fuse-
targetsfx=-ui
imglist="apicurito"

for img in $imglist
do
 echo $srcreg$ns$origpfx$img:$tag
 skopeo copy --screds $user:$pass $srcreg$ns$origpfx$img:$tag oci:./target:$ns$origpfx$img:$tag
done

for img in $imglist
do
 echo $srcreg$ns$origpfx$img:$tag
 skopeo --insecure-policy copy --dest-creds=$ocuser:$octoken oci:./target:$ns$origpfx$img:$tag docker://$REGISTRY/openshift/$img$targetsfx:$tag
done

```

## Downloading Apicurito Fuse images with Skopeo

Credentials to registry.redhat.io are required here to use these enterprise images.

```
srcreg="docker://registry.redhat.io/"
tag="1.3"
ns="fuse7/"
origpfx=fuse-
targetpfx=fuse-
imglist="apicurito-generator"

for img in $imglist
do
 echo $srcreg$ns$origpfx$img:$tag
 skopeo copy --screds $user:$pass $srcreg$ns$origpfx$img:$tag oci:./target:$ns$origpfx$img:$tag
done

for img in $imglist
do
 echo $srcreg$ns$origpfx$img:$tag
 skopeo --insecure-policy copy --dest-creds=$ocuser:$octoken oci:./target:$ns$origpfx$img:$tag docker://$REGISTRY/openshift/$targetpfx$img:$tag
done
```

## Downloading AMQ Broker images with Skopeo

Credentials to registry.redhat.io are required here to use these enterprise images.

```
srcreg="docker://registry.redhat.io/"
tag="7.3"
ns="amq-broker-7/"
imglist="amq-broker-73-openshift"

for img in $imglist
do
 echo $srcreg$ns$img:$tag
 skopeo copy --screds $user:$pass $srcreg$ns$img:$tag oci:./target:$ns$img:$tag
done

for img in $imglist
do
 echo $srcreg$ns$img:$tag
 skopeo --insecure-policy copy --dest-creds=$ocuser:$octoken oci:./target:$ns$img:$tag docker://$REGISTRY/openshift/$img:$tag
done

```

## Downloading AMQ Interconnect images with Skopeo

Credentials to registry.redhat.io registry are required here to use these enterprise images.

```
srcreg="docker://registry.redhat.io/"
tag="1.4"
ns="amq7/"
imglist="amq-interconnect"

for img in $imglist
do
 echo $srcreg$ns$img:$tag
 skopeo copy --screds $user:$pass $srcreg$ns$img:$tag oci:./target:$ns$img:$tag
done

for img in $imglist
do
 echo $srcreg$ns$img:$tag
 skopeo --insecure-policy copy --dest-creds=$ocuser:$octoken oci:./target:$ns$img:$tag docker://$REGISTRY/openshift/$img:$tag
done
```

# Create Fuse application templates


```
BASEURL=https://raw.githubusercontent.com/jboss-fuse/application-templates/application-templates-2.1.fuse-730065-redhat-00002

for template in eap-camel-amq-template.json \
 eap-camel-cdi-template.json \
 eap-camel-cxf-jaxrs-template.json \
 eap-camel-cxf-jaxws-template.json \
 eap-camel-jpa-template.json \
 karaf-camel-amq-template.json \
 karaf-camel-log-template.json \
 karaf-camel-rest-sql-template.json \
 karaf-cxf-rest-template.json \
 spring-boot-camel-amq-template.json \
 spring-boot-camel-config-template.json \
 spring-boot-camel-drools-template.json \
 spring-boot-camel-infinispan-template.json \
 spring-boot-camel-rest-sql-template.json \
 spring-boot-camel-teiid-template.json \
 spring-boot-camel-template.json \
 spring-boot-camel-xa-template.json \
 spring-boot-camel-xml-template.json \
 spring-boot-cxf-jaxrs-template.json \
 spring-boot-cxf-jaxws-template.json ;
 do
 oc create -n openshift -f \
 ${BASEURL}/quickstarts/${template}
 done

oc create -n openshift -f ${BASEURL}/fis-console-cluster-template.json
oc create -n openshift -f ${BASEURL}/fis-console-namespace-template.json

oc create -n openshift -f ${BASEURL}/fuse-apicurito.yml


```

# Deploy Fuse Console

```
oc new-project apps-prod
oc new-project apps
oc new-app fuse73-console
```

# Deploy devops stuff

```
oc new-project devops
oc new-app apicurito -p ROUTE_HOSTNAME=apicurito-devops.apps.88.198.65.4.nip.io
oc new-app sonatype/nexus
oc expose svc/nexus
oc set probe dc/nexus \
	--liveness \
	--failure-threshold 3 \
	--initial-delay-seconds 30 \
	-- echo ok
oc set probe dc/nexus \
	--readiness \
	--failure-threshold 3 \
	--initial-delay-seconds 30 \
	--get-url=http://:8081/nexus/content/groups/public

oc set volume dc/nexus --add \
	--name 'nexus-volume-1' \
	--type 'pvc' \
	--mount-path '/sonatype-work/' \
	--claim-name 'nexus-pv' \
	--claim-size '20G' \
	--overwrite

oc new-app jenkins-persistent -p MEMORY_LIMIT=4Gi -p VOLUME_CAPACITY=4Gi

oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:devops:jenkins

oc create -f https://raw.githubusercontent.com/microcks/microcks/master/install/openshift/openshift-persistent-full-template.yml -n openshift

oc new-app --template=microcks-persistent --param=APP_ROUTE_HOSTNAME=microcks-devops.app.88.198.65.4.nip.io --param=KEYCLOAK_ROUTE_HOSTNAME=keycloak-devops.app.88.198.65.4.nip.io --param=OPENSHIFT_MASTER=https://console.88.198.65.4.nip.io:8443 --param=OPENSHIFT_OAUTH_CLIENT_NAME=microcks-client

```

Setup proxy repos in nexus.

```
red-hat-early-access-repository
https://maven.repository.redhat.com/earlyaccess/all

red-hat-ga-repository
https://maven.repository.redhat.com/ga
```

You may also test a pipeline.




# Install syndesis

```
oc new-project syndesis
git clone https://github.com/syndesisio/syndesis.git -b 1.6.x
cd syndesis/tools/bin
./syndesis install -s --tag 1.6
./syndesis install --tag 1.6
```

# Install Strimzi (Kafka)

```
namespace=strimzi-kafka
brokername=event-kafka-broker
oc new-project $namespace

oc delete all,configmap,pvc,serviceaccount,rolebinding,CustomResourceDefinition,ClusterRole,clusterrolebindings -l app=strimzi

curl -L https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.12.1/strimzi-cluster-operator-0.12.1.yaml | sed "s/namespace: .*/namespace: $namespace/" | oc create  -n $namespace -f -

curl -L https://raw.githubusercontent.com/strimzi/strimzi-kafka-operator/0.12.1/examples/kafka/kafka-persistent-single.yaml | sed "s/name: .*/name: $brokername/" | oc create  -n $namespace -f -
```

# Configure Prometheus to scrape apps

```
pass=REPLACE_PASSWORD

oc new-project apps-monitoring
oc create secret generic prom --from-file distrib/prometheus.yml -o yaml --dry-run | oc create -f -

echo $pass | htpasswd -c -i -s distrib/auth internal

oc create secret generic prometheus-htpasswd --from-file distrib/auth -o yaml --dry-run | oc create -f -
rm distrib/auth internal

oc create -f distrib/prometheus-standalone.yml
oc new-app prometheus
oc adm policy add-cluster-role-to-user cluster-reader system:serviceaccount:apps-monitoring:prom
```

#Install grafana

```
cat distrib/promconnect.yml | sed "s/PASSWORD/$pass/" > promconnect.yml
oc create secret generic grafana-datasources --from-file promconnect.yml -o yaml --dry-run | oc create -f -
rm promconnect.yml

oc delete all -l app=grafana
oc delete service grafana
oc delete serviceaccount grafana
oc delete clusterrolebinding grafana-cluster-reader
oc delete secret grafana-proxy
oc delete routes grafana
oc delete configmap grafana-config

oc create -f distrib/grafana.yml
oc replace -f distrib/grafana.yml
oc new-app grafana

```
