#26-homeWork. 44 - Postgres: Backup + Репликация  
## Описание файлов в директории
logFileFull.log - полный лог выполнения  

Vagrant_folder - все что понадобится для поднятия ВМ и краткое описание файлов в ней  
Vagrantfile - вагрант файл  
provisioning - директория с файлами провижинга  
```
├── provisioning
│   ├── playbook.yml
│   └── roles
│       ├── backup
│       │   ├── files
│       │   │   ├── barman.conf
│       │   │   ├── master.conf
│       │   │   ├── pgpass
│       │   │   └── ssh
│       │   │       ├── authorized_keys
│       │   │       ├── id_rsa
│       │   │       └── id_rsa.pub
│       │   └── tasks
│       │       └── main.yml
│       ├── create_test_db
│       │   └── tasks
│       │       ├── main.yml
│       │       └── main.yml.save
│       ├── master
│       │   ├── files
│       │   │   ├── pg_hba.conf
│       │   │   ├── postgresql.conf
│       │   │   └── ssh
│       │   │       ├── config
│       │   │       ├── id_rsa
│       │   │       └── id_rsa.pub
│       │   ├── handlers
│       │   │   └── main.yml
│       │   ├── tasks
│       │   │   └── main.yml
│       │   ├── templates
│       │   └── vars
│       │       └── main.yml
│       └── slave
│           ├── files
│           ├── handlers
│           │   └── main.yml
│           ├── tasks
│           │   └── main.yml
│           ├── templates
│           └── vars
│               └── main.yml
└── Vagrantfile
```

## Описание как запустить виртуальную машину (кратко)
```
vagrant up
```

## Проверка работы

