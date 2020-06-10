
## Core concepts

### Practice Test - PODs

`kubectl get pods`

`kubectl run --generator=run-pod/v1 nginx --image=nginx`

`kubectl describe pod newpods-c92cg`

`kubectl get pods -o wide`

`kubectl delete pod webapp`

cat > pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: redis

spec:
  containers:
  - name: redis
    image: redis123
```

`kubectl create -f pod.yml`

`kubectl edit pod redis`
`kubectl apply -f pod.yml`
`kubectl delete pod redis`, next udate image in pod.yml and then `kubectl create -f pod.yml`


### ReplicaSet

`kubectl craete -f replicaset.yml`
`kubectl replace -f replicaset.yml`
`kubectl scale --replicas=6 -f replicaset.yml`
`kubectl scale --replicas=6 replicaset name`

`kubectl get replicaset`

`kubectl describe replicaset name`

`kubectl describe pod new-replica-set-zfqdx`

`kubectl delete pod new-replica-set-zfqdx`

cat replicaset-definition-2.yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

replicaset api version: `apps/v1`

labels should be correct in replicaset

```
kubectl get replicasets
kubectl delete replicasets replicaset-1
kubectl delete replicasets replicaset-2
```

```
kubectl edit replicasets new-replica-set
kubectl delete pod new-replica-set-7cz8q new-replica-set-87f86 new-replica-set-qjx2j new-replica-set-tgktv
kubectl get pods
```

`kubectl scale --replicas=5 replicaset new-replica-set`

### Deployments

```
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml > pod.yml
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml
```

cat deployment-definition-1.yaml
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - name: busybox-container
        image: busybox
        command:
        - sh
        - "-c"
        - echo Hello Kubernetes! && sleep 3600
```

```
kubectl create -f deployment.yml
kubectl get deployments
kubectl get replicaset
kubectl get pods
kubect get all

kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --dry-run -o yaml > deployment.yml
vi deployment.yml
kubectl create -f deployment.yml
```

### Namespaces

`kubectl create namespace dev`

cat > namespace.yml
```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```
`kubectl create -f namespace.yml`

```
kubectl get pods
kubectl get pods --namespace=dev
kubectl config set-context $(kubectl config current-context) --namespace=dev
kubectl get pods
kubectl get pods --namespace=default
kubectl get pods --all-namespaces
```

cat resource-quota.yaml
```
---
apiVersion: v1
kind: ResouceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

`kubectl create -f resource-quota.yaml`

`kubectl get namespaces`

`kubectl get pods --namespace=research`

cat redis.yml
```
apiVersion: v1
kind: Pod
metadata:
  namespace: finance
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis
    name: redis
```
`kubectl create -f redis.yml`

db-service - Within the same namespace services can call with the name of the service
db-service.dev.svc.cluster.local - Within the same namespace services cannot be call with the name of the service alone

### Services

cat service-definition-1.yml
```
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30080
  selector:
    name: simple-webapp
```

`kubectl create -f service-definition-1.yml`

`kubectl get services`
crul http://192.168.2.1:3008

`kubectl get service`
`kubectl describe service kubernetes`

`kubectl get deployment`
`kubectl describe deployment simple-webapp-deployment`

cat clusterip-service.yml
```
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: myapp
    type: front-end
```

`kubectl create -f clusterip-service.yml`


```
kubectl expose pod redis --port=6379 --name redis-service --dry-run -o yaml
kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml
```

We can not specify nodeport
```
kubectl expose pod nginx --port=80 --name nginx-service --dry-run -o yaml
```

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run -o yaml`

### Imperative commands

```
kubectl run --generator=run-pod/v1 nginx --image=nginx
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml

kubectl run --generator=deployment/v1beta1 nginx --image=nginx
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run -o yaml

kubectl create deployment --image=nginx nginx
kubectl create deployment --image=nginx nginx --dry-run -o yaml

kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml

kubectl expose pod redis --port=6379 --name redis-service --dry-run -o yaml
```

The below one will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)
`kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml`

The below one will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod
`kubectl expose pod nginx --port=80 --name nginx-service --dry-run -o yaml`

The below one will not use the pods labels as selectors
`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run -o yaml`

`kubectl run --generator=run-pod/v1 nginx-pod --image=nginx:alpine`

`kubectl run --generator=run-pod/v1 redis --image=redis:alpine -l tier=db`

`kubectl expose pod redis --port=6379 --name redis-service`

`kubectl run --generator=deployment/v1beta1 webapp --image=kodekloud/webapp-color --replicas=3`
```
kubectl create deployment webapp2 --image=kodekloud/webapp-color
kubectl scale --replicase=3 deployment webapp2
```

`kubectl expose deployment webapp --type=NodePort --port=8080 --name=webapp-service --dry-run -o yaml > webapp-service.yaml`

edit webapp-service.yaml and add nodePort
`kubectl create -f webapp-service.yaml`


## Scheduling

### Manual scheduling

cat > nginx.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -  image: nginx
     name: nginx
```

`kubectl create -f nginx.yaml`

`kubectl get pod`

scheduler is not there so pod is in pending state
`kubectl get pods --namespace=kube-system`

manually schedule pod on node01
```
vi nginx.yml
kubectl create -f nginx.yaml
kubectl get pods
```

vi nginx.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
      - containerPort: 8080
  nodeName: node02
```

```
kubectl delete pod nginx
kubectl create -f nginx.yml
kubectl get pods
```

Run pod on master
```
vi nginx.yml
kubectl delete pod nginx
kubectl create -f nginx.yml
kubectl get pods
```

cat > already-schedule.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
      - containerPort: 8080
```

`kubectl create -f already-schedule.yml`

cat > pod-binidng.yml
```
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

curl --header "Content-Type:application/json" --request POST --data '{"apiVersion":"v1", "kinda": "Binding", ...}'' http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding

### Labels and Selectors

```
kubectl get pods --selector=tier,env,bu
kubectl get pods -l env=dev
kubectl get pods --selector env=dev
kubectl get pods --selector bu=finance
kubectl get all --selector env=prod
kubectl get pod --selector env=prod,bu=finance,tier=frontend
```

selector label name is not available in pod labels list

### Taints and Tolerations

`kubectl taint node node-name key=value:taint-effect`

