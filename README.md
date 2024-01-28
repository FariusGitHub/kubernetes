# Kubernetes 101 <br>

![](/images/05-image01.jpg)
<br><br>

## POD BASIC, IMPERATIVE VS DECLARATIVE 
The smallest compute in kubernetes is called a pod. There are couple ways to create one:
- imperative
- declarative

The example of imperative way would be like as follow

```txt
kubectl run imperative --image=nginx --restart=Never
```

And the example of declarative one would be like
```txt
kubectl apply -f declarative.yaml
```
We assume the equivalent declarative.yaml to above imperative one is as simple as

```txt
apiVersion: v1
kind: Pod
metadata:
  name: declarative
  labels:
    app: web-server
spec:
  containers:
  - name: declarative-container
    image: nginx
    ports:
    - containerPort: 80
```

In fact it was not. Take a look into reverse engineering of imperative yaml file with
```txt
kubectl get pod imperative -o yaml > imperative.yaml
```
In this case we will copy that configuration as a base of declarative.yaml
```txt
mv imperative.yaml declarative.yaml
```

changing imperative words into declarative, we should have declarative.yaml like.

```txt
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-01-26T23:15:12Z"
  labels:
    run: declarative
  name: declarative
  namespace: default
  resourceVersion: "108222"
  uid: c295e838-a170-42f4-96fb-279ee8a053ea
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: declarative-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-n48sh
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: docker-desktop
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Never
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-n48sh
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2024-01-26T23:15:12Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2024-01-26T23:15:17Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2024-01-26T23:15:17Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2024-01-26T23:15:12Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://465a1327aaca23f2e84394e86a47a432024283989dd982466a8f09965e597f11
    image: nginx:latest
    imageID: docker-pullable://nginx@sha256:4c0fdaa8b6341bfdeca5f18f7837462c80cff90527ee35ef185571e1c327beac
    lastState: {}
    name: declarative
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2024-01-26T23:15:16Z"
  hostIP: 192.168.65.9
  phase: Running
  podIP: 10.1.1.56
  podIPs:
  - ip: 10.1.1.56
  qosClass: BestEffort
  startTime: "2024-01-26T23:15:12Z"
```

Says that we copy paste imperative.yaml into imperative2.yaml. The reason I am doing that is to test the ability of declaratively generated pod to update imperatively created pod. Both file has the same content. The update may have some warnings but should show a success to configure imperative pod like below.
```txt
kubectl apply -f imperative2.yaml
```
```txt
Warning: resource pods/imperative is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
```
```txt
pod/imperative configured
```

On the flip side, if initially imperatively generated pod was created and imagine the imperative2.yaml is simply like this
```txt
apiVersion: v1
kind: Pod
metadata:
  name: imperative
  labels:
    app: web-server
spec:
  containers:
  - name: imperative-container
    image: nginx
    ports:
    - containerPort: 80
```
and when we try to update the imperatively generated pod like this
```txt
kubectl apply -f imperative2.yaml
```
It will then showing an unseuccessful update like below
```txt
Warning: resource pods/imperative is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
The Pod "imperative" is invalid: spec.containers: Forbidden: pod updates may not add or remove containers
```
![](/images/05-image02.png)
Figure 1: Summary of Imperative and Declarative Style in Pod Creation and Update  
<br><br>

## KUBECONFIG
It was an industry standard to say so but physically the file was located at<br>
```txt
~/.kube/config
```
at for example, it would look like below
```txt
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0 ...
    server: https://kubernetes.docker.internal:6443
  name: docker-desktop
contexts:
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-desktop
current-context: docker-desktop
kind: Config
preferences: {}
users:
- name: docker-desktop
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURRakNDQWlxZ0F3SU ...
    client-key-data: LS0tLS1CRUdJTiBS ...
```
Take a look at context, which is a combination of user and cluster.
In term of GitOps, there could be 2 clusters and 2 users in a kubeconfig file.

Herewith are the syntax to call them out
```txt
kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop   docker-desktop   docker-desktop   
```
```txt
kubectl config get-users
NAME
docker-desktop
```
```txt
kubectl config get-clusters
NAME
docker-desktop
```

Let's say if there is some error in this config file as we purposely doing so below. <br>
The example below is just an example what could go wrong in kubeconfig.

![](/images/05-image00.gif)
Figure 2: Error 403 and 200 OK when an error was introduced and later fixed  
<br>

## WATCH and LOGS
Let's see another feature from kubectl called --watch. If we use K9S this feature already there. Let's see how the pod was created and deleted while we can see the progress like below.

![](/images/05-image03.gif)
Figure 3: Comparable Screen from K9S and Traditional terminal with kubectl get pods --watch -v 6  
<br>

Similarly we can also watch the deployment. We can use hello-world.yaml for example as one of a deployment like below

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello-world
  name: hello-world
  namespace: default
spec:
  replicas: 5
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-world
        ports:
        - containerPort: 8080
