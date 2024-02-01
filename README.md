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
<br><br>

# CONTEXT, USERNAME, CLUSTER
Let's try to create a new context based on the same cluster docker-desktop and username also docker-desktop. <br>

Currently we have only one context and one cluster as follow

```txt
kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop   docker-desktop   docker-desktop   
```
Or in K9s it would looks like below
```txt
 Context: docker-desktop ✏️                         <0> all       <a>      Attach     <l>       Logs            <f> Show PortForward                           ____  __.________        
 Cluster: docker-desktop                           <1> default   <ctrl-d> Delete     <p>       Logs Previous   <t> Transfer                                  |    |/ _/   __   \______ 
 User:    docker-desktop                                         <d>      Describe   <shift-f> Port-Forward    <y> YAML                                      |      < \____    /  ___/ 
 K9s Rev: v0.31.7                                                <e>      Edit       <z>       Sanitize                                                      |    |  \   /    /\___ \  
 K8s Rev: v1.28.2                                                <?>      Help       <s>       Shell                                                         |____|__ \ /____//____  > 
 CPU:     n/a                                                    <ctrl-k> Kill       <n>       Show Node                                                             \/            \/  
 MEM:     n/a                                                                                                                                                                          
┌───────────────────────────────────────────────────────────────────────────────── Pods(default)[0] ──────────────────────────────────────────────────────────────────────────────────┐
│ NAME↑                  PF                  READY                  STATUS                                   RESTARTS IP                  NODE                  AGE                   │
│                                                                                                                                                                                     │
│                                                                                                                                                                                     │
│                                                                                                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
  <pod>                                                                       
```
Let's see existing username like below

```txt
kubectl config view --minify -o jsonpath='{.users[].name}'
docker-desktop
```
Herewith how we make a new context named new-context <br>
with the same username docker-desktop, the same cluster docker-desktop<br>
and switch to it as a default context (shown with *)

```txt
kubectl config set-context new-context --cluster=docker-desktop --user=docker-desktop --namespace=default
Context "new-context" created.

kubectl config use-context new-context
Switched to context "new-context".

kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
          docker-desktop   docker-desktop   docker-desktop   
*         new-context      docker-desktop   docker-desktop   default
```


When we create a pod like below on the other terminal and press ":" on K9S screen "context new-context"<br>
The combined screen should look like below. <br>
As we shared the same username and cluster the same pod will be shown as well in docker-desktop context.

```txt
kubectl create -f pod.yaml
pod/hello-world-pod created

 Context: new-context ✏️                            <0> all       <a>      Attach     <l>       Logs            <f> Show PortForward                           ____  __.________        
 Cluster: docker-desktop                           <1> default   <ctrl-d> Delete     <p>       Logs Previous   <t> Transfer                                  |    |/ _/   __   \______ 
 User:    docker-desktop                                         <d>      Describe   <shift-f> Port-Forward    <y> YAML                                      |      < \____    /  ___/ 
 K9s Rev: v0.31.7                                                <e>      Edit       <z>       Sanitize                                                      |    |  \   /    /\___ \  
 K8s Rev: v1.28.2                                                <?>      Help       <s>       Shell                                                         |____|__ \ /____//____  > 
 CPU:     n/a                                                    <ctrl-k> Kill       <n>       Show Node                                                             \/            \/  
 MEM:     n/a                                                                                                                                                                          
┌───────────────────────────────────────────────────────────────────────────────── Pods(default)[0] ──────────────────────────────────────────────────────────────────────────────────┐
│ NAME↑                  PF                  READY                  STATUS                                   RESTARTS IP                  NODE                  AGE                   │
│ hello-world-pod        ●                   1/1                    Running                                  0        10.1.1.125          docker-desktop                              │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
  <pod>       

```

## STATIC PODS
I will not be able to show an exact demo here as I have a super simple kubectl version that does not have /var/lib/kubelet/config.yaml or /etc/kubernetes/manifests folder. But the idea is showing you an untouchable pod if the yaml file was located in staticPod folder. <br>

If you have the right kubectl version <br>
/var/lib/kubelet/config.yaml is normally stated <br>
staticPodPath: /etc/kubernetes/manifests

Says you have a yaml file called test.yaml. <br>
When you do
```txt
kubectl create -f test.yaml
```
and move that yaml file into 
```txt
sudo cp test.yaml /etc/kubernetes/manifests
```
Assuming the pod from above test.yaml was named hello-world, see what happen if you do below
```txt
kubectl delete pod hello-world
```
It is very likely this pod will be re-created once a deletion attempt was initiated.
<br> <br>

## INIT CONTAINER
In this section we will go through some initiation process neeed such as to install a database. If this pod is not available, subsequent pod will not work, for example before an application can be launched we need to extract some source data unless the app will be meaningless.

Let's see below situation from init-containers.yaml

```txt
apiVersion: v1
kind: Pod
metadata:
  name: init-containers
spec:
  initContainers:
  - name: init-service
    image: ubuntu
    command: ['sh', '-c', "echo waiting for service; sleep 2"]
  - name: init-database
    image: ubuntu
    command: ['sh', '-c', "echo waiting for database; sleep 2"]
  containers:
  - name: app-container
    image: nginx
```

If you do kubectl describe pods init-containers and go the Events description down below<br>
you will normally see below event happened in sequence. Pod was created and status is running.  
  Type    |Reason     |Age    |From               |Message
  ----    |------     |----   |----               |-------
  Normal  |Scheduled  |2m31s  |default-scheduler  |Successfully assigned default/init-containers to docker-desktop
  Normal  |Pulling    |2m29s  |kubelet            |Pulling image "ubuntu"
  Normal  |Pulled     |2m28s  |kubelet            |Successfully pulled image "ubuntu" in 1.214s (1.214s including waiting)
  Normal  |Created    |2m27s  |kubelet            |Created container init-service
  Normal  |Started    |2m26s  |kubelet            |Started container init-service
  Normal  |Pulling    |2m24s  |kubelet            |Pulling image "ubuntu"
  Normal  |Pulled     |2m23s  |kubelet            |Successfully pulled image "ubuntu" in 1.064s (1.064s including waiting)
  Normal  |Created    |2m22s  |kubelet            |Created container init-database
  Normal  |Started    |2m22s  |kubelet            |Started container init-database
  Normal  |Pulling    |2m18s  |kubelet            |Pulling image "nginx"
  Normal  |Pulled     |2m17s  |kubelet            |Successfully pulled image "nginx" in 1.354s (1.354s including waiting)
  Normal  |Created    |2m16s  |kubelet            |Created container app-container
  Normal  |Started    |2m16s  |kubelet            |Started container app-container

### TIPS
Normally this command below will create a pod for you regardless there is an error or not
```txt
kubectl create -f init-containers.yaml
```
But this command will show you an error if there is an error regardless at pod creation
```txt
kubectl apply -f init-containers.yaml
```
To encounter subsequent error, we need to fix yaml file, delete previous pod and apply it again
```txt
nano init-containers                     # fix the syntax
kubectl delete pods init-containers      # delete previous pod to avoid confusion
kubectl apply -f init-containers.yaml    # recreate pod with apply or create command
```
<br>

Let's see the flip side when there is an error in init-containers.yaml
  Type     |Reason     |Age                |From               |Message
  ----     |------     |----               |----               |-------
  Normal   |Scheduled  |46s                |default-scheduler  |Successfully assigned default/init-containers to docker-desktop
  Normal   |Pulled     |43s                |kubelet            |Successfully pulled image "ubuntu" in 1.171s (1.171s including waiting)
  Normal   |Pulled     |40s                |kubelet            |Successfully pulled image "ubuntu" in 1.062s (1.062s including waiting)
  Normal   |Pulling    |28s (x3 over 44s)  |kubelet            |Pulling image "ubuntu"
  Normal   |Pulled     |27s                |kubelet            |Successfully pulled image "ubuntu" in 1.066s (1.066s including waiting)
  Normal   |Created    |26s (x3 over 42s)  |kubelet            |Created container init-service
  Warning  |Failed     |26s (x3 over 42s)  |kubelet            |Error: failed to start container "init-service": Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "": executable file not found in $PATH: unknown
  Warning  |BackOff    |9s (x3 over 24s)   |kubelet            |Back-off restarting failed container init-service in pod init-containers_default(a4fd0f3b-59fc-4d88-825a-412ddbabf082)

When you run kubectl get pods commands, herewith the display for those two scenarios

```txt
kubectl get pods
NAME              READY   STATUS                  RESTARTS     
init-containers   0/1     running                 0    
```

```txt
kubectl get pods
NAME              READY   STATUS                  RESTARTS     
init-containers   0/1     Init:CrashLoopBackOff   2    
```

## HEALTH PROBE
For the same purpose as above, some people can use init containers and some use health probe.
Herewith some comparison of the two.

| Feature | Health Probe | Init Containers |
|---------|--------------|----------------|
| Purpose | To check the health of a container and determine if it should receive traffic or be restarted | To perform initialization tasks before the main container starts |
| Execution | Periodically sends requests to a specified endpoint within the container and checks the response | Runs and completes before the main container starts |
| Failure Handling | If a health probe fails, the container is considered unhealthy and may be restarted or replaced | If an init container fails, the pod will not start and will remain in a pending state |
| Dependencies | Can depend on the container's readiness probe to determine if it should receive traffic | Can depend on other init containers to complete before it starts |
| Configuration | Configured using the readinessProbe and livenessProbe fields in the pod or container definition | Configured as a separate container within the pod's specification |
| Use Cases | Used to ensure that containers are healthy and ready to receive traffic | Used to perform tasks such as database initialization, downloading configuration files, or setting up shared volumes |
| Ordering | Health probes do not have a specific order and can run concurrently with other containers | Init containers are executed in order, with each init container waiting for the previous one to complete before starting |
| Restart Policy | Health probes can trigger container restarts based on their failure | Init containers do not trigger container restarts |
| Resource Usage | Health probes consume minimal resources as they are lightweight checks | Init containers can consume resources depending on the tasks they perform |
| Logging | Health probe results can be logged for monitoring and troubleshooting purposes | Init container logs can be used to track the initialization process and identify any issues |

