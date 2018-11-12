
# Kubernetes memo to prepare CKA - v 1.11.0 (Or just using k8s!)

# Index list


# Commands list

Monitor the progress for a Deployment
``` bash kubectl rollout status ```

Monitor the progress for a Deployment
``` bash kubectl rollout status

``` bash
kubectl create namespace mem-example
kubectl create ns linuxcon
kubectl create secret generic mysql --from-literal=password=root


kubectl --v=9 get pods

kubectl get endpoints
kubectl get pvc
kubectl get secrets
kubectl get configmap colors
kubectl get nodes --show-labels
kubectl get pods -n accounting
kubectl get pods --show-labels
kubectl get deployment,rs,pods -o json
kubectl get pods -L system

kubectl rollout pause deployment/ghost
kubectl rollout resume deployment/ghost
kubectl rollout history ds ds-one --revision=2

kubectl exec busybox cat /etc/resolv.conf
kubectl exec -it shell-demo -- /bin/bash -c 'echo $ilike'
kubectl exec -ti busybox -- /bin/sh

kubectl run -i -t busybox --image=busybox --restart=Never
kubectl run busybox --image=busybox --command sleep 3600

kubectl proxy --api-prefix=/
kubectl logs date-1539006240-dhb78

kubectl edit deployments dev-web

kubectl label pods ghost-864655d77f-rnddk foo=bar

kubectl delete rs rs-one --cascade=false

kubectl set image ds ds-one nginx=nginx:1.8.1-alpine

kubectl describe secrets default-token-6chzc

kubectl -n kube-system get secrets certificate-controller-token-j9psf  -o yaml
kubectl config set-credentials -h
kubeadm token -h
kubeadm config -h
```

## Independants
#### Using Zsh
``` bash
if [ $commands[kubectl] ]; then
  source <(kubectl completion zsh)
fi
```
or
``` bash
plugins=(kubectl)
```

#### Kubectl
``` bash
## Download
wget https://storage.googleapis.com/kubernetes-release/release/v1.11.0/bin/linux/amd64/kubectl
mv kubectl /usr/local/bin/kubectl
chmod +x /usr/local/bin/kubectl
/usr/local/bin/kubectl version
```

#### Minikube
``` bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.29.0/minikube-linux-amd64
chmod +x minikube
mv minikube /usr/local/bin/
minikube
```



### ETCD
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

#### Backup
https://coreos.com/etcd/docs/latest/v2/admin_guide.html
``` bash
    etcdctl backup \
      --data-dir %data_dir% \
      [--wal-dir %wal_dir%] \
      --backup-dir %backup_data_dir%
      [--backup-wal-dir %backup_wal_dir%]
```

## Basic setting up with kubeadm
https://v1-11.docs.kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

#### Docker (Master + worker)
``` bash
sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
apt-key fingerprint 0EBFCD88

add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"

apt-get update
apt-get install docker-ce
```

#### Kubernetes (Master + worker)
``` bash
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet=1.11.4-00 kubeadm=1.11.4-00 kubectl=1.11.4-00
apt-mark hold kubelet kubeadm kubectl
```

#### Kubernetes Init (Master)
``` bash
kubeadm init  --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Joining your nodes (Worker)
``` bash
kubeadm join 172.16.0.111:6443 --token pnaven.wxx..nu10c --discovery-token-ca-cert-hash sha256:3e....4ed39f9b9ede
```

#### Calico network (Master)
``` bash
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

## Cluster auditing && logs
https://v1-11.docs.kubernetes.io/docs/tasks/debug-application-cluster/audit/

#### Testing
``` bash
kubectl cluster-info
kubectl get nodes
kubectl get componentstatuses
kubectl get pods -o wide --show-labels --all-namespaces
kubectl get svc  -o wide --show-labels --all-namespaces
```

#### Full log
``` bash
kubectl apply -f https://raw.githubusercontent.com/ThomasGsp/K8s-memo/master/ressources/audit-policy-full.yaml
```

#### Log all requests at the Metadata level.
``` bash
kubectl apply -f https://raw.githubusercontent.com/ThomasGsp/K8s-memo/master/ressources/audit/audit-policy-min.yaml
```

## Basic deployment

#### Nginx version 1.7.9 in 2 pods
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
``` bash
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      # unlike pod-nginx.yaml, the name is not included in the meta data as a unique name is
      # generated from the deployment name
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.12.2
        ports:
        - containerPort: 80

```
#### Scale this to 4 pods.
``` bash
kubectl scale deployment nginx-deployment --replicas=4
```
#### Scale it back to 2 pods.
``` bash
kubectl scale deployment nginx-deployment --replicas=2
```
#### Upgrade nginx version to 1.13.8
``` bash
kubectl edit deployment nginx-deployment
```
#### Check the status of the upgrade
``` bash
kubectl get pods -l app=nginx
```
#### How do you do this in a way that you can see history of what happened?
``` bash
kubectl rollout history deployment/nginx-deployment
```
#### Undo the upgrade
``` bash
kubectl rollout undo deployment/nginx-deployment
```


## Basic deployment -- without using a manifest (Busybox)

#### Create
``` bash
kubectl run busybox --image=busybox:1.28.4 --command -- sleep 3600
```

#### Edit
``` bash
kubectl edit pod busybox-c8d74cd49-64fzq
```

## Liveness and Readiness Probes
https://v1-11.kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/

#### liveness
The kubelet uses liveness probes to know when to restart a Container. For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a Container in such a state can help to make the application more available despite bugs.

#### readiness
The kubelet uses readiness probes to know when a Container is ready to start accepting traffic. A Pod is considered ready when all of its Containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.

#### Examples

``` bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600

    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5

    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