```

![](/images/05-image04.gif)
Figure 4: Comparable Screen from K9S and Traditional terminal with kubectl get deployment --watch -v 6  
<br>

For Logs, the `--watch` flag is not available for logs in Kubernetes. The `--watch` flag is used to continuously monitor changes in resources like pods, services, or deployments. However, for logs, you can use the `kubectl logs` command to retrieve the logs of a specific pod. By default, it will show the most recent logs, but you can use the `-f` flag to follow the logs in real-time. 

![](/images/05-image05.gif)
Figure 5: Logs command in kubectl does not have --watch, it has to logs very specific pod by its name  
<br><br>

## NAMESPACE
We will go to Kubernetes Namespace, which can be seen from
```txt
kubectl get ns
```
That usually shows below default namespace

```txt
NAME              STATUS   AGE
default           Active   3d4h
kube-node-lease   Active   3d4h
kube-public       Active   3d4h
kube-system       Active   3d4h

```
These namespaces are having pods too, see below
```txt
kubectl get pods -n kube-system
```
```txt
NAME                                     READY   STATUS    RESTARTS        AGE
coredns-5dd5756b68-2fz27                 1/1     Running   0               3d5h
coredns-5dd5756b68-jdptg                 1/1     Running   0               3d5h
etcd-docker-desktop                      1/1     Running   0               3d5h
kube-apiserver-docker-desktop            1/1     Running   0               3d5h
kube-controller-manager-docker-desktop   1/1     Running   1 (7h48m ago)   3d5h
kube-proxy-thcxj                         1/1     Running   0               3d5h
kube-scheduler-docker-desktop            1/1     Running   0               3d5h
storage-provisioner                      1/1     Running   2 (123m ago)    3d4h
vpnkit-controller                        1/1     Running   0               3d4h
```

## SINGLE CONTAINER POD AND MULTIPLE CONTAINER POD

Let's see a bare container pod from pod.yaml like below
Having multiple yaml like this to generate multiple similar pod will not be practical.

```txt
apiVersion: v1
kind: Pod
metadata:
  name: hello-world-pod
spec:
  containers:
  - name: hello-world
    image: psk8s.azurecr.io/hello-app:1.0
    ports:
    - containerPort: 80
```

Let's see another alternative called controller pod below <br>
From this deployment.yaml we can see repllicas command wil populate the number of pods needed.

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world
        image: psk8s.azurecr.io/hello-app:1.0
        ports:
        - containerPort: 8080
```
here are some possible command to spin the deployment, scale up/down, delete

```txt
kubectl apply -f deployment.yaml
kubectl scale deployment hello-world --replicas=2
kubectl scale deployment hello-world --replicas=1
kubectl delete -f deployment.yaml
```

![](/images/05-image06.png)
Figure 6: As hello-world-789db8d56-mnknb was deleted/failed, hello-world-789db8d56-qthrh automatically generated for example. This is the real reason Kubernetes was born ! 
<br>

From above illustration, the way to reduce the number of pods from this deployment is by using scale command, not delete pod command. To remove all hello-world pods we can delete the entire deployment. 

## NETWORKING CHECK, RUNNING INSIDE A CONTAINER
Let's try a simple networking from each pod for the sake of pinging. <br>

To access to /bin/sh, we will replace gcr.io image into psk8s
```txt
...   containers:
      - image: psk8s.azurecr.io/hello-app:1.0
#      - image: gcr.io/google-samples/hello-app:1.0
...
```


We can use kubectl command like below to see the pod names and go into interactive mode accordingly.
```test
kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE     IP           NODE             NOMINATED NODE   READINESS GATES
hello-world-5f64785679-4cwvv   1/1     Running   0          8m44s   10.1.1.114   docker-desktop   <none>           <none>
hello-world-5f64785679-zbdbc   1/1     Running   0          8m44s   10.1.1.115   docker-desktop   <none>           <none>
hello-world-pod                1/1     Running   0          25m     10.1.1.106   docker-desktop   <none>           <none>

kubectl exec -it hello-world-5f64785679-4cwvv /bin/sh
/app # ping 10.1.1.114
PING 10.1.1.114 (10.1.1.114): 56 data bytes
64 bytes from 10.1.1.114: seq=0 ttl=64 time=0.034 ms
64 bytes from 10.1.1.114: seq=1 ttl=64 time=0.089 ms
64 bytes from 10.1.1.114: seq=2 ttl=64 time=0.075 ms

--- 10.1.1.114 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.034/0.066/0.089 ms
/app # ping 10.1.1.115
PING 10.1.1.115 (10.1.1.115): 56 data bytes
64 bytes from 10.1.1.115: seq=0 ttl=64 time=0.667 ms
64 bytes from 10.1.1.115: seq=1 ttl=64 time=0.120 ms
64 bytes from 10.1.1.115: seq=2 ttl=64 time=0.083 ms

--- 10.1.1.115 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.083/0.290/0.667 ms
/app # ping 10.1.1.106
PING 10.1.1.106 (10.1.1.106): 56 data bytes
64 bytes from 10.1.1.106: seq=0 ttl=64 time=0.134 ms
64 bytes from 10.1.1.106: seq=1 ttl=64 time=0.104 ms
64 bytes from 10.1.1.106: seq=2 ttl=64 time=0.131 ms
64 bytes from 10.1.1.106: seq=3 ttl=64 time=0.171 ms
```

