# OVS
В качестве примера
```
systemctl restart openvswitch.service
cd /etc/net/ifaces
vim ens192/options
```
#### Наполняем
```
BOOTPROTO=static
TYPE=eth
DISABLED=no
```
```
mkdir ens224
cp ens192/options ens224/
```
```
vim MGMT/options
```

```
VID=VLan
BRIDGE=hq-sw
BOOTPROTO
TYPE=ovsport
DISABLED=no
```

```
vim vlan153/ipv4address
```
`_._._._/_`
```
vim vlan153/ipv4route
```
`default via _._._._`
```
systemctl restart network
```

```
ovs-vsctl add-br br0 
ovs-vsctl add-port br0 ens192 trunk=133,143,153
ovs-vsctl add-port br0 ens224 tag=133
systemctl restart network 
```
```
vim /etc/net/sysctl.conf
net.ipv4.ip_forward=1
```
