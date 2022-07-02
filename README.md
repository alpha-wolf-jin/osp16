# osp16.2

**Git**
```
echo "# osp16" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/alpha-wolf-jin/osp16.git
git config --global credential.helper 'cache --timeout 7200'
git push -u origin main

git add . ; git commit -a -m "update README" ; git push -u origin main

```
**Hyperconverged Infrastructure Guide**

https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/16.2/html-single/hyperconverged_infrastructure_guide/index

**Create thin Diks**

```
# qemu-img create -f qcow2 osp16-compute-01-disk-01.qcow2 100g
Formatting 'osp16-compute-01-disk-01.qcow2', fmt=qcow2 size=107374182400 cluster_size=65536 lazy_refcounts=off refcount_bits=16

# qemu-img create -f qcow2 osp16-compute-01-disk-02.qcow2 100g

# qemu-img create -f qcow2 osp16-compute-01-disk-03.qcow2 100g

# qemu-img create -f qcow2 osp16-compute-02-disk-01.qcow2 100g

# qemu-img create -f qcow2 osp16-compute-02-disk-02.qcow2 100g

# qemu-img create -f qcow2 osp16-compute-02-disk-03.qcow2 100g

[root@localhost cluster01]# ll
total 336820556
-rw-r--r--. 1 qemu qemu 22300196864 Jun 26 00:18 osp16-ceph-01-disk-01.qcow2
-rw-r--r--. 1 qemu qemu 35350118400 Jun 26 00:18 osp16-ceph-01-disk-02.qcow2
-rw-r--r--. 1 qemu qemu 18341363712 Jun 26 00:18 osp16-ceph-01-disk-03.qcow2
-rw-r--r--. 1 qemu qemu 17017798656 Jun 26 00:18 osp16-ceph-01.qcow2
-rw-r--r--. 1 root root      197568 Jun  7  2020 osp16-ceph-02.qcow2
-rw-r--r--. 1 root root      198208 Jun 26 10:09 osp16-compute-01-disk-01.qcow2
-rw-r--r--. 1 root root      198208 Jun 26 10:09 osp16-compute-01-disk-02.qcow2
-rw-r--r--. 1 root root      198208 Jun 26 10:09 osp16-compute-01-disk-03.qcow2
-rw-r--r--. 1 qemu qemu 55607230464 Jun 26 00:18 osp16-compute-01.qcow2
-rw-r--r--. 1 root root      198208 Jun 26 10:09 osp16-compute-02-disk-01.qcow2
-rw-r--r--. 1 root root      198208 Jun 26 10:09 osp16-compute-02-disk-02.qcow2
-rw-r--r--. 1 root root      198208 Jun 26 10:09 osp16-compute-02-disk-03.qcow2
-rw-r--r--. 1 qemu qemu 60627484672 Jun 26 00:18 osp16-compute-02.qcow2
-rw-r--r--. 1 qemu qemu 45053444096 Jun 26 00:18 osp16-control-01.qcow2
-rw-r--r--. 1 qemu qemu 90572193792 Jun 26 01:07 osp16-director-01.qcow2

# qemu-img info osp16-compute-02-disk-01.qcow2
image: osp16-compute-02-disk-01.qcow2
file format: qcow2
virtual size: 100 GiB (107374182400 bytes)
disk size: 196 KiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false

```
**Clean UP**

https://access.redhat.com/solutions/5217561

https://access.redhat.com/solutions/2210421

```
re-running openstack undercloud install

$ openstack baremetal node list
+--------------------------------------+------------------+--------------------------------------+-------------+--------------------+-------------+
| UUID                                 | Name             | Instance UUID                        | Power State | Provisioning State | Maintenance |
+--------------------------------------+------------------+--------------------------------------+-------------+--------------------+-------------+
| 2eddeb14-488b-49f9-93a7-d4bd12092dc3 | osp16-control-01 | 34e8fb38-8436-4078-b0b7-9a7e064d79a9 | power on    | active             | False       |
| a7db5ef7-01e4-4758-a076-0a56565ca2ef | osp16-compute-01 | ab7ed22b-3c34-4bbe-ade8-abda9371fcaf | power on    | active             | False       |
| 05d36755-28ae-4b83-b71d-a504dd3407f8 | osp16-compute-02 | 9287b52c-c65e-49f2-8b7c-0bc0057d68c6 | power on    | active             | False       |
| f68ea388-2d16-4429-bacc-0d22e5ca64c7 | osp16-ceph-01    | 7620dcd3-3bef-4034-8742-b3c2de9b5882 | power on    | active             | False       |
+--------------------------------------+------------------+--------------------------------------+-------------+--------------------+-------------+

$ openstack stack delete overcloud --wait --yes

$ heat stack-list
WARNING (shell) "heat stack-list" is deprecated, please use "openstack stack list" instead
+----+------------+--------------+---------------+--------------+
| id | stack_name | stack_status | creation_time | updated_time |
+----+------------+--------------+---------------+--------------+
+----+------------+--------------+---------------+--------------+

$ openstack baremetal node delete $(openstack baremetal node list -f value -c UUID)

$ cat ./uninstall-undercloud.sh 
source stackrc
if [[ $(grep -oP " [0-9]+" /etc/rhosp-release) -ge 16 ]]; then 
  echo "OSP16+ deleting overcloud and containers";
  openstack overcloud delete overcloud -y
  sudo podman rm -f $(sudo podman ps -aq)
  sudo podman rmi $(sudo podman images -q)
fi
rm /home/stack/undercloud-passwords.conf
systemctl list-unit-files | grep openstack | awk '{print $1}' | xargs -I {} sudo systemctl stop {}
systemctl list-unit-files | grep neutron | awk '{print $1}' | xargs -I {} sudo systemctl stop {}
sudo systemctl stop docker
sudo systemctl stop podman
sudo systemctl stop keepalived
sudo rm -rf /var/lib/mysql /var/lib/rabbitmq /etc/keystone /opt/stack/.undercloud-setup /root/stackrc /home/stack/stackrc  /var/opt/undercloud-stack /var/lib/ironic-discoverd/discoverd.sqlite /var/lib/ironic-inspector/inspector.sqlite  /home/stack/.instack /root/tripleo-undercloud-passwords /var/lib/registry /etc/pki/ca-trust/source/anchors/cm-local-ca.pem /httpboot/ /tftpboot/ /srv/* /var/lib/docker /etc/os-net-config/config.json
dirs="ceilometer heat glance horizon ironic ironic-discoverd keystone neutron nova swift haproxy"; for dir in $dirs; do sudo rm -rf /etc/$dir ; sudo rm -rf /var/log/$dir ;  sudo rm -rf /var/lib/$dir; done
sudo yum remove -y rabbitmq-server mariadb haproxy openvswitch keepalived $(rpm -qa | grep openstack) python-tripleoclient docker os-collect-config os-net-config
sudo rm -rf /etc/httpd/conf.d/10*
sudo rm -rf /etc/httpd/conf.d/25*
sudo rm -rf /etc/httpd/conf.d/openstack-tripleo-ui.conf.rpmsave
sudo sed -i '/:3000/d ; /:35357/d ; /:5000/d ; /:8042/d ; /:8774/d ; /:8777/d' /etc/httpd/conf/ports.conf
sudo rm -rf /etc/pki/tls/certs/undercloud*
sudo rm -rf /etc/pki/tls/private/undercloud*
sudo rm -rf /etc/pki/ca-trust/source/anchors/*local*
sudo rm -rf /etc/sysconfig/network-scripts/ifcfg-br-ctlplane
sudo mv  /etc/os-net-config/config.json  /etc/os-net-config/config.json.orig
sudo sed -i '/ovs/d ; /OVS/d'  /etc/sysconfig/network-scripts/ifcfg-eth1
#!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
#!!!! remove all interface configuration which is related to openvswitch from /etc/sysconfig/network-scripts, e.g. ifcfg-br_int, ifcfg-ovs_system, ifcfg-brctlplane, etc. roll back the configuration of the local interface!!!!
#!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
#!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
#!!! Starting from RHOSP 12 /root/.my.cnf file should also be removed:
sudo rm /root/.my.cnf
#!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
#
# With containerized undercloud, we also need to delete /var/lib/config-data/
sudo rm -rf /var/lib/config-data
sudo rm -rf /etc/cloud/
sudo reboot

$ ./uninstall-undercloud.sh 

```

The home lab setup. Refer to 

https://docs.google.com/document/d/1AhHar-mkisFiXXdRxIJNh9Q0tA4IDUhKZN04rfx7yq4/edit


**Check vbmc**
```
$ vbmc list
+------------------+---------+---------+------+
| Domain name      | Status  | Address | Port |
+------------------+---------+---------+------+
| osp16-ceph-01    | running | ::      | 6453 |
| osp16-compute-01 | running | ::      | 6451 |
| osp16-comupte-02 | running | ::      | 6452 |
| osp16-control-01 | running | ::      | 6450 |
+------------------+---------+---------+------+

```
# clean undercloud in Red Hat Enterprise Linux Openstack Platform 

