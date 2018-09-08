
### Add `prometheus` system user and group:

```
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```
This user will manage the exporter service.

### Download and install Prometheus MySQL Exporter:

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



### Configure MySQL endpoint to be scraped by Prometheus Server

Login to your Prometheus server and Configure endpoint to scrape. Below is an example for two MySQL database servers.

```
scrape_configs:
  - job_name: server1_db
    static_configs:
      - targets: ['10.10.1.10:9104']
        labels:
          alias: db1

  - job_name: server2_db
    static_configs:
      - targets: ['10.10.1.11:9104']
        labels:
          alias: db2
```

Add other targets by using similar format.

## Creating / Importing MySQL Grafana dashboards

Now that we have the targets configured and agents to be monitored, we shuld be good to add Prometheus data source
to Grafana so that we can do metrics visualization. If you don't have a ready Grafana server, use any of the guide below to install Grafana


When installed, login to admin dashboard and add Datasource by navigating to ` Configuration > Data Sources`.

```
Name: Prometheus
Type: Prometheus
URL: http://localhost:9090
```

If Prometheus server is not on the same host as Grafana, provide IP address of the server.


### Create / Import Grafana Dashboard for MySQL Prometheus exporter

If you don't have all the golden time to create your own dashboards, you can use one created by Percona, they are Open source

https://github.com/percona/grafana-dashboards