taint effect - NoSchedule, PreferNoSchedule, NoExecute

`kubectl taint nodes node1 app=blue:NoSchedule`

```
kubectl get nodes
kubectl describe node node01 | grep Taint
kubectl taint node node01 spray=mortein:NoSchedule
kubectl run --generator=run-pod/v1 mosquito --image=nginx --dry-run -o yaml > pod.yml
kubectl create -f pod.yml
kubectl get pods
kubectl describe pod mosquito
```

Having toleration issue on node1

`kubectl run --generator=run-pod/v1 bee --image=nginx --dry-run -o yaml > bee-pod.yml`


Add toleration content
vi bee-pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: bee
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
  tolerations:
  - key: "spray"
    operator: "Equal"
    value: "mortein"
    effect: "NoSchedule"
```

```
kubectl create -f bee-pod.yml
kubectl get pods
kubectl describe pod bee
```

```
kubectl describe nodes  master | grep -i taint
kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-
kubectl get pods
kubectl get pods -o wide
```

### Node Selectors

`kubectl label nodes node-1 size=Large`

cat > pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - image: data-processor
    name: data-processor
  nodeSelector:
    size: Large
```

`kubectl create -f pod.yml`

### Node Affinity

cat > pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - image: data-processor
    name: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoreDuringExecution:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
```

```
          - key: size
            operator: NotIn
            values:
            - Small
```

```
          - key: size
            operator: Exists
```

Available:
requiredDuringSchedulingIgnoreDuringExecution
preferredDuringSchedulingIgnoreDuringExecution

Planned:
requiredDuringSchedulingRequiredDuringExecution

```
kubectl describe node node01
kubectl label node node01 color=blue
kubectl create deployment blue --image=nginx
```

Update replicas
```
kubectl edit deployments blue
kubectl get pods -o wide
```

cat /var/answers/blue-deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 6
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
```

```
kubectl create -f blue-deployment.yaml
kubectl get pods -o wide
```

cat > red-deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
```

```
kubectl create -f red-deployment.yaml
kubectl get pods -o wide
```

### Resource limit

cat > pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - image: data-processor
    name: data-processor
  resouces:
    requests:
      memory: "10Mi"
      cpu: "1"
    limits:
      memory: "20Mi"
      cpu:"2"
```

```
kubectl describe pod rabbit
kubectl delete pod rabbit
```

`kubectl get pod elephant -o yaml > elephant.yml`

Increase memory limit
```
vi elephant.yml
kubectl delete pod elephant
kubectl create -f elephant.yml
kubectl get pod
```

### DaemonSets

DemonSet is same as ReplicaSet. Pod will be available in each node of the cluster using DemonSet. Ex: kube-proxy

```
kubectl get daemonset
kubectl describe daemonset name
kubectl get daemonsets --all-namespaces
kubectl get pods -o wide --all-namespaces
kubectl describe daemonset kube-proxy --namespace=kube-system
kubectl describe daemonset weave-net --namespace=kube-system

```

cat > daemonset.yaml
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: k8s.gcr.io/fluentd-elasticsearch:1.20
```

```
kubectl create -f daemonset.yaml
kubectl get daemonset --namespace=kube-system
```

### Static pods

kubelet can alone creates pods if we keep pod definition files under `/etc/kubernetes/manifests`

Only pods can be created like this.

To see container `docker ps`

We can see static pods from master but cannot delete/edit from master. Can change only from manifests files

Use case: manage control-plane components like apiserver,etcd,controller-manager

Kube-scheduler has no idea about static pods and daemonset pods.

```
kubectl get pods --all-namespaces
kubectl get pods --all-namespaces -o wide
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -i image
```

cat > /etc/kubernetes/manifests/pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: static-busybox
spec:
  containers:
  - image: busybox
    name: static-busybox
    command:
    - "sh"
    - "-c"
    - "sleep 100"
```    

```
kubectl get pods --all-namespaces -o wide
kubectl get nodes --all-namespaces -o wide

ssh nodeip
cat /var/lib/kubelet/config.yaml
```

### Multiple schedulers

```
kubectl get pods --all-namespaces
kubectl describe pod kube-scheduler-master --namespace kube-system | grep -i image
```

cat my-scheduler.yaml
```
apiVersion: v1
kind: Pod
metadata:
  annotations:
    scheduler.alpha.kubernetes.io/critical-pod: ""
  creationTimestamp: null
  labels:
    component: my-scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false
    - --port=10282
    - --scheduler-name=my-scheduler
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10282
        scheme: HTTP
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
status: {}
```

cat > /etc/kubernetes/manifests/pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
  schedulerName: my-scheduler
```

```
kubectl get events
kubectl logs my-scheduler --namespace=kube-system
```

## Logging & Monitoring

### Monintoring

For monitoring - metrics-server - In memory not store in disk so no history

cAdvisor is a kubelet component colllects metrics from nodes/pods and exposes metrics to metrics-server

`minikube addons enable metrics-server`

```
git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
cd kubernetes-metrics-server
kubectl create -f .
kubectl top node
kubectl top pod
```

### Managing Application logs

cat > event-simulator.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
  - image: event-simulator
    name: kodekloud/event-simulator
```    

```
kubectl create -f event-simulator.yml
kubectl logs -f event-simulator-pod
kubectl logs -f event-simulator-pod event-simulator
```

## Application Lifecycle Management

### Rolling Updates and Rollbacks

Recreate and Rolling upgrades

```
kubectl create -f deployment.yml
kubectl get deployments
kubectl describe deployments frontend
kubectl apply -f deployment.yml
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
kubectl get replicasets
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment
kubectl get replicasets
kubectl run nginx --image=nginx
```

### Commands and Arguments
cat > pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - image: ubuntu-sleeper
    name: ubuntu-sleeper
    command: ["sleep2.0"]
    args: ["10"]
```  

cat > ubuntu-sleeper-3.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-3
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
      - sleep
      - "1200"
```      

### Configure Environment Variables in Applications

cat > pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    env:
      - name: APP_COLOR
        value: pink
```     

```
    env:
      - name: APP_COLOR
        valueFrom:
          configMapKeyRef:
```        

```
    env:
      - name: APP_COLOR
        valueFrom:
          secretKeyRef:
