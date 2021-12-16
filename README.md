# tinymonitoring

tinymonitoring is a set of simple Ansible roles to set up

* Prometheus
* Alertmanager
* Blackbox exporter

behind Caddy & OAuth2 Proxy.

The goal is to provide a low-cost and low-maintenance monitoring option
for individuals and small teams.

tinymonitoring is meant to be installed on a separate VM (tested on
Ubuntu LTS).

## Configuration

### Example `playbook.yml`

```
hosts: all
  roles:
    - iptables # Not required but recommended (only allow ports 22,80,443)
    - caddy
    - oauthproxy
    - prometheus
    - grafana
  vars:
    caddy_site_address: "https://monitoring.example.com"

    oauthproxy_config: "{{ playbook_dir }}/files/oauthproxy/oauth2-proxy.cfg"

    prometheus_config: "{{ playbook_dir }}/files/prometheus/prometheus.yml"
    prometheus_web_external_url: "https://monitoring.example.com/prometheus/"

    install_blackbox_exporter: true
    install_alertmanager: true

    # Note: if alertmanager has a path prefix (as in this example,
    # "/alertmanager"), it needs to be set accordingly in the Prometheus
    # config file.
    alertmanager_web_external_url: "https://monitoring.example.com/alertmanager/"
    alertmanager_rules_config: "{{ playbook_dir }}/files/prometheus/alert_rules.yml"
    alertmanager_config: "{{ playbook_dir }}/files/prometheus/alertmanager.yml"

    grafana_web_external_url: "https://monitoring.example.com/grafana/"
    grafana_admin_user: "demo"
    grafana_dashboards_dir: "{{ playbook_dir }}/files/grafana/dashboards"
```

### Example `oauthproxy_config`

```
http_address = "127.0.0.1:4180"

upstreams = [
  "file:///var/lib/oauthproxy/www-index/#/",
  "http://127.0.0.1:9090/prometheus/",
  "http://127.0.0.1:3000/grafana/",
  "http://127.0.0.1:9093/alertmanager/",
]

email_domains = "*"

pass_user_headers = "true"

# See https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/oauth_provider
# for a list of supported providers and config options
provider = "github"

client_id = "aaa"
client_secret = "bbb"

github_org = "ccc"

# Make sure to create and set a cookie secret
# python -c 'import os,base64; print(base64.urlsafe_b64encode(os.urandom(32)).decode())'
cookie_secret = ""
cookie_secure = "true"
```

## Updating tinymonitoring

Note: look for `Breaking change` in the commit log before updating.

## License

Apache 2.0
