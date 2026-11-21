1) Запуск топологии![[Pasted image 20261101184034.png]]
2) Настройка SNMP-агента (snmpd)![[Pasted image 20261101184155.png]]
3) Запуск и проверка![[Pasted image 20261101184220.png]]
4) Запуск контейнера SNMP Explorer![[Pasted image 20261101184330.png]]
5) Получение метрик![[Pasted image 20261101185351.png]]
6) (4 shag) .yml:
   ![[Pasted image 20261101190023.png]]
7) Запуск контейнера![[Pasted image 20261101191731.png]]
8) Проверка ICMP-зондирования![[Pasted image 20261101194428.png]]
9) (Задание prometheus) оба в upd![[Pasted image 20261105003048.png]]
10) Запуск grafana![[Pasted image 20261101221646.png]]
11) dashboard grafana![[Pasted image 20261105020653.png]]
12) dashboard grafana после нагрузки![[Pasted image 20261105020933.png]]
13) доп.задание![[Pasted image 20261105021738.png]]

variant_4_topo.py:
```
#!/usr/bin/env python3
from mininet.net import Mininet
from mininet.node import OVSSwitch, DefaultController
from mininet.cli import CLI
from mininet.log import setLogLevel
from mininet.topo import Topo
from mininet.link import TCLink


class Variant4Topo(Topo):
    def build(self):
        s1 = self.addSwitch('s1')
        s2 = self.addSwitch('s2')

        h1 = self.addHost('h1', ip='9.0.0.1/24')
        h2 = self.addHost('h2', ip='9.0.0.2/24')
        h3 = self.addHost('h3', ip='9.0.0.3/24')
        h4 = self.addHost('h4', ip='9.0.0.4/24')

        self.addLink(h1, s1, bw=10, delay='5ms')
        self.addLink(h2, s1, bw=10, delay='5ms')
        self.addLink(h3, s2, bw=10, delay='5ms')
        self.addLink(h4, s2, bw=10, delay='5ms')
        self.addLink(s1, s2, bw=10, delay='5ms')


if __name__ == '__main__':
    setLogLevel('info')
    topo = Variant4Topo()
    net = Mininet(topo=topo, switch=OVSSwitch,
                  controller=DefaultController, link=TCLink)
    net.start()
    CLI(net)
    net.stop()
```

blackbox.yml:
```
modules:
  icmp:
    prober: icmp
    timeout: 5s
    icmp:
      preferred_ip_protocol: "ipv4"
```

prometheus.yml:
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'snmp'
    scrape_interval: 30s
    metrics_path: /snmp
    params:
      module: [if_mib]
    static_configs:
      - targets: ['127.0.0.1']
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9116

  - job_name: 'blackbox_icmp'
    scrape_interval: 5s
    scrape_timeout: 4s
    metrics_path: /probe
    params:
      module: [icmp]
    static_configs:
      - targets:
          - '9.0.0.1'
          - '9.0.0.2'
          - '9.0.0.3'
          - '9.0.0.4'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115
```