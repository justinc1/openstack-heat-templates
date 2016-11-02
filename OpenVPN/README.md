# Usage

## Spin up a server VM

Setup virtualenv, with correct openstack client side utilities.

```
mkvirtualenv mitaka-client
pip install python-openstackclient  # correct version for your openstack
pip install python-neutronclient python-heatclient
```

We assume project already has its internal network setup. So just pick it up.
Same goes for VM image and user SSH keypair.

The ```vpn_name``` parameter is used only to name VPN connection on client side, e.g. in ```systemctl start openvpn@VPN_NAME.service```.
Make it descripte and unique. The sole reason for it is to avoid overwriting existing configuration files (and certificates) on client computer.

```
workon mitaka-client
source openrc.sh  # for keystone API v3
cd openstack-heat-templates/OpenVPN
glance image-list  # ubuntu-14.04 image ID
neutron net-list  # internal and external network ID
nova keypair-list  # name of your ssh keypair
# vpn_name: is used to name VPN connection on client side, e.g. in "systemctl start openvpn@VPN_NAME.service"

heat stack-create -f openvpn.heat vpn-server -P "key_name=my_key_name;vpn_name=my-project-name;instance_type=m1.xsmall;vpn_cidr=10.8.0.0/24;image_id=UUID;public_net_id=UUID;private_net_id=UUID"
```

## Adjust client configuration template

The created client example/template configuration will auto-detect its external IP by doing HTTP request to say http://checkip.dyndns.org.
Adjust configuration template in ```/etc/openvpn/easy-rsa/client.conf``` if required - e.g. fix ```remote``` line.

Fedora (v24 at least) is missing group with name nogroup. You can comment out the group line (or use nobody group).

## Add new client

```
ssh root@IP
cd /etc/openvpn/easy-rsa/
source vars
./client-add.sh user_name user_name@example.org
```

Now send file ```vpnaccess-vpn_name-user_name.zip``` to the user.
