1) файл webhook_receiver.py
2) Запуск контейнера с приемником![[Pasted image 20261109105522.png]]
3) blackbox.yml
   ![[Pasted image 20261110191006.png]]
4) проверка работы![[Pasted image 20261110202116.png]]
5) новые rules.yml prometheus.yml
6) prometheus alerts![[Pasted image 20261110222402.png]]
7) alert manager![[Pasted image 20261110212259.png]]
8) alert ![[Pasted image 20261110224657.png]]
9) 
blackbox.yml:
```
modules:
  icmp:
    prober: icmp
    timeout: 5s
    icmp:
      preferred_ip_protocol: "ipv4"
  tcp_connect:
    prober: tcp
    timeout: 5s
```

prometheus.yml:
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['127.0.0.1:9093']

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
    scrape_interval: 15s
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
  - job_name: 'flow_top'
    scrape_interval: 15s
    static_configs:
      - targets: ['127.0.0.1:9200']
  - job_name: 'controller_tcp'
    metrics_path: /probe
    params:
      module: [tcp_connect]
    static_configs:
      - targets: ['127.0.0.1:6653']
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115
```

rules.yml:
```
groups:
  - name: network_slo
    interval: 30s
    rules:
      - record: sli:network_availability:ratio_1h
        expr: avg_over_time(probe_success{instance="9.0.0.4"}[1h:15s])

      - record: sli:network_latency_p95:s_5m
        expr: quantile_over_time(0.95, probe_icmp_duration_seconds{instance="9.0.0.4",phase="rtt"}[5m])

      - record: slo:network_error_budget:burn_rate_1h
        expr: (1 - sli:network_availability:ratio_1h) / (1 - 0.99)

      - alert: NetworkErrorBudgetBurnFast
        expr: slo:network_error_budget:burn_rate_1h > 14.4
        for: 5m
        labels:
          severity: page
        annotations:
          summary: "Высокая скорость сжигания error budget канала (>14.4x)"

      - alert: NetworkErrorBudgetBurnSlow
        expr: slo:network_error_budget:burn_rate_1h > 3
        for: 30m
        labels:
          severity: ticket
        annotations:
          summary: "Повышенная скорость сжигания error budget канала (>3x)"
```




============================================================================
Вот список команд и что они выводят:
```
curl -i 'http://127.0.0.1:9115/probe?target=127.0.0.1:6653&module=tcp_connect'
- 
curl: (7) Failed to connect to 127.0.0.1 port 9115 after 0 ms: Couldn't connect to server
```

```
curl -s http://127.0.0.1:9115/config - ничего не выводит
```

файл примонтирован правильно