```

```
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MOD=prod
kubectl create configmap app-config --from-file=app_config.properties
```

cat > config-map.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MOD: prod
```

```
kubectl create -f config-map.yml
kubectl get configmaps
kubectl describe configmaps config-map
```

cat > pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    envFrom:
      - configMapRef:
        name: app-config
```        

```
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

```
volumes:
- name: app-config-volume
  configMap:
    name: app-config
```

cat > config-map.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config-map
data:
  APP_COLOR: darkblue
```

### Secrets

```
echo -n 'mysql' | base64
echo -n 'root' | base64
```

cat > secret.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_HOST: bx1zWw=
```

```
kubectl create -f secret-data.yml
kubectl get secrets
kubectl describe secret
kubectl get secret app-secret -o yaml
```

```
echo -n 'bxsdfps=' | base64 --decode
echo -n 'dsfnkls79=' | base64 --decode
```

cat > pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    envFrom:
      - secretRef:
          name: app-secret
```    

```
kubectl create -f pod.yml
```

`kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123`

###  Multi Container PODs

Number of containers available
`kubectl get pod red -o yaml`

cat > multi.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: yellow
spec:
  containers:
  - image: busybox
    name: lemon
  - image: redis
    name: gold
```

`kubectl create -f multi.yml`

`kubectl describe pod app --namespace=elastic-stack`

`kubectl --namespace=elastic-stack exec -it app cat /log/app.log`

cat > /var/answers/answer-app.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: elastic-stack
  labels:
    name: app
spec:
  containers:
  - name: app
    image: kodekloud/event-simulator
    volumeMounts:
    - mountPath: /log
      name: log-volume

  - name: sidecar
    image: kodekloud/filebeat-configured
    volumeMounts:
    - mountPath: /var/log/event-simulator/
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: DirectoryOrCreate
```

### initContainers

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

## Cluster Maintenance

### OS Upgrades

```
kubectl get nodes
kubectl get deployments
kubectl get pod -o wide
kubectl drain node01 --ignore-daemonsets
kubectl uncordon node01
kubectl describe node master
kubectl drain node02 --ignore-daemonsets --force
kubectl cordon node03
```

### Cluster Upgrade Process

kubectl x+1 < x-1

kube-apiserver x

kube-scheduler kube-controller-manager - x-1 

kubelet kube-proxy - x-2 

etcd dns - diff

upgrade all nodes once - downtime

upgrade one by one by moving the load

comeup with new nodes with upgraded version and move one by one


kubelet has to upgrade. If requried on master as well.
`apt-get upgrade -y kubelet=1.12.0-00` or `apt install kubelet=1.12.0-00`

```
kubectl get nodes
kubectl get pods -o wide
kubectl get deployment
```

```
kubeadm upgrade plan
kubectl drain master --ignore-daemonsets
apt install kubeadm=1.12.0-00
kubeadm upgrade apply v1.12.0
apt install kubelet=1.12.0-00
systemctl restart kubelet
kubectl uncordon master
```

```
kubectl drain node01 --ignore-daemonsets
apt install kubeadm=1.12.0-00
apt install kubelet=1.12.0-00
kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2)
systemctl restart kubelet
kubectl uncordon node01
```

### Backup and Restore Methods

Resource config
GitHub
Query to api server - `kubectl get all --all-namespaces - o yaml > all-deploy-services.yaml`

etcd - state of cluster
back up of `--data-dir=/var/lib/etcd`
```
ETCDCTL_API=3 etcdctl snapshot save sanpshot.db
ls
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

Restore
```
service kube-apiserver stop
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db
systemctl daemon-reload
service etcd restart
service kube-apiserver start
```

```
kubectl get deployments
kubectl get pods -n kube-system
kubectl describe pod etcd-master -n kube-system

ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
     snapshot save /tmp/snapshot-pre-boot.db

ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --name=master \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
     --data-dir /var/lib/etcd-from-backup \
     --initial-cluster=master=https://127.0.0.1:2380 \
     --initial-cluster-token etcd-cluster-1 \
     --initial-advertise-peer-urls=https://127.0.0.1:2380 \
     snapshot restore /tmp/snapshot-pre-boot.db
```

Update --data-dir to use new target location
`--data-dir=/var/lib/etcd-from-backup`

Update new initial-cluster-token to specify new cluster
`--initial-cluster-token=etcd-cluster-1`

Update volumes and volume mounts to point to new path
```
    volumeMounts:
    - mountPath: /var/lib/etcd-from-backup
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
```

## Security

**Serverside:** api-server, etcd, kubelet

**client side:** admin, user, kube-scheduler, controller-manager, apiserver-kubelet, apiserver-etcd, kubelet-client

### TLS in Kubernetes - Certificate Creation

```
openssl genrsa -out ca.key 2048
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

```
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

`curl https://kube-apiserver:6443/api/v1/pods --key admin.key --cert admin.crt --cacert ca.crt`

scheduler.key, scheduler.csr, scheduler.crt
controller-manager.key, controller-manager.csr, controller-manager.crt
kube-proxy.key, kube-proxy.csr, kube-proxy.crt

`openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr -config openssl.cnf`
etcdserver.key, etcdserver.csr, etcdserver.crt
etcdpeer1.key, etcdpeer1.csr, etcdpeer1.crt

/etc/systemd/system/api-server.service
/etc/kubernetes/manifests/api-server.yml

/etc/kubernetes/pki/ca.crt

`openssl x509 -in apiserver.crt -text -noout`

`journalctl -u etcd.service -l`

`kubectl logs etcd-master`

```
docker ps -a
docker logs containerid
```

```
kubectl get pods -n kube-system
kubectl describe pod kube-apiserver-master -n kube-system
cat /etc/kubernetes/manifests/kube-apiserver.yaml
cat /etc/kubernetes/manifests/etcd.yaml
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text
```

```
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cert
cat /etc/kubernetes/manifests/etcd.yaml | grep cert
```

`openssl x509 -req -in /etc/kubernetes/pki/apiserver-etcd-client.csr -CA /etc/kubernetes/pki/etcd/ca.crt -CAkey /etc/kubernetes/pki/etcd/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-etcd-client.crt`

### Certificates API

```
openssl genrsa -out jane.key 2048
openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr
```

