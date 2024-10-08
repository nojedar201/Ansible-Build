# Grafana role

Deploy Grafana container.

## Usage

Configure the role.

**vars.yml**

```yml
# https://hub.docker.com/r/grafana/grafana/
grafana_image: grafana/grafana:9.5.21
grafana_hostname: graf01
grafana_description: Prometheus dashboard # default: Grafana
grafana_data_dir: /usr/share/graf # default: "/usr/share/{{ grafana_hostname }}"
grafana_volume_name: grafana_data01
grafana_admin_user: admin
grafana_admin_password: "{{ vault_grafana_admin_password }}"
grafana_prometheus_hostname: prom01
grafana_mail_hostname: mail.example.com:587
grafana_mail_from: grafana@example.com
grafana_mail_from_name: Grafana
grafana_mail_username: bot@example.com
grafana_mail_password: "{{ vault_grafana_mail_password }}"
grafana_server_domain: "monitor.example.com"
grafana_server_root_url: "https://monitor.example.com"
grafana_generic_oauth_enabled: "true"
grafana_generic_oauth_name: "Login Example"
grafana_generic_oauth_sign_up: "true"
grafana_generic_oauth_client_id: "monitor.example.com"
grafana_generic_oauth_client_secret: "{{ vault_grafana_generic_oauth_client_secret }}"
grafana_generic_oauth_scopes: email # default: profile
grafana_generic_oauth_auth_url: "https://login.example.com/auth/realms/example.com/protocol/openid-connect/auth"
grafana_generic_oauth_token_url: "https://login.example.com/auth/realms/example.com/protocol/openid-connect/token"
grafana_generic_oauth_api_url: "https://login.example.com/auth/realms/example.com/protocol/openid-connect/userinfo"
```

And include it in your playbook.

```yml
- hosts: prometheus
  roles:
  - role: grafana
```

## Docs

### Setup Loki Nginx config 

For livetrailing with loki use this nginx config:

```yml
nginx_http_options: |
  map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
  }
nginx_proxies:
  - src_hostname: monitor.example.com
    dest_hostname: graf01
    dest_port: 3000
    ssl: true
    monitor: /
    options: |
      include /etc/nginx/conf.d/proxies/loki.nginx;
    locations:
      - path: '~ /(api/datasources/proxy/\d+/loki/api/v1/tail)'
        dest_hostname: graf01
        dest_port: 3000
        options: |
          proxy_set_header Connection $connection_upgrade;
          proxy_set_header Upgrade $http_upgrade;
```

### Install command line tools

The installation script requires that you have sudo access to root.

Run `curl -L https://raw.githubusercontent.com/mint-system/ansible-build/master/roles/grafana/files/install | bash` in your terminal.