https://access.redhat.com/solutions/2210421

# Director Installation

https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/16.2/html-single/director_installation_and_usage/index

```
$ echo $LANG
en_US.UTF-8

```


# 3.1. Preparing the undercloud

https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/16.2/html-single/director_installation_and_usage/index

```
[stack@osp16-director-01 ~]$ hostname
osp16-director-01.example.com
[stack@osp16-director-01 ~]$ hostname -f
osp16-director-01.example.com

# this IP is used for ssh. is not used for OSP
$ grep $(hostname) /etc/hosts
#127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 osp16-director-01 osp16-director-01.example.com
192.168.122.30 osp16-director-01.example.com

```

**CDN regiter issue**

https://access.redhat.com/solutions/5187461

```
# mkdir -p /backup//etc/pki/product/

# cd /etc/pki/product/

# mv ./* /backup//etc/pki/product/.

# cp /etc/pki/product-default/* /etc/pki/product/

# cd /etc/pki/

# mv consumer/ /backup/etc/pki/

# mv entitlement /backup/etc/pki/

```

**Continue repositories setting**

```

$ id
uid=1000(stack) gid=1000(stack) groups=1000(stack) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

$ sudo subscription-manager register
Registering to: subscription.rhsm.redhat.com:443/subscription
Username: jinzha1@redhat.com
Password:
The system has been registered with ID: 1d6d7abd-4021-44b2-8e76-219675fc5f3e
The registered system name is: osp16-director-01.example.com

$ sudo subscription-manager list --available --all --matches="Red Hat OpenStack" >/tmp/osp.txt

$ grep -E "Subscription Name|Red Hat OpenStack|Pool ID" /tmp/osp.txt
...
Subscription Name:   Employee SKU
                     Red Hat OpenStack Beta Certification Test Suite
                     Red Hat OpenStack Certification Test Suite
                     Red Hat OpenStack Beta for IBM Power LE
                     Red Hat OpenStack - Extended Life Cycle Support
                     Red Hat OpenStack Beta
                     Red Hat OpenStack
                     Red Hat OpenStack 7 Extended Life Cycle Support
                     Red Hat OpenStack Advanced
                     Red Hat OpenStack for IBM Power
                     Red Hat OpenStack Beta Advanced
Pool ID:             8a85f99c7d76f2fd017d96c52b3a0676
...

$ sudo subscription-manager attach --pool=8a85f99c7d76f2fd017d96c52b3a0676

$ sudo subscription-manager release --set=8.4

$ sudo subscription-manager repos --disable=*

$ sudo subscription-manager repos \
--enable=rhel-8-for-x86_64-baseos-eus-rpms \
--enable=rhel-8-for-x86_64-appstream-eus-rpms \
--enable=rhel-8-for-x86_64-highavailability-eus-rpms \
--enable=ansible-2.9-for-rhel-8-x86_64-rpms \
--enable=openstack-16.2-for-rhel-8-x86_64-rpms \
--enable=fast-datapath-for-rhel-8-x86_64-rpms 

$ sudo dnf module disable -y container-tools:rhel8

$ sudo dnf module enable -y container-tools:3.0

$ sudo dnf remove -y python3-tripleoclient

$ sudo dnf remove -y ceph-ansible

$ sudo dnf remove rhosp-director-images rhosp-director-images-ipa-x86_64

$ ll /var/lib/ironic/httpboot/
ls: cannot access '/var/lib/ironic/httpboot/': No such file or directory

$ sudo dnf install git -y

$ sudo dnf update -y

$ sudo reboot

```

## 3.4. Installing director packages

```
$ sudo subscription-manager attach --pool=8a85f99c7d76f2fd017d96c52b3a0676

$ sudo subscription-manager release --set=8.4

$ sudo subscription-manager repos --disable=*

$ sudo subscription-manager repos \
--enable=rhel-8-for-x86_64-baseos-eus-rpms \
--enable=rhel-8-for-x86_64-appstream-eus-rpms \
--enable=rhel-8-for-x86_64-highavailability-eus-rpms \
--enable=ansible-2.9-for-rhel-8-x86_64-rpms \
--enable=openstack-16.2-for-rhel-8-x86_64-rpms \
--enable=fast-datapath-for-rhel-8-x86_64-rpms 

$ sudo dnf module disable -y container-tools:rhel8

$ sudo dnf install -y python3-tripleoclient

```

## 3.5. Installing ceph-ansible

```
$ sudo subscription-manager repos --enable=rhceph-4-tools-for-rhel-8-x86_64-rpms

$ sudo dnf install -y ceph-ansible

```

## 3.6. Preparing container images

```
$ mv containers-prepare-parameter.yaml containers-prepare-parameter.yaml-25-Jun-2022

$ sudo openstack tripleo container image prepare default \
  --local-push-destination \
  --output-env-file containers-prepare-parameter.yaml

$ vi containers-prepare-parameter.yaml
# Generated with the following on 2020-09-14T22:27:41.264210
#
#   openstack tripleo container image prepare default --local-push-destination --output-env-file containers-prepare-parameter.yaml
#

parameter_defaults:
  ContainerImagePrepare:
  - push_destination: true
    set:
      ceph_alertmanager_image: ose-prometheus-alertmanager
      ceph_alertmanager_namespace: registry.redhat.io/openshift4
      ceph_alertmanager_tag: 4.1
      ceph_grafana_image: rhceph-4-dashboard-rhel8
      ceph_grafana_namespace: registry.redhat.io/rhceph
      ceph_grafana_tag: 4
      ceph_image: rhceph-4-rhel8
      ceph_namespace: registry.redhat.io/rhceph
      ceph_node_exporter_image: ose-prometheus-node-exporter
      ceph_node_exporter_namespace: registry.redhat.io/openshift4
      ceph_node_exporter_tag: v4.1
      ceph_prometheus_image: ose-prometheus
      ceph_prometheus_namespace: registry.redhat.io/openshift4
      ceph_prometheus_tag: 4.1
      ceph_tag: latest
      name_prefix: openstack-
      name_suffix: ''
      namespace: registry.redhat.io/rhosp-rhel8
      neutron_driver: ovn
      rhel_containers: false
      tag: '16.1'
    tag_from_label: '{version}-{release}'
  ContainerImageRegistryCredentials:
    registry.redhat.io:
      jinzha1@redhat.com: W*M*@9

$ diff containers-prepare-parameter.yaml containers-prepare-parameter.yaml-25-Jun-2022
1c1
< # Generated with the following on 2022-06-25T17:03:22.966295
---
> # Generated with the following on 2020-09-14T22:27:41.264210
12c12
<       ceph_alertmanager_tag: v4.6
---
>       ceph_alertmanager_tag: 4.1
20c20
<       ceph_node_exporter_tag: v4.6
---
>       ceph_node_exporter_tag: v4.1
23c23
<       ceph_prometheus_tag: v4.6
---
>       ceph_prometheus_tag: 4.1
30c30
<       tag: '16.2'
---
>       tag: '16.1'


```

# Chapter 4. Installing director on the undercloud

Compare the below undercloud.conf with /usr/share/python-tripleoclient/undercloud.conf.sample 
 
And noted 'enable_ui' is not valid any more

```
[stack@osp16-director-01 ~]$ cat undercloud.conf 
[DEFAULT]
undercloud_hostname = osp16-director-01.example.com
container_images_file = containers-prepare-parameter.yaml
local_ip = 172.16.0.81/24
undercloud_public_host = 172.16.0.82
undercloud_admin_host = 172.16.0.83
#undercloud_nameservers = 192.0.2.254
#undercloud_ntp_servers =
overcloud_domain_name = example.com
subnets = ctlplane-subnet
local_subnet = ctlplane-subnet
#undercloud_service_certificate =
generate_service_certificate = true
certificate_generation_ca = local
local_interface = enp2s0
inspection_extras = false
undercloud_debug = false
enable_tempest = false
#enable_ui = false
clean_nodes=true
hieradata_override = /home/stack/instack-hieradata/undercloud-haproxy-hieradata-overrides.yaml

#undercloud_service_certificate = /etc/pki/instack-certs/undercloud.pem
#generate_service_certificate = false

[auth]

[ctlplane-subnet]
cidr = 172.16.0.0/24
dhcp_start = 172.16.0.85
dhcp_end = 172.16.0.95
inspection_iprange = 172.16.0.96,172.16.0.106
gateway = 172.16.0.81
masquerade = true

```

> Refer to below for how to create undercloud-haproxy-hieradata-overrides.yaml 

https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/16.2/html-single/director_installation_and_usage/index

29.3. Changing the SSL/TLS cipher rules for HAProxy

```
$ cat /home/stack/instack-hieradata/undercloud-haproxy-hieradata-overrides.yaml
tripleo::haproxy::ssl_cipher_suite: ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS
tripleo::haproxy::ssl_options: no-sslv3 no-tls-tickets

```

## 4.8. Installing director

