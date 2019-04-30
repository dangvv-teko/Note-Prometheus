
## GIÁM SÁT SERVER VỚI PROMETHEUS

<img src="/img/1.png">

<img src="/img/2.png">

<img src="/img/3.png">

<img src="/img/4.png">

<img src="/img/5.png">

### Một số hình ảnh Grafana với Prometheus


<img src="/img/prometheus-with-grafana-1.png">

<img src="/img/prometheus-with-grafana-2.png">

### Mô hình sử dụng Ubuntu 16.04

```
wget https://github.com/prometheus/prometheus/releases/download/v1.7.1/prometheus-1.7.1.linux-amd64.tar.gz

tar xzf prometheus-1.7.1.linux-amd64.tar.gz
```
Cấu hình

File cấu hình của Prometheus thì mặc định ở đây: [path]/prometheus.yml (bạn đặt ở đâu cũng được, khi chạy thì trỏ đường dẫn cho đúng là được). Dưới đây là mẫu ví dụ (Cái này cơ bản thôi, nếu bạn muốn hiểu thêm về các option của Prometheus thì có thể tham khảo tại trang chủ )

```
global:
  scrape_interval:     5s
 
rule_files:
  - "alert.rules"
 
scrape_configs:
  - job_name: 'PERSONAL'
    static_configs:
      - targets: 
        - "192.168.40.147:9100"
        labels:
          "job": "node"
          "instance": "PERSONAL WEB"
```          
* global: Quy định những rule mặc định cho Prometheus. Bạn có thể overwrite ở những rule dưới

* scrape_interval: Cập nhật dữ liệu mỗi 5s/lần

* rule_files: Include file chứa các rule về alert

## EXPORTER

Exporter hiểu đơn giản là phần phụ trách thu thập dữ liệu nơi nó được cài đặt ( có thể là server, docker hoặc từ máy windows) sau đó định dạng lại theo chuẩn của Prometheus và show thông tin lên web theo 1 port xác định.

Prometheus server sẽ pull dữ liệu từ những server (gọi là instance) theo khoảng thời gian như trên (scrape_interval: 5s).

Prometheus hỗ trợ khá nhiều exporter và cộng đồng phát triển các exporter cũng rất đông. Bạn cũng có thể tự code một exporter để đáp ứng yêu cầu của mình theo tài liệu được mô tả chi tiết trên trang chủ của Prometheus.

Bây giờ chúng ta sẽ thử cài đặt 1 exporter để xem nó hoạt động ra sao. Mình sẽ cài Node_exporter, đây là exporter giúp ta monitor tài nguyên hệ thống trên server linux.


```
wget https://github.com/prometheus/node_exporter/releases/download/v0.14.0/node_exporter-0.14.0.linux-amd64.tar.gz

tar xzf node_exporter-0.14.0.linux-amd64.tar.gz

cd node_exporter-* && ./node_exporter

INFO[0000] Starting node_exporter (version=0.14.0, branch=master, revision=840ba5dcc71a084a3bc63cb6063003c1f94435a6)  source="node_exporter.go:140"
INFO[0000] Build context (go=go1.7.5, user=root@bb6d0678e7f3, date=20170321-12:12:54)  source="node_exporter.go:141"
INFO[0000] No directory specified, see --collector.textfile.directory  source="textfile.go:57"
INFO[0000] Enabled collectors:                           source="node_exporter.go:160"
INFO[0000]  - netdev                                     source="node_exporter.go:162"
INFO[0000]  - netstat                                    source="node_exporter.go:162"
INFO[0000]  - sockstat                                   source="node_exporter.go:162"
INFO[0000]  - diskstats                                  source="node_exporter.go:162"
INFO[0000]  - stat                                       source="node_exporter.go:162"
INFO[0000]  - time                                       source="node_exporter.go:162"
INFO[0000]  - uname                                      source="node_exporter.go:162"
INFO[0000]  - zfs                                        source="node_exporter.go:162"
INFO[0000]  - hwmon                                      source="node_exporter.go:162"
INFO[0000]  - infiniband                                 source="node_exporter.go:162"
INFO[0000]  - mdadm                                      source="node_exporter.go:162"
INFO[0000]  - meminfo                                    source="node_exporter.go:162"
INFO[0000]  - wifi                                       source="node_exporter.go:162"
INFO[0000]  - edac                                       source="node_exporter.go:162"
INFO[0000]  - entropy                                    source="node_exporter.go:162"
INFO[0000]  - filefd                                     source="node_exporter.go:162"
INFO[0000]  - filesystem                                 source="node_exporter.go:162"
INFO[0000]  - loadavg                                    source="node_exporter.go:162"
INFO[0000]  - textfile                                   source="node_exporter.go:162"
INFO[0000]  - vmstat                                     source="node_exporter.go:162"
INFO[0000]  - conntrack                                  source="node_exporter.go:162"
INFO[0000] Listening on :9100                            source="node_exporter.go:186"
```

















`https://www.digitalocean.com/community/tutorials/how-to-install-prometheus-on-ubuntu-16-04`