``` bash
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

## Daemon set
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

https://www.mirantis.com/blog/scaling-kubernetes-daemonsets/
#### Create
``` bash
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: frontend
spec:
  template:
    metadata:
      labels:
        app: frontend-webserver
    spec:
      nodeSelector:
        app: frontend-node
      containers:
        - name: webserver
          image: nginx
          ports:
          - containerPort: 80
```

#### Change strategy
``` bash
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: frontend
spec:
  updateStrategy: RollingUpdate
    maxUnavailable: 1
    minReadySeconds: 0
  template:
    metadata:
      labels:
        app: frontend-webserver
    spec:
      nodeSelector:
        app: frontend-node
      containers:
        - name: webserver
          image: nginx
          ports:
          - containerPort: 80

```

## Secrets
https://kubernetes.io/docs/concepts/configuration/secret/

#### Create secret
``` bash
# Create files needed for rest of example.
$ echo -n 'admin' > ./username.txt
$ echo -n '1f2d1e2e67df' > ./password.txt
```

``` bash
kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
```

``` bash
kubectl get secrets
```

#### Decoding a Secret
``` bash
echo 'MWYyZDFlMmU2N2Rm' | base64 --decode
1f2d1e2e67df
```

#### Secrets from environment variables.
https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables
``` bash
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
```


#### Secrets from a volume.
``` bash
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```

``` bash
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
```

## Scheduled Job
https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/

#### Create with yaml (Basic)
``` bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

#### Create  direct (Basic)
``` bash
 kubectl run hello --schedule="*/1 * * * *" --restart=OnFailure --image=busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster"
```


#### Create with yaml (Advanced //)
``` bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  parallelism: 5
  completions: 10
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```


#### Commands
``` bash
kubectl get cronjob
kubectl get jobs --watch
kubectl get cronjob hello
kubectl delete cronjob hello
```

## Create a service
https://kubernetes.io/docs/concepts/services-networking/service/
https://kubernetes.io/docs/tasks/access-application-cluster/service-access-application-cluster/
https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0

#### Basic example
Run a Hello World application in your cluster:
``` bash
kubectl run hello-world --replicas=2 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0  --port=8080
```

Display information about the Deployment:
``` bash
kubectl get deployments hello-world
kubectl describe deployments hello-world
```

Display information about your ReplicaSet objects:
``` bash
kubectl get replicasets
kubectl describe replicasets
```

Create a Service object that exposes the deployment:
``` bash
kubectl expose deployment hello-world --type=NodePort --name=example-service
```

Display information about the Service:
``` bash
kubectl describe services example-service
```

List the pods that are running the Hello World application:
kubectl get pods --selector="run=load-balancer-example" --output=wide


#### Expose Node Port
``` bash
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  selector:
    app: my-app
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30036
    protocol: TCP
```

#### Expose Cluster
``` bash
apiVersion: v1
kind: Service
metadata:
  name: my-internal-service
spec:
  selector:
    app: my-app
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
```


#### Expose LB service
``` bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-ingress
spec:
  backend:
    serviceName: other
    servicePort: 8080
  rules:
  - host: foo.mydomain.com
    http:
      paths:
      - backend:
          serviceName: foo
          servicePort: 8080
  - host: mydomain.com
    http:
      paths:
      - path: /bar/*
        backend:
          serviceName: bar
          servicePort: 8080

```


``` bash
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
  clusterIP: 10.0.171.239
  loadBalancerIP: 78.11.24.19
  type: LoadBalancer
```
#### Change Kind
``` bash
kubectl expose deployment nginx --type ClusterIP
```

## Autoscaling
https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

``` bash
kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m --expose --port=80
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

## Limiting ressource
https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/#before-you-begin
https://kubernetes.io/docs/concepts/policy/resource-quotas/

#### NameSpace
``` bash
kubectl create namespace quota-mem-cpu-example
```


``` bash
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    persistentvolumeclaims: "4"
```

``` bash
kubectl create -f https://k8s.io/examples/admin/resource/quota-mem-cpu.yaml --namespace=quota-mem-cpu-example
```

``` bash
kubectl get resourcequota mem-cpu-demo --namespace=quota-mem-cpu-example --output=yaml
```


#### Direct POD
``` bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.12.2
        ports:
        - containerPort: 80
      resources:
          limits:
            cpu: "2"
            memory: "100Mi"
          requests:
            cpu: "500m"

```




## Init container
``` bash
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```


## Security
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

#### With an user ID
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
``` bash
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 1000
  containers:
  - name: nginx
    image: nginx:1.12.2
    ports:
    - containerPort: 80
    securityContext:
      allowPrivilegeEscalation: false
```

#### Networking policy

``` bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: nginx
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```


#### Certificates
https://kubernetes.io/docs/tasks/tls/certificate-rotation/
https://sysdig.com/blog/kubernetes-security-rbac-tls/

``` bash
kubectl get csr
```

## Ingress Rules

#### Basic Example:
https://kubernetes.io/docs/concepts/services-networking/ingress/

``` bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: s1
          servicePort: 80
      - path: /bar
        backend:
          serviceName: s2
          servicePort: 80
```