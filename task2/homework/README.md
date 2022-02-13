# Homework 2
# Task 1
In Minikube in namespace kube-system, there are many different pods running. Your task is to figure out who creates them, and who makes sure they are running (restores them after deletion).
# Search pod and kill them
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl get pod --namespace kube-system                                                                            
NAME                               READY   STATUS    RESTARTS        AGE
coredns-64897985d-cwgnl            1/1     Running   2               29h
etcd-minikube                      1/1     Running   2               29h
kube-apiserver-minikube            1/1     Running   2               36h
kube-controller-manager-minikube   1/1     Running   2               36h
kube-proxy-vvs7h                   1/1     Running   2               36h
kube-scheduler-minikube            1/1     Running   2               36h
metrics-server-864fbcfc8f-bfdlq    1/1     Running   2               35h
storage-provisioner                1/1     Running   5 (4h35m ago)   36h
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl delete pod etcd-minikube --namespace kube-system                                          
pod "etcd-minikube" deleted
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl get pod --namespace kube-system                 
NAME                               READY   STATUS    RESTARTS        AGE
coredns-64897985d-cwgnl            1/1     Running   2               29h
etcd-minikube                      1/1     Running   2               5s
kube-apiserver-minikube            1/1     Running   2               36h
kube-controller-manager-minikube   1/1     Running   2               36h
kube-proxy-vvs7h                   1/1     Running   2               36h
kube-scheduler-minikube            1/1     Running   2               36h
metrics-server-864fbcfc8f-bfdlq    1/1     Running   2               35h
storage-provisioner                1/1     Running   5 (4h35m ago)   36h
# Search a controller of the pod
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl describe pod etcd-minikube --namespace kube-system     
Name:                 etcd-minikube
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 minikube/192.168.59.100
Start Time:           Fri, 11 Feb 2022 13:40:38 +0300
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.59.100:2379
                      kubernetes.io/config.hash: fc45a20ce68c7f73e419c622ef6d0434
                      kubernetes.io/config.mirror: fc45a20ce68c7f73e419c622ef6d0434
                      kubernetes.io/config.seen: 2022-02-12T19:01:38.669859464Z
                      kubernetes.io/config.source: file
                      seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:               Running
IP:                   192.168.59.100
IPs:
  IP:           192.168.59.100
Controlled By:  Node/minikube
Containers:
  etcd:
    Container ID:  docker://a68384e393a691a3bd040d9bd63f17b84d7273b35d20b25a6fa8306714b49a23
    Image:         k8s.gcr.io/etcd:3.5.1-0
    Image ID:      docker-pullable://k8s.gcr.io/etcd@sha256:64b9ea357325d5db9f8a723dcf503b5a449177b17ac87d69481e126bb724c263
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://192.168.59.100:2379
      --cert-file=/var/lib/minikube/certs/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/minikube/etcd
      --initial-advertise-peer-urls=https://192.168.59.100:2380
      --initial-cluster=minikube=https://192.168.59.100:2380
      --key-file=/var/lib/minikube/certs/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://192.168.59.100:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://192.168.59.100:2380
      --name=minikube
      --peer-cert-file=/var/lib/minikube/certs/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/var/lib/minikube/certs/etcd/peer.key
      --peer-trusted-ca-file=/var/lib/minikube/certs/etcd/ca.crt
      --proxy-refresh-interval=70000
      --snapshot-count=10000
      --trusted-ca-file=/var/lib/minikube/certs/etcd/ca.crt
    State:          Running
      Started:      Sat, 12 Feb 2022 22:01:29 +0300
    Ready:          True
    Restart Count:  2
    Requests:
      cpu:        100m
      memory:     100Mi
    Liveness:     http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /var/lib/minikube/certs/etcd from etcd-certs (rw)
      /var/lib/minikube/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/minikube/certs/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/minikube/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:            <none>
```
* Pod Controlled By:  Node/minikube
# Task 2
### Deploy first nginx deployment
```bash
andry@Andry-PC:~/devops-school-k8s/task2/homework$ kubectl apply -f nginx1-deployment.yaml 
deployment.apps/nginx1 created
configmap/nginx1-configmap created
service/nginx1-np created
```
## Create Ingress
```bash
andry@Andry-PC:~/devops-school-k8s/task2/homework$ kubectl apply -f ingress.yaml           
ingress.networking.k8s.io/example-ingress created
```
### Check output
```bash
andry@Andry-PC:~/devops-school-k8s/task2/homework$ while true; do curl -s hello-world.info | grep nginx; sleep 1; done
nginx1
nginx1
nginx1
nginx1
nginx1
nginx1
nginx1
nginx1
nginx1
nginx1
```
### Deploy second nginx deployment
```bash
andry@Andry-PC:~/devops-school-k8s/task2/homework$ kubectl apply -f nginx2-deployment.yaml  
deployment.apps/nginx2 created
configmap/nginx2-configmap created
service/nginx2-np created
```
## Create Canary Ingress
```bash
andry@Andry-PC:~/devops-school-k8s/task2/homework$ kubectl apply -f ingress_canary.yaml    
ingress.networking.k8s.io/nginx-canary created
```
### Check output
```bash
andry@Andry-PC:~/devops-school-k8s/task2/homework$ while true; do curl -s hello-world.info | grep nginx; sleep 1; done
nginx1
nginx1
nginx2
nginx1
nginx2
nginx2
nginx1
nginx1
nginx1
nginx1
nginx2
```

Canary deployment is working!