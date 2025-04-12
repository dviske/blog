Title: Deploying VLAN provider networks with Kolla-Ansible
Date: 2025-04-12 15:00:00
Category: OpenStack
Tags: openstack, homelab, neutron, kolla

By default, Kolla deploys a flat physical network called `physnet1`. While this works if you're happy to put all your customers in one massive VLAN, you might occasionally need to allow these customers to use existing VLANs.

Attempting to add a VLAN-based provider network to physnet1 will result in an error because it won't be allowed by the Neutron configuration.

To get around this, we need to add an override in ml2_conf.ini:

In `/etc/kolla/config/neutron/ml2_conf.ini`, add the following:
```ini
[ml2_type_vlan]
network_vlan_ranges = physnet1:1:4094
```

Then apply this configuration with `kolla-ansible -i <your inventory>.yml reconfigure -t neutron`

Once the config has applied, you will be able to create VLAN networks tied to physnet1.

```sh
openstack network create \
  --share \
  --provider-physical-network physnet1 \
  --provider-network-type vlan \
  --provider-segment 21 \
  vlan-21
openstack subnet create \
  --network vlan-21 \
  --ip-version 4 \
  --cidr 172.31.0.0/24 \
  --gateway_ip 172.13.0.1 \
  --allocation_pools start=172.31.0.2,end=172.16.0.254 \
  --dns-nameserver 1.1.1.1,1.0.0.1 \
  vlan-21-subnet-1
```

In my testing, my physnet1 port was on a switchport that had both tagged and untagged VLANs. I was still able to use the untagged VLAN on the flat network, while also using tagged VLANs on the newly-made provider networks, however it may be preferably to have a separate physnet for untagged traffic.