Take a look at below container-probes.yaml below
```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 1
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
        livenessProbe:
          tcpSocket:
            port: 8081
          initialDelaySeconds: 10
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /
            port: 8081
          initialDelaySeconds: 10
          periodSeconds: 5
```

As you can see Health probes using child containers
  - livenessProbe
  - readinessProbe


| Probe Type | Purpose | Action on Failure |
|------------|---------|------------------|
| Liveness   | Checks if the container is running and responsive | Restart the container |
| Readiness  | Checks if the container is ready to receive traffic | Temporarily remove the container from the service's load balancer |
|------------|---------|------------------|

From the Events of kubectl describe pod would be normally like

  Type    |Reason     |Age   |From               |Message
  ----    |------     |----  |----               |-------
  Normal  |Scheduled  |19s   |default-scheduler  |Successfully assigned default/hello-world-596f8db57-bmdb9 to docker-desktop
  Normal  |Pulling    |17s   |kubelet            |Pulling image "psk8s.azurecr.io/hello-app:1.0"
  Normal  |Pulled     |11s   |kubelet            |Successfully pulled image "psk8s.azurecr.io/hello-app:1.0" in 5.913s (5.914s including waiting)
  Normal  |Created    |10s   |kubelet            |Created container hello-world
  Normal  |Started    |10s   |kubelet            |Started container hello-world

In case if an error occured, such as a security group not opening certain port

  Type     |Reason     |Age   |From               |Message
  ----     |------     |----  |----               |-------
  Normal   |Scheduled  |19s   |default-scheduler  |Successfully assigned default/hello-world2-67d9fc96bc-ct9k6 to docker-desktop
  Normal   |Pulled     |16s   |kubelet            |Container image "psk8s.azurecr.io/hello-app:1.0" already present on machine
  Normal   |Created    |16s   |kubelet            |Created container hello-world2
  Normal   |Started    |15s   |kubelet            |Started container hello-world2
  Warning  |Unhealthy  |3s    |kubelet            |Readiness probe failed: Get "http://10.1.1.151:8081/": dial tcp 10.1.1.151:8081: connect: connection refused
  Warning  |Unhealthy  |3s    |kubelet            |Liveness probe failed: dial tcp 10.1.1.151:8081: connect: connection refused

In above case, the port is still running although it is not functioning. <br>
In case we do not want this confusion, perhaps we can choose init containers.

<br><br>

# K8S DEPLOYMENT BASICS
As we know a kubernetes is a nanny of pods. <br>

## IMPERATIVE WAY TO SCALE
Herewith is an example of calling a generic image from Google.

```txt
kubectl create deployment nginx --image=nginx:latest
deployment.apps/nginx created

kubectl get pods,deploy
NAME                                       READY   STATUS             RESTARTS   AGE
pod/nginx-56fcf95486-rs59b                 1/1     Running            0          2m13s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx                  1/1     1            1           2m13s

```

Look below exercise attempting to delete a pod, in this case nginx-56fcf95486-rs59b.<br>
As soon as this pod was deleted, another one came out (-x5j74 replaced -rs59b). <br>
It sounds like a same technique to preserve pod deletion in Static Pod discussion above.<br>

```txt
kubectl get pods,deploy
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-56fcf95486-rs59b   1/1     Running   0          3h9m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           3h9m

kubectl delete pod nginx-56fcf95486-rs59b
pod "nginx-56fcf95486-rs59b" deleted

kubectl get pods,deploy
NAME                         READY   STATUS              RESTARTS   AGE
pod/nginx-56fcf95486-x5j74   0/1     ContainerCreating   0          7s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   0/1     1            0           3h12m
```
In this case the middle name of the pod 56fcf95486 was remain the same.
This id also can be found when we do
```txt
kubectl describe deploy nginx
```
and take a look at NewReplicaSet, where we can find the same information.
```txt
NewReplicaSet:   nginx-56fcf95486 (1/1 replicas created)
```

Let's try to populate the same deployment with more replicas like below.<br>
Also see the same attempt to delete one of the pod and same thing happened.
```txt
kubectl create deployment nginx --image=nginx:latest --replicas=5
deployment.apps/nginx created

kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-56fcf95486-4mt5w   1/1     Running   0          9m47s
pod/nginx-56fcf95486-7kpxx   1/1     Running   0          9m47s
pod/nginx-56fcf95486-85c4x   1/1     Running   0          9m47s
pod/nginx-56fcf95486-qnzp2   1/1     Running   0          9m47s
pod/nginx-56fcf95486-xnttn   1/1     Running   0          9m47s

kubectl delete pod nginx-56fcf95486-xnttn
pod "nginx-56fcf95486-xnttn" deleted

kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-56fcf95486-4mt5w   1/1     Running   0          10m
pod/nginx-56fcf95486-7kpxx   1/1     Running   0          10m
pod/nginx-56fcf95486-85c4x   1/1     Running   0          10m
pod/nginx-56fcf95486-qnzp2   1/1     Running   0          10m
pod/nginx-56fcf95486-rhj8h   1/1     Running   0          8s
```
## USING MANIFEST TO SCALE
The declarative style is based on the use of manifests (YAML files). Compared to imperative, this approach makes it easy to put our deployment manifests into source control, increasing their reusability and maintenance.

Although it might not be related, pipeline and freestyle approach to build CI/CD in Jenkins night be comparable to
imperative and declarative approach to build pods in Kubernetes in term of limited to more flexible of configuration. Although in Jenkins pipeline style (less flexible way) is more popular and in Kubernetes it is declarative one (more flexible) that is more popular.

![](/images/05-image08.png)<br>
Figure 7: Creating Deployments Declaratively  
<br>

Let's see different way using manifest using the same hello-world.yaml
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
### Label
Let's see the removing and editing label to one of the pod like below.<br>
When deployment was deleted, pod with edited label won't be removed.

```txt
kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx-56fcf95486-4mt5w   1/1     Running   0          25m   app=nginx,pod-template-hash=56fcf95486
nginx-56fcf95486-7kpxx   1/1     Running   0          25m   app=nginx,pod-template-hash=56fcf95486
nginx-56fcf95486-85c4x   1/1     Running   0          25m   app=nginx,pod-template-hash=56fcf95486
nginx-56fcf95486-qnzp2   1/1     Running   0          25m   app=nginx,pod-template-hash=56fcf95486
nginx-56fcf95486-rhj8h   1/1     Running   0          15m   app=nginx,pod-template-hash=56fcf95486

kubectl label pod nginx-56fcf95486-4mt5w app-
pod/nginx-56fcf95486-4mt5w unlabeled

kubectl label --overwrite pod nginx-56fcf95486-4mt5w workload=staging
pod/nginx-56fcf95486-4mt5w labeled

kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx-56fcf95486-4mt5w   1/1     Running   0          35m   pod-template-hash=56fcf95486,workload=staging
nginx-56fcf95486-7kpxx   1/1     Running   0          35m   app=nginx,pod-template-hash=56fcf95486
nginx-56fcf95486-85c4x   1/1     Running   0          35m   app=nginx,pod-template-hash=56fcf95486
nginx-56fcf95486-mh2p5   1/1     Running   0          29s   app=nginx,pod-template-hash=56fcf95486
nginx-56fcf95486-qnzp2   1/1     Running   0          35m   app=nginx,pod-template-hash=56fcf95486
nginx-56fcf95486-rhj8h   1/1     Running   0          24m   app=nginx,pod-template-hash=56fcf95486

kubectl delete deployment nginx
deployment.apps "nginx" deleted

kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-56fcf95486-4mt5w   1/1     Running   0          39m
```

Let's change the hello-world.yaml having replicas=2 instead.<br>
Previous left over pod was considered as 1 replica and the new one coming with standard label.
```txt
nano hello-world.yaml # changing replicas=2 from replicas=5

kubectl create deployment nginx --image=nginx:latest
deployment.apps/nginx created

kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx-56fcf95486-4mt5w   1/1     Running   0          46m   pod-template-hash=56fcf95486,workload=staging
nginx-56fcf95486-5rjz4   1/1     Running   0          7s    app=nginx,pod-template-hash=56fcf95486
```

Changing the label is the easiest way to isolate the pod for troubleshoot.

### Pod, ReplicaSet and Deployment
In fact it is ReplicaSet acting as the Pod nanny, not Deployment.

![](/images/05-image07.png)
Figure 8: ReplicaSet is the one that actually managing the pods  
<br>

```txt
kubectl get all -l app=nginx
NAME                         READY   STATUS    RESTARTS   
pod/nginx-56fcf95486-5rjz4   1/1     Running   0          

NAME                    READY   UP-TO-DATE   AVAILABLE   
deployment.apps/nginx   1/1     1            1           

NAME                               DESIRED   CURRENT   READY   
replicaset.apps/nginx-56fcf95486   1         1         1      



kubectl get pod,deploy,rs
NAME                         READY   STATUS    RESTARTS   
pod/nginx-56fcf95486-4mt5w   1/1     Running   0          
pod/nginx-56fcf95486-5rjz4   1/1     Running   0          

NAME                    READY   UP-TO-DATE   AVAILABLE   
deployment.apps/nginx   1/1     1            1           

NAME                               DESIRED   CURRENT   READY   
replicaset.apps/nginx-56fcf95486   1         1         1      
```

### Avoiding Down Time During Upgrade

Let's take another scenario with deployment-manifest.yaml file below where we will save it into deployment.yaml

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
 labels:
   app: hello-world
 name: hello-world-deployment
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
     name: hello-world-container
     ports:
     - containerPort: 8080
```

Take a look at below process, see how the pods were replaced staggeringly to avoid downtime

```txt
kubectl create -f deployment.yaml
deployment.apps/hello-world-deployment created

kubectl describe deployment hello-world
...
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  82s   deployment-controller  Scaled up replica set hello-world-deployment-75554c97db to 5

nano deployment.yaml   # here we simply change the image from V1 to V2

kubectl apply -f deployment.yaml
deployment.apps/hello-world-deployment configured

