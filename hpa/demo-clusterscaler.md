# Demo AKS Cluster Autoscaler 

1. Enable Node Scaler 
    1. navigate to [Azure Portal](https://portal.azure.com)
    1. AKS Cluster -> Node Pools -> UserPool -> SCale Node Pool 
    1. Autoscale -> 1-5 nodes
    1. Click Apply and wait for it to complete 


>TIP:  Watch nodes and pods in a sperate window
```bash
watch -n 1 kubectl get nodes,pods -o wide
```
1. Deploy application and scale back to 10 pods
    ```bash
    kubectl apply -f workload-dep-limits.yaml
    kubectl scale deploy workload-dep --replicas=10
    ```
1. Show several pods are in a Pending status and a **describe** indicates that pending pods cannot be scheduled to a node due to resource constraints 
    ```bash
    kubectl get nodes,pods -o wide
    ```
1. Watch nodes and in about 3 minutes a new node will be added and a few seconds later the **Pending** pods will be deployed to the new node.

## Cleanup
```bash
    kubectl delete deploy workload-dep
```