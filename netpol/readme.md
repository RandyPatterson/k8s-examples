# Network Policy
Kubernetes Tutorial 

#### Objective 
The objective of this tutorial is to create a Network Policy for a a Pod and test connectivity using a *debug* pod and a *BusyBox* image.


#### Create a pod
* Assign the name nginx-pod
* Use the nginx image
* Add the label “app” with a value of “test”
* Expose pod using a ClusterIP service on port 80
* Prevent all connections to pod unless the pod has a label of role=debug
---


### Create a pod using the NGINX image 
create a pod named **nginx-pod**
```bash
kubectl run nginx-pod --image=nginx -l app=test
kubectl get pods --show-labels
```

### Expose pod using a Service
create a ClusterIP service mapped to the **nginx-pod** pod 
```bash
kubectl expose --name nginx-svc pod/nginx-pod --port=80 
kubectl get all --show-labels -o wide
```


### Debug pod 
attach to a debug container inside the cluster to access the **nginx-pod**

```bash 
kubectl run busybox -it --rm --image=busybox
```

Test connection to pod via service 
```bash
wget -S -T 5 -O -  http://nginx-svc
```

Sample output for a successful connection
```
Connecting to nginx-svc (10.0.238.205:80)
  HTTP/1.1 200 OK
  Server: nginx/1.21.1
  Date: Thu, 19 Aug 2021 18:44:32 GMT
  Content-Type: text/html
  Content-Length: 612
  Last-Modified: Tue, 06 Jul 2021 14:59:17 GMT
  Connection: close
  ETag: "60e46fc5-264"
  Accept-Ranges: bytes

index.html
```

Open another terminal and create Network Policy YAML.
 Review [Documentation](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
```bash
vi netpol.yaml
```

Past the following YAML
```YAML
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-netpol-ingress
spec:
  podSelector:
    matchLabels:
      app: test
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: debug
```
Verify contents 
```bash
cat netpol.yaml
```

Create the Network Policy 
```bash 
kubectl create -f netpol.yaml
```
Describe the network policy to make sure the rules were applied as expected 

```bash
kubectl  describe netpol test-netpol-ingress
```
sample output 

```
Name:         test-netpol-ingress
Namespace:    default
Created on:   2021-08-19 15:01:34 -0400 EDT
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=test
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      PodSelector: role=debug
  Not affecting egress traffic
  Policy Types: Ingress
  ```


Verify the debug pod can no longer communicate with pod **nginx-pod**
>The debug pod does not have the label **role=debug** so the pod is **not** allowed to communicate with the nginx-pod Pod

From the attached debug pod - issue the command 
```bash
wget -S -T 5 -O -  http://nginx-svc
```
sample output 

```
Connecting to nginx-svc (10.0.xxx.xxx:80)
wget: download timed out
```

Add the label *role=debug* to the pod so the Network Policy will allow the traffic to the pod **nginx-pod**

```bash
kubectl label pod/busybox  role=debug
```
Verify that the debug pod can now communicate with the NGINX pod. From the attached debug pod - issue the command 
```bash
wget -S -T 5 -O -  http://nginx-svc
```


## Cleanup

```bash 
kubectl delete  netpol/test-netpol-ingress pod/nginx-pod service/nginx-svc
```