kubectl describe deployment hello-world
...
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m53s  deployment-controller  Scaled up replica set hello-world-deployment-75554c97db to 5
  Normal  ScalingReplicaSet  3m3s   deployment-controller  Scaled up replica set hello-world-deployment-7bd6455957 to 2
  Normal  ScalingReplicaSet  3m3s   deployment-controller  Scaled down replica set hello-world-deployment-75554c97db to 4 from 5
  Normal  ScalingReplicaSet  3m2s   deployment-controller  Scaled up replica set hello-world-deployment-7bd6455957 to 3 from 2
  Normal  ScalingReplicaSet  2m48s  deployment-controller  Scaled down replica set hello-world-deployment-75554c97db to 3 from 4
  Normal  ScalingReplicaSet  2m48s  deployment-controller  Scaled up replica set hello-world-deployment-7bd6455957 to 4 from 3
  Normal  ScalingReplicaSet  2m46s  deployment-controller  Scaled down replica set hello-world-deployment-75554c97db to 2 from 3
  Normal  ScalingReplicaSet  2m45s  deployment-controller  Scaled up replica set hello-world-deployment-7bd6455957 to 5 from 4
  Normal  ScalingReplicaSet  2m44s  deployment-controller  Scaled down replica set hello-world-deployment-75554c97db to 1 from 2
  Normal  ScalingReplicaSet  2m40s  deployment-controller  (combined from similar events): Scaled down replica set hello-world-deployment-75554c97db to 0 from 1
```

In summary this is what the above message tell us in 8 different stages.<br>
At any given time there would be six (replicas=5 + 1 more to avoid down time = 6).
If you use K9S you could see live how older pods are gradually removed from five to zero and new pods are gradually created from zero to five.

```txt
step 1 & 2
              Scaled up   new replica set from 1 to 2 
              Scaled down old replica set from 5 to 4 
                                              ---+ ---+
                                              6    6
step 3 & 4
              Scaled up   new replica set from 2 to 3 
              Scaled down old replica set from 4 to 3 
                                              ---+ ---+
                                              6    6
step 5 & 6
              Scaled up   new replica set from 3 to 4 
              Scaled down old replica set from 3 to 2 
                                              ---+ ---+
                                              6    6
step 7 & 8
              Scaled up   new replica set from 4 to 5 
              Scaled down old replica set from 2 to 1
                                              ---+ ---+
                                              6    6
```

from kubectl describe command we will get a lengthy result but the one that we are interested is in RollingUpdateStrategy. 25% of 5 pods is about 1 pods, that's why at any given time about we have 1 more pod at works during transitioning.

```txt
kubectl describe deployment hello-world
...
RollingUpdateStrategy:  25% max unavailable, 25% max surge
...
```

In summary Kubernetes is giving an illusion that everything is working fine but in the back end it is removing some pods and adding some pods.<br>

Let's see another example of rollingUpdate strategy by makine a mistake as follow.<br>
Kubernetes by default uses RollingUpdate to avoid sudden change and maintain as many uptime as possible.

```txt
# step 1: see current pods
kubectl get pods     
NAME                                      READY   STATUS    RESTARTS   AGE
hello-world-deployment-7bd6455957-95n8r   1/1     Running   0          7h33m
hello-world-deployment-7bd6455957-fc89f   1/1     Running   0          7h33m
hello-world-deployment-7bd6455957-s7cfr   1/1     Running   0          7h33m
hello-world-deployment-7bd6455957-sgm8r   1/1     Running   0          7h33m
hello-world-deployment-7bd6455957-vk8s2   1/1     Running   0          7h33m
hello-world-ds-rlhjv                      1/1     Running   0          6h47m

# step 2: see current yaml file
cat deployment.yaml  
   ...
     spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
   ...

# step 3: make some silly mistake in yaml file
nano deployment.yaml 

# step 4: Notice that app:1.0 become ap:1.0 (wrong image, never been found)
cat deployment.yaml  
   ...  spec:
         containers:
         - image: gcr.io/google-samples/hello-ap:1.0
   ...

# step 5: update the changes
kubectl apply -f deployment.yaml
deployment.apps/hello-world-deployment configured

# --> step 6: show 4  = 5 - 1 (round down from 25% of 5) running hello-world-deployment
#             and  3 (round up from 25% of 5) ImagePullBackOff hello-world-deployment
kubectl get pods                           
NAME                                      READY   STATUS             RESTARTS   AGE
hello-world-deployment-7bd6455957-fc89f   1/1     Running            0          7h36m
hello-world-deployment-7bd6455957-s7cfr   1/1     Running            0          7h36m
hello-world-deployment-7bd6455957-sgm8r   1/1     Running            0          7h36m
hello-world-deployment-7bd6455957-vk8s2   1/1     Running            0          7h36m
hello-world-deployment-7fcb559cfb-8plnd   0/1     ImagePullBackOff   0          25s
hello-world-deployment-7fcb559cfb-hmwvq   0/1     ImagePullBackOff   0          26s
hello-world-deployment-7fcb559cfb-r4mvz   0/1     ImagePullBackOff   0          26s
hello-world-ds-rlhjv                      1/1     Running            0          6h50m
```

### RollingUpdate vs Recreate Update Strategy

| Update Strategy | Explanation |
| --- | --- |
| RollingUpdate | Updates pods gradually, one by one |
| Recreate | Terminates all pods, then creates new ones |

using the same deployment.yaml and tweak it as follow.
By default the update strategy was RollingUpdate, herewith how we overwite it into Recreate. You may delete all previous deployment.

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello-world
  name: hello-world-deployment
  namespace: default
spec:
  replicas: 5
  selector:
    matchLabels:
      app: hello-world
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-world-container
        ports:
        - containerPort: 8080
```
Using the same steps for RollingUpdate herewith how Recreate looks like
```txt
kubectl create -f recreate.yaml
deployment.apps/hello-world-deployment created

kubectl describe deployment hello-world-deployment
...
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  37s   deployment-controller  Scaled up replica set hello-world-deployment-75554c97db to 5

nano recreate.yaml  # --> change hello-app:1.0 to hello-app:2.0

kubectl apply -f recreate.yaml
deployment.apps/hello-world-deployment configured

kubectl describe deployment hello-world-deployment
...
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  82s   deployment-controller  Scaled up replica set hello-world-deployment-75554c97db to 5
  Normal  ScalingReplicaSet  21s   deployment-controller  Scaled down replica set hello-world-deployment-75554c97db to 0 from 5
  Normal  ScalingReplicaSet  14s   deployment-controller  Scaled up replica set hello-world-deployment-7bd6455957 to 5

```
### what happen if the deployment is struggling?
I will pick this example below from deployment-broken.yaml and saved it into broken.yaml In this case we add /change
- progressDeadlineSeconds: 10
- make an error from hello-api:2.0 --> hello-ap:2.0<br>

You may need to delete previous deployment to try this one.
```txt
apiVersion: apps/v1
kind: Deployment
metadata:
 labels:
   app: hello-world
 name: hello-world-deployment
 namespace: default
spec:
 progressDeadlineSeconds: 10
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
   - image: gcr.io/google-samples/hello-ap:1.0  
     name: hello-world-container
     ports:
     - containerPort: 8080
```

As soon as we fix
- from gcr.io/google-samples/hello-ap:1.0
- into gcr.io/google-samples/hello-app:1.0

The deployment will gradually running well.

## ROLLOUT HISTORY
We might cover this already in previous blog, herewith is the example

```txt
nano broken.yaml  # --> 

kubectl apply -f broken.yaml --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/hello-world-deployment configured

kubectl rollout history deployment hello-world-deployment
deployment.apps/hello-world-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl apply --filename=broken.yaml --record=true
```

## DAEMONSET
A daemonset ensures that a copy of a Pod is running across a set of nodes in Kubernets cluster. DaemonSets are used to deploy system daemons such as log collectors and monitoring agents which typically must run on every node. <br>

Let's try this DaemonSet.yaml below
```txt
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: hello-world-ds
spec:
  selector:
    matchLabels:
      app: hello-world-app
  template:
    metadata:
      labels:
        app: hello-world-app
    spec:
      containers:
        - name: hello-world
          image: psk8s.azurecr.io/hello-app:1.0
```



And herewith is the result
```txt
kubectl get daemonsets --namespace default
NAME             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
hello-world-ds   1         1         0       1            0           <none>          9s
```

This daemonset is actually one of the pods, see below
```txt
kubectl get pods --show-labels
NAME                                      READY   STATUS    RESTARTS   AGE     LABELS
hello-world-deployment-7bd6455957-95n8r   1/1     Running   0          19m     app=hello-world,pod-template-hash=7bd6455957
hello-world-deployment-7bd6455957-fc89f   1/1     Running   0          19m     app=hello-world,pod-template-hash=7bd6455957
hello-world-deployment-7bd6455957-s7cfr   1/1     Running   0          19m     app=hello-world,pod-template-hash=7bd6455957
hello-world-deployment-7bd6455957-sgm8r   1/1     Running   0          19m     app=hello-world,pod-template-hash=7bd6455957
hello-world-deployment-7bd6455957-vk8s2   1/1     Running   0          19m     app=hello-world,pod-template-hash=7bd6455957
hello-world-ds-tn6hj                      1/1     Running   0          4m36s   app=hello-world-app,controller-revision-hash=67f975cb86,pod-template-generation=1

kubectl get pods -l controller-revision-hash=67f975cb86
NAME                   READY   STATUS    RESTARTS   AGE
hello-world-ds-tn6hj   1/1     Running   0          6m54s

kubectl get pods -l controller-revision-hash=67f975cb86 -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP          NODE             NOMINATED NODE   READINESS GATES
hello-world-ds-tn6hj   1/1     Running   0          7m18s   10.1.2.39   docker-desktop   <none>           <none>
```
As we seen before, this daemon set also can ping the other five IP as below
```txt
kubectl exec -it hello-world-ds-tn6hj /bin/sh
/app # ping 10.1.2.39
PING 10.1.2.39 (10.1.2.39): 56 data bytes
64 bytes from 10.1.2.39: seq=0 ttl=64 time=0.028 ms
64 bytes from 10.1.2.39: seq=1 ttl=64 time=0.090 ms
64 bytes from 10.1.2.39: seq=2 ttl=64 time=0.082 ms

/app # ping 10.1.2.38
PING 10.1.2.38 (10.1.2.38): 56 data bytes
64 bytes from 10.1.2.38: seq=0 ttl=64 time=0.377 ms
64 bytes from 10.1.2.38: seq=1 ttl=64 time=0.121 ms
64 bytes from 10.1.2.38: seq=2 ttl=64 time=0.177 ms
```

