# DEMO2024-GDZ

## Порядок выполнения для милых и пушистых

- Маршрутизация + имена
- Туннель 
- OSPF для туннеля
- DHCP
- Пользователи
- iperf
- ssh + firewalld(hq-r)
- backup
- cups
- ntp
- raid5
- ClamAV
- Настройте систему управления трафиком на роутере (BR-R)
- DNS
- Samba Домен
- LMS Apache


## SSHка + firewall

### Настройка SSH

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/6933a23b-553f-4d1a-98d1-f749ae30b05c)

Чуть ниже запрещаем пустые пароли

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/0ecb24e5-2d39-40ab-998a-77ce65d700d8)

Еще чуть ниже указываем путь к файлу внутри которого пишем че там по заданию надо
Сам файл создать не забудь 

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/4ccdc6d0-9c36-4b08-bddc-04e76fd93ca6)

Настройте контроль доступа до HQ-SRV по SSH со всех устройств, кроме CLI.

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/4603b6d3-0e2b-4592-9bd8-a4080deb9600)

### Настройка перенаправления порта через Firewalld

```
apt install firewalld
firewall-cmd --set-default-zone=trusted
firewall-cmd --add-forward-port=port=22:proto=tcp:toport=*новый_порт_ssh*:toaddr=*айпи_HQ-SRV*
firewall-cmd --runtime-to-permanent
```

## Backup script

mkdir /opt/backup

cd /opt/backup

nano backup.sh

```
#!/bin/bash

src="/home /etc"
dist="/opt/backup"

date=$(date +%A-%F)
hostname=$(hostname -s)
backup_name="$hostname-$date.tgz"

tar czf $dist/$backup_name $src
```

chmod +x backup.sh

./backup.sh

## CUPs PDF

```
apt install cups-pdf
lpstat -p -d
```

Вывод скринь. Ну да это все задание а вы что думали

## NTP

```
apt install chrony
```

### На сервачке HQ-R

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/4fc469a2-2b91-4d6c-bbbb-c5c8c985c829)

```
timedatectl set-timezone Europe/Moscow
```

### На хостах

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/a92c7eae-2566-4828-9f29-5bda226224b6)

Заменить на вашу айпишку сервака HQ-R

## RAID5

```
apt install mdadm
mdadm --create --verbose /dev/md/md0 --level=5 –raid-device=3 /dev/диск1 /dev/диск2 /dev/диск3
mkfs -t ext4 /dev/md
```

--verbose – имя диска

Скриним UUID

```
nano /etc/fstab
UUID=твой_айди	/media/raid	ext4	defaults,noatime,nofail 0	0
```

Папку в медиа создать руками

```
systemctl daemon-reload
```

df -h	-	должно показать raid в списке

## Реализуйте антивирусную защиту по средствам ClamAV на устройствах HQ-SRV и BR-SRV

```
crontab -e
```

Выбираем редактор nano мыж не дауны какие то

Пишем в файлик:

```
0 0 1 * * root /usr/bin/clamscan -ri /
```

## Настройте систему управления трафиком на роутере BR-R для контроля входящего трафика в соответствии со следующими правилами

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/01eee4ec-5398-477f-807e-87d4d2d6bf13)

## DNS

```
apt install bind9
cd /etc/bind
mkdir zones
cp zones/db.local zones/hq.work
cp zones/db.local zones/br.work
cp zones/db127.local zones/1.168.192.hq.work
cp zones/db127.local zones/1.168.172.br.work
nano named.conf.local
```

Объявляем зоны и пути к ним

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/7184a411-999d-47b5-8d71-aa8dce9176c3)

Прямая зона hq.work

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/cb1ffe94-eb03-4243-bd84-ff20927d6c6e)

Обратная зона hq.work

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/f9748e45-96ae-40be-95e3-9e369cdf63e3)

Прямая зона branch.work

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/16fe8d4b-8a1e-4e4b-afbb-0ab8a0e8d124)

Обратная зона branch.work

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/f061f30c-8df8-428b-af2c-82413276d0b6)

Конфигурация named.conf.options

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/cff09202-b48e-45ec-868b-8355da7af2f0)

В resolv.conf

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/c281cf44-7f1a-404d-82ad-316f24bfbb1f)

Прописываем в nmtui DNS сервер и поиск домена если надо

## SambaDC

```
apt install samba*
nano /etc/hosts
```

Приводим к такому виду файл hosts

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/84c2f6d5-44a7-4925-836a-06fb63156943)


```
rm /etc/samba/smb.conf
samba-tool domain provision
```

На клиентах

```
apt install samba-client
rm /etc/samba/smb.conf
```

## LMS Apache

```
apt install php8.2 mariadb-server apache2 php-xml php-zip php-intl php-curl php-mbstring php-gd php-mysql
git clone https://github.com/moodle/moodle.git
mkdir /var/www/moodledata
chmod 777 /var/www/moodledata
```
Конфиг апача

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/814a4fbd-b2b7-4b84-a792-4d62274820b8)

Создаем пользователя

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/b94f4b72-94f1-4588-804e-400b2498e4e7)

Права на базу

![image](https://github.com/KisSsArt/DEMO2024-GDZ/assets/59938902/a859ad94-5bf7-4be9-969d-c7bfc6174541)

Дальше дрочить на CLI через веб интерфейс. Устанавливать зависимости и тд. Конфиг проще передать по ssh(scp)

