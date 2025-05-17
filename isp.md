# Глава 1 

## Настройка isp 
> 1.nmtui
> 2.nftables 
{.is-success}

## Настройка nmtui
~~~
172.16.4.1/28 1-hq-srv
172.16.5.1/28 2-br-srv
hostnamectl set-hostname isp.au-team.irpo && bash
~~~
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
iifname ens256 oifname ens192 counter masquerade 
~~~
`vim или vi /etc/sysconfig/nftables.conf`

```
systemctl restart nftables 
systemctl enable nftables 
```







