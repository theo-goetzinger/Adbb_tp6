## Théo Goetzinger

## TP5

## docker-compose.yml

```
version:  '3.7'
services:
 master:
  image: mariadb:10.4

  restart: on-failure
  environment:
   MYSQL_ROOT_PASSWORD: password
  volumes:
   - ./master:/var/lib/mysql
   - ./backups:/backups
   - ./config/master.cnf:/etc/mysql/mariadb.conf.d/master.cnf
   - ./scripts:/scripts
  networks:
    - internals
  ports:
    - 3306

 slave:
  image: mariadb:10.4
  restart: on-failure
  environment:
   MYSQL_ROOT_PASSWORD: password
  volumes:
   - ./slave:/var/lib/mysql
   - ./backups:/backups
   - ./config/slave.cnf:/etc/mysql/mariadb.conf.d/slave.cnf
   - ./scripts:/scripts
  networks:
    - internals
  ports:
    - 3306


networks:
 internals:

```

## lancement du docker-compose
```
sudo docker-compose up -d

```

## ajout du fichier de conf master /config/master.cnf
```
[mariadb]
log-bin
server_id=1
log-basename=master1
binlog-format=mixed

```

## ajout du fichier de conf slave /config/slave.cnf
```
[mariadb]
log-bin
server_id=2
log-basename=master1
binlog-format=mixed

```

## Création du script pour ajouter l'utilisateur avec les droits de replication sur master

```
CREATE USER IF NOT EXISTS 'replicant'@'%' IDENTIFIED BY 'replicant_password',
grant replication slave on *.* to replicant;

flush privileges;

```

## Vérification de l'état du master 

```
MariaDB [(none)]> SHOW MASTER STATUS;
+--------------------+----------+--------------+------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+--------------------+----------+--------------+------------------+
| master1-bin.000005 |      344 |              |                  |
+--------------------+----------+--------------+------------------+

```
## Mise en place du point de réplication sur le slave 

```
CHANGE MASTER TO
MASTER_HOST='master',
MASTER_USER='replicant',
MASTER_PASSWORD='replicant_password',
MASTER_PORT=3306,
MASTER_LOG_FILE='master1-bin.000005',
MASTER_LOG_POS=344,
MASTER_CONNECT_RETRY=10;

```
## mise en route du Slave

```
START SLAVE;

```
## Vérification de l'état du slave 

```
SHOW SLAVE STATUS;

```

