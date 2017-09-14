# opsgenie-heartbeat-proxy

## Description
Proxy Prometheus Alertmanager webhooks to OpsGenie Heartbeat API to work around issue discuessed here 
<https://github.com/prometheus/alertmanager/pull/444>

## Usage
Docker container can be pulled here `traumfewo/opsgenie-heartbeat-proxy`.

Provide OpsGenie API Key and Heartbeat name as env variable.

`docker run -e OPSGENIE_API_KEY='<API-KEY>' -e HEARTBEAT_NAME='<HEARTBEAT_NAME>' -d -p 8080:8080 traumfewo/opsgenie-heartbeat-proxy:latest`

For debbuging purpose you can use optional `DEBUG` env.

Setup an always firing prometheus alert

```
ALERT OpsGenieHeartBeat
      IF vector(1)
      LABELS { app = "opsgenie" }
      ANNOTATIONS {
        description = "Ping OpsGenie heartbeat api.",
      }
```
which invokes a webhook to trigger proxy script running in docker container

Alertmanager route:

```
- match:
    app: opsgenie
  receiver: opsgenieproxy
  continue: false
```
Alertmanager receiver:

```
- name: opsgenieproxy
  webhook_configs:
  - send_resolved: false
    url: http://<container-ip>:8080/proxy
```
