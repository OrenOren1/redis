# Redis on Kubernetes

requierments
```
minikube
Redis 6.x
A laptop / virtual machines with 8GB RAM
```



## Namespace

```
kubectl create ns redis
```

## Storage Class
$ kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml

```
kubectl get storageclass
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  84s
```

## Deployment: Redis node- basic-conf

```
cd /your/system/libary 
kubectl apply -n redis -f redis-configmap.yaml
kubectl apply -n redis -f redis-statefulset.yaml

kubectl -n redis get pods
kubectl -n redis get pv

kubectl -n redis logs redis-0

```

## Test replication status

```
kubectl -n redis exec -it redis-0 sh
redis-cli 
auth a-very-complex-password-here
info replication
```

## Deployment: Redis-TLS Secured
watch [New Security Features in Redis 6 Open Source](https://kind.sigs.k8s.io/docs/user/quick-start/)

create TLS redis.key and redis.crt
```
$openssl genrsa -out redis.key 4098
$req -new -key redis.key -out server.csr

fill in your connection detailes

$openssl x509 -req -days 366 -in server.csr  -signkey redis.key -out server.crt

$kubectl apply -n redis -f redis-configmap-TLSSECURED.yaml


```

verify TLS connection

```
$kubectl -n redis exec -it redis-0 sh

redis-cli --tls --cert ./redis.crt ./redis.key ./cacert ./ca.crt

127.0.0.1:6379>ping
pong 
if you recived a pong massage - you secured connection succseeded 
```
Applying  config map sets redis logging to debug mode
```
$kubectl apply -n redis -f redis-configmap-debbugmode.yaml
```

## Helm chart steps for redis
```
1. install  [ helm cli](https://kind.sigs.k8s.io/docs/user/quick-start/)
   
2. building helm charts - basic mode  / debug mode / TLS secured-   
$helm create [my_chart_name]
   
3. config templates - service , deployment , pvc .

4. config hpa file - min/max replicas 

5. $helm package [my_chart_name] [my_chart_name0.0.x]-\
   my_chrat_name-0.0.x.tgz

6. $helm --dry-run [my_chart_name] [my_chart_name0.0.x].tgz- verify package on compiled yaml file 

7. $helm --dry-run [my_chart_name] [my_chart_name0.0.x].tgz- apply deployment
```
helm charts online 

reccomend helm chart [bitami redis](https://github.com/bitnami/bitnami-docker-redis) -Non-root container images add an extra layer of security and are generally recommended for production environments. 
