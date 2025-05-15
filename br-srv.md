# BR-SRV

### Интерфейс ens192 
```
Адрес -> 192.168.4.2/27
Шлюз -> 192.168.4.1
Днс -> 192.168.1.2
```
### Имя вм
```
br-srv.au-team.irpo
```
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
## Создание пользователя
```
useradd sshuser
passwd sshuser
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
