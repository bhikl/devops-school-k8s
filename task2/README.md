# Task 2
### ConfigMap & Secrets
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl create secret generic connection-string --from-literal=DATABASE_URL=postgres://connect --dry-run=client -o yaml > secret.yaml
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl create configmap user --from-literal=firstname=firstname --from-literal=lastname=lastname --dry-run=client -o yaml > cm.yaml
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl apply -f secret.yaml
secret/connection-string configured
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl apply -f cm.yaml
configmap/user configured
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl apply -f pod.yaml
pod/nginx created
```
## Check env in pod
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl exec -it nginx -- bash
root@nginx:/# printenv
```
### Output
```bash
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
DATABASE_URL=postgres://connect
HOSTNAME=nginx
PWD=/
PKG_RELEASE=1~bullseye
HOME=/root
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
NJS_VERSION=0.7.2
TERM=xterm
SHLVL=1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
lastname=lastname
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
firstname=firstname
NGINX_VERSION=1.21.6
_=/usr/bin/printenv
root@nginx:/# exit
exit
```
### Create deployment with simple application
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl apply -f nginx-configmap.yaml
configmap/nginx-configmap created
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl apply -f deployment.yaml
deployment.apps/web created
```
### Get pod ip address
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE    IP            NODE       NOMINATED NODE   READINESS GATES
nginx                  1/1     Running   0          2m6s   172.17.0.6    minikube   <none>           <none>
web-6745ffd5c8-8q6qz   1/1     Running   0          17s    172.17.0.8    minikube   <none>           <none>
web-6745ffd5c8-km2nz   1/1     Running   0          17s    172.17.0.9    minikube   <none>           <none>
web-6745ffd5c8-nqqtq   1/1     Running   0          17s    172.17.0.10   minikube   <none>           <none>
```
### From PC
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ curl 172.17.0.8
curl: (7) Failed to connect to 172.17.0.8 port 80: No route to host
```
### From minikube (minikube ssh)
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 172.17.0.8
web-6745ffd5c8-8q6qz
```
### From another pod (kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash)
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl exec -it web-6745ffd5c8-km2nz  bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@web-6745ffd5c8-km2nz:/# curl 172.17.0.8
web-6745ffd5c8-8q6qz
```
### Create service (ClusterIP)
Create a manifest template
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl expose deployment/web --type=ClusterIP --dry-run=client -o yaml > service_template.yaml
```
Apply manifest
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl apply -f service_template.yaml
service/web created
```
Get service CLUSTER-IP
```bash
kubectl get svc
```
### Output
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   6h36m
web          ClusterIP   10.106.248.119   <none>        80/TCP    2m35s

```
### From PC
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ curl 10.106.248.119
curl: (28) Failed to connect to 10.106.248.119 port 80: Connection timed out
```
### From minikube (minikube ssh)
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.106.248.119
web-6745ffd5c8-nqqtq
$ curl 10.106.248.119
web-6745ffd5c8-km2nz
```
### From another pod (kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash)
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@web-6745ffd5c8-8q6qz:/# curl 10.106.248.119
web-6745ffd5c8-nqqtq
root@web-6745ffd5c8-8q6qz:/# curl 10.106.248.119
web-6745ffd5c8-nqqtq
root@web-6745ffd5c8-8q6qz:/# curl 10.106.248.119
web-6745ffd5c8-nqqtq
root@web-6745ffd5c8-8q6qz:/# curl 10.106.248.119
web-6745ffd5c8-nqqtq
root@web-6745ffd5c8-8q6qz:/# curl 10.106.248.119
web-6745ffd5c8-km2nz
```
### NodePort
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl apply -f service-nodeport.yaml
service/web-np created
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl get service
```
### Output
```bash
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        6h59m
web          ClusterIP   10.106.248.119   <none>        80/TCP         24m
web-np       NodePort    10.106.187.250   <none>        80:32624/TCP   0s
```
Note how port is specified for a NodePort service
### Checking the availability of the NodePort service type
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ minikube ip
192.168.59.100
andry@Andry-PC:~/devops-school-k8s/task2$ curl 192.168.59.100:32624
web-6745ffd5c8-8q6qz
andry@Andry-PC:~/devops-school-k8s/task2$ curl 192.168.59.100:32624
web-6745ffd5c8-8q6qz
andry@Andry-PC:~/devops-school-k8s/task2$ curl 192.168.59.100:32624
web-6745ffd5c8-8q6qz
andry@Andry-PC:~/devops-school-k8s/task2$ curl 192.168.59.100:32624
web-6745ffd5c8-km2nz
andry@Andry-PC:~/devops-school-k8s/task2$ curl 192.168.59.100:32624
web-6745ffd5c8-8q6qz
andry@Andry-PC:~/devops-school-k8s/task2$ curl 192.168.59.100:32624
web-6745ffd5c8-km2nz
andry@Andry-PC:~/devops-school-k8s/task2$ 
```
### Headless service
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl apply -f service-headless.yaml
service/web-headless created
```
### DNS
### resolv.conf
```bash
root@web-6745ffd5c8-8q6qz:/# cat /etc/resolv.conf 
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local 
options ndots:5
```