1. Master
Проверяем что при выполнении роли create_test_db создалась база данных acme  
![Image 1](https://github.com/ballrak86/26-homeWork/blob/main/screenshots/postgresql-master.jpg)

2. Slave
Проверяем что база данных acme появилась на slave  
![Image 2](https://github.com/ballrak86/26-homeWork/blob/main/screenshots/postgresql-slave.jpg)

2. Backup
```
root@otssTestServer:/home/lx/git/postgersql# vagrant ssh backup
Last login: Thu Dec 15 08:15:35 2022 from 10.0.2.2
[vagrant@backup ~]$ sudo -i
[root@backup ~]# barman switch-wal --force --archive master
The WAL file 000000010000000000000003 has been closed on server 'master'
Waiting for the WAL file 000000010000000000000003 from server 'master' (max: 30 seconds)
Processing xlog segments from streaming for master
        000000010000000000000003
[root@backup ~]# barman check master
Server master:
        PostgreSQL: OK
        superuser or standard user with backup privileges: OK
        PostgreSQL streaming: OK
        wal_level: OK
        replication slot: OK
        directories: OK
        retention policy settings: OK
        backup maximum age: OK (no last_backup_maximum_age provided)
        backup minimum size: OK (0 B)
        wal maximum age: OK (no last_wal_maximum_age provided)
        wal size: OK (0 B)
        compression settings: OK
        failed backups: OK (there are 0 failed backups)
        minimum redundancy requirements: OK (have 0 backups, expected at least 0)
        pg_basebackup: OK
        pg_basebackup compatible: OK
        pg_basebackup supports tablespaces mapping: OK
        systemid coherence: OK (no system Id stored on disk)
        pg_receivexlog: OK
        pg_receivexlog compatible: OK
        receive-wal running: OK
        archive_mode: OK
        archive_command: OK
        archiver errors: OK
[root@backup ~]# barman backup master --wait
Starting backup using postgres method for server master in /var/lib/barman/master/base/20221215T082738
Backup start at LSN: 0/4000060 (000000010000000000000004, 00000060)
Starting backup copy via pg_basebackup for 20221215T082738
Copy done (time: 9 seconds)
Finalising the backup.
This is the first backup for server master
WAL segments preceding the current backup have been found:
        000000010000000000000003 from server master has been removed
Backup size: 31.1 MiB
Backup end at LSN: 0/6000060 (000000010000000000000006, 00000060)
Backup completed (start time: 2022-12-15 08:27:38.599174, elapsed time: 10 seconds)
Waiting for the WAL file 000000010000000000000006 from server 'master'
Processing xlog segments from streaming for master
        000000010000000000000004
        000000010000000000000005
        000000010000000000000006
Processing xlog segments from file archival for master
        000000010000000000000001
        000000010000000000000002
        000000010000000000000002.00000028.backup
        000000010000000000000003
        000000010000000000000004
        000000010000000000000005
        000000010000000000000005.00000028.backup
        000000010000000000000006
[root@backup ~]# barman status master
Server master:
        Description: PostgreSQL Backup
        Active: True
        Disabled: False
        PostgreSQL version: 12.13
        Cluster state: in production
        pgespresso extension: Not available
        Current data size: 31.3 MiB
        PostgreSQL Data directory: /var/lib/pgsql/12/data
        Current WAL segment: 000000010000000000000007
        PostgreSQL 'archive_command' setting: rsync -a %p barman@backup:/var/lib/barman/master/incoming/%f
        Last archived WAL: 000000010000000000000006, at Thu Dec 15 08:27:49 2022
        Failures of WAL archiver: 36 (000000010000000000000001 at Thu Dec 15 08:27:19 2022)
        Server WAL archiving rate: 27.74/hour
        Passive node: False
        Retention policies: not enforced
        No. of available backups: 1
        First available backup: 20221215T082738
        Last available backup: 20221215T082738
        Minimum redundancy requirements: satisfied (1/0)
[root@backup ~]# barman check master
Server master:
        PostgreSQL: OK
        superuser or standard user with backup privileges: OK
        PostgreSQL streaming: OK
        wal_level: OK
        replication slot: OK
        directories: OK
        retention policy settings: OK
        backup maximum age: OK (no last_backup_maximum_age provided)
        backup minimum size: OK (31.1 MiB)
        wal maximum age: OK (no last_wal_maximum_age provided)
        wal size: OK (0 B)
        compression settings: OK
        failed backups: OK (there are 0 failed backups)
        minimum redundancy requirements: OK (have 1 backups, expected at least 0)
        pg_basebackup: OK
        pg_basebackup compatible: OK
        pg_basebackup supports tablespaces mapping: OK
        systemid coherence: OK
        pg_receivexlog: OK
        pg_receivexlog compatible: OK
        receive-wal running: OK
        archive_mode: OK
        archive_command: OK
        continuous archiving: OK
        archiver errors: OK
[root@backup ~]# barman replication-status master
Status of streaming clients for server 'master':
  Current LSN on master: 0/70000C8
  Number of streaming clients: 2

  1. Async standby
     Application name: walreceiver
     Sync stage      : 5/5 Hot standby (max)
     Communication   : TCP/IP
     IP Address      : 192.168.1.20 / Port: 34622 / Host: -
     User name       : replicator
     Current state   : streaming (async)
     Replication slot: pgstandby1
     WAL sender PID  : 5229
     Started at      : 2022-12-15 08:13:23.405714+00:00
     Sent LSN   : 0/70000C8 (diff: 0 B)
     Write LSN  : 0/70000C8 (diff: 0 B)
     Flush LSN  : 0/70000C8 (diff: 0 B)
     Replay LSN : 0/70000C8 (diff: 0 B)

  2. Async WAL streamer
     Application name: barman_receive_wal
     Sync stage      : 3/3 Remote write
     Communication   : TCP/IP
     IP Address      : 192.168.1.30 / Port: 37712 / Host: -
     User name       : barman_streaming_user
     Current state   : streaming (async)
     Replication slot: barman
     WAL sender PID  : 5275
     Started at      : 2022-12-15 08:16:02.904880+00:00
     Sent LSN   : 0/70000C8 (diff: 0 B)
     Write LSN  : 0/70000C8 (diff: 0 B)
     Flush LSN  : 0/7000000 (diff: -200 B)

```

📚Домашнее задание/проектная работа разработано(-на) для курса ["Administrator Linux. Professional"](https://otus.ru/lessons/linux-professional/)