```
$ openstack undercloud install

########################################################

Deployment successful!

########################################################

Writing the stack virtual update mark file /var/lib/tripleo-heat-installer/update_mark_undercloud

##########################################################

The Undercloud has been successfully installed.

Useful files:

Password file is at /home/stack/undercloud-passwords.conf
The stackrc file is at ~/stackrc

Use these files to interact with OpenStack services, and
ensure they are secured.

##########################################################

$ sudo podman ps
CONTAINER ID  IMAGE                                                                                             COMMAND               CREATED             STATUS                 PORTS   NAMES
539c0617b7af  docker.io/prom/node-exporter:v0.17.0                                                              --path.procfs=/ho...  2 hours ago         Up 2 hours ago                 node-exporter
68492162b418  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-keepalived:16.2                 /usr/local/bin/ko...  12 minutes ago      Up 12 minutes ago              keepalived
830232948b47  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-memcached:16.2                  kolla_start           12 minutes ago      Up 12 minutes ago              memcached
2c5c18d9ffac  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-haproxy:16.2                    kolla_start           12 minutes ago      Up 12 minutes ago              haproxy
05adb038db83  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-rabbitmq:16.2                   kolla_start           12 minutes ago      Up 12 minutes ago              rabbitmq
3a60296dce5f  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-mariadb:16.2                    kolla_start           11 minutes ago      Up 11 minutes ago              mysql
fcd45242c398  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-iscsid:16.2                     kolla_start           10 minutes ago      Up 10 minutes ago              iscsid
7cb8d9c40773  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-keystone:16.2                   kolla_start           10 minutes ago      Up 10 minutes ago              keystone
dc9471603d69  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-heat-api:16.2                   kolla_start           7 minutes ago       Up 7 minutes ago               heat_api
d38261952461  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-heat-api:16.2                   kolla_start           7 minutes ago       Up 7 minutes ago               heat_api_cron
b6abe481ecef  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-heat-engine:16.2                kolla_start           6 minutes ago       Up 6 minutes ago               heat_engine
e7a030bf2bc5  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-cron:16.2                       kolla_start           6 minutes ago       Up 6 minutes ago               logrotate_crond
6e779f51da54  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-mistral-engine:16.2             kolla_start           6 minutes ago       Up 6 minutes ago               mistral_engine
3f28dd5d0c0a  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-mistral-event-engine:16.2       kolla_start           6 minutes ago       Up 6 minutes ago               mistral_event_engine
bae159757305  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-mistral-executor:16.2           kolla_start           5 minutes ago       Up 5 minutes ago               mistral_executor
6a867e83bada  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-neutron-server-ovn:16.2         kolla_start           5 minutes ago       Up 5 minutes ago               neutron_api
be7dab174706  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-nova-conductor:16.2             kolla_start           5 minutes ago       Up 5 minutes ago               nova_conductor
d16312f3b8ba  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-nova-scheduler:16.2             kolla_start           5 minutes ago       Up 5 minutes ago               nova_scheduler
4c149cfaf369  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-swift-account:16.2              kolla_start           5 minutes ago       Up 5 minutes ago               swift_account_reaper
225d382b4956  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-swift-account:16.2              kolla_start           5 minutes ago       Up 5 minutes ago               swift_account_server
c836740e5d4e  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-swift-container:16.2            kolla_start           5 minutes ago       Up 5 minutes ago               swift_container_server
35570d1af927  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-swift-container:16.2            kolla_start           5 minutes ago       Up 5 minutes ago               swift_container_updater
a97ba38031a8  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-swift-proxy-server:16.2         kolla_start           5 minutes ago       Up 5 minutes ago               swift_object_expirer
d9df089635af  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-swift-object:16.2               kolla_start           5 minutes ago       Up 5 minutes ago               swift_object_server
bc139315517a  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-swift-object:16.2               kolla_start           5 minutes ago       Up 5 minutes ago               swift_object_updater
6b7c22d56a27  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-zaqar-wsgi:16.2                 kolla_start           5 minutes ago       Up 5 minutes ago               zaqar
563a7c4b8cbf  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-zaqar-wsgi:16.2                 kolla_start           4 minutes ago       Up 4 minutes ago               zaqar_websocket
9224dc9fe1c1  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-placement-api:16.2              kolla_start           4 minutes ago       Up 4 minutes ago               placement_api
f5fae67af9f6  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-glance-api:16.2                 kolla_start           4 minutes ago       Up 4 minutes ago               glance_api
736b01967935  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-nova-api:16.2                   kolla_start           4 minutes ago       Up 4 minutes ago               nova_api
5e08379aac86  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-swift-proxy-server:16.2         kolla_start           4 minutes ago       Up 4 minutes ago               swift_proxy
017f7d4152eb  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-nova-api:16.2                   kolla_start           4 minutes ago       Up 4 minutes ago               nova_api_cron
e918e058c616  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-ironic-api:16.2                 kolla_start           4 minutes ago       Up 4 minutes ago               ironic_api
53a14c37ff5d  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-neutron-dhcp-agent:16.2         kolla_start           4 minutes ago       Up 4 minutes ago               neutron_dhcp
bb151ad00f61  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-neutron-l3-agent:16.2           kolla_start           3 minutes ago       Up 3 minutes ago               neutron_l3_agent
a26172e3304c  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-neutron-openvswitch-agent:16.2  kolla_start           3 minutes ago       Up 3 minutes ago               neutron_ovs_agent
341c1b709bca  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-mistral-api:16.2                kolla_start           3 minutes ago       Up 3 minutes ago               mistral_api
800ddf1be4e5  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-ironic-conductor:16.2           kolla_start           3 minutes ago       Up 3 minutes ago               ironic_conductor
fce9eaa71760  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-ironic-neutron-agent:16.2       kolla_start           3 minutes ago       Up 3 minutes ago               ironic_neutron_agent
1e4ae51e9083  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-ironic-pxe:16.2                 /bin/bash -c BIND...  3 minutes ago       Up 3 minutes ago               ironic_pxe_tftp
dc4a3b0dcc80  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-ironic-pxe:16.2                 kolla_start           3 minutes ago       Up 3 minutes ago               ironic_pxe_http
99081a5e4c9f  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-ironic-inspector:16.2           kolla_start           3 minutes ago       Up 3 minutes ago               ironic_inspector
dafe3301622c  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-ironic-inspector:16.2           kolla_start           3 minutes ago       Up 3 minutes ago               ironic_inspector_dnsmasq
3d8b7471805c  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-nova-compute-ironic:16.2        kolla_start           2 minutes ago       Up 2 minutes ago               nova_compute
42c3500f0671  osp16-director-01.ctlplane.example.com:8787/rhosp-rhel8/openstack-neutron-dhcp-agent:16.2         /usr/sbin/dnsmasq...  About a minute ago  Up About a minute ago          neutron-dnsmasq-qdhcp-c264d6b2-6a7f-497c-a4b6-094fb70e31fe

$ source ~/stackrc
(undercloud) [stack@osp16-director-01 ~]$ 

$ ll /var/lib/ironic/httpboot/*
-rw-r--r--. 1 42422 42422 758 Jun 25 21:50 /var/lib/ironic/httpboot/boot.ipxe
-rw-r--r--. 1 42422 42422 371 Jun 25 21:44 /var/lib/ironic/httpboot/inspector.ipxe

$ openstack baremetal driver list
+---------------------+-------------------------------+
| Supported driver(s) | Active host(s)                |
+---------------------+-------------------------------+
| idrac               | osp16-director-01.example.com |
| ilo                 | osp16-director-01.example.com |
| ipmi                | osp16-director-01.example.com |
| redfish             | osp16-director-01.example.com |
+---------------------+-------------------------------+

```

## 4.10.1. Single CPU architecture overcloud images
Your Red Hat OpenStack Platform (RHOSP) installation includes packages that provide you with the following overcloud images for director:

- overcloud-full
- overcloud-full-initrd
- overcloud-full-vmlinuz



```
$ source ~/stackrc

$ sudo dnf install rhosp-director-images rhosp-director-images-ipa-x86_64

$ rm -rf ./images/

$ mkdir /home/stack/images

$ cd ~/images

$ for i in /usr/share/rhosp-director-images/overcloud-full-latest-16.2.tar /usr/share/rhosp-director-images/ironic-python-agent-latest-16.2.tar; do tar -xvf $i; done

$ ll
total 1813340
-rw-r--r--. 1 stack stack  452668994 Jun 10 05:25 ironic-python-agent.initramfs
-rwxr-xr-x. 1 stack stack   10034544 Jun 10 05:25 ironic-python-agent.kernel
-rw-r--r--. 1 stack stack   86768425 Jun 10 05:30 overcloud-full.initrd
-rw-r--r--. 1 stack stack 1297219584 Jun 10 05:57 overcloud-full.qcow2
-rw-r--r--. 1 stack stack      35519 Jun 10 05:36 overcloud-full-rpm.manifest
-rw-r--r--. 1 stack stack      90355 Jun 10 05:36 overcloud-full-signature.manifest
-rw-r--r--. 1 stack stack   10034544 Jun 10 05:30 overcloud-full.vmlinuz

$ openstack image list

$ openstack overcloud image upload --image-path /home/stack/images/

$ openstack image list
+--------------------------------------+------------------------+--------+
| ID                                   | Name                   | Status |
+--------------------------------------+------------------------+--------+
| 53752af1-f646-42bf-ab88-ef6bdc7004b6 | overcloud-full         | active |
| 6cf2f25a-e94f-4cf2-a26f-34951def60b7 | overcloud-full-initrd  | active |
| 7e60b2a2-08e7-46c1-a0d3-432ac38673fb | overcloud-full-vmlinuz | active |
+--------------------------------------+------------------------+--------+

$  ls -l /var/lib/ironic/httpboot
total 451868
-rwxr-xr-x. 1 root  root   10034544 Jun 25 22:26 agent.kernel
-rw-r--r--. 1 root  root  452668994 Jun 25 22:26 agent.ramdisk
-rw-r--r--. 1 42422 42422       758 Jun 25 21:50 boot.ipxe
-rw-r--r--. 1 42422 42422       371 Jun 25 21:44 inspector.ipxe

$ openstack tripleo container image list


```

