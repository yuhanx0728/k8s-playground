# k8s-playground

## Working with pods
```
kubectl apply -f pod.yml
kubectl get pods --watch
kubectl get pods hello-pod -o yaml
kubectl describe pods hello-pod
kubectl exec hello-pod -- ps aux // execute commands in hello-pod
kubectl exec -it hello-pod -- sh // log into hello-pod
apk add curl // inside hello-pod
curl localhost:8080 // inside hello-pod
exit // inside hello-pod
kubectl logs hello-pod
kubectl delete -f pod.yml
```
## Working with deployments
```
kubectl apply -f deploy.yml
kubectl get deploy hello-deploy
kubectl describe deploy hello-deploy
kubectl get rs // ReplicaSet
kubectl describe rs // ReplicaSet
kubectl delete -f deploy.yml
```
## Working with services
```
kubectl apply -f svc.yml
// for Docker Desktop, go to http://localhost:30001/
kubectl delete -f svc.yml
```
## Rolling updates & rollbacks
```
/* You can specify the --record flag to write the command executed in the 
 * resource annotation kubernetes.io/change-cause. The recorded change is 
 * useful for future introspection.
 */ 
kubectl apply -f deploy-w-rolling-update.yml --record
// for Docker Desktop, go to http://localhost:30001/
// should be able to see a different webpage than before
kubectl rollout status deployment hello-deploy
kubectl get deploy hello-deploy
kubectl describe deploy hello-deploy
kubectl rollout history deployment hello-deploy
kubectl get rs // previous ReplicaSet not deleted
// rollback; imperative commands, need to change YAML file to reflect imperative changes
kubectl rollout undo deployment hello-deploy --to-revision=1
kubectl get deploy hello-deploy
kubectl rollout status deployment hello-deploy
kubectl delete -f deploy.yml
```
## Imperative way to create a service (don't do this)
```
kubectl apply -f deploy.yml
// imperatively create a service
kubectl expose deployment hello-deploy --name=hello-svc --target-port=8080 --type=NodePort
// website deployed and available
kubectl describe svc hello-svc
kubectl delete svc hello-svc
```
## Declarative way to create a service (do this)
```
kubectl apply -f svc.yml
kubectl get svc hello-svc
kubectl describe svc hello-svc
// get Endpoint objects
kubectl get ep hello-svc
kubectl describe ep hello-svc
kubectl delete -f deploy.yml
kubectl delete -f svc.yml
```
## K8s cluster DNS (every Service needs to register with it when created)
```
$ kubectl get svc -n kube-system -l k8s-app=kube-dns
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   192.168.200.10   <none>        53/UDP,53/TCP,9153/TCP   3h44m

$ kubectl get deploy -n kube-system -l k8s-app=kube-dns
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           3h45m

$ kubectl get pods -n kube-system -l k8s-app=kube-dns
NAME                       READY   STATUS    RESTARTS   AGE
coredns-5644d7b6d9-fk4c9   1/1     Running   0          3h45m
coredns-5644d7b6d9-s5zlr   1/1     Running   0          3h45m

$ cat /etc/resolv.conf 
search svc.cluster.local cluster.local default.svc.cluster.local
nameserver 192.168.200.10
options ndots:5
```
## Service Discovery
```
$ kubectl apply -f sd-example.yml
namespace/dev created
namespace/prod created
deployment.apps/enterprise created
deployment.apps/enterprise created
service/ent created
service/ent created
pod/jump-pod created

// to check everything is deployed
kubectl get all -n dev
kubectl get all -n prod

// log into jump pod
kubectl exec -it jump -n dev -- bash
cat /etc/resolv.conf
apt-get update && apt-get install curl -y
curl ent:8080
curl ent.prod.svc.cluster.local:8080
```
## Troubleshooting service discovery
```
// make sure coredns deployment and pods are up and running
$ kubectl get deploy -n kube-system -l k8s-app=kube-dns
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           2d21h

$ kubectl get pods -n kube-system -l k8s-app=kube-dns
NAME                       READY   STATUS    RESTARTS   AGE
coredns-5644d7b6d9-74pv7   1/1     Running   0          2d21h
coredns-5644d7b6d9-s759f   1/1     Running   0          2d21h

// check logs of each coredns pod
$ kubectl logs coredns-5644d7b6d9-74pv7 -n kube-system
2020-02-19T21:31:01.456Z [INFO] plugin/reload: Running configuration...
2020-02-19T21:31:01.457Z [INFO] CoreDNS-1.6.2
2020-02-19T21:31:01.457Z [INFO] linux/amd64, go1.12.8, 795a3eb
CoreDNS-1.6.2
linux/amd64, go1.12.8, 795a3eb

// check kube-dns Service and Endpoint are up and running
$ kubectl get svc kube-dns -n kube-system
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   192.168.200.10   <none>        53/UDP,53/TCP,9153/TCP   2d21h

// Endpoint should have pod IPs at port 53
$ kubectl get ep -n kube-system -l k8s-app=kube-dns
NAME       ENDPOINTS                                                          AGE
kube-dns   192.168.128.24:53,192.168.128.3:53,192.168.128.24:53 + 3 more...   2d21h

// start and log intot a new pod with the image below for troubleshooting
kubectl run -it dnsutils --image gcr.io/kubernetes-e2e-test-images/dnsutils:1.3
nslookup kubernetes
/* Line 2 should be equal to address of kube-dns Service, and
 * line 4 should be equal to address of kubernetes Service(kubectl get svc kubernetes).
 * if nslookup does not work, restart coredns pod.
 */
kubectl delete pod -n kube-system  -l k8s-app=kube-dns
```
## Persistent Volume
```
/* Use GKE instead of Docker for this exercise.
 * Create a SSD disk instance with Google Compute Engine of 10GB,
 * and make sure it is in the same zone as the k8s cluster and name it uber-disk.
 * Run 'gcloud compute disks list' to check that disk is set up.
 */
kubectl apply -f gke-pv.yml
kubectl get pv pv1
kubectl describe pv pv1
```
## Persistent Volume Claim
```
kubectl apply -f gke-pvc.yml
kubectl get pvc pvc1
kubectl describe pvc pvc1

// create a new pod to be binded with pv1
kubectl apply -f volpod.yml
kubectl describe pod volpod

kubectl delete pods volpod
kubectl delete pvc pvc1
kubectl delete pv pv1
```
## Storage Class
```
kubectl apply -f google-sc.yml
kubectl get sc slow
kubectl describe sc slow

kubectl apply -f google-pvc.yml
kubectl get pvc pv-ticket

kubectl apply -f google-pod.yml

kubectl delete pod class-pod
kubectl delete pvc pv-ticket
kubectl delete sc slow
// go to compute engine console to delete the external storage instance
```
## ConfigMap (imperative)
```
// from command line literal values 
kubectl create configmap testmap1 --from-literal shortname=msb.com --from-literal longname=magicsandbox.com
kubectl describe cm testmap1
kubectl get cm testmap1 -o yaml

// from file
kubectl create cm testmap2 --from-file cmfile.txt
kubectl describe cm testmap2

kubectl get cm
```
## ConfigMap (declarative)
```
kubectl apply -f multimap.yml
kubectl apply -f singlemap.yml

/* Use ConfigMap as environment variables  */
kubectl apply -f envpod.yml
kubectl exec -it envpod -- sh
echo $FIRSTNAME $LASTNAME
kubectl exec envpod -- env

/* Use ConfigMap in container startup commands */
kubectl apply -f envpod-startup.yml
kubectl logs envpod-startup -c ctr1
kubectl describe pod envpod-startup

/* Use ConfigMap with volumes */
kubectl apply -f cmpod.yml
kubectl exec volpod -- ls /etc/name
```
