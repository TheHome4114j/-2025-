# HQ-RTR

### Интерфейс ens192 
```
Адрес -> 172.16.4.2/28
Шлюз -> 172.16.4.1
```

### Имя вм
```
hq-rtr.au-team.irpo
```


| Имя ВМ   | Интерфейс  |Адрес       | Маска         |Шлюз       | ДНС         |
|----------|------------|------------|---------------|-----------|-------------|
|  hq-rtr  | ens192     |172.16.4.2  |255.255.255.240| 172.16.4.1|      -      |
|          | vlan400 + x|192.168.1.1 |255.255.255.192|     -     | 192.168.1.2 |
|          | vlan430 + x|192.168.2.1 |255.255.255.240|     -     |      -      |
|          | vlan460 + x|192.168.3.1 |255.255.255.248|     -     |      -      | 
|          |   tun1     |192.168.10.1|255.255.255.252|     -     |      -      | 
| hq-srv   | ens192     |192.168.1.2 |255.255.255.192|192.168.1.1|192.168.1.2  |
| hq-cli   | ens192     |dhcp        |dhcp           | dhcp      |dhcp         |
| br-rtr   | ens192     |172.16.5.2  |255.255.255.240| 172.16.5.1|      -      |
|          | ens224     |192.168.4.1 |255.255.255.224|     -     | 192.168.1.2 |
|          |   tun2     |192.168.10.2|255.255.255.252|     -     |      -      | 
| br-srv   | ens192     |192.168.4.2 |255.255.255.224|192.168.4.1| 192.168.1.2 |
x - твой номер по списку
# Настройка hq-rtr

```
dnf install -y nftables frr dhcp network-scripts
dnf remove -y NetworkManager 
```

## Настройка интерфейса, vlans и gre tunnel 

`cd /etc/sysconfig/network-scripts`

>Надо будет создать 5 файлов 
>Первый файл `ifcfg-ens192`. Заходим в него `vi или vim ifcfg-ens192` и заполняем вот так 
~~~
DEVICE=ens192
ONBOOT=yes
BOOTPROTO=static
IPADDR=172.16.4.2 
NETMASK=255.255.255.240
GATEWAY=172.16.4.1
~~~
>Второй для vlan `ifcfg-vlan400+x`, где x - твой номер по списку также заходим и заполняем вот так `vi или vim ifcfg-vlan400+x`
~~~
VLAN=yes
VLAN_ID=400+x
TYPE=Ethernet
DEVICE=vlan400+x
ONBOOT=yes
PHYSDEV=ens224
IPADDR=192.168.1.1
NETMASK=255.255.255.192
DNS1=192.168.1.2
~~~
>Дальше надо будет настроить еще два vlan, что бы не писать файл заново скопируем `cp ifcfg-vlan400+x ifcfg-vlan430+x` `cp ifcfg-vlan400+x ifcfg-vlan460+x`

>`vi или vim ifcfg-vlan430+x`
```
VLAN=yes
VLAN_ID=430+x
TYPE=Ethernet
DEVICE=vlan430+x
ONBOOT=yes
PHYSDEV=ens224
IPADDR=192.168.2.1
NETMASK=255.255.255.240
```
>`vi или vim ifcfg-vlan460+x`
```
VLAN=yes
VLAN_ID=460+x
TYPE=Ethernet
DEVICE=vlan460+x
ONBOOT=yes
PHYSDEV=ens224
IPADDR=192.168.3.1
NETMASK=255.255.255.248
```
> Осталься пятый файл это тунель заходим в него `vi или vim ifcfg-tun1` и заполняем
```
NAME=tun1
DEVICE=tun1
ONBOOT=yes
STARTMODE=onboot
BOOTPROTO=none
TYPE=GRE
MY_INNER_IPADDR=192.168.10.1/30
MY_OUTER_IPADDR=172.16.4.2
PEER_OUTER_IPADDR=172.16.5.2
ZONE=trusted
TTL=30
MTU=1400
```
После чего надо будет перезагрузить и добавить в автозагрузку 

```
systemctl daemon-reload
systemctl restart network
systemctl enable network 
```
## Настройка dhcp

>Заходим в файл `vi или vim /etc/dhcp/dhcpd.conf` там будут строчки их не трогаем и снизу пишем
```
subnet 192.168.2.0 netmask 255.255.255.240 {
		range 192.168.2.1 192.168.2.14;
        option domain-name-servers 192.168.1.2;
        option domain-name "au-team.irpo";
        option routers 192.168.2.1;
        option broadcast-address 192.168.2.15;
}
```
>Также перезагрузим и добавим в автозагрузку
```
systemctl restart dhcpd
systemctl enable dhcpd
```
## Настройка frr
>Перейдем в файл `vi или vim /etc/frr/daemons` заменяем строку `osfpd=no`  на `osfpd=yes`


```
systemctl	restart frr
vtysh
conf t
router ospf
network 192.168.10.0/30 area 0
network 192.168.1.0/26 area 0
network 192.168.2.0/28 area 0
network 192.168.3.0/29 area 0
exit
interface tun1
ip ospf authentication message-digest
ip ospf authentication-key P@ssw0rd
ip ospf network broadcast 
do wr
exit
```
```
systemctl restart frr
systemctl enable frr
```
## Настройка linux на разрешение трафику идти мимо вашей ВМ
~~~
vim или vi  /etc/sysctl.conf`
net.ipv4.ip_forward=1 
sysctl -p
~~~
##  Выключаем selinux
#### 1 Способ
```
sed -i "s/enforcing/disabled/g" /etc/selinux/config
setenforse 0
```
#### 2 Способ 
>Зайти файл `vim или vi /etc/selinux/config`
>Изменить `enforcing` на `disabled`
>После использовать команду `setenforse 0`
## Настройка nftables
~~~
dnf install -y nftables
~~~
`vim или vi /etc/nftables/nat.nft`

>Смотрим ветку `chain postrouting` и в неё вписываешь две строки 
~~~
iifname vlan400+x oifname ens192 counter masquerade
iifname vlan430+x oifname ens192 counter masquerade
iifname vlan460+x oifname ens192 counter masquerade 
~~~
`vim или vi /etc/sysconfig/nftables.conf`

```
systemctl restart nftables 
systemctl enable nftables 

```
## Создание пользователя
```
useradd net_admin
passwd net_admin
```
Пароль: P@$$word















