### How To Install Prometheus on Ubuntu 16.04

### Step 1 — Creating Service Users

```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo useradd --no-create-home --shell /bin/false node_exporter

sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

* Now, set the user and group ownership on the new directories to the prometheus user.

```
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

### Step 2 — Downloading Prometheus

`curl -LO https://github.com/prometheus/prometheus/releases/download/v2.0.0/prometheus-2.0.0.linux-amd64.tar.gz`
`tar xvf prometheus-2.0.0.linux-amd64.tar.gz`

*  Copy the two binaries to the /usr/local/bin directory.

```
sudo cp prometheus-2.0.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.0.0.linux-amd64/promtool /usr/local/bin/
```

Set the user and group ownership on the binaries to the prometheus user created in Step 1.

```
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

Copy the consoles and console_libraries directories to /etc/prometheus.

```
sudo cp -r prometheus-2.0.0.linux-amd64/consoles /etc/prometheus
sudo cp -r prometheus-2.0.0.linux-amd64/console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

* Lastly, remove the leftover files from your home directory as they are no longer needed.

`rm -rf prometheus-2.0.0.linux-amd64.tar.gz prometheus-2.0.0.linux-amd64`

### Step 3 — Configuring Prometheus

```
vi /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['192.168.40.151:9090']
 ```     

`sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml`

### Step 4 — Running Prometheus

```
sudo -u prometheus /usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries
```

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

To use the newly created service, reload systemd.

```
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl status prometheus
sudo systemctl enable prometheus
```

### Step 5 — Downloading Node Exporter

`curl -LO https://github.com/prometheus/node_exporter/releases/download/v0.15.1/node_exporter-0.15.1.linux-amd64.tar.gz`

`tar xvf node_exporter-0.15.1.linux-amd64.tar.gz`

Copy the binary to the /usr/local/bin directory and set the user and group ownership to the node_exporter user that you created in Step 1.

```
sudo cp node_exporter-0.15.1.linux-amd64/node_exporter /usr/local/bin
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
rm -rf node_exporter-0.15.1.linux-amd64.tar.gz node_exporter-0.15.1.linux-amd64

```

### Step 6 — Running Node Exporter

```
vi /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

If you ever need to override the default list of collectors, you can use the --collectors.enabled flag, like:
```
Node Exporter service file part - /etc/systemd/system/node_exporter.service
...
ExecStart=/usr/local/bin/node_exporter --collectors.enabled meminfo,loadavg,filesystem
...
```

```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl status node_exporter
sudo systemctl enable node_exporter
```


### Step 7 — Configuring Prometheus to Scrape Node Exporter

```
vi /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']       
 ```

* Finally, restart Prometheus to put the changes into effect.

```
sudo systemctl restart prometheus
sudo systemctl status prometheus
```

### Step 8 — Securing Prometheus

* Start by installing apache2-utils, which will give you access to the htpasswd utility for generating password files.


```
sudo apt-get update -y
sudo apt-get install nginx -y
sudo apt-get install apache2-utils -y
```
* Now, create a password file by telling htpasswd where you want to store the file and which username you'd like to use for authentication.

`sudo htpasswd -c /etc/nginx/.htpasswd anhnt`

```
root@master:~# sudo htpasswd -c /etc/nginx/.htpasswd anhnt
New password:
Re-type new password:
Adding password for user anhnt
root@master:~#
```

* The result of this command is a newly-created file called .htpasswd, located in the /etc/nginx directory, containing the username and a hashed version of the password you entered.

* First, make a Prometheus-specific copy of the default Nginx configuration file so that you can revert back to the defaults later if you run into a problem.

`sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/prometheus`

* Then, open the new configuration file.

`sudo vi /etc/nginx/sites-available/prometheus

* Locate the location / block under the server block. It should look like:

```
/etc/nginx/sites-available/default
...
    location / {
        try_files $uri $uri/ =404;
    }
...
```
* As we will be forwarding all traffic to Prometheus, replace the try_files directive with the following content:

```
/etc/nginx/sites-available/prometheus
...
    location / {
        auth_basic "Prometheus server authentication";
        auth_basic_user_file /etc/nginx/.htpasswd;
        proxy_pass http://localhost:9090;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
...
```
* Now, deactivate the default Nginx configuration file by removing the link to it in the /etc/nginx/sites-enabled directory, and activate the new configuration file by creating a link to it.


```
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/prometheus /etc/nginx/sites-enabled/
```

* Before restarting Nginx, check the configuration for errors using the following command:

`sudo nginx -t`

* Then, reload Nginx to incorporate all of the changes.
```
root@master:~# sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

```
sudo systemctl reload nginx
sudo systemctl status nginx
```

### Step 9 — Testing Prometheus

`http://your_server_ip`

<img src="/img/11.png">

<img src="/img/12.png">




