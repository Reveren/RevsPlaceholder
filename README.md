# RevsPlaceholder
Just a place to publicly store some stuff. 

After installing Postgresql ona Mac from brew, find out username:
```PLpgSQL
$ cd ~
$ pwd
/Users/someuser
$ psql -d postgres -U someuser
```
Now that you have logged into the system, and you can create the DB.
```
postgres=# create database mydb;
CREATE DATABASE
postgres=# create user myuser with encrypted password 'pass123';
CREATE ROLE
postgres=# grant all privileges on database mydb to myuser;
GRANT
```
