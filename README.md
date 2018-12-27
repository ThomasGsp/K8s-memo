
# Kubernetes synthetic command list (Oriented day-to-day work)
## CTRL + F is your friend

## Ressources links
[kubernetes.io/kubectl](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

[kubernetes.io/cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

[kubernetes.io](https://kubernetes.io/docs/)

[kubernetesbyexample](http://kubernetesbyexample.com/)

[metal-k8s.readthedocs.io](https://metal-k8s.readthedocs.io/en/latest/installation-guide/quickstart.html)

[www.katacoda.com](https://www.katacoda.com/courses/kubernetes)

[blog.csnet.me](https://blog.csnet.me/2018/04/on-prem-k8s-thw/)

[kubespray](https://github.com/kubernetes-sigs/kubespray)


## Commands list
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
https://kubernetes.io/docs/reference/kubectl/cheatsheet/
``` bash

kubectl create namespace mem-example
kubectl create ns linuxcon
kubectl create secret generic mysql --from-literal=password=root

kubectl --v=9 get pods

kubectl get all
kubectl get endpoints
kubectl get pvc
kubectl get secrets
kubectl get secrets --all-namespaces
kubectl get configmap colors
kubectl get nodes --show-labels
kubectl get pods -L system
kubectl get pods -n accounting
kubectl get pods --show-labels
kubectl get deployment,rs,pods -o json
kubectl get cronjob
kubectl get cronjob date
kubectl get jobs --watch
kubectl get LimitRange
kubectl get LimitRange --all-namespaces

kubectl rollout status
kubectl rollout pause deployment/ghost
kubectl rollout resume deployment/ghost
kubectl rollout history ds ds-one --revision=2

kubectl exec busybox cat /etc/resolv.conf
kubectl exec -it security-context-demo -- sh
kubectl exec -it shell-demo -- /bin/bash -c 'echo $ilike'

kubectl run -i -t busybox --image=busybox --restart=Never
kubectl run nginx --image=nginx --dry-run
kubectl run nginx --image=nginx --replicas=5
kubectl run pi --schedule="0/5 * * * ?" --image=perl --restart=OnFailure -- perl -Mbignum=bpi -wle 'print bpi(2000)'


kubectl proxy --api-prefix=/
kubectl logs date-1539006240-dhb78

kubectl edit deployments dev-web

kubectl label pods ghost-864655d77f-rnddk foo=bar

kubectl delete rs rs-one --cascade=false
kubectl delete cronjob date

kubectl set image ds ds-one nginx=nginx:1.8.1-alpine

kubectl describe secrets default-token-6chzc
kubectl describe rs rs-one
kubectl describe ingress test

kubectl -n kube-system get secrets certificate-controller-token-j9psf  -o yaml
kubectl config set-credentials -h
kubectl config use-context kubernetes
kubectl config view

kubectl top
kubectl get pv --sort-by=

kubeadm token -h
kubeadm token list
kubeadm config -h
```

```bash
kubectl run hello-world --replicas=2 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0  --port=8080
kubectl get deployment mydeployment -o yaml --export > myappdep.yaml
kubectl run myapp --image=me/myapp:v1  -o yaml --dry-run > myapp.yaml
```

## Delete all pods
``` bash
kubectl get pods | awk -F' ' '{print $1}' | xargs  kubectl delete pods
```


## Install ssl tools
``` bash
curl -o cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
curl -o cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x cfssl cfssljson
mv cfssl cfssljson /usr/local/bin/
```

## Using Zsh/bash completion

``` bash
source <(kubectl completion bash)
```

``` bash
if [ $commands[kubectl] ]; then
  source <(kubectl completion zsh)
fi
```

or

``` bash
# Oh-My-Zsh
apt install zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

vi ~/.zshrc

plugins=(kubectl)
ZSH_THEME="pygmalion"

source ~/.zshrc
```



## Kubectl from bin
``` bash
## Download
wget https://storage.googleapis.com/kubernetes-release/release/v1.11.0/bin/linux/amd64/kubectl
mv kubectl /usr/local/bin/kubectl
chmod +x /usr/local/bin/kubectl
/usr/local/bin/kubectl version
```

## Minikube from bin
``` bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.29.0/minikube-linux-amd64
chmod +x minikube
mv minikube /usr/local/bin/
minikube
```

## Dashboard
``` bash
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
kubectl proxy
https://<master-ip>:<apiserver-port>/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

### Auth with token
``` bash
# (we assume default)
kubectl create serviceaccount my-dashboard-sa

kubectl create clusterrolebinding my-dashboard-sa \
  --clusterrole=cluster-admin \
  --serviceaccount=default:my-dashboard-sa

kubectl get secrets
kubectl describe secret my-dashboard-sa-token-xxxxx
```

### Auth with admin.conf
``` bash
# TO DO
```


## ETCD
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
https://www.mirantis.com/blog/everything-you-ever-wanted-to-know-about-using-etcd-with-kubernetes-v1-6-but-were-afraid-to-ask/
https://coreos.com/etcd/docs/latest/v2/admin_guide.html

### etcd Backup
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
``` bash

    etcdctl backup \
      --data-dir %data_dir% \
      [--wal-dir %wal_dir%] \
      --backup-dir %backup_data_dir%
      [--backup-wal-dir %backup_wal_dir%]
```

``` bash
mkdir /backuptest
etcdctl backup --data-dir /var/lib/etcd/ --backup-dir /backuptest
```


## Basic setting up with kubeadm
https://v1-11.docs.kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

### Docker install (Master + worker) -- Debian
https://docs.docker.com/install/linux/docker-ce/debian/#set-up-the-repository
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

### Kubernetes sources (Master + worker)
``` bash
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
```


### Kubernetes Master
``` bash
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

kubeadm init  --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Load k8s env
``` bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

### Calico network (play on Master)
https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network
``` bash
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

### Kubernetes Worker
``` bash
apt-get install -y kubelet
apt-mark hold kubelet
```


### Join your Worker nodes
``` bash
kubeadm join 172.16.0.111:6443 --token pnaven.wxx..nu10c --discovery-token-ca-cert-hash sha256:3e....4ed39f9b9ede
```

## Cluster auditing && logs
https://kubernetes.io/docs/tasks/debug-application-cluster/audit/

### Testing
``` bash
kubectl cluster-info
kubectl get nodes
kubectl get componentstatuses
kubectl get pods -o wide --show-labels --all-namespaces
kubectl get svc  -o wide --show-labels --all-namespaces
```

## Basic deployment / upgrade / rollback / checks

### Nginx version 1.7.9 in 2 pods
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

### Create without using a manifest (Busybox / nginx) -- Option 2
``` bash
kubectl run nginx-deployment --replicas=2 --labels="app=nginx" --image=nginx:1.12.2  --port=80
kubectl run busybox --image=busybox:1.28.4 --command -- sleep 3600
```

### Scale 4 pods.
``` bash
kubectl scale deployment nginx-deployment --replicas=4
```

### DownScale 2 pods.
``` bash
kubectl scale deployment nginx-deployment --replicas=2
```

### Upgrade nginx
``` bash
kubectl edit deployment nginx-deployment
# + Change nginx version
```

### Check the status of the upgrade
``` bash
kubectl get pods -l app=nginx
```

### Rollout history for an deployment
``` bash
kubectl rollout history deployment/nginx-deployment
```

### Undo the upgrade
``` bash
kubectl rollout undo deployment/nginx-deployment
```


### Hot edit
``` bash
kubectl get pods |grep busy
kubectl edit pod busybox-c8d74cd49-64fzq
```



## Liveness and Readiness Probes
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/

### liveness
The kubelet uses liveness probes to know when to restart a Container. For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a Container in such a state can help to make the application more available despite bugs.

### readiness
The kubelet uses readiness probes to know when a Container is ready to start accepting traffic. A Pod is considered ready when all of its Containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.

### Examples

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
### Create
``` bash
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
   name: ds-one
spec:
   template:
      metadata:
         labels:
            system: ReplicaOne
      spec:
         containers:
         - name: nginx
           image: nginx:1.7.9
           ports:
           - containerPort: 80
```

### Change strategy for RollingUpdate
``` bash
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
   name: ds-one
spec:

  minReadySeconds: 0
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: RollingUpdate
    
   template:
      metadata:
         labels:
            system: ReplicaOne
      spec:
         containers:
         - name: nginx
           image: nginx:1.7.9
           ports:
           - containerPort: 80

```

## ReplicaSets
``` bash
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
   name: rs-one
spec:
   replicas: 2
   template:
      metadata:
         labels:
            system: ReplicaOne
      spec:
         containers:
         - name: nginx
           image: nginx:1.7.9
           ports:
           - containerPort: 80
```

## Static Pods
https://kubernetes.io/docs/tasks/administer-cluster/static-pod/

/etc/kubernetes/kubelet.env or  /etc/kubelet.d/

``` bash
--pod-manifest-path=/etc/kubelet.d/
```

``` bash
cat <<EOF >/etc/kubelet.d/static-web.yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
EOF
```

## Secrets
https://kubernetes.io/docs/concepts/configuration/secret/

### Create secret
``` bash
# Create files needed for rest of example.
echo -n 'admin' > ./username
echo -n '1f2d1e2e67df' > ./password
```

``` bash
kubectl create secret generic mysecret --from-file=./username --from-file=./password
```

OR

``` bash
echo -n 'admin' | base64
YWRtaW4=
echo -n '1f2d1e2e67df' | base64
MWYyZDFlMmU2N2Rm
```

``` bash
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

``` bash
kubectl get secrets
```

### Show a Secret
``` bash
kubectl get secret mysecret -o yaml
```

### Decoding a Secret
``` bash
echo 'MWYyZDFlMmU2N2Rm' | base64 --decode
1f2d1e2e67df
```

### Secrets from environment variables.
https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables
``` bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx-user-pass
spec:
  containers:
  - name: mycontainer
    image: nginx
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

### Secrets from a volume.
``` bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx-user-pass
spec:
  containers:
  - name: mypod
    image: nginx
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

## Scheduled / Cron Job
https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/
https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/

### Create a direct Scheduled job (Basic)
``` bash
 kubectl run hello --schedule="*/1 * * * *" --restart=OnFailure --image=busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster"
```

### Create CronJob with yaml (Basic)
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

### Create Job with yaml (Advanced )
``` bash
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  parallelism: 5
  completions: 10
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["/bin/sh",  "-c", "date; echo Hello from the Kubernetes cluster"]
      restartPolicy: Never
```

## Create a service
https://kubernetes.io/docs/concepts/services-networking/service/
https://kubernetes.io/docs/tasks/access-application-cluster/service-access-application-cluster/
https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0

### Basic Run example
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
kubectl expose deployment hello-world --type=ClusterIP --name=example-service
kubectl expose deployment hello-world --type=LoadBalancer --name=example-service
```

Display information about the Service:
``` bash
kubectl describe services example-service
```

List the pods that are running the Hello World application:
kubectl get pods --selector="run=load-balancer-example" --output=wide

### Expose Node Port
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

### Expose Cluster
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

### Expose LB service
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

https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/
``` bash
kubectl expose rc example --port=8765 --target-port=9376 --name=example-service --type=LoadBalancer
```

### Change Kind
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

### NameSpace
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
kubectl get resourcequota --all-namespaces
```

### Direct in POD
``` bash
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container

```

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

```

## Init container
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/

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


``` bash
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  # These containers are run during pod initialization

  initContainers:
  - name: install
    image: busybox
    command:
    - wget
    - "-O"
    - "/work-dir/index.html"
    - http://kubernetes.io
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  dnsPolicy: Default
  volumes:
  - name: workdir
    emptyDir: {}
```

## Security
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

### With an user UID (POD)
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

### POD
``` bash
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-2
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: sec-ctx-demo-2
    image: gcr.io/google-samples/node-hello:1.0
    
```

### POD + Container
``` bash
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-2
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: sec-ctx-demo-2
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false

```

### Test 
``` bash
kubectl exec -it  security-context-demo-2 -- /bin/bash -c "ps  aux"
```

### Networking policy
https://kubernetes.io/docs/concepts/services-networking/network-policies/

https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/

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


### Certificates
https://kubernetes.io/docs/tasks/tls/certificate-rotation/
https://sysdig.com/blog/kubernetes-security-rbac-tls/

``` bash
kubectl get csr
```

### Authentication 
https://kubernetes.io/docs/reference/access-authn-authz/authentication/


## Ingress Rules

### Basic Example (Nginx controler + Backend + ingress service):
https://kubernetes.io/docs/concepts/services-networking/ingress/

#### Deployment + Backend
``` bash
kubectl create deployment app --image=nginx
kubectl expose deployment app --type=NodePort --port=80
```

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
          serviceName: app
          servicePort: 80
      - path: /bar
        backend:
          serviceName: app
          servicePort: 80
```

``` bash
kubectl describe ingress test
kubectl get svc
curl -H "Host: foo.bar.com/foo" http://10.233.35.53/
```


### Advanced Example (deployment + rbac + role + binding Traefik controler + Backend + ingress service):
https://kubernetes.io/docs/concepts/services-networking/ingress/

#### Deployment + Backend
``` bash
kubectl create deployment secondapp --image=nginx
kubectl expose deployment secondapp --type=NodePort --port=80
```

#### rbac
ingress.rbac.yaml
``` bash
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
    - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
```


#### role + binding Traefik controler
traefik-ds.yaml
``` bash
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: DaemonSet
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      hostNetwork: true
      containers:
      - image: traefik
        name: traefik-ingress-lb
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8080
          hostPort: 8080
        securityContext:
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        args:
        - --api
        - --kubernetes
        - --logLevel=INFO
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
    - protocol: TCP
      port: 80
      name: web
    - protocol: TCP
      port: 8080
      name: admin
```

#### ingress service
ingress.rule.yaml
``` bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-test
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: www.example.com
    http:
      paths:
      - backend:
          serviceName: secondapp
          servicePort: 80
        path: /
```

#### Quick test
``` bash
curl -H "Host: www.example.com" http://clusterip/
```

## Logs debug
``` bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/audit/audit-policy.yaml
```

``` bash
find / -name "*apiserver*log"
/var/log/pods/56c55117e68ed986eaddeb0f78ca405e/kube-apiserver_0.log
/var/log/containers/kube-apiserver-lfs458-node-9q6r_kube-system_kube...

locate apiserver
...
```

## Disks / Volumes
https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/

### PersistentVolume
``` bash
# SINGLE NODE ONLY
kind: PersistentVolume
apiVersion: v1
metadata:
   name: pv-volume1
   labels:
      type:local
spec:
   capacity:
      storage: 10Gi
   accessModes:
      - ReadWriteOnce
   hostPath:
      path: "/mnt/localVol"

```

### PersistentVolume NFS
``` bash
# OPTIONAL
apt-get install -y nfs-kernel-server
mkdir /opt/sfw
chmod 1777 /opt/sfw/
echo "/opt/sfw/ *(rw,sync,no_root_squash,subtree_check)" >> /etc/exports
exportfs -ra
```

``` bash
# OPTIONAL
apt-get -y install nfs-common
mount NFS-SERVER-NAME:/opt/sfw /mnt
```

``` bash
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  capacity:
     storage: 1Gi
  accessModes:
     - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /opt/sfw
    server: NFS-SERVER-NAME
    readOnly: false
```


### PersistentVolumeClaim + Pod Volume Claim
``` bash
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
```

``` bash
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

### Pod with emptyDir (Very limited usage)
``` bash
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```


## Helm example with redis
https://github.com/kubernetes/helm/releases
``` bash
curl -LO https://storage.googleapis.com/kubernetes-helm/helm-v2.8.2-linux-amd64.tar.gz
tar -xvf helm-v2.8.2-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/

helm init
helm repo update

helm search redis

helm install stable/redis
helm ls
```

## DNS
``` bash
# install
kubectl apply -f https://raw.githubusercontent.com/mch1307/k8s-thw/master/coredns.yaml
kubectl get pod --all-namespaces -l k8s-app=coredns -o wide

# Test
kubectl run busyboxv1284 --image=busybox:1.28.4 --command -- sleep 3600
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
echo $POD_NAME
kubectl exec -ti $POD_NAME -- nslookup kubernetes
```


## Standalone node kubelet

### pre tasks
``` bash
Install Docker (cf top)
Install kubelet (cf top)
Delete swap if available
Create the following systemd file
```

### Systemd conf
``` bash
/etc/systemd/system/kubelet.service

[Unit]
Description=kubelet

[Service]
Restart=always
ExecStart=/usr/bin/kubelet \
  --allow-privileged=true \
  --enable-server=false \
  --hostname-override=127.0.0.1 \
  --pod-manifest-path=/etc/kubernetes/manifests

EnvironmentFile=-/etc/kubernetes/kubelet.env

[Install]
WantedBy=multi-user.target

```

### Apply changes
``` bash
systemctl daemon-reload
systemctl restart kubelet.service

```

### Check
``` bash
docker images
docker ps
```

## Metric
https://github.com/kubernetes-incubator/metrics-server
``` bash
# INSTALL
git clone https://github.com/kubernetes-incubator/metrics-server.git
cd metrics-server
kubectl create -f deploy/1.8+/

# CHECK
kubectl get --raw /apis/metrics.k8s.io/v1beta1
kubectl get apiservice v1beta1.metrics.k8s.io -o yaml

# USE
kubectl get node <nodename>

```