# Chapter 7. Configuring a basic overcloud

## 7.1. Registering nodes for the overcloud

```
$ cat ./introspect/nodes.json
{
    "nodes": [
        {
            "mac": [
                "52:54:00:95:42:24"
            ],
            "name": "osp16-control-01",
            "pm_addr": "127.0.0.1",
            "pm_port": "6450",
            "pm_password": "redhat",
            "pm_type": "pxe_ipmitool",
            "pm_user": "admin"
        },
        {
            "mac": [
                "52:54:00:f1:97:53"
            ],
            "name": "osp16-compute-01",
            "pm_addr": "127.0.0.1",
            "pm_port": "6451",
            "pm_password": "redhat",
            "pm_type": "pxe_ipmitool",
            "pm_user": "admin"
        },
        {
            "mac": [
                "52:54:00:cc:f3:8f"
            ],
            "name": "osp16-compute-02",
            "pm_addr": "127.0.0.1",
            "pm_port": "6452",
            "pm_password": "redhat",
            "pm_type": "pxe_ipmitool",
            "pm_user": "admin"
        },
        {
            "mac": [
                "52:54:00:ed:ab:3d"
            ],
            "name": "osp16-ceph-01",
            "pm_addr": "127.0.0.1",
            "pm_port": "6453",
            "pm_password": "redhat",
            "pm_type": "pxe_ipmitool",
            "pm_user": "admin"
        }
    ]
}

# Sample for Dell Server
actlr0.json

{
  "nodes": [{
    "name": "actlr0",
    "mac": ["f0:d4:e2:e6:0d:26"],
    "pm_type": "ipmi",
    "pm_user": "root",
    "pm_password": "il0mpswd16#",
    "pm_addr": "192.168.15.180"
  }]
}

$ source ~/stackrc

$  openstack overcloud node import --validate-only  ~/introspect/nodes.json 
Waiting for messages on queue 'tripleo' with no timeout.

Successfully validated environment file

$ openstack baremetal node list

$ openstack overcloud node import ~/introspect/nodes.json

$ openstack baremetal node list
+--------------------------------------+------------------+---------------+-------------+--------------------+-------------+
| UUID                                 | Name             | Instance UUID | Power State | Provisioning State | Maintenance |
+--------------------------------------+------------------+---------------+-------------+--------------------+-------------+
| 34e6c725-bfb3-47f9-8230-655e70dcc16b | osp16-control-01 | None          | power off   | manageable         | False       |
| 21508e01-fc9b-410f-92f8-2659efbe13a3 | osp16-compute-01 | None          | power off   | manageable         | False       |
| c6eb1460-3a03-4ec3-9460-f593d47fa8fb | osp16-compute-02 | None          | power off   | manageable         | False       |
| 46f53654-981f-44b6-9f7b-764ce26ea9e6 | osp16-ceph-01    | None          | power off   | manageable         | False       |
+--------------------------------------+------------------+---------------+-------------+--------------------+-------------+

$ openstack baremetal node list
+--------------------------------------+------------------+---------------+-------------+--------------------+-------------+
| UUID                                 | Name             | Instance UUID | Power State | Provisioning State | Maintenance |
+--------------------------------------+------------------+---------------+-------------+--------------------+-------------+
| 34e6c725-bfb3-47f9-8230-655e70dcc16b | osp16-control-01 | None          | power off   | available          | False       |
| 21508e01-fc9b-410f-92f8-2659efbe13a3 | osp16-compute-01 | None          | power off   | available          | False       |
| c6eb1460-3a03-4ec3-9460-f593d47fa8fb | osp16-compute-02 | None          | power off   | available          | False       |
| 46f53654-981f-44b6-9f7b-764ce26ea9e6 | osp16-ceph-01    | None          | power off   | available          | False       |
+--------------------------------------+------------------+---------------+-------------+--------------------+-------------+

```

## 7.2.1. Using director introspection to collect bare metal node hardware information

```
$ cd introspect/

$ openstack overcloud node import --introspect --provide nodes.json 
aiting for messages on queue 'tripleo' with no timeout.

0 node(s) successfully moved to the "manageable" state.
Successfully registered node UUID 34e6c725-bfb3-47f9-8230-655e70dcc16b
Successfully registered node UUID 21508e01-fc9b-410f-92f8-2659efbe13a3
Successfully registered node UUID c6eb1460-3a03-4ec3-9460-f593d47fa8fb
Successfully registered node UUID 46f53654-981f-44b6-9f7b-764ce26ea9e6
Waiting for introspection to finish...
Waiting for messages on queue 'tripleo' with no timeout.
Introspection of node completed:34e6c725-bfb3-47f9-8230-655e70dcc16b. Status:SUCCESS. Errors:None
Introspection of node completed:c6eb1460-3a03-4ec3-9460-f593d47fa8fb. Status:SUCCESS. Errors:None
Introspection of node completed:21508e01-fc9b-410f-92f8-2659efbe13a3. Status:SUCCESS. Errors:None
Introspection of node completed:46f53654-981f-44b6-9f7b-764ce26ea9e6. Status:SUCCESS. Errors:None
Successfully introspected 4 node(s).
Waiting for messages on queue 'tripleo' with no timeout.
4 node(s) successfully moved to the "available" state.


```

**Monitor the introspection progress logs in a separate terminal window**

```
$ sudo tail -f /var/log/containers/ironic-inspector/ironic-inspector.log
2022-06-26 00:15:11.465 8 INFO eventlet.wsgi.server [-] 172.16.0.81 "OPTIONS / HTTP/1.0" status: 200  len: 272 time: 0.0011783
2022-06-26 00:15:13.475 8 INFO eventlet.wsgi.server [-] 172.16.0.81 "OPTIONS / HTTP/1.0" status: 200  len: 272 time: 0.0012264
2022-06-26 00:15:15.484 8 INFO eventlet.wsgi.server [-] 172.16.0.81 "OPTIONS / HTTP/1.0" status: 200  len: 272 time: 0.0006032
2022-06-26 00:15:17.485 8 INFO eventlet.wsgi.server [-] 172.16.0.81 "OPTIONS / HTTP/1.0" status: 200  len: 272 time: 0.0006082
2022-06-26 00:15:19.486 8 INFO eventlet.wsgi.server [-] 172.16.0.81 "OPTIONS / HTTP/1.0" status: 200  len: 272 time: 0.0005541
2022-06-26 00:15:19.687 8 INFO eventlet.wsgi.server [req-b101c1f4-0a47-4555-bd5c-8db4284957d1 069c053c471c447298200a87a05bdc59 a093361368044a8bb6e61057c95990be - default default] 172.16.0.81 "GET /v1/introspection/34e6c725-bfb3-47f9-8230-655e70dcc16b HTTP/1.1" status: 200  len: 483 time: 0.0045085
```

## 7.2.2. Manually configuring bare-metal node hardware information

```
$ openstack baremetal node show osp16-control-01 | grep properties
| properties             | {'vendor': 'unknown', 'local_gb': '59', 'cpus': '4', 'cpu_arch': 'x86_64', 'memory_mb': '16000', 'capabilities': 'cpu_vt:true,cpu_aes:true,cpu_hugepages:true,cpu_hugepages_1g:true'}             

$ openstack baremetal node set --property capabilities='node:controller-0,profile:control,boot_option:local,cpu_vt:true,cpu_aes:true,cpu_hugepages:true,cpu_hugepages_1g:true'  osp16-control-01

```
## 2.5. Defining the root disk for multi-disk clusters

https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/16.2/html-single/deploying_an_overcloud_with_containerized_red_hat_ceph/index#manual-node-tag