Get encoded text of jane.csr using `cat jane.csr | base64`

cat > jane-csr.yaml
```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata: 
  name: jane
spec:
  groups:
  - system:autheticated
  usages:
  - digital signature
  - key encipherment
  - server auth
  request:
    xxENCODEDTEXTxx
```

```
kubectl get csr
kubectl certificate approve jane
```

To end user share certificate
```
kubectl get csr jane -o yaml
echo "xxENCODEDTEXTxx" | base64 --decode
```

contoller manager takes care of certificates signing requests

`cat /etc/kubenetes/manifests/kube-controller-manager.yaml`

`cat > /var/answers/akshay-csr.yaml`
```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZV3R6YUdGNU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQXV2bHBESjVyaGUyYXd4b0ptZUltN1MwL3dBN3hJRGhKT0xuZnJSMzhZMzJzCkJ6NHRxS3R1MFE1enJzZTE2bkI0dGNqUzFjNGlNYVBMRFVqTis5aUpjWm1lclpTdTYyMDFKaHFBN2JzWDArSVAKeFI4dXpEREY2UmFLdWN5S2pCZnFRdlIvNzQ1eFNTLzNhOXdOaUwycStMUXNzNEpkNU0vbHhMb0w1N2JzdXhmUgpZbTkwdnI4czFyYUJOVUJuUVJGd1VYcDIyNFJVcm8rc1VpZ05FZWN3aTFkaVVESWZoVUFRVFkzV1RPQVhmM21xCnFLcURSMnBpd3cwVkdIN1YzZ3N1Zit2WHVLbGV3QnFmNEpYeTV3UWsrVXNwc0VXZTkrY0k3QjllcTcyRHYydWsKQjF3YjlaK0t2cmVTWWNjeGI1bHpsRXkwVFg5Ulo4akpIV3hJUEZMYU13SURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBSVg3UmEzSzlsUEVYNEEyeWZxVG0rcEMybndwenNsYjhsdGV5MzhrRnhoZHcydkRNbHNpCjhNQkhzeXlaT1dXdHc3bWEzc3g2bFBDcDM4ZzMwUUw2UUZMYnFsdXFWRHRRdlJaU2h1VUgyNmNMdzhKZEpHUDcKT3FxMlJlTXo1eklsWjV4emtYajRIN0JPbzl6ZjVtOVRIV3lLTmx4aVRldEplTmZYcXIwWWFFdFp1eFhSaGdqawpVVDg0NzMvazgrakdSL1V2aFZzMWVNdE5pM1FnajhSYW4xdythOUdvSDNOdUpnOXB6czV1a0Y4LzhJcG5MYzJZCmZSOFZlWnpDcnFtYnEySWd4ZEt1cTNXN29yY2FacjArcHVKMWhTM2ZsR2pNNWxEa3lpTCtmanR3dXNXdGRiVXMKZWdVaXpJdHRQM3ppTURjYnNhek5wUk9KMm5ET25aQkRmS1k9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  usages:
  - digital signature
  - key encipherment
  - server auth
```

```
kubectl create -f /var/answers/akshay-csr.yaml
kubectl get csr akshay
kubectl certificate approve akshay
kubectl get csr agent-smith -o yaml
kubectl certificate deny agent-smith
kubectl delete csr agent-smith
```

### KubeConfig

```
curl https://my-kube-playground:6443/api/v1/pods --key admin.key --cert admin.crt --cacert ca.crt
kubectl get pods --server my-kube-playground:6443 --client-key admin.key --client-certificate admin.crt --certificate-authority ca.crt
```

Save the configuration in a file

`kubectl get pods --kubeconfig config`

cat > $HOME/.kube/config
```
--server my-kube-playground:6443
--client-key admin.key
--client-certificate admin.crt
--certificate-authority ca.crt
```

`kubectl get pods`


kubeconfig file has 3 - clusters, contexts, users

```
apiVersion: v1
kind: Config

current-context: dev-user@google

clusters:
- name: production
  cluster:
    certificate-authority: ca.crt
    server: https://172.17.0.51:6443

contexts:
- name: dev-user@google
  context:
    cluster: production
    user:dev-user
    namespace: default
- name: prod-user@production
  context:
    cluster: production
    user:prod-user
    namespace: finance

users:
- name: dev-user
  user:
    client-certificate: dev-user.crt
    client-key: dev-user.key
- name: prod-user
  user:
    client-certificate: prod-user.crt
    client-key: prod-user.key
```

```
kubectl config view
kubectl config view --kubeconfig=my-custom-config
kubectl config use-context prod-user@production
```

`cat certificate-authority: ca.crt | base64` and add it to configfile under `certificate-authority-date:`

`kubectl config --kubeconfig=/root/my-kube-config use-context research`


### API Groups

cluster apis - core and named named

API Groups(/apps v1, etc) - Resources(/deployments, etc) - Verbs(get, delete, etc)

To get available apis: `curl http://localhost:6443 -k` `curl http://localhost:6443 -k | grep name`

```
kubectl proxy
curl http://localhost:8001 -k
```

### RBAC

```
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep auth
kubectl get roles
kubectl get roles --all-namespaces
kubectl describe role weave-net -n kube-system

kubectl get rolebindings
kubectl get rolebindings --all-namespaces
kubectl describe rolebindings weave-net -n kube-system

kubectl get pods --as dev-user
```

cat > developer.yml
```
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get, create, delete"]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

`kubectl create -f developer.yml`

```
kubectl get pod dark-blue-app -n blue --as dev-user
```

cat > dev-user-deploy.yaml
```
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: blue
  name: deploy-role
rules:
- apiGroups: ["apps", "extensions"]
  resources: ["deployments"]
  verbs: ["create"]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-deploy-binding
  namespace: blue
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: deploy-role
  apiGroup: rbac.authorization.k8s.io
```

`kubectl create -f dev-user-deploy.yaml`

### Cluster Roles and Role Bindings

```
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```

```
kubectl get clusterroles | wc -l
kubectl get clusterrolebindings | wc -l

kubectl describe clusterrolebinding cluster-admin
kubectl describe  clusterrole cluster-admin
```

cat > michelle-node-admin.yaml
```
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: node-admin
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "watch", "list", "create", "delete"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-binding
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-admin
  apiGroup: rbac.authorization.k8s.io
