## Mariadb

### Centos7

#### Install

`sudo yum install mariadb-server`

#### Start mariadb

`sudo systemctl start mariadb`

#### Create user and database

```
create database DATABASE_NAME;
grant all privileges on DATABASE_NAME.* TO 'USER_NAME'@'localhost' identified by 'PASSWORD';
grant all privileges on DATABASE_NAME.* TO 'USER_NAME'@'%' identified by 'PASSWORD';
flush privileges;
```

#### Start on boot

```sh
sudo systemctl enable mariadb
```

