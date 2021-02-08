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