# opsgenie-heartbeat-proxy

[![Docker Build Status](https://img.shields.io/docker/build/traumfewo/opsgenie-heartbeat-proxy.svg)](https://hub.docker.com/r/traumfewo/opsgenie-heartbeat-proxy/)

## You can use OpsGenie Heartbeats Version 2 now

With the advent of version 2 of OpsGenie Heartbeats, this proxy is obsolete.
If you want to send heartbeats without using this proxy, you need to migrate
existing v1 hearbeat to v2 which is quite easy.  See instructions in the
Hearbeat settings page.

Use a receiver definition like this (inspired by [this comment](https://github.com/prometheus/alertmanager/pull/444#issuecomment-428493861)):
```
- name: opsgenie
  webhook_configs:
  - url: 'https://api.opsgenie.com/v2/heartbeats/<the-heartbeat-name>/ping'
    send_resolved: false
    http_config:
      basic_auth:
        password: 123e4567-e89b-12d3-a456-426655440000
```

## Description
Proxy Prometheus Alertmanager webhooks to OpsGenie Heartbeat API to work around issue discussed here 
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

## Kubernetes Deployment

If you want to use opsgenie heartbeat proxy for a prometheus setup on kubernetes you can use the yaml file from examples to deploy the proxy to your running kubernetes cluster.
If you run prometheus inside your kubernetes cluster the service type "ClusterIP" should be sufficient. Otherwise, you can change the kubernetes service type to  "LoadBalancer" to request a public routable ip.