```
$ openstack baremetal introspection data save osp16-compute-01 | jq .
{
  "inventory": {
    "interfaces": [
      {
        "name": "enp2s0",
        "mac_address": "52:54:00:08:d4:46",
        "ipv4_address": null,
        "ipv6_address": "fe80::5054:ff:fe08:d446%enp2s0",
        "has_carrier": true,
        "lldp": null,
        "vendor": "0x1af4",
        "product": "0x0001",
        "client_id": null,
        "biosdevname": null
      },
      {
        "name": "enp5s0",
        "mac_address": "52:54:00:5b:0d:eb",
        "ipv4_address": null,
        "ipv6_address": "fe80::5054:ff:fe5b:deb%enp5s0",
        "has_carrier": true,
        "lldp": null,
        "vendor": "0x1af4",
        "product": "0x0001",
        "client_id": null,
        "biosdevname": null
      },
      {
        "name": "enp1s0",
        "mac_address": "52:54:00:f1:97:53",
        "ipv4_address": "172.16.0.99",
        "ipv6_address": "fe80::491f:a03:e6e8:76f0%enp1s0",
        "has_carrier": true,
        "lldp": null,
        "vendor": "0x1af4",
        "product": "0x0001",
        "client_id": null,
        "biosdevname": null
      },
      {
        "name": "enp4s0",
        "mac_address": "52:54:00:c1:72:dc",
        "ipv4_address": null,
        "ipv6_address": "fe80::5054:ff:fec1:72dc%enp4s0",
        "has_carrier": true,
        "lldp": null,
        "vendor": "0x1af4",
        "product": "0x0001",
        "client_id": null,
        "biosdevname": null
      },
      {
        "name": "enp3s0",
        "mac_address": "52:54:00:3f:2f:12",
        "ipv4_address": null,
        "ipv6_address": "fe80::5054:ff:fe3f:2f12%enp3s0",
        "has_carrier": true,
        "lldp": null,
        "vendor": "0x1af4",
        "product": "0x0001",
        "client_id": null,
        "biosdevname": null
      }
    ],
    "cpu": {
      "model_name": "Intel Core Processor (Skylake, IBRS)",
      "frequency": "2712.000",
      "count": 12,
      "architecture": "x86_64",
      "flags": [
        "fpu",
        "vme",
        "de",
        "pse",
        "tsc",
        "msr",
        "pae",
        "mce",
        "cx8",
        "apic",
        "sep",
        "mtrr",
        "pge",
        "mca",
        "cmov",
        "pat",
        "pse36",
        "clflush",
        "mmx",
        "fxsr",
        "sse",
        "sse2",
        "ss",
        "syscall",
        "nx",
        "pdpe1gb",
        "rdtscp",
        "lm",
        "constant_tsc",
        "rep_good",
        "nopl",
        "xtopology",
        "cpuid",
        "tsc_known_freq",
        "pni",
        "pclmulqdq",
        "vmx",
        "ssse3",
        "fma",
        "cx16",
        "pcid",
        "sse4_1",
        "sse4_2",
        "x2apic",
        "movbe",
        "popcnt",
        "tsc_deadline_timer",
        "aes",
        "xsave",
        "avx",
        "f16c",
        "rdrand",
        "hypervisor",
        "lahf_lm",
        "abm",
        "3dnowprefetch",
        "cpuid_fault",
        "invpcid_single",
        "pti",
        "ssbd",
        "ibrs",
        "ibpb",
        "stibp",
        "tpr_shadow",
        "vnmi",
        "flexpriority",
        "ept",
        "vpid",
        "ept_ad",
        "fsgsbase",
        "tsc_adjust",
        "bmi1",
        "hle",
        "avx2",
        "smep",
        "bmi2",
        "erms",
        "invpcid",
        "rtm",
        "mpx",
        "rdseed",
        "adx",
        "smap",
        "clflushopt",
        "xsaveopt",
        "xsavec",
        "xgetbv1",
        "xsaves",
        "arat",
        "umip",
        "md_clear"
      ]
    },
    "disks": [
      {
        "name": "/dev/vda",
        "model": "",
        "size": 64424509440,
        "rotational": true,
        "wwn": null,
        "serial": null,
        "vendor": "0x1af4",
        "wwn_with_extension": null,
        "wwn_vendor_extension": null,
        "hctl": null,
        "by_path": "/dev/disk/by-path/pci-0000:08:00.0"
      },
      {
        "name": "/dev/vdb",
        "model": "",
        "size": 107374182400,
        "rotational": true,
        "wwn": null,
        "serial": null,
        "vendor": "0x1af4",
        "wwn_with_extension": null,
        "wwn_vendor_extension": null,
        "hctl": null,
        "by_path": "/dev/disk/by-path/pci-0000:0b:00.0"
      },
      {
        "name": "/dev/vdc",
        "model": "",
        "size": 107374182400,
        "rotational": true,
        "wwn": null,
        "serial": null,
        "vendor": "0x1af4",
        "wwn_with_extension": null,
        "wwn_vendor_extension": null,
        "hctl": null,
        "by_path": "/dev/disk/by-path/pci-0000:0c:00.0"
      },
      {
        "name": "/dev/vdd",
        "model": "",
        "size": 107374182400,
        "rotational": true,
        "wwn": null,
        "serial": null,
        "vendor": "0x1af4",
        "wwn_with_extension": null,
        "wwn_vendor_extension": null,
        "hctl": null,
        "by_path": "/dev/disk/by-path/pci-0000:0d:00.0"
      }
    ],
    "memory": {
      "total": 55627997184,
      "physical_mb": 54000
    },
    "bmc_address": "0.0.0.0",
    "bmc_v6address": "::/0",
    "system_vendor": {
      "product_name": "KVM",
      "serial_number": "",
      "manufacturer": "Red Hat"
    },
    "boot": {
      "current_boot_mode": "bios",
      "pxe_interface": "52:54:00:f1:97:53"
    },
    "hostname": "localhost.localdomain"
  },
  "root_disk": {
    "name": "/dev/vda",
    "model": "",
    "size": 64424509440,
    "rotational": true,
    "wwn": null,
    "serial": null,
    "vendor": "0x1af4",
    "wwn_with_extension": null,
    "wwn_vendor_extension": null,
    "hctl": null,
    "by_path": "/dev/disk/by-path/pci-0000:08:00.0"
  },
  "boot_interface": "52:54:00:f1:97:53",
  "error": null,
  "ipmi_address": null,
  "ipmi_v6address": null,
  "all_interfaces": {
    "enp2s0": {
      "ip": "fe80::5054:ff:fe08:d446",
      "mac": "52:54:00:08:d4:46",
      "client_id": null,
      "pxe": false
    },
    "enp5s0": {
      "ip": "fe80::5054:ff:fe5b:deb",
      "mac": "52:54:00:5b:0d:eb",
      "client_id": null,
      "pxe": false
    },
    "enp1s0": {
      "ip": "172.16.0.99",
      "mac": "52:54:00:f1:97:53",
      "client_id": null,
      "pxe": true
    },
    "enp4s0": {
      "ip": "fe80::5054:ff:fec1:72dc",
      "mac": "52:54:00:c1:72:dc",
      "client_id": null,
      "pxe": false
    },
    "enp3s0": {
      "ip": "fe80::5054:ff:fe3f:2f12",
      "mac": "52:54:00:3f:2f:12",
      "client_id": null,
      "pxe": false
    }
  },
  "interfaces": {
    "enp1s0": {
      "ip": "172.16.0.99",
      "mac": "52:54:00:f1:97:53",
      "client_id": null,
      "pxe": true
    }
  },
  "macs": [
    "52:54:00:f1:97:53"
  ],
  "local_gb": 59,
  "cpus": 12,
  "cpu_arch": "x86_64",
  "memory_mb": 54000
}

$ openstack baremetal introspection data save osp16-compute-01 | jq ".inventory.disks"
[
  {
    "name": "/dev/vda",
    "model": "",
    "size": 64424509440,
    "rotational": true,
    "wwn": null,
    "serial": null,
    "vendor": "0x1af4",
    "wwn_with_extension": null,
    "wwn_vendor_extension": null,
    "hctl": null,
    "by_path": "/dev/disk/by-path/pci-0000:08:00.0"
  },
  {
    "name": "/dev/vdb",
    "model": "",
    "size": 107374182400,
    "rotational": true,
    "wwn": null,
    "serial": null,
    "vendor": "0x1af4",
    "wwn_with_extension": null,
    "wwn_vendor_extension": null,
    "hctl": null,
    "by_path": "/dev/disk/by-path/pci-0000:0b:00.0"
  },
  {
    "name": "/dev/vdc",
    "model": "",
    "size": 107374182400,
    "rotational": true,
    "wwn": null,
    "serial": null,
    "vendor": "0x1af4",
    "wwn_with_extension": null,
    "wwn_vendor_extension": null,
    "hctl": null,
    "by_path": "/dev/disk/by-path/pci-0000:0c:00.0"
  },
  {
    "name": "/dev/vdd",
    "model": "",
    "size": 107374182400,
    "rotational": true,
    "wwn": null,
    "serial": null,
    "vendor": "0x1af4",
    "wwn_with_extension": null,
    "wwn_vendor_extension": null,
    "hctl": null,
    "by_path": "/dev/disk/by-path/pci-0000:0d:00.0"
  }
]


```

