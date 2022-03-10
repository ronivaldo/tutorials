# Backup and Restore a Docker Volume

Reference:
https://github.com/loomchild/volume-backup

As the docker user guide explains, [data volumes](https://docs.docker.com/userguide/dockervolumes/#data-volumes) are meant to persist data outside of a container filesystem. This also ease the sharing of data between multiple containers.

A Metabase Docker compose file will be used as containers:
```yml
version: '3.8'

services:
  db:
    build:
      context: .
      dockerfile: 'Dockerfile-${DB_TYPE}'
      args:
        db_name: ${DB_NAME}
        db_user: ${DB_USER}
        db_password: ${DB_PASSWORD}
        db_port: ${DB_PORT}
    restart: always
    volumes:
      - type: volume
        source: ${DB_OUTSIDE_VOLUME}
        target: ${DB_INSIDE_VOLUME}

  adminer:
    image: adminer
    restart: always
    ports:
      - '${ADMINER_PORT}:8080'
    depends_on:
      - 'db'

  metabase:
    image: metabase/metabase
    restart: always
    environment:
      MB_DB_FILE: '${MB_DB_FILE}'
      MB_DB_TYPE: '${DB_TYPE}'
      MB_DB_DBNAME: '${DB_NAME}'
      MB_DB_PORT: '${DB_PORT}'
      MB_DB_USER: '${DB_USER}'
      MB_DB_HOST: 'db'
      MB_DB_PASS: '${DB_PASSWORD}'
      JAVA_TIMEZONE: '${MB_JAVA_TIMEZONE}'
    ports:
      # <Port exposed>:<Port running inside container>
      - '${MB_PORT}:3000'
    volumes:
      # Volumes where Metabase data will be persisted
      - 'mb-data:/metabase-data'
    depends_on:
      - 'db'

volumes:
  db-data:
  mb-data:
```

## Inspect the Volume

### List

List all the docker volumes and find the one you would like to backup.

```bash
docker volume ls
```
``` 
...
local     fb6462408f1e809f652e54d7f98c0f59bb6d1e86af20d7fdab50b5e56a8b0bb3
local     metabase-compose_db-data
local     metabase-compose_mb-data
...
```
For this example, we need to backup `metabase-compose_db-data`.

### Inspect

Inspect the volume in order to find out where the data is saved.

```bash
docker volume inspect metabase-compose_db-data
```
```json
[
    {
        "CreatedAt": "2022-03-10T11:33:54-03:00",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "metabase-compose",
            "com.docker.compose.version": "1.29.2",
            "com.docker.compose.volume": "db-data"
        },
        "Mountpoint": "/var/lib/docker/volumes/metabase-compose_db-data/_data",
        "Name": "metabase-compose_db-data",
        "Options": null,
        "Scope": "local"
    }
]
```
This volume data is saved in folder `/var/lib/docker/volumes/metabase-compose_db-data/_data`.

### Check Files

Let's check all the files in the folder.

```bash
ls -la /var/lib/docker/volumes/metabase-compose_db-data/_data
```
```
total 188488
drwxrwxrwt 6 systemd-coredump systemd-coredump     4096 Mar 10 11:33 .
drwx-----x 3 root             root                 4096 Mar 10 11:32 ..
-rw-r----- 1 systemd-coredump systemd-coredump       56 Mar 10 11:33 auto.cnf
-rw------- 1 systemd-coredump systemd-coredump     1680 Mar 10 11:33 ca-key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1112 Mar 10 11:33 ca.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1112 Mar 10 11:33 client-cert.pem
-rw------- 1 systemd-coredump systemd-coredump     1680 Mar 10 11:33 client-key.pem
-rw-r----- 1 systemd-coredump systemd-coredump     1352 Mar 10 11:33 ib_buffer_pool
-rw-r----- 1 systemd-coredump systemd-coredump 79691776 Mar 10 15:55 ibdata1
-rw-r----- 1 systemd-coredump systemd-coredump 50331648 Mar 10 15:55 ib_logfile0
-rw-r----- 1 systemd-coredump systemd-coredump 50331648 Mar 10 11:33 ib_logfile1
-rw-r----- 1 systemd-coredump systemd-coredump 12582912 Mar 10 14:21 ibtmp1
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Mar 10 11:36 my_db
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Mar 10 11:33 mysql
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Mar 10 11:33 performance_schema
-rw------- 1 systemd-coredump systemd-coredump     1680 Mar 10 11:33 private_key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump      452 Mar 10 11:33 public_key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1112 Mar 10 11:33 server-cert.pem
-rw------- 1 systemd-coredump systemd-coredump     1676 Mar 10 11:33 server-key.pem
drwxr-x--- 2 systemd-coredump systemd-coredump    12288 Mar 10 11:33 sys
```

## Backup Volume

### Stop Running Containers

In order to make a backup file of the volume, we need to stop the running containers.

```bash
docker-compose down
```
```
Stopping metabase-compose_adminer_1  ... done
Stopping metabase-compose_metabase_1 ... done
Stopping metabase-compose_db_1       ... done
Removing metabase-compose_adminer_1  ... done
Removing metabase-compose_metabase_1 ... done
Removing metabase-compose_db_1       ... done
Removing network metabase-compose_default
```

### Backup Volume

The backup can be made easily by `loomchild/volume-backup` image, we need to pass the volume `metabase-compose_db-data`, the backup destination folder `/tmp` and the destination filename `db-data_backup_20220310.tar.bz2`

```bash
docker run -v metabase-compose_db-data:/volume -v /tmp:/backup --rm loomchild/volume-backup backup db-data_backup_20220310.tar.bz2
```

### Verify Backup file

Check if the backup file was created.

```bash
ls /tmp/db-data_backup_20220310.tar.bz2
```
```
/tmp/db-data_backup_20220310.tar.bz2
```

## Simulate a Disaster

### Delete Volume Files

We can simulate a disaster by removing all the files and directories saved in the volume data folder.

```bash
rm -r /var/lib/docker/volumes/metabase-compose_db-data/_data/*
```

### Files are Missing

Let's check if the files aren't indeed in the directory.

```bash
ls -la /var/lib/docker/volumes/metabase-compose_db-data/_data/
```
```
total 8
drwxr-xr-x 2 systemd-coredump root 4096 Mar 10 17:38 .
drwx-----x 3 root             root 4096 Mar 10 17:32 ..
```

## Restore Volume

### Restore

The restore can be made easily by `loomchild/volume-backup` image, we need to pass the volume `metabase-compose_db-data`, the backup source folder `/tmp` and the backfou source filename `db-data_backup_20220310.tar.bz2`, use the `-f` argument if you want to force current volume files to be overwritten.

```
docker run -v metabase-compose_db-data:/volume -v /tmp:/backup --rm loomchild/volume-backup restore -f db-data_backup_20220310.tar.bz2
```

### Verify Files in Restored Volume

Let's check whether all the files are back in the volume data folder.

```
ls -la /var/lib/docker/volumes/metabase-compose_db-data/_data/
```
```
total 176204
drwxr-xr-x 6 systemd-coredump root                 4096 Mar 10 17:06 .
drwx-----x 3 root             root                 4096 Mar 10 17:32 ..
-rw-r----- 1 systemd-coredump root                   56 Mar 10 16:06 auto.cnf
-rw------- 1 systemd-coredump root                 1680 Mar 10 16:06 ca-key.pem
-rw-r--r-- 1 systemd-coredump root                 1112 Mar 10 16:06 ca.pem
-rw-r--r-- 1 systemd-coredump root                 1112 Mar 10 16:06 client-cert.pem
-rw------- 1 systemd-coredump root                 1680 Mar 10 16:06 client-key.pem
-rw-r----- 1 systemd-coredump systemd-coredump     1094 Mar 10 16:08 ib_buffer_pool
-rw-r----- 1 systemd-coredump root             79691776 Mar 10 16:08 ibdata1
-rw-r----- 1 systemd-coredump root             50331648 Mar 10 16:08 ib_logfile0
-rw-r----- 1 systemd-coredump root             50331648 Mar 10 16:06 ib_logfile1
drwxr-x--- 2 systemd-coredump root                 4096 Mar 10 16:06 my_db
drwxr-x--- 2 systemd-coredump root                 4096 Mar 10 16:06 mysql
drwxr-x--- 2 systemd-coredump root                 4096 Mar 10 16:06 performance_schema
-rw------- 1 systemd-coredump root                 1680 Mar 10 16:06 private_key.pem
-rw-r--r-- 1 systemd-coredump root                  452 Mar 10 16:06 public_key.pem
-rw-r--r-- 1 systemd-coredump root                 1112 Mar 10 16:06 server-cert.pem
-rw------- 1 systemd-coredump root                 1676 Mar 10 16:06 server-key.pem
drwxr-x--- 2 systemd-coredump root                12288 Mar 10 16:06 sys
-rw-r--r-- 1 root             root                    9 Mar 10 17:06 test.txt
```

### Restart Containers

Restart containers and check if they are running as expected.

```bash
docker-compose up -d
```
```
Creating network "metabase-compose_default" with the default driver
Creating metabase-compose_db_1 ... done
Creating metabase-compose_adminer_1  ... done
Creating metabase-compose_metabase_1 ... done
```
