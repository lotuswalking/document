# Intall Postgresql 11 on centos 7

Environment:  centos 7, without firewall

disk: 

/ ==> 100GB

/data ==> 500GB for database store

basic steps refered on this [link](https://computingforgeeks.com/how-to-install-postgresql-11-on-centos-7/):


## Step 1: Update system

`sudo yum update -y`


## Step 2: Add PostgreSQL Yum Repository

`sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm`


## Step 3: Install PostgreSQL 11 on CentOS 7 / RHEL 7

`sudo yum -y install postgresql11-server postgresql11`


Confirm the installed package:

```
$ rpm -qi postgresql11-server
Name        : postgresql11-server
Version     : 11.16
Release     : 1PGDG.rhel7
Architecture: x86_64
Install Date: Wed May 18 20:32:25 2022
Group       : Unspecified
Size        : 19804199
License     : PostgreSQL
Signature   : DSA/SHA1, Wed May 11 19:27:21 2022, Key ID 1f16d2e1442df0f8
Source RPM  : postgresql11-11.16-1PGDG.rhel7.src.rpm
Build Date  : Wed May 11 13:54:55 2022
Build Host  : koji-centos7-x86-64-pgbuild
Relocations : (not relocatable)
Vendor      : PostgreSQL Global Development Group
URL         : https://www.postgresql.org/
```

## Step 4: Change the init database path([refer on this link](https://dba.stackexchange.com/questions/292431/postgresql-setup-initdb-with-custom-data-directory))

Create folder and assign to postgres

```bash
mkdir -p /pgdata/11/data
sudo chown postgres:postgres /pgdata/11/data
```

Then get `systemctl` to create a `service` file(the file name depend on your version, don's just follow):

```bash
sudo systemctl edit postgresql-11.service
```

As mentioned above, just add the `PGDATA` location:

```bash
[Service]
Environment=PGDATA=/data/11/data
```

This will create a `/etc/systemd/system/postgresql-14.service.d/override.conf` file. This will be merged with the original service file.

To check its content:

```bash
# cat /etc/systemd/system/postgresql-11.service.d/override.conf
[Service]
Environment=PGDATA=/data/11/data
```

Reload systemd:

```bash
# systemctl daemon-reload
```

Initialize the PostgreSQL data directory:

```bash
sudo /usr/pgsql-11/bin/postgresql-11-setup initdb
```

Start and enable the service:

```bash
sudo systemctl enable postgresql-11
sudo systemctl start postgresql-11
```


## Step 5: Enable remote Access to PostgreSQL (Not recommended)

Edit the file `/var/lib/pgsql/11/data/postgresql.conf` and set Listen address to your server IP address or “ ***** ” for all interfaces.

```
listen_addresses = '*'
```

Also set PostgreSQL to accept remote connections

```
$ sudo vim /var/lib/pgsql/11/data/pg_hba.conf
# Accept from anywhere
host all all 0.0.0.0/0 md5

# Accept from trusted subnet
host all all 192.168.18.0/24 md5
```

Restart service

```
sudo systemctl restart postgresql-11
```

## Step 6: Set PostgreSQL admin user’s password

Set PostgreSQL admin user

```
$ sudo su - postgres 
bash-4.2$ psql -c "alter user postgres with password 'StrongPassword'" 
ALTER ROLE
-bash-4.2$
```

Create a test user and database

```
createuser test_user
createdb test_db -O test_user
grant all privileges on database test_db to test_user;
```

Login as a `test_user`  user try to create a table on the Database.


## Other CMDS to create user/database

psql:

CREATE USER teamspcr WITH PASSWORD 'ZAQ!2wsxXSW@1qaz';

CREATE DATABASE teamspcr OWNER teamspcr;
GRANT ALL PRIVILEGES ON DATABASE teamspcr TO teamspcr;