```

`kubectl create -f michelle-node-admin.yaml`

cat > michelle-storage-admin.yaml
```
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: storage-admin
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "watch", "list", "create", "delete"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "watch", "list", "create", "delete"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-storage-admin
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: storage-admin
  apiGroup: rbac.authorization.k8s.io
```

`kubectl create -f michelle-storage-admin.yaml`


### Image Security

image: nginx

image: registry/account/image

image: docker.io/nginx/nginx

`docker login private-registry.io` with username and password

`docker run private-registry.io/apps/internal-app`

`kubectl create secret docker-registry regcred --docker-server=private-registry.io --docker-username=registry-user --docker-passwrod=registry-password --docker-email=registry-user@org.com`

cat > redis.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - image: redis
    name: redis
  imagePullSecrets:
  - name: regcred
```

`kubectl create secret docker-registry private-reg-cred --docker-server=myprivateregistry.com:5000 --docker-username=dock_user --docker-password=dock_password --docker-email=dock_user@myprivateregistry.com`

cat > deployment.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    run: web
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
      run: web
  template:
    metadata:
      labels:
        run: web
    spec:
      containers:
      - image:  myprivateregistry.com:5000/nginx:alpine
        name: web
      imagePullSecrets:
      - name: regcred  
```        

### Security Contexts

cat > pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
  - image: ubuntu
    name: ubuntu
    command: ["sleep", "3600"]
    securityContext:
      runAsUser: 1000
      capabilities:
        add: ["MAC_ADMIN"]
```

`kubectl exec ubuntu-sleeper whoami`

cat > multi-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext:
      runAsUser: 1002

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]
```

```
kubectl exec ubuntu-sleeper whoami

```

### Network Policy

wed - api - dd

web - ingress 80, egress 5000

api - ingress 5000, egress 3306

db - ingress 3306

"All Allow" by default

Allow ingress traffic to db app only on 3306 from api pod
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db   
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```

```
kubectl get networkpolicies
kubectl get networkpolicies -o wide
kubectl describe networkpolicies payroll-policy
```

cat > /var/answers/answer-internal-policy.yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
    - {}
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080
```

### Volumes

cat > volumes.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/su", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out"]
    
    volumeMounts:
    - name: data-volume
      mountPath: /opt

  volumes:
  - name: data-volume
    hostpath:
      path: /data
      type: Directory   
```      

hostpath replaced with awsElasticBlockStorage
```
  volumes:
  - name: data-volume
    awsElasticBlockStorage:
      volumeID: <volume-id>
      fsType: ext4 
```

### Persistent Volumes

cat > pv.yml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 2Gi
  hostPath:
    path: /tmp/data
```

`kubectl create -f pv.yml`

`kubectl get persistentvalumes`

### Persistent Volumes Claims

cat > pvc.yml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

```
kubectl create -f pvc.yml`
kubectl get persistentvalumesclaims
kubectl delete persistentvalumesclaim myclaim
```

by default `persistentVolumeReclaimPolicy: Retatin` and other options are `Delete` and `Recycle`.


```
kubectl exec webapp cat /log/app.log
```

cat > /var/answers/webapp-pod-hostpath.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: Directory
```

```
kubectl delete pod webapp
ubectl create -f webapp-pod-hostpath.yaml
```

cat > pv.yml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
```

```
kubectl create -f pv.yml
kubectl get pv
```

cat > pvc.yml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
```    

```
kubectl create -f pvc.yml
kubectl get pvc
```

cat > /var/answers/webapp-pod-pvc.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: claim-log-1
```

```
kubectl delete pod webapp
kubectl create -f webapp-pod-pvc.yaml
```

```
kubectl get pv
kubectl delete pvc claim-log-1
kubectl delete pod webapp
kubectl delete pvc claim-log-1
kubectl get pvc
kubectl get pv
```

## Networking

### Cluster Networking

`nslookup www.google.com`

```
ip link
ip link add v-net-0 type bridge
ip addr
ip addr add 192.168.1.10/24 dev eth0
ip route
ip route add 192.168.1.0/24 via 192.168.2.1
cat /proc/sys/net/ipv4/ip_forward
arp
netstat -plnt
route
```

```
kubectl get nodes
ip link
kubectl get nodes -o wide
arp
ip route
netstat -nplt
```

create netrwork - `ip link add v-net-0 type bridge`
bringup network - `ip link set dev v-net-0 up`
assign ip to network - `ip addr add 10.244.1.1/24 dev v-net-0`

### CNI Weave

```
ps -aux | grep kubelet
kubectl get pods -n kube-system -o wide
ip link
ip addr show weave
```

cat > nginx.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  labels:
    name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["sleep", "10"]
  nodeName: node03
```

```
kubectl create -f pod.yml
kubectl exec busybox ip route
```

`kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')`

### Service Networking

`iptables -L -t net | grep db-service`

```
ip addr
```

### CoreDNS in Kubernetes

```
kubectl get pods -n kube-system
kubectl get service -n kube-system
kubectl exec coredns-78fcdf6894-l2vhz -n kube-system ps
kubectl get configmap -n kube-system
kubectl describe configmap coredns -n kube-system 
```

### Ingress

```
kubectl get all --all-namespaces
kubectl get ingress --all-namespaces
kubectl describe ingress --namespace app-space
```

cat > ingress-pay.yaml
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```

cat > ingress.yml
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: ingress-wear-watch
  namespace: app-space
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: wear-service
          servicePort: 8080
        path: /wear
      - backend:
          serviceName: video-service
          servicePort: 8080
        path: /watch
      - backend:
          serviceName: video-service
          servicePort: 8080
        path: /stream
      - backend:
          serviceName: food-service
          servicePort: 8080
        path: /eat
```

```
kubectl create namespace ingress-space
kubectl create configmap nginx-configuration -n ingress-space
kubectl create serviceaccount ingress-serviceaccount -n ingress-space
kubectl get roles,rolebindings --namespace ingress-space
```