Last but not least, let's see if the new daemonset.yaml where we add below could catch the selector.<br>
Last command shows the NODE SELECTOR with node=my-fav-node could catch the pods that we just labeled.

```txt
kubectl get nodes
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   8h    v1.28.2

kubectl get nodes -o wide
NAME             STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION    CONTAINER-RUNTIME
docker-desktop   Ready    control-plane   8h    v1.28.2   192.168.65.9   <none>        Docker Desktop   6.5.11-linuxkit   docker://24.0.7

kubectl label node docker-desktop node=my-fav-node
node/docker-desktop labeled

kubectl get nodes --show-labels
NAME             STATUS   ROLES           AGE   VERSION   LABELS
docker-desktop   Ready    control-plane   8h    v1.28.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=docker-desktop,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=,node=my-fav-node

kubectl apply -f daemonset.yaml
daemonset.apps/hello-world-ds configured

kubectl get ds 
NAME             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR      AGE
hello-world-ds   1         1         1       1            1           node=my-fav-node   32m
```

## NAMESPACE
You may find Namespace often in yaml file, if it is not mentioned it will be
```txt
namespace: default
```
We can also find namespace in kubeconfig (kube/.config) in relation to username, etc
![](/images/05-image09.png)
Figure 9: Relation between Namespace and other attributes in Kubernetes
<br><br>

Namespace must be created first before it was written in yaml file. <br>
These are the syntax to create and delete namespaces.
```txt
kubectl create namespace <namespace-name>
kubectl delete namespace <namespace-name>
```

Unlike context, we don't have manually switching default namespace like below.<br>
```txt
kubectl config use-context new-context
Switched to context "new-context".
```

| Aspect   | Context  | Namespace |
|----------|----------|-----------|
| Definition | A context in Kubernetes is a set of access parameters for a cluster, including the cluster's server, authentication credentials, and default namespace. It allows users to switch between different clusters or authentication settings easily. | A namespace in Kubernetes is a virtual cluster within a physical cluster. It provides a way to divide cluster resources between multiple users or teams, ensuring isolation and resource management. |
| Usage | Contexts are used to manage multiple clusters or authentication settings. They allow users to switch between different clusters or authentication configurations easily without having to reconfigure their command-line tools. | Namespaces are used to divide cluster resources between multiple users or teams. They provide a way to organize and manage resources within a cluster, ensuring isolation and resource management. |
| Scope | Contexts are used at the cluster level. They define the access parameters for a cluster, including the server URL, authentication credentials, and default namespace. | Namespaces are used at the resource level. They provide a way to organize and manage resources within a cluster, such as pods, services, and deployments. Each resource belongs to a specific namespace. |
| Example | An example of a context could be a development cluster with its server URL, authentication credentials, and default namespace set. Another context could be a production cluster with its own server URL, authentication credentials, and default namespace. | An example of a namespace could be a "dev" namespace used by a development team to deploy and manage their resources. Another namespace could be a "prod" namespace used by a production team to deploy and manage their resources. |
| Relationship | Contexts can be associated with one or more namespaces. When switching between contexts, the default namespace can also be changed. | Namespaces are independent of each other and can exist within different contexts. Multiple namespaces can coexist within a single context. |

An easy situation to explain where namespace was needed would be in a situation where an organization does not have luxury to operate multiple cluster. In that case, finance department and HR department for example has to use the same cluster but in different namespaces and to add further restriction who can access the namespace. This is an example how namespace can isolate some of departmental objects as part of primary objective of Kubernetes.

## JOBS, PARALLELISM, CRONJOB
### Jobs
There is another command in Kubernetes called jobs which usually run like below
```txt
kubectl get jobs
```
You may wonder how jobs, deployment and DaemonSet different to each other.

| Feature       | Jobs           | DaemonSet      | Deployments    |
| ------------- | -------------- | -------------- | -------------- |
| Purpose       | Run batch tasks| Run on all nodes| Manage app updates|
| Scaling       | Manual scaling | Auto scaling   | Auto scaling   |
| Pods          | One or more    | One per node   | Multiple pods  |
| Completion    | Succeed or fail| Run indefinitely| Rolling updates|

Deployment and DaemonSet are continuously going, For example if a pod was down a new one will replace it instantly. Job may do the same thing but not running continuously. Typically we are using this approach to re-populate the database in an app. <br>

Herewith an example
```txt
#STEP 1: adding a new yaml file like below
nano job.yaml

#STEP 2: example of job yaml file
cat job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-world-job
spec:
  template:
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command:
         - "/bin/bash"
         - "-c"
         - "/bin/echo Hello from Pod $(hostname) at $(date)"
      restartPolicy: Never

#STEP 3: Create a Job
kubectl create -f job.yaml
job.batch/hello-world-job created

#STEP 4: Review above newly created job
kubectl get jobs
NAME              COMPLETIONS   DURATION   
hello-world-job   1/1           19s        
```
Look at these two important attributes inside a job

