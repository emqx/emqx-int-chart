## Install
```bash
helm install ${release-name} . --namespace ${release-namespace} --create-namespace \
--set prometheus-pushgateway.serviceMonitor.namespace=${release-namespace} \
--set emqx-ee.emqxConfig.EMQX_PROMETHEUS__PUSH__GATEWAY__SERVER=http://${release-name}-prometheus-pushgateway.${release-namespace}.svc.cluster.local:9091
```

## Config grafana
```bash
k edit cm ${release-name}-grafana -n ${release-namespace}

[server]
domain = ${nginx_ingress}
root_url = %(protocol)s://%(domain)s:%(http_port)s/grafana/
serve_from_sub_path = true
```
> Don't forget to restart grafana pod

## Testing
```bash
helm test emqx-init --logs -n emqx-init
```

## Uninstall
```bash
helm uninstall ${release-name} --namespace ${release-namespace}

kubectl delete crd alertmanagerconfigs.monitoring.coreos.com
kubectl delete crd alertmanagers.monitoring.coreos.com
kubectl delete crd podmonitors.monitoring.coreos.com
kubectl delete crd probes.monitoring.coreos.com
kubectl delete crd prometheuses.monitoring.coreos.com
kubectl delete crd prometheusrules.monitoring.coreos.com
kubectl delete crd servicemonitors.monitoring.coreos.com
kubectl delete crd thanosrulers.monitoring.coreos.com
```

## EMQX rule engine Configuration
### Postgres
> db: mqtt
> user: chaos
> passwd: public

### Kafka
> user:
> passwd:
> topic: testTopic
