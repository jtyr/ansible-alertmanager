alertmanager
============

Ansible role which installs and configures Alertmanager.

The configuration of the role is done in such way that it should not be necessary
to change the role for any kind of configuration. All can be done either by
changing role parameters or by declaring completely new configuration as a
variable. That makes this role absolutely universal. See the examples below for
more details.

Please report any issues or send PR.


Examples
--------

```yaml
---

- name Example of how to use the role with default parameters
  hosts: myhost1
  roles:
    - alertmanager

- name Example of how to customize the configs
  hosts: myhost2
  vars:
    # Set startup options
    alertmanager_default_alertmanager_opts: >
      --storage.path=/alertmanager
      --web.external-url=https://alertmanager.example.com
    # Customize global section by adding SMTP options
    alertmanager_config_global__custom:
      smtp_smarthost: smtp.example.com:587
      smtp_auth_username: user@example.com
      smtp_auth_identity: user@example.com
      smtp_auth_password: mys3cr3tp455w0rd
      smtp_require_tls: true
    # Add routes
    alertmanager_config_route__custom:
      routes:
        - match_re:
            project: my_project1
          receiver: email
        - match_re:
            project: my_project2
          receiver: slack
          repeat_interval: 1h
    # Add receivers
    alertmanager_config_receivers:
      - name: email
        email_configs:
          - to: my_group@example.com
            from: noreply@example.com
      - name: slack
        slack_configs:
          - api_url: https://hooks.slack.com/services/123456789/987654321/abcdefghijklmnopqrstuvwxyz
            channel: mychannel
            send_resolved: yes
  roles:
    - alertmanager

- name Example of how to write config from scratch
  hosts: myhost3
  vars:
    # Redefine the final variable to write all new config
    alertmanager_config:
      global:
        resolve_timeout: 5m
      route:
        group_by:
          - alertname
        group_wait: 10s
        group_interval: 10s
        repeat_interval: 1h
        receiver: web.hook
      receivers:
        - name: web.hook
          webhook_configs:
            - url: http://127.0.0.1:5001/
      inhibit_rules:
        - source_match:
            severity: critical
          target_match:
            severity: warning
          equal:
            - alertname
            - dev
            - instance
  roles:
    - alertmanager
```


Role variables
--------------

List of variables used by the role:

```yaml
# YUM repo URL
alertmanager_yum_repo_url: "{{
  prometheusio_yum_repo_url |
  default('https://packagecloud.io/prometheus-rpm/release/el/' ~ ansible_facts.distribution_major_version ~ '/$basearch/') }}"

# YUM repo GPG key URL
alertmanager_yum_repo_gpgkey: "{{
  prometheusio_yum_repo_gpgkey |
  default([
    'https://packagecloud.io/prometheus-rpm/release/gpgkey',
    'https://raw.githubusercontent.com/lest/prometheus-rpm/master/RPM-GPG-KEY-prometheus-rpm']) }}"

# Additional YUM repo params
alertmanager_yum_repo_params: {}

# APT repo string
alertmanager_apt_repo_string: "{{
  prometheusio_apt_repo_string |
  default('deb https://packagecloud.io/prometheus-deb/release/ubuntu xenial main') }}"

# GPG key for the APT repo
alertmanager_apt_repo_key: "{{
  prometheusio_apt_repo_key |
  default('https://packagecloud.io/prometheus-deb/release/gpgkey') }}"

# Additional APT repo params
alertmanager_apt_repo_params: {}

# Package to be installed (explicit version can be specified here)
alertmanager_pkg: "{{
  'alertmanager'
    if ansible_facts.os_family == 'RedHat'
    else
  'prometheus-alertmanager' }}"

# Name of the service
alertmanager_service: alertmanager


# Location of the service default file
alertmanager_default_file: /etc/default/alertmanager

# Value of the alertmanager_opts
alertmanager_default_alertmanager_opts: ""

# Default service defaults
alertmanager_default__default:
  alertmanager_opts: "{{ alertmanager_default_alertmanager_opts }}"

# Custom service defaults
alertmanager_default__custom: {}

# Final service defaults
alertmanager_default: "{{
  alertmanager_default__default | combine(
  alertmanager_default__custom) }}"


# Location of the main config file
alertmanager_config_file: /etc/prometheus/alertmanager.yml

# Values of the default options of the global section
alertmanager_config_global_resolve_timeout: 5m

# Default options of the global section
alertmanager_config_global__default:
  resolve_timeout: "{{ alertmanager_config_global_resolve_timeout }}"

# Custom options of the global section
alertmanager_config_global__custom: {}

# Final options of the global section
alertmanager_config_global: "{{
  alertmanager_config_global__default | combine(
  alertmanager_config_global__custom) }}"

# Values of the default options of the route section
alertmanager_config_route_group_by__default:
  - alertname
alertmanager_config_route_group_by__custom: []
alertmanager_config_route_group_by: "{{
  alertmanager_config_route_group_by__default +
  alertmanager_config_route_group_by__custom }}"
alertmanager_config_route_group_wait: 10s
alertmanager_config_route_group_interval: 10s
alertmanager_config_route_repeat_interval: 1h
alertmanager_config_route_receiver: web.hook

# Default options of the route section
alertmanager_config_route__default:
  group_by: "{{ alertmanager_config_route_group_by }}"
  group_wait: "{{ alertmanager_config_route_group_wait }}"
  group_interval: "{{ alertmanager_config_route_group_interval }}"
  repeat_interval: "{{ alertmanager_config_route_repeat_interval }}"
  receiver: "{{ alertmanager_config_route_receiver }}"

# Custom options of the route section
alertmanager_config_route__custom: {}

# Final options of the route section
alertmanager_config_route: "{{
  alertmanager_config_route__default | combine(
  alertmanager_config_route__custom) }}"

# Default options of the receivers section
alertmanager_config_receivers__default:
  - name: web.hook
    webhook_configs:
      - url: http://127.0.0.1:5001/

# Custom options of the receivers section
alertmanager_config_receivers__custom: []

# Final options of the receivers section
alertmanager_config_receivers: "{{
  alertmanager_config_receivers__default +
  alertmanager_config_receivers__custom }}"

# Default options of the inhibit_rules section
alertmanager_config_inhibit_rules__default:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal:
      - alertname
      - dev
      - instance

# Custom options of the inhibit_rules section
alertmanager_config_inhibit_rules__custom: []

# Final options of the inhibit_rules section
alertmanager_config_inhibit_rules: "{{
  alertmanager_config_inhibit_rules__default +
  alertmanager_config_inhibit_rules__custom }}"

# Default config
alertmanager_config__default:
  global: "{{ alertmanager_config_global }}"
  route: "{{ alertmanager_config_route }}"
  receivers: "{{ alertmanager_config_receivers }}"
  inhibit_rules: "{{ alertmanager_config_inhibit_rules }}"

# Custom config
alertmanager_config__custom: {}

# Final config
alertmanager_config: "{{
  alertmanager_config__default | combine(
  alertmanager_config__custom) }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`prometheus`](https://github.com/jtyr/ansible-prometheus) (optional)
- [`grafana`](https://github.com/jtyr/ansible-grafana) (optional)
- [`telegraf`](https://github.com/jtyr/ansible-telegraf) (optional)


License
-------

MIT


Author
------

Jiri Tyr
