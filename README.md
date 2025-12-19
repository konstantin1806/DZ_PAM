# DZ_PAM

поскольку задание делал в будний день, то скрипт login.sh переделан на аутентификацию в будни.
порядок выполнения:
sudo useradd otusadm && sudo useradd otus
passwd otus
passwd otusadm
sudo groupadd -f admin
sudo usermod -a -G admin otusadm 
sudo usermod -a -G admin vagrant 

root@client:~# cat /etc/group | grep admin
admin:x:117:otusadm,root,vagrant

sudo nano /usr/local/bin/login.sh:

#!/bin/bash

# Получаем день недели: 1-понедельник, 7-воскресенье
DAY=$(date +%u)

# Выходные дни (суббота=6, воскресенье=7) - разрешаем всем
if [ "$DAY" -eq 6 ] || [ "$DAY" -eq 7 ]; then
    exit 0
fi

# Будние дни - проверяем группу admin
if groups "$PAM_USER" | grep -q '\badmin\b'; then
    exit 0
else
    echo "SSH access on weekdays is only allowed for admin group" >&2
    exit 1
fi

chmod +x /usr/local/bin/login.sh

в файл /etc/pam.d/sshd добавляем  модуль pam_exec и login.sh

root@client:~# sudo pam-auth-update
root@client:~# sudo systemctl restart ssh


результат попыток подключения:

OTUS
  2025-12-19   14:31.38   /home/mobaxterm  ssh otus@192.168.56.20
(otus@192.168.56.20) Password:
/usr/local/bin/login.sh failed: exit code 1

Connection closed by 192.168.56.20 port 22
                                                                                             ✗
OTUSADM
  2025-12-19   14:32.27   /home/mobaxterm  ssh otusadm@192.168.56.20
(otusadm@192.168.56.20) Password:
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-216-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Dec 19 11:32:37 UTC 2025




