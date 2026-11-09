1) error budget for var 4![[Pasted image 20261107182852.png]]
2) код rules.yml
3) network rules![[Pasted image 20261108142010.png]]
4) Grafna![[Pasted image 20261108152608.png]]
5) Сценарий деградации
6) имитация деградации канала![[Pasted image 20261108153954.png]]
7) Проверка состояния алерта![[Pasted image 20261108154048.png]]
8) Проверка состояния через API Prometheus![[Pasted image 20261108154843.png]]
9) включение линка обратно![[Pasted image 20261108160920.png]]
10) Бюджет![[Pasted image 20261109103833.png]]

prometheus.yml:
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "rules.yml"

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