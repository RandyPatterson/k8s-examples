# Demo Horizontal Pod Autoscaler

## Setup 
```bash
watch -n 1 kubectl get all
```

```bash
watch -n 1 kubectl top pods
```

### Deploy Application

```bash
kubectl apply -f complex-web-dep.yaml -f complex-web-svc.yaml
```

### Deploy HPA

```bash
kubectl apply -f complex-web-hpa.yaml
```


```bash
#create a namespace to "hide" load testing pods 
kubectl create ns load-test
kubectl apply -f complex-web-load.yaml -n load-test
```

Remove load and watch app scale down
```bash
kubectl delete -f complex-web-load.yaml -n load-test
```

## Cleanup 
```bash
kubectl delete ns load-test
kubectl delete -f complex-web-hpa.yaml
kubectl delete -f complex-web-dep.yaml -f complex-web-svc.yaml
```
