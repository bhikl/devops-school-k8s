# Homework 3

## Publish minio via ingress so that minio by ip_minikube and nginx returning hostname (previous job) by path ip_minikube/web are available at the same time.
### Create a web deployment and web nodeport 
```bash
andry@Andry-PC:~/devops-school-k8s/task3/homework$ kubectl apply -f deployment_nginx.yaml 
deployment.apps/web created
andry@Andry-PC:~/devops-school-k8s/task3/homework$ kubectl apply -f service-nodeport_nginx.yaml 
service/web-np created
```
### Make ingress file for web and minio deployment
```bash
andry@Andry-PC:~/devops-school-k8s/task3/homework$ cat ingress.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: hello-world.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: minio-app
                port:
                  number: 9001
          - path: /web
            pathType: Prefix
            backend:
              service:
                name: web-np
                port:
                  number: 80
```
### Create ingress
```bash
andry@Andry-PC:~/devops-school-k8s/task3/homework$ kubectl apply -f ingress.yaml 
ingress.networking.k8s.io/example-ingress created
```
### Check availability
 ```bash
andry@Andry-PC:~/devops-school-k8s/task3/homework$ curl hello-world.info
<!doctype html><html lang="en"><head><meta charset="utf-8"/><base href="/"/><meta content="width=device-width,initial-scale=1" name="viewport"/><meta content="#081C42" media="(prefers-color-scheme: light)" name="theme-color"/><meta content="#081C42" media="(prefers-color-scheme: dark)" name="theme-color"/><meta content="MinIO Console" name="description"/><link href="./styles/root-styles.css" rel="stylesheet"/><link href="./apple-icon-180x180.png" rel="apple-touch-icon" sizes="180x180"/><link href="./favicon-32x32.png" rel="icon" sizes="32x32" type="image/png"/><link href="./favicon-96x96.png" rel="icon" sizes="96x96" type="image/png"/><link href="./favicon-16x16.png" rel="icon" sizes="16x16" type="image/png"/><link href="./manifest.json" rel="manifest"/><link color="#3a4e54" href="./safari-pinned-tab.svg" rel="mask-icon"/><title>MinIO Console</title><script defer="defer" src="./static/js/main.c7ee1a1d.js"></script><link href="./static/css/main.c4c1effe.css" rel="stylesheet"></head><body><noscript>You need to enable JavaScript to run this app.</noscript><div id="root"><div id="loader-block"><svg class="loader-svg-container" viewBox="22 22 44 44"><circle class="loader-style MuiCircularProgress-circle MuiCircularProgress-circleIndeterminate" cx="44" cy="44" fill="none" r="20.2" stroke-width="3.6"></circle></svg></div></div></body></html>andry@Andry-PC:~/devops-school-k8s/task3/homework$ curl hello-world.info/web
web-6745ffd5c8-vxqqj
```
When i use curl, an error appears, but everything is ok in the browser.

