# Demo Script

Look at the default **Tolerations** for a deployed pod
```
kubectl describe pod <pod-name>
```
Show the **not-ready** and **unreachable** tolerations with a default of 300 seconds (5 minutes)

```
Tolerations:     node.kubernetes.io/memory-pressure:NoSchedule op=Exists
                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
```
 


### Step 1: Deploy Pod 
deploy the pod changing the  **not-ready** and **unreachable** tolerations to 10 seconds.

```kubectl apply -f deployment.yaml```

### Step 2: View the nodes the pods are deployed to 
```watch -n .5 kubectl get pods -o wide```

### Step 3: Watch the nodes
```watch -n .5 get nodes```

### Step 4: in the Azure Portal,  shutdown one of the node VMs
The shutdown node will change from a status of '***Ready***' to '***NotReady***' and 10 seconds later the pods on the node will terminate and new pods created on the remaining Nodes.

### Step 5: Clean up
```kubectl delete -f deployment.yaml```
