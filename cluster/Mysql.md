# Mysql
## Installation
#### OS 
Operating System: CentOS 6.5
Arch: x86_64 and i386 (Both)
Mysql: 5.6.36

#### Install 
[Rpm Repo](http://dev.mysql.com/downloads/repo/)

1. download rpm package
```
yum install wget
wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
```

2. install rpm pak

```
rpm -ivh mysql-community-release-el6-5.noarch.rpm
```

3. install

```
yum install mysql-server
```
4. Start

```
/etc/init.d/mysqld start
```
5. Security configuration

```
mysql_secure_installation
```
6. New user and grant permissions

```
CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password';
grant all privileges on *.* to 'master'@'%' with grant option;
flush privileges;
```



7. Possible error

Happened when connect to mysql on other host, local access OK.
>  ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)

```
sudo mysqld_safe --skip-grant-tables &
```



