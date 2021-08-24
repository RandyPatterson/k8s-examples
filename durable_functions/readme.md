

## Reference Material 

[func cli ](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cportal%2Cbash%2Ckeda)

[Create a function on Linux using a custom container](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-function-linux-custom-image?tabs=bash%2Cportal&pivots=programming-language-csharp)

[Deploy to Kubernetes](https://docs.microsoft.com/en-us/azure/azure-functions/functions-kubernetes-keda)

[Azurite Image](https://hub.docker.com/_/microsoft-azure-storage-azurite)

[Azurite Image Documentation](https://github.com/Azure/Azurite/blob/main/README.md)

[Keda ScaledObject Documentation](https://keda.sh/docs/1.4/concepts/scaling-deployments/)

[Keda Durable Functions ScaledObject](https://github.com/kedacore/keda-external-scaler-azure-durable-functions)

[Run a Durable Azure Function in a Container (old)](https://carlos.mendible.com/2018/01/14/run-a-durable-azure-function-in-a-container/)



>NOTE:</br>
>Default storage emulator information
>
>Account name: `devstoreaccount1` </br>
>Account key: `Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==`

---

## Docker Deployment 

### Azure Storage Simulator running in a container
```bash
docker run -d  -p 10000:10000 -p 10001:10001 -p 10002:10002 mcr.microsoft.com/azure-storage/azurite
```
### Default Connection string for Docker Deployment
```
DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;TableEndpoint=http://127.0.0.1:10002/devstoreaccount1;
```

Once deployed you can now connect to the storage emulator using [Azure Storage Explorer](https://docs.microsoft.com/en-us/azure/vs-azure-tools-storage-manage-with-storage-explorer?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&tabs=linux)

---

## Kubernetes Deployment

### Deploy storage emulator to k8s
Deploy the storage emulator Pod and a ClusterIP service to give the emulator a *fqnd* of **storage-emulator**
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: azurite
  name: azurite
spec:
  containers:
  - image: mcr.microsoft.com/azure-storage/azurite
    name: azurite
    ports:
    - containerPort: 10000
    - containerPort: 10001
    - containerPort: 10002

    resources: {}
  restartPolicy: Always
status: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: azurite
  name: storage-emulator
spec:
  ports:
  - port: 10000
    protocol: TCP
    targetPort: 10000
    name: blob
  - port: 10001
    protocol: TCP
    targetPort: 10001
    name: queue
  - port: 10002
    protocol: TCP
    targetPort: 10002
    name: table
  selector:
    app: azurite
```

### Connect localhost to Storage Emulator in Kubernetes
In order to run the [Azure Storage Explorer](https://docs.microsoft.com/en-us/azure/vs-azure-tools-storage-manage-with-storage-explorer?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&tabs=linux) you will need to forward the required storage emulator ports from you laptop to your Kubernetes cluster


```bash
 kubectl port-forward service/storage-emulator 10000 10001 10002
 ```

### Connection string when deployed to Kubernetes
To work with Keda, the full URL to the storage emulator is needed including the namespace.  When deployed to the namespace **default** the url is `storage-emulator.default.svc.cluster.local`.  If the Storage emulator is deployed to a different namespace then change the connection string

Use this connection string for your application running in pods that need access to a storage account 

```
* AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;DefaultEndpointsProtocol=http;BlobEndpoint=http://storage-emulator.default.svc.cluster.local:10000/devstoreaccount1;QueueEndpoint=http://storage-emulator.default.svc.cluster.local:10001/devstoreaccount1;TableEndpoint=http://storage-emulator.default.svc.cluster.local:10002/devstoreaccount1
```

## Troubleshooting
### Webhooks are not configured
If the pod receives the error `Webhooks are not configured` then add `"WEBSITE_HOSTNAME": "localhost:8080"` to the **local.settings.json** file 

> Change the value of WEBSITE_HOSTNAME to the ip address or URL of the service exposing the Orchestrators trigger function


```json
{
    "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=rrpfunc;AccountKey=hXlZ2k9pNBkv4e3YveT9YQGwLPt4jXv7ArpBPcJjf+xwu0I1ufJ0f+6QSodhYEqmolxMRFot8EMhiuJA+MIp6Q==;BlobEndpoint=https://rrpfunc.blob.core.windows.net/;TableEndpoint=https://rrpfunc.table.core.windows.net/;QueueEndpoint=https://rrpfunc.queue.core.windows.net/;FileEndpoint=https://rrpfunc.file.core.windows.net/",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "WEBSITE_HOSTNAME": "localhost:8080"
  }
}
```