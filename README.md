# osp16

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

```

## 4.10.1. Single CPU architecture overcloud images
Your Red Hat OpenStack Platform (RHOSP) installation includes packages that provide you with the following overcloud images for director:

- overcloud-full
- overcloud-full-initrd
- overcloud-full-vmlinuz


```
[root@osp16-director-01 ~]# mkdir -p /backup//var/lib/ironic/httpboot
[root@osp16-director-01 ~]# cp -rp /var/lib/ironic/httpboot/* /backup//var/lib/ironic/httpboot/.
[root@osp16-director-01 ~]# ll /var/lib/ironic/httpboot/ /backup//var/lib/ironic/httpboot
/backup//var/lib/ironic/httpboot:
total 558004
-rwxr-xr-x. 1 42422 42422   8917856 Aug  8  2020 agent.kernel
-rw-r--r--. 1 42422 42422 562464013 Aug  8  2020 agent.ramdisk
-rw-r--r--. 1 42422 42422       758 Aug  8  2020 boot.ipxe
-rw-r--r--. 1 42422 42422       371 Aug  8  2020 inspector.ipxe
drwxr-xr-x. 2 42422 42422         6 Nov 23  2020 pxelinux.cfg

/var/lib/ironic/httpboot/:
total 558004
-rwxr-xr-x. 2 42422 42422   8917856 Aug  8  2020 agent.kernel
-rw-r--r--. 2 42422 42422 562464013 Aug  8  2020 agent.ramdisk
-rw-r--r--. 1 42422 42422       758 Aug  8  2020 boot.ipxe
-rw-r--r--. 1 42422 42422       371 Aug  8  2020 inspector.ipxe
drwxr-xr-x. 2 42422 42422         6 Nov 23  2020 pxelinux.cfg
[root@osp16-director-01 ~]# rm -rf /var/lib/ironic/httpboot/*
[root@osp16-director-01 ~]# ll /var/lib/ironic/httpboot/
total 0

$ sudo dnf remove rhosp-director-images rhosp-director-images-ipa-x86_64

```


```
$ source ~/stackrc

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
+--------------------------------------+------------------------+--------+
| ID                                   | Name                   | Status |
+--------------------------------------+------------------------+--------+
| 2f92967a-acc9-4225-9768-3ff102c9deed | overcloud-full         | active |
| fc98b79a-6a07-4f9b-a727-9b5e0971cc25 | overcloud-full-initrd  | active |
| 65de3ff5-0202-4c2d-9136-2166a7c6daf1 | overcloud-full-vmlinuz | active |
+--------------------------------------+------------------------+--------+

$ openstack image delete overcloud-full overcloud-full-initrd overcloud-full-vmlinuz

$ openstack overcloud image upload --image-path /home/stack/images/

$ openstack image list
+--------------------------------------+------------------------+--------+
| ID                                   | Name                   | Status |
+--------------------------------------+------------------------+--------+
| 03974049-5f68-4405-9e7d-f760403ea6e6 | overcloud-full         | active |
| 6868da36-e9ba-4c9c-aef4-e2ba1c6486e9 | overcloud-full-initrd  | active |
| c90ed7b6-f1d0-48db-8efe-9d5e4eeae063 | overcloud-full-vmlinuz | active |
+--------------------------------------+------------------------+--------+

$ ls -l /var/lib/ironic/httpboot
total 451860
-rwxr-xr-x. 1 root root  10034544 Jun 25 18:53 agent.kernel
-rw-r--r--. 1 root root 452668994 Jun 25 18:53 agent.ramdisk


```
