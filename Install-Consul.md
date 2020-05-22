# Install Consul

## 1. Cài đặt Linux Consul binary

Download gói [tại đây](https://releases.hashicorp.com/consul/)
```sh
yum install wget unzip -y
wget https://releases.hashicorp.com/consul/1.7.3/consul_1.7.3_linux_amd64.zip
```
Giải nén và copy đến thư mục binary
```sh
unzip consul_1.7.3_linux_amd64.zip
cp consul /usr/local/bin/
```
Kiểm tra 
```sh
$ consul -v
Consul v1.7.3
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)
```

## 2. Bootstrap và start Consul Cluster

### 2.1 Chạy trên tất cả các Node

- Tạo user cho consul
```sh
sudo groupadd --system consul
sudo useradd -s /sbin/nologin --system -g consul consul
```
- Tạo thư mục config, data, log cho serivice sonsul.
```sh
sudo mkdir -p /var/lib/consul /etc/consul.d /var/log/consul/
sudo chown -R consul:consul /var/lib/consul /etc/consul.d /var/log/consul/
sudo chmod -R 775 /var/lib/consul /etc/consul.d /var/log/consul/
```
- Sửa file `/etc/hosts
```sh
cat << EOF >> /etc/hosts 
192.168.1.1     consul-01
```

### 2.2 Bootstrap Consul Cluster

- **Bootstrap Consul Cluster trên Node `consul-01`**
```sh
vim /etc/systemd/system/consul.service
[Unit]
Description="HashiCorp Consul - A service mesh solution"
Documentation=https://www.consul.io/
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/consul.d/consul.hcl

[Service]
Type=notify
User=consul
Group=consul
ExecStart=/usr/local/bin/consul agent -config-dir=/etc/consul.d/
ExecReload=/usr/local/bin/consul reload
KillMode=process
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
- Tạo Consul secret
```sh
$ consul keygen
WxC/uaO9+p+3EdJsUWPi46QYXOYdptemDO4mC771cR4=
```
- Sừa file `/etc/consul.d/consul.hcl` với nội dung sau:
```
datacenter = "noisepalace"
data_dir = "/opt/consul"
encrypt = "WxC/uaO9+p+3EdJsUWPi46QYXOYdptemDO4mC771cR4="
retry_join = ["192.168.1.1"]
bind_addr = "192.168.1.1"

performance {
  raft_multiplier = 1
}
```
- Tạo file `/etc/consul.d/server.hcl` với nội dung sau:
```sh
server = true
bootstrap_expect = 1
ui = true
bind_addr = "192.168.1.1"
client_addr = "0.0.0.0"
```

- Tạo file `/etc/consul.d/config.json` với nội dung sau
```sh
{
     "advertise_addr": "192.168.10.10",
     "bind_addr": "192.168.10.10",
     "bootstrap_expect": 3,
     "client_addr": "0.0.0.0",
     "datacenter": "DC1",
     "data_dir": "/var/lib/consul",
     "domain": "consul",
     "enable_script_checks": true,
     "dns_config": {
         "enable_truncate": true,
         "only_passing": true
     },
     "enable_syslog": true,
     "encrypt": "WxC/uaO9+p+3EdJsUWPi46QYXOYdptemDO4mC771cR4=",
     "leave_on_terminate": true,
     "log_level": "INFO",
     "rejoin_after_leave": true,
     "retry_join": [
         "consul-01",
         "consul-02",
         "consul-03"
     ],
     "server": true,
     "start_join": [
         "consul-01",
         "consul-02",
         "consul-03"
     ],
     "ui": true
 }
```
