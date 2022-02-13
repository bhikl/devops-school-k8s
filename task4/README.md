# Task 4
### Check what I can do
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ kubectl auth can-i create deployments --namespace kube-system
yes
```
### Configure user authentication using x509 certificates
### Create private key
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ openssl genrsa -out k8s_user.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
...........................................................+++++
.......................................+++++
e is 65537 (0x010001)
```
### Create a certificate signing request
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ openssl req -new -key k8s_user.key \
> -out k8s_user.csr \
> -subj "/CN=k8s_user"
```
### Sign the CSR in the Kubernetes CA. We have to use the CA certificate and the key, which are usually in /etc/kubernetes/pki. But since we use minikube, the certificates will be on the host machine in ~/.minikube
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ openssl x509 -req -in k8s_user.csr \
> -CA ~/.minikube/ca.crt \
> -CAkey ~/.minikube/ca.key \
> -CAcreateserial \
> -out k8s_user.crt -days 500
Signature ok
subject=CN = k8s_user
Getting CA Private Key
```
### Create user in kubernetes
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ kubectl config set-credentials k8s_user \
> --client-certificate=k8s_user.crt \
> --client-key=k8s_user.key
User "k8s_user" set.
```
### Set context for user
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ kubectl config set-context k8s_user \
> --cluster=minikube --user=k8s_user
Context "k8s_user" created.
```
### check ~/.kube/config
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ cat ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/andry/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Sun, 13 Feb 2022 12:32:57 MSK
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: cluster_info
    server: https://192.168.59.100:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: k8s_user
  name: k8s_user
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Sun, 13 Feb 2022 12:32:57 MSK
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: k8s_user
kind: Config
preferences: {}
users:
- name: k8s_user
  user:
    client-certificate: /home/andry/devops-school-k8s/task4/k8s_user.crt
    client-key: /home/andry/devops-school-k8s/task4/k8s_user.key
- name: minikube
  user:
    client-certificate: /home/andry/.minikube/profiles/minikube/client.crt
    client-key: /home/andry/.minikube/profiles/minikube/client.key
```
### Switch to use new context
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ kubectl config use-context k8s_user
Switched to context "k8s_user".
```
### Check privileges
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ kubectl get node
Error from server (Forbidden): nodes is forbidden: User "k8s_user" cannot list resource "nodes" in API group "" at the cluster scope
andry@Andry-PC:~/devops-school-k8s/task4$ kubectl get pod
Error from server (Forbidden): pods is forbidden: User "k8s_user" cannot list resource "pods" in API group "" in the namespace "default"
```
### Switch to default(admin) context
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ kubectl config use-context minikube
Switched to context "minikube".
```
### Bind role and clusterrole to the user
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ kubectl apply -f binding.yaml
rolebinding.rbac.authorization.k8s.io/k8s_user created
```
### Switch to k8s_user
```bash
andry@Andry-PC:~/devops-school-k8s/task4$ kubectl config use-context k8s_user
Switched to context "k8s_user".
```
### Check output
```bash
ndry@Andry-PC:~/devops-school-k8s/task4$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
minio-94fd47554-h96tt           1/1     Running   0          3h50m
minio-state-0                   1/1     Running   0          3h48m
nginx                           1/1     Running   1          15h
web-6745ffd5c8-csm5v            1/1     Running   0          144m
web-6745ffd5c8-gbcxv            1/1     Running   0          144m
web-6745ffd5c8-vxqqj            1/1     Running   0          144m
web-emptydir-59ff4c7f8f-b6qbl   1/1     Running   0          62m
```
Now we can see pods.


### Homework
* Create users deploy_view and deploy_edit. Give the user deploy_view rights only to view deployments, pods. Give the user deploy_edit full rights to the objects deployments, pods.
* Create namespace prod. Create users prod_admin, prod_view. Give the user prod_admin admin rights on ns prod, give the user prod_view only view rights on namespace prod.
* Create a serviceAccount sa-namespace-admin. Grant full rights to namespace default. Create context, authorize using the created sa, check accesses.