```txt
kubectl describe job hello-world-job
  ...
    Parallelism:      1
    Completions:      1
  ...
```
### Parallelism
Herewith an example of a parallel jobs in Kubernetes
![](/images/05-image10.png)
Figure 10: Kubernetes Parallel Jobs (Batch and Seconds above were generated from Excel)
<br>
```txt
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-job
spec:
  completions: 50
  parallelism: 10
  template:
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command:
         - "/bin/bash"
         - "-c"
         - "/bin/echo Hello from Pod $(hostname) at $(date)"
      restartPolicy: Never
```
By default it is a bit tricky to know which pod belong to which batch of the parallel job.
I identified the first 10 jobs (which is likely starting at the same time) as the first serial jobs at each parallel groups.
```txt
devops@msi:~$ kubectl get pods -o custom-columns=NAME:.metadata.name,START_TIME:.status.startTime,COMPLETION_TIME:.status.completionTime
NAME                START_TIME            COMPLETION_TIME
parallel-job-5l6d4	2024-01-29T18:08:31Z	<none>
parallel-job-tjvjr	2024-01-29T18:08:31Z	<none>
parallel-job-6z8nf	2024-01-29T18:08:32Z	<none>
parallel-job-9t5c9	2024-01-29T18:08:32Z	<none>
parallel-job-hpfpp	2024-01-29T18:08:32Z	<none>
parallel-job-kntfq	2024-01-29T18:08:32Z	<none>
parallel-job-p6hlh	2024-01-29T18:08:32Z	<none>
parallel-job-t9k2k	2024-01-29T18:08:32Z	<none>
parallel-job-xwghn	2024-01-29T18:08:32Z	<none>
parallel-job-z6npw	2024-01-29T18:08:32Z	<none>
parallel-job-5btsf	2024-01-29T18:08:48Z	<none>
parallel-job-bgfcx	2024-01-29T18:08:52Z	<none>
parallel-job-m254s	2024-01-29T18:08:55Z	<none>
...
```
Then I added manually a new column called batch
```txt
NAME                START TIME            BATCH
parallel-job-5l6d4	2024-01-29T18:08:31Z	1
parallel-job-tjvjr	2024-01-29T18:08:31Z	2
parallel-job-6z8nf	2024-01-29T18:08:32Z	3
parallel-job-9t5c9	2024-01-29T18:08:32Z	4
parallel-job-hpfpp	2024-01-29T18:08:32Z	5
parallel-job-kntfq	2024-01-29T18:08:32Z	6
parallel-job-p6hlh	2024-01-29T18:08:32Z	7
parallel-job-t9k2k	2024-01-29T18:08:32Z	8
parallel-job-xwghn	2024-01-29T18:08:32Z	9
parallel-job-z6npw	2024-01-29T18:08:32Z	10
parallel-job-5btsf	2024-01-29T18:08:48Z	1
parallel-job-bgfcx	2024-01-29T18:08:52Z	2
parallel-job-m254s	2024-01-29T18:08:55Z	3
parallel-job-4l6ph	2024-01-29T18:08:59Z	4
parallel-job-5jj4f	2024-01-29T18:09:04Z	5
parallel-job-9b48q	2024-01-29T18:09:05Z	6
parallel-job-vt522	2024-01-29T18:09:10Z	7
parallel-job-pgtk8	2024-01-29T18:09:12Z	8
parallel-job-vth69	2024-01-29T18:09:12Z	9
parallel-job-btzhl	2024-01-29T18:09:14Z	10
parallel-job-jklcf	2024-01-29T18:09:14Z	1
parallel-job-8mgm8	2024-01-29T18:09:21Z	2
parallel-job-cdvm4	2024-01-29T18:09:23Z	3
parallel-job-6n9h9	2024-01-29T18:09:24Z	4
parallel-job-n5fc8	2024-01-29T18:09:30Z	5
parallel-job-vtddx	2024-01-29T18:09:33Z	6
parallel-job-s2vhd	2024-01-29T18:09:36Z	7
parallel-job-h85fm	2024-01-29T18:09:37Z	8
parallel-job-9l6sr	2024-01-29T18:09:41Z	9
parallel-job-zff6x	2024-01-29T18:09:41Z	10
parallel-job-665pm	2024-01-29T18:09:48Z	1
parallel-job-8d9v6	2024-01-29T18:09:51Z	2
parallel-job-6w5wk	2024-01-29T18:09:55Z	3
parallel-job-jh7wk	2024-01-29T18:09:55Z	4
parallel-job-2ph22	2024-01-29T18:09:58Z	5
parallel-job-wcml4	2024-01-29T18:10:00Z	6
parallel-job-nd6vf	2024-01-29T18:10:04Z	7
parallel-job-w4mnc	2024-01-29T18:10:08Z	8
parallel-job-55cxx	2024-01-29T18:10:09Z	9
parallel-job-84xzq	2024-01-29T18:10:11Z	10
parallel-job-b2csc	2024-01-29T18:10:12Z	1
parallel-job-fjhnx	2024-01-29T18:10:14Z	2
parallel-job-sxtkx	2024-01-29T18:10:19Z	3
parallel-job-xcmdc	2024-01-29T18:10:22Z	4
parallel-job-879cg	2024-01-29T18:10:24Z	5
parallel-job-tqzl2	2024-01-29T18:10:26Z	6
parallel-job-tcxmj	2024-01-29T18:10:29Z	7
parallel-job-fkpm8	2024-01-29T18:10:32Z	8
parallel-job-pwq66	2024-01-29T18:10:34Z	9
parallel-job-mqsdq	2024-01-29T18:10:36Z	10
```
And I added manually the duration column in second as the start time difference between the next tenth row and its own start time. As the last ten rows does not have subsequent 10 values to build the duration, I use the average of previous four as its duration.
```txt
NAME                START TIME            BATCH SECONDS
parallel-job-5l6d4	2024-01-29T18:08:31Z	1	    17
parallel-job-tjvjr	2024-01-29T18:08:31Z	2	    21
parallel-job-6z8nf	2024-01-29T18:08:32Z	3	    23
parallel-job-9t5c9	2024-01-29T18:08:32Z	4	    27
parallel-job-hpfpp	2024-01-29T18:08:32Z	5	    32
parallel-job-kntfq	2024-01-29T18:08:32Z	6	    33
parallel-job-p6hlh	2024-01-29T18:08:32Z	7	    38
parallel-job-t9k2k	2024-01-29T18:08:32Z	8	    40
parallel-job-xwghn	2024-01-29T18:08:32Z	9	    40
parallel-job-z6npw	2024-01-29T18:08:32Z	10	  42
parallel-job-5btsf	2024-01-29T18:08:48Z	1	    26
parallel-job-bgfcx	2024-01-29T18:08:52Z	2	    29
parallel-job-m254s	2024-01-29T18:08:55Z	3	    28
parallel-job-4l6ph	2024-01-29T18:08:59Z	4	    25
parallel-job-5jj4f	2024-01-29T18:09:04Z	5	    26
parallel-job-9b48q	2024-01-29T18:09:05Z	6	    28
parallel-job-vt522	2024-01-29T18:09:10Z	7	    26
parallel-job-pgtk8	2024-01-29T18:09:12Z	8	    25
parallel-job-vth69	2024-01-29T18:09:12Z	9	    29
parallel-job-btzhl	2024-01-29T18:09:14Z	10	  27
parallel-job-jklcf	2024-01-29T18:09:14Z	1	    34
parallel-job-8mgm8	2024-01-29T18:09:21Z	2	    30
parallel-job-cdvm4	2024-01-29T18:09:23Z	3	    32
parallel-job-6n9h9	2024-01-29T18:09:24Z	4	    31
parallel-job-n5fc8	2024-01-29T18:09:30Z	5	    28
parallel-job-vtddx	2024-01-29T18:09:33Z	6	    27
parallel-job-s2vhd	2024-01-29T18:09:36Z	7	    28
parallel-job-h85fm	2024-01-29T18:09:37Z	8	    31
parallel-job-9l6sr	2024-01-29T18:09:41Z	9	    28
parallel-job-zff6x	2024-01-29T18:09:41Z	10	  30
parallel-job-665pm	2024-01-29T18:09:48Z	1	    24
parallel-job-8d9v6	2024-01-29T18:09:51Z	2	    23
parallel-job-6w5wk	2024-01-29T18:09:55Z	3	    24
parallel-job-jh7wk	2024-01-29T18:09:55Z	4	    27
parallel-job-2ph22	2024-01-29T18:09:58Z	5	    26
parallel-job-wcml4	2024-01-29T18:10:00Z	6	    26
parallel-job-nd6vf	2024-01-29T18:10:04Z	7	    25
parallel-job-w4mnc	2024-01-29T18:10:08Z	8	    24
parallel-job-55cxx	2024-01-29T18:10:09Z	9	    25
parallel-job-84xzq	2024-01-29T18:10:11Z	10	  25
parallel-job-b2csc	2024-01-29T18:10:12Z	1	    25
parallel-job-fjhnx	2024-01-29T18:10:14Z	2	    25
parallel-job-sxtkx	2024-01-29T18:10:19Z	3	    26
parallel-job-xcmdc	2024-01-29T18:10:22Z	4	    27
parallel-job-879cg	2024-01-29T18:10:24Z	5	    28
parallel-job-tqzl2	2024-01-29T18:10:26Z	6	    28
parallel-job-tcxmj	2024-01-29T18:10:29Z	7	    29
parallel-job-fkpm8	2024-01-29T18:10:32Z	8	    30
parallel-job-pwq66	2024-01-29T18:10:34Z	9	    30
parallel-job-mqsdq	2024-01-29T18:10:36Z	10	  31
```
By having these additional columns (preferably built in Excel) we can then build a schematic above.

### Cronjob
A cronjob is a time-based job scheduler in Unix-like operating systems. It allows users to schedule and automate the execution of commands or scripts at specified intervals or specific times. Cronjobs are commonly used for tasks such as system maintenance, backups, and running periodic scripts or programs. They are configured using the cron daemon, which reads a crontab (cron table) file that contains the schedule and commands for each job.

