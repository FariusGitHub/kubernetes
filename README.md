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
## Watchs and Logs
Let's see another feature from kubectl called --watch. If we use K9S this feature already there. Let's see how the pod was created and deleted while we can see the progress like below.

![](/images/05-image03.gif)
Figure 2: Comparable Screen from K9S and Traditional terminal with kubectl get pods --watch -v 6  
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
Figure 3: Comparable Screen from K9S and Traditional terminal with kubectl get deployment --watch -v 6  
<br>

For Logs, the `--watch` flag is not available for logs in Kubernetes. The `--watch` flag is used to continuously monitor changes in resources like pods, services, or deployments. However, for logs, you can use the `kubectl logs` command to retrieve the logs of a specific pod. By default, it will show the most recent logs, but you can use the `-f` flag to follow the logs in real-time. 

![](/images/05-image05.gif)
Figure 4: Logs command in kubectl does not have --watch, it has to logs very specific pod by its name  
<br>
