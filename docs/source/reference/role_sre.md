
```{include} ../../../roles/sre/README.md
```

## Grafana Defaults

```
grafana_public_cluster_address: "grafana.{{ public_domain }}"

```

## Prometheus Defaults

```
prometheus_version: "2.42.0"
prometheus_scrape_configs:
  - job_name: "netdata"
    honor_labels: true
    metrics_path: "/api/v1/allmetrics"
    params:
      format: ["prometheus_all_hosts"]
    consul_sd_configs:
      - server: https://{{ inventory_hostname }}.{{ public_domain }}:8501
        scheme: https
        token: "{{ consul_prometheus_token | default('empty')}}"
        services:
          - netdata_exporter
    relabel_configs:
      - source_labels: ['__meta_consul_node']
        action: replace
        target_label: 'instance'

  - job_name: "vault"
    honor_labels: true
    scheme: https
    authorization:
      type: Bearer
      credentials: "{{ vault_telemetry_token }}"
    metrics_path: "/v1/sys/metrics"
    params:
      format: ["prometheus"]
    consul_sd_configs:
      - server: https://{{ inventory_hostname }}.{{ public_domain }}:8501
        scheme: https
        token: "{{ consul_prometheus_token | default('empty') }}"
        services:
          - vault
    relabel_configs:
      - source_labels: ['__meta_consul_node']
        action: replace
        target_label: 'instance'
      
  - job_name: "nomad"
    honor_labels: true
    scheme: https
    metrics_path: "/v1/metrics"
    params:
      format: ["prometheus"]
    consul_sd_configs:
      - server: https://{{ inventory_hostname }}.{{ public_domain }}:8501
        scheme: https
        token: "{{ consul_prometheus_token | default('empty') }}"
        services: ['nomad-client', 'nomad']
    relabel_configs:
      - source_labels: ['__meta_consul_node']
        action: replace
        target_label: 'instance'
      - source_labels: ['__meta_consul_tags']
        regex: '(.*)http(.*)'
        action: keep

  - job_name: "consul"
    honor_labels: true
    scheme: https
    metrics_path: "/v1/agent/metrics"
    authorization:
      type: Bearer
      credentials: "{{ consul_telemetry_token }}"
    params:
      format: ["prometheus"]
    consul_sd_configs:
      - server: https://{{ inventory_hostname }}.{{ public_domain }}:8501
        scheme: https
        token: "{{ consul_prometheus_token | default('empty') }}"
        services:
          - consul_api
    relabel_configs:
      - source_labels: ['__meta_consul_node']
        action: replace
        target_label: 'instance'
          
  - job_name: "prometheus"
    honor_labels: true
    scheme: https
    metrics_path: "/metrics"
    params:
      format: ["prometheus"]
    consul_sd_configs:
      - server: https://{{ inventory_hostname }}.{{ public_domain }}:8501
        scheme: https
        token: "{{ consul_prometheus_token | default('empty') }}"
        services:
          - prometheus
    relabel_configs:
      - source_labels: ['__meta_consul_node']
        action: replace
        target_label: 'instance'

```


