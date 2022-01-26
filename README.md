## Requirement
### Install nginx ingress controller (classic)
```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace

or

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.0/deploy/static/provider/cloud/deploy.yaml
```

## Deployment
```bash
helm install ${release-name} . --namespace ${release-namespace} --create-namespace \
--set prometheus-pushgateway.serviceMonitor.namespace=${release-namespace} \
--set grafana."grafana\.ini".server.domain=${nginx-ingress}
--set emqx-ee.emqxConfig.EMQX_PROMETHEUS__PUSH__GATEWAY__SERVER=http://${release-name}-prometheus-pushgateway.${release-namespace}.svc.cluster.local:9091
```

## Testing
```bash
helm test ${release-name} --logs -n ${release-namespace}
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

## Config prometheus and grafana
### Install ingress for prometheus and grafana
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: ${release-namespace}
  name: prom-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - http:
      paths:
      - path: /prom(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: ${promethues-service-name}
            port:
              number: 9090
      - path: /grafana(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: ${grafana-service-name}
            port:
              number: 80
```

### Url
> http://${nginx-ingress}/prom/graph  
http://${nginx-ingress}/grafana/login

### Add Data Sources
> ${prometheus-service-name}.${release-namespace}.svc.cluster.local

## EMQX rule engine Configuration
### Postgres
> db: mqtt  
user: chaos  
passwd: public

### Kafka
> user:  
passwd:  
topic: testTopic