cat > ingress-controller-answer-file.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: ingress-controller
  namespace: ingress-space
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      serviceAccountName: ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --default-backend-service=app-space/default-http-backend
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
```

`kubectl expose deployment -n ingress-space ingress-controller --type=NodePort --port=80 --name=ingress --dry-run -o yaml > ingress.yaml`

cat > ingress.yaml
```
apiVersion: v1
kind: Service
metadata:
  labels:
    name: nginx-ingress
  name: ingress
  namespace: ingress-space
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30080
  selector:
    name: nginx-ingress
  type: NodePort
```


## Networking

### Switching & Routing
```
ip link
ip addr
ip addr add 192.168.1.10/24 dev eth0
ip addr
```

```
route
ip route add 192.168.2.0/24 via 192.168.1.1
ip route add default via 192.168.1.1
route
```

`echo 1 > /proc/sys/net/ipv4/ip_forward`

/etc/sysctl.conf
`net.ipv4.ip_forward = 1`


### DNS

`ping db`

cat >> /etc/hosts
```
192.168.1.11 db
```

`ping db`

cat /etc/resolv.conf
```
nameserver 192.168.1.180
search mycompany.com
```

`ping db`

/etc/hosts in the server has higher precedence than /etc/resolv.conf

```
nslookup www.google.com
dig www.google.com
```

### Core DNS

```
wget https://github.com/coredns/coredns/releases/download/v1.4.0/coredns_1.4.0_linux_amd64.tgz
tar –xzvf coredns_1.4.0_linux_amd64.tgz
```

In dns server all ip addresses and names will keep in file called `cat /etc/hosts`

cat > Corefile
```
. {
    hosts /etc/hosts
}
```
`./coredns`


```
ip netns add red
ip netns add blue
ip netns
```

```
ip link
ip netns exec red ip link
```

```
arp
ip netns exec red arp
```

```
route
ip netns exec red route
```

```
ip link add veth-red type veth peer name veth-blue
ip link set veth-red netns red
ip link set veth-blue netns blue
ip -n red addr add 192.168.15.1 dev veth-red
ip -n blue addr add 192.168.15.2 dev veth-blue
ip -n red link set veth-red up
ip -n blue link set veth-blue up
ip netns exec red ping 192.168.15.2
ip netns exec red arp
ip netns exec blue arp
arp
```

```
ip link add v-net-o type bridge
ip link
ip link set dev v-net-0 up
ip link add veth-red type veth peer name veth-red-br
ip link add veth-blue type veth peer name veth-blue-br
ip link set veth-red netns red
ip link set veth-red-br master v-net-0
ip link set veth-blue netns blue
ip link set veth-blue-br master v-net-0
ip -n red addr add 192.168.15.1 dev veth-red
ip -n blue addr add 192.168.15.2 dev veth-blue
ip -n red link set veth-red up
ip -n blue link set veth-blue up
ping 192.168.15.1
ip addr add 192.168.15.5/24 dev v-net-0
ping 192.168.15.1
ip netns exec blue ping 192.168.1.3
ip netns exec blue route
ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5
ip netns exec blue ping 192.168.1.3
```

```
ip netns exec blue ping 8.8.8.8
ip netns exec blue route
ip netns exec blue ip route add default via 192.168.15.5
```


## Install

https://github.com/mmumshad/kubernetes-the-hard-way

https://github.com/mmumshad/kubernetes-the-hard-way/blob/master/practice-questions-answers/install/bootstrap-worker-node-2/tls-bootstrap-worker-node-2.md

```
kubeadm version
sudo apt-get update && sudo apt-get install -qy kubeadm=1.16.0-00
sudo apt-get update && sudo apt-get install -qy kubelet=1.16.0-00
kubeadm init
```

### Practice Test - TLS Bootstrap Worker Node

**In master**
```
kubectl get nodes
kubeadm generate token
kubeadm token list
```
  
cat > token.yml
```
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-wut4dy
  namespace: kube-system

type: bootstrap.kubernetes.io/token
stringData:
  description: "The default bootstrap token generated by 'kubeadm init'."

  token-id: wut4dy
  token-secret: mbamenfklgffh9ww

  expiration: 2020-03-10T03:22:11Z

  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"

  auth-extra-groups: system:bootstrappers:node03
```
```
kubectl create -f token.yml
kubeadm token list

kubectl create clusterrolebinding crb-bootstrappers --clusterrole=system:node-bootstrapper --group=system:bootstrappers
kubectl create clusterrolebinding crb-to-approve-csr --clusterrole=system:certificates.k8s.io:certificatesigningrequests:nodeclient --group=system:bootstrappers
kubectl create clusterrolebinding crb-autorenew-csr-for-nodes --clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeclient --group=system:nodes
```

**In Node**
```
kubectl cluster-info

kubectl config --kubeconfig=/tmp/bootstrap-kubeconfig set-cluster bootstrap --server='https://172.17.0.21:6443' --certificate-authority=/etc/kubernetes/pki/ca.crt
kubectl config --kubeconfig=/tmp/bootstrap-kubeconfig set-credentials kubelet-bootstrap --token=wut4dy.mbamenfklgffh9ww
kubectl config --kubeconfig=/tmp/bootstrap-kubeconfig set-context bootstrap --user=kubelet-bootstrap --cluster=bootstrap
kubectl config --kubeconfig=/tmp/bootstrap-kubeconfig use-context bootstrap
```

cat > /etc/systemd/system/kubelet.service
```
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/bin/kubelet \
  --bootstrap-kubeconfig=/var/lib/kubelet/bootstrap-kubeconfig \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --register-node=true \
  --v=2
Restart=on-failure
StandardOutput=file:/var/kubeletlog1.log
StandardError=file:/var/kubeletlog2.log
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/tmp/bootstrap-kubeconfig --kubeconfig=/var/lib/kubelet/kubeconfig"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```
```
systemctl daemon-reload
service kubelet start
```

**In master**
`kubectl get ndoes`

### kubeadm cluster

```
apt-get update
apt-get install -y kubeadm
kubeadm version

ssh node01
apt-get update
apt-get install -y kubeadm
kubeadm version
exit

kubeadm init

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get nodes

kubeadm token list

ssh node01
```

```
kubeadm join 172.17.0.40:6443 --token cmlzjb.lpw3cx7adaxyn6i9 \
    --discovery-token-ca-cert-hash sha256:6afb6327025dd449b787a60761cee820e3a99ea9d9bd9e5248ee1b99395d857c

exit
```
`kubectl get nodes`