## 1.2. Preparing the overcloud role for hyperconverged nodes

https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/16.2/html-single/hyperconverged_infrastructure_guide/index

```
$ source ~/stackrc

$ openstack overcloud roles  generate -o /home/stack/templates/roles_data_hci.yaml Controller ComputeHCI

$ vi /home/stack/templates/roles_data_hci.yaml
###############################################################################
# Role: ComputeHCI                                                            #
###############################################################################
- name: ComputeHCI
  description: |
    Compute Node role hosting Ceph OSD too
  networks:
    InternalApi:
      subnet: internal_api_subnet
    Tenant:
      subnet: tenant_subnet
    Storage:
      subnet: storage_subnet
    StorageMgmt:
      subnet: storage_mgmt_subnet

```
## 3.3. Creating a custom role and flavor for the Ceph MDS service

https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/16.2/html-single/deploying_an_overcloud_with_containerized_red_hat_ceph/index#dedicated-nodes


```

$ openstack flavor create --id auto --ram 54000 --disk 60 --vcpus 12 computeHCI
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| disk                       | 60                                   |
| id                         | 9a11df9b-afa3-447a-9882-ea3a428c4752 |
| name                       | computeHCI                           |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 54000                                |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
| vcpus                      | 12                                   |
+----------------------------+--------------------------------------+

$ openstack baremetal node list
+--------------------------------------+------------------+---------------+-------------+--------------------+-------------+
| UUID                                 | Name             | Instance UUID | Power State | Provisioning State | Maintenance |
+--------------------------------------+------------------+---------------+-------------+--------------------+-------------+
| 7934360c-ae2e-4422-9658-15e2700f7fb2 | osp16-control-01 | None          | power off   | available          | False       |
| 775df8d1-6320-47f8-86cb-b04c95fca766 | osp16-compute-01 | None          | power off   | available          | False       |
| f3d86cf9-1c4a-4d9b-88c9-b2ebd47fea1c | osp16-compute-02 | None          | power off   | available          | False       |
| 9894edb2-314b-4193-b15e-5c138a1d3db4 | osp16-ceph-01    | None          | power off   | available          | False       |
+--------------------------------------+------------------+---------------+-------------+--------------------+-------------+

$ openstack baremetal node set --resource-class baremetal.HCI osp16-compute-01

```

**Associate the computeHCI flavor with the custom HCI resource class**

To determine the name of a custom resource class that corresponds to a resource class of a Bare Metal service node, convert the resource class to uppercase, replace all punctuation with an underscore, and prefix with CUSTOM_.

```
$ openstack flavor set --property resources:CUSTOM_BAREMETAL_HCI=1 computeHCI

```

> Note: A flavor can request only one instance of a bare metal resource class.

**Set the following flavor properties to prevent the Compute scheduler from using the bare metal flavor properties to schedule instances**

```
$ openstack flavor set --property resources:VCPU=0 --property resources:MEMORY_MB=0 --property resources:DISK_GB=0 computeHCI

```

**Add the following parameters to the node-info.yaml file to specify the number of hyperconverged and Controller nodes, and the flavor to use for the hyperconverged and controller designated nodes**

```
$ vim templates/environments/02_node-info.yaml
parameter_defaults:
  OvercloudComputeHCIFlavor: computeHCI
  ComputeHCICount: 1
  OvercloudControllerFlavor: baremetal
  ControllerCount: 1

```
## 1.2.1. Defining the root disk for multi-disk clusters

https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/16.2/html-single/hyperconverged_infrastructure_guide/index

