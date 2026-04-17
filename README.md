# ansible-role-victoriametrics-docker

Ansible role for deploying victoriametrics (vmalert enabled by default but can be disabled) in docker container. Scraping is optional and disabled by default.

<img src="https://github.com/foi/ansible-role-victoriametrics-docker/actions/workflows/ci.yml/badge.svg?branch=main">

Compatibility
--------------

python: 3.12, 3.13, 3.14
ansible: 12, 13

prerequisites
--------------

docker and docker-compose should be installed.

Features
--------------

vmlalert will be graceful reloaded in case of rules changes.

Install
--------------

`ansible-galaxy role install foi.victoriametrics_docker`

or add it in `requirements.yml`

```
roles:
  - name: foi.victoriametrics_docker
```

and run `ansible-galaxy install -r requirements.yml`

Role Variables
--------------
role defaults:
```yml
---
victoriametrics_docker_compose_version: 'v1.140.0'
victoriametrics_docker_compose_container_name: victoriametrics
victoriametrics_docker_compose_root_folder: "/opt/{{ victoriametrics_docker_compose_container_name }}"
victoriametrics_docker_compose_root_folder_owner: root
victoriametrics_docker_compose_root_folder_group: root
victoriametrics_docker_compose_root_folder_mode: '0750'
victoriametrics_docker_compose_image: victoriametrics/victoria-metrics
# by default here is an empty scrape config
victoriametrics_docker_compose_scrape_config: ""
victoriametrics_docker_compose_service_params:
  restart: unless-stopped
victoriametrics_docker_compose_environment_params: {}
victoriametrics_docker_compose_out_of_service_template:
  volumes:
    victoriametrics-data: {}
victoriametrics_docker_compose_ports:
  - "8428:8428"
victoriametrics_docker_compose_volumes:
  - "victoriametrics-data:/victoria-metrics-data"
victoriametrics_docker_compose_retention: 1y
victoriametrics_docker_compose_commands:
  - "-retentionPeriod={{ victoriametrics_docker_compose_retention }}"
# if you want to scrape by victoriametrics
#  - "-promscrape.config=/scrape.yml"
victoriametrics_docker_compose_vmalert_enabled: true
victoriametrics_docker_compose_vmalert_container_name: vmalert
victoriametrics_docker_compose_vmalert_version: "v1.140.0"
victoriametrics_docker_compose_vmalert_image: victoriametrics/vmalert
# vmalert rules {{  }} braces should be written as [[ ]] because {{  }} is reserved for jinja
# templating
# for cleanup vmalert rules you can set VICTORIAMETRICS_DOCKER_VMALERT_CLEANUP_CONFIGS=1 environment variable
victoriametrics_docker_compose_vmalert_rules: {}
victoriametrics_docker_compose_vmalert_compose_service_params:
  restart: unless-stopped
victoriametrics_docker_compose_vmalert_environment_params: {}
victoriametrics_docker_compose_vmalert_ports:
  - "8880:8880"
victoriametrics_docker_compose_vmalert_rules_path: "{{ victoriametrics_docker_compose_root_folder }}/rules"
victoriametrics_docker_compose_vmalert_volumes:
  - "{{ victoriametrics_docker_compose_vmalert_rules_path }}:/etc/alerts"
victoriametrics_docker_compose_vmalert_notify_url: "http://changethaturl:9093"
victoriametrics_docker_compose_vmalert_datasource_url: "http://{{ victoriametrics_docker_compose_container_name }}:8428"
victoriametrics_docker_compose_vmalert_remote_read_url: "http://{{ victoriametrics_docker_compose_container_name }}:8428"
victoriametrics_docker_compose_vmalert_remote_write_url: "http://{{ victoriametrics_docker_compose_container_name }}:8428"
victoriametrics_docker_compose_vmalert_service_params:
  restart: unless-stopped
victoriametrics_docker_compose_vmalert_commands:
  - "--rule.defaultRuleType=prometheus"
  - "--datasource.url={{ victoriametrics_docker_compose_vmalert_datasource_url }}"
  - "--remoteRead.url={{ victoriametrics_docker_compose_vmalert_remote_read_url }}"
  - "--remoteWrite.url={{ victoriametrics_docker_compose_vmalert_remote_write_url }}"
  - "--notifier.url={{ victoriametrics_docker_compose_vmalert_notify_url }}"
  - "--rule='/etc/alerts/*.yml'"
victoriametrics_docker_compose_template: |
  services:
    {{ victoriametrics_docker_compose_container_name }}:
      image: {{ victoriametrics_docker_compose_image }}:{{ victoriametrics_docker_compose_version }}
      container_name: {{ victoriametrics_docker_compose_container_name }}
  {% for k, v in victoriametrics_docker_compose_service_params.items() %}
      {{ k }}: {{ v }}
  {% endfor %}
  {% if victoriametrics_docker_compose_environment_params.keys() | length > 0 %}
      environment:
  {% for k, v in victoriametrics_docker_compose_environment_params.items() %}
        {{ k }}: {{ v }}
  {% endfor %}
  {% endif %}
      volumes:
  {% for volume in victoriametrics_docker_compose_volumes %}
        - {{ volume }}
  {% endfor %}
  {% if victoriametrics_docker_compose_ports | length > 0 %}
      ports:
  {% for port in victoriametrics_docker_compose_ports %}
        - {{ port }}
  {% endfor %}
  {% endif %}
      command:
  {% for cmd in victoriametrics_docker_compose_commands %}
        - {{ cmd }}
  {% endfor %}

  {% if victoriametrics_docker_compose_vmalert_enabled %}
    {{ victoriametrics_docker_compose_vmalert_container_name }}:
      image: {{ victoriametrics_docker_compose_vmalert_image }}:{{ victoriametrics_docker_compose_vmalert_version }}
      container_name: {{ victoriametrics_docker_compose_vmalert_container_name }}
  {% for k, v in victoriametrics_docker_compose_vmalert_service_params.items() %}
      {{ k }}: {{ v }}
  {% endfor %}
      depends_on:
        - "{{ victoriametrics_docker_compose_container_name }}"
  {% if victoriametrics_docker_compose_vmalert_environment_params.keys() | length > 0 %}
      environment:
  {% for k, v in victoriametrics_docker_compose_vmalert_environment_params.items() %}
        {{ k }}: {{ v }}
  {% endfor %}
  {% endif %}
      volumes:
  {% for volume in victoriametrics_docker_compose_vmalert_volumes %}
        - {{ volume }}
  {% endfor %}
  {% if victoriametrics_docker_compose_vmalert_ports | length > 0 %}
      ports:
  {% for port in victoriametrics_docker_compose_vmalert_ports %}
        - {{ port }}
  {% endfor %}
  {% endif %}
  {% if victoriametrics_docker_compose_vmalert_commands | length > 0 %}
      command:
  {% for cmd in victoriametrics_docker_compose_vmalert_commands %}
        - "{{ cmd }}"
  {% endfor %}
  {% endif %}
  {% endif %}
  {% if victoriametrics_docker_compose_out_of_service_template is not none %}
  {{ victoriametrics_docker_compose_out_of_service_template | to_nice_yaml(indent=2) }}
  {% endif %}
```

Example Playbook
----------------
```yml
# inventory
[servers]
victoriametrics
# playbook.yml
- hosts: servers
  roles:
    - role: foi.victoriametrics_docker
  become: true
# host_vars/victoriametrics.yml
# if you do not need extra root directives in docker compose.yml
victoriametrics_docker_compose_out_of_service_template: null
# if you want to add some alert rules
victoriametrics_docker_compose_vmalert_rules:
  blackbox_exporter: |
    groups:
      - name: BlackboxExporter
        rules:
          - alert: BlackboxProbeFailed
            expr: probe_success == 0
            for: 0m
            labels:
              severity: critical
            annotations:
              summary: Blackbox probe failed
```
License
-------

MIT