Or we can use docker command as I use docker-desktop context.
```txt

docker ps -a
CONTAINER ID   IMAGE          COMMAND         CREATED          STATUS          PORTS     NAMES
b31f0453da5b   7f20d355455e   "./hello-app"   21 minutes ago   Up 21 minutes             k8s_hello-world_hello-world-5f64785679-zbdbc_default_46db1094-ca71-4391-aef5-acb78adcdc56_0
b00b1776e526   7f20d355455e   "./hello-app"   21 minutes ago   Up 21 minutes             k8s_hello-world_hello-world-5f64785679-4cwvv_default_4480435a-1611-4ef8-bc63-60ff0d6bc53a_0
c50c596a5385   7f20d355455e   "./hello-app"   37 minutes ago   Up 37 minutes             k8s_hello-world_hello-world-pod_default_4c2edb00-0f9a-4fcf-9793-79474de04ece_0

docker exec -it b31f0453da5b /bin/sh
/app # ping 10.1.1.60
PING 10.1.1.60 (10.1.1.60): 56 data bytes
64 bytes from 10.1.1.60: seq=0 ttl=64 time=0.351 ms
64 bytes from 10.1.1.60: seq=1 ttl=64 time=0.157 ms
--- 10.1.1.60 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.157/0.254/0.351 ms
```

## MULTICONTAINER WITH PRODUCER AND CONSUMER, PORT FORWARDING

```txt
apiVersion: v1
kind: Pod
metadata:
  name: multicontainer-pod
spec:
  containers:
  - name: producer
    image: ubuntu
    command: ["/bin/bash"]
    args: ["-c", "while true; do echo $(hostname) $(date) >> /var/log/index.html; sleep 10; done"]
    volumeMounts:
    - name: webcontent
      mountPath: /var/log
  - name: consumer
    image: nginx
    ports:
      - containerPort: 80
    volumeMounts:
    - name: webcontent
      mountPath: /usr/share/nginx/html
  volumes:
  - name: webcontent 
    emptyDir: {}
```
Looks at the detail of this pods

```txt
kubectl create -f multicontainer.yaml
pod/multicontainer-pod created
devops@msi:~$ kubectl describe pod multicontainer
Name:             multicontainer-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             docker-desktop/192.168.65.9
Start Time:       Sat, 27 Jan 2024 23:03:39 -0500
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.1.1.117
IPs:
  IP:  10.1.1.117
Containers:
  producer:
    Container ID:  docker://d5c139de2860e00819c6fdb38d22b913c7ce5047e415c27e203acb828c9c8388
    Image:         ubuntu
    Image ID:      docker-pullable://ubuntu@sha256:e6173d4dc55e76b87c4af8db8821b1feae4146dd47341e4d431118c7dd060a74
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/bash
    Args:
      -c
      while true; do echo $(hostname) $(date) >> /var/log/index.html; sleep 10; done
    State:          Running
      Started:      Sat, 27 Jan 2024 23:03:52 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/log from webcontent (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-r8xjr (ro)
  consumer:
    Container ID:   docker://feb508f77eaf8adb25740b25e5a311bd8696a1a2a1734c7a2bee704a04de0ee1
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:4c0fdaa8b6341bfdeca5f18f7837462c80cff90527ee35ef185571e1c327beac
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 27 Jan 2024 23:03:56 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from webcontent (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-r8xjr (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  webcontent:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-r8xjr:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  5m51s  default-scheduler  Successfully assigned default/multicontainer-pod to docker-desktop
  Normal  Pulling    5m48s  kubelet            Pulling image "ubuntu"
  Normal  Pulled     5m42s  kubelet            Successfully pulled image "ubuntu" in 6.208s (6.208s including waiting)
  Normal  Created    5m38s  kubelet            Created container producer
  Normal  Started    5m38s  kubelet            Started container producer
  Normal  Pulling    5m38s  kubelet            Pulling image "nginx"
  Normal  Pulled     5m36s  kubelet            Successfully pulled image "nginx" in 2.213s (2.213s including waiting)
  Normal  Created    5m35s  kubelet            Created container consumer
  Normal  Started    5m34s  kubelet            Started container consumer

```
let's go inside it

```txt
kubectl exec -it multicontainer-pod --container producer -- /bin/sh
# cd /var/log
# wc -l index.html
109 index.html
# wc -l index.html
110 index.html
# 
```
there is a new line added into index.html every 10 second.
On the consumer side, we can see something similar

```txt
kubectl exec -it multicontainer-pod --container producer -- /bin/sh
# cd /usr/share/nginx/html
# wc -l index.html
134 index.html
# wc -l index.html
135 index.html
# 
```
This demo shows how we can use two containers inside the same pod and also connect them to a single volume/directory where we can share information between them. Where is this thing happen? This could happen during logging and data sharing. <br>

Last part of this exercise, we also could add port-forwarding as follow
```txt
Forwarding from 127.0.0.1:8080 -> 80
```
From the other terminal we could monitor every 10 seconds increment like
```txt
curl http://localhost:8080
```
