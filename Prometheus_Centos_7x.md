## Prometheus setup

Node Prometheus : 192.168.40.149

Node slave : 192.168.40.450

```
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
init 6
yum -y install wget

# Download source cai dat cua Prometheus
cd ~
wget https://github.com/prometheus/prometheus/releases/download/v2.6.0/prometheus-2.6.0.linux-amd64.tar.gz
mkdir /opt/prometheus
tar zxf prometheus-2.6.0.linux-amd64.tar.gz -C /opt/prometheus --strip-components=1
 
# Tao user cho Prometheus
useradd --no-create-home --shell /bin/false prometheus
 
# Tao folder cho Prometheus
mkdir /etc/prometheus
mkdir /var/lib/prometheus
 
# Cau hinh file config Prometheus
cat << EOF > /etc/prometheus/prometheus.yml
global:
  scrape_interval:     5s
  evaluation_interval: 5s
scrape_configs:
  - job_name: linux
    static_configs:
      - targets: ['192.168.40.150:9100']
        labels:
          alias: vesta_cp
 
  - job_name: mysql
    static_configs:
      - targets: ['192.168.40.150:9104']
        labels:
          alias: vesta_cp
EOF
```
Ex:

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
– targets: [‘192.168.40.1506:9104’]
labels:
alias: db1
– targets: [‘192.168.40.151:9105’]
labels:
alias: db2
– targets: [‘192.168.40.152:9106’]
labels:
alias: db4
– targets: [‘localhost:9106’]
labels:
alias: db3
```
```
# Copy file thuc thi prometheus vao folder /user/local/bin
cp /opt/prometheus/prometheus /usr/local/bin/
cp /opt/prometheus/promtool /usr/local/bin/
cp -r /opt/prometheus/consoles /etc/prometheus
cp -r /opt/prometheus/console_libraries /etc/prometheus
 
# Phan quyen cho user prometheus
chown -R prometheus:prometheus /etc/prometheus
chown -R prometheus:prometheus /var/lib/prometheus
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool
 
# Chay prometheus
prometheus /usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries
    
Neu bao loi chay lenh sau:    

sudo -u prometheus /usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries     
```

Ta cần phải tạo 1 file systemd để tự động restart lại service khi bị crash hoặc reboot server.
```
vi /etc/systemd/system/prometheus.service

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
Restart lại service prometheus và enable tính năng auto restart của prometheus

```
systemctl daemon-reload
systemctl restart prometheus
systemctl status prometheus
systemctl enable prometheus
```

Truy cap trang monitor: `http://192.168.40.149:9090`

### Prometheus exporters setup

### Install MySQL

Download and add the repository, then update.

```
yum -y install wget
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum update
```

Install MySQL as usual and start the service. During installation, you will be asked if you want to accept the results from the .rpm file’s GPG verification. If no error or mismatch occurs, enter y.

```
sudo yum install mysql-server
sudo systemctl start mysqld
sudo systemctl status mysqld
```

```
# Tai 2 agent ve node Slave
wget https://github.com/prometheus/node_exporter/releases/download/v0.17.0/node_exporter-0.17.0.linux-amd64.tar.gz
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.11.0/mysqld_exporter-0.11.0.linux-amd64.tar.gz
 
# Tao folder chua 2 agent
mkdir /opt/prometheus_exporters

# Giai nen source cai dat
tar zxf node_exporter-0.17.0.linux-amd64.tar.gz -C /opt/prometheus_exporters
tar zxf mysqld_exporter-0.11.0.linux-amd64.tar.gz -C /opt/prometheus_exporters
 
# Start Linux agent
cd /opt/prometheus_exporters/node_exporter-0.17.0.linux-amd64
./node_exporter &
```

```
mysql_secure_installation
mysql -u root -p
GRANT REPLICATION CLIENT, PROCESS ON *.* TO 'prom'@'localhost' identified by 'Welcome123';
GRANT SELECT ON performance_schema.* TO 'prom'@'localhost';
```
```

cat << EOF > .my.cnf
[client]
user=prom
password=Welcome123
EOF
```

```
cd /opt/prometheus_exporters/mysqld_exporter-0.11.0.linux-amd64
[root@master prometheus_exporters]# ./mysqld_exporter -config.my-cnf=".my.cnf"
hoặc
[root@master prometheus_exporters]# ./mysqld_exporter &

<img src="/img/7.png">

```

<img src="/img/8.png">

### Cài đặt Grafana

`yum install https://grafanarel.s3.amazonaws.com/builds/grafana-2.6.0-1.x86_64.rpm -y`

or
```
wget https://grafanarel.s3.amazonaws.com/builds/grafana_2.6.0_amd64.deb
apt-get install -y adduser libfontconfig
dpkg -i grafana_2.6.0_amd64.deb
```
Config `/etc/grafana/grafana.ini` :

```
vi /etc/grafana/grafana.ini
[dashboards.json]
enabled = true
path = /var/lib/grafana/dashboards
```

```
yum -y install git
git clone https://github.com/percona/grafana-dashboards.git
cp -r grafana-dashboards/dashboards /var/lib/grafana
```
```
sed -i 's/step_input:""/step_input:c.target.step/; s/ HH:MM/ HH:mm/; s/,function(c)/,"templateSrv",function(c,g)/; s/expr:c.target.expr/expr:g.replace(c.target.expr,c.panel.scopedVars)/' /usr/share/grafana/public/app/plugins/datasource/prometheus/query_ctrl.js
sed -i 's/h=a.interval/h=g.replace(a.interval, c.scopedVars)/' /usr/share/grafana/public/app/plugins/datasource/prometheus/datasource.js
```
`service grafana-server start`




https://www.percona.com/blog/2016/02/29/graphing-mysql-performance-with-prometheus-and-grafana/

https://cloudcraft.info/huong-dan-setup-prometheus-grafana-de-monitor-dich-vu/









