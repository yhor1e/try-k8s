# try-k8s

## https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/

<details>
  <summary>log</summary>
  
```bash
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

```bash
$ kubectl get nodes 
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   21m   v1.16.2
```
</details>



## https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/

<details>
  <summary>log</summary>

```bash
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
deployment.apps/kubernetes-bootcamp created
```

```bash
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   29m   v1.16.2
```

```bash
$ kubectl get deployments      
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           109s
```

```bash
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

```bash
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

```bash
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

```bash
$ kubectl get pods 
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-69fbc6f4cf-755h5   1/1     Running   0          8m21s
```
</details>

### delete deployments

<details>
  <summary>log</summary>

deployments を削除し pods を確認すると STATUS が Terminating。
その後、再度、deployments を作成すると pods の STATUS が Running。

```bash
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           8m30s
```

```bash
$ kubectl delete deployment kubernetes-bootcamp 
deployment.apps "kubernetes-bootcamp" deleted
```

```bash
$ kubectl get pods                          
NAME                                   READY   STATUS        RESTARTS   AGE
kubernetes-bootcamp-69fbc6f4cf-755h5   1/1     Terminating   0          10m
```

```bash
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
deployment.apps/kubernetes-bootcamp created
```

```bash
$ kubectl get pods                  
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-69fbc6f4cf-26hdk   1/1     Running   0          6s
```
</details>

## https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/

<details>
  <summary>log</summary>

```bash
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

```bash
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-69fbc6f4cf-m7c7q

$ curl http://127.0.0.1:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
Error trying to reach service: 'dial tcp 172.17.0.6:80: connect: connection refused'
```

`connection refused` される

</details>

### デバッグ

katacode 環境だと正常にレスポンスがあるが、ローカルでは refused されるのでデバッグする。

```bash
$ curl http://127.0.0.1:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
Error trying to reach service: 'dial tcp 172.17.0.6:80: connect: connection refused'
```

Pod の Bash からは curl できている。

```bash
$ kubectl exec -ti $POD_NAME bash
root@kubernetes-bootcamp-69fbc6f4cf-m7c7q:/# cat server.js
var http = require('http');
var requests=0;
var podname= process.env.HOSTNAME;
var startTime;
var host;
var handleRequest = function(request, response) {
  response.setHeader('Content-Type', 'text/plain');
  response.writeHead(200);
  response.write("Hello Kubernetes bootcamp! | Running on: ");
  response.write(host);
  response.end(" | v=1\n");
  console.log("Running On:" ,host, "| Total Requests:", ++requests,"| App Uptime:", (new Date() - startTime)/1000 , "seconds", "| Log Time:",new Date());
}
var www = http.createServer(handleRequest);
www.listen(8080,function () {
    startTime = new Date();;
    host = process.env.HOSTNAME;
    console.log ("Kubernetes Bootcamp App Started At:",startTime, "| Running On: " ,host, "\n" );
});

root@kubernetes-bootcamp-69fbc6f4cf-m7c7q:/# curl localhost:8080
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-69fbc6f4cf-m7c7q | v=1
```

Pod の listening port を確認する。

```bash
$ minikube ssh
```

```bash
$ sudo ip netns add foo
$ ip netns ls
foo
$ ls /var/run/netns/
foo
$ sudo ln -s /proc/4177/ns/net /var/run/netns/4177
$ ip netns ls
4177
foo
$ sudo ip net exec 4177 netstat -lnt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       
tcp        0      0 :::8080                 :::*                    LISTEN    
```

80 ポートが Listen されてない
