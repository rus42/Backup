# Домашнее задание к занятию "Резервное копирование" - Васёв А.В.

## Задание 1
### В чём разница между:
1. полным резервным копированием,
2. дифференциальным резервным копированием,
3. инкрементным резервным копированием.

## Задание 2
### Установите программное обеспечении Bacula, настройте bacula-dir, bacula-sd, bacula-fd. Протестируйте работу сервисов.

bacula-dir
```java
Director {
  Name = netology-dir
  DIRport = 9101
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  Password = "123"
  Messages = Standard
  DirAddress = 127.0.0.1
}
Catalog {
  Name = default
  dbname = "bacula"; DB Address = "localhost"; dbuser = "bacula"; dbpassword = "12345678"
}
Messages {
  Name = Standard
  director = netology-dir = all, !skipped, !restored
}
Console {
  Name = netology-mon
  Password = "123"
  CommandACL = status, .status
}
Job {
  Name = "LocalBackup"
  Type = Backup
  Level = Incremental
  Client = netology-fd
  FileSet = "Local"
  Schedule = "Local"
  Messages = Standard
  Pool = "LocalPool"
  Priority = 10
  Allow Duplicate Jobs = no
  Cancel Queued Duplicates = yes
  Cancel Running Duplicates = no
}
FileSet {
  Name = "Local"
  Include {
    Options {
      signature = MD5
    }
    File = /etc
  }
}
Pool {
  Catalog = default
  Name = LocalPool
  Pool Type = Backup
  Recycle = yes
  AutoPrune = yes
  Volume Retention = 30 days
  Recycle Oldest Volume = yes
  Purge Oldest Volume = yes
  Action On Purge = Truncate
  Maximum Volume Jobs = 1
  Maximum Volumes = 30
  Label Format = "Local-"
  Storage = netology-sd
}
Client {
  Name = netology-fd
  Address = localhost
  FDPort = 9102
  Catalog = default
  Password = "123"
  AutoPrune = yes
  Maximum Concurrent Jobs = 1
}
Storage {
  Name = netology-sd
  Address = localhost
  SDPort = 9103
  Password = "123"
  Device = Local-Device
  Media Type = File
  Maximum Concurrent Jobs = 1
}

Schedule {
  Name = "Local"
  Run = Level=Full daily at 10:10
}
```

bacula-sd
```java
Storage {
  Name = netology-sd
  SDPort = 9103
  WorkingDirectory = "/var/lib/bacula"
  Pid Directory = "/run/bacula"
  Plugin Directory = "/usr/lib/bacula"
  Maximum Concurrent Jobs = 20
  SDAddress = 127.0.0.1
}
Director {
  Name = netology-dir
  Password = "123"
}
Device {
  Name = Local-Device
  Media Type = File
  Archive Device = /backup
  LabelMedia = yes;
  Random Access = Yes;
  AutomaticMount = yes;
  RemovableMedia = no;
  AlwaysOpen = no;
  Maximum Concurrent Jobs = 5
}
Messages {
  Name = Standard
  director = netology-dir = all
}
```

bacula-fd
```java
Director {
  Name = netology-dir
  Password = "123"
}
FileDaemon {
  Name = netology-fd
  FDport = 9102
  WorkingDirectory = /var/lib/bacula
  Pid Directory = /run/bacula
  Maximum Concurrent Jobs = 20
  Plugin Directory = /usr/lib/bacula
  FDAddress = 127.0.0.1
}
Messages {
  Name = Standard
  director = netology-dir = all, !skipped, !restored
}
```
![alt text](https://github.com/rus42/Backup/blob/main/Task_2.png)

## Задание 3
### Установите программное обеспечении Rsync. Настройте синхронизацию на двух нодах. Протестируйте работу сервиса.

rsyncd.conf
```java
pid file = /var/run/rsyncd.pid
log file = /var/log/rsyncd.log
transfer logging = true
munge symlinks = yes

[data]
path = /data
uid = root
read only = yes
list = yes
comment = Data backup Dir
auth users = backup
secrets file = /etc/rsyncd.scrt
```

rsyncd.scrt
```java
backup:12345
```

backup-server1.sh
```java
#!/bin/bash
date
syst_dir=/backup/
srv_name=netology
srv_ip=10.0.0.2
srv_user=backup
srv_dir=data
echo "Start backup ${srv_name}"
mkdir -p ${syst_dir}${srv_name}/increment/
/usr/bin/rsync -avz --progress --delete --password-file=/etc/rsyncd.scrt ${srv_user}@${srv_ip}::${srv_dir} ${syst_dir}>
/usr/bin/find ${syst_dir}${srv_name}/increment/ -maxdepth 1 -type d -mtime +30 -exec rm -rf {} \;
date
echo "Finish backup ${srv_name}"
```

![alt text](https://github.com/rus42/Backup/blob/main/Task_3.1.png)

![alt text](https://github.com/rus42/Backup/blob/main/Task_3.2.png)




