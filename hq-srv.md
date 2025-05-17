# HQ-SRV


### Интерфейс ens192 
```
Адрес -> 192.168.1.2/26
Шлюз -> 192.168.1.1
Днс -> 192.168.1.2
```
### Имя вм
```
hq-srv.au-team.irpo
```
##  Выключаем selinux
#### 1 Способ
```
sed -i "s/enforcing/disabled/g" /etc/selinux/config
setenforse 0
```
```
sed -i "s/enforcing/disabled/g" /etc/selinux/config && setenforce 0 && echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf && sysctl -p
```
#### 2 Способ 
>Зайти файл `vim или vi /etc/selinux/config`
>Изменить `enforcing` на `disabled`
>После использовать команду `setenforse 0`
## Создание пользователя
```
useradd sshuser -u 1010
passwd sshuser
echo "sshuser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```
Пароль: P@ssw0rd

## Настройка ssh
`vi /etc/ssh/sshd_config`

```
port 2024
AllowUsers sshuser
Banner /etc/ssh/banner
MaxAuthTries 2 
```
`vi /etc/ssh/banner`
```
Authorized access only!
```
```
systemctl restart ssh
systemctl enable ssh
```
## Настройка ДНС

```
dnf install -y bind
```

`vi /etc/name.conf`

```
listen-on 53 {192.168.1.2;};
listen-on-v6 port 53 {none;};
allow-query {192.168.0.0/24;};
allow-recursion {192.168.0.0/24;};
forwarders {77.88.8.8; };
dnssec-validation no;
```

```
zone "au-team.irpo" {
			type master;
      file "au.db";
      allow-update {192.168.0.0/24;};
};
```
```
zone "168.192.in-addr.arpa" {
			type master;
      file "192.db";
      allow-update {192.168.0.0/24;};
};

```

`cd /var/named/`
`cp named.localhost au.db`

![безымянный.jpg](/_4_kurs/безымянный.jpg)

`cp named.localhost 192.db`

![1.jpg](/_4_kurs/1.jpg)

`chmod 777 au.db 192.db`

```
systemctl restart named
systemctl enable named
```