### Compare headless and clusterip

```bash
root@web-6745ffd5c8-8q6qz:/# cat /etc/resolv.conf 
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local 
options ndots:5
```
### Compare the IP address of the DNS server in the pod and the DNS service of the Kubernetes cluster.
```bash
root@web-6745ffd5c8-8q6qz:/# nslookup 10.106.248.119   
119.248.106.10.in-addr.arpa     name = web.default.svc.cluster.local.

root@web-6745ffd5c8-8q6qz:/# nslookup 10.106.187.250
250.187.106.10.in-addr.arpa     name = web-np.default.svc.cluster.local.

root@web-6745ffd5c8-8q6qz:/# nslookup web-np.default.svc.cluster.local.
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   web-np.default.svc.cluster.local
Address: 10.106.187.250

root@web-6745ffd5c8-8q6qz:/# nslookup web.default.svc.cluster.local.   
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   web.default.svc.cluster.local
Address: 10.106.248.119
```

### [Ingress](https://kubernetes.github.io/ingress-nginx/deploy/#minikube)
Enable Ingress controller
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ minikube addons enable ingress
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.1.0
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```
Let's see what the ingress controller creates for us
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-5bcvk        0/1     Completed   0          29h
ingress-nginx-admission-patch-pgjfw         0/1     Completed   1          29h
ingress-nginx-controller-6d5f55986b-2pr9s   1/1     Running     1          29h
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl get pod $(kubectl get pod -n ingress-nginx|grep ingress-nginx-controller|awk '{print $1}') -n ingress-nginx -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"creationTimestamp":"2022-02-11T17:44:45Z","generateName":"ingress-nginx-controller-6d5f55986b-","labels":{"app.kubernetes.io/component":"controller","app.kubernetes.io/instance":"ingress-nginx","app.kubernetes.io/name":"ingress-nginx","gcp-auth-skip-secret":"true","pod-template-hash":"6d5f55986b"},"name":"ingress-nginx-controller-6d5f55986b-2pr9s","namespace":"ingress-nginx","ownerReferences":[{"apiVersion":"apps/v1","blockOwnerDeletion":true,"controller":true,"kind":"ReplicaSet","name":"ingress-nginx-controller-6d5f55986b","uid":"aa5cc126-2a31-4961-ac4e-ee37e75de9ce"}],"resourceVersion":"19059","uid":"530e7ad3-9de1-4b12-8895-3bcf6c41cc4b"},"spec":{"containers":[{"args":["/nginx-ingress-controller","--election-id=ingress-controller-leader","--controller-class=k8s.io/ingress-nginx","--watch-ingress-without-class=true","--publish-status-address=localhost","--configmap=$(POD_NAMESPACE)/ingress-nginx-controller","--report-node-internal-ip-address","--tcp-services-configmap=$(POD_NAMESPACE)/tcp-services","--udp-services-configmap=$(POD_NAMESPACE)/udp-services","--validating-webhook=:8443","--validating-webhook-certificate=/usr/local/certificates/cert","--validating-webhook-key=/usr/local/certificates/key"],"env":[{"name":"POD_NAME","valueFrom":{"fieldRef":{"apiVersion":"v1","fieldPath":"metadata.name"}}},{"name":"POD_NAMESPACE","valueFrom":{"fieldRef":{"apiVersion":"v1","fieldPath":"metadata.namespace"}}},{"name":"LD_PRELOAD","value":"/usr/local/lib/libmimalloc.so"}],"image":"k8s.gcr.io/ingress-nginx/controller:v1.1.0@sha256:f766669fdcf3dc26347ed273a55e754b427eb4411ee075a53f30718b4499076a","imagePullPolicy":"IfNotPresent","lifecycle":{"preStop":{"exec":{"command":["/wait-shutdown"]}}},"livenessProbe":{"failureThreshold":5,"httpGet":{"path":"/healthz","port":10254,"scheme":"HTTP"},"initialDelaySeconds":10,"periodSeconds":10,"successThreshold":1,"timeoutSeconds":1},"name":"controller","ports":[{"containerPort":80,"hostPort":80,"name":"http","protocol":"TCP"},{"containerPort":443,"hostPort":443,"name":"https","protocol":"TCP"},{"containerPort":8443,"name":"webhook","protocol":"TCP"}],"readinessProbe":{"failureThreshold":3,"httpGet":{"path":"/healthz","port":10254,"scheme":"HTTP"},"initialDelaySeconds":10,"periodSeconds":10,"successThreshold":1,"timeoutSeconds":1},"resources":{"requests":{"cpu":"100m","memory":"90Mi"}},"securityContext":{"allowPrivilegeEscalation":true,"capabilities":{"add":["NET_BIND_SERVICE"],"drop":["ALL"]},"runAsUser":101},"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File","volumeMounts":[{"mountPath":"/usr/local/certificates/","name":"webhook-cert","readOnly":true},{"mountPath":"/var/run/secrets/kubernetes.io/serviceaccount","name":"kube-api-access-pp6js","readOnly":true}]}],"dnsPolicy":"ClusterFirst","enableServiceLinks":true,"nodeName":"minikube","preemptionPolicy":"PreemptLowerPriority","priority":0,"restartPolicy":"Always","schedulerName":"default-scheduler","securityContext":{},"serviceAccount":"ingress-nginx","serviceAccountName":"ingress-nginx","terminationGracePeriodSeconds":30,"tolerations":[{"effect":"NoExecute","key":"node.kubernetes.io/not-ready","operator":"Exists","tolerationSeconds":300},{"effect":"NoExecute","key":"node.kubernetes.io/unreachable","operator":"Exists","tolerationSeconds":300}],"volumes":[{"name":"webhook-cert","secret":{"defaultMode":420,"secretName":"ingress-nginx-admission"}},{"name":"kube-api-access-pp6js","projected":{"defaultMode":420,"sources":[{"serviceAccountToken":{"expirationSeconds":3607,"path":"token"}},{"configMap":{"items":[{"key":"ca.crt","path":"ca.crt"}],"name":"kube-root-ca.crt"}},{"downwardAPI":{"items":[{"fieldRef":{"apiVersion":"v1","fieldPath":"metadata.namespace"},"path":"namespace"}]}}]}}]},"status":{"conditions":[{"lastProbeTime":null,"lastTransitionTime":"2022-02-11T17:44:45Z","status":"True","type":"Initialized"},{"lastProbeTime":null,"lastTransitionTime":"2022-02-11T17:45:25Z","status":"True","type":"Ready"},{"lastProbeTime":null,"lastTransitionTime":"2022-02-11T17:45:25Z","status":"True","type":"ContainersReady"},{"lastProbeTime":null,"lastTransitionTime":"2022-02-11T17:44:45Z","status":"True","type":"PodScheduled"}],"containerStatuses":[{"containerID":"docker://17d24c366dcba5000407a01c7f44814c06b2688df079ba1efe7999789dc9fab5","image":"k8s.gcr.io/ingress-nginx/controller@sha256:f766669fdcf3dc26347ed273a55e754b427eb4411ee075a53f30718b4499076a","imageID":"docker-pullable://k8s.gcr.io/ingress-nginx/controller@sha256:f766669fdcf3dc26347ed273a55e754b427eb4411ee075a53f30718b4499076a","lastState":{},"name":"controller","ready":true,"restartCount":0,"started":true,"state":{"running":{"startedAt":"2022-02-11T17:45:10Z"}}}],"hostIP":"192.168.59.100","phase":"Running","podIP":"172.17.0.11","podIPs":[{"ip":"172.17.0.11"}],"qosClass":"Burstable","startTime":"2022-02-11T17:44:45Z"}}
  creationTimestamp: "2022-02-11T17:44:45Z"
  generateName: ingress-nginx-controller-6d5f55986b-
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    gcp-auth-skip-secret: "true"
    pod-template-hash: 6d5f55986b
  name: ingress-nginx-controller-6d5f55986b-2pr9s
  namespace: ingress-nginx
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: ingress-nginx-controller-6d5f55986b
    uid: aa5cc126-2a31-4961-ac4e-ee37e75de9ce
  resourceVersion: "46650"
  uid: 530e7ad3-9de1-4b12-8895-3bcf6c41cc4b
spec:
  containers:
  - args:
    - /nginx-ingress-controller
    - --election-id=ingress-controller-leader
    - --controller-class=k8s.io/ingress-nginx
    - --watch-ingress-without-class=true
    - --publish-status-address=localhost
    - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
    - --report-node-internal-ip-address
    - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
    - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
    - --validating-webhook=:8443
    - --validating-webhook-certificate=/usr/local/certificates/cert
    - --validating-webhook-key=/usr/local/certificates/key
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.namespace
    - name: LD_PRELOAD
      value: /usr/local/lib/libmimalloc.so
    image: k8s.gcr.io/ingress-nginx/controller:v1.1.0@sha256:f766669fdcf3dc26347ed273a55e754b427eb4411ee075a53f30718b4499076a
    imagePullPolicy: IfNotPresent
    lifecycle:
      preStop:
        exec:
          command:
          - /wait-shutdown
    livenessProbe:
      failureThreshold: 5
      httpGet:
        path: /healthz
        port: 10254
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    name: controller
    ports:
    - containerPort: 80
      hostPort: 80
      name: http
      protocol: TCP
    - containerPort: 443
      hostPort: 443
      name: https
      protocol: TCP
    - containerPort: 8443
      name: webhook
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        path: /healthz
        port: 10254
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    resources:
      requests:
        cpu: 100m
        memory: 90Mi
    securityContext:
      allowPrivilegeEscalation: true
      capabilities:
        add:
        - NET_BIND_SERVICE
        drop:
        - ALL
      runAsUser: 101
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /usr/local/certificates/
      name: webhook-cert
      readOnly: true
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-pp6js
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: ingress-nginx
  serviceAccountName: ingress-nginx
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
  - name: webhook-cert
    secret:
      defaultMode: 420
      secretName: ingress-nginx-admission
  - name: kube-api-access-pp6js
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
    lastTransitionTime: "2022-02-11T17:44:45Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2022-02-12T19:01:54Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2022-02-12T19:01:54Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2022-02-11T17:44:45Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://b66660c65c0d365d11b49112bbef8276bd68a3abd1e1f49ade70da64e6b9268a
    image: sha256:ae1a7201ec9545194b2889da30face5f2a7a45e2ba8c7479ac68c9a45a73a7eb
    imageID: docker-pullable://k8s.gcr.io/ingress-nginx/controller@sha256:f766669fdcf3dc26347ed273a55e754b427eb4411ee075a53f30718b4499076a
    lastState: {}
    name: controller
    ready: true
    restartCount: 1
    started: true
    state:
      running:
        startedAt: "2022-02-12T19:01:42Z"
  hostIP: 192.168.59.100
  phase: Running
  podIP: 172.17.0.3
  podIPs:
  - ip: 172.17.0.3
  qosClass: Burstable
  startTime: "2022-02-11T17:44:45Z"

```
Create Ingress
```bash
andry@Andry-PC:~/devops-school-k8s/task2$ kubectl apply -f ingress.yaml
ingress.networking.k8s.io/ingress-web created

andry@Andry-PC:~/devops-school-k8s/task2$ curl $(minikube ip)
web-6745ffd5c8-nqqtq
```