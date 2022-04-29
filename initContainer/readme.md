# [Init Containers ](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

Demonstrates the use of shared volumes and init containers 

Creates a pod with an Initialization Container and a Web Site (NGINX). Both containers in the pod share a volume mapped to the NGINX home directory. The init container uses git to pull the web site from GitHub into the shared volume.  When the init container completes the NGINX container starts and hosts the content from the git repository.  

```bash 
kubectl apply -f ./initWithGit.yaml
```

Review logs from the init container 
```bash 
kubectl logs myapp-pod -c init-gitclone
```
Sample Output: 
```
# kubectl logs myapp-pod -c init-gitclone
Cloning into '/wwwroot'...

```

Wait for the service to retrieve a Public IP Address listed above
```bash
kubectl get svc myapp-svc -w
```

```
# kubectl get svc -w
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
myapp-svc    LoadBalancer   10.0.139.105   20.81.x.x   80:30254/TCP   12s
```
Open your browser and navigate to the **EXTERNAL-IP** address 

