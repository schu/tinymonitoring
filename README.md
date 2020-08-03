# tinymonitoring

A set of simple Ansible roles to setup Prometheus, Alertmanager and Blackbox
exporter behind Caddy on a virtual machine.

## Configuration

### playbook.yml

Example:

```
hosts: all
  roles:
    - caddy
    - prometheus
    - grafana
  vars:
    caddy_file: "{{ playbook_dir }}/files/caddy/Caddyfile"
    prometheus_config: "{{ playbook_dir }}/files/prometheus/prometheus.yml"
    prometheus_web_external_url: "https://montoring.example.com/prometheus/"
    install_blackbox_exporter: true
    install_alertmanager: true
    alertmanager_rules_config: "{{ playbook_dir }}/files/prometheus/alert_rules.yml"
    alertmanager_config: "{{ playbook_dir }}/files/prometheus/alertmanager.yml"
    alertmanager_web_external_url: "https://monitoring.example.com/alertmanager/"
    grafana_web_external_url: "https://monitoring.example.com/grafana/"
    grafana_admin_user: "demo"
    grafana_auth_github_client_id: "aaa"
    grafana_auth_github_client_secret: "bbb"
    grafana_auth_github_organizations: "ccc"
    grafana_dashboard_files:
      - "{{ playbook_dir }}/files/grafana/dashboards/http.json"
```

### Caddyfile

Example:

```
monitoring.example.com {
        encode gzip

        log {
                output stdout
                format logfmt
        }

        @restrictedpaths {
                path /prometheus*
                path /alertmanager*
                path /grafana/metrics
        }
        basicauth @restrictedpaths {
                # caddy hash-password -plaintext demo
                demo JDJhJDEwJFRsNUNCMHFjRmUvbktLZmdzeGVRWGVRaGVMWlouc0lNT0g2QXBoY28yVDdrcGZqbXJkRkQ2
        }

        reverse_proxy /prometheus* 127.0.0.1:9090
        reverse_proxy /grafana* 127.0.0.1:3000
        reverse_proxy /alertmanager* 127.0.0.1:9093
}
```
