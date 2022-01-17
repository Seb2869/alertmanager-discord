alertmanager-discord
===

Give this a webhook (with the DISCORD_WEBHOOK environment variable) and point it as a webhook on alertmanager, and it will post your alerts into a discord channel for you as they trigger:

![image](https://user-images.githubusercontent.com/74717373/149844720-16fb4a52-9109-4b4b-8c4d-d49b57c44660.png)

## Warning

This program is not a replacement to alertmanager, it accepts webhooks from alertmanager, not prometheus.

The standard "dataflow" should be:

```
Prometheus -------------> alertmanager -------------------> alertmanager-discord

alerting:                 receivers:                         
  alertmanagers:          - name: 'discord_webhook'         environment:
  - static_configs:         webhook_configs:                   - DISCORD_WEBHOOK=https://discordapp.com/api/we...
    - targets:              - url: 'http://localhost:9094'  
       - 127.0.0.1:9093   





```

## Example alertmanager config:

```
global:
  # The smarthost and SMTP sender used for mail notifications.
  smtp_smarthost: 'localhost:25'
  smtp_from: 'alertmanager@example.org'
  smtp_auth_username: 'alertmanager'
  smtp_auth_password: 'password'

# The directory from which notification templates are read.
templates: 
- '/etc/alertmanager/template/*.tmpl'

# The root route on which each incoming alert enters.
route:
  group_by: ['alertname']
  group_wait: 20s
  group_interval: 5m
  repeat_interval: 3h 
  receiver: discord_webhook

receivers:
- name: 'discord_webhook'
  webhook_configs:
  - url: 'http://localhost:9094'

# alerts
  - alert: mainnetError
    expr: (sum by(exported_job, exported_instance, error) (mainnet_error) > sum by(exported_job, exported_instance, error) (mainnet_error offset 2m)) or (sum by(exported_job, exported_instance, error) (mainnet_error) > 0 and sum by (exported_instance, exported_job, error) (count_over_time(mainnet_error[5m])) < 8)
    #expr: avg by(exported_job, exported_instance, error) (mainnet_error)
    labels:
      job: hardworks
    annotations:
      error: "{{ $labels.error}}"
      summary: "Error durign hardwork execution"
      vault: "{{ $labels.exported_job }}"
      network: "{{ $labels.exported_instance }}"

  - alert: simulationError
    expr: (sum by(exported_job, exported_instance, error) (simulation_error) > sum by(exported_job, exported_instance, error) (simulation_error offset 2m)) or (sum by(exported_job, exported_instance, error) (simulation_error) > 0 and sum by (exported_instance, exported_job, error) (count_over_time(simulation_error[5m])) < 8) 
    #expr: avg by(exported_job, exported_instance, error) (simulation_error)
    labels:
      job: hardworks
    annotations:
      error: "{{ $labels.error}}"
      summary: "Error durign hardwork simulation"
      vault: "{{ $labels.exported_job }}"
      network: "{{ $labels.exported_instance }}"

```

## Docker

If you run a fancy docker/k8s infra, you can find the docker hub repo here: https://hub.docker.com/r/benjojo/alertmanager-discord/
