---
tags: cheatsheet, consul, service discovery
---

# Consul HTTP API cheat sheet

Below is a compilation of Consul HTTP API calls that I frequently rely on for
troubleshooting purposes. If you've found this document, I kindly advise against
utilizing it as a primary reference.

## Retrieve Service Information

```bash
curl -X GET \
http://consul.service.dc2b-prod.consul:28500/v1/catalog/service/<service_name> | jq
```

## Register a service with the Curl command:

```bash
curl -X PUT -H "Content-Type: application/json" \
--data @data.json \
http://consul.service.<data_center>.consul:8500/v1/agent/service/register
```

where `data.json` looks like: 

```json
{
    "ID": "prometheus-main-id",
    "Name": "prometheus-main",
    "Tags": ["aws", "dc2", "prod", "prometheus", "prometheus-b"],
    "Address": "10.3.246.132",
    "Port": 9090,
    "Check": {
        "HTTP": "http://10.3.246.132:9090",
        "Interval": "30s"
    }
}
```

## Remove a service from the Consul catalog

```bash
curl -X PUT
http://<consul_node_ip>:8500/v1/agent/service/deregister/prometheus-main-id
```

You can find the `consul_node_ip` with: 

```bash
curl -X GET http://consul.service.consul_dc>.consul:8500/v1/catalog/service/<service_name> | jq
```

## Remove a node from Consul

```bash
curl -X PUT http://consul.service.<consul_dc>.consul:8500/v1/catalog/deregister \
--header 'Content-Type: application/json' \
--data-raw '{
  "Datacenter": "<consul_dc>",
  "Node": "foo.example.com"
}'
```
