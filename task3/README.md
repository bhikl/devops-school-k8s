# Task 3
### Create pv in kubernetes
```bash
andry@Andry-PC:~/devops-school-k8s/task3$ kubectl apply -f pv.yaml
persistentvolume/minio-deployment-pv created
```
### Check our pv
```bash
andry@Andry-PC:~/devops-school-k8s/task3$ kubectl get pv
```
### Output
```bash
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
minio-deployment-pv   5Gi        RWO            Retain           Available      
```
### Create pvc
```bash
andry@Andry-PC:~/devops-school-k8s/task3$ kubectl apply -f pvc.yaml
persistentvolumeclaim/minio-deployment-claim created
```
### Check our output in pv 
```bash
andry@Andry-PC:~/devops-school-k8s/task3$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                            STORAGECLASS   REASON   AGE
minio-deployment-pv                        5Gi        RWO            Retain           Bound    default/minio-deployment-claim                           13m
pvc-e98ac7ab-a098-467b-8a6c-a132c21fa349   1Gi        RWO            Delete           Bound    default/minio-minio-state-0      standard                8m16s                   94s
```
Output is change. PV get status bound.
### Check pvc
```bash
andry@Andry-PC:~/devops-school-k8s/task3$ kubectl get pvc
NAME                     STATUS   VOLUME                CAPACITY   ACCESS MODES   STORAGECLASS   AGE
minio-deployment-claim   Bound    minio-deployment-pv   5Gi        RWO                           7s
```
### Apply deployment minio
```bash
andry@Andry-PC:~/devops-school-k8s/task3$ kubectl apply -f deployment.yaml
deployment.apps/minio created
```
### Apply svc nodeport
```bash
andry@Andry-PC:~/devops-school-k8s/task3$ kubectl apply -f minio-nodeport.yaml
service/minio-app created
```
### Open minikup_ip:node_port in you browser
```bash
andry@Andry-PC:~/devops-school-k8s/task3$ minikube ip
192.168.59.100
andry@Andry-PC:~/devops-school-k8s/task3$ kubectl get service
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP          18h
minio-app      NodePort    10.107.136.253   <none>        9001:30008/TCP   11s
web            ClusterIP   10.106.248.119   <none>        80/TCP           11h
web-headless   ClusterIP   None             <none>        80/TCP           10h
web-np         NodePort    10.106.187.250   <none>        80:32624/TCP     11h
```
### Apply statefulset
```bash
kubectl apply -f statefulset.yaml
```
### Check pod and statefulset
```bash
kubectl get pod
kubectl get sts
```

### Homework
* We published minio "outside" using nodePort. Do the same but using ingress.
* Publish minio via ingress so that minio by ip_minikube and nginx returning hostname (previous job) by path ip_minikube/web are available at the same time.
* Create deploy with emptyDir save data to mountPoint emptyDir, delete pods, check data.
* Optional. Raise an nfs share on a remote machine. Create a pv using this share, create a pvc for it, create a deployment. Save data to the share, delete the deployment, delete the pv/pvc, check that the data is safe.