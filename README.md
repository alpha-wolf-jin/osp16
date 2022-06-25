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