Let's tweak job.yaml into cronjob.yaml like below by adding/replacing couple lines
- kind: CronJob
- schedule: "*/1 * * * *"
```txt
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-world-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command:
         - "/bin/bash"
         - "-c"
         - "/bin/echo Hello from Pod $(hostname) at $(date)"
      restartPolicy: Never
```
The expression "*/1 * * * *" [means](https://crontab.guru/#*/1_*_*_*_*) the job will be run at every minute

[Herewith](https://www.netiq.com/documentation/cloud-manager-2-5/ncm-reference/data/bexyssf.html) is the good reference to understand cron syntax, such as below
|Cron Expression 		|Example Description
|----				|----
|0 0 12 * * ?			|Fire at 12:00 p.m. (noon) every day
|0 15 10 ? * *			|Fire at 10:15 a.m. every day
|0 15 10 * * ?			|Fire at 10:15 a.m. every day
|0 15 10 * * ? *		|Fire at 10:15 a.m. every day
|0 15 10 * * ? 2012		|Fire at 10:15 a.m. every day during the year 2012
|0 * 14 * * ?			|Fire every minute starting at 2:00 p.m. and ending at 2:59.p.m., every day
|0 0/5 14 * * ?			|Fire every five minutes starting at 2:00 p.m. and ending at 2:55 p.m., every day
|0 0/5 14,18 * * ?		|Fire every five minutes starting at 2:00 p.m. and ending at 2:55 p.m., and fire every five minutes starting at 6:00 p.m. and ending at 6:55 p.m., every day
|0 0-5 14 * * ?			|Fire every minute starting at 2:00 p.m. and ending at 2:05.p.m., every day
|0 10,44 14 ? 3 WED		|Fire at 2:10 p.m. and at 2:44 p.m. every Wednesday in the month of March
|0 15 10 ? * MON-FRI		|Fire at 10:15 a.m. every Monday, Tuesday, Wednesday, Thursday and Friday
|0 15 10 15 * ?			|Fire at 10:15 a.m. on the 15th day of every month
|0 15 10 15 * ?			|Fire at 10:15 a.m. on the last day of every month
|0 15 10 ? * 6L			|Fire at 10:15 a.m. on the last Friday of every month
|0 15 10 ? * 6L 2011-2014	|Fire at 10:15 a.m. on every last Friday of every month during the years 2011, 20012, 2014, and 2014
|0 15 10 ? * 6#3		|Fire at 10:15 a.m. on the third Friday of every month
|0 0 12 1/5 * ?			|Fire at 12:00 p.m. (noon) every five days every month, starting on the first day of the month
|0 11 11 11 11 ?		|Fire every November 11th at 11:11 a.m.

Lets's see below example

```txt
# STEP1: CREATE CRONJOB LIKE BELOW
nano cronjob.yaml

# STEP2: REVIEW THE CODE
cat cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-world-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: hello-world
            image: ubuntu
            command: ["echo", "hello world"]

# STEP 3: CREATE CRONJOB
kubectl create -f cronjob.yaml
cronjob.batch/hello-world-cronjob created

# STEP 4: REVIEW EVERY MINUTE PODS, by default Kubernetes will only show the last three
kubectl get pods
NAME                                 READY   STATUS      RESTARTS   AGE
hello-world-cronjob-28442703-6jnjc   0/1     Completed   0          2m37s
hello-world-cronjob-28442704-fzz4l   0/1     Completed   0          97s
hello-world-cronjob-28442705-7hpfq   0/1     Completed   0          37s
hello-world-job-z8fwm                0/1     Completed   0          16m
```

## TAINTS AND SCHEDULING
In K8S, when the Deployment like below launched, normally control panel or master is tainted and none of the pods would be on master / control-node. <br>
In my case, the kubernetes cluster with K3S or Docker Kubernetes cluster was not tainted.
In such cases, master nodes can accept pods from deployment.

```txt
STEP1: create and review deployment.yaml
nano deployment.yaml
cat deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 10
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

STEP2: launch deployment
kubectl create -f deployment.yaml

STEP 3: See that NODE column has 50% master and 50% worker
kubectl get pods -o wide
NAME                           READY   STATUS              RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
hello-world-5f64785679-bwrtw   0/1     ContainerCreating   0          12s   <none>   worker   <none>           <none>
hello-world-5f64785679-vknwv   0/1     ContainerCreating   0          12s   <none>   master   <none>           <none>
hello-world-5f64785679-hk7cz   0/1     ContainerCreating   0          12s   <none>   worker   <none>           <none>
hello-world-5f64785679-jqg66   0/1     ContainerCreating   0          12s   <none>   master   <none>           <none>
hello-world-5f64785679-86vln   0/1     ContainerCreating   0          12s   <none>   master   <none>           <none>
hello-world-5f64785679-q7gxv   0/1     ContainerCreating   0          12s   <none>   worker   <none>           <none>
hello-world-5f64785679-wxjq7   0/1     ContainerCreating   0          12s   <none>   master   <none>           <none>
hello-world-5f64785679-dfn2j   0/1     ContainerCreating   0          12s   <none>   master   <none>           <none>
hello-world-5f64785679-gnnzd   0/1     ContainerCreating   0          12s   <none>   worker   <none>           <none>
hello-world-5f64785679-tf69j   0/1     ContainerCreating   0          12s   <none>   worker   <none>           <none>

STEP 4: In this case we only have one worker and one master
kubectl get nodes
NAME     STATUS   ROLES                  AGE    VERSION
master   Ready    control-plane,master   6d2h   v1.28.5+k3s1
worker   Ready    <none>                 6d2h   v1.28.5+k3s1

```

These are the annotations and taints from K3S. 

```txt
# ANNOTATIONS AND TAINTS FROM K3S
kubectl describe nodes master
Name:               master
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/instance-type=k3s
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=master
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=true
                    node-role.kubernetes.io/master=true
                    node.kubernetes.io/instance-type=k3s
Annotations:        alpha.kubernetes.io/provided-node-ip: 10.73.136.115
                    flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"32:f3:9e:5f:0e:68"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 10.73.136.115
                    k3s.io/hostname: master
                    k3s.io/internal-ip: 10.73.136.115
                    k3s.io/node-args: ["server"]
                    k3s.io/node-config-hash: 4X6FFBYGNZCCKC4WZILLGNG5RGJQDDYNTTQWYJSFA474RBARXQ4A====
                    k3s.io/node-env: {"K3S_DATA_DIR":"/var/lib/rancher/k3s/data/28f7e87eba734b7f7731dc900e2c84e0e98ce869f3dcf57f65dc7bbb80e12e56"}
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 23 Jan 2024 13:54:40 -0500
Taints:             <none>
```
and these are the annotations and taints from Docker's K8S.

```txt
# ANNOTATIONS AND TAINTS FROM Docker's K8S
kubectl describe node docker-desktop
Name:               docker-desktop
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=docker-desktop
                    kubernetes.io/os=linux
                    node=my-fav-node
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/cri-dockerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
                    CreationTimestamp:  Sun, 28 Jan 2024 17:40:47 -0500
Taints:             <none>
```

If you see your taints like
```txt
Taints: node.role.kubernetes.io/control-plane:NoSchedule
```
So it is likely no pods will be assigned to control-plane.


<br><br><br>

## CPU Request
Take a look at below request.yaml which looks almost like deployment.yaml.<br>
Over here, we will introduce cpu request like below.

```txt
STEP1 : add yaml file
nano request.yaml

STEP2 : review yaml file
cat request.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-requests
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-world-requests
  template:
    metadata:
      labels:
        app: hello-world-requests
    spec:
      containers:
      - name: hello-world
        image: psk8s.azurecr.io/hello-app:1.0
        resources:
          requests:
            cpu: "1"
        ports:
        - containerPort: 8080

STEP3 : launch deployment
kubectl create -f request.yaml
deployment.apps/hello-world-requests created
```
<br>

### CPU Request - 1st Scenario : 3 replicas, CPU Request = 1 <br>
Take a look of STEP 4 and STEP 5 for replicas = 3, CPU: "1" at request.yaml
```txt
STEP4 : review newly created pods
kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-world-requests-59776bd988-ktm5t   1/1     Running   0          29s
hello-world-requests-59776bd988-lgj6v   1/1     Running   0          29s
hello-world-requests-59776bd988-trb8l   1/1     Running   0          29s
```
Also take a look at the output of kubectl describe command below
```txt
STEP 5: Review Nodes
kubectl describe node docker-desktop
  ...
  Non-terminated Pods:          (12 in total)
    ...see the output table with 12 rows below
  Allocated resources:
    (Total limits may be over 100 percent, i.e., overcommitted.)
    ...see the output table with 3 rows below right after
  ...
```

Non-terminated Pods section (from kubectl describe node docker-desktop) 
|Namespace    |Name                                    |CPU Requests  |CPU Limits  |Memory Requests  |Memory Limits  |Age
|---------    |----                                    |------------  |----------  |---------------  |-------------  |---
| default     |hello-world-requests-59776bd988-ktm5t   |1 (12%)       |0 (0%)      |0 (0%)           |0 (0%)         |99s|
| default     |hello-world-requests-59776bd988-lgj6v   |1 (12%)       |0 (0%)      |0 (0%)           |0 (0%)         |99s|
| default     |hello-world-requests-59776bd988-trb8l   |1 (12%)       |0 (0%)      |0 (0%)           |0 (0%)         |99s|
| kube-system |coredns-5dd5756b68-gl66r                |100m (1%)     |0 (0%)      |70Mi (1%)        |170Mi (4%)     |26h|
| kube-system |coredns-5dd5756b68-p75fr                |100m (1%)     |0 (0%)      |70Mi (1%)        |170Mi (4%)     |26h|
| kube-system |etcd-docker-desktop                     |100m (1%)     |0 (0%)      |100Mi (2%)       |0 (0%)         |26h|
| kube-system |kube-apiserver-docker-desktop           |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |26h|
| kube-system |kube-controller-manager-docker-desktop  |200m (2%)     |0 (0%)      |0 (0%)           |0 (0%)         |26h|
| kube-system |kube-proxy-4cr8h                        |0 (0%)        |0 (0%)      |0 (0%)           |0 (0%)         |26h|
| kube-system |kube-scheduler-docker-desktop           |100m (1%)     |0 (0%)      |0 (0%)           |0 (0%)         |26h|
| kube-system |storage-provisioner                     |0 (0%)        |0 (0%)      |0 (0%)           |0 (0%)         |26h|
| kube-system |vpnkit-controller                       |0 (0%)        |0 (0%)      |0 (0%)           |0 (0%)         |26h|
<br>

Allocated resources section (from ubectl describe node docker-desktop)

| Resource           |Requests     |Limits     |
| --------           |--------     |------     |
| cpu                |3850m (48%)  |0 (0%)     |
| memory             |240Mi (6%)   |340Mi (9%) |
| ephemeral-storage  |0 (0%)       |0 (0%)     |

<br>
Let's now try to increase the replicas inside the yaml file, while maintaining cpu request 1.
<br>

### CPU Request - 2nd Scenario : 10 replicas, CPU Request = 1 <br>
Herewith STEP4 & STEP 5 for replicas=10 and CPU: "1" at request.yaml --> 7 running, 3 pending
```txt
STEP4 : review newly created pods
kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-world-requests-59776bd988-9bfrs   1/1     Running   0          9m23s
hello-world-requests-59776bd988-cd7rs   0/1     Pending   0          9m23s
hello-world-requests-59776bd988-gnznp   1/1     Running   0          9m23s
hello-world-requests-59776bd988-h8nnh   1/1     Running   0          9m23s
hello-world-requests-59776bd988-ktm5t   1/1     Running   0          44m
hello-world-requests-59776bd988-lgj6v   1/1     Running   0          44m
hello-world-requests-59776bd988-trb8l   1/1     Running   0          44m
hello-world-requests-59776bd988-wfnd5   0/1     Pending   0          9m23s
hello-world-requests-59776bd988-z5pqx   1/1     Running   0          9m23s
hello-world-requests-59776bd988-znh89   0/1     Pending   0          9m23s
```
Also take a look at the output of kubectl describe command below
```txt
STEP 5: Review Nodes
kubectl describe node docker-desktop
  ...
  Non-terminated Pods:          (16 in total)
    ...see the output table with 16 rows below
  Allocated resources:
    (Total limits may be over 100 percent, i.e., overcommitted.)
    ...see the output table with 3 rows below right after
  ...
```
Non-terminated Pods section (from kubectl describe node docker-desktop) 
| Namespace    |Name                                      |CPU Requests  |CPU Limits  |Memory Requests  |Memory Limits  |Age
| ---------    |----                                      |------------  |----------  |---------------  |-------------  |---
| default      |hello-world-requests-59776bd988-9bfrs     |1 (12%)       |0 (0%)      |0 (0%)           |0 (0%)         |94s
| default      |hello-world-requests-59776bd988-gnznp     |1 (12%)       |0 (0%)      |0 (0%)           |0 (0%)         |94s
| default      |hello-world-requests-59776bd988-h8nnh     |1 (12%)       |0 (0%)      |0 (0%)           |0 (0%)         |94s
| default      |hello-world-requests-59776bd988-ktm5t     |1 (12%)       |0 (0%)      |0 (0%)           |0 (0%)         |36m
| default      |hello-world-requests-59776bd988-lgj6v     |1 (12%)       |0 (0%)      |0 (0%)           |0 (0%)         |36m
| default      |hello-world-requests-59776bd988-trb8l     |1 (12%)       |0 (0%)      |0 (0%)           |0 (0%)         |36m
| default      |hello-world-requests-59776bd988-z5pqx     |1 (12%)       |0 (0%)      |0 (0%)           |0 (0%)         |94s
| kube-system  |coredns-5dd5756b68-gl66r                  |100m (1%)     |0 (0%)      |70Mi (1%)        |170Mi (4%)     |27h
| kube-system  |coredns-5dd5756b68-p75fr                  |100m (1%)     |0 (0%)      |70Mi (1%)        |170Mi (4%)     |27h
| kube-system  |etcd-docker-desktop                       |100m (1%)     |0 (0%)      |100Mi (2%)       |0 (0%)         |27h
| kube-system  |kube-apiserver-docker-desktop             |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |27h
| kube-system  |kube-controller-manager-docker-desktop    |200m (2%)     |0 (0%)      |0 (0%)           |0 (0%)         |27h
| kube-system  |kube-proxy-4cr8h                          |0 (0%)        |0 (0%)      |0 (0%)           |0 (0%)         |27h
| kube-system  |kube-scheduler-docker-desktop             |100m (1%)     |0 (0%)      |0 (0%)           |0 (0%)         |27h
| kube-system  |storage-provisioner                       |0 (0%)        |0 (0%)      |0 (0%)           |0 (0%)         |27h
| kube-system  |vpnkit-controller                         |0 (0%)        |0 (0%)      |0 (0%)           |0 (0%)         |27h
<br>

Allocated resources section (from ubectl describe node docker-desktop)
|  Resource           |Requests     |Limits
|  --------           |--------     |------
|  cpu                |7850m (98%)  | 0 (0%)
|  memory             |240Mi (6%)   |340Mi (9%)
|  ephemeral-storage  |0 (0%)       |0 (0%)
<br>

### CPU Request - 3rd Scenario : 7 replicas, CPU Request = 1 <br>
Herewith STEP4 & STEP 5 for replicas=7 and CPU: "1" at request.yaml --> 7 running, 0 pending
<br>
```txt
STEP4 : review newly created pods
kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-world-requests-59776bd988-9bfrs   1/1     Running   0          24m
hello-world-requests-59776bd988-gnznp   1/1     Running   0          24m
hello-world-requests-59776bd988-h8nnh   1/1     Running   0          24m
hello-world-requests-59776bd988-ktm5t   1/1     Running   0          59m
hello-world-requests-59776bd988-lgj6v   1/1     Running   0          59m
hello-world-requests-59776bd988-trb8l   1/1     Running   0          59m
hello-world-requests-59776bd988-z5pqx   1/1     Running   0          24m
```
and STEP 5 here would be similar to STEP 5 in scenario 2. 

### CPU Request - 4rd Scenario : 10 replicas, CPU Request = 0.25 <br>
Herewith STEP4 & STEP 5 for replicas=10 and CPU: "0.25" at request.yaml --> 10 running, 0 pending
```txt
STEP4 : review newly created pods
kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
hello-world-requests-d9955fdd9-7sfmc   1/1     Running   0          6m5s
hello-world-requests-d9955fdd9-8gt24   1/1     Running   0          6m6s
hello-world-requests-d9955fdd9-ftzk9   1/1     Running   0          6m19s
hello-world-requests-d9955fdd9-h6p2j   1/1     Running   0          6m5s
hello-world-requests-d9955fdd9-n7d4f   1/1     Running   0          6m21s
hello-world-requests-d9955fdd9-sjrs7   1/1     Running   0          6m20s
hello-world-requests-d9955fdd9-t5ncb   1/1     Running   0          6m21s
hello-world-requests-d9955fdd9-v2hwd   1/1     Running   0          6m1s
hello-world-requests-d9955fdd9-wcxxg   1/1     Running   0          6m3s
hello-world-requests-d9955fdd9-xq7xh   1/1     Running   0          6m21s
```
and finally take a look at the output of kubectl describe command below
```txt
STEP 5: Review Nodes
kubectl describe node docker-desktop
  ...
  Non-terminated Pods:          (19 in total)
    ...see the output table with 19 rows below
  Allocated resources:
    (Total limits may be over 100 percent, i.e., overcommitted.)
    ...see the output table with 3 rows below right after
  ...
```
Non-terminated Pods section (from kubectl describe node docker-desktop) 

|  Namespace    |Name                                      |CPU Requests  |CPU Limits  |Memory Requests  |Memory Limits  |Age
|  ---------    |----                                      |------------  |----------  |---------------  |-------------  |---
|  default      |hello-world-requests-d9955fdd9-7sfmc      |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |8m37s
|  default      |hello-world-requests-d9955fdd9-8gt24      |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |8m38s
|  default      |hello-world-requests-d9955fdd9-ftzk9      |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |8m51s
|  default      |hello-world-requests-d9955fdd9-h6p2j      |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |8m37s
|  default      |hello-world-requests-d9955fdd9-n7d4f      |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |8m53s
|  default      |hello-world-requests-d9955fdd9-sjrs7      |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |8m52s
|  default      |hello-world-requests-d9955fdd9-t5ncb      |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |8m53s
|  default      |hello-world-requests-d9955fdd9-v2hwd      |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |8m33s
|  default      |hello-world-requests-d9955fdd9-wcxxg      |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |8m35s
|  default      |hello-world-requests-d9955fdd9-xq7xh      |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |8m53s
|  kube-system  |coredns-5dd5756b68-gl66r                  |100m (1%)     |0 (0%)      |70Mi (1%)        |170Mi (4%)     |27h
|  kube-system  |coredns-5dd5756b68-p75fr                  |100m (1%)     |0 (0%)      |70Mi (1%)        |170Mi (4%)     |27h
|  kube-system  |etcd-docker-desktop                       |100m (1%)     |0 (0%)      |100Mi (2%)       |0 (0%)         |27h
|  kube-system  |kube-apiserver-docker-desktop             |250m (3%)     |0 (0%)      |0 (0%)           |0 (0%)         |27h
|  kube-system  |kube-controller-manager-docker-desktop    |200m (2%)     |0 (0%)      |0 (0%)           |0 (0%)         |27h
|  kube-system  |kube-proxy-4cr8h                          |0 (0%)        |0 (0%)      |0 (0%)           |0 (0%)         |27h
|  kube-system  |kube-scheduler-docker-desktop             |100m (1%)     |0 (0%)      |0 (0%)           |0 (0%)         |27h
|  kube-system  |storage-provisioner                       |0 (0%)        |0 (0%)      |0 (0%)           |0 (0%)         |27h
|  kube-system  |vpnkit-controller                         |0 (0%)        |0 (0%)      |0 (0%)           |0 (0%)         |27h
<br>

Allocated resources section (from ubectl describe node docker-desktop)
|  Resource           |Requests     |Limits
|  --------           |--------     |------
|  cpu                |3350m (41%)  |0 (0%)
|  memory             |240Mi (6%)   |340Mi (9%)
|  ephemeral-storage  |0 (0%)       |0 (0%)
<br><br>

![](/images/05-image11.png)
Figure 11: All the 4 scenarios put together in term of CPU Request, Number of Replicas

## NODE SELECTOR
Let's see below situation. 

```txt

# INITIALLY WE MISTAKENLY SELECT A WRONG nodeSelector
cat deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
  namespace: default
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
        image: psk8s.azurecr.io/hello-app:2.0
        ports:
        - containerPort: 8080
      nodeSelector:
        node: my-fav-node2

# WHEN WE DEPLOY IT AND CHECKING THE PODS --> PODS PENDING
kubectl apply -f deployment.yaml
  deployment.apps/hello-world created

kubectl get pods
  NAME                           READY   STATUS    RESTARTS   AGE
  hello-world-5c6b94bf6b-cd5sm   0/1     Pending   0          24s
  hello-world-5c6b94bf6b-krf9l   0/1     Pending   0          24s
  hello-world-5c6b94bf6b-tktrv   0/1     Pending   0          24s

# AFTER WE EDIT THE YAML FILE CORRECTLY
nano deployment.yaml
cat deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: hello-world
    namespace: default
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
          image: psk8s.azurecr.io/hello-app:2.0
          ports:
          - containerPort: 8080
        nodeSelector:
          node: my-fav-node

# NODE SELECTOR IS NOT POINTING TO THE RIGHT LABEL --> PODS RUNNING
kubectl apply -f deployment.yaml
  deployment.apps/hello-world configured
kubectl get pods
  NAME                           READY   STATUS    RESTARTS   AGE
  hello-world-677cccb5d4-ff5n6   1/1     Running   0          17s
  hello-world-677cccb5d4-kr7x2   1/1     Running   0          6s
  hello-world-677cccb5d4-qnrd5   1/1     Running   0          11s
```

Node selector is used in Kubernetes to specify the nodes on which a particular pod should be scheduled. It allows users to control the placement of pods on specific nodes based on various criteria such as hardware capabilities, network locality, or any other custom labels assigned to nodes.

For example if one bank is about to migrate their machine learning models to the cloud, some models are more expensive than the others. These stronger machines can be labeled to land on specific nodes.

## AFFINITY FOR POD PLACEMENT
Two complex checks/boolean logic predicates, used in Affinity and Anti-Affinity scheme are
- requiredDuringSchedulingIgnoreDuringExecution and
- preferredDuringSchedulingIgnoreDuringExecution

Let's use deployment.yaml with few changes below
```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
      - name: hello-world
        image: psk8s.azurecr.io/hello-app:2.0
        ports:
        - containerPort: 8080
```
We don't have multiple workers here, so could not show any round robin in node column
```txt
evops@msi:~$ kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE    IP           NODE             NOMINATED NODE   READINESS GATES
caching-6f7b7dcccf-4wmhb       1/1     Running   0          108s   10.1.2.252   docker-desktop   <none>           <none>
hello-world-645c45876d-64fb7   1/1     Running   0          18m    10.1.2.248   docker-desktop   <none>           <none>
hello-world-645c45876d-l5g7c   1/1     Running   0          18m    10.1.2.250   docker-desktop   <none>           <none>
hello-world-645c45876d-mj8tj   1/1     Running   0          18m    10.1.2.249   docker-desktop   <none>           <none>

```

then also take a look at below caching.yaml

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  name: caching
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cache
  template:
    metadata:
      labels:
        app: cache
    spec:
      containers:
      - name: hello-world-cache
        image: psk8s.azurecr.io/hello-app:1.0
        ports:
        - containerPort: 8080
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            -  matchExpressions:
               - key: node
                 operator: In
                 values:
                 - my-fav-node
```

Says we changed my-fav-node into my-fav-node2, let's how affinity works.

```txt
#STEP 1 : WE INTENTIONALLY PUT A WRONG LABEL IN AFFINITY ATTRIBUTE
cat caching.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: caching
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cache
  template:
    metadata:
      labels:
        app: cache
    spec:
      containers:
      - name: hello-world-cache
        image: psk8s.azurecr.io/hello-app:1.0
        ports:
        - containerPort: 8080
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            -  matchExpressions:
               - key: node
                 operator: In
                 values:
                 - my-fav-node2

#STEP 2: WE COULD CREATE IT BUT..
kubectl create -f caching.yaml
deployment.apps/caching created

#STEP 3: THE POD WILL BE PENDING 
kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
caching-6589594449-ss5fq       0/1     Pending   0          6s
hello-world-645c45876d-64fb7   1/1     Running   0          34m
hello-world-645c45876d-l5g7c   1/1     Running   0          34m
hello-world-645c45876d-mj8tj   1/1     Running   0          34m

#STEP 4: CHANGING BACK THE LABEL, RERUN AND SEE POD IS NOW RUNNING 
nano caching.yaml
cat caching.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: caching
    namespace: default
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: cache
    template:
      metadata:
        labels:
          app: cache
      spec:
        containers:
        - name: hello-world-cache
          image: psk8s.azurecr.io/hello-app:1.0
          ports:
          - containerPort: 8080
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              -  matchExpressions:
                - key: node
                  operator: In
                  values:
                  - my-fav-node

kubectl apply -f caching.yaml
  deployment.apps/caching configured

kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
caching-6f7b7dcccf-fjgc8       1/1     Running   0          10s
hello-world-645c45876d-64fb7   1/1     Running   0          34m
hello-world-645c45876d-l5g7c   1/1     Running   0          34m
hello-world-645c45876d-mj8tj   1/1     Running   0          34m
```
### TOPOLOGY
Take a look the other example says with 
- web-cache-affinity.yaml
- web-cache-no-need.yaml
- cache2.yaml

as follow

```txt
cat web-cache-affinity.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: web-cache-affinity
    namespace: default
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: web-cache-affinity
    template:
      metadata:
        labels:
          app: web-cache-affinity
      spec:
        containers:
        - name: hello-world-web
          image: psk8s.azurecr.io/hello-app:2.0
          ports:
          - containerPort: 8080

cat web-cache-no-need.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: web-cache-no-need
    namespace: default
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: web-cache-no-need
    template:
      metadata:
        labels:
          app: web-cache-no-need
      spec:
        containers:
        - name: hello-world-web
          image: psk8s.azurecr.io/hello-app:2.0
          ports:
          - containerPort: 8080

cat cache2.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: caching
    namespace: default
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: cache
    template:
      metadata:
        labels:
          app: cache
      spec:
        containers:
        - name: hello-world-cache
          image: psk8s.azurecr.io/hello-app:1.0
          ports:
          - containerPort: 8080
        affinity:
          podAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - web-cache-no-need
              topologyKey: "kubernetes.io/hostname"
```

The topology line is required for podAffinity because it specifies the key used to match the topology domain of the node where the pods are scheduled. This ensures that the pods are scheduled on nodes that have the specified topology domain.

On the other hand, nodeAffinity does not require the topology line because it does not involve matching the topology domain of the node. NodeAffinity is used to specify rules for scheduling pods based on node labels or other node attributes, without considering the topology domain.

The last two line of cache2.yaml could be either or both
- web-cache-no-need
- web-cache-affinity


## ANTI-AFFINITY
As the name suggest, everything will remain the same but one line will be different, see anti.yaml below

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-server
  template:
    metadata:
      labels:
        app: web-server
    spec:
      containers:
      - name: hello-world-cache
        image: psk8s.azurecr.io/hello-app:1.0
        ports:
        - containerPort: 8080
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-server
            topologyKey: "kubernetes.io/hostname"
```
Look at below case when we want to scale anti-affinity. <br>
By the way, anti-affinitity does not need previous existing pods like affinity example above.

```txt
#FIRST, TAKE A LOOK ON A YAML FILE FOR ANTI-AFFINITY WHICH ALLOW SCALING
cat anti2.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: web-server
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: web-server
    template:
      metadata:
        labels:
          app: web-server
      spec:
        containers:
        - name: hello-world-cache
          image: psk8s.azurecr.io/hello-app:1.0
          ports:
          - containerPort: 8080
        affinity:
          podAntiAffinity:
            preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                  - key: app
                    operator: In
                    values:
                    - web-server
                topologyKey: "kubernetes.io/hostname"

#SECONDLY, WE CREATE THE DEPLOYMENT
kubectl create -f anti2.yaml
  deployment.apps/web-server created

#THIRDLY, WE CAN SEE ALLOW PODS ARE RUNNING
kubectl scale deployment web-server --replicas=4
  deployment.apps/web-server scaled
  devops@msi:~$ kubectl get pods
  NAME                          READY   STATUS    RESTARTS   AGE
  web-server-6bfd64d5bf-4zq9z   1/1     Running   0          87s
  web-server-6bfd64d5bf-jf4bm   1/1     Running   0          30s
  web-server-6bfd64d5bf-q6jhh   1/1     Running   0          30s
  web-server-6bfd64d5bf-wwzjl   1/1     Running   0          30s

#NEXT, LET'S DELETE THIS DEPLOYMENT AND RUN THE OLDER ANTI-AFFINITY
kubectl delete -f anti2.yaml
  deployment.apps "web-server" deleted

cat anti.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: web-server
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: web-server
    template:
      metadata:
        labels:
          app: web-server
      spec:
        containers:
        - name: hello-world-cache
          image: psk8s.azurecr.io/hello-app:1.0
          ports:
          - containerPort: 8080
        affinity:
          podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - web-server
              topologyKey: "kubernetes.io/hostname"

# NEXT WE CREATE THE DEPLOYMENT WITH THE SAME NAME
kubectl scale deployment web-server --replicas=4
  deployment.apps/web-server scaled

# INTERESTINGLY, IT WON'T WORK. IT SUGGESTS MODIFICATION ABOVE TO SCALE
kubectl get pods
  NAME                          READY   STATUS    RESTARTS   AGE
  web-server-75bd558b87-8sptp   0/1     Pending   0          15s
  web-server-75bd558b87-mtp4w   0/1     Pending   0          15s
  web-server-75bd558b87-vh2xv   1/1     Running   0          23s
  web-server-75bd558b87-whs5x   0/1     Pending   0          15s
```

## TAINT AND TOLERATION
### TAINT
In Kubernetes (K8S), taints are a way to mark a node with a specific attribute that affects which pods can be scheduled on that node. Taints are used to indicate that a node has a specific limitation or requirement, such as being reserved for certain types of workloads or having specific hardware capabilities.

When a node is tainted, it means that by default, no pods will be scheduled on that node unless the pod tolerates the taint. Tolerations are set on pods to indicate that they can be scheduled on nodes with specific taints.

Taints and tolerations are useful for scenarios where certain nodes need to be dedicated to specific workloads or have specific requirements. They help ensure that pods are scheduled on appropriate nodes based on their requirements and constraints.
Taints also available in K3S.


```txt
kubectl get nodes
  NAME     STATUS   ROLES                  AGE     VERSION
  master   Ready    control-plane,master   6d10h   v1.28.5+k3s1
  worker   Ready    <none>                 6d10h   v1.28.5+k3s1

kubectl taint node worker key=MyTaint:NoSchedule
  node/worker tainted

kubectl taint node worker key=MyTaint:NoSchedule-
  node/worker untainted
```

Same thing we can taint and untaint docker-desktop node like below

```txt
kubectl taint node docker-desktop key=MyTaint:NoSchedule
  node/docker-desktop tainted

kubectl create -f deployment.yaml
  deployment.apps/hello-world created

kubectl get pods
  NAME                           READY   STATUS    RESTARTS   AGE
  hello-world-756bb58cbd-m87jb   0/1     Pending   0          18s
  hello-world-756bb58cbd-ms547   0/1     Pending   0          18s
  hello-world-756bb58cbd-w9zws   0/1     Pending   0          18s

kubectl taint node docker-desktop key=MyTaint:NoSchedule-
  node/docker-desktop untainted

kubectl get pods
  NAME                           READY   STATUS    RESTARTS   AGE
  hello-world-756bb58cbd-m87jb   1/1     Running   0          60s
  hello-world-756bb58cbd-ms547   1/1     Running   0          60s
  hello-world-756bb58cbd-w9zws   1/1     Running   0          60s
```

According to Merriam-Webster dictionary, Taint is to to contaminate morally. In this case taint discourage the placement of pods in the node until and unless we tolerate it.

### TOLERATION
Let's recap taint process above by using running toleration.yaml below & K3S

```txt
#STEP 1: create and review a new yaml file name toleration.yaml as follow
cat toleration.yaml
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

#STEP 2: Simply create a deployment where master and worker node not tainted
kubectl create -f toleration.yaml 
  deployment.apps/hello-world created

#STEP 3: As you can see master node also part of deployment (ideally not)
kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
hello-world-5f64785679-xcsbn   1/1     Running   0          7s    10.42.0.36   master   <none>           <none>
hello-world-5f64785679-krmzb   1/1     Running   0          7s    10.42.1.34   worker   <none>           <none>
hello-world-5f64785679-ctg78   1/1     Running   0          7s    10.42.1.35   worker   <none>           <none>

#STEP 4: Let's delete this deployment, taint the master and re-deploy again
kubectl delete -f toleration.yaml
  deployment.apps "hello-world" deleted
kubectl taint node master key=MyTaint:NoSchedule
  node/master tainted
kubectl create -f toleration.yaml
  deployment.apps/hello-world created
kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
hello-world-5f64785679-f99fd   1/1     Running   0          4s    10.42.1.36   worker   <none>           <none>
hello-world-5f64785679-74rrs   1/1     Running   0          4s    10.42.1.38   worker   <none>           <none>
hello-world-5f64785679-5n66h   1/1     Running   0          4s    10.42.1.37   worker   <none>           <none>
```
Now we will add few lines to toleration.yaml

```
cat toleration.yaml
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
        tolerations:
        - key: "key"
          operator: "Equal"
          value: "MyTaint"
          effect: "NoSchedule"

kubectl create -f toleration.yaml
  deployment.apps/hello-world created

kubectl get pods -o wide
  NAME                           READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
  hello-world-75f87cb94f-q6cps   1/1     Running   0          30s   10.42.0.40   master   <none>           <none>
  hello-world-75f87cb94f-rh26h   1/1     Running   0          23s   10.42.1.48   worker   <none>           <none>
  hello-world-75f87cb94f-9cpkv   1/1     Running   0          20s   10.42.0.41   master   <none>           <none>
```

Master node is actually still tainted. In Kubernetes, a node can still deploy a pod even if it is tainted, as long as the pod has a toleration specified for that particular taint. Taints are used to repel pods from being scheduled on specific nodes, but tolerations allow pods to be scheduled on tainted nodes. By specifying a toleration in the pod's configuration, it can be deployed on a tainted node, overriding the taint effect. <br>
