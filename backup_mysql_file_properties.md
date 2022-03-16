
# Backup a MySQL Running Container with File Properties

_Backup_ and restore a _MySql_ container database from a _running_ Docker mysql container can be tricky. In this tutorial we will find an easy way keeping the file properties.

## List Containers

List the running containers you want to backup.

```bash
docker ps | grep dbms
```
```
ab1b3c25a7c0   nao-similaridade-app_dbms                        "docker-entrypoint.s…"   5 days ago    Up 5 days    33060/tcp, 0.0.0.0:33306->3306/tcp, :::33306->3306/tcp          nao-similaridade-app_dbms_1
```
Now we have the container name `nao-similaridade-app_dbms_1`.

## Start a shell in the container

Use docker to start a shell:
```bash
docker exec -it nao-similaridade-app_dbms_1 'bash'
```

## Install Gzip to compress data

Inside the container, install gzip:
```bash
apt-get install bzip2
```
## Compress the MySql directory

Compress the `mysql` directory to a file:
```bash
tar -C /var/lib/mysql -j -cf /tmp/db-data_backup_20220316_FINAL.tar.bz2 ./
```

## Copy backup file from container

You must `exit` the shell you started earlier using `docker exec`, then copy the compressed file:
```bash
exit
```
```bash
docker cp nao-similaridade-app_dbms_1:/tmp/db-data_backup_20220316_FINAL.tar.bz2 /tmp/db-data_backup_20220316_FINAL.tar.bz2
```

## Restore to a Docker Volume

With the file `/tmp/db-data_backup_20220315_FINAL.tar.bz2` you can restore the data to a `docker volume`:

```
docker run -v nao-similaridade-app_nao_sim_dbms_data:/volume -v /tmp:/backup --rm loomchild/volume-backup restore -f db-data_backup_20220316_FINAL.tar.bz2
```

More details in [Backup and Restore a Docker Volume](./backup_docker.md)