```
$ openstack baremetal node set osp16-control-01 --property capabilities='node:controller-0,profile:control,boot_option:local,cpu_vt:true,cpu_aes:true,cpu_hugepages:true,cpu_hugepages_1g:true'

$ openstack baremetal node set osp16-compute-01 --property capabilities=profile:computehci,boot_option:local

$ openstack baremetal node set osp16-compute-01 --property capabilities='node:computerHCI-0,profile:ComputeHCI,boot_option:local,cpu_vt:true,cpu_aes:true,cpu_hugepages:true,cpu_hugepages_1g:true'

$ openstack baremetal node set --property root_device='{"by_path": "/dev/disk/by-path/pci-0000:08:00.0"}' osp16-compute-01

$ openstack baremetal node show osp16-compute-01 --fit-width
+------------------------+-------------------------------------------------------------------------------------+
| Field                  | Value                                                                               |
+------------------------+-------------------------------------------------------------------------------------+
| allocation_uuid        | None                                                                                |
| automated_clean        | None                                                                                |
| bios_interface         | no-bios                                                                             |
| boot_interface         | ipxe                                                                                |
| chassis_uuid           | None                                                                                |
| clean_step             | {}                                                                                  |
| conductor              | osp16-director-01.example.com                                                       |
| conductor_group        |                                                                                     |
| console_enabled        | False                                                                               |
| console_interface      | ipmitool-socat                                                                      |
| created_at             | 2022-06-26T03:44:44+00:00                                                           |
| deploy_interface       | iscsi                                                                               |
| deploy_step            | {}                                                                                  |
| description            | None                                                                                |
| driver                 | ipmi                                                                                |
| driver_info            | {'deploy_kernel': 'file:///var/lib/ironic/httpboot/agent.kernel', 'rescue_kernel':  |
|                        | 'file:///var/lib/ironic/httpboot/agent.kernel', 'deploy_ramdisk':                   |
|                        | 'file:///var/lib/ironic/httpboot/agent.ramdisk', 'rescue_ramdisk':                  |
|                        | 'file:///var/lib/ironic/httpboot/agent.ramdisk', 'ipmi_address': '127.0.0.1',       |
|                        | 'ipmi_port': '6451', 'ipmi_password': '******', 'ipmi_username': 'admin'}           |
| driver_internal_info   | {'agent_erase_devices_iterations': 1, 'agent_erase_devices_zeroize': True,          |
|                        | 'agent_continue_if_ata_erase_failed': False, 'agent_enable_ata_secure_erase': True, |
|                        | 'disk_erasure_concurrency': 1, 'last_power_state_change':                           |
|                        | '2022-06-26T03:49:56.324476', 'agent_version': '5.0.5.dev27',                       |
|                        | 'agent_last_heartbeat': '2022-06-26T03:49:55.930998', 'hardware_manager_version':   |
|                        | {'generic_hardware_manager': '1.1'}, 'agent_cached_clean_steps': {'deploy':         |
|                        | [{'step': 'erase_devices', 'priority': 10, 'interface': 'deploy',                   |
|                        | 'reboot_requested': False, 'abortable': True}, {'step': 'erase_devices_metadata',   |
|                        | 'priority': 99, 'interface': 'deploy', 'reboot_requested': False, 'abortable':      |
|                        | True}], 'raid': [{'step': 'delete_configuration', 'priority': 0, 'interface':       |
|                        | 'raid', 'reboot_requested': False, 'abortable': True}, {'step':                     |
|                        | 'create_configuration', 'priority': 0, 'interface': 'raid', 'reboot_requested':     |
|                        | False, 'abortable': True}]}, 'agent_cached_clean_steps_refreshed': '2022-06-26      |
|                        | 03:49:50.399556', 'clean_steps': None}                                              |
| extra                  | {}                                                                                  |
| fault                  | None                                                                                |
| inspect_interface      | inspector                                                                           |
| inspection_finished_at | None                                                                                |
| inspection_started_at  | None                                                                                |
| instance_info          | {}                                                                                  |
| instance_uuid          | None                                                                                |
| last_error             | None                                                                                |
| maintenance            | False                                                                               |
| maintenance_reason     | None                                                                                |
| management_interface   | ipmitool                                                                            |
| name                   | osp16-compute-01                                                                    |
| network_interface      | flat                                                                                |
| owner                  | None                                                                                |
| power_interface        | ipmitool                                                                            |
| power_state            | power off                                                                           |
| properties             | {'vendor': 'unknown', 'local_gb': '59', 'cpus': '12', 'cpu_arch': 'x86_64',         |
|                        | 'memory_mb': '54000', 'capabilities': 'node:computerHCI-0,profile:ComputeHCI,boot_o |
|                        | ption:local,cpu_vt:true,cpu_aes:true,cpu_hugepages:true,cpu_hugepages_1g:true',     |
|                        | 'root_device': {'by_path': '/dev/disk/by-path/pci-0000:08:00.0'}}                   |
| protected              | False                                                                               |
| protected_reason       | None                                                                                |
| provision_state        | available                                                                           |
| provision_updated_at   | 2022-06-26T03:50:08+00:00                                                           |
| raid_config            | {}                                                                                  |
| raid_interface         | no-raid                                                                             |
| rescue_interface       | agent                                                                               |
| reservation            | None                                                                                |
| resource_class         | baremetal.HCI                                                                       |
| storage_interface      | noop                                                                                |
| target_power_state     | None                                                                                |
| target_provision_state | None                                                                                |
| target_raid_config     | {}                                                                                  |
| traits                 | []                                                                                  |
| updated_at             | 2022-06-26T07:28:38+00:00                                                           |
| uuid                   | 775df8d1-6320-47f8-86cb-b04c95fca766                                                |
| vendor_interface       | ipmitool                                                                            |
+------------------------+-------------------------------------------------------------------------------------+

$ openstack flavor show computeHCI --fit-width
+----------------------------+---------------------------------------------------------------------------------+
| Field                      | Value                                                                           |
+----------------------------+---------------------------------------------------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                                                           |
| OS-FLV-EXT-DATA:ephemeral  | 0                                                                               |
| access_project_ids         | None                                                                            |
| disk                       | 60                                                                              |
| id                         | 9a11df9b-afa3-447a-9882-ea3a428c4752                                            |
| name                       | computeHCI                                                                      |
| os-flavor-access:is_public | True                                                                            |
| properties                 | resources:CUSTOM_BAREMETAL_HCI='1', resources:DISK_GB='0',                      |
|                            | resources:MEMORY_MB='0', resources:VCPU='0'                                     |
| ram                        | 54000                                                                           |
| rxtx_factor                | 1.0                                                                             |
| swap                       |                                                                                 |
| vcpus                      | 12                                                                              |
+----------------------------+---------------------------------------------------------------------------------+

$ openstack baremetal node show osp16-control-01 --fit-width
+------------------------+-------------------------------------------------------------------------------------+
| Field                  | Value                                                                               |
+------------------------+-------------------------------------------------------------------------------------+
| allocation_uuid        | None                                                                                |
| automated_clean        | None                                                                                |
| bios_interface         | no-bios                                                                             |
| boot_interface         | ipxe                                                                                |
| chassis_uuid           | None                                                                                |
| clean_step             | {}                                                                                  |
| conductor              | osp16-director-01.example.com                                                       |
| conductor_group        |                                                                                     |
| console_enabled        | False                                                                               |
| console_interface      | ipmitool-socat                                                                      |
| created_at             | 2022-06-26T03:44:43+00:00                                                           |
| deploy_interface       | iscsi                                                                               |
| deploy_step            | {}                                                                                  |
| description            | None                                                                                |
| driver                 | ipmi                                                                                |
| driver_info            | {'deploy_kernel': 'file:///var/lib/ironic/httpboot/agent.kernel', 'rescue_kernel':  |
|                        | 'file:///var/lib/ironic/httpboot/agent.kernel', 'deploy_ramdisk':                   |
|                        | 'file:///var/lib/ironic/httpboot/agent.ramdisk', 'rescue_ramdisk':                  |
|                        | 'file:///var/lib/ironic/httpboot/agent.ramdisk', 'ipmi_address': '127.0.0.1',       |
|                        | 'ipmi_port': '6450', 'ipmi_password': '******', 'ipmi_username': 'admin'}           |
| driver_internal_info   | {'agent_erase_devices_iterations': 1, 'agent_erase_devices_zeroize': True,          |
|                        | 'agent_continue_if_ata_erase_failed': False, 'agent_enable_ata_secure_erase': True, |
|                        | 'disk_erasure_concurrency': 1, 'last_power_state_change':                           |
|                        | '2022-06-26T03:49:39.645067', 'agent_version': '5.0.5.dev27',                       |
|                        | 'agent_last_heartbeat': '2022-06-26T03:49:39.155557', 'hardware_manager_version':   |
|                        | {'generic_hardware_manager': '1.1'}, 'agent_cached_clean_steps': {'deploy':         |
|                        | [{'step': 'erase_devices', 'priority': 10, 'interface': 'deploy',                   |
|                        | 'reboot_requested': False, 'abortable': True}, {'step': 'erase_devices_metadata',   |
|                        | 'priority': 99, 'interface': 'deploy', 'reboot_requested': False, 'abortable':      |
|                        | True}], 'raid': [{'step': 'delete_configuration', 'priority': 0, 'interface':       |
|                        | 'raid', 'reboot_requested': False, 'abortable': True}, {'step':                     |
|                        | 'create_configuration', 'priority': 0, 'interface': 'raid', 'reboot_requested':     |
|                        | False, 'abortable': True}]}, 'agent_cached_clean_steps_refreshed': '2022-06-26      |
|                        | 03:49:37.862473', 'clean_steps': None}                                              |
| extra                  | {}                                                                                  |
| fault                  | None                                                                                |
| inspect_interface      | inspector                                                                           |
| inspection_finished_at | None                                                                                |
| inspection_started_at  | None                                                                                |
| instance_info          | {}                                                                                  |
| instance_uuid          | None                                                                                |
| last_error             | None                                                                                |
| maintenance            | False                                                                               |
| maintenance_reason     | None                                                                                |
| management_interface   | ipmitool                                                                            |
| name                   | osp16-control-01                                                                    |
| network_interface      | flat                                                                                |
| owner                  | None                                                                                |
| power_interface        | ipmitool                                                                            |
| power_state            | power off                                                                           |
| properties             | {'vendor': 'unknown', 'local_gb': '59', 'cpus': '4', 'cpu_arch': 'x86_64',          |
|                        | 'memory_mb': '16000', 'capabilities': 'node:controller-0,profile:control,boot_optio |
|                        | n:local,cpu_vt:true,cpu_aes:true,cpu_hugepages:true,cpu_hugepages_1g:true'}         |
| protected              | False                                                                               |
| protected_reason       | None                                                                                |
| provision_state        | available                                                                           |
| provision_updated_at   | 2022-06-26T03:49:52+00:00                                                           |
| raid_config            | {}                                                                                  |
| raid_interface         | no-raid                                                                             |
| rescue_interface       | agent                                                                               |
| reservation            | None                                                                                |
| resource_class         | baremetal                                                                           |
| storage_interface      | noop                                                                                |
| target_power_state     | None                                                                                |
| target_provision_state | None                                                                                |
| target_raid_config     | {}                                                                                  |
| traits                 | []                                                                                  |
| updated_at             | 2022-06-26T04:41:18+00:00                                                           |
| uuid                   | 7934360c-ae2e-4422-9658-15e2700f7fb2                                                |
| vendor_interface       | ipmitool                                                                            |
+------------------------+-------------------------------------------------------------------------------------+

$ openstack flavor show control --fit-width
+----------------------------+---------------------------------------------------------------------------------+
| Field                      | Value                                                                           |
+----------------------------+---------------------------------------------------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                                                           |
| OS-FLV-EXT-DATA:ephemeral  | 0                                                                               |
| access_project_ids         | None                                                                            |
| disk                       | 40                                                                              |
| id                         | 5b34996f-f6f8-45b4-ba48-8174442e1de7                                            |
| name                       | control                                                                         |
| os-flavor-access:is_public | True                                                                            |
| properties                 | capabilities:profile='control', resources:CUSTOM_BAREMETAL='1',                 |
|                            | resources:DISK_GB='0', resources:MEMORY_MB='0', resources:VCPU='0'              |
| ram                        | 4096                                                                            |
| rxtx_factor                | 1.0                                                                             |
| swap                       |                                                                                 |
| vcpus                      | 1                                                                               |
+----------------------------+---------------------------------------------------------------------------------+

```

## 1.3. Configuring resource isolation on hyperconverged nodes

https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/16.2/html-single/hyperconverged_infrastructure_guide/index

```
$ vi /home/stack/templates/storage-container-config.yaml
parameter_defaults:
  CephAnsibleExtraConfig:
    is_hci: true

```

# Customize Roles and Network Data

Create network_data.yaml based on /usr/share/openstack-tripleo-heat-templates/network_data.yaml

- /home/stack/templates/environments/05_network_data.yaml

