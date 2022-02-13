### Homework 4
## Create users deploy_view and deploy_edit. Give the user deploy_view rights only to view deployments, pods. Give the user deploy_edit full rights to the objects deployments, pods.

### Configuring certs and minikube for user deploy_view like in task 4 

### Trying get pods
```bash
kubectl config use-context deploy_view
Switched to context "deploy_view".
andry@Andry-PC:~/devops-school-k8s/task4/homework/certs$ kubectl get pods   
Error from server (Forbidden): pods is forbidden: User "deploy_view" cannot list resource "pods" in API group "" in the namespace "default"
```
### Apply role to user and check again
```bash
andry@Andry-PC:~/devops-school-k8s/task4/homework/certs$ kubectl config use-context minikube
Switched to context "minikube".
andry@Andry-PC:~/devops-school-k8s/task4/homework$ cat deploy_view.yaml 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deploy-view-role
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deploy_view
subjects:
- kind: User
  name: deploy_view
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: deploy-view-role
  apiGroup: rbac.authorization.k8s.io
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl apply -f deploy_view.yaml 
role.rbac.authorization.k8s.io/deploy-view-role created
rolebinding.rbac.authorization.k8s.io/deploy_view created
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl config use-context deploy_view
Switched to context "deploy_view".
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl get deployment
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
minio          1/1     1            1           5h45m
web            3/3     3            3           4h19m
web-emptydir   1/1     1            1           179m
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
minio-94fd47554-h96tt           1/1     Running   0          5h45m
minio-state-0                   1/1     Running   0          5h43m
nginx                           1/1     Running   1          17h
web-6745ffd5c8-csm5v            1/1     Running   0          4h19m
web-6745ffd5c8-gbcxv            1/1     Running   0          4h19m
web-6745ffd5c8-vxqqj            1/1     Running   0          4h19m
web-emptydir-59ff4c7f8f-b6qbl   1/1     Running   0          177m
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl delete pod nginx
Error from server (Forbidden): pods "nginx" is forbidden: User "deploy_view" cannot delete resource "pods" in API group "" in the namespace "default"
```
Everything is ok. We can get pods, but not delete.

### Do same thing to deploy_edit user.
```bash
andry@Andry-PC:~/devops-school-k8s/task4/homework$ cat deploy_edit.yaml 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deploy-edit-role
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["*"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deploy_edit
subjects:
- kind: User
  name: deploy_edit
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: deploy-edit-role
  apiGroup: rbac.authorization.k8s.io
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl config use-context minikube
Switched to context "minikube".
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl apply -f deploy_edit.yaml 
role.rbac.authorization.k8s.io/deploy-edit-role created
rolebinding.rbac.authorization.k8s.io/deploy_edit created
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl config use-context deploy_edit
Switched to context "deploy_edit".
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
minio-94fd47554-h96tt           1/1     Running   0          6h8m
minio-state-0                   1/1     Running   0          6h7m
nginx                           1/1     Running   1          17h
web-6745ffd5c8-csm5v            1/1     Running   0          4h43m
web-6745ffd5c8-gbcxv            1/1     Running   0          4h43m
web-6745ffd5c8-vxqqj            1/1     Running   0          4h43m
web-emptydir-59ff4c7f8f-b6qbl   1/1     Running   0          3h21m
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl delete pod nginx
pod "nginx" deleted
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl get service
Error from server (Forbidden): services is forbidden: User "deploy_edit" cannot list resource "services" in API group "" in the namespace "default"
```
## Create namespace prod. Create users prod_admin, prod_view. Give the user prod_admin admin rights on ns prod, give the user prod_view only view rights on namespace prod.

## Create namespace
```bash
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl create namespace prod
namespace/prod created
```
## Create user prod_admin like other user, but with little changes in context
```bash
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl config set-context prod_admin --cluster=minikube --user=prod_admin --namespace=prod
Context "prod_admin" created.
andry@Andry-PC:~/devops-school-k8s/task4/homework$ cat binding_admin.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prod_admin
  namespace: prod
subjects:
- kind: User
  name: prod_admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: admin
  apiGroup: rbac.authorization.k8s.io
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl apply -f binding_admin.yaml 
rolebinding.rbac.authorization.k8s.io/prod_admin created
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
minio-94fd47554-h96tt           1/1     Running   0          6h23m
minio-state-0                   1/1     Running   0          6h21m
web-6745ffd5c8-csm5v            1/1     Running   0          4h57m
web-6745ffd5c8-gbcxv            1/1     Running   0          4h57m
web-6745ffd5c8-vxqqj            1/1     Running   0          4h57m
web-emptydir-59ff4c7f8f-b6qbl   1/1     Running   0          3h35m
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl config use-context prod_admin                                                                     Switched to context "prod_admin".
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl get pod                
No resources found in prod namespace.
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl apply -f pod.yaml 
pod/nginx created
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl get pod
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          2s
```
### Create prod_view and check rights
```bash
andry@Andry-PC:~/devops-school-k8s/task4/homework$ cat binding_view.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prod_view
  namespace: prod
subjects:
- kind: User
  name: prod_view
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl apply -f binding_view.yaml 
rolebinding.rbac.authorization.k8s.io/prod_view created
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl config use-context prod_view
Switched to context "prod_view".
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl get pods     
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          15m
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl delete pod nginx
Error from server (Forbidden): pods "nginx" is forbidden: User "prod_view" cannot delete resource "pods" in API group "" in the namespace "prod"
```
## Create a serviceAccount sa-namespace-admin. Grant full rights to namespace default. Create context, authorize using the created sa, check accesses.

### Create sa-namespace-admin via file sa-admin_create.yaml 
```bash
andry@Andry-PC:~/devops-school-k8s/task4/homework$ cat sa-admin_create.yaml 
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-namespace-admin
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-namespace-admin
  namespace: default
subjects:
- kind: ServiceAccount
  name: sa-namespace-admin
  namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io 
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl apply -f sa-admin_create.yaml 
serviceaccount/sa-namespace-admin created
rolebinding.rbac.authorization.k8s.io/sa-namespace-admin created
```
###
```bash
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl apply -f pod_sa.yaml 
pod/nginx-sa created
andry@Andry-PC:~/devops-school-k8s/task4/homework$ kubectl describe pod nginx-sa
Name:         nginx-sa
Namespace:    default
Priority:     0
Node:         minikube/192.168.59.100
Start Time:   Sun, 13 Feb 2022 19:52:35 +0300
Labels:       run=nginx-sa
Annotations:  <none>
Status:       Running
IP:           172.17.0.14
IPs:
  IP:  172.17.0.14
...
...
...
spec:
  containers:
  - image: nginx:latest
    imagePullPolicy: Always
    name: nginx-sa
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-8d7xw
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: sa-namespace-admin
  serviceAccountName: sa-namespace-admin
  terminationGracePeriodSeconds: 30
   ...
   ...
   ...
  startTime: "2022-02-13T16:52:35Z"
```
serviceAccount: sa-namespace-admin  serviceAccountName: sa-namespace-admin

This fields tells us that the sa-namespace-admin is serviceAccount for nginx-sa.