or

```
kubeadm token generate
ssh node01
```

Get --discovery-token-ca-cert-hash from below command
```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'

kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>

exit
```

```
kubectl get nodes

sysctl net.bridge.bridge-nf-call-iptables=1
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

kubectl get nodes
```

## Troubleshooting

### Application Failure

mysql-service ServiceName is worng
```
kubectl -n alpha get all
kubectl -n alpha get svc mysql -o yaml > mysql-service.yml
kubectl -n alpha delete svc mysql
vi mysql-service.yml
kybectl create -f mysql-service.yml
```

mysql-service Port number is wrong
```
kubectl -n beta get all
kubectl -n beta describe svc mysql-service
kubectl -n beta get svc mysql-service -o yaml > mysql-service.yml
kubectl -n beta delete svc mysql
vi mysql-service.yml
kybectl create -f mysql-service.yml
```

mysql-service selector is wrong
```
kubectl -n gamma get all
kubectl -n gamma get ep
kubectl -n gamma get svc mysql-service -o yaml
kubectl -n gamma describe pod mysql
kubectl -n gamma delete svc mysql
kubectl -n gamma expose pod mysql --name=mysql-service
kubectl -n gamma get ep
```

environment variables wrong
```
kubectl -n delta describe deployment webapp-mysql
kubectl -n delta get deployment webapp-mysql -o yaml > web.yml
kubectl -n delta delete deployment webapp-mysql
vi web.yml
kubectl create -f web.yml
```

environment variables wrong and update password in mysql pod
```
kubectl -n epsilon get all 
kubectl -n epsilon get deployment webapp-mysql -o yaml > web.yml
kubectl -n epsilon delete deployment webapp-mysql
vi web.yml
kubectl create -f web.yml
kubectl -n epsilon get pod mysql -o yaml > mysql.yml
vi mysql.yml
kubectl -n epsilon delete pod mysql
kubectl create -f mysql.yml
```

Wrong nodeport for web-service, DB_USER name is wrong and MYSQL_ROOT_PASSWORD is wrong
```
kubectl -n zeta get svc web-service
kubectl -n zeta get svc web-service -o yaml > web-service.yml
kubectl -n zeta delete svc web-service
kubectl create -f web-service.yml
kubectl -n zeta get deployment webapp-mysql -o yaml > web.yml
kubectl -n zeta delete deployment webapp-mysql
vi web.yml
kubectl create -f web.yml
kubectl -n zeta get pod mysql -o yaml > mysql.yml
kubectl -n zeta delete pod mysql
kubectl create -f mysql.yml
```

### Control Plane Failure

```
kubectl get nodes
kubectl get pods
kubectl get pods -n kube-system

service kube-apiserver status
service kube-controller-manager status
service kube-scheduler status

service kubelet status
service kube-proxy status

kubectl logs kube-apiserver-master -n kube-system

sudo journalctl -u kube-apiserver
```

kube-scheduler-master pod failure
```
kubectl get all
kubectl get all -n kube-system
kubectl logs kube-scheduler-master -n kube-system
vi /etc/kubernetes/manifests/kube-scheduler.yaml
kubectl get pod kube-scheduler-master -n kube-system
```

kube-controller-manager-master pod failure
```
kubectl get all
kubectl get all -n kube-system
kubectl logs kube-controller-manager-master -n kube-system
vi /etc/kubernetes/manifests/kube-scheduler.yaml
kubectl get pod kube-controller-manager-master -n kube-system
```

kube-controller-manager-master pod failure - volume mount path is wrong

### Worker Node Failure

```
kubectl get nodes
kubectl describe node worker-1

top
df -h
service kubelet status
sudo journalctl -u kubelet
```

check certificates


kubelet service stopped
```
kubectl get nodes
ssh node01 
service kubelet status
service kubelet start
exit
kubectl get nodes
```

ca cert file is wrong
```
kubectl get nodes
ssh node01 
/var/lib/kubelet/config.yaml
```

kube-apiserver port is wrong
```
vi /etc/kubernetes/kubelet.conf
service kubelet restart
```

## Other

### JSON PATH

```
kind
metadata.name
spec.nodeName
spec.containers[0]
spec.containers[0].image
status.phase
status.containerStatuses[0].state.waiting.reason
status.containerStatuses[1].restartCount
status.containerStatuses[?(@.name == 'redis-container')].restartCount
```

```
$[*].metadata.name
$.users[*].name
```

```
kubectl get nodes -o json > /opt/outputs/nodes.json
kubectl get node node01 -o json > /opt/outputs/node01.json
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}' > /opt/outputs/node_names.txt
kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os.txt
kubectl config view --kubeconfig=/root/my-kube-config -o=jsonpath='{.users[*].name}' > /opt/outputs/users.txt
kubectl get pv --sort-by=.spec.capacity.storage > /opt/outputs/storage-capacity-sorted.txt

kubectl get pv --sort-by=.spec.capacity.storage -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage > /opt/outputs/pv-and-capacity-sorted.txt

kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}" > /opt/outputs/pv-and-capacity-sorted.txt
```

### Mock Exam-1

```
kubectl run --generator=run-pod/v1 nginx-pod --image=nginx:alpine

kubectl run --generator=run-pod/v1 messaging --image=redis:alpine
kubectl edit pod messaging

kubectl create namespace apx-x9984574

kubectl get nodes -o json > /opt/outputs/nodes-z3444kd9.json

kubectl expose pod messaging --port=6379 --name messaging-service

kubectl create deployment hr-web-app --image=kodekloud/webapp-color
kubectl edit deployment hr-web-app

kubectl run --generator=run-pod/v1 static-busybox --image=busybox --dry-run -o yaml > pod.yml
```

vi pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: static-busybox
  name: static-busybox
spec:
  containers:
  - image: busybox
    name: static-busybox
    command:
    - "sh"
    - "-c"
    - "sleep 1000"
```
`cp pod.yml /etc/kubernetes/manifests/`

```
kubectl run --generator=run-pod/v1 temp-bus --namespace=finance --image=redis:alpine
kubectl get pods --namespace=finance
```

`sleeeep` command name is wrong

```
kubectl expose deployment hr-web-app --type=NodePort --port=8080 --name=hr-web-app-service --dry-run -o yaml > hr-web-app-service.yaml

kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os_x43kj56.txt
```

cat > pv.yml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
  - ReadWriteMany
  hostPath:
    path: /pv/data-analytics
```

`kubectl create -f pv.yml`


### Mock Exam-2

```
ETCDCTL_API=3 etcdctl --endpoints https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /tmp/etcd-backup.db
```

cat > empty-pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: redis-container
    volumeMounts:
    - mountPath: /data/redis
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```
`kubectl create -f empty-pod.yml`

cat > busybox.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: super-user-pod
spec:
  containers:
  - image: busybox:1.28
    name: busybox-container
    command:
    - "sh"
    - "-c"
    - "sleep 4800"
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
```
`kubectl create -f busybox.yml`

`kubectl taint node master node-role.kubernetes.io/master:NoSchedule-`

cat > pvc.yml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Mi
```
`kubectl create -f pvc.yml`

cat > /root/use-pv.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: use-pv
spec:
  containers:
  - image: nginx
    name: nginx-container
    volumeMounts:
      - mountPath: "/data"
        name: my-pv
  volumes:
    - name: my-pv
      persistentVolumeClaim:
        claimName: my-pvc
```
`kubectl create -f /root/use-pv.yaml`

```
kubectl run nginx-deploy --image=nginx:1.16 --replicas=1 --record
kubectl rollout history deployment nginx-deploy

kubectl set image deployment/nginx-deploy nginx=nginx:17 --record
kubectl rollout history deployment nginx-deploy
```

```
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: john-developer
  namespace: development
spec:
  request: $(cat /root/john.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF
```
```
kubectl get csr
kubectl certificate approve john-developer
kubectl get csr
```

```
kubectl create role developer --resources=pods --verbs=create,list,get,update,delete --namespace=development
kubectl create rolebinding developer-role-binding --role=developer --user=john --namespace=development
kubectl auth can-i update pods --as=john --namespace=development
```

```
kubectl run --generator=run-pod/v1 nginx-resolver --image=nginx
kubectl expose pod nginx-resolver --name=nginx-resolver-service --port=80 --target-port=80 --type=ClusterIP

kubectl run --generator=run-pod/v1 test-nslookup --image=busybox:1.28 --rm -it -- nslookup nginx-resolver-service > /root/nginx.svc

kubectl get pod mginx-resolver -o wide
kubectl run --generator=run-pod/v1 test-nslookup --image=busybox:1.28 --rm -it -- nslookup 10-32-0-5.default.pod > /root/nginx.pod
```

```
kubectl get nodes
kubectl run --generator=run-pod/v1 nginx-critical --image=nginx --dry-run -o yaml > nginx-critical.yml
cat nginx-critical.yml

ssh node01
systemctl status kubelet
vi /var/lib/kubelet/config.yml
mkdir -p /etc/kubernetes/manifests
cat > /etc/kubernetes/manifests/nginx-critical.yml
docker ps| grep -i "nginx-critical"
exit

kubectl get pods
```

### Mock exam-3

```
kubectl create serviceaccount pvviewer
kubectl create clusterrole pvviewer-role --resource=persistentvolumes --verb=list
kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default.pvviewer
kubectl run --generator=run-pod/v1 pvviewer --image=redis --dry-run -o yaml > pod.yml
```

vi pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pvviewer
  name: pvviewer
spec:
  containers:
  - image: redis
    imagePullPolicy: IfNotPresent
    name: pvviewer
  serviceAccountName: pvviewer
```
`kubectl create -f pod.yml`


`kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'`

`kubectl run --generator=run-pod/v1 multi-pod --image=nginx --dry-run -o yaml > multi-pod.yml`

vi multi-pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  containers:
  - name: alpha
    image: nginx
    env:
      - name: name
        value: alpha

  - name: beta
    image: busybox
    command: ["sh", "-c", "sleep 4800"]
    env:
      - name: name
        value: beta
```
`kubectl create -f multi-pod.yml`


```
kubectl get pods
kubectl describe pod np-test-1
kubectl describe svc np-test-service
kubectl run --generator=run-pod/v1 test-np --image=busybox:1:28 --rm -it -- sh
# nc -z -v -w 2 np-tes-service 80
kubectl get netpol
kubectl describe netpol default-deny
```

`kubectl api-version | grep -i netpol`

cat > network.yml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
```
```
kubectl create -f network.yml
kubectl run --generator=run-pod/v1 test-np --image=busybox:1.28 --rm -it -- sh
# nc -z -v -w 2 np-test-service 80
```

`kubectl run --generator=run-pod/v1 non-root-pod --image=redis:alpine --dry-run -o yaml > non-root-pod.yml`

vi non-root-pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  containers:
  - image: redis:alpine
    name: non-root-pod
    securityContext:
      runAsUser: 1000
      fsGroup: 2000
```
`kubectl create -f non-root-pod.yml`


```
kubectl taint node node01 env_type=production:NoSchedule
kubectl run --generator=run-pod/v1 dev-redis --image=redis:alpine
kubectl get pod dev-redis -o wide
kubectl run --generator=run-pod/v1 prod-redis --image=redis:alpine --dry-run -o yaml > prod-pod.yml
```

vi prod-pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: prod-redis
  name: prod-redis
spec:
  containers:
  - image: redis:alpine
    name: prod-redis
  tolerations:
  - key: "env_type"
    operator: "Equal"
    value: "production"
    effect: "NoSchedule"
```
```
kubectl create -f prod-pod.yml
kubectl get pod prod-redis -o wide
```

`kubectl run --generator=run-pod/v1 hr-pod --image=redis:alpine --dry-run -o yaml > hr-pod.yml`

cat hr-pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    environment: production
    tier: frontend
  name: hr-pod
  namespace: hr
spec:
  containers:
  - image: redis:alpine
    name: hr-container
```
```
kubectl create -f hr-pod.yml
kubectl get pod hr-pod -n hr
```

```
kubectl cluster-info --kubeconfig=/root/super.kubeconfig
```
api-server port is wrong

kube-controller is not working
```
kubectl get pods -n kube-system
sed -i 's/kube-contro1ler-manager/kube-controller-manager/g' /etc/kubernetes/manifests/kube-controller-manager.yaml
```