```
$ ll /home/stack/templates/environments/05_network_data.yaml
-rw-r--r--. 1 stack stack 3447 Jun 26 18:05 /home/stack/templates/environments/05_network_data.yaml

$ cat /home/stack/templates/environments/05_network_data.yaml | grep -v "^\s*#"
- name: Storage
  vip: true
  vlan: 301
  name_lower: storage
  ip_subnet: '172.17.3.0/24'
  allocation_pools: [{'start': '172.17.3.40', 'end': '172.17.3.100'}]
  ipv6_subnet: 'fd00:fd00:fd00:3000::/64'
  ipv6_allocation_pools: [{'start': 'fd00:fd00:fd00:3000::10', 'end': 'fd00:fd00:fd00:3000:ffff:ffff:ffff:fffe'}]
  mtu: 1500
- name: StorageMgmt
  name_lower: storage_mgmt
  vip: true
  vlan: 401
  ip_subnet: '172.17.4.0/24'
  allocation_pools: [{'start': '172.17.4.40', 'end': '172.17.4.100'}]
  ipv6_subnet: 'fd00:fd00:fd00:4000::/64'
  ipv6_allocation_pools: [{'start': 'fd00:fd00:fd00:4000::10', 'end': 'fd00:fd00:fd00:4000:ffff:ffff:ffff:fffe'}]
  mtu: 1500
- name: InternalApi
  name_lower: internal_api
  vip: true
  vlan: 101
  ip_subnet: '172.17.1.0/24'
  allocation_pools: [{'start': '172.17.1.40', 'end': '172.17.1.100'}]
  ipv6_subnet: 'fd00:fd00:fd00:2000::/64'
  ipv6_allocation_pools: [{'start': 'fd00:fd00:fd00:2000::10', 'end': 'fd00:fd00:fd00:2000:ffff:ffff:ffff:fffe'}]
  mtu: 1500
- name: Tenant
  vip: false  # Tenant network does not use VIPs
  name_lower: tenant
  vlan: 201
  ip_subnet: '172.17.2.0/24'
  allocation_pools: [{'start': '172.17.2.40', 'end': '172.17.2.100'}]
  ipv6_subnet: 'fd00:fd00:fd00:5000::/64'
  ipv6_allocation_pools: [{'start': 'fd00:fd00:fd00:5000::10', 'end': 'fd00:fd00:fd00:5000:ffff:ffff:ffff:fffe'}]
  mtu: 1500
- name: External
  vip: true
  name_lower: external
  vlan: 10
  ip_subnet: '192.168.122.0/24'
  allocation_pools: [{'start': '192.168.122.40', 'end': '192.168.122.100'}]
  gateway_ip: '192.168.122.1'
  ipv6_subnet: '2001:db8:fd00:1000::/64'
  ipv6_allocation_pools: [{'start': '2001:db8:fd00:1000::10', 'end': '2001:db8:fd00:1000:ffff:ffff:ffff:fffe'}]
  gateway_ipv6: '2001:db8:fd00:1000::1'
  mtu: 1500
- name: Management
  enabled: true
  vip: false  # Management network does not use VIPs
  name_lower: management
  vlan: 60
  ip_subnet: '10.0.1.0/24'
  allocation_pools: [{'start': '10.0.1.40', 'end': '10.0.1.100'}]
  ipv6_subnet: 'fd00:fd00:fd00:6000::/64'
  ipv6_allocation_pools: [{'start': 'fd00:fd00:fd00:6000::10', 'end': 'fd00:fd00:fd00:6000:ffff:ffff:ffff:fffe'}]
  mtu: 1500

  TimeZone: 'Singapore'
  NeutronBridgeMappings: 'datacentre:br-ex'
  NeutronFlatNetworks: 'datacentre'

$ cat ~/templates/roles_data_hci.yaml 
###############################################################################
# File generated by TripleO
###############################################################################
###############################################################################
# Role: Controller                                                            #
###############################################################################
- name: Controller
  description: |
    Controller role that has all the controler services loaded and handles
    Database, Messaging and Network functions.
  CountDefault: 1
  tags:
    - primary
    - controller
    # Create external Neutron bridge for SNAT (and floating IPs when using
    # ML2/OVS without DVR)
    - external_bridge
  networks:
    External:
      subnet: external_subnet
    InternalApi:
      subnet: internal_api_subnet
    Storage:
      subnet: storage_subnet
    StorageMgmt:
      subnet: storage_mgmt_subnet
    Tenant:
      subnet: tenant_subnet
. . .
###############################################################################
# Role: ComputeHCI                                                            #
###############################################################################
- name: ComputeHCI
  description: |
    Compute Node role hosting Ceph OSD too
  networks:
    InternalApi:
      subnet: internal_api_subnet
    Tenant:
      subnet: tenant_subnet
    Storage:
      subnet: storage_subnet
    StorageMgmt:
      subnet: storage_mgmt_subnet
. . .

```

# Generate Environment Files

```
$ mkdir ~/workplace

$ mkdir ~/output

$ cp -rp /usr/share/openstack-tripleo-heat-templates/* workplace

$ cd workplace

$ tools/process-templates.py -r  ~/templates/roles_data_hci.yaml  -n  ~/templates/environments/05_network_data.yaml  -o ~/output

$ cd ../output

$  cat environments/network-environment.yaml | grep -v "^\s*#" | uniq 
resource_registry:
  OS::TripleO::Controller::Net::SoftwareConfig:
    ../network/config/single-nic-vlans/controller.yaml
  OS::TripleO::ComputeHCI::Net::SoftwareConfig:
    ../network/config/single-nic-vlans/computehci.yaml

parameter_defaults:

  StorageNetCidr: '172.17.3.0/24'
  StorageAllocationPools: [{'start': '172.17.3.40', 'end': '172.17.3.100'}]
  StorageNetworkVlanID: 301

  StorageMgmtNetCidr: '172.17.4.0/24'
  StorageMgmtAllocationPools: [{'start': '172.17.4.40', 'end': '172.17.4.100'}]
  StorageMgmtNetworkVlanID: 401

  InternalApiNetCidr: '172.17.1.0/24'
  InternalApiAllocationPools: [{'start': '172.17.1.40', 'end': '172.17.1.100'}]
  InternalApiNetworkVlanID: 101

  TenantNetCidr: '172.17.2.0/24'
  TenantAllocationPools: [{'start': '172.17.2.40', 'end': '172.17.2.100'}]
  TenantNetworkVlanID: 201
  TenantNetPhysnetMtu: 1500

  ExternalNetCidr: '192.168.122.0/24'
  ExternalAllocationPools: [{'start': '192.168.122.40', 'end': '192.168.122.100'}]
  ExternalInterfaceDefaultRoute: '192.168.122.1'
  ExternalNetworkVlanID: 10

  ManagementNetCidr: '10.0.1.0/24'
  ManagementAllocationPools: [{'start': '10.0.1.40', 'end': '10.0.1.100'}]
  ManagementNetworkVlanID: 60

  DnsServers: []
  NeutronNetworkType: 'geneve,vlan'
  NeutronNetworkVLANRanges: 'datacentre:1:1000'
  BondInterfaceOvsOptions: "bond_mode=active-backup"

$ cp environments/network-environment.yaml /home/stack/templates/environments/.

# modify /home/stack/templates/environments/network-environment.yaml

$ cat /home/stack/templates/environments/network-environment.yaml | grep -v "^\s*#" | uniq
resource_registry:
  OS::TripleO::Controller::Net::SoftwareConfig:
    ../network/config/bond-with-vlans/controller.yaml
  OS::TripleO::ComputeHCI::Net::SoftwareConfig:
    ../network/config/bond-with-vlans/computehci.yaml

parameter_defaults:
. . .
  CloudName: osp16-1-clouter01.example.com

  TimeZone: 'Singapore'
  NetworkDeploymentActions: ['CREATE','UPDATE']

  NeutronPluginExtensions: "dns"
  NeutronDnsDomain: "example.com"

```

# create Flavor and set profile

> There is different in OSP16.2

```
$ openstack flavor create --id auto --ram 4096 --disk 40 --vcpus 1 ComputeHCI

$ openstack flavor set --property capabilities:profile='ComputeHCI' --property resources:CUSTOM_BAREMETAL=1 --property resources:VCPU=0 --property resources:MEMORY_MB=0  --property resources:DISK_GB=0 ComputeHCI

$ openstack flavor show ComputeHCI --fit-width
+----------------------------+--------------------------------------------------------------+
| Field                      | Value                                                        |
+----------------------------+--------------------------------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                                        |
| OS-FLV-EXT-DATA:ephemeral  | 0                                                            |
| access_project_ids         | None                                                         |
| disk                       | 40                                                           |
| id                         | 14f911ab-4b53-4dd9-9f95-5ec72b3de0d3                         |
| name                       | ComputeHCI                                                   |
| os-flavor-access:is_public | True                                                         |
| properties                 | capabilities:profile='ComputeHCI',                           |
|                            | resources:CUSTOM_BAREMETAL='1', resources:DISK_GB='0',       |
|                            | resources:MEMORY_MB='0', resources:VCPU='0'                  |
| ram                        | 4096                                                         |
| rxtx_factor                | 1.0                                                          |
| swap                       |                                                              |
| vcpus                      | 1                                                            |

+----------------------------+--------------------------------------------------------------+

$ openstack baremetal node set osp16-compute-01 --property capabilities='node:computehci-0,profile:ComputeHCI,boot_option:local,cpu_vt:true,cpu_aes:true,cpu_hugepages:true,cpu_hugepages_1g:true'

$ openstack baremetal introspection data save osp16-compute-01 | jq ".inventory.disks"

$ openstack baremetal node set --property root_device='{"by_path": "/dev/disk/by-path/pci-0000:08:00.0"}' osp16-compute-01

$ openstack baremetal node show osp16-compute-01 -c properties -f value 
{'vendor': 'unknown', 'local_gb': '59', 'cpus': '12', 'cpu_arch': 'x86_64', 'memory_mb': '54000', 'capabilities': 'node:computehci-0,profile:ComputeHCI,boot_option:local,cpu_vt:true,cpu_aes:true,cpu_hugepages:true,cpu_hugepages_1g:true', 'root_device': {'by_path': '/dev/disk/by-path/pci-0000:08:00.0'}}

$ openstack baremetal node show osp16-compute-01 -c resource_class -f value 
baremetal

```
