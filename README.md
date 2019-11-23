# try-k8s

## https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/

```
$ minikube start --v=0
😄  minikube v1.5.2 on Darwin 10.14.6
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🔄  Starting existing virtualbox VM for "minikube" ...
⌛  Waiting for the host to be provisioned ...
🐳  Preparing Kubernetes v1.16.2 on Docker '18.09.9' ...
🔄  Relaunching Kubernetes using kubeadm ... 
⌛  Waiting for: apiserver
🏄  Done! kubectl is now configured to use "minikube"
```

```
$ kubectl get nodes 
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   21m   v1.16.2
```
## https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/

```
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
deployment.apps/kubernetes-bootcamp created
```

```
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   29m   v1.16.2
```
```
$ kubectl get deployments      
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           109s
```

```
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

```
$ curl http://127.0.0.1:8001
{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
     :
    "/version"
  ]
}
```

```
$ curl http://127.0.0.1:8001/version
{
  "major": "1",
  "minor": "16",
  "gitVersion": "v1.16.2",
  "gitCommit": "c97fe5036ef3df2967d086711e6c0c405941e14b",
  "gitTreeState": "clean",
  "buildDate": "2019-10-15T19:09:08Z",
  "goVersion": "go1.12.10",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

```
$ kubectl get pods 
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-69fbc6f4cf-755h5   1/1     Running   0          8m21s
```

### delete deployments

deployments を削除し pods を確認すると STATUS が Terminating。
その後、再度、deployments を作成すると pods の STATUS が Running。

```
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           8m30s
```

```
$ kubectl delete deployment kubernetes-bootcamp 
deployment.apps "kubernetes-bootcamp" deleted
```

```
$ kubectl get pods                          
NAME                                   READY   STATUS        RESTARTS   AGE
kubernetes-bootcamp-69fbc6f4cf-755h5   1/1     Terminating   0          10m
```

```
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
deployment.apps/kubernetes-bootcamp created
```

```
$ kubectl get pods                  
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-69fbc6f4cf-26hdk   1/1     Running   0          6s
```

## https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/

```
$ kubectl describe pods
Name:         kubernetes-bootcamp-69fbc6f4cf-m7c7q
Namespace:    default
Priority:     0
Node:         minikube/192.168.99.107
Start Time:   Sun, 24 Nov 2019 04:32:01 +0900
Labels:       app=kubernetes-bootcamp
              pod-template-hash=69fbc6f4cf
Annotations:  <none>
Status:       Running
IP:           172.17.0.6
IPs:
  IP:           172.17.0.6
Controlled By:  ReplicaSet/kubernetes-bootcamp-69fbc6f4cf
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://c06f84570d6005ece40ec1bcec09ff1d7f24b585c1324f5eea21b21a7b54857e
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 24 Nov 2019 04:32:03 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-n2n6m (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-n2n6m:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-n2n6m
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  <unknown>  default-scheduler  Successfully assigned default/kubernetes-bootcamp-69fbc6f4cf-m7c7q to minikube
  Normal  Pulled     4s         kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal  Created    4s         kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    3s         kubelet, minikube  Started container kubernetes-bootcamp
```

```
$ curl http://127.0.0.1:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
Error trying to reach service: 'dial tcp 172.17.0.6:80: connect: connection refused'
```

`connection refused` される
