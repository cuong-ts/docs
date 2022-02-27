# Install redmine on centos 7

### Requirements

**centos 7**

```bash
[root@new-redmine ~]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
```

**mariadb 5.5.68**

```text
[root@new-redmine ~]# mysql --version
mysql  Ver 15.1 Distrib 5.5.68-MariaDB, for Linux (x86_64) using readline 5.1
```

**rbenv installed, ruby 2.5.5**

```text
[redmine@new-redmine ~]$ rbenv --version
rbenv 1.1.2-40-g62d7798
```

```text
[redmine@new-redmine current]$ rbenv version
2.5.5 (set by /opt/redmine/current/.ruby-version)
```

**redmine version 4.1.1**

```text
ssh keypair setup
```

[https://docs.gitlab.com/ee/ssh/README.html](https://docs.gitlab.com/ee/ssh/README.html)

### Install redmine

**Install dependencies**

```text
sudo yum update
sudo yum install vim curl zlib-devel curl-devel openssl-devel httpd-devel apr-devel apr-util-devel mysql-devel ftp wget ImageMagick-devel gcc-c++ patch readline readline-devel zlib libyaml-devel libffi-devel make bzip2 autoconf automake libtool bison subversion sqlite-devel
sudo yum install epel-release
sudo yum install nginx
```

**Setup database**

install mariadb

```text
sudo yum install mariadb-server
sudo systemctl start mariadb
sudo systemctl enable mariadb

mysql_secure_installation

mysql -uroot -p
```

create redmine database and setting password

```text
mysql -uroot -p
MariaDB [(none)]> CREATE DATABASE redmine CHARACTER SET utf8;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost' IDENTIFIED BY 'your-passs-';
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> \\q
```

**Setup user redmine, rbenv, ruby**

```bash
[root@new-redmine ~]# adduser redmine

[root@new-redmine ~]# sudo su - redmine

curl -sL <https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-installer> | bash -

echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc

echo 'eval "$(rbenv init -)"' >> ~/.bashrc

source ~/.bashrc
```

**Setting bashrc**

```bash
[root@new-redmine current]# cat /home/redmine/.bashrc
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

export PG_DATABASE=redmine
export PG_USERNAME=redmine
export PG_PASSWORD=your-passs-
export PG_HOST=localhost

export RACK_ENV="production"
export RAILS_ENV="production"

# User specific aliases and functions
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
export PATH="$HOME/.rbenv/bin:$PATH"
```

**Setting nginx**

```bash
[root@new-redmine ~]# cat /etc/nginx/conf.d/redmine.conf
upstream redmine {
     server unix:///opt/redmine/shared/tmp/redmine-puma.sock;
}

server {
    listen 80;
    server_name xxxxxx;
    root /opt/redmine/current/public;

        location / {
                try_files $uri @redmine;
        }

        location @redmine {
                proxy_set_header  X-Forwarded-For $remote_addr;
                proxy_pass <http://redmine>;
        }
}
```

```text

[root@new-redmine ~]# service nginx start


```

### Restart server

```text
[root@new-redmine current]# service nginx restart
[root@new-redmine current]# service mariadb restart
```



