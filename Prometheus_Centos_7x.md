## Prometheus setup

```
wget https://github.com/prometheus/prometheus/releases/download/0.17.0rc2/prometheus-0.17.0rc2.linux-amd64.tar.gz
mkdir /opt/prometheus
tar zxf prometheus-0.17.0rc2.linux-amd64.tar.gz -C /opt/prometheus --strip-components=1
```
Create a simple config:



```
cat << EOF > /opt/prometheus/prometheus.yml
global:
  scrape_interval:     5s
  evaluation_interval: 5s
scrape_configs:
  - job_name: linux
    target_groups:
      - targets: ['192.168.40.148:9100']
        labels:
          alias: db1
  - job_name: mysql
    target_groups:
      - targets: ['192.168.40.148:9104']
        labels:
          alias: db1
EOF
```
```
global:
scrape_interval: 5s
evaluation_interval: 5s
scrape_configs:
– job_name: linux
target_groups:
– targets: [‘localhost:9100’]
labels:
alias: localhost
– job_name: mysql
target_groups:
– targets: [‘192.168.56.106:9104’]
labels:
alias: db1
– targets: [‘192.168.56.107:9105’]
labels:
alias: db2
– targets: [‘192.168.56.108:9106’]
labels:
alias: db4
– targets: [‘localhost:9106’]
labels:
alias: db3
```
```
[root@master ~]# cd /opt/prometheus
[root@master prometheus]# ls
console_libraries  consoles  prometheus  prometheus.yml  promtool
[root@master prometheus]# ./prometheus
prometheus, version 0.17.0rc2 (branch: release-0.17, revision: 667c221)
  build user:       fabianreinartz@macpro
  build date:       20160205-13:35:53
  go version:       1.5.3
INFO[0000] Loading configuration file prometheus.yml     source=main.go:201
INFO[0000] Loading series map and head chunks...         source=storage.go:297
INFO[0000] 0 series loaded.                              source=storage.go:302
WARN[0000] No AlertManager configured, not dispatching any alerts  source=notification.go:165
INFO[0000] Starting target manager...                    source=targetmanager.go:114
INFO[0000] Target manager started.                       source=targetmanager.go:168
INFO[0000] Listening on :9090                            source=web.go:239

```

`http://192.168.40.148:9090`

### Prometheus exporters setup

### Install MySQL

Download and add the repository, then update.

```
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum update
```

Install MySQL as usual and start the service. During installation, you will be asked if you want to accept the results from the .rpm file’s GPG verification. If no error or mismatch occurs, enter y.

```
sudo yum install mysql-server
sudo systemctl start mysqld
```

```
wget https://github.com/prometheus/node_exporter/releases/download/0.12.0rc3/node_exporter-0.12.0rc3.linux-amd64.tar.gz
wget https://github.com/prometheus/mysqld_exporter/releases/download/0.7.1/mysqld_exporter-0.7.1.linux-amd64.tar.gz
mkdir /opt/prometheus_exporters
tar zxf node_exporter-0.12.0rc3.linux-amd64.tar.gz -C /opt/prometheus_exporters
tar zxf mysqld_exporter-0.7.1.linux-amd64.tar.gz -C /opt/prometheus_exporters
```

```
mysql_secure_installation
mysql -u root -p
GRANT REPLICATION CLIENT, PROCESS ON *.* TO 'prom'@'localhost' identified by 'Welcome123';
GRANT SELECT ON performance_schema.* TO 'prom'@'localhost';
```
```
cd /opt/prometheus_exporters

cat << EOF > .my.cnf
[client]
user=prom
password=Welcome123
EOF
```

```
./mysqld_exporter -config.my-cnf=".my.cnf"
[root@master prometheus_exporters]# ./mysqld_exporter -config.my-cnf=".my.cnf"
INFO[0000] Starting Server: :9104                        file=mysqld_exporter.go line=1997
```
```

https://www.percona.com/blog/2016/02/29/graphing-mysql-performance-with-prometheus-and-grafana/

https://cloudcraft.info/huong-dan-setup-prometheus-grafana-de-monitor-dich-vu/









