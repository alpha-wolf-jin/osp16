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

$ sudo dnf update -y

$ sudo reboot

```

## 3.4. Installing director packages

```
$ sudo dnf install -y python3-tripleoclient

$ sudo dnf update -y python3-tripleoclient

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

```
