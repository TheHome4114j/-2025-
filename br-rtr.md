# BR-RTR

### Интерфейс ens192 
```
Адрес -> 172.16.5.2/28
Шлюз -> 172.16.5.1
```
### Интерфейс ens224

```
Адрес -> 192.168.4.1/27
Днс -> 192.168.1.2
```
### Имя вм
```
br-rtr.au-team.irpo
```
| Имя ВМ   | Интерфейс  |Адрес       | Маска         |Шлюз       | ДНС         |
|----------|------------|------------|---------------|-----------|-------------|
| br-rtr   | ens192     |172.16.5.2  |255.255.255.240| 172.16.5.1|      -      |
|          | ens224     |192.168.4.1 |255.255.255.224|     -     | 192.168.1.2 |
|          |   tun2     |192.168.10.2|255.255.255.252|     -     |      -      | 
| br-srv   | ens192     |192.168.4.2 |255.255.255.224|192.168.4.1| 192.168.1.2 |



## Настройка linux на разрешение трафику идти мимо вашей ВМ
~~~
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf && sysctl -p
~~~
##  Выключаем selinux
#### 1 Способ
```
sed -i "s/enforcing/disabled/g" /etc/selinux/config && setenforce 0
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
iifname ens224 oifname ens192 counter masquerade
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
echo "net_admin ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```
Пароль: P@$$word

## Настройка Tunnel
`vi или vim ifcfg-tun1` и заполняем
```
NAME=tun1
DEVICE=tun1
ONBOOT=yes
STARTMODE=onboot
BOOTPROTO=none
TYPE=GRE
MY_INNER_IPADDR=192.168.10.2/30
MY_OUTER_IPADDR=172.16.5.2
PEER_OUTER_IPADDR=172.16.4.2
ZONE=trusted
TTL=30
MTU=1400
```
```
echo "NAME=tun1" >> /etc/sysconfig/ifcfg-tun1" && echo "DEVICE=tun1" >> /etc/sysconfig/ifcfg-tun1" && echo "ONBOOT=yes" >> /etc/sysconfig/ifcfg-tun1" && echo "STARTMODE=onboot" >> /etc/sysconfig/ifcfg-tun1" && echo "BOOTPROTO=none" >> /etc/sysconfig/ifcfg-tun1" && echo "TYPE=GRE" >> /etc/sysconfig/ifcfg-tun1" && echo "MY_INNER_IPADDR=192.168.10.2/30" >> /etc/sysconfig/ifcfg-tun1" && echo "MY_OUTER_IPADDR=172.16.5.2" >> /etc/sysconfig/ifcfg-tun1" && echo "PEER_OUTER_IPADDR=172.16.4.2" >> /etc/sysconfig/ifcfg-tun1" && echo "ZONE=trusted" >> /etc/sysconfig/ifcfg-tun1" && echo "TTL=30"" >> /etc/sysconfig/ifcfg-tun1" && echo "MTU=1400" >> /etc/sysconfig/ifcfg-tun1" 
```

После чего надо будет перезагрузить и добавить в автозагрузку 

```
systemctl daemon-reload
systemctl restart network
systemctl enable network 

```
## Настройка frr
>Перейдем в файл `vi или vim /etc/frr/daemons` заменяем строку `osfpd=no`  на `osfpd=yes`
```
systemctl	restart frr
vtysh
conf t
router ospf
network 192.168.10.0/30 area 0
network 192.168.4.0/27 area 0
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


