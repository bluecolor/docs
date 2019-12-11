## Postgres 11

### Ubuntu 18.04 and 16.04

- `sudo apt-get install wget ca-certificates`
- `wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -`

- ``sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'``

- `sudo apt-get update`
- `sudo apt-get install postgresql postgresql-contrib`

### Create user and database

```sh
sudo -u postgres psql
create database mydb;
create user myuser with encrypted password 'mypass';
grant all privileges on database mydb to myuser;
```

### Start postgres

```sh
sudo service postgresql start
```

### Start on boot

```sh
sudo systemctl enable postgresql
```