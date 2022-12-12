# Demo Resource Limits 

## Task 1: No Limits

1. View [workload-dep.yam](./workload-dep.yaml) and point out that there are 50 replicas and NO resource limits defined 

1. Deploy workload (50 pods)
    ```bash
    kubectl apply -f workload-dep.yaml
    ```
1. View pods - wait for all pods to show a status of **running** 
    ```bash
    kubectl get pods 
    ```
    > NOTE: with no defined **Request limits** kubernetes cannot calculate how many pods can comfortably run on a node
1. Delete deployment
    ```bash
    kubectl delete deploy workload-dep
    ```

## Task 2: Limits configured 
1. View [workload-dep-resourceLimits.yaml](./workload-dep-limits.yaml) and show the Resource Limits defined 
    ```yaml
    resources: 
      requests:
        cpu: 500m
        memory: 512M
      limits:
        cpu: 500m
        memory: 512M
    ```
1. Deploy workload (50 pods)
    ```bash
    kubectl apply -f workload-dep-limits.yaml
    ```
1. View pods and notice that most pods are pending 
    ```bash
    kubectl get pods 
    ```
    filter RUNNING/PENDING pods 
    ```bash
    kubectl get pods --field-selector="status.phase==Running"
    kubectl get pods --field-selector="status.phase==Pending"
    ```
    >NOTE: In my case the node has approx 3 cores available and since we requested 1/2 core per container kubernetes could only schedule 6 pods to run on the single node. - YMMV
1. Clean up
    ```bash
    kubectl delete deploy workload-dep
    ```
## Task 3: Default Limits 
1. Deploy nginx pod with no defined limits 
    ```bash
    kubectl create -f pod-nolimits.yaml
    ```
    Show that the pod does NOT have defined request and limits and QOS is set to **BestEffort** so it has a low priority and will be one of the first pods ejected if node resources become constrained 
    ```bash
    kubectl describe pod nginx 
    ```
    Example 
    ```yaml
    QoS Class:  BestEffort
    ```
    Delete pod 
    ```bash
    kubectl delete pod nginx
    ```
1. Deploy Default request & limits 
    View and discuss [namespace-limit-range.yaml](./namespace-limit-range.yaml) 
    ```bash
    kubectl apply -f namespace-limit-range.yaml
    kubectl get limitrange
    ```
1. Verify default limits applied to resources deployed with no defined resource limits 
    ```bash
    kubectl create -f pod-nolimits.yaml
    kubectl describe pod nginx
    ```
    Output
    ```yaml
    Limits:
      cpu:     200m
      memory:  256Mi
    Requests:
      cpu:        100m
      memory:     128Mi
    ...
    QoS Class:  Burstable
    ```
1. Cleanup 
    ```bash
    kubectl delete -f pod-nolimits.yaml -f namespace-limit-range.yaml
    ```

