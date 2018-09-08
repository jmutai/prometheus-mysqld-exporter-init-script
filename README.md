This is a setup guide for Prometheus MySQLD exporter init script.

For a systemd server, use:

[Monitoring MySQL / MariaDB with Prometheus in five minutes](https://computingforgeeks.com/monitoring-mysql-mariadb-with-prometheus-in-five-minutes/)

### Add `prometheus` system user and group:

```
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```
This user will manage the exporter service.

### Download and install Prometheus MySQL Exporter:

You may need to check [Prometheus MySQL exporter releases](https://github.com/prometheus/mysqld_exporter/releases) page for the latest release, then export the latest version  to VER variable as shown below:


```
export VER=0.11.0
wget https://github.com/prometheus/mysqld_exporter/releases/download/v${VER}/mysqld_exporter-${VER}.linux-amd64.tar.gz
tar xvf mysqld_exporter-${VER}.linux-amd64.tar.gz
sudo mv  mysqld_exporter-${VER}.linux-amd64/mysqld_exporter /usr/local/bin/
rm -rf mysqld_exporter-${VER}.linux-amd64
rm mysqld_exporter-${VER}.linux-amd64.tar.gz 
```

### Create Prometheus exporter database user with `PROCESS, SELECT, REPLICATION CLIENT` grants

```
CREATE USER 'mysqld_exporter'@'localhost' IDENTIFIED BY 'StrongPassword' WITH MAX_USER_CONNECTIONS 2;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'mysqld_exporter'@'localhost';
FLUSH PRIVILEGES;
EXIT
```

If you have a Master-Slave database architecture, create user on the master servers only.

`WITH MAX_USER_CONNECTIONS 2` is used to set a max connection limit for the user to avoid overloading the server with monitoring scrapes under heavy load.


### Configure database credentials 

Create database credentials file

```
sudo vim /etc/.mysqld_exporter.cnf
```

Add correct username and password for user create

```
[client]
user=mysqld_exporter
password=StrongPassword
```

Set ownership permissions:

```
sudo chown root:prometheus /etc/.mysqld_exporter.cnf
```

### Download the init script

Install daemonize package which will be used to run the process in the background

For Ubuntu / Debian systems, install it using:

```
sudo apt-get install daemonize
```

For CentOS 6.x, use

```
sudo yum install daemonize
```

Download the script and place it on `/etc/init.d`

```
git clone https://github.com/jmutai/prometheus-mysqld-exporter-init-script.git
cd prometheus-mysqld-exporter-init-script
chmod +x mysqld_exporter.init
sudo mv mysqld_exporter.init /etc/init.d/mysqld_exporter
```

To start the service, just run:

```
sudo /etc/init.d/mysqld_exporter start
```

Set it to start on boot

```
$ sudo chkconfig mysqld_exporter on
$ sudo chkconfig --list | grep mysqld_exporter
mysqld_exporter	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```

### Configure MySQL endpoint to be scraped by Prometheus Server

For configuration of MySQL endpoint to be scraped by Prometheus Server, check the complete guide 

[Monitoring MySQL / MariaDB with Prometheus in five minutes](https://computingforgeeks.com/monitoring-mysql-mariadb-with-prometheus-in-five-minutes/)

