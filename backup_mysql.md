# Backup a MySQL Running Container

_Backup_ and restore a _MySql_ container database from a _running_ Docker mysql container can be tricky. In this tutorial we will find an easy way.

## List Containers

List the running containers you want to backup.

```bash
docker ps | grep dbms
```
```
ab1b3c25a7c0   nao-similaridade-app_dbms                        "docker-entrypoint.s…"   5 days ago    Up 5 days    33060/tcp, 0.0.0.0:33306->3306/tcp, :::33306->3306/tcp          nao-similaridade-app_dbms_1
```
Now we have the container name `nao-similaridade-app_dbms_1`.

## Copy files from container

It's best to stop the container, but in case you can't just make sure no one is modifying database data at the moment.

```bash
docker cp nao-similaridade-app_dbms_1:/var/lib/mysql /tmp/db_backup
```
The backup folder is created at `/tmp/db_backup`.

You can check the copied files:
```bash
ls -la /tmp/db_backup
```
```
total 188492
drwxrwxrwt 6 root root     4096 Mar 10 18:43 .
drwxrwxrwt 7 root root     4096 Mar 15 23:03 ..
-rw-r----- 1 root root       56 Mar 10 18:43 auto.cnf
-rw------- 1 root root     1680 Mar 10 18:43 ca-key.pem
-rw-r--r-- 1 root root     1112 Mar 10 18:43 ca.pem
-rw-r--r-- 1 root root     1112 Mar 10 18:43 client-cert.pem
-rw------- 1 root root     1676 Mar 10 18:43 client-key.pem
drwxr-x--- 2 root root     4096 Mar 10 18:43 db_nao_similaridade
-rw-r----- 1 root root     1335 Mar 10 18:43 ib_buffer_pool
-rw-r----- 1 root root 79691776 Mar 15 13:53 ibdata1
-rw-r----- 1 root root 50331648 Mar 15 13:53 ib_logfile0
-rw-r----- 1 root root 50331648 Mar 10 18:43 ib_logfile1
-rw-r----- 1 root root 12582912 Mar 15 22:51 ibtmp1
drwxr-x--- 2 root root     4096 Mar 10 18:43 mysql
drwxr-x--- 2 root root     4096 Mar 10 18:43 performance_schema
-rw------- 1 root root     1680 Mar 10 18:43 private_key.pem
-rw-r--r-- 1 root root      452 Mar 10 18:43 public_key.pem
-rw-r--r-- 1 root root     1112 Mar 10 18:43 server-cert.pem
-rw------- 1 root root     1680 Mar 10 18:43 server-key.pem
drwxr-x--- 2 root root    12288 Mar 10 18:43 sys

```


## Compress the data

Using parts of the script `https://github.com/loomchild/volume-backup/blob/master/volume-backup.sh`, compress the folder:

```bash
tar -C /tmp/db_backup -j -cf /tmp/db-data_backup_20220315_FINAL.tar.bz2 ./
```
```bash
ls -la /tmp/db-data_backup_20220315_FINAL.tar.bz2
-rw-r--r-- 1 root root 12270809 Mar 15 23:11 /tmp/db-data_backup_20220315_FINAL.tar.bz2
```

## Uncompress files back to a folder

You can uncompress the files to a folder using `tar`:

```bash
mkdir /tmp/db_restore_test
```
```bash
tar -C /tmp/db_restore_test/ -j -xf /tmp/db-data_backup_20220315_FINAL.tar.bz2
```
```bash
ls -la /tmp/db_restore_test/
total 188488
drwxrwxrwt  6 root root     4096 Mar 10 18:43 .
drwxrwxrwt 14 root root     4096 Mar 15 23:15 ..
-rw-r-----  1 root root       56 Mar 10 18:43 auto.cnf
-rw-------  1 root root     1680 Mar 10 18:43 ca-key.pem
-rw-r--r--  1 root root     1112 Mar 10 18:43 ca.pem
-rw-r--r--  1 root root     1112 Mar 10 18:43 client-cert.pem
-rw-------  1 root root     1676 Mar 10 18:43 client-key.pem
drwxr-x---  2 root root     4096 Mar 10 18:43 db_nao_similaridade
-rw-r-----  1 root root     1335 Mar 10 18:43 ib_buffer_pool
-rw-r-----  1 root root 79691776 Mar 15 13:53 ibdata1
-rw-r-----  1 root root 50331648 Mar 15 13:53 ib_logfile0
-rw-r-----  1 root root 50331648 Mar 10 18:43 ib_logfile1
-rw-r-----  1 root root 12582912 Mar 15 22:51 ibtmp1
drwxr-x---  2 root root     4096 Mar 10 18:43 mysql
drwxr-x---  2 root root     4096 Mar 10 18:43 performance_schema
-rw-------  1 root root     1680 Mar 10 18:43 private_key.pem
-rw-r--r--  1 root root      452 Mar 10 18:43 public_key.pem
-rw-r--r--  1 root root     1112 Mar 10 18:43 server-cert.pem
-rw-------  1 root root     1680 Mar 10 18:43 server-key.pem
drwxr-x---  2 root root    12288 Mar 10 18:43 sys
```

## Restore to a Docker Volume

With the file `/tmp/db-data_backup_20220315_FINAL.tar.bz2` you can restore the data to a `docker volume`:

```
docker run -v metabase-compose_db-data:/volume -v /tmp:/backup --rm loomchild/volume-backup restore -f db-data_backup_20220315_FINAL.tar.bz2
```

More details in [Backup and Restore a Docker Volume](./backup_docker.md)

## Backup using Dump SQL files

You can backup docker my sql database to a dump.sql using this tutorials:
- https://gist.github.com/spalladino/6d981f7b33f6e0afe6bb
- https://dev.to/lanandra/backup-mysql-database-that-running-on-docker-container-1k8h