## Create deploy with emptyDir save data to mountPoint emptyDir, delete pods, check data.
### Deploy nginx_emptydir deployment
```bash
andry@Andry-PC:~/devops-school-k8s/task3/homework$ kubectl apply -f deployment_nginx_emptydir.yaml 
deployment.apps/web-emptydir created
```
### Search our pod
```bash
andry@Andry-PC:~/devops-school-k8s/task3/homework$ kubectl get pod
NAME                            READY   STATUS              RESTARTS   AGE
minio-94fd47554-h96tt           1/1     Running             0          165m
minio-state-0                   1/1     Running             0          164m
nginx                           1/1     Running             1          14h
web-6745ffd5c8-csm5v            1/1     Running             0          80m
web-6745ffd5c8-gbcxv            1/1     Running             0          80m
web-6745ffd5c8-vxqqj            1/1     Running             0          80m
web-emptydir-59ff4c7f8f-qxsrv   0/1     ContainerCreating   0          3s
```
### Connect to pod
```bash
andry@Andry-PC:~/devops-school-k8s/task3/homework$ kubectl exec -it web-emptydir-59ff4c7f8f-qxsrv bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
```
## Make Myfile.txt in /data
```bash
root@web-emptydir-59ff4c7f8f-qxsrv:/# cd data
root@web-emptydir-59ff4c7f8f-qxsrv:/data# echo "aadsad" > Myfile.txt
```
## Install ps and kill main nginx procces to crush pod
```bash
root@web-emptydir-59ff4c7f8f-qxsrv:/data# apt update
root@web-emptydir-59ff4c7f8f-qxsrv:/data# apt install procps 
root@web-emptydir-59ff4c7f8f-qxsrv:/data# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   8852  5364 ?        Ss   12:22   0:00 nginx: master process nginx -g daemon off;
nginx         31  0.0  0.0   9240  2604 ?        S    12:22   0:00 nginx: worker process
nginx         32  0.0  0.0   9240  2604 ?        S    12:22   0:00 nginx: worker process
root          33  0.0  0.0   4096  3316 pts/0    Ss   12:23   0:00 bash
root         375  0.0  0.0   6696  2972 pts/0    R+   12:23   0:00 ps aux
root@web-emptydir-59ff4c7f8f-qxsrv:/data# kill 1
root@web-emptydir-59ff4c7f8f-qxsrv:/data# command terminated with exit code 137
```
### Look at our pod. It restarted
```bash
andry@Andry-PC:~/devops-school-k8s/task3/homework$ kubectl get pod
NAME                            READY   STATUS    RESTARTS     AGE
minio-94fd47554-h96tt           1/1     Running   0            167m
minio-state-0                   1/1     Running   0            165m
nginx                           1/1     Running   1            14h
web-6745ffd5c8-csm5v            1/1     Running   0            82m
web-6745ffd5c8-gbcxv            1/1     Running   0            82m
web-6745ffd5c8-vxqqj            1/1     Running   0            82m
web-emptydir-59ff4c7f8f-qxsrv   1/1     Running   1 (4s ago)   78s
```
### Check data dir
```bash
andry@Andry-PC:~/devops-school-k8s/task3/homework$ kubectl exec -it web-emptydir-59ff4c7f8f-qxsrv bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@web-emptydir-59ff4c7f8f-qxsrv:/# cat /data/Myfile.txt 
aadsad
root@web-emptydir-59ff4c7f8f-qxsrv:/# exit
exit
```
Data saves after crush
### Delete pod and connect to new pod
```bash
andry@Andry-PC:~/devops-school-k8s/task3/homework$ kubectl delete pod  web-emptydir-59ff4c7f8f-qxsrv
pod "web-emptydir-59ff4c7f8f-qxsrv" deleted
andry@Andry-PC:~/devops-school-k8s/task3/homework$ kubectl get pod
NAME                            READY   STATUS              RESTARTS   AGE
minio-94fd47554-h96tt           1/1     Running             0          167m
minio-state-0                   1/1     Running             0          166m
nginx                           1/1     Running             1          14h
web-6745ffd5c8-csm5v            1/1     Running             0          82m
web-6745ffd5c8-gbcxv            1/1     Running             0          82m
web-6745ffd5c8-vxqqj            1/1     Running             0          82m
web-emptydir-59ff4c7f8f-b6qbl   0/1     ContainerCreating   0          3s
andry@Andry-PC:~/devops-school-k8s/task3/homework$ kubectl exec -it web-emptydir-59ff4c7f8f-b6qbl bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
```
### Check data
```bash
root@web-emptydir-59ff4c7f8f-b6qbl:/# ls data
root@web-emptydir-59ff4c7f8f-b6qbl:/# ls /data
root@web-emptydir-59ff4c7f8f-b6qbl:/# 
```
Data empty, because emptydir saves data after crush but not after deleting.