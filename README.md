# Домашнее задание:
Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client.
Настроить удаленный бекап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:
-    директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования. В данном случае для демонстрации размер не принципиален, достаточно будет и 2GB;
-    репозиторий для резервных копий должен быть зашифрован ключом или паролем - на ваше усмотрение;
-    имя бекапа должно содержать информацию о времени снятия бекапа;
-    глубина бекапа должна быть год, хранить можно по последней копии на конец месяца, кроме последних трех. Последние три месяца должны содержать копии на каждый день. Т.е. должна быть правильно настроена политика удаления старых бэкапов;
-    резервная копия снимается каждые 5 минут. Такой частый запуск в целях демонстрации;
-    написан скрипт для снятия резервных копий. Скрипт запускается из соответствующей Cron джобы, либо systemd timer-а - на ваше усмотрение;
-    настроено логирование процесса бекапа. Для упрощения можно весь вывод перенаправлять в logger с соответствующим тегом. Если настроите не в syslog, то обязательна ротация логов.
-    Запустите стенд на 30 минут. Убедитесь что резервные копии снимаются.
-    Остановите бекап, удалите (или переместите) директорию /etc и восстановите ее из бекапа.
    Для сдачи домашнего задания ожидаем настроенные стенд, логи процесса бэкапа и описание процесса восстановления.


# Выполнение

После запуска `Vagrantfile` создаются две ВМ: `backup` и `client` с уже настроенным автоматическим бэкапом (для наглядности бэкап каждую минуту).

Заходим на `client` и проверяем, работатет ли таймер:
```bash
[root@client ~]# systemctl list-timers 
NEXT                         LEFT       LAST                         PASSED  UNIT                         ACTIVATES
Fri 2023-03-03 16:32:07 MSK  21s left   Fri 2023-03-03 16:31:07 MSK  38s ago borg-backup.timer            borg-backup.service
...

2 timers listed.
```

Логирование настроено через `rsyslog`, логи сохраняются в `/var/log/borg.log`:
```bash
[root@client ~]# tail -f /var/log/borg.log 
Mar  3 16:33:12 client borg: Duration: 1.08 seconds
Mar  3 16:33:12 client borg: Number of files: 1701
Mar  3 16:33:12 client borg: Utilization of max. archive size: 0%
Mar  3 16:33:12 client borg: ------------------------------------------------------------------------------
Mar  3 16:33:12 client borg: Original size      Compressed size    Deduplicated size
Mar  3 16:33:12 client borg: This archive:               28.43 MB             13.50 MB             38.23 kB
Mar  3 16:33:12 client borg: All archives:               56.85 MB             26.99 MB             11.88 MB
Mar  3 16:33:12 client borg: Unique chunks         Total chunks
Mar  3 16:33:12 client borg: Chunk index:                    1286                 3400
Mar  3 16:33:12 client borg: ------------------------------------------------------------------------------
Mar  3 16:35:02 client borg: ------------------------------------------------------------------------------
Mar  3 16:35:02 client borg: Archive name: etc-2023-03-03_16:34
Mar  3 16:35:02 client borg: Archive fingerprint: 68ded700fad396b969454c2befe36453f1284c65410a5dc3b08efa1559dcf89e
Mar  3 16:35:02 client borg: Time (start): Fri, 2023-03-03 16:34:59
Mar  3 16:35:02 client borg: Time (end):   Fri, 2023-03-03 16:35:00
Mar  3 16:35:02 client borg: Duration: 0.61 seconds
Mar  3 16:35:02 client borg: Number of files: 1701
Mar  3 16:35:02 client borg: Utilization of max. archive size: 0%
Mar  3 16:35:02 client borg: ------------------------------------------------------------------------------
Mar  3 16:35:02 client borg: Original size      Compressed size    Deduplicated size
Mar  3 16:35:02 client borg: This archive:               28.43 MB             13.50 MB                681 B
Mar  3 16:35:02 client borg: All archives:               56.85 MB             26.99 MB             11.84 MB
Mar  3 16:35:02 client borg: Unique chunks         Total chunks
Mar  3 16:35:02 client borg: Chunk index:                    1285                 3400
Mar  3 16:35:02 client borg: ------------------------------------------------------------------------------
```

Остановим бэкап, удалим директорию `/etc` и восстановим ее из бэкапа:
```bash
[root@client ~]# borg extract borg@192.168.56.10:/var/borg/backup/::etc-2023-03-03_16:37 etc/
Enter passphrase for key ssh://borg@192.168.56.10/var/borg/backup: 
[root@client ~]# ls
anaconda-ks.cfg  etc  original-ks.cfg
[root@client ~]# ls -l etc/ | wc -l
181
[root@client ~]# ls -l /etc | wc -l
181
[root@client ~]# rm -rf /etc
rm: cannot remove '/etc': Device or resource busy
[root@client ~]# ls -l /etc | wc -l
1
[root@client ~]# cp -Rf etc/* /etc/
[root@client ~]# ls -l /etc | wc -l
181
[root@client ~]# 